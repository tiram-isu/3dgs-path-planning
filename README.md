# Motion Planning using 3D Gaussian Splatting
This repository contains the functionalities for motion planning using 3D Gaussian Splatting (3DGS) scenes as a scene representation for collision detection. 
The 3DGS scene is first converted into a voxel grid by dividing the scene into voxel cubes of a specified size. Then, for each of these voxels it is checked whether it contains at least one 3D Gaussian. If so, the voxel is marked as occupied.

Path Planning is done using the Open Motion Planning Library (OMPL). The voxel grid representation of the 3DGS scene is used for collision checking, ensuring that generated paths don't collide with any obstacles.
Additionally, a maximum distance from the ground where paths are allowed can be specified to ensure the path stays at a realistic distance to the ground.

After a path has been generated, it is smoothed to remove sharp corners. Then, camera orientation is added to the coordinates and the path converted to Nerfstudio format and saved as a .json file.
Optionally, this can then be sent to be rendered by Nerfstudio via a Flask server.

# Features
- 3DGS to voxel grid conversion for collision detection
- Motion Planning using OMPL
- Path visualization using Open3D and data analysis with Matplotlib
- [Optional] Rendering using Nerfstudio

# Project Overview
This repository integrates two submodules for path planning and rendering:
1. `ompl_path_planning`: Uses [OMPL](https://ompl.kavrakilab.org/index.html) for motion planning, running within [WSL 2](https://learn.microsoft.com/de-de/windows/wsl/install) and [Docker](https://www.docker.com/).
2. `nerfstudio_flask`: Hosts a [Flask](https://flask.palletsprojects.com/en/stable/) server to receive generated paths from `ompl_path_planning` and render them using [nerfstudio](https://docs.nerf.studio/).

See the respective README files of these submodules for more details.

# Prerequisites

## Required  
- **Python** `3.12.3`
-  **Docker** and **WSL 2**  
- **Torch** `2.1.2` with **CUDA 11.8**  

## Required Dependencies (inside Docker)  
- **Base Image:** `ry0ka/ompl_python:latest`  
- **System Packages:**  
  - `libfcl-dev`  
  - `libgl1-mesa-glx`  
  - `libgl1-mesa-dev`  
  - `libglu1-mesa`  
  - `python3-dev`  
  - `python3-pip`  
- **Python Packages:**  
  - `numpy==1.24.4`  
  - `open3d`, `rtree`, `python-fcl`, `opencv-python`  
  - `torch==2.1.2+cu118`, `torchvision==0.16.2+cu118`  
  - `networkx==2.5`  

## [Optional] For Rendering with Nerfstudio  
- **Conda**  
- **Nerfstudio** `1.1.5`
- **Flask** `3.0.3`

# Installation
- To set up the environment, build the Docker file located in the `setup` folder:
  ```bash
  cd setup
  docker built -t motion_planning .
  ```
- [Optional] Create conda environment named `nerfstudio`.
- [Optional] Install Nerfstudio in this environment following the [installation guide](https://docs.nerf.studio/quickstart/installation.html).

# Usage
The following parameters need to be set in the main.py script:
- Define the 3DGS scene to be used.
- Configure the voxel grid settings. If a voxel grid with the given parameters already exists, it will be loaded; otherwise, it is created and saved.
- Select path planning algorithms from OMPL.
- Specify start and end coordinate pairs for pathfinding.
- Enable/disable debugging options, such as logging and visualization.
- [Optional] Configure Nerfstudio paths for rendering. These are only required if rendering is enabled.

## Execution
After setting these parameters, the script can be executed like so:
```bash
docker run -it --rm --gpus all -v "$(pwd):/app" -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix motion_planning /usr/bin/python3 /app/main.py
```
or, when using Flask and Nerfstudio to render the camera paths:
First, run the Flask server in the `nerfstudio_flask` submodule:
```bash
python app.py
```
Then, run the path planning code in `ompl_path_planning`:
```bash
docker run -it --rm --gpus all -v "$(pwd):/app" -e DISPLAY=$DISPLAY -p 5005:5005 -v /tmp/.X11-unix:/tmp/.X11-unix motion_planning /usr/bin/python3 /app/main.py
```

# Output
Results are stored in the output/output_name folder, with subdirectories created for each generated path set. This includes:\
- Logs for each planner
- Screenshots of the generated paths
- A summary log and plots with key statistics from all planners (computation time, path lengths, curvatures)
- [Optional] Video of the path rendered using nerfstudio as an `.mp4` file

