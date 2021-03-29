[TOC]



## 1. 安装depot tools

**Windows**：
国外下载：[https://storage.googleapis.com/chrome-infra/depot_tools.zip](https://link.jianshu.com/?t=https://storage.googleapis.com/chrome-infra/depot_tools.zip)
下载完把压缩包解压，然后把解压目录加入PATH环境变量

**Linux（Android）/Mac（IOS）**：
安装git
国外：git clone [https://chromium.googlesource.com/chromium/tools/depot_tools.git](https://link.jianshu.com/?t=https://chromium.googlesource.com/chromium/tools/depot_tools.git)
国内：git clone [https://source.codeaurora.org/quic/lc/chromium/tools/depot_tools](https://link.jianshu.com/?t=https://source.codeaurora.org/quic/lc/chromium/tools/depot_tools)

- 配置坏境变量

```
$ echo "export PATH=$PWD/depot_tools:$PATH" > $HOME/.bash_profile
$ source $HOME/.bash_profile
```

- 检测配置是否成功

```
$ echo $PATH
```


## 2.安装依赖软件

**Windows**：
a、系统locale最好设置成English，就是控制面板里面的Region.
b、安装”Visual Studio 2017“，其他版本都不受官方支持。
c、操作系统必须是Windows 7 x64及以上版本，x86操作系统都不支持。
d、安装VS2017时必须有下列组件：
•Visual C++, which will select three sub-categories including MFC
•Universal Windows Apps Development Tools > Tools
•Universal Windows Apps Development Tools > Windows 10 SDK (10.0.10586)
e、新开个cmd中运行`set DEPOT_TOOLS_WIN_TOOLCHAIN=0`，之后所以脚本都在这个cmd中运行
f、编译是用ninja而不是VS！
**Linux**：看后面
**Android**：
安装Java OpenJDK：

```
$ sudo apt-get install openjdk-7-jdk
$ sudo update-alternatives --config javac
$ sudo update-alternatives --config java
$ sudo update-alternatives --config javaws
$ sudo update-alternatives --config javap
$ sudo update-alternatives --config jar
$ sudo update-alternatives --config jarsigner
```

**Mac（IOS）**：
安装最新XCode



## 3.下源码

### 3.1 创建目录

```
mkdir webrtc-checkout
cd webrtc-checkout
```

### 3.2 fetch and sync

**Windows**：

```
fetch --nohooks webrtc
gclient sync
```

**Linux**：

```
export GYP_DEFINES="OS=linux"
fetch --nohooks webrtc_android
gclient sync
./build/install-build-deps.sh
```

**Android**：

```
export GYP_DEFINES="OS=android"
fetch --nohooks webrtc_android
gclient sync
./build/install-build-deps.sh
```

**Mac**：

```
export GYP_DEFINES="OS=mac"
fetch --nohooks webrtc_ios
gclient sync
```

**IOS**：

```
export GYP_DEFINES="OS=ios"
fetch --nohooks webrtc_ios
gclient sync
```



## 4. 切换分支

```
 git checkout -b m89 refs/remotes/branch-heads/4389
 gclinet sync
```

或者

```
gclient sync -r 52b6562a10b495cf771d8388ee51990d56074059 --force
```

可以从 [**Release Notes**](https://webrtc.org/release-notes/) 中找到，可替换成自己所需的版本的 commit id 或者直接使用最新的 commit id



## 5. 生成ninja项目文件

**Windows/Linux**：
生成debug版ninja项目文件：`gn gen out/Default`
生成release版ninja项目文件：`gn gen out/Default --args='is_debug=false'`
清空ninja项目文件：`gn clean out/Default`

**Android**：
使用gn生成：`gn gen out/Default --args='target_os="android" target_cpu="arm"'`
生成ARM64版：`gn gen out/Default --args='target_os="android" target_cpu="arm64"'`
生成32位 x86版：`gn gen out/Default --args='target_os="android" target_cpu="x86"'`
生成64位 x64版：`gn gen out/Default --args='target_os="android" target_cpu="x64"'`

**Mac**:
使用gn生成：

`gn gen out/Release --ide=xcode --args='is_debug=false target_os="mac" target_cpu="x64" is_component_build=false' --mac_deployment_target=10.15 --rtc_include_tests=true`



**IOS**：
生成ARM版：`gn gen out/Debug-device-arm32 --args='target_os="ios" target_cpu="arm" is_component_build=false'`
生成ARM64版：`gn gen out/Debug-device-arm64 --args='target_os="ios" target_cpu="arm64" is_component_build=false'`
生成32位模拟器版：`gn gen out/Debug-sim32 --args='target_os="ios" target_cpu="x86" is_component_build=false'`
生成64位模拟器版：`gn gen out/Debug-sim64 --args='target_os="ios" target_cpu="x64" is_component_build=false'`

## 6. iOS 编译和调试

### 6.1 编译release 包

**armv32**

```
#armv32
gn gen out/Release_armv32 --args='target_os="ios" target_cpu="arm" is_component_build=false is_debug=false'
ninja -C out/Release_armv32
```

**armv64**

```
#armv64
gn gen out/Release_armv64 --args='target_os="ios" target_cpu="arm64" is_component_build=false is_debug=false'
ninja -C out/Release_armv64
```

### 6.2 编译debug 包

#### 6.2.1 修改下脚本

- `src/examples/BUILD.gn` 中，搜索 `ios_app_bundle("AppRTCMobile")`，为其中增加以下内容（bundle id 设置为实际使用的独特 id）：

```gn
      extra_substitutions = [
        "PRODUCT_BUNDLE_IDENTIFIER=com.github.piasy.AppRTCMobile",
      ]
```

- `src` 目录下执行 

  **armv32**

  ```
  #armv32
  gn gen out/Debug_armv32 --args='target_os="ios" target_cpu="arm" is_component_build=false enable_dsyms=true is_debug=true' --ide=xcode
  ninja -C out/Debug_armv32 AppRTCMobile
  ```

  **armv64**

  ```
  #armv64
  gn gen out/Debug_armv64 --args='target_os="ios" target_cpu="arm64" is_component_build=false enable_dsyms=true is_debug=true' --ide=xcode
  ninja -C out/Debug_armv64 AppRTCMobile
  ```

  > enable_dsyms 断点调试
  > ide 生成工程
  > is_debug debug版本

- 用 Xcode 打开 `src/out/**/all.xcworkspace`，run target 选择 `AppRTCMobile`，工程文件的设置 target 也选择 `AppRTCMobile`；



## 7. mac 编译和调试

### 7.1 编译release 包

```
gn gen out/Release --args='is_debug=false target_cpu="x64" target_os="mac" is_component_build=false' --mac_deployment_target=10.15 --rtc_include_tests=true
```

### 7.2 编译debug 包

#### 7.2.1 修改下脚本

- `src/examples/BUILD.gn` 中，搜索 `mac_app_bundle("AppRTCMobile")`，为其中增加以下内容（bundle id 设置为实际使用的独特 id）：

```gn
      extra_substitutions = [
        "PRODUCT_BUNDLE_IDENTIFIER=com.github.piasy.AppRTCMobile",
      ]
```

- `src` 目录下执行 

  ```
  gn gen out/Debug --ide=xcode --args='is_debug=true is_component_build=false enable_dsyms=true target_cpu="x64" target_os="mac"' --mac_deployment_target=10.15 --rtc_include_tests=true
  ```

  > enable_dsyms 断点调试
  > ide 生成工程
  > is_debug debug版本

- 用 Xcode 打开 `src/out/**/all.xcworkspace`，run target 选择 `AppRTCMobile`，工程文件的设置 target 也选择 `AppRTCMobile`；





**拷贝**

```
for i in `find /Users/zf/webrtc1128/webrtc-checkout/src/out_ios32 -name "lib*.a"`
do
echo $i
cp $i ./out_ios32/
done
libtool -static -v -o out_ios32/libwebrtc32.a out_ios32/*.a
strip -S -X out_ios32/libwebrtc32.a
for i in `find /Users/zf/webrtc1128/webrtc-checkout/src/out_ios64 -name "lib*.a"`
do
echo $i
cp $i ./out_ios64/
done
libtool -static -v -o out_ios64/libwebrtc64.a out_ios64/*.a
strip -S -X out_ios64/libwebrtc64.a
```



## 6.编译源码

Windows/Linux/Android/Mac/iOS：

```
ninja -C out/Default
```

好了，这样就编译出来所有相关的库和测试程序。



## 7. 其他

### 6.1 生成vs工程windows
1.生成VS项目文件

```
set DEPOT_TOOLS_WIN_TOOLCHAIN=0
set GYP_GENERATORS=msvs-ninja,ninja
set GYP_MSVS_VERSION=2015 (这里是2013会出现问题，生成的文件缺失很多)
```

生成VS2013项目文件(推荐使用)

```
gn gen out/Default –ide=vs2013
```

生成VS2015项目文件

```
gn gen out/Default –ide=vs2015
gn gen out/Default -ide=vs2015 --args="is_debug=true is_component_build=true target_cpu=\
```



## 7. 参考资料

### 7.1 [WebRTC 里程碑](https://chromiumdash.appspot.com/branches)

