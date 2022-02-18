### MacOS 构建步骤

参考 https://qv2ray.net/lang/zh/hacking/manuallybuild.html

1. 下载源码和子模块，并编译 qt 的 MacOS 打包插件 macdeployqt

```
git clone --depth=1 --recurse-submodules https://github.com/Qv2ray/Qv2ray.git && cd Qv2ray
brew install protobuf grpc ninja pkg-config openssl qt@5
git clone https://github.com/Qv2ray/macdeployqt-patched
cd macdeployqt-patched
mkdir build; cd build;
cmake .. -DCMAKE_BUILD_TYPE=Release; cmake --build .
cp -v ./macdeployqt /usr/local/opt/qt@5/bin/macdeployqt
export PATH="/usr/local/opt/qt@5/bin:$PATH"
```

可能需要 export openssl 和 qt 的环境变量，让 pkg-config 找到他们

```
export PKG_CONFIG_PATH="/usr/local/opt/openssl@3/lib/pkgconfig:$PKG_CONFIG_PATH"
export PKG_CONFIG_PATH="/usr/local/opt/qt@5/lib/pkgconfig:$PKG_CONFIG_PATH"
```

手动复制到 qt5 插件文件夹

```
cp -v  ../macdeployqt-patched/build/macdeployqt /usr/local/Cellar/qt@5/5.15.2_1/bin
```

2. 生成编译配置文件 makefile.目标系统根据实际情况修改。

```
cd .. ; mkdir build; cd build;
cmake .. -GNinja \
-DCMAKE_BUILD_TYPE=Release \
-DCMAKE_OSX_DEPLOYMENT_TARGET=12.0

```

可能需要修改 cmake/protobuf.cmake 文件，让编译器找到 protobuf。在 find_package(Protobuf REQUIRED)前面添加。具体路径根据实际情况修改。

```
set(Protobuf_PREFIX_PATH
"/usr/local/Cellar/protobuf/3.19.4/include"
"/usr/local/Cellar/protobuf/3.19.4/lib"
"/usr/local/Cellar/protobuf/3.19.4/bin" )
list(APPEND CMAKE_PREFIX_PATH "${Protobuf_PREFIX_PATH}")
```

3. 编译

```
cmake --build . -j 12
```

4. 修改 qv2ray.app/Contents/Resources/qt.conf 为以下内容

```
[Paths]
Plugins = PlugIns
Imports = Resources/qml
Qml2Imports = Resources/qml

```

5. 最后手动在 qv2ray.app/Contents/Resources 下创建 plugins 文件夹，并将 build 文件夹下的 libQvPlugin-BuiltinProtocolSupport.so 和 libQvPlugin-BuiltinSubscriptionSupport.so 复制进去
