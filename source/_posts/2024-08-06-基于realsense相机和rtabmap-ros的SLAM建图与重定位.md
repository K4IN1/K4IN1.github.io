---
title: 基于realsense相机和rtabmap_ros的SLAM建图与重定位
date: 2024-08-06 22:46:57
tags: 

- SLAM
- rtabmap
- realsense
- ROS
categories:
- 教程

---

# 基于realsense相机和rtabmap_ros的SLAM建图与重定位

地图和定位对于可以动作的设备来说十分重要，对一些移动平台来讲也是如此，现研究采用视觉传感器进行SLAM建图  

## 硬件

D435i为realsense相机中十分经典的一款，双目+RGB+IMU的传感器配置，给他提供了丰富的传感器信息。  
<!-- more -->
T265相机是配备了双目鱼眼+IMU的相机，内置有Intel自己的微处理器，作为视觉里程计。目前已经停止了许多软件支持。  

## 软件

在ROS中使用，考虑了ORB_SLAM和RTABMAP  

### ORB_SLAM

由于ORB_SLAM的功能过于匮乏，保存载入的功能需要自己MOD。在Github上找到了高翔他们的点云MOD版本，但是生成的点云方向需要调整，而且对地面的过滤算法需要自己来写，也有尝试转成OctoMap地图，然后投影到地面的方案，但是由于点云的地面不平、倾斜较大，放弃了这个方案。  
虽然定位较为稳定，但是因为地图导出困难，保存和加载功能需要自己MOD  

### RTAB_MAP

RTAB_MAP可以接收视觉，2D激光和3D激光多种信息，进行建图保存、加载、定位，其功能较为完善，使用的tutorial也相对较多，缺点是性能开销较大，但是可以通过关闭3D显示相关的功能来缓解，或者直接使用RVIZ来可视化，另外其参数调整的空间也很大，比较适合建图重定位。  

# 安装和调试

本次软件使用采用了UBuntu18.04的版本，因此也对硬件的型号有一定要求，过于新的网卡显卡甚至是触摸板都是需要自己额外打驱动上去的，建议使用

## RealsenseSDK

由于Intel官方在新的版本中放弃了对T265等老旧设备的支持，因此选择安装SDK为2.50版本。  
依照官方文档的[操作](https://github.com/IntelRealSense/librealsense/blob/master/doc/distribution_linux.md#installing-the-packages)  
下载对应的公钥

```bash
sudo mkdir -p /etc/apt/keyrings
curl -sSf https://librealsense.intel.com/Debian/librealsense.pgp | sudo tee /etc/apt/keyrings/librealsense.pgp > /dev/null
```

随后添加到列表

```bash
echo "deb [signed-by=/etc/apt/keyrings/librealsense.pgp] https://librealsense.intel.com/Debian/apt-repo `lsb_release -cs` main" | \
sudo tee /etc/apt/sources.list.d/librealsense.list
sudo apt-get update
```

然后就可以安装了  
> 注意:*安装全程请拔出相机。接下来的步骤需要查询并输入对应版本号，请直接从查询结果里复制，或者使用tab进行补全，本文的内容经过验证，但是文章输入难免有错误，请见谅。按照顺序进行，一旦有报错，请使用`sudo apt --fix-broken install`解决之后再进行下一步安装*
使用`apt list -a librealsense2-dkms` 或者 `apt-cache madison librealsense2-dkms` 来查询软件包的版本
依次安装udev-rules dkms 和其他的软件包

```bash
sudo apt-get install librealsense2-udev-rules:amd64=2.50.0-0~realsense0.6127
sudo apt-get install librealsense2-dkms=1.3.13-0ubuntu1
```

再安装其他的utils

```bash
sudo apt-get insall librealsense2=2.50.0-0~realsense0.6127
sudo apt-get insall librealsense2-gl=2.50.0-0~realsense0.6127
sudo apt-get insall librealsense2-net=2.50.0-0~realsense0.6127
sudo apt-get insall librealsense2-utils=2.50.0-0~realsense0.6127
sudo apt-get insall librealsense2-dev=2.50.0-0~realsense0.6127
sudo apt-get insall librealsense2-dbg=2.50.0-0~realsense0.6127
```

安装完成后直接在控制台输入`realsense-viewer`打开软件，接入摄像头即可查看不同传感器的数据，还可以在3D页面预览深度效果  

## RTAB-MAP

### 安装

直接参照RTABMAP_ROS [wiki 页面](https://github.com/introlab/rtabmap_ros#rtabmap_ros)  

### 建图

查看rtabmap_ros的launch文件可以发现，每次启动的时候都需要去指定订阅的话题，以及一部分ros相关的参数，例如是否启动rviz，深度是否和RGB对齐等相关参数，具体的内容如下  

#### **如果只使用D435/D415**  

首先是启动realsenseros节点,其中的参数就是指把RGB和深度进行对齐，这样会多输出一个对齐后的topic

```bash
roslaunch realsense2_camera rs_camera.launch align_depth:=true
```

这个launch文件不会启用相机内的IMU功能，把相机当做纯RGBD相机使用，因此接下来使用RTABMAP时，需要启用视觉里程计算法。

```bash
roslaunch rtabmap_launch rtabmap.launch \
    rtabmap_args:="--delete_db_on_start" \
    depth_topic:=/camera/aligned_depth_to_color/image_raw \
    rgb_topic:=/camera/color/image_raw \
    camera_info_topic:=/camera/color/camera_info \
    approx_sync:=false
```

建议测试完成后直接修改对应的launch文件，直接包含两个launch文件并修改参数，直接一键启动  
参数的功能直接去看rtabmap_ros的.launch和他们的wiki，注释写的很详细

#### 启动D435i/L515的IMU

```bash
roslaunch realsense2_camera rs_camera.launch \
    align_depth:=true \
    unite_imu_method:="linear_interpolation" \
    enable_gyro:=true \
    enable_accel:=true
```

为了把IMU数据转换成rtabmap可以订阅的格式，我们使用另外一个节点

```bash
rosrun imu_filter_madgwick imu_filter_node \
    _use_mag:=false \
    _publish_tf:=false \
    _world_frame:="enu" \
    /imu/data_raw:=/camera/imu \
    /imu/data:=/rtabmap/imu
```

随后启动rtabmap建图

```bash
 rtabmap_args:="--delete_db_on_start --Optimizer/GravitySigma 0.3" \
    depth_topic:=/camera/aligned_depth_to_color/image_raw \
    rgb_topic:=/camera/color/image_raw \
    camera_info_topic:=/camera/color/camera_info \
    approx_sync:=false \
    wait_imu_to_init:=true \
    imu_topic:=/rtabmap/imu
```

#### 使用D400/L515+T265外置里程计

realsense内置了对应的launch文件，
> **请注意**：*在使用launch文件之前，请使用rs_camera.launch单独打开每一个设备，在控制台的log中查看相机的SN，然后将对应的SN填入launch文件的arg中*
**请确保使用了官方的3D打印支架，如果二者之间的几何关系发生改变，请编辑launch文件中的TFNode启动参数，从而确保相机坐标转换的正确**

```bash
roslaunch realsense2_camera rs_d400_and_t265.launch
```

由于T265会自己计算里程计，因此不再需要启用视觉里程计算法

```bash
roslaunch rtabmap_launch rtabmap.launch \
   args:="-d --Mem/UseOdomGravity true --Optimizer/GravitySigma 0.3" \
   odom_topic:=/t265/odom/sample \
   frame_id:=t265_link \
   rgbd_sync:=true \
   depth_topic:=/d400/aligned_depth_to_color/image_raw \
   rgb_topic:=/d400/color/image_raw \
   camera_info_topic:=/d400/color/camera_info \
   approx_rgbd_sync:=false \
   visual_odometry:=false \
   queue_size:=30
```

你可以编写自己的launch文件，把上述需要更改的参数都丢进去，甚至可以参照rtabmap官方的ini文件，更改mapping的各种参数，然后启动的时候自动加载  
例如下方的launch文件，参考rtabmap官方的launch文件，知道载入一个.ini文件既可以设定好mapping的参数，所以参照了rtabmap修改参数apply时控制台的输出，添加对应参数到.ini的[Core]下即可

```xml
<launch>
    <arg name="device_type_camera1"    default="t265"/>
    <arg name="device_type_camera2"    default="d435"/>
    <arg name="serial_no_camera1"      default=""/>
    <arg name="serial_no_camera2"      default=""/>
    <arg name="camera1"                default="t265"/>  <!-- Note: Replace with camera name -->
    <arg name="camera2"                default="d400"/>  <!-- Note: Replace with camera name -->
    <arg name="clip_distance"          default="-2"/>
    <arg name="use_rviz"               default="true"/>
    <arg name="use_rtabmapviz"         default="true"/>
    

    <include file="$(find realsense2_camera)/launch/rs_d400_and_t265.launch">
            <arg name="device_type_camera1"             value="$(arg device_type_camera1)"/>
            <arg name="device_type_camera2"             value="$(arg device_type_camera2)"/>
            <arg name="serial_no_camera1"               value="$(arg serial_no_camera1)"/>
            <arg name="serial_no_camera2"               value="$(arg serial_no_camera2)"/>
            <arg name="camera1"                         value="$(arg camera1)"/>
            <arg name="camera2"                         value="$(arg camera2)"/>
            <arg name="clip_distance"                   value="$(arg clip_distance)"/>
            
    </include>

    <include file="$(find rtabmap_ros)/launch/rtabmap.launch">
            <arg name="rtabmap_args"       value="--delete_db_on_start"/>
            <arg name="depth_topic"        value="/$(arg camera2)/aligned_depth_to_color/image_raw"/>
            <arg name="frame_id"           value="$(arg camera2)_link"/>
            <arg name="cfg"                value="$(find realsense2_camera)/launch/cfg/slam.ini"/>
            <arg name="visual_odometry"    value="false"/>
            <arg name="odom_topic"         value="/$(arg camera1)/odom/sample"/>
            <arg name="rgb_topic"          value="/$(arg camera2)/color/image_raw"/>
            <arg name="camera_info_topic"  value="/$(arg camera2)/color/camera_info"/>
            <arg name="queue_size"         value="200"/>
            <arg name="rviz"               value="$(arg use_rviz)"/>
            <arg name="rtabmapviz"         value="$(arg use_rtabmapviz)"/>
    </include>
</launch>
```

随后是.ini文件

```ini
[Core]
Version = 0.20.23
BRIEF\Bytes = 32
BRISK\Octaves = 3
BRISK\PatternScale = 1
BRISK\Thresh = 30
Bayes\FullPredictionUpdate = false
Bayes\PredictionLC = 0.1 0.36 0.30 0.16 0.062 0.0151 0.00255 0.000324 2.5e-05 1.3e-06 4.8e-08 1.2e-09 1.9e-11 2.2e-13 1.7e-15 8.5e-18 2.9e-20 6.9e-23
Bayes\VirtualPlacePriorThr = 0.9
Db\TargetVersion = 
DbSqlite3\CacheSize = 10000
DbSqlite3\InMemory = false
DbSqlite3\JournalMode = 3
DbSqlite3\Synchronous = 0
DbSqlite3\TempStore = 2
FAST\CV = 0
FAST\Gpu = false
FAST\GpuKeypointsRatio = 0.05
FAST\GridCols = 0
FAST\GridRows = 0
FAST\MaxThreshold = 200
FAST\MinThreshold = 7
FAST\NonmaxSuppression = true
FAST\Threshold = 20
FREAK\NOctaves = 4
FREAK\OrientationNormalized = true
FREAK\PatternScale = 22
FREAK\ScaleNormalized = true
GFTT\BlockSize = 3
GFTT\K = 0.04
GFTT\MinDistance = 7
GFTT\QualityLevel = 0.001
GFTT\UseHarrisDetector = false
GMS\ThresholdFactor = 6.0
GMS\WithRotation = false
GMS\WithScale = false
GTSAM\Optimizer = 1
Grid\3D = true
Grid\CellSize = 0.05
Grid\ClusterRadius = 0.1
Grid\DepthDecimation = 4
Grid\DepthRoiRatios = 0.0 0.0 0.0 0.0
Grid\FlatObstacleDetected = true
Grid\FootprintHeight = 0.0
Grid\FootprintLength = 0.0
Grid\FootprintWidth = 0.0
Grid\GroundIsObstacle = false
Grid\MapFrameProjection = false
Grid\MaxGroundAngle = 45
Grid\MaxGroundHeight = 0.0
Grid\MaxObstacleHeight = 0.0
Grid\MinClusterSize = 10
Grid\MinGroundHeight = 0.0
Grid\NoiseFilteringMinNeighbors = 5
Grid\NoiseFilteringRadius = 0.0
Grid\NormalK = 20
Grid\NormalsSegmentation = true
Grid\PreVoxelFiltering = true
Grid\RangeMax = 5.0
Grid\RangeMin = 0.0
Grid\RayTracing = false
Grid\Scan2dUnknownSpaceFilled = false
Grid\ScanDecimation = 1
Grid\Sensor = 1
GridGlobal\AltitudeDelta = 0
GridGlobal\Eroded = false
GridGlobal\FloodFillDepth = 0
GridGlobal\FootprintRadius = 0.0
GridGlobal\FullUpdate = true
GridGlobal\MaxNodes = 0
GridGlobal\MinSize = 0.0
GridGlobal\OccupancyThr = 0.5
GridGlobal\ProbClampingMax = 0.971
GridGlobal\ProbClampingMin = 0.1192
GridGlobal\ProbHit = 0.7
GridGlobal\ProbMiss = 0.4
GridGlobal\UpdateError = 0.01
Icp\CCFilterOutFarthestPoints = false
Icp\CCMaxFinalRMS = 0.2
Icp\CCSamplingLimit = 50000
Icp\CorrespondenceRatio = 0.1
Icp\DebugExportFormat = 
Icp\DownsamplingStep = 1
Icp\Epsilon = 0
Icp\Force4DoF = false
Icp\Iterations = 30
Icp\MaxCorrespondenceDistance = 0.1
Icp\MaxRotation = 0.78
Icp\MaxTranslation = 0.2
Icp\OutlierRatio = 0.85
Icp\PMConfig = 
Icp\PMMatcherEpsilon = 0.0
Icp\PMMatcherIntensity = false
Icp\PMMatcherKnn = 1
Icp\PointToPlane = true
Icp\PointToPlaneGroundNormalsUp = 0.0
Icp\PointToPlaneK = 5
Icp\PointToPlaneLowComplexityStrategy = 1
Icp\PointToPlaneMinComplexity = 0.02
Icp\PointToPlaneRadius = 0.0
Icp\RangeMax = 0
Icp\RangeMin = 0
Icp\Strategy = 1
Icp\VoxelSize = 0.05
ImuFilter\ComplementaryBiasAlpha = 0.01
ImuFilter\ComplementaryDoAdpativeGain = true
ImuFilter\ComplementaryDoBiasEstimation = true
ImuFilter\ComplementaryGainAcc = 0.01
ImuFilter\MadgwickGain = 0.1
ImuFilter\MadgwickZeta = 0.0
KAZE\Diffusivity = 1
KAZE\Extended = false
KAZE\NOctaveLayers = 4
KAZE\NOctaves = 4
KAZE\Threshold = 0.001
KAZE\Upright = false
Kp\BadSignRatio = 0.5
Kp\ByteToFloat = false
Kp\DetectorStrategy = 8
Kp\DictionaryPath = 
Kp\FlannRebalancingFactor = 2.0
Kp\GridCols = 1
Kp\GridRows = 1
Kp\IncrementalDictionary = true
Kp\IncrementalFlann = true
Kp\MaxDepth = 0
Kp\MaxFeatures = 500
Kp\MinDepth = 0
Kp\NNStrategy = 1
Kp\NewWordsComparedTogether = true
Kp\NndrRatio = 0.8
Kp\Parallelized = true
Kp\RoiRatios = 0.0 0.0 0.0 0.0
Kp\SubPixEps = 0.02
Kp\SubPixIterations = 0
Kp\SubPixWinSize = 3
Kp\TfIdfLikelihoodUsed = true
Marker\CornerRefinementMethod = 0
Marker\Dictionary = 0
Marker\Length = 0
Marker\MaxDepthError = 0.01
Marker\MaxRange = 0.0
Marker\MinRange = 0.0
Marker\Priors = 
Marker\PriorsVarianceAngular = 0.001
Marker\PriorsVarianceLinear = 0.001
Marker\VarianceAngular = 0.01
Marker\VarianceLinear = 0.001
Mem\BadSignaturesIgnored = false
Mem\BinDataKept = true
Mem\CompressionParallelized = true
Mem\CovOffDiagIgnored = true
Mem\DepthAsMask = true
Mem\GenerateIds = true
Mem\ImageCompressionFormat = .jpg
Mem\ImageKept = false
Mem\ImagePostDecimation = 1
Mem\ImagePreDecimation = 1
Mem\IncrementalMemory = true
Mem\InitWMWithAllNodes = false
Mem\IntermediateNodeDataKept = false
Mem\LaserScanDownsampleStepSize = 1
Mem\LaserScanNormalK = 0
Mem\LaserScanNormalRadius = 0.0
Mem\LaserScanVoxelSize = 0.0
Mem\LocalizationDataSaved = false
Mem\MapLabelsAdded = true
Mem\NotLinkedNodesKept = true
Mem\RawDescriptorsKept = true
Mem\RecentWmRatio = 0.2
Mem\ReduceGraph = false
Mem\RehearsalIdUpdatedToNewOne = false
Mem\RehearsalSimilarity = 0.6
Mem\RehearsalWeightIgnoredWhileMoving = false
Mem\STMSize = 10
Mem\SaveDepth16Format = false
Mem\StereoFromMotion = false
Mem\TransferSortingByWeightId = false
Mem\UseOdomFeatures = true
Mem\UseOdomGravity = false
ORB\EdgeThreshold = 19
ORB\FirstLevel = 0
ORB\Gpu = false
ORB\NLevels = 3
ORB\PatchSize = 31
ORB\ScaleFactor = 2
ORB\ScoreType = 0
ORB\WTA_K = 2
Optimizer\Epsilon = 0.0
Optimizer\GravitySigma = 0.3
Optimizer\Iterations = 20
Optimizer\LandmarksIgnored = false
Optimizer\PriorsIgnored = true
Optimizer\Robust = false
Optimizer\Strategy = 1
Optimizer\VarianceIgnored = false
PyDetector\Cuda = true
PyDetector\Path = 
PyMatcher\Cuda = true
PyMatcher\Iterations = 20
PyMatcher\Model = indoor
PyMatcher\Path = 
PyMatcher\Threshold = 0.2
RGBD\AngularSpeedUpdate = 0.0
RGBD\AngularUpdate = 0.1
RGBD\CreateOccupancyGrid = true
RGBD\Enabled = true
RGBD\GoalReachedRadius = 0.5
RGBD\GoalsSavedInUserData = false
RGBD\InvertedReg = false
RGBD\LinearSpeedUpdate = 0.0
RGBD\LinearUpdate = 0.1
RGBD\LocalBundleOnLoopClosure = false
RGBD\LocalImmunizationRatio = 0.25
RGBD\LocalRadius = 10
RGBD\LoopClosureIdentityGuess = false
RGBD\LoopClosureReextractFeatures = false
RGBD\LoopCovLimited = false
RGBD\MarkerDetection = false
RGBD\MaxLocalRetrieved = 2
RGBD\MaxLoopClosureDistance = 0.0
RGBD\MaxOdomCacheSize = 10
RGBD\NeighborLinkRefining = false
RGBD\NewMapOdomChangeDistance = 0
RGBD\OptimizeMaxError = 0.0
RGBD\OptimizeFromGraphEnd = false
RGBD\PlanAngularVelocity = 0
RGBD\PlanLinearVelocity = 0
RGBD\PlanStuckIterations = 0
RGBD\ProximityAngle = 45
RGBD\ProximityBySpace = true
RGBD\ProximityByTime = false
RGBD\ProximityGlobalScanMap = false
RGBD\ProximityMaxGraphDepth = 50
RGBD\ProximityMaxPaths = 3
RGBD\ProximityMergedScanCovFactor = 100.0
RGBD\ProximityOdomGuess = false
RGBD\ProximityPathFilteringRadius = 1
RGBD\ProximityPathMaxNeighbors = 0
RGBD\ProximityPathRawPosesUsed = true
RGBD\ScanMatchingIdsSavedInLinks = true
RGBD\StartAtOrigin = false
Reg\Force3DoF = false
Reg\RepeatOnce = true
Reg\Strategy = 0
Rtabmap\ComputeRMSE = true
Rtabmap\CreateIntermediateNodes = false
Rtabmap\DetectionRate = 2
Rtabmap\ImageBufferSize = 1
Rtabmap\ImagesAlreadyRectified = true
Rtabmap\LoopGPS = true
Rtabmap\LoopRatio = 0
Rtabmap\LoopThr = 0.11
Rtabmap\MaxRepublished = 2
Rtabmap\MaxRetrieved = 2
Rtabmap\MemoryThr = 0
Rtabmap\PublishLastSignature = true
Rtabmap\PublishLikelihood = true
Rtabmap\PublishPdf = true
Rtabmap\PublishRAMUsage = false
Rtabmap\PublishStats = true
Rtabmap\RectifyOnlyFeatures = false
Rtabmap\SaveWMState = false
Rtabmap\StartNewMapOnGoodSignature = false
Rtabmap\StartNewMapOnLoopClosure = false
Rtabmap\StatisticLogged = false
Rtabmap\StatisticLoggedHeaders = true
Rtabmap\StatisticLogsBufferedInRAM = true
Rtabmap\TimeThr = 0
Rtabmap\WorkingDirectory = /home/asudct/.ros
SIFT\ContrastThreshold = 0.04
SIFT\EdgeThreshold = 10
SIFT\NFeatures = 0
SIFT\NOctaveLayers = 3
SIFT\RootSIFT = false
SIFT\Sigma = 1.6
SURF\Extended = false
SURF\GpuKeypointsRatio = 0.01
SURF\GpuVersion = false
SURF\HessianThreshold = 500
SURF\OctaveLayers = 2
SURF\Octaves = 4
SURF\Upright = false
Stereo\DenseStrategy = 0
Stereo\Eps = 0.01
Stereo\Iterations = 30
Stereo\MaxDisparity = 128.0
Stereo\MaxLevel = 5
Stereo\MinDisparity = 0.5
Stereo\OpticalFlow = true
Stereo\SSD = true
Stereo\WinHeight = 3
Stereo\WinWidth = 15
StereoBM\BlockSize = 15
StereoBM\Disp12MaxDiff = -1
StereoBM\MinDisparity = 0
StereoBM\NumDisparities = 128
StereoBM\PreFilterCap = 31
StereoBM\PreFilterSize = 9
StereoBM\SpeckleRange = 4
StereoBM\SpeckleWindowSize = 100
StereoBM\TextureThreshold = 10
StereoBM\UniquenessRatio = 15
StereoSGBM\BlockSize = 15
StereoSGBM\Disp12MaxDiff = 1
StereoSGBM\MinDisparity = 0
StereoSGBM\Mode = 2
StereoSGBM\NumDisparities = 128
StereoSGBM\P1 = 2
StereoSGBM\P2 = 5
StereoSGBM\PreFilterCap = 31
StereoSGBM\SpeckleRange = 4
StereoSGBM\SpeckleWindowSize = 100
StereoSGBM\UniquenessRatio = 20
SuperPoint\Cuda = true
SuperPoint\ModelPath = 
SuperPoint\NMS = true
SuperPoint\NMSRadius = 4
SuperPoint\Threshold = 0.010
VhEp\Enabled = false
VhEp\MatchCountMin = 8
VhEp\RansacParam1 = 3
VhEp\RansacParam2 = 0.99
Vis\BundleAdjustment = 1
Vis\CorFlowEps = 0.01
Vis\CorFlowIterations = 30
Vis\CorFlowMaxLevel = 3
Vis\CorFlowWinSize = 16
Vis\CorGuessMatchToProjection = false
Vis\CorGuessWinSize = 40
Vis\CorNNDR = 0.8
Vis\CorNNType = 1
Vis\CorType = 0
Vis\DepthAsMask = true
Vis\EpipolarGeometryVar = 0.1
Vis\EstimationType = 1
Vis\FeatureType = 8
Vis\ForwardEstOnly = true
Vis\GridCols = 1
Vis\GridRows = 1
Vis\InlierDistance = 0.1
Vis\Iterations = 300
Vis\MaxDepth = 0
Vis\MaxFeatures = 1000
Vis\MeanInliersDistance = 0.0
Vis\MinDepth = 0
Vis\MinInliers = 15
Vis\MinInliersDistribution = 0.0
Vis\PnPFlags = 0
Vis\PnPMaxVariance = 0.0
Vis\PnPRefineIterations = 0
Vis\PnPReprojError = 2
Vis\RefineIterations = 5
Vis\RoiRatios = 0.0 0.0 0.0 0.0
Vis\SubPixEps = 0.02
Vis\SubPixIterations = 0
Vis\SubPixWinSize = 3
g2o\Baseline = 0.075
g2o\Optimizer = 0
g2o\PixelVariance = 1.0
g2o\RobustKernelDelta = 8
g2o\Solver = 0
```

其实有很多参数根本就没有修改，但是因为保存的时候会自动保存，也就无所谓了  

### 保存

为了保存建好的地图到2D Occupancy，我们需要使用map_server的map_saver 节点
在上述建图运行时

```bash
rosrun map_server map_saver map:=/rtabmap/grid_map -f filename
```

随后CTRL+C退出建图即可，rtabmap会自动保存database到.ros/下，请进行备份
通过使用

``` bash
rtabmapdatabase-Viewer .ros/rtabmap.db
```

就可以预览刚才建立的.db了，既可以导出3D点云，Octomap地图，也可以对2D地图进行编辑，修改，删除地图中因噪声产生的错误点位。

### 重定位

只需要在之前的launch文件中添加

``` xml
<include file="$(find rtabmap_ros)/launch/rtabmap.launch">
...
        <arg name="localization"               value="true"/>
...
d</include>
```

或者运行launch文件的时候使用

``` bash
roslaunch rtabmap_launch rtabmap.launch localization:=true
```

即可开启定位
> **注意**：*在使用定位的时候，尽量保证定位开始时，摄像头处于地图的起始位置，并且在节点启动时，不要让里程计有变化的输出值，不然会导致定位的漂移，需要重新启动定位才能解决*  
定位信息在`/rtabmap/localization_pose`topic内，里程计相对于地图的offset则在`/tf`中可以找到，根据之前保存的Occupancy地图和对应的yaml文件，就可以确定位置了。  
