# 目标跟踪
- [目标跟踪](#目标跟踪)
- [一、SDK功能](#一sdk功能)
- [二、技术规格](#二技术规格)
  - [移动端](#移动端)
  - [PC端](#pc端)
- [三、资源依赖](#三资源依赖)
  - [3.1 头文件](#31-头文件)
  - [3.2 模型文件](#32-模型文件)
  - [3.3 动态库](#33-动态库)
- [四、相关说明](#四相关说明)
  - [4.1 处理流程](#41-处理流程)
  - [4.2 调用注意事项](#42-调用注意事项)
  - [4.3 Demo示例](#43-demo示例)
- [五、API文档](#五api文档)
  - [5.1 初始化 VNN_Create_ObjTracking](#51-初始化-vnn_create_objtracking)
  - [5.2 目标跟踪 VNN_Apply_ObjTracking_CPU](#52-目标跟踪-vnn_apply_objtracking_cpu)
  - [5.3 资源释放 VNN_Destroy_ObjTracking](#53-资源释放-vnn_destroy_objtracking)
  - [5.4 设置参数 VNN_Set_ObjTracking_Attr](#54-设置参数-vnn_set_objtracking_attr)
  - [5.5 获取参数 VNN_Get_ObjTracking_Attr](#55-获取参数-vnn_get_objtracking_attr)
- [六、更新记录](#六更新记录)

# 一、SDK功能

指定跟踪目标在起始帧的位置，跟踪目标在后续视频帧中的位置   

API的输入、输出对象可参考[VNN 数据结构](./vnn_data_structure.md)   
API的执行情况（是否成功、错误原因等）可参考 ```VNN_Result``` 对应的[状态码表](./status_code.md)

---

# 二、技术规格

## 移动端

| 指标            | 参数                                |
| --------------- | ----------------------------------- |
| 支持图片格式    | BGRA、RGBA、RGB、NV12、NV21、YUV420 |
| 支持架构        | armeabi-v7、arm64-v8a               |
| Android系统版本 | 5.0+                                |
| iOS系统版本     | 9.0+                                |
| 最大跟踪目标数  | 1                                   |

## PC端

| 指标            | 参数                                       |
| --------------- | ------------------------------------------ |
| 支持图片格式    | BGRA、RGBA、RGB、YUV420F                   |
| 支持架构        | x86(Win Only)、x86_64、arm64(MacOS Only) |
| Windows系统版本 | Win 7+                                     |
| MacOS系统版本   | 10.10+                                     |
| 最大跟踪目标数  | 1                                          |

---

# 三、资源依赖
## 3.1 头文件

```
vnn_objtracking.h
vnn_kit.h
vnn_define.h
```
## 3.2 模型文件
```
object_tracking[1.0.0].vnnmodel
```
## 3.3 动态库
Android/Linux
```
libvnn_core.so
libvnn_kit.so
libvnn_objtracking.so
```
iOS
```
Accelerate.framework
CoreVideo.framework
Foundation.framework
vnn_core_ios.framework
vnn_kit_ios.framework
vnn_objtracking_ios.framework
```
MacOS
```
Accelerate.framework
CoreVideo.framework
Cocoa.framework
vnn_core_osx.framework
vnn_kit_osx.framework
vnn_objtracking_osx.framework
```
Windows
```
vnn_core.dll
vnn_kit.dll
vnn_objtracking.dll
```

---

# 四、相关说明
## 4.1 处理流程   

![pipline](./resource/pipline_object_tracking.png)

## 4.2 调用注意事项
调用 VNN_Apply_ObjTracking_CPU 后，SDK会动态申请内存，用于记录不确定个数的人脸位置信息。在信息处理完成后需调用 VNN_ObjCountDataArr_Free 以释放申请的内存，避免内存泄露。

## 4.3 Demo示例   
Android: [链接](../demos/Android/vnn_android_demo/app/src/main/java/com/duowan/vnndemo/CameraActivity.java)   
iOS: [链接](../demos/iOS/vnn_ios_demo/ios/CameraViewctrls/ViewCtrl_Camera_ObjectTracking.mm)
Windows: [链接](../demos/Windows/vnn_win_demo/demo/src/vnn_helper.cpp)   
MaoOS: [链接](../demos/MacOS/vnn_macos_demo/osx/CameraWindowCtrls/WindowCtrl_Camera_ObjectTracking.mm)   
Linux: [链接](../demos/Linux/vnn_linux_demo/demo/src/vnn_helper.cpp)   

---
# 五、API文档
## 5.1 初始化 VNN_Create_ObjTracking
说明: 输入模型路径，完成SDK的初始化，获得用于调用后续功能的Handle
```cpp
VNN_Result VNN_Create_ObjTracking( VNNHandle * handle, const int argc, const void * argv[] )
```
| 参数   | 含义                                                                                               |
| ------ | -------------------------------------------------------------------------------------------------- |
| handle | 函数调用成功后记录合法的索引，用于调用后续功能，类型为VNN_Handle*，调用成功后handle数值大于0，输出 |
| argc   | 输入模型文件数，类型为const int，输入                                                              |
| argv   | 每个模型文件的具体路径，类型为const char*[ ]，输入                                                 |

返回值: VNN_Result，具体值参见 状态码表  
调用示例:  
``` cpp
VNN_Handle _handle;

std::string model = _modelpath + "/object_tracking[1.0.0].vnnmodel";

const char* argv[] = {
	model.c_str(),
};

const int argc = sizeof(argv)/sizeof(argv[0]);

VNN_Result ret = VNN_Create_ObjTracking(&_handle, argc, argv);
```
## 5.2 目标跟踪 VNN_Apply_ObjTracking_CPU
说明: 输入包含人脸的图像，输出检测结果
```cpp
VNN_Result VNN_Apply_ObjTracking_CPU( VNNHandle handle, const void * input, void * output )
```
| 参数   | 含义                                        |
| ------ | ------------------------------------------- |
| handle | SDK实例索引，类型为 VNN_Handle，输入        |
| input  | 后续的视频帧，类型为const VNN_Image*，输入  |
| output | 跟踪结果，类型为 VNN_ObjCountDataArr*，输出 |

返回值: VNN_Result，具体值参见 状态码表  
调用示例:  
``` cpp
// input: 后续的视频帧，类型VNN_Image
VNN_ObjCountDataArr tracking_result_bbox;
VNN_Apply_ObjTracking_CPU(_handle, &input, &tracking_result_bbox);
```

## 5.3 资源释放 VNN_Destroy_ObjTracking
说明: 不再使用SDK，释放内存等资源
```cpp
VNN_Result VNN_Destroy_ObjTracking(VNNHandle* handle)
```
| 参数   | 含义                                                                           |
| ------ | ------------------------------------------------------------------------------ |
| handle | SDK实例索引，成功释放资源后将被修改为0（无效值），类型为VNN_Handle*，输入&输出 |

返回值: VNN_Result，具体值参见 状态码表  
调用示例:  
``` cpp
VNN_Result ret = VNN_Destroy_ObjTracking(&_handle);
```

## 5.4 设置参数 VNN_Set_ObjTracking_Attr
说明: 设定SDK实例的运行参数
```cpp
VNN_Result VNN_Set_ObjTracking_Attr( VNNHandle handle, const char * name, const void * value )
```
| 参数   | 含义                                |
| ------ | ----------------------------------- |
| handle | SDK实例索引，类型为VNN_Handle，输入 |
| name   | 属性名，类型const char*，输入       |
| value  | 属性值，类型参见下表，输入          |

 **合法属性名和属性值**  

 | 属性名       | 属性含义                 | 属性值                          | 属性值类型  |
 | ------------ | ------------------------ | ------------------------------- | ----------- |
 | _targetImage | 起始帧                   | 有效的VNN_Image对象的地址/指针  | VNN_Image*  |
 | _objRect     | 跟踪目标在起始帧中的坐标 | 有效的VNN_Rect2D对象的地址/指针 | VNN_Rect2D* |

返回值: VNN_Result，具体值参见 状态码表  
调用示例:  
``` cpp
// input：起始帧，类型 VNN_Image
// template_bbox：跟踪目标在起始帧中的坐标，类型 VNN_Rect2D

// 设置起始帧
VNN_Set_ObjTracking_Attr(_handle_objtracking, "_targetImage", &input);

// 设置跟踪目标
VNN_Set_ObjTracking_Attr(_handle_objtracking, "_objRect", &template_bbox);
```

## 5.5 获取参数 VNN_Get_ObjTracking_Attr
说明: 获取SDK实例的运行参数
```cpp
VNN_Result VNN_Get_ObjTracking_Attr( VNNHandle handle, const char * name, const void * value )
```
| 参数   | 含义                                |
| ------ | ----------------------------------- |
| handle | SDK实例索引，类型为VNN_Handle，输入 |
| name   | 属性名，类型const char*，输入       |
| value  | 属性值，类型参见下表，输出          |

 **合法属性名和属性值**  

 | 属性名   | 属性含义 | 属性值 | 属性值类型 |
 | -------- | -------- | ------ | ---------- |
 | 暂时为空 |

   
返回值: 类型为VNN_Result，具体值参见 状态码表  
调用示例:  
``` cpp
//暂时为空
```

---

# 六、更新记录
| 版本   | 日期       | 更新说明 |
| ------ | ---------- | -------- |
| v1.0.0 | 2021.12.07 | 初次发布 |
