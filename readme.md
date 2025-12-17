# Add a Custom Scene in LeIsaac

This tutorial walks you through how to add a **custom scene** in **LeIsaac**, enabling you to build and evaluate a variety of tasks based on your own environments.

---

## Step 1: Prepare the USD Scene

To add a custom scene in LeIsaac, you first need to prepare a USD-compatible scene using **Marble**.

### 1.1 Create a World in Marble

Navigate to the **[Marble platform](https://marble.worldlabs.ai/)**.

Follow the instructions in the **[Marble documentation](https://docs.worldlabs.ai/)** to create your custom world model. Once you are satisfied with the result, download the following files:


- **Splats file** (`.ply`)
- **High-quality mesh** (`.glb`)

> **Tips:**  
> For best results, please use high-resolution images or videos.  
> It is recommended to refine and finalize the **panorama** before generating the full world.  

---

### 1.2 Convert Splats (PLY) to USDZ

After obtaining the splats file (`.ply`), convert it into **USDZ** format using **NVIDIA 3DGrut**.

#### Install 3DGrut
Download and install the **[3DGrut](https://github.com/nv-tlabs/3dgrut)**.

Follow the installation instructions provided in the repository.

> **Note (RTX 50-Series GPUs):**  
> If you encounter installation issues on RTX 50-series GPUs, here might be helpful:  
> https://github.com/nv-tlabs/3dgrut/issues/167

#### Convert PLY to USDZ

If you have existing Gaussian splat data in **PLY format**, run the following command:

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
- These two assets must then be spatially aligned; in most cases, rotating `/World/Xform` by **180 degrees around the Z axis** is sufficient, although some scenes may also require scaling (commonly ×100) or additional translation and rotation adjustments. Make sure that the Gaussian splats and mesh geometry overlap correctly in the viewport.

https://github.com/user-attachments/assets/9ab50828-8de1-4d55-b243-c320a7c91cac

#### Step 2: Configure Physics and Colliders for the Mesh

- After alignment is complete, configure physics on the collision mesh. Select `/World/Xform` and add physics using the **Rigid Body with Colliders Preset**, then enable **Kinematic** in the Rigid Body settings so the mesh behaves as a static collision object.  
- Next, select `/World/Xform/decimated_mesh` and, under **Physics → Collider**, set the **Approximation** mode to `meshSimplification`. This setup provides accurate collision behavior while maintaining good simulation performance.

https://github.com/user-attachments/assets/ab391d89-e228-4476-b55c-cec093ab25f4

#### Step 3: Optimize Visuals and Export the Final USD

- For improved visual quality, you may optionally **hide the mesh geometry and keep only the Gaussian splats visible**, while still preserving the underlying collision volumes.  
- **Collision visualization** can be enabled when needed for debugging or inspection.  
- Once both rendering and collision behavior are verified, **save the combined scene as a single USD file** (for example, `scene.usd`). This USD file will serve as the scene entry point and will be referenced directly by LeIsaac in subsequent task and environment configurations.

https://github.com/user-attachments/assets/59b924ad-2d7c-48b4-b4d4-875af7268438
