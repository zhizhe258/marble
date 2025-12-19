
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

### 4.1 Add Robot Asset to the Scene

Load the scene and create an `Xform` prim.  
Add the **SO101 Follower** USD as a reference under this `Xform`, then drag the robot to the desired pose.

Record the final transform:
- **Translation**: `(x, y, z)`
- **Orientation**: quaternion **(w, x, y, z)**

> 💡 If you are using a **dual-arm setup**, see [Dual-Arm Configuration](#dual-arm-configuration).  
> 💡 If you want to replace the default table for **toys** or **cloth** tasks, see [Table Replacement](#table-replacement-optional).

Then run the following script:

```bash
python scripts/tutorials/marble_compose.py \
  --task your_task \
  --background path/to/scene.usd \
  --output path/to/output.usd \
  --assets-base /path/to/assets \
  --target-pos X Y Z \
  --target-quat W X Y Z
````

<details>
<summary><strong>Parameter descriptions for marble_compose.py</strong></summary>

* `--task`: Task type to configure.
  Supported values: `toys`, `orange`, `cloth`, `cube`.

* `--background`: Path to the background scene USD
  (the scene created via **marble**).

* `--output`: Output path of the composed USD scene.

* `--assets-base`: Base directory of asset USD files.

* `--target-pos`: Target robot position `(x, y, z)`.

* `--target-quat`: Target robot orientation quaternion `(w, x, y, z)`.

* `--include-table`: Include a task-specific table in the composed scene
  (see [Table Replacement](#table-replacement-optional)).

* `--dual-arm`: Enable dual-arm configuration
  (see [Dual-Arm Configuration](#dual-arm-configuration)).
  Only supported for `toys` and `cloth`; the **left arm** is used as the reference.

</details>

---

## Table Replacement (Optional)

Applicable to **cloth** and **toyroom** tasks.

Use this option if you want to include the **full table and related assets** instead of the default table.

1. Add a new `Xform` prim for the table.
2. Reference the task-specific table USD.
3. Disable physics properties on the original table.
4. Add a **Collider** and required physics to the new table.
5. Move the table to the desired pose and press **Play** once to let it settle under gravity.
   This helps ensure stability during later data collection.
6. Record the table transform:

   * **Translation**: `(x, y, z)`
   * **Orientation**: quaternion **(w, x, y, z)**

Then run the script with `--include-table` enabled:

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

## Dual-Arm Configuration

The dual-arm setup follows the same workflow as the single-arm configuration, with **one important constraint**:

> [!IMPORTANT]
> **Left-arm reference**
> When placing the robot, always drag and align using the **left arm**.
> The system uses the left arm transform as the global reference.

All other steps remain unchanged.

To enable dual-arm mode, run:

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

```


