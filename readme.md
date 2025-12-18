# Add a Custom Scene in LeIsaac

This tutorial walks you through how to add a **custom scene** in **LeIsaac**, enabling you to build and evaluate a variety of tasks based on your own environments.

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
