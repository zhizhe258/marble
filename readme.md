非常抱歉，之前的回复可能在格式连贯性上出现了疏忽。现在为您提供一份**完全对齐 “Add a Custom Task” 风格**、且内容**绝对完整**的 Markdown 源代码。

这份代码涵盖了从 Marble 生成到 LeIsaac 代码配置的所有步骤，去除了冗余，统一了语气。

```markdown
# 🚀 Add a Custom Scene in LeIsaac

This tutorial walks you through how to add a **custom high-fidelity scene** in **LeIsaac**, enabling you to build and evaluate a variety of tasks based on your own environments.

---

## 1. Create Assets in Marble

The first step is to generate the digital twin of your environment using the **Marble** platform. 

Every task environment in LeIsaac starts with these high-fidelity assets.

1.  Navigate to the **[Marble Platform](https://marble.worldlabs.ai/)**.
2.  Follow the **[Marble Documentation](https://docs.worldlabs.ai/)** to create your custom world model.
3.  Once satisfied, download the following assets:
    * **Splats file** (`.ply`): Used for high-quality Gaussian visual rendering.
    * **High-quality mesh** (`.glb`): Recommended for accurate physical collisions.

> [!TIP]
> **For best results:** Use high-resolution images or videos for generation. It is highly recommended to refine and finalize the **panorama** before generating the full 3D world to ensure consistent lighting.

---

## 2. Convert Splats to USDZ

Since Isaac Sim renders Gaussian Splats via USD, you must convert the raw `.ply` data into a USD-compatible format using **NVIDIA 3DGrut**.

### Install 3DGrut
Clone the repository and follow the official installation instructions:
**[GitHub: nv-tlabs/3dgrut](https://github.com/nv-tlabs/3dgrut)**

> [!IMPORTANT]
> **Note for RTX 50-Series GPUs:** If you encounter installation or kernel issues on the latest hardware, refer to this community fix: [Issue #167](https://github.com/nv-tlabs/3dgrut/issues/167).

### Run Conversion
Execute the following command in your terminal to generate the USDZ file:

```bash
python -m threedgrut.export.scripts.ply_to_usd path/to/your/splats.ply \
    --output_file path/to/output.usdz

```

---

## 3. Integrate Visuals and Collisions

In this step, we perform **"Visual-Physics Pairing"**: combining the **Gaussian Splats** (visuals) with the **Mesh Geometry** (physics) inside Isaac Sim to create a simulation-ready scene.

### Step 1: Load and Align Assets

1. **Load Visuals**: Extract your generated `.usdz` file, locate `default.usda`, and drag it into the **Isaac Sim Viewport**. This creates a prim at `/World/gauss`.
2. **Import Collision Mesh**: In the **Stage** panel, create an Xform at `/World/Xform`. Add a **Reference** to your `texture_mesh.glb` file using an **absolute file path**.
3. **Spatial Alignment**: The two assets must overlap perfectly.
* **Common Fix**: Rotate `/World/Xform` by **180° on the Z-axis**.
* **Scaling**: Some scenes may require a scale adjustment (e.g., **100x**).
* *Verification*: Use the wireframe mode to ensure the mesh aligns with the Gaussian splat visual boundaries.



https://www.google.com/search?q=https://github.com/user-attachments/assets/9ab50828-8de1-4d55-b243-c320a7c91cac

### Step 2: Configure Physics and Colliders

To make the mesh interactable while keeping it static:

1. **Add Physics Preset**: Select `/World/Xform` and apply **Add > Physics > Rigid Body with Colliders Preset**.
2. **Set to Kinematic**: In the **Rigid Body** settings, enable **Kinematic**. This ensures the environment doesn't fall due to gravity.
3. **Optimize Collider**: Select `/World/Xform/decimated_mesh`. Under **Physics > Collider**, set **Approximation** to `meshSimplification`.

https://www.google.com/search?q=https://github.com/user-attachments/assets/ab391d89-e228-4476-b55c-cec093ab25f4

### Step 3: Visual Optimization and Export

1. **Refine Visibility**: Click the eye icon for `/World/Xform` to hide the mesh geometry. This keeps only the beautiful Gaussian splats visible while preserving underlying collisions.
2. **Final Export**: Save the combined stage as a single USD file (e.g., `scene.usd`). This will be your primary scene entry point.

https://www.google.com/search?q=https://github.com/user-attachments/assets/59b924ad-2d7c-48b4-b4d4-875af7268438

---

## 4. Add Asset Configuration

Once the scene file is ready, add the asset configuration in code. The root of the LeIsaac source is `source/leisaac/leisaac`.

In `source/leisaac/leisaac`, create `assets/scenes/custom_scene.py` with:

```python
from pathlib import Path

import isaaclab.sim as sim_utils
from isaaclab.assets import AssetBaseCfg
from leisaac.utils.constant import ASSETS_ROOT

"""Configuration for the Custom Scene"""
SCENES_ROOT = Path(ASSETS_ROOT) / "scenes"

# Point to the USD entry file you just exported
CUSTOM_SCENE_USD_PATH = str(SCENES_ROOT / "custom_scene" / "scene.usd")

CUSTOM_SCENE_CFG = AssetBaseCfg(
    spawn=sim_utils.UsdFileCfg(
        usd_path=CUSTOM_SCENE_USD_PATH,
    )
)

```

Here are some notes on the code:

* `CUSTOM_SCENE_USD_PATH` points to the USD entry file for the scene. You can rename the file or variables as needed, just update the references accordingly.
* `CUSTOM_SCENE_CFG` uses `UsdFileCfg` to define how the asset should be spawned. This configuration will be imported by your task environment configuration.

---

## ✅ Checklist

* [ ] Gaussian scene is visible at `/World/gauss`.
* [ ] Mesh is set to `Kinematic` and correctly aligned.
* [ ] Collision approximation is set to `meshSimplification`.
* [ ] `scene.usd` is saved and referenced correctly in `custom_scene.py`.

**Next Step:** With your high-fidelity scene prepared, you can now proceed to **[Add a Custom Task](https://www.google.com/search?q=./add_custom_task.md)** to define the robot's behavior!

```


```
