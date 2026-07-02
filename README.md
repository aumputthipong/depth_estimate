# Image Depth Estimation

Monocular **depth estimation** from indoor room photos using a **U-Net** architecture with a **ResNet50** encoder. Given a single RGB image, the model predicts a per-pixel depth map (how far each point in the scene is from the camera).

> A deep-learning side project. My primary focus is software development — this repo shows hands-on experience building and training an end-to-end computer-vision model.

---

## Results

Left to right: **input image · ground-truth depth · predicted depth**.

<!-- 📷 PREDICTION IMAGE 1 (Screenshot 181247 — samples 00013 / 00020). Add here: -->
 <img src="https://github.com/user-attachments/assets/8aa8b303-5d35-463a-ab3a-16c81a75439d"   alt="prediction results 1" width="600" />


<!-- 📷 PREDICTION IMAGE 2 (Screenshot 181240 — samples 00000 / 00008). Add here: -->
 <img src="https://github.com/user-attachments/assets/d7730c74-32a9-44c7-aa70-ab4ad386660e"  alt="prediction results 1" width="600" />

- **Best validation MSE ≈ 0.0007** (depth normalized to the `0–1` range).
- Best checkpoint reached around **epoch 7**, with only a negligible increase in error afterward — **no significant overfitting**.
- Training and validation loss both dropped quickly in the first few epochs, then flattened out.

<!-- 📷 TRAINING CURVE — loss vs. epoch. Add here: -->
<img src="https://github.com/user-attachments/assets/860eb60e-f134-4304-b4d9-1615afe2f337"  alt="training curve" width="300" />

---

## Overview

- **Task:** Predict a depth map from a single RGB image (image-to-image regression).
- **Model:** U-Net + ResNet50 encoder (pretrained on ImageNet).
- **Framework:** PyTorch + [`segmentation-models-pytorch`](https://github.com/qubvel-org/segmentation_models.pytorch).

---

## Dataset

[NYU Depth V2](https://cs.nyu.edu/~fergus/datasets/nyu_depth_v2.html) (Silberman et al., ECCV 2012) — downloaded from [Kaggle: Image Depth Estimation](https://www.kaggle.com/datasets/sohaibanwaar1203/image-depth-estimation).

Each sample is a photo of an indoor room paired with a ground-truth depth map, where every pixel encodes a distance value.

**What makes it challenging:** photos are taken from many rooms, angles, and lighting conditions, so the same object or structure can look very different across images — the model has to learn depth cues that generalize across a lot of visual variety.

---

## Approach

### Data pipeline
- **Split:** 80% train / 20% validation via `random_split()`.
- **Loading:** `DataLoader` with `batch_size = 8`.
- **Preprocessing:** resize to `256 × 320`, convert to tensor, normalize the RGB input.

### Model architecture

| Config | Value |
| --- | --- |
| Architecture | U-Net |
| Encoder | ResNet50 |
| Pretrained weights | ImageNet |
| Input channels | 3 (RGB) |
| Output classes | 1 (depth) |
| Activation | None (linear — regression output) |

- **Transfer learning:** the ResNet50 encoder starts from ImageNet weights, so it already knows strong low-level image features.
- **Skip connections:** U-Net links encoder and decoder at matching resolutions, preserving fine spatial detail in the predicted depth map better than a plain CNN.

### Training setup

| Config | Value |
| --- | --- |
| Loss | Mean Squared Error (MSE) |
| Optimizer | Adam |
| Learning rate | 1e-4 |
| Epochs | 8 |

---

## Repository

| File | Description |
| --- | --- |
| `depth_train.ipynb` | Data loading, model definition, and training loop (saves a checkpoint per epoch + the best model). |
| `depth_inference.ipynb` | Loads a trained checkpoint and visualizes predictions against ground truth. |

**Tech stack:** Python · PyTorch · segmentation-models-pytorch · torchvision · Pillow · Matplotlib

---

## How to Run

1. **Download the trained weights** — [`best_model.pth`](https://github.com/aumputthipong/depth_estimate/releases/download/v1.0/best_model.pth) (~124 MB, published under **Releases**) and place it in the project root.

2. **Install dependencies.** ⚠️ **Pillow must be an older version.** Pillow ≥ 10 can no longer resize the 16-bit depth PNGs and raises `ValueError: image has wrong mode`, so pin it to `9.5.0`:

   ```bash
   pip install torch torchvision segmentation-models-pytorch matplotlib pandas
   pip install "Pillow==9.5.0"
   ```

3. **Run** — open `depth_inference.ipynb` and run all cells (restart the kernel first if a newer Pillow was already imported).
