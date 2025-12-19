
# Add a Custom Scene in LeIsaac

This tutorial walks you through how to add a **custom scene** in **LeIsaac**, enabling you to build and evaluate a variety of manipulation tasks using your own environments.

---

## Step 1: Prepare the USD Scene

To add a custom scene in LeIsaac, you first need to prepare a USD-compatible scene using **Marble**.

### 1.1 Create a World in Marble

Navigate to the **[Marble platform](https://marble.worldlabs.ai/)**.

Follow the instructions in the **[Marble documentation](https://docs.worldlabs.ai/)** to create your custom world model.  
Once you are satisfied with the result, download the following files:

- **Splats file** (`.ply`)
- **High-quality mesh (recommended)** (`.glb`) or **Collider mesh** (`.glb`)

> **Tips:**  
> - Use high-resolution images or videos for best reconstruction quality.  
> - It is recommended to refine and finalize the **panorama** before generating the full world.

---

### 1.2 Convert Splats (PLY) to USDZ

The splats file (`.ply`) needs to be converted to **USDZ** format using **NVIDIA 3DGrut**.

#### Install 3DGrut

Download and install **[3DGrut](https://github.com/nv-tlabs/3dgrut)** following the repository instructions.

> **Note (RTX 50-Series GPUs):**  
> If you encounter installation issues, see:  
> https://github.com/nv-tlabs/3dgrut/issues/167

#### Convert PLY to USDZ

```bash
python -m threedgrut.export.scripts.ply_to_usd path/to/your/splats.ply \
    --output_file path/to/output.usdz
````

---

### 1.3 Combine Gaussian Rendering and Mesh Collisions in Isaac Sim

In this step, we combine **Gaussian Splatting** (for high-quality rendering) with **mesh geometry** (for accurate physical collisions) into a single usable USD scene.

#### Step 1: Load and Align Assets

1. Double-click the generated `.usdz` file and locate `default.usda`.
2. Drag `default.usda` into the **Isaac Sim viewport** to load the Gaussian scene.
3. Create an `Xform` at `/World/Xform` and add a reference to `texture_mesh.glb`
   using an **absolute path**.
4. Align `/World/gauss` and `/World/Xform` so that the Gaussian splats and mesh overlap correctly.

   * Typical adjustments include rotations (e.g., 180° around Z) and scaling (often ×100).

#### Step 2: Configure Physics

1. Select `/World/Xform` and apply **Rigid Body with Colliders Preset**.
2. Enable **Kinematic** so the mesh behaves as a static collision object.
3. Select the mesh prim (Type = `Mesh`) and set:

   * **Physics → Collider → Approximation** = `meshSimplification`.

#### Step 3: Export Final Scene

* Optionally hide the mesh geometry while keeping collisions.
* Verify rendering and collision behavior.
* Save the combined scene as a single USD file (e.g., `scene.usd`).

This USD will be used as the **background scene** in the next step.

---

## Step 2: Scene Composition for Tasks

Some manipulation tasks in LeIsaac (e.g., **cloth folding**, **toy cleaning**) are executed **on a table surface**.
To support a wide range of custom scenes, LeIsaac separates:

* **Background scene** (environment geometry)
* **Robot**
* **Task assets** (objects and optional table)

This design makes task execution more robust across scanned or reconstructed environments.

---

### 2.1 Add Robot Asset to the Scene

#### Step 1: Place the Robot

1. Open the background USD exported in **Step 1.3**.
2. Create a new `Xform` prim to serve as the robot container.
3. Add the **SO101 Follower** USD as a **reference** under this `Xform`.
4. Drag the robot to the desired pose in the scene.

Record the robot transform:

* **Translation**: **(x, y, z)**
* **Orientation**: quaternion **(w, x, y, z)**

#### Step 2: Compose the Scene
To compose the scene, please run the follwing script:

```bash
python scripts/tutorials/marble_compose.py \
  --task your_task \
  --background path/to/scene.usd \
  --output path/to/output.usd \
  --assets-base /path/to/assets \
  --target-pos X Y Z \
  --target-quat W X Y Z
```

<details>
<summary><strong>Parameter descriptions for marble_compose.py</strong></summary>

* `--task`: Task type (`toys`, `orange`, `cloth`, `cube`).
* `--background`: Background scene USD (from Step 1.3).
* `--output`: Output composed USD path.
* `--assets-base`: Base directory for task-related asset USDs.
* `--target-pos`: Robot position `(x, y, z)`.
* `--target-quat`: Robot orientation quaternion `(w, x, y, z)`.
* `--include-table`: Include a task-specific table asset
  (see [Table Replacement](#table-replacement)).
* `--dual-arm`: Enable dual-arm configuration
  (see [Dual-Arm Configuration](#dual-arm-configuration)).

</details>

> 💡 **Why include a table option?**
> Custom scenes may not contain a reliable or well-aligned table.
> Enabling `--include-table` inserts a known, well-tested table asset to ensure stable task execution.

---

### 2.2 Table Replacement

Applicable to **cloth** and **toyroom** tasks.

Use this option if your background scene does not provide a stable table surface.

Steps:

1. Create a new `Xform` prim for the table.
2. Reference the task-specific table USD.
3. Disable physics of the loaded table USD.
4. apply **Rigid Body with Colliders Preset** to the`Xform`.
5. Move the table into place and press **Play** once to let it settle under gravity.
6. Record the table transform:

   * **Translation**: `(x, y, z)`
   * **Orientation**: quaternion **(w, x, y, z)**

To compose the scene, please run the follwing script:

```bash
python scripts/tutorials/marble_compose.py \
  --task your_task \
  --background path/to/scene.usd \
  --output path/to/output.usd \
  --assets-base /path/to/assets \
  --target-pos X Y Z \
  --target-quat W X Y Z \
  --include-table
```

---

### 2.3 Dual-Arm Configuration

By default, tasks use a **single-arm** SO101 robot.

For dual-arm tasks, the workflow remains the same with one key assumption:

> **Left-arm reference**
> You still place **one single-arm SO101 robot** in the scene.
> This robot is interpreted as the **left arm**, and its transform is used as the global reference when composing the dual-arm setup.

Enable dual-arm mode, please run the follwing script::

```bash
python scripts/tutorials/marble_compose.py \
  --task your_task \
  --background path/to/scene.usd \
  --output path/to/output.usd \
  --assets-base /path/to/assets \
  --target-pos X Y Z \
  --target-quat W X Y Z \
  --include-table \
  --dual-arm
```

## Step 3: Verify the Scene

After composing the scene USD, you need to verify if it can be loaded and operated correctly.


---

### 3.1 Add Asset Configuration in Code

All scene asset configurations are defined under:

```

leisaac/source/leisaac/leisaac/assets/scenes

```

Take toyroom as exmple:

```python
from pathlib import Path

import isaaclab.sim as sim_utils
from isaaclab.assets import AssetBaseCfg
from leisaac.utils.constant import ASSETS_ROOT

"""Configuration for the Toy Room Scene"""
SCENES_ROOT = Path(ASSETS_ROOT) / "scenes"

LIGHTWHEEL_TOYROOM_USD_PATH = str(SCENES_ROOT / "lightwheel_toyroom" / "scene.usd")

LIGHTWHEEL_TOYROOM_CFG = AssetBaseCfg(
    spawn=sim_utils.UsdFileCfg(
        usd_path=LIGHTWHEEL_TOYROOM_USD_PATH,
    )
)

````

> 🔧 **What you need to change**
>
> * Replace `"LIGHTWHEEL_TOYROOM_USD_PATH"` with your composed USD path.

### 3.2 Verify via teleop_se3_agent.py 




