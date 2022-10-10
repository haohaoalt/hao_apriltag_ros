# AprilTag_ros using

## 1.AprilTag_ros包的安装

1.先安装依赖库apriltag，安装方法如下。这个注意不要把它放到ROS的工作空间里

```shell
//安装依赖库apriltag
git clone https://github.com/AprilRobotics/apriltag.git  
mkdir build
cd build
cmake ..
make
sudp make install
```

2.安装AprilTag_ros包，将源码拷贝到ROS工作空间中的src工作目录下

```
git clone https://github.com/AprilRobotics/apriltag_ros.git  #在src文件夹内
```

3.回到工作空间的文件夹下编译

```
catkin_make
```

## 2.AprilTag_ros包的使用

在使用该功能包前，要先对单目摄像头进行标定。

### 2.1 ros中单目摄像头的标定

第一步：安装标定功能包

```
sudo apt-get install ros-melodic-camera-calibration
```

注：这里要确定安装了usb摄像头的驱动包，如果没安装安下面步骤安装

```
 git clone https://github.com/bosch-ros-pkg/usb_cam.git  #拷贝到ROS工作空间中的src工作目录下，然后编译
```

具体可以参考：
https://blog.csdn.net/qq_42585108/article/details/105562061

第二步：打开摄像头启动标定节点

```bash
roslaunch usb_cam usb_cam-test.launch  #启动摄像头
rosrun camera_calibration cameracalibrator.py --size 8x5 --square 0.028 image:=/usb_cam/image_raw camera:=/usb_cam
```

参数说明：
size：棋盘内角点的个数，行*列
square：一个格子的边长，单位是m
image：摄像头发布的图像话题明
camera：相机名
第三步开始标定

上下左右移动摄像头，当标定完成时，CALIBRATE会有颜色，点击它，然后点击save，再点击COMMIT。这是会自动保存标定文件。

标定文件保存路径

```bash
 writing calibration data to /home/mue/.ros/camera_info/head_camera.yaml   #ros默认自动找这个路径的文件
```

输出标定参数

```bash
[image]
width
640
height
480
[narrow_stereo]
camera matrix
496.458369 0.000000 321.704479
0.000000 498.045492 245.523491
0.000000 0.000000 1.000000
distortion
0.156540 -0.184929 0.004541 0.004684 0.000000
rectification
1.000000 0.000000 0.000000
0.000000 1.000000 0.000000
0.000000 0.000000 1.000000
projection
513.121582 0.000000 324.144520 0.000000
0.000000 515.390259 246.954678 0.000000
0.000000 0.000000 1.000000 0.000000
```

## 2.2 运行AprilTag_ros包

在运行节点需要配置两个配置文件
config/settings.yaml

```bash
tag_family:        'tag36h11' # options: tagStandard52h13, tagStandard41h12, tag36h11, tag25h9, tag16h5, tagCustom48h12, tagCircle21h7, tagCircle49h12  #支持单一标签类型
tag_threads:       2          # default: 2           # 设置Tag_Threads允许核心APRILTAG 2算法的某些部分运行并行计算。 典型的多线程优点和限制适用
tag_decimate:      1.0        # default: 1.0       #减小图像分辨率
tag_blur:          0.0        # default: 0.0            #设置tag_blur> 0模糊图像，tag_blur  < 0锐化图像
tag_refine_edges:  1          # default: 1       #增强了计算精度，但消耗了算力
tag_debug:         0          # default: 0            #1为保存中间图像到~/.ros
max_hamming_dist:  2          # default: 2 (Tunable parameter with 2 being a good choice - values >=3 consume large amounts of memory. Choose the largest value possible.)
# Other parameters
publish_tf:        true       # default: false       #发布tf坐标
```

这里有两点注意：
1.tag_family:时设置检测的标签类型，一般用’tag36h11’ 。openmv的IDE可以生成这个标签。
2.如果想发布TF坐标，需要把publish_tf设置为true。
tags.yaml

```bash
# 这个标签中至少要填入三个参数
standalone_tags:
  [
    {id: 1, size: 0.05},   #size对应标签的大小
    {id: 2, size: 0.05},
    #{id: 2, size: 0.05},
    #{id: 3, size: 0.05},
    {id: 3, size: 0.05}
  ]
  #这个标签是根据大小和位置检测的
 tag_bundles:
  [
    {
      name: 'my_bundle',
      layout:
        [
          {id: 10, size: 0.05, x: 0.0000, y: 0.0000, z: 0.0, qw: 1.0, qx: 0.0, qy: 0.0, qz: 0.0}
        ]
     } 
  ]
```

说明：如果只用于相机的定位，只需要填standalone_tags里的参数就可以了。其id号对应其生车标签的id，即每个数字生成的标签是唯一的。
这里要修改launch文件continuous_detection.launch。要与相机的话题保持一致。

```bash
 <arg name="camera_name" default="/usb_cam" />
<arg name="camera_frame" default="usb_cam" />  
 <arg name="image_topic" default="image_raw" />
```

启动节点

```bash
roslaunch apriltag_ros continuous_detection.launch
```