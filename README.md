# 使用QT+VS开发`CSR`相机应用（2）

上一篇文章我们讲了关于`CSR`相机的新建工程以及`SDK`的开发环境配置，若不知道怎么配置的可去康耐德智能控制公司的网站上寻找上一篇，要进行开发的步骤，新建以及配置环境都是不可缺少的部分，下面讲解下一部分和关于开发`CSR`相机的`SDK`的编程例子。

#### 定义`CSR`相机相关成员变量

```c++
	//相机控制相关变量
	bool							  DeviceFlag;				  //设备标志
	bool							  DeviceStreamFlag;			  //设备流标志
	CGXDevicePointer                  m_objDevicePtr;             ///< 设备句柄
	CGXStreamPointer                  m_objStreamPtr;             ///< 设备流
	CGXFeatureControlPointer          m_objFeatureControlPtr;     ///< 属性控制器
```

#### 初始化`GxIAPICPP`库

`GxIAPICPP`库在使用之前必须执行初始化。在调用其他接口之前必须调用`IGXFactory::GetInstance().Init()`接口执行初始化

```C++
//使用其他接口之前，必须执行初始化
IGXFactory::GetInstance().Init();
```

#### 寻找可用相机

初始化完成后，需要检测当前是否连接了`CSR`相机，寻找可用的`CSR`相机：

```C++
		GxIAPICPP::gxdeviceinfo_vector vectorDeviceInfo;
		IGXFactory::GetInstance().UpdateDeviceList(1000, vectorDeviceInfo);
		qDebug()<< "可用设备数：" << QString::number(vectorDeviceInfo.size())<<endl;
```

#### 操作相机

找到可用的相机，接着就是对当前相机进行操作，首先打开相机，之后便是开始准备拍照：

```C++
	m_objDevicePtr = \	IGXFactory::GetInstance().OpenDeviceBySN(vectorDeviceInfo[0].GetSN(),\ GX_ACCESS_EXCLUSIVE);
			DeviceFlag = true;
			m_objFeatureControlPtr = m_objDevicePtr->GetRemoteFeatureControl();
		int StreamCount = m_objDevicePtr->GetStreamCount();
		if (StreamCount > 0)
		{
			m_objStreamPtr = m_objDevicePtr->OpenStream(0);
			DeviceStreamFlag = true;
		}
```

#### 采单帧图像

上述步骤完成后，用户开启流对象采集并且给设备发送开采命令之后，就可以调用`GetImage`接口采单帧了，如下：

```C++
CGXStreamPointer objStreamPtr = m_objDevicePtr->OpenStream(0);
//开启流通道采集
objStreamPtr->StartGrab();
//给设备发送开采命令
CGXFeatureControlPointer objFeatureControlPtr = m_objDevicePtr->GetRemoteFeatureControl();
objFeatureControlPtr->GetCommandFeature("AcquisitionStart")->Execute();
//采单帧
CImageDataPointer objImageDataPtr;
objImageDataPtr = objStreamPtr->GetImage(500);//超时时间使用500ms，用户可以自行设定
if (objImageDataPtr->GetStatus() == GX_FRAME_STATUS_SUCCESS)
{
//采图成功而且是完整帧，可以进行图像处理...
       qDebug()<< "ImageInfo: " << objImageDataPointer->GetStatus() << endl;
       qDebug() << "ImageInfo: " << objImageDataPointer->GetWidth() << endl;
       qDebug() << "ImageInfo: " << objImageDataPointer->GetHeight() << endl;
       qDebug() << "ImageInfo: " << objImageDataPointer->GetPayloadSize() << endl;
}
//停采
objFeatureControlPtr->GetCommandFeature("AcquisitionStop")->Execute();
objStreamPtr->StopGrab();
//关闭流通道
objStreamPtr->Close();
```

#### 反格式化

在进程退出之前必须调用`IGXFactory::GetInstance().Uninit()`接口释放`GxIAPICPP`申请的所有资源。

```C++
//关闭设备之后，不能再调用其他任何库接口
IGXFactory::GetInstance().Uninit();
```

