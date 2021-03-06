# indemind-sdk-Linux64
Binocular Vision Inertial Module

#### 简介  

INDEMIND双目视觉惯性模组采用全局快门的2X1280X800@50FPS高清摄像头，可提供水平120°、垂向75°视场角，结合高帧率6轴IMU传感器，为SLAM等算法提供强有力前端数据采集能力。
本SDK提供了双目视觉惯性模组的开发接口及依赖环境。  
已在以下环境下测试过：  
Linux Ubuntu 16.04 gcc5.4 和 Ubuntu 18.04 gcc7.3

#### 硬件要求  
双目视觉惯性模组要求支持USB3.0接口，仅支持通过SDK获取数据。深度解算以插件形式存在，该插件依赖CUDA9.0，建议使用Geforce GTX 1050以上的显卡

#### 安装依赖库
安装cmake
~~~
    sudo apt-get install cmake
    
安装google-glog + gflags
~~~
    sudo apt-get install libgoogle-glog-dev
    
安装BLAS & LAPACK
~~~
    sudo apt-get install libatlas-base-dev

下载SDK
~~~
    https://github.com/INDEMIND/SDK-Linux

#### 使用方式  
创建SDK对象  
~~~
    CIMRSDK* pSDK = new CIMRSDK();  
~~~
设置使用的SLAM  
~~~
    MRCONFIG config = { 0 };
    strcpy(config.slamPath, "slam.dll");
    config.bSlam = true;
~~~
获取模组图像数据
~~~
    pSDK->RegistModuleCameraCallback(SdkCameraCallBack,NULL);
~~~
获取IMU数据
~~~
    pSDK->RegistModuleIMUCallback(sdkImuCallBack,NULL);
~~~
获取SLAM结果
~~~
    pSDK->RegistModulePoseCallback(sdkSLAMResult,NULL);
~~~
获取深度图
~~~
    pSDK->AddPluginCallback("depthimage", "depth", DepthImageCallback, NULL);
~~~
回调函数中的pData是如下的一个数据结构:
~~~
struct ImrDepthImageTarget
{
    double _time;
    float _cubesize;
    int _image_w;
    int _image_h;
    float* _deepptr;    //深度图数据指针,长度为_image_w*_image_h,每个值对应像素位置的深度
};
~~~
释放资源
~~~
    pSDK->Release();
    delete pSDK;
~~~
注意：不要在以上回调函数里做延时比较多的操作，比如把数据写入文件等，否则会造成数据丢失，影响数据正确性等后果。
详细信息参考《双目惯性模组用户使用说明》。
#### 自定义Slam  
用户可以自定义slam，并以动态库形式在运行时加载到SDK中。如果希望替换SDK内置的slam算法，请按照下述方式操作：  
1. 创建名为slam的项目，确保其输出名字为slam.dll（linux环境下为libslam.so）
2. 将plugin/SlamPlugin.h添加到项目中，并继承ISlamPlugin，实现如下虚函数：
~~~
/* 在slam初始化的时候被调用,此时标定参数会通过参数传入 */
virtual bool Init(CameraCalibrationParam pParams);

/* SDK退出前调用Release，完成slam资源的释放 */
virtual void Release();

/* 初始化之后,会以1kHz的频率调用该接口，将IMU数据传入给自定义slam */
virtual void AddIMUAsync(double time, float accX, float accY, float accZ, float gyrX, float gyrY, float gyrZ);

/* 初始化之后,会以50Hz的频率调用该接口，将图像数据传入给自定义slam */
virtual void AddIMGAsync(double time, unsigned char* pLeft,unsigned char* pRight,int width,int height,int channel);

/* 初始化之后,会以1kHz的频率调用该接口，获取解算后的位姿数据 */
virtual SlamStatus GetPoseAsync(double* time, float* p, float* q);

/* SDK以默认指令调用该函数，以执行指定的slam操作，保留 */
virtual bool InvokeCommand(const char* commandName, void* pIn, void* pOut);
~~~
3. 实现`indem::ISlamPlugin* SlamFactory()`并返回Slam的实现。SDK会在运行时调用该接口，获得自定Slam对象。
4. 之后将生成的slam.dll/libslam.so拷贝替换掉自带的动态库即可。
#### 添加算法插件  
用户也可以添加自己的算法扩展到SDK中。demo\plugin中提供了一份示例，展示了如何添加自己的算法插件。所有的插件统一放置在plugin目录下：plugin文件夹下存放子文件夹,每个子文件夹存放着插件动态库及其依赖项,SDK会按照文件夹名字动态加载里头同名的dll/so,作为入口。  
#### 更新  
2018.11.16更新
1. `ImrModulePose`结构添加欧拉角
2. 宏定义`MRSDK_VERSION`提升到2  

2018.1.16更新  
1. 升级驱动,支持25/50Hz频率的图像
2. 修复SDK数据捕获时崩溃的问题
3. SDK移除对boost1.68版本的依赖
4. 深度图能够获取ROS需要的P值了
5. 修复了slam关闭的情况下不能获取模组参数信息的问题
6. 在不使用slam的情况下不再加载slam模块
7. 增加更多的错误信息提示
8. 宏定义`MRSDK_VERSION`提升到3  
#### FAQ  
常见问题请参考[FAQ](https://github.com/INDEMIND/SDK-Win64/wiki)
