# lidar_camera_record

This is a ros repo for simutaniously recording **Livox HAP** lidar, **Livox AVIA** lidar and **Hikrobot camera** iamge messagess.

## 1. Reference

```./src/livox_camera_calib``` from [hku-mars/livox_camera_calib](https://github.com/hku-mars/livox_camera_calib)
```./src/livox_ros_driver2``` from [Livox-SDK/livox_ros_driver2](https://github.com/Livox-SDK/livox_ros_driver2)
Other code in ```./src``` from [sheng00125/LIV_handhold](https://github.com/sheng00125/LIV_handhold)

## 2. Dependencies

### 2.1. ROS

Follow **1.Preparation** part in [livox_ros_driver2/README.md](./src/livox_ros_driver2/README.md)

### 2.2. Eigen

Follow [Eigen Installation](http://ceres-solver.org/installation.html)

### 2.3. Ceres Solver

**Attention!** Latest version of Ceres would incure errors when biulding, please install **Ceres 2.0.0** instead following [Ceres Installation](http://ceres-solver.org/installation.html)

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

# remove confliction files
rm -f ./lib/32/libusb-1.0.so.0
rm -f ./lib/64/libusb-1.0.so.0

# test the software again

bash ./bin/MVS.sh
```

## 3. Build

```shell
cd ~/lidar_camera_record_ws

# make sure here is only a src folder
ls
# src

source /opt/ros/noetic/setup.sh

# livox_ros_driver2 cannot be biuld with catkin_make
bash ./src/livox_ros_driver2/biuld.sh ROS1

# biuld the remaining modules
catkin_make

```

## 4. Config

### 4.1. AVIA Lidar

- Edit ```lidar_config``` in ```./src/livox_ros_driver/config/livox_lidar_config.json``` to

```json
"lidar_config": [
        {
            // Change it to your AVIA PIN, and add a "1" at the rear
            // Example: PIN: 123456, "broadcast_code": "1234561"
            "broadcast_code": "3JEDM2U00169251",
            // Enable it
            "enable_connect": true,
            "return_mode": 0,
            "coordinate": 0,
            // Turn on IMU
            "imu_rate": kImuFreq200Hz,
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
//example
"HAP": {
    "host_net_info" : {
      "cmd_data_ip" : "192.168.8.1",
      "point_data_ip": "192.168.8.1",
      "imu_data_ip" : "192.168.8.1",
    }
  },

"lidar_configs" : [
    {
      "ip" : "192.168.8.2", // It should be set as the IP of HAP lidar
    }
  ]

```

- Link HAP, then config the network manually

```
Example host IPv4 config for HAP:
Address:        192.168.8.1 // make sure it is defferent from what you set for AVIA
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