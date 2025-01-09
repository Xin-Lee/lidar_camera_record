# lidar_camera_record

This is a ros repo for simultaneously recording **Livox HAP** lidar, **Livox AVIA** lidar and **Hikrobot camera** image messages.

## 1. Reference

```./src/livox_camera_calib``` from [hku-mars/livox_camera_calib](https://github.com/hku-mars/livox_camera_calib)
```./src/livox_ros_driver2``` from [Livox-SDK/livox_ros_driver2](https://github.com/Livox-SDK/livox_ros_driver2)
Other code in ```./src``` from [sheng00125/LIV_handhold](https://github.com/sheng00125/LIV_handhold)

## 2. Dependencies

### 2.1. ROS

Follow **1.Preparation** part in [livox_ros_driver2/README.md](./src/livox_ros_driver2/README.md)

### 2.2. Eigen

```shell
sudo apt install libeigen3-dev
```

### 2.3. Ceres Solver

**Attention!** Latest version of Ceres would incur errors when building, please install **Ceres 2.0.0** instead following [Ceres Installation](http://ceres-solver.org/installation.html)

### 2.4. PCL

```shell
sudo apt install libpcl-dev
```

### 2.5. Hikrobot MVS for Linux

- Download in [Hikrobot Download Support](https://www.hikrobotics.com/cn/machinevision/service/download/?module=0)
- Following the installation
- Remove two confliction files:

```shell
# MVS folder
cd /opt/MVS

# test the software
bash ./bin/MVS.sh

# remove conflict files
rm -f ./lib/32/libusb-1.0.so.0
rm -f ./lib/64/libusb-1.0.so.0

# test the software again

bash ./bin/MVS.sh
```

### 2.6 Livos-SDK

```shell
cd ~
git clone https://github.com/Livox-SDK/Livox-SDK.git
cd Livox-SDK
cd build && cmake ..
make
sudo make install
```
### 2.7 Livox-SDK2

```shell
git clone https://github.com/Livox-SDK/Livox-SDK2.git
cd ./Livox-SDK2/
mkdir build
cd build
cmake .. && make -j
sudo make install
```

## 3. Build

```shell
cd ~/lidar_camera_record_ws

# make sure here is only a src folder
ls
# src

source /opt/ros/noetic/setup.sh

# livox_ros_driver2 cannot be built with catkin_make
bash ./src/livox_ros_driver2/build.sh ROS1

# biuld the remaining modules
catkin_make

```

## 4. Config

### 4.1. AVIA Lidar

- Edit ```lidar_config``` in ```./src/livox_ros_driver/config/livox_lidar_config.json``` to

Example:
Change the broadcast_code to your AVIA PIN, and add a "1" at the rear
Example: PIN: 123456, "broadcast_code": "1234561"

```json
"lidar_config": [
        {
            
            "broadcast_code": "3JEDM2U00169251",
            "enable_connect": true,
            "return_mode": 0,
            "coordinate": 0,
            "imu_rate": 1,
            "extrinsic_parameter_source": 0,
            "enable_high_sensitivity": false
        },
]
```

- Linking the camera, then config the network manually

```
Example host IPv4 config for AVIA:
Address:        192.168.7.1
Subnet mask:    255.255.255.0
```

### 4.2. Hikrobot Camera

Edit parameters in ```./src/mvs_ros_pkg/config/left_camera_trigger.yaml``` to what you want.

### 4.3. HAP Lidar

- Edit ip config in ```src/livox_ros_driver2/config/HAP_config.json```

```json
"HAP": {
    "host_net_info" : {
      "cmd_data_ip" : "192.168.8.1",
      "point_data_ip": "192.168.8.1",
      "imu_data_ip" : "192.168.8.1",
    }
  },

"lidar_configs" : [
    {
      "ip" : "192.168.8.2",
    }
  ]

```

- Link HAP, then config the network manually

```
Example host IPv4 config for HAP:
Address:        192.168.8.1 // make sure it is different from what you set for AVIA
Subnet mask:    255.255.255.0
```

## 5. Launch

```shell
source ./devel/setup.bash

roslaunch livox_ros_driver livox_lidar_msg.launch multi_topic:=1
# topic: /livox/lidar_PIN ; /livox/imu_PIN

roslaunch livox_ros_driver2  msg_HAP.launch
# topic: /livox/lidar ; /livox/imu

roslaunch mvs_ros_pkg mvs_camera_trigger.launch
# topic: /left_camera/image
```
