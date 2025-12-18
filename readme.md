# 🚀 Add a Custom Scene in LeIsaac

This tutorial walks you through how to add a **custom scene** in **LeIsaac**, enabling you to build and evaluate a variety of tasks based on your own high-fidelity environments.

---

## 📋 Prerequisites

Before starting, ensure you have the following ready:
* **Omniverse Isaac Sim** (2023.1.1 or later recommended)
* **Python 3.10+** environment
* **NVIDIA RTX GPU** (Compatible with Gaussian Splatting rendering)

---

## 🛠 Step 1: Prepare the USD Scene

To add a custom scene, you need to bridge the gap between visual Splats and physical Colliders.

### 1.1 Create a World in Marble

1.  Navigate to the **[Marble Platform](https://marble.worldlabs.ai/)**.
2.  Follow the **[Marble Documentation](https://docs.worldlabs.ai/)** to create your custom world model. 
3.  Once satisfied, download the following assets:
    * **Splats file** (`.ply`)
    * **High-quality mesh** (`.glb`) — *Recommended for collision accuracy.*
    * **Collider mesh** (`.glb`) — *Alternative for simpler scenes.*

> [!TIP]
> **For best results:** Use high-resolution images or videos. It is highly recommended to refine and finalize the **panorama** before generating the full 3D world.

---

### 1.2 Convert Splats (PLY) to USDZ

Since Isaac Sim renders Gaussian Splats via USD, we use **NVIDIA 3DGrut** for conversion.

#### 1. Install 3DGrut
Clone the repository and follow the official installation instructions:
**[GitHub: nv-tlabs/3dgrut](https://github.com/nv-tlabs/3dgrut)**

> [!IMPORTANT]
> **Note for RTX 50-Series GPUs:** > If you encounter installation or kernel issues on the latest hardware, refer to this community fix: [Issue #167](https://github.com/nv-tlabs/3dgrut/issues/167).

#### 2. Convert PLY to USDZ
Run the following command in your terminal:

```bash
python -m threedgrut.export.scripts.ply_to_usd path/to/your/splats.ply \
    --output_file path/to/output.usdz

### 1.3 Integrate Gaussian Rendering and Mesh Collisions in Isaac Sim

In this step, we combine **Gaussian Splatting** (for high-quality visuals) with **Simplified Mesh Geometry** (for physical collisions). This creates a complete USD scene ready for physical interaction in LeIsaac.

---

#### 🛠 Step 1: Load and Align Assets

1.  **Load Gaussian Scene**: 
    * Extract your generated `.usdz` file.
    * Locate `default.usda` in the extracted folder and drag it into the **Isaac Sim Viewport**. 
    * This usually creates a prim at `/World/gauss`.
2.  **Import Collision Mesh**:
    * In the **Stage** panel, create an Xform at `/World/Xform`.
    * With the Xform selected, add a **Reference** to your `texture_mesh.glb` file.
3.  **Spatial Alignment**:
    * The two assets must overlap perfectly. 
    * **Common Fix**: Rotate `/World/Xform` by **180° on the Z-axis**.
    * **Scaling**: Some scenes may require a scale adjustment (e.g., **100x**).
    * *Verification*: Use the wireframe mode to ensure the mesh aligns with the Gaussian splat visual boundaries.

> [!TIP]
> Always use **absolute file paths** when adding references to ensure the USD scene can find the assets regardless of where it is saved.

https://github.com/user-attachments/assets/9ab50828-8de1-4d55-b243-c320a7c91cac

---

#### ⚙️ Step 2: Configure Physics and Colliders

To make the mesh interactable while keeping it static in the world:

1.  **Add Physics Preset**: Select `/World/Xform` and apply **Add > Physics > Rigid Body with Colliders Preset**.
2.  **Set to Kinematic**: In the **Rigid Body** settings, enable **Kinematic**. This prevents the mesh from falling due to gravity while still allowing it to collide with robots.
3.  **Optimize Collision Mesh**:
    * Select `/World/Xform/decimated_mesh`.
    * Navigate to **Physics > Collider**.
    * Set **Approximation** to `meshSimplification`. This balances collision accuracy and simulation speed.

https://github.com/user-attachments/assets/ab391d89-e228-4476-b55c-cec093ab25f4

---

#### 💾 Step 3: Visual Optimization and Export

1.  **Refine Visibility**: You can hide the mesh geometry (click the eye icon for `/World/Xform`) to keep only the beautiful Gaussian splats visible. The physics engine will still recognize the hidden mesh for collisions.
2.  **Debug Collisions**: If needed, enable **Physics > Show Collisions** to verify the interaction boundaries.
3.  **Final Export**: Save the stage as a single USD file (e.g., `scene.usd`). 

> [!IMPORTANT]
> This `scene.usd` file will serve as the entry point for LeIsaac. Ensure all relative paths are correctly maintained if you move the file later.

https://github.com/user-attachments/assets/59b924ad-2d7c-48b4-b4d4-875af7268438

---
**Checklist before moving to Section 1.4:**
- [ ] Gaussian scene is visible at `/World/gauss`.
- [ ] Mesh is set to `Kinematic` and correctly aligned.
- [ ] Collision approximation is set to `meshSimplification`.
- [ ] Scene is saved as a `.usd` file.
