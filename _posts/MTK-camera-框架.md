---
title: MTK camera 框架
date: 2018-05-11 10:19:25
tags:
---

Author JOY.
<!-- excerpt -->

### 源码路径 (mtk_6735 android_5.1)

```

system/core/rootdir/init.rc
frameworks/av/media/mediaserver/main_mediaserver.cpp

```

### 启动流程
Android的各个子模块的启动都是从它们的Service的启动开始的，所以我们将从CameraService的启动开始分析。CameraService的启动就在MediaServer的main函数中，代码路径在：frameworks/av/media/mediaserver/main_mediaserver.cpp，而mediaserver是在init.rc里配置启动的。

* system/core/rootdir/init.rc

```

service media /system/bin/mediaserver
    class main
    user root
    group audio camera inet net_bt net_bt_admin net_bw_acct drmrpc mediadrm media sdcard_r system net_bt_stack
    ioprio rt 4

```

* frameworks/av/media/mediaserver/main_mediaserver.cpp

```

int main(int argc __unused, char** argv) {
  CameraService::instantiate();  
}

```

* frameworks/av/services/camera/libcameraservice/CameraService.h

```

class CameraService :
    public BinderService<CameraService>,
    public BnCameraService,
    public IBinder::DeathRecipient,
    public camera_module_callbacks_t{
    virtual int32_t     getNumberOfCameras();
    virtual status_t    getCameraInfo(int cameraId,
    static char const* getServiceName() { return "media.camera"; }

    class Client : public BnCamera, public BasicClient
    {
      virtual status_t      connect(const sp<ICameraClient>& client) = 0;
      virtual void          disconnect();
      virtual status_t      startPreview() = 0;
      virtual void          stopPreview() = 0;
      virtual status_t      startRecording() = 0;
      virtual void          stopRecording() = 0;
      virtual status_t      takePicture(int msgType) = 0;
      virtual String8       getParameters() const = 0;
      virtual status_t      sendCommand(int32_t cmd, int32_t arg1, int32_t arg2) = 0;
    }

    class BasicClient : public virtual RefBase
    {
      virtual status_t    initialize(camera_module_t *module) = 0;
      sp<IBinder> getRemote() {
          return mRemoteBinder;
      }
    }
}

class BinderService
{
  static status_t publish(bool allowIsolated = false) {
          sp<IServiceManager> sm(defaultServiceManager());
          return sm->addService(
                  String16(SERVICE::getServiceName()),
                  new SERVICE(), allowIsolated); // 添加servic到servicemanager
  }
  static void instantiate() { publish(); }
}

```

* frameworks/av/services/camera/libcameraservice/CameraService.cpp

```

void CameraService::onFirstRef()
{
    BnCameraService::onFirstRef();

    if (hw_get_module(CAMERA_HARDWARE_MODULE_ID,
                (const hw_module_t **)&mModule) < 0) {
        ALOGE("Could not load camera HAL module");
        mNumberOfCameras = 0;
    } else {
        ALOGI("Loaded \"%s\" camera module", mModule->common.name);
        mNumberOfCameras = mModule->get_number_of_cameras();

        if (mModule->common.module_api_version >=
                CAMERA_MODULE_API_VERSION_2_1) {
            mModule->set_callbacks(this);
        }
        CameraDeviceFactory::registerService(this);
    }
}

```

* vendor/mediatek/proprietary/hardware/mtkcam/module_hal/module/module.h

```

static
camera_module
get_camera_module()
{
    camera_module module = {
        common:{
             tag                    : HARDWARE_MODULE_TAG,
             #if (PLATFORM_SDK_VERSION >= 21)
             module_api_version     : CAMERA_MODULE_API_VERSION_2_3,
             #else
             module_api_version     : CAMERA_DEVICE_API_VERSION_1_0,
             #endif
             hal_api_version        : HARDWARE_HAL_API_VERSION,
             id                     : CAMERA_HARDWARE_MODULE_ID,
             name                   : "MediaTek Camera Module",
             author                 : "MediaTek",
             methods                : get_module_methods(),
             dso                    : NULL,
             reserved               : {0},
        },
        get_number_of_cameras       : get_number_of_cameras,
        get_camera_info             : get_camera_info,
        set_callbacks               : set_callbacks,
        get_vendor_tag_ops          : get_vendor_tag_ops,
        #if (PLATFORM_SDK_VERSION >= 21)
        open_legacy                 : open_legacy,
        #endif
        reserved                    : {0},
    };
    return  module;
};

static
int
get_number_of_cameras(void)
{
    return  NSCam::getCamDeviceManager()->getNumberOfDevices();
}

```

* vendor/mediatek/proprietary/hardware/mtkcam/module_hal/devicemgr/CamDeviceManagerBase.cpp

```

int32_t
CamDeviceManagerBase::
getNumberOfDevices()
{
    mi4DeviceNum = enumDeviceLocked();
    return  mi4DeviceNum;
}

```

* vendor/mediatek/proprietary/platform/mt6735/hardware/mtkcam/D1/devicemgr/CamDeviceManagerImp.cpp

```

int32_t
CamDeviceManagerImp::
enumDeviceLocked()
{
    IHalSensorList*const pHalSensorList = IHalSensorList::get();
    size_t const sensorNum = pHalSensorList->searchSensors();
}

```

* vendor/mediatek/proprietary/platform/mt6735/hardware/mtkcam/D1/hal/sensor/HalSensorList.cpp

```
MUINT
HalSensorList::
searchSensors()
{
    return  enumerateSensor_Locked();
}

```

* vendor/mediatek/proprietary/platform/mt6735/hardware/mtkcam/D2/hal/sensor/HalSensorList.enumList.cpp

```

MUINT
HalSensorList::
enumerateSensor_Locked()
{
  SensorDrv *const pSensorDrv = SensorDrv::get();
  SeninfDrv *const pSeninfDrv = SeninfDrv::createInstance();
  int const iSensorsList = pSensorDrv->impSearchSensor(NULL);
  querySensorDrvInfo();
  buildSensorMetadata();
  pSensorInfo = pSensorDrv->getMainSensorInfo();
  pSensorInfo = pSensorDrv->getSubSensorInfo();
  pSensorInfo = pSensorDrv->getMain2SensorInfo();
}

```

* vendor/mediatek/proprietary/platform/mt6735/hardware/mtkcam/D1/hal/sensor/imgsensor_drv.cpp

```

SensorDrv*
SensorDrv::
get()
{
    return ImgSensorDrv::singleton();
}

ImgSensorDrv*
ImgSensorDrv::
singleton()
{
    static ImgSensorDrv inst;
    return &inst;
}

MINT32
ImgSensorDrv::impSearchSensor(pfExIdChk pExIdChkCbf)
{
  GetSensorInitFuncList(&m_pstSensorInitFunc);
  sprintf(cBuf,"/dev/%s",CAMERA_HW_DEVNAME);
  m_fdSensor = ::open(cBuf, O_RDWR);
  err = ioctl(m_fdSensor, KDIMGSENSORIOC_X_SET_DRIVER,&id[KDIMGSENSOR_INVOKE_DRIVER_0] );
  err = ioctl(m_fdSensor, KDIMGSENSORIOC_T_CHECK_IS_ALIVE);
}

```

* vendor/mediatek/proprietary/platform/mt6735/hardware/mtkcam/D1/hal/sensor/seninf_drv.cpp

```

SeninfDrv*
SeninfDrv::createInstance()
{
    return SeninfDrvImp::getInstance();
}

SeninfDrv*
SeninfDrvImp::
getInstance()
{
    static SeninfDrvImp singleton;
    return &singleton;
}

int SeninfDrvImp::init()  //kk test
{
  // Open isp driver
  m_pIspDrv = IspDrv::createInstance();
  m_fdSensor = open(SENSOR_DEV_NAME, O_RDWR);
}

```

* kernel-3.10/drivers/misc/mediatek/imgsensor/src/mt6735/kd_sensorlist.c

```

static long CAMERA_HW_Ioctl(
    struct file *a_pstFile,
    unsigned int a_u4Command,
    unsigned long a_u4Param
)
{
  case KDIMGSENSORIOC_T_CHECK_IS_ALIVE:
      i4RetValue = adopt_CAMERA_HW_CheckIsAlive();
      break;
}
inline static int adopt_CAMERA_HW_CheckIsAlive(void)
{
  kdModulePowerOn((CAMERA_DUAL_CAMERA_SENSOR_ENUM *)g_invokeSocketIdx, g_invokeSensorNameStr, true, CAMERA_HW_DRVNAME1);
  err = g_pSensorFunc->SensorFeatureControl(g_invokeSocketIdx[i], SENSOR_FEATURE_CHECK_SENSOR_ID, (MUINT8 *)&sensorID, &retLen);
}

int
kdModulePowerOn(
CAMERA_DUAL_CAMERA_SENSOR_ENUM socketIdx[KDIMGSENSOR_MAX_INVOKE_DRIVERS],
char sensorNameStr[KDIMGSENSOR_MAX_INVOKE_DRIVERS][32],
BOOL On,
char *mode_name)
{
  ret = kdCISModulePowerOn(socketIdx[i], sensorNameStr[i], On, mode_name);
}

```

* kernel-3.10/drivers/misc/mediatek/mach/mt6735/tx6735_65c_xz_l1/camera/camera/kd_camera_hw.c

```

int kdCISModulePowerOn(CAMERA_DUAL_CAMERA_SENSOR_ENUM SensorIdx, char *currSensorName, BOOL On, char* mode_name)
{
  // camera上电代码
}

```

* kernel-3.10/drivers/misc/mediatek/imgsensor/src/mt6735/vk1206_mipi_raw/vk1206mipi_Sensor.c // 具体camera内核驱动

```

UINT32 VK1206MIPISensorInit(PSENSOR_FUNCTION_STRUCT *pfFunc)

{
	/* To Do : Check Sensor status here */
	if (pfFunc!=NULL)
		*pfFunc=&sensor_func;
	return ERROR_NONE;
}	/*	vk1206_MIPI_RAW_SensorInit	*/

static kal_uint32 feature_control(MSDK_SENSOR_FEATURE_ENUM feature_id,
							 UINT8 *feature_para,UINT32 *feature_para_len)
{
  case SENSOR_FEATURE_CHECK_SENSOR_ID:
  get_imgsensor_id(feature_return_para_32);
}
static kal_uint32 get_imgsensor_id(UINT32 *sensor_id)
{
  //通过i2c 读取芯片id
}

```


### Camera open 流程

代码调用栈

```
frameworks/base/core/java/android/hardware/Camera.java
  frameworks/base/core/jni/android_hardware_Camera.cpp
    frameworks/av/services/camera/libcameraservice/CameraService.cpp
      vendor/mediatek/proprietary/hardware/mtkcam/module_hal/module/module.h
        vendor/mediatek/proprietary/hardware/mtkcam/module_hal/devicemgr/CamDeviceManagerBase.cpp
```

* /home/hml/joy/mt6735/mt6735_53_v2.47/frameworks/base/core/java/android/hardware/Camera.java

```
public static Camera open() {
    int numberOfCameras = getNumberOfCameras();
    CameraInfo cameraInfo = new CameraInfo();
    for (int i = 0; i < numberOfCameras; i++) {
        getCameraInfo(i, cameraInfo);
        if (cameraInfo.facing == CameraInfo.CAMERA_FACING_BACK) {
            return new Camera(i);
        }
    }
}

Camera(int cameraId) {
        int err = cameraInitNormal(cameraId);
}

private int cameraInitNormal(int cameraId) {
    return cameraInitVersion(cameraId, CAMERA_HAL_API_VERSION_NORMAL_CONNECT);
}

private int cameraInitVersion(int cameraId, int halVersion) {
  return native_setup(new WeakReference<Camera>(this), cameraId, halVersion, packageName);
}

```

* frameworks/base/core/jni/android_hardware_Camera.cpp

```
static jint android_hardware_Camera_native_setup(JNIEnv *env, jobject thiz,
    jobject weak_this, jint cameraId, jint halVersion, jstring clientPackageName)
{
  camera = Camera::connect(cameraId, clientName,
                  Camera::USE_CALLING_UID);
}
```

* frameworks/av/camera/Camera.cpp

```
sp<Camera> Camera::connect(int cameraId, const String16& clientPackageName,
        int clientUid)
{
    return CameraBaseT::connect(cameraId, clientPackageName, clientUid);
}```

* frameworks/av/camera/CameraBase.cpp

```
emplate <typename TCam, typename TCamTraits>
sp<TCam> CameraBase<TCam, TCamTraits>::connect(int cameraId,
                                               const String16& clientPackageName,
                                               int clientUid)
{
  TCamConnectService fnConnectService = TCamTraits::fnConnectService;
        status = (cs.get()->fnConnectService)(cl, cameraId, clientPackageName, clientUid, c->mCamera);
}
```

* frameworks/av/services/camera/libcameraservice/CameraService.cpp

```
status = connectHelperLocked(client,
                                     cameraClient,
                                     cameraId,
                                     clientPackageName,
                                     clientUid,
                                     callingPid);

status_t CameraService::connectHelperLocked(
       /*out*/
       sp<Client>& client,
       /*in*/
       const sp<ICameraClient>& cameraClient,
       int cameraId,
       const String16& clientPackageName,
       int clientUid,
       int callingPid,
       int halVersion,
       bool legacyMode) {
   client = new CameraClient(this, cameraClient,
                   clientPackageName, cameraId,
                   facing, callingPid, clientUid, getpid(), legacyMode);
   status_t status = connectFinishUnsafe(client, client->getRemote());                
}      

status_t CameraService::connectFinishUnsafe(const sp<BasicClient>& client,
                                            const sp<IBinder>& remoteCallback) {
    status_t status = client->initialize(mModule);
}                           
```

* frameworks/av/services/camera/libcameraservice/api1/CameraClient.cpp

```
status_t CameraClient::initialize(camera_module_t module) {
  mHardware = new CameraHardwareInterface(camera_device_name);
  res = mHardware->initialize(&module->common);
  mHardware->setMtkCallbacks(
              mtkMetadataCallback,
              (void )(uintptr_t)mCameraId
              );
  mHardware->setCallbacks(
              notifyCallback,
              dataCallback,
              dataCallbackTimestamp,
              (void )(uintptr_t)mCameraId);              
}

```

* frameworks/av/services/camera/libcameraservice/device1/CameraHardwareInterface.h

```
status_t initialize(hw_module_t module)
{
  camera_module_t cameraModule = reinterpret_cast<camera_module_t >(module);
  module->methods->open(
                module, mName.string(), (hw_device_t)&mDevice);
  initHalPreviewWindow();
}
```

* vendor/mediatek/proprietary/hardware/mtkcam/module_hal/module/module.h

```
static
int
open_device(hw_module_t const* module, const char* name, hw_device_t** device)
{
    return  NSCam::getCamDeviceManager()->open(device, module, name);
}

```

* vendor/mediatek/proprietary/hardware/mtkcam/module_hal/devicemgr/CamDeviceManagerBase.cpp

```
status_t
CamDeviceManagerBase::
open(
    hw_device_t** device,
    hw_module_t const* module,
    char const* name,
    uint32_t device_version
)
{
  return  openDeviceLocked(device, module, i4OpenId, device_version);
}
```

* vendor/mediatek/proprietary/hardware/mtkcam/module_hal/devicemgr/CamDeviceManagerBase.openDevice.cpp

```
status_t
CamDeviceManagerBase::
openDeviceLocked(
    hw_device_t** device,
    hw_module_t const* module,
    int32_t const i4OpenId,
    uint32_t device_version
)
{
  pDevice = pPlatform->createCam1Device(s8ClientAppMode.string(), i4DebugOpenID);
  device = const_cast<hw_device_t>(pDevice->get_hw_device());
  pDevice->set_hw_module(module);
  pDevice->set_module_callbacks(mpModuleCallbacks);
  pDevice->setDeviceManager(this);
  attachDeviceLocked(pDevice, device_version);
}
```

* vendor/mediatek/proprietary/hardware/mtkcam/v1/device/Cam1DeviceFactory.cpp

```
extern "C"
NSCam::Cam1Device*
createCam1Device(
    String8 const   s8ClientAppMode,
    int32_t const   i4OpenId
)
{
  String8 const s8CamDeviceInstFactory = String8::format("createCam1Device_Default");
  void* pCreateInstance = ::dlsym(handle, s8CamDeviceInstFactory.string());
  pdev = reinterpret_cast<NSCam::Cam1Device* ()(String8 const&, int32_t const)>
                     (pCreateInstance)(s8ClientAppMode, i4OpenId)
  pdev->initialize();
}
```

* vendor/mediatek/proprietary/platform/mt6735/hardware/mtkcam/D1/v1/device/DefaultCam1Device.cpp

```
extern "C"
NSCam::Cam1Device*
createCam1Device_Default(
    String8 const&          rDevName,
    int32_t const           i4OpenId
)
{
    return new DefaultCam1Device(rDevName, i4OpenId);
}

DefaultCam1Device::
DefaultCam1Device(
    String8 const&          rDevName,
    int32_t const           i4OpenId
)
    : Cam1DeviceBase(rDevName, i4OpenId)
{
}

bool
DefaultCam1Device::
onInit()
{
  //(0) power on sensor
  pthread_create(&mThreadHandle, NULL, doThreadInit, this);
  (1) Open 3A
  (2) Init Base.
  Cam1DeviceBase::onInit()
}

void*     
DefaultCam1Device::  
doThreadInit(void* arg)  
{  
    DefaultCam1Device* pSelf = reinterpret_cast<DefaultCam1Device*>(arg);  
    pSelf->mRet = pSelf->powerOnSensor();  
    pthread_exit(NULL);  
    return NULL;  
}  

bool  
DefaultCam1Device::  
powerOnSensor()  
{  
    IHalSensorList* pHalSensorList = IHalSensorList::get();  
    mpHalSensor = pHalSensorList->createSensor(USER_NAME, getOpenId());  
    sensorIdx = getOpenId();  
    mpHalSensor->powerOn(USER_NAME, 1, &sensorIdx);
}

```

* vendor/mediatek/proprietary/platform/mt6735/hardware/mtkcam/D1/hal/sensor/HalSensor.cpp

```
MBOOL
HalSensor::
powerOn(
    char const* szCallerName,
    MUINT const uCountOfIndex,
    MUINT const*pArrayOfIndex
)
{
 mpSeninfDrv->init();
 mpSensorDrv->init(sensorDev);
 setSensorIODrivingCurrent(sensorDev);
 mpSensorDrv->open(sensorDev);
}
```

* vendor/mediatek/proprietary/platform/mt6735/hardware/mtkcam/D1/hal/sensor/imgsensor_drv.cpp

```
MINT32
ImgSensorDrv::open(MINT32 sensorIdx)
{
  err = ioctl(m_fdSensor, KDIMGSENSORIOC_X_SET_CURRENT_SENSOR, &sensorIdx);
  err = ioctl(m_fdSensor, KDIMGSENSORIOC_T_OPEN);
}
```


* vendor/mediatek/proprietary/hardware/mtkcam/v1/device/Cam1DeviceBase.cpp

```
Cam1DeviceBase::
Cam1DeviceBase(
    String8 const&          rDevName,
    int32_t const           i4OpenId
)
    : Cam1Device()
{
}

status_t
Cam1DeviceBase::
initialize()
{
  onInit();
}

bool
Cam1DeviceBase::
onInit()
{
  mpParamsMgr->init();
  mpCamClient = ICamClient::createInstance(mpParamsMgr);
}


```

* vendor/mediatek/proprietary/hardware/mtkcam/v1/device/Cam1Device.cpp

```
Cam1Device::
Cam1Device()
    : ICamDevice()
    , mpModuleCallbacks(NULL)
    , mDevice()
    , mDeviceOps()
{
    MY_LOGD("ctor");
    ::memset(&mDevice, 0, sizeof(mDevice));
    mDevice.priv    = this;
    mDevice.common  = gHwDevice;
    mDevice.ops     = (camera_device_ops*)&mDeviceOps;
    mDeviceOps      = gCameraDevOps;
    //
}

static mtk_camera_device_ops const
gCameraDevOps =
{
    #define OPS(name) name: camera_##name

    {
        OPS(set_preview_window),
        OPS(set_callbacks),
        OPS(enable_msg_type),
        OPS(disable_msg_type),
        OPS(msg_type_enabled),
        OPS(start_preview),
        OPS(stop_preview),
        OPS(preview_enabled),
        OPS(store_meta_data_in_buffers),
        OPS(start_recording),
        OPS(stop_recording),
        OPS(recording_enabled),
        OPS(release_recording_frame),
        OPS(auto_focus),
        OPS(cancel_auto_focus),
        OPS(take_picture),
        OPS(cancel_picture),
        OPS(set_parameters),
        OPS(get_parameters),
        OPS(put_parameters),
        OPS(send_command),
        OPS(release),
        OPS(dump)
    },
    OPS(mtk_set_callbacks),
};

```

### Camera preview 流程

* frameworks/base/core/java/android/hardware/Camera.java

```
public final void setPreviewDisplay(SurfaceHolder holder) throws IOException {
    if (holder != null) {
        setPreviewSurface(holder.getSurface());
    } else {
        setPreviewSurface((Surface)null);
    }
}

```

* frameworks/base/core/jni/android_hardware_Camera.cpp

```
static void android_hardware_Camera_setPreviewSurface(JNIEnv env, jobject thiz, jobject jSurface)
{
  surface = android_view_Surface_getSurface(env, jSurface);
  gbp = surface->getIGraphicBufferProducer();
  camera->setPreviewTarget(gbp);
}
```

* frameworks/av/camera/Camera.cpp

```

// pass the buffered IGraphicBufferProducer to the camera service
status_t Camera::setPreviewTarget(const sp<IGraphicBufferProducer>& bufferProducer)
{
    sp <ICamera> c = mCamera;
    return c->setPreviewTarget(bufferProducer);
}

```

* frameworks/av/camera/ICamera.cpp

```
/ pass the buffered IGraphicBufferProducer to the camera service
status_t setPreviewTarget(const sp<IGraphicBufferProducer>& bufferProducer)
{
    remote()->transact(SET_PREVIEW_TARGET, data, &reply);
}

```

* frameworks/av/services/camera/libcameraservice/api1/CameraClient.cpp

```
status_t CameraClient::setPreviewTarget(
        const sp<IGraphicBufferProducer>& bufferProducer) {

    window = new Surface(bufferProducer, /*controlledByApp*/ true);
    return setPreviewWindow(binder, window);
}

status_t CameraClient::setPreviewWindow(const sp<IBinder>& binder,
        const sp<ANativeWindow>& window) {
    result = native_window_api_connect(window.get(), NATIVE_WINDOW_API_CAMERA);
    // If preview has been already started, register preview buffers now.
    if (mHardware->previewEnabled()) {
        if (window != 0) {
            native_window_set_scaling_mode(window.get(),
                    NATIVE_WINDOW_SCALING_MODE_SCALE_TO_WINDOW);
            native_window_set_buffers_transform(window.get(), mOrientation);
            result = mHardware->setPreviewWindow(window);
        }
    }
    disconnectWindow(mPreviewWindow);
    mSurface = binder;
    mPreviewWindow = window;
}

```

* frameworks/av/services/camera/libcameraservice/device1/CameraHardwareInterface.h

```

status_t setPreviewWindow(const sp<ANativeWindow>& buf)
{
  return mDevice->ops->set_preview_window(mDevice,
                     buf.get() ? &mHalPreviewWindow.nw : 0);
}

```

* vendor/mediatek/proprietary/hardware/mtkcam/v1/device/Cam1Device.cpp

```
static int camera_set_preview_window(
    struct camera_device * device,
    struct preview_stream_ops window
)
{
  pDev->setPreviewWindow(window);
}

status_t
Cam1DeviceBase::
startPreview()
{
  onStartPreview();
  enableDisplayClient();
  mpCamClient->startPreview();
  status = mpCamAdapter->startPreview();
}

DefaultCam1Device::
onStartPreview()
{
  initCameraAdapter();
}
```

* vendor/mediatek/proprietary/hardware/mtkcam/v1/device/Cam1DeviceBase.cpp

```

status_t
Cam1DeviceBase::
setPreviewWindow(preview_stream_ops* window)
{
    status_t status = initDisplayClient(window);
    enableDisplayClient();
}

status_t
Cam1DeviceBase::
initDisplayClient(preview_stream_ops* window)
{
  queryPreviewSize(previewSize.width, previewSize.height);
  mpDisplayClient = IDisplayClient::createInstance();
  mpDisplayClient->init();
  mpDisplayClient->setWindow(window, previewSize.width, previewSize.height, queryDisplayBufCount());
  mpDisplayClient->setImgBufProviderClient(mpCamAdapter);
}

status_t
Cam1DeviceBase::
enableDisplayClient()
{
  queryPreviewSize(previewSize.width, previewSize.height);
  mpDisplayClient->enableDisplay(previewSize.width, previewSize.height, queryDisplayBufCount(), mpCamAdapter);
}

bool
Cam1DeviceBase::
initCameraAdapter()
{
  mpCamAdapter = ICamAdapter::createInstance(mDevName, mi4OpenId, mpParamsMgr);
  mpCamAdapter->init();
  mpCamAdapter->setCallbacks(mpCamMsgCbInfo);
  mpCamAdapter->enableMsgType(mpCamMsgCbInfo->mMsgEnabled);
  mpCamAdapter->setParameters();
  mpDisplayClient->setImgBufProviderClient(mpCamAdapter);
  mpCamClient->setImgBufProviderClient(mpCamAdapter);
}

```

* vendor/mediatek/proprietary/platform/mt6735/hardware/mtkcam/D1/v1/adapter/MtkDefault/MtkDefaultCamAdapter.cpp

```
bool
CamAdapter::
init()
{
  mpResMgrDrv = ResMgrDrv::createInstance(getOpenId());
  mpResMgrDrv->init();
  mpResMgrDrv->setMode(&mode);
  mpCapBufMgr         = CapBufMgr::createInstance();
  mpAllocBufHdl       = AllocBufHandler::createInstance();
  mpDefaultBufHdl     = DefaultBufHandler::createInstance();
  mpDefaultCtrlNode   = DefaultCtrlNode::createInstance("", DefaultCtrlNode::CTRL_NODE_DEFAULT);
  mpStateManager = IStateManager::createInstance();
  mpStateManager->init();
  init3A();
  mpCaptureCmdQueThread = ICaptureCmdQueThread::createInstance(this);
  mpCpuCtrl = CpuCtrl::createInstance();
  mpCpuCtrl->init();

}
```

* vendor/mediatek/proprietary/hardware/mtkcam/v1/client/DisplayClient/DisplayClient.cpp

```
bool
DisplayClient::
enableDisplay(
    int32_t const   i4Width,
    int32_t const   i4Height,
    int32_t const   i4BufCount,
    sp<IImgBufProviderClient>const& rpClient
)
{
  uninit();
  init();
  setWindow(pStreamOps, i4Width, i4Height, i4BufCount);
  setImgBufProviderClient(rpClient);
  enableDisplay();
}

bool
DisplayClient::
enableDisplay()
{
  mpDisplayThread->postCommand(Command(Command::eID_WAKEUP));
}

```

* vendor/mediatek/proprietary/hardware/mtkcam/v1/client/DisplayClient/DisplayThread.cpp

```
bool
DisplayThread::
threadLoop()
{
  case Command::eID_WAKEUP:
  mpThreadHandler->onThreadLoop(cmd);
}

```

* vendor/mediatek/proprietary/hardware/mtkcam/v1/client/DisplayClient/DisplayClient.BufOps.cpp

```
bool
DisplayClient::
onThreadLoop(Command const& rCmd)
{
  prepareAllTodoBuffers(pImgBufQueue);
  pImgBufQueue->startProcessor();
}
```


[!](https://blog.csdn.net/eternity9255/article/details/52790962)
