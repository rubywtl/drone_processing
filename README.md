## Prerequisites
### WSL and Docker
Use WSL2 for a local Linux environment because it's integrated well in VSCode. Any Ubuntu is good. For **WSL (not Linux OS)** specifically, install Docker application on the Windows side for easier dev:
1. Install Docker Desktop: [Download Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. Enable WSL integration in Docker Desktop settings.
3. Test Docker in your WSL terminal:

   ```bash
   docker --version
   ```

WSL distros are default installed to your OS drive, and moving it to another SSD for example could be a little tricky...

### Download docker-compose
On Linux (or WSL)
```bash
# install jq if not already
sudo apt install jq
sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | jq -r .tag_name)/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
# verify installation
docker-compose --version
# python dependencies
pip install --upgrade setuptools
```

## Simulation (local testing)
### step 1: prepare docker image
```bash
cd ./docker/ros2_dev
sudo docker build -t ros2_dev_image .
```
### step 2: run docker container
```bash
sudo docker run -it --rm \
  -v ~/<path to repo>:/workspace \
  -v /etc/localtime:/etc/localtime:ro \
  -v /etc/timezone:/etc/timezone:ro \
  ros2_dev_image bash
ls /workspace # after entering the container make sure ur files are there
```
### step 3: build project
```bash
rm -rf build log install # ensures clean build
colcon build --symlink-install
source /opt/ros/humble/setup.bash 
```
### step 4: launch nodes
launching nodes individually
```bash
# for simulation
ros2 run control_node control_node __params:=mode:=simulation
ros2 run camera_node camera_node __params:=mode:=simulation
ros2 run object_detection_node object_detection_node __params:=mode:=simulation
```
launching the system together
(ensure to set the mode in the launch file before running)
```bash
ros2 launch ./launch/full_system.launch.py
```


## Raspberry Pi Deployment with Docker
### step 1: connect to raspberry pi via ssh
```bash
ssh your_ssh_address
```
### step 2: prepare docker image
this only needs to be done once or when the dockerfile is updated
```bash
cd ./docker/ros2_deploy
docker build -t ros2_deploy_image .
```
### step 3: launch pigpio daemon (outside the docker)
```bash
sudo apt update
sudo apt install pigpio
sudo pigpiod
ps aux | grep pigpiod # check if it is running

# IF you want to STOP the daemon
sudo killall pigpiod

# IF you want to find the address of pigpio address
hostname -I
```
### step 4: run docker container
```bash
sudo docker run -it --rm \
  --device /dev/gpiomem \
  --network host \
  -v /home/perryz0/Documents/drone_processing:/workspace \
  -v /etc/localtime:/etc/localtime:ro \
  -v /etc/timezone:/etc/timezone:ro \
  ros2_deploy_image bash
```
### step 5: build project
```bash
rm -rf build log install # ensures clean build
colcon build --symlink-install
source /opt/ros/humble/setup.bash 
source install/setup.bash
```
### step 6: launch nodes
launching nodes individually
```bash
# for simulation
ros2 run grabber_node grabber_node __params:=mode:=deployment
ros2 run camera_node camera_node __params:=mode:=deployment
ros2 run object_detection_node object_detection_node __params:=mode:=deployment
```
launching the system together
(ensure to set the mode in the launch file before running)
```bash
ros2 launch ./launch/full_system.launch.py
```

## Raspberry Pi Native Deployment 
