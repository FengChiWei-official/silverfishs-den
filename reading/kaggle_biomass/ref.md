---
tags:
  - type/lit
  - topic/learning
  - status/archive
source: 《卡片盒笔记法》 by Sönke Ahrens
---

## Text
```
import warnings
warnings.filterwarnings('ignore')

import os
import numpy as np
import pandas as pd
import cv2
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader
import timm
import albumentations as A
from albumentations.pytorch import ToTensorV2
from tqdm import tqdm
from sklearn.model_selection import train_test_split
from sklearn.metrics import r2_score
import matplotlib.pyplot as plt
import seaborn as sns
import xgboost as xgb

# Set plotting style
sns.set_style('whitegrid')
plt.rcParams['figure.figsize'] = (14, 10)

# ====================================================
# CONFIG
# ====================================================
class Config:
    TRAIN_CSV = '/kaggle/input/csiro-biomass/train.csv'
    IMG_SIZE = 384
    SWIN_MODEL = 'swin_base_patch4_window12_384'
    DINOV2_MODEL = 'vit_large_patch14_dinov2.lvd142m'
    BATCH_SIZE = 16
    EPOCHS = 35
    LR = 2e-5
    WEIGHT_DECAY = 1e-4
    SEED = 42
    DEVICE = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    
    TARGET_MEAN = None
    TARGET_STD = None

# ====================================================
# DATASET
# ====================================================
def get_data():
    df = pd.read_csv(Config.TRAIN_CSV)
    
    df_pivot = df.pivot(index='image_path',
                        columns='target_name',
                        values='target').reset_index()
    df_pivot = df_pivot.fillna(0)

    target_cols = ['Dry_Green_g', 'Dry_Dead_g', 'Dry_Clover_g',
                   'GDM_g', 'Dry_Total_g']

    Config.TARGET_MEAN = df_pivot[target_cols].mean().values.astype(np.float32)
    Config.TARGET_STD = df_pivot[target_cols].std().values.astype(np.float32)

    print("TARGET_MEAN:", Config.TARGET_MEAN)
    print("TARGET_STD :", Config.TARGET_STD)

    return df_pivot, target_cols

class BiomassDataset(Dataset):
    def __init__(self, df, target_cols, transform=None):
        self.df = df.reset_index(drop=True)
        self.target_cols = target_cols
        self.transform = transform

    def __len__(self):
        return len(self.df)

    def __getitem__(self, idx):
        row = self.df.iloc[idx]
        img_path = "/kaggle/input/csiro-biomass/" + row['image_path']

        image = cv2.imread(img_path)
        if image is None:
            image = np.zeros((Config.IMG_SIZE, Config.IMG_SIZE, 3),
                             dtype=np.uint8)
        else:
            image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)

        if self.transform is not None:
            image = self.transform(image=image)['image']

        targets = row[self.target_cols].values.astype(np.float32)
        targets_norm = (targets - Config.TARGET_MEAN) / Config.TARGET_STD

        return image, torch.tensor(targets_norm), img_path

def get_transforms():
    train_tf = A.Compose([
        A.Resize(Config.IMG_SIZE, Config.IMG_SIZE),
        A.HorizontalFlip(p=0.5),
        A.VerticalFlip(p=0.5),
        A.RandomRotate90(p=0.5),
        A.Transpose(p=0.5),
        A.ShiftScaleRotate(
            shift_limit=0.15,
            scale_limit=0.15,
            rotate_limit=30,
            border_mode=cv2.BORDER_REFLECT_101,
            p=0.6
        ),
        A.RandomBrightnessContrast(
            brightness_limit=0.2,
            contrast_limit=0.2,
            p=0.5
        ),
        A.OneOf([
            A.GaussNoise(var_limit=(10, 30)),
            A.GaussianBlur(blur_limit=(3, 5)),
            A.MotionBlur(blur_limit=3),
        ], p=0.3),
        A.CoarseDropout(max_holes=8, max_height=32, max_width=32, p=0.2),
        A.CLAHE(clip_limit=2.0, p=0.3),
        A.Normalize(mean=[0.485, 0.456, 0.406],
                    std=[0.229, 0.224, 0.225]),
        ToTensorV2()
    ])

    val_tf = A.Compose([
        A.Resize(Config.IMG_SIZE, Config.IMG_SIZE),
        A.Normalize(mean=[0.485, 0.456, 0.406],
                    std=[0.229, 0.224, 0.225]),
        ToTensorV2()
    ])

    return train_tf, val_tf

# ====================================================
# SWIN MODEL
# ====================================================
class SwinRegressor(nn.Module):
    def __init__(self):
        super().__init__()
        
        self.backbone = timm.create_model(
            Config.SWIN_MODEL,
            pretrained=True,
            num_classes=0,
            img_size=Config.IMG_SIZE
        )
        
        feat_dim = self.backbone.num_features
        print(f"Swin Base feature dim: {feat_dim}")
        
        self.head = nn.Sequential(
            nn.Linear(feat_dim, 1024),
            nn.LayerNorm(1024),
            nn.GELU(),
            nn.Dropout(0.4),
            
            nn.Linear(1024, 512),
            nn.LayerNorm(512),
            nn.GELU(),
            nn.Dropout(0.3),
            
            nn.Linear(512, 256),
            nn.LayerNorm(256),
            nn.GELU(),
            nn.Dropout(0.2),
            
            nn.Linear(256, 5)
        )
        
    def forward(self, x):
        features = self.backbone.forward_features(x)
        
        # Pool spatial dimensions
        if features.dim() == 4:
            features = features.mean(dim=[1, 2])
        elif features.dim() == 3:
            features = features.mean(dim=1)
        
        out = self.head(features)
        return out

# ====================================================
# TRAIN SWIN
# ====================================================
def train_swin(train_loader, val_loader):
    print("\n" + "="*70)
    print("TRAINING SWIN BASE")
    print("="*70)
    
    model = SwinRegressor().to(Config.DEVICE)
    
    optimizer = optim.AdamW(
        model.parameters(),
        lr=Config.LR,
        weight_decay=Config.WEIGHT_DECAY
    )
    
    scheduler = optim.lr_scheduler.CosineAnnealingLR(
        optimizer,
        T_max=Config.EPOCHS,
        eta_min=1e-7
    )
    
    criterion = nn.SmoothL1Loss()
    scaler = torch.cuda.amp.GradScaler()
    
    best_r2 = -np.inf
    patience = 0
    max_patience = 8
    
    history = {
        'train_loss': [],
        'val_loss': [],
        'val_r2': []
    }
    
    for epoch in range(Config.EPOCHS):
        # Train
        model.train()
        train_loss = 0.0
        
        pbar = tqdm(train_loader, desc=f"Epoch {epoch+1}/{Config.EPOCHS}")
        
        for imgs, tars_norm, _ in pbar:
            imgs = imgs.to(Config.DEVICE)
            tars_norm = tars_norm.to(Config.DEVICE)
            
            optimizer.zero_grad()
            
            with torch.cuda.amp.autocast():
                preds_norm = model(imgs)
                loss = criterion(preds_norm, tars_norm)
            
            scaler.scale(loss).backward()
            scaler.unscale_(optimizer)
            torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
            scaler.step(optimizer)
            scaler.update()
            
            train_loss += loss.item()
            pbar.set_postfix({'loss': loss.item()})
        
        train_loss /= len(train_loader)
        
        # Validate
        model.eval()
        all_preds = []
        all_targs = []
        val_loss = 0.0
        
        with torch.no_grad():
            for imgs, tars_norm, _ in val_loader:
                imgs = imgs.to(Config.DEVICE)
                tars_norm_gpu = tars_norm.to(Config.DEVICE)
                tars_norm_np = tars_norm.numpy()
                
                with torch.cuda.amp.autocast():
                    preds_norm = model(imgs)
                    loss = criterion(preds_norm, tars_norm_gpu)
                    val_loss += loss.item()
                    
                    preds_norm_np = preds_norm.cpu().numpy()
                
                preds = preds_norm_np * Config.TARGET_STD + Config.TARGET_MEAN
                targs = tars_norm_np * Config.TARGET_STD + Config.TARGET_MEAN
                preds = np.clip(preds, 0, None)
                
                all_preds.append(preds)
                all_targs.append(targs)
        
        val_loss /= len(val_loader)
        all_preds = np.concatenate(all_preds, axis=0)
        all_targs = np.concatenate(all_targs, axis=0)
        
        r2_scores = [r2_score(all_targs[:, i], all_preds[:, i])
                     for i in range(5)]
        
        weights = np.array([0.1, 0.1, 0.1, 0.2, 0.5])
        weighted_r2 = np.average(r2_scores, weights=weights)
        
        scheduler.step()
        
        history['train_loss'].append(train_loss)
        history['val_loss'].append(val_loss)
        history['val_r2'].append(weighted_r2)
        
        print(f"Ep{epoch+1:02d} | TrLoss: {train_loss:.4f} | "
              f"ValLoss: {val_loss:.4f} | R²: {weighted_r2:.4f}")
        print(f"  → Green={r2_scores[0]:.3f}, Dead={r2_scores[1]:.3f}, "
              f"Clover={r2_scores[2]:.3f}, GDM={r2_scores[3]:.3f}, "
              f"Total={r2_scores[4]:.3f}")
        
        if weighted_r2 > best_r2:
            best_r2 = weighted_r2
            patience = 0
            torch.save(model.state_dict(), 'swin_best.pth')
            print(f"  ✓ Saved (best R²: {best_r2:.4f})")
        else:
            patience += 1
        
        if patience >= max_patience:
            print("  ⚠ Early stopping")
            break
    
    # Load best model
    model.load_state_dict(torch.load('swin_best.pth'))
    return model, history

# ====================================================
# EXTRACT DINOV2 EMBEDDINGS
# ====================================================
def extract_dinov2_embeddings(df, target_cols):
    print("\n" + "="*70)
    print("EXTRACTING DINOv2 EMBEDDINGS")
    print("="*70)
    
    # Load DINOv2 model
    dinov2 = timm.create_model(
        Config.DINOV2_MODEL,
        pretrained=True,
        num_classes=0
    ).to(Config.DEVICE)
    dinov2.eval()
    
    print(f"DINOv2 feature dim: {dinov2.num_features}")
    
    # DINOv2 expects 518x518 images
    DINOV2_SIZE = 518
    
    transform = A.Compose([
        A.Resize(DINOV2_SIZE, DINOV2_SIZE),
        A.Normalize(mean=[0.485, 0.456, 0.406],
                    std=[0.229, 0.224, 0.225]),
        ToTensorV2()
    ])
    
    embeddings = []
    targets = []
    
    with torch.no_grad():
        for idx, row in tqdm(df.iterrows(), total=len(df), desc="Extracting"):
            img_path = "/kaggle/input/csiro-biomass/" + row['image_path']
            
            image = cv2.imread(img_path)
            if image is None:
                image = np.zeros((DINOV2_SIZE, DINOV2_SIZE, 3),
                               dtype=np.uint8)
            else:
                image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
            
            image = transform(image=image)['image'].unsqueeze(0).to(Config.DEVICE)
            
            with torch.cuda.amp.autocast():
                emb = dinov2.forward_features(image)
                
                # Pool to get fixed size
                if emb.dim() == 4:
                    emb = emb.mean(dim=[2, 3])
                elif emb.dim() == 3:
                    emb = emb.mean(dim=1)
                
                embeddings.append(emb.cpu().numpy())
            
            targs = row[target_cols].values.astype(np.float32)
            targets.append(targs)
    
    embeddings = np.vstack(embeddings)
    targets = np.array(targets)
    
    print(f"Embeddings shape: {embeddings.shape}")
    print(f"Targets shape: {targets.shape}")
    
    return embeddings, targets

# ====================================================
# TRAIN XGBOOST
# ====================================================
def train_xgboost(X_train, y_train, X_val, y_val):
    print("\n" + "="*70)
    print("TRAINING XGBOOST ON EMBEDDINGS")
    print("="*70)
    
    models = []
    
    for i, target_name in enumerate(['Green', 'Dead', 'Clover', 'GDM', 'Total']):
        print(f"\nTraining {target_name}...")
        
        xgb_model = xgb.XGBRegressor(
            n_estimators=500,
            max_depth=8,
            learning_rate=0.05,
            subsample=0.8,
            colsample_bytree=0.8,
            random_state=Config.SEED,
            tree_method='hist',
            device='cuda'
        )
        
        xgb_model.fit(
            X_train, y_train[:, i],
            eval_set=[(X_val, y_val[:, i])],
            verbose=False
        )
        
        val_pred = xgb_model.predict(X_val)
        val_pred = np.clip(val_pred, 0, None)
        r2 = r2_score(y_val[:, i], val_pred)
        print(f"  Val R²: {r2:.4f}")
        
        models.append(xgb_model)
    
    return models

# ====================================================
# FIND OPTIMAL ENSEMBLE WEIGHTS
# ====================================================
def find_best_ensemble_weights(swin_preds, xgb_preds, targets):
    """Find optimal ensemble weights via grid search"""
    print("\n" + "="*70)
    print("SEARCHING FOR OPTIMAL ENSEMBLE WEIGHTS")
    print("="*70)
    
    best_weight = 0.0
    best_r2 = -np.inf
    weights_array = np.array([0.1, 0.1, 0.1, 0.2, 0.5])
    
    results = []
    
    for swin_weight in np.arange(0.0, 1.01, 0.05):
        xgb_weight = 1.0 - swin_weight
        ensemble = swin_weight * swin_preds + xgb_weight * xgb_preds
        
        r2_scores = [r2_score(targets[:, i], ensemble[:, i]) for i in range(5)]
        weighted_r2 = np.average(r2_scores, weights=weights_array)
        
        results.append((swin_weight, weighted_r2))
        
        if weighted_r2 > best_r2:
            best_r2 = weighted_r2
            best_weight = swin_weight
    
    print(f"\nBest Swin weight: {best_weight:.2f}")
    print(f"Best XGBoost weight: {1-best_weight:.2f}")
    print(f"Best ensemble R²: {best_r2:.4f}")
    
    # Plot weight search
    weights_list = [r[0] for r in results]
    r2_list = [r[1] for r in results]
    
    plt.figure(figsize=(10, 6))
    plt.plot(weights_list, r2_list, linewidth=2.5, marker='o', markersize=4)
    plt.axvline(x=best_weight, color='red', linestyle='--', linewidth=2, 
                label=f'Best: {best_weight:.2f}')
    plt.axhline(y=best_r2, color='green', linestyle=':', linewidth=1.5, alpha=0.7)
    plt.xlabel('Swin Weight', fontsize=13, fontweight='bold')
    plt.ylabel('Weighted R² Score', fontsize=13, fontweight='bold')
    plt.title('Ensemble Weight Optimization', fontsize=15, fontweight='bold')
    plt.legend(fontsize=11)
    plt.grid(True, alpha=0.3)
    plt.tight_layout()
    plt.savefig('ensemble_weight_search.png', dpi=150, bbox_inches='tight')
    print("✓ Saved plot: ensemble_weight_search.png")
    plt.show()
    
    return best_weight

# ====================================================
# PLOT TRAINING HISTORY
# ====================================================
def plot_history(history):
    fig, ax1 = plt.subplots(1, 1, figsize=(12, 6))
    
    epochs = range(1, len(history['train_loss']) + 1)
    
    # Plot losses
    ax1.plot(epochs, history['train_loss'], 
            label='Train Loss', color='blue', linewidth=2.5, marker='o')
    ax1.plot(epochs, history['val_loss'], 
            label='Val Loss', color='orange', linewidth=2.5, 
            linestyle='--', marker='s')
    ax1.set_xlabel('Epoch', fontsize=13, fontweight='bold')
    ax1.set_ylabel('Loss', fontsize=13, fontweight='bold')
    ax1.tick_params(axis='y')
    ax1.grid(True, alpha=0.3)
    
    # Plot R² on second axis
    ax2 = ax1.twinx()
    best_r2 = max(history['val_r2'])
    ax2.plot(epochs, history['val_r2'], 
            label=f'Val R² (best: {best_r2:.4f})', 
            color='green', linewidth=2.5, marker='D', linestyle='-.')
    ax2.axhline(y=best_r2, color='red', linestyle=':', linewidth=2, alpha=0.6)
    ax2.set_ylabel('R² Score', fontsize=13, fontweight='bold', color='green')
    ax2.tick_params(axis='y', labelcolor='green')
    
    # Combined legend
    lines1, labels1 = ax1.get_legend_handles_labels()
    lines2, labels2 = ax2.get_legend_handles_labels()
    ax1.legend(lines1 + lines2, labels1 + labels2, 
              loc='upper right', fontsize=11, framealpha=0.9)
    
    plt.title('Swin Base Training History', fontsize=15, fontweight='bold', pad=15)
    plt.tight_layout()
    plt.savefig('swin_training_history.png', dpi=150, bbox_inches='tight')
    print("\n✓ Saved plot: swin_training_history.png")
    plt.show()

# ====================================================
# SAVE MODELS
# ====================================================
def save_models(swin_model, xgb_models, best_swin_weight):
    """Save all trained models and ensemble configuration"""
    import pickle
    import json
    
    print("\n" + "="*70)
    print("SAVING MODELS")
    print("="*70)
    
    # Save Swin model (already saved as 'swin_best.pth' during training)
    print("✓ Swin model saved: swin_best.pth")
    
    # Save XGBoost models
    for i, model in enumerate(xgb_models):
        target_name = ['green', 'dead', 'clover', 'gdm', 'total'][i]
        filename = f'xgboost_{target_name}.json'
        model.save_model(filename)
        print(f"✓ XGBoost model saved: {filename}")
    
    # Save ensemble configuration
    ensemble_config = {
        'swin_weight': float(best_swin_weight),
        'xgboost_weight': float(1 - best_swin_weight),
        'target_mean': Config.TARGET_MEAN.tolist(),
        'target_std': Config.TARGET_STD.tolist(),
        'target_names': ['Dry_Green_g', 'Dry_Dead_g', 'Dry_Clover_g', 'GDM_g', 'Dry_Total_g'],
        'swin_img_size': Config.IMG_SIZE,
        'dinov2_img_size': 518,
        'swin_model_name': Config.SWIN_MODEL,
        'dinov2_model_name': Config.DINOV2_MODEL
    }
    
    with open('ensemble_config.json', 'w') as f:
        json.dump(ensemble_config, f, indent=4)
    print("✓ Ensemble config saved: ensemble_config.json")
    
    print("\nAll models saved successfully!")
    print("="*70)

# ====================================================
# MAIN
# ====================================================
if __name__ == "__main__":
    torch.manual_seed(Config.SEED)
    np.random.seed(Config.SEED)
    
    print("="*70)
    print("SWIN BASE + XGBOOST ENSEMBLE")
    print("="*70)
    
    # Load data
    df, target_cols = get_data()
    
    # 80:20 split
    train_df, val_df = train_test_split(
        df, test_size=0.2, random_state=Config.SEED
    )
    
    print(f"\nTrain: {len(train_df)}, Val: {len(val_df)}")
    
    # Prepare data loaders
    train_tf, val_tf = get_transforms()
    
    train_ds = BiomassDataset(train_df, target_cols, transform=train_tf)
    val_ds = BiomassDataset(val_df, target_cols, transform=val_tf)
    
    train_loader = DataLoader(train_ds, batch_size=Config.BATCH_SIZE,
                              shuffle=True, num_workers=4, pin_memory=True)
    val_loader = DataLoader(val_ds, batch_size=Config.BATCH_SIZE,
                            shuffle=False, num_workers=4, pin_memory=True)
    
    # 1. Train Swin
    swin_model, history = train_swin(train_loader, val_loader)
    plot_history(history)
    
    # 2. Get Swin predictions
    print("\n" + "="*70)
    print("GETTING SWIN PREDICTIONS")
    print("="*70)
    
    swin_model.eval()
    swin_val_preds = []
    val_targets = []
    
    with torch.no_grad():
        for imgs, tars_norm, _ in val_loader:
            imgs = imgs.to(Config.DEVICE)
            tars_norm_np = tars_norm.numpy()
            
            with torch.cuda.amp.autocast():
                preds_norm = swin_model(imgs).cpu().numpy()
            
            preds = preds_norm * Config.TARGET_STD + Config.TARGET_MEAN
            targs = tars_norm_np * Config.TARGET_STD + Config.TARGET_MEAN
            preds = np.clip(preds, 0, None)
            
            swin_val_preds.append(preds)
            val_targets.append(targs)
    
    swin_val_preds = np.concatenate(swin_val_preds, axis=0)
    val_targets = np.concatenate(val_targets, axis=0)
    
    # 3. Extract DINOv2 embeddings
    train_embs, train_targets = extract_dinov2_embeddings(train_df, target_cols)
    val_embs, _ = extract_dinov2_embeddings(val_df, target_cols)
    
    # 4. Train XGBoost
    xgb_models = train_xgboost(train_embs, train_targets, val_embs, val_targets)
    
    # 5. Get XGBoost predictions
    print("\n" + "="*70)
    print("GETTING XGBOOST PREDICTIONS")
    print("="*70)
    
    xgb_val_preds = np.column_stack([
        np.clip(model.predict(val_embs), 0, None) 
        for model in xgb_models
    ])
    
    # 6. Find optimal ensemble weights
    best_swin_weight = find_best_ensemble_weights(
        swin_val_preds, xgb_val_preds, val_targets
    )
    
    # 7. Create ensembles
    print("\n" + "="*70)
    print("ENSEMBLE RESULTS")
    print("="*70)
    
    ensemble_simple = (swin_val_preds + xgb_val_preds) / 2
    ensemble_optimized = (best_swin_weight * swin_val_preds + 
                          (1 - best_swin_weight) * xgb_val_preds)
    
    # Calculate R² scores
    target_names = ['Dry_Green_g', 'Dry_Dead_g', 'Dry_Clover_g', 'GDM_g', 'Dry_Total_g']
    weights = np.array([0.1, 0.1, 0.1, 0.2, 0.5])
    
    print("\n" + "-"*85)
    print(f"{'Model':<20} {'Green':<9} {'Dead':<9} {'Clover':<9} {'GDM':<9} {'Total':<9} {'Weighted':<10}")
    print("-"*85)
    
    for model_name, preds in [('Swin', swin_val_preds), 
                               ('XGBoost (DINOv2)', xgb_val_preds), 
                               ('Ensemble (50/50)', ensemble_simple),
                               ('Ensemble (Optimal)', ensemble_optimized)]:
        r2_scores = [r2_score(val_targets[:, i], preds[:, i]) for i in range(5)]
        weighted_r2 = np.average(r2_scores, weights=weights)
        
        print(f"{model_name:<20} ", end="")
        for r2 in r2_scores:
            print(f"{r2:<9.4f} ", end="")
        print(f"{weighted_r2:<10.4f}")

    print("="*70)
    # Print final ensemble weight prominently
    print("\n" + "="*70)
    print("FINAL ENSEMBLE CONFIGURATION")
    print("="*70)
    print(f"Swin Weight:    {best_swin_weight:.4f}")
    print(f"XGBoost Weight: {1-best_swin_weight:.4f}")
    print("\nFor inference, use:")
    print(f"  ensemble_pred = {best_swin_weight:.4f} * swin_pred + {1-best_swin_weight:.4f} * xgb_pred")
    print("="*70)

    # 8. SAVE ALL MODELS
    save_models(swin_model, xgb_models, best_swin_weight)
    print("="*85)
    
    # Additional analysis
    print("\n" + "="*70)
    print("KEY INSIGHTS")
    print("="*70)
    
    swin_r2 = np.average([r2_score(val_targets[:, i], swin_val_preds[:, i]) 
                          for i in range(5)], weights=weights)
    xgb_r2 = np.average([r2_score(val_targets[:, i], xgb_val_preds[:, i]) 
                         for i in range(5)], weights=weights)
    opt_r2 = np.average([r2_score(val_targets[:, i], ensemble_optimized[:, i]) 
                         for i in range(5)], weights=weights)
    
    print(f"• DINOv2 embeddings capture fine-grained texture features")
    print(f"• XGBoost outperforms Swin by {(xgb_r2-swin_r2)/swin_r2*100:.1f}%")
    print(f"• Optimal ensemble improves over XGBoost by {(opt_r2-xgb_r2)/xgb_r2*100:.1f}%")
    print(f"• Optimal weight: {best_swin_weight:.2f} Swin + {1-best_swin_weight:.2f} XGBoost")
    print("="*70)
```

## Thoughts