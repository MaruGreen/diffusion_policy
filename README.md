# Diffusion Policy

## 🛠️ Installation
### 🖥️ For Simulation
For Mujoco:
```
$ sudo apt install -y libosmesa6-dev libgl1-mesa-glx libglfw3 patchelf
```
For Space-Mouse:
```
$ sudo apt install libspnav-dev spacenavd
$ sudo systemctl start spacenavd
```
Install [miniforge](https://github.com/conda-forge/miniforge/releases) and run:
```
$ mamba env create -f conda_environment.yaml
```

## 🖥️ Reproducing Simulation Benchmark (Push-T) Results 
### Obtain Training Data by Download or Demonstration
Download dataset provided by the author:
```
$ mkdir data && cd data
$ wget https://diffusion-policy.cs.columbia.edu/data/training/pusht.zip
$ unzip pusht.zip && rm -f pusht.zip && cd ..
```
Or, generate dataset by demonstration:
```
$ mamba activate robodiff
$ python demo_pusht.py -o data/pusht_demo.zarr
```

### Download and Edit Config Yaml
```
$ wget -O image_pusht_cnn.yaml https://diffusion-policy.cs.columbia.edu/data/experiments/image/pusht/diffusion_policy_cnn/config.yaml
```

### Train with A Single Seed
```
$ mamba activate robodiff
$ python train.py --config-dir=. --config-name=image_pusht_cnn.yaml training.seed=43 training.device=cuda:0 hydra.run.dir='data/outputs/${now:%Y.%m.%d}/${now:%H.%M.%S}_${name}_${task_name}'
```

### 🆕 Evaluate Pre-trained Checkpoints
Run the evaluation script:
```
$ mamba activate robodiff
$ python eval.py --checkpoint xxx.ckpt --output_dir xxx --device cuda:0
```

### 🦾 For Real Robot
Hardware (for Push-T):
* 1x [UR5-CB3](https://www.universal-robots.com/cb3) or [UR5e](https://www.universal-robots.com/products/ur5-robot/) ([RTDE Interface](https://www.universal-robots.com/articles/ur/interface-communication/real-time-data-exchange-rtde-guide/) is required)
* 2x [RealSense D435](https://www.intelrealsense.com/depth-camera-d435/)
* 2x [RealSense L515](https://www.intelrealsense.com/lidar-camera-l515/)
* 1x [3Dconnexion SpaceMouse](https://3dconnexion.com/us/product/spacemouse-wireless/) (for teleop)
* 1x [Millibar Robotics Manual Tool Changer](https://www.millibar.com/manual-tool-changer/) (only need robot side)
* 1x 3D printed [End effector](https://cad.onshape.com/documents/a818888644a15afa6cc68ee5/w/2885b48b018cda84f425beca/e/3e8771c2124cee024edd2fed?renderMode=0&uiState=63ffcba6631ca919895e64e5)
* 1x 3D printed [T-block](https://cad.onshape.com/documents/f1140134e38f6ed6902648d5/w/a78cf81827600e4ff4058d03/e/f35f57fb7589f72e05c76caf?renderMode=0&uiState=63ffcbc9af4a881b344898ee)
* USB-C cables and screws for RealSense

Software:
* Ubuntu 20.04.3
* [RealSense SDK](https://github.com/IntelRealSense/librealsense/blob/master/doc/distribution_linux.md)
* Virtual environment:
```
$ mamba env create -f conda_environment_real.yaml
```

## 🦾 Demo, Training and Eval on a Real Robot
Make sure your UR5 robot is running and accepting command from its network interface (emergency stop button within reach at all time), your RealSense cameras plugged in to your workstation (tested with `realsense-viewer`) and your SpaceMouse connected with the `spacenavd` daemon running (verify with `systemctl status spacenavd`).

Start the demonstration collection script. Press 'c' to start recording.
Use SpaceMouse to move the robot. Press 's' to stop recording. 
```
$ mamba activate robodiffreal
$ python demo_real_robot.py -o data/demo_pusht_real --robot_ip <robot_ip>
```

This should result in a demonstration dataset in `data/demo_pusht_real` with in the same structure as [real Push-T training dataset](https://diffusion-policy.cs.columbia.edu/data/training/pusht_real.zip).

To train a Diffusion Policy, launch training with config:
```
$ python train.py --config-name=train_diffusion_unet_real_image_workspace task.dataset_path=data/demo_pusht_real
```
Edit [`diffusion-policy/config/task/real_pusht_image.yaml`](./diffusion_policy/config/task/real_pusht_image.yaml) if your camera setup is different.

After training, evaluate with:
```
python eval_real_robot.py -i data/outputs/blah/checkpoints/latest.ckpt -o data/eval_pusht_real --robot_ip <robot_ip>
```
Press 'c' to start evaluation (handing control over to the policy). Press 's' to stop the current episode.

## 🏷️ License
This repository is released under the MIT license. See [LICENSE](LICENSE) for additional details.
