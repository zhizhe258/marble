# Add a Custom Scene in LeIsaac

This tutorial walks you through how to add a **custom scene** in **LeIsaac**, enabling you to build and evaluate a variety of tasks based on your own environments.

## 🎥 Custom Scene Simulation
<table>
  <tr>
    <td width="50%" align="center">
      <div>
        <video
          src="https://github.com/user-attachments/assets/5919f96d-b93c-449e-b0b6-a39114faa4a9"
          controls
        ></video>
      </div>
    </td>
    <td width="50%" align="center">
      <div>
        <video
          src="https://github.com/user-attachments/assets/3cb52d1e-827d-4047-b010-e347bea0ad41"
          controls
        ></video>
      </div>
    </td>
  </tr>
</table>

---

## Step 1: Prepare the USD Scene

To add a custom scene in LeIsaac, you first need to prepare a USD-compatible scene using **Marble**.

### 1.1 Create a World in Marble

Navigate to the **[Marble platform](https://marble.worldlabs.ai/)**.

Follow the instructions in the **[Marble documentation](https://docs.worldlabs.ai/)** to create your custom world model. Once you are satisfied with the result, download the following files:


- **Splats file** (`.ply`)
- **High-quality mesh(recommended)** (`.glb`) or **Collider mesh** (`.glb`)

> **Tips:**  
> For best results, please use high-resolution images or videos.  
> It is recommended to refine and finalize the **panorama** before generating the full world.

---

### 1.2 Convert Splats (PLY) to USDZ

After obtaining the splats file (`.ply`), it needs to be converted to **USDZ** format using **NVIDIA 3DGrut**.

#### Install 3DGrut
Download and install the **[3DGrut](https://github.com/nv-tlabs/3dgrut)**.

Follow the installation instructions provided in the repository.

> **Note (RTX 50-Series GPUs):**  
> If you encounter installation issues on RTX 50-series GPUs, here might be helpful:  
> https://github.com/nv-tlabs/3dgrut/issues/167

#### Convert PLY to USDZ

To convert splat data **PLY format** to **USDZ** format, run the following command:

```bash
python -m threedgrut.export.scripts.ply_to_usd path/to/your/splats.ply \
    --output_file path/to/output.usdz
```

---

### 1.3 Integrate Gaussian Rendering and Mesh Collisions in Isaac Sim

In this step, we combine **Gaussian Splatting** for high-quality visual rendering with **mesh geometry** for accurate physical collisions. The result is a single, complete USD scene that can be directly used for simulation and interaction in LeIsaac.

#### Step 1: Load and Align the Gaussian Scene and Collision Mesh

- Begin by double-clicking the generated `.usdz` file to extract its contents. Locate `default.usda` in the extracted folder and drag it into the **Isaac Sim GUI viewport** to load the Gaussian splatting scene used for rendering.  
- Next, in the **Stage** panel, create an Xform at `/World/Xform`, select it, and add a reference to the `texture_mesh.glb` file using an **absolute file path**. At this point, the scene should contain `/World/gauss` for Gaussian rendering and `/World/Xform` for mesh-based collisions.  
- Before adjusting the mesh, first ensure that `/World/gauss` is **aligned with the world coordinate system** (i.e., its local axes and orientation are consistent with `/World`). Then **align `/World/Xform` to match the Gaussian scene**. In most cases, rotating `/World/Xform` by **180 degrees around the Z axis** is sufficient, although some scenes may also require scaling (commonly ×100) or additional translation and rotation adjustments. In this example, `/World/gauss` is first **rotated 180 degrees around the X axis**, followed by rotating `/World/Xform` **90 degrees around the X axis** and then **180 degrees around the Z axis**. **Make sure that the Gaussian splats and mesh geometry overlap correctly in the viewport.**

https://github.com/user-attachments/assets/e610ee7c-9bf5-4bf8-84fd-e42510012371

#### Step 2: Configure Physics and Colliders for the Mesh

- After alignment is complete, configure physics on the collision mesh. Select `/World/Xform` and add physics using the **Rigid Body with Colliders Preset**, then enable **Kinematic** in the Rigid Body settings so the mesh behaves as a static collision object.  
- Next, locate the mesh prim under `/World/Xform` (typically `/World/Xform/decimated_mesh` or `/World/Xform/decimated_mesh/Mesh0`, i.e., **the prim whose Type is `Mesh`**).Under **Physics → Collider**, set the **Approximation** mode to `meshSimplification`. This setup provides accurate collision behavior while maintaining good simulation performance.

https://github.com/user-attachments/assets/a84133a2-63dd-4182-bc73-e7f3e17e0f0f

#### Step 3: Optimize Visuals and Export the Final USD

- For improved visual quality, you may optionally **hide the mesh geometry and keep only the Gaussian splats visible**, while still preserving the underlying collision volumes.  
- **Collision visualization** can be enabled when needed for debugging or inspection.  
- Once both rendering and collision behavior are verified, **save the combined scene as a single USD file** (for example, `scene.usd`). This USD file will serve as the scene entry point and will be referenced directly by LeIsaac in subsequent task and environment configurations.

https://github.com/user-attachments/assets/0b8ded40-aa41-4e32-9a1d-360a4241ea91




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



https://github.com/user-attachments/assets/9fb82505-ef36-4d86-a913-919b1866ed83



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
  


https://github.com/user-attachments/assets/56c705a6-6ecf-4e40-851e-024dade9f918



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

### 3.1 Replace the default scene with custom scene

All scene configurations are defined under:

```
leisaac/source/leisaac/leisaac/assets/scenes
```

Take toyroom as an example:

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

### 3.2 Verify via Teleoperation (`teleop_se3_agent.py`)

After updating the task configuration, use the teleoperation script to **verify that the scene is correctly composed**.

Run the teleoperation script:

```bash
python scripts/environments/teleoperation/teleop_se3_agent.py \
    --task=LeIsaac-SO101-CleanToyTable-v0 \
    --teleop_device=so101leader \
    --port=/dev/ttyACM0 \
    --num_envs=1 \
    --device=cuda \
    --enable_cameras \
    --record \
    --dataset_file=./datasets/dataset.hdf5
```

[notable_record.webm](https://github.com/user-attachments/assets/b64f6e5b-c97f-45bf-be9d-308af2d4adee)



