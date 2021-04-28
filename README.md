# arduino-ide 2.0.0 for Raspberry Pi

## Beta

## Installation

Installation is simply unzipping in a directory, eg. ``/opt/arduino-ide``

For arm 64 bit:
```
mkdir /opt/arduino-ide
cd /opt/arduino-ide
unzip ~/Downloads/arduino-ide_2.0.0-beta.5-snapshot.0dd1e45_Linux_ARM64.zip
```

To run:

```
/opt/arduino-ide/arduino-ide
```

## AppImages

Experimental AppImages:

```
chmod +x Arduino_IDE-aarch64.AppImage
./Arduino_IDE-aarch64.AppImage
```

## Compiling the IDE

At this moment, arduino IDE 2.0.0 is in beta.
The arduino ide requires [clangd](https://github.com/llvm/llvm-project/tree/main/clang-tools-extra/clangd), [arduino language server](), and the [arduino cli](). 
At this moment, this requires compiling clangd and arduino-language-server from source.

 
To compile the arduino ide on raspberry:

## install debian packages

add backports to apt sources, needed for go 1.14.

```
echo 'deb http://deb.debian.org/debian buster-backports main' >> /etc/apt/sources.list

apt-get update -q --fix-missing
apt-get install -y -q software-properties-common
apt-get install -y -q --no-install-recommends build-essential libssl-dev golang-1.14-go golang-1.14-go libxkbfile-dev libnss3-dev
apt-get install autoconf automake libtool curl make cmake ninja-build g++ unzip
 export PATH=/usr/lib/go-1.14/bin:$PATH
```

## clangd

```
git clone -b llvmorg-12.0.0 https://github.com/llvm/llvm-project
cd llvm-project
export LLVM_ROOT=$PWD
mkdir build
cd build
cmake $LLVM_ROOT/llvm/ -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra" -DLLVM_STATIC_LINK_CXX_STDLIB=ON -DLLVM_BUILD_STATIC=ON -G Ninja
cmake --build $LLVM_ROOT/build --target clangd
zip -r clangd-12.0.0_Linux_aarch64.zip ./bin/clangd ./lib/clang/
```
Copy clangd-12.0.0_Linux_aarch64.zip, will be needed later.

## arduino language server

```
export PATH=/usr/lib/go-1.14/bin:$PATH
git clone https://github.com/arduino/arduino-language-server
cd arduino-language-server
go version
# check go version 1.14
go build
zip arduino-language-server_Linux_aarch64.zip ./arduino-language-server
```
Copy arduino-language-server_Linux_aarch64.zip, will be needed later.

## arduino-cli

``arduino-cli`` binaries for Linux_ARMv6, Linux_ARMv7, Linux_ARM64 available from [arduino-cli git](https://github.com/arduino/arduino-cli). The binaries are downloaded by ``arduino-ide/arduino-ide-extension/scripts/download-cli.js`` 

## arduino-ide

```
git clone https://github.com/arduino/arduino-ide
cd arduino-ide
export ARD_DIR=$PWD
cd $ARD_DIR/arduino-ide-extension/
mkdir build
cd build
```
unzip the two binaries created earlier in this directory:
```
unzip ~/src/binaries/arduino-language-server_Linux_aarch64.zip
unzip ~/src/binaries/clangd-12.0.0_Linux_aarch64.zip
```
Continue the build
```
cd $ARD_DIR
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
 . ~/.nvm/nvm.sh
nvm install 12.14.1
npm install --global yarn
npm install "@octokit/core@>=3"
npm install "react@16.14.0" "react-dom@16.14.0"
npm install "less@^2.3.1"
npm install "font-awesome@>=4.3.0"
npm install request

yarn
yarn rebuild:electron
yarn --cwd ./electron/packager/
yarn --cwd ./electron/packager/ package

```

This creates the file ``$ARD_DIR/electron/build/dist/arduino-ide_2.0.0-beta.5-snapshot.0dd1e45_Linux_ARM64.zip``

## appimage

To make deployment easier, create an appimage.
Download latest AppRun and appimagetool from [git](https://github.com/AppImage/AppImageKit/releases/tag/continuous)

```
mkdir -p arduino-ide-appdir/usr/bin arduino-ide-appdir/usr/lib
cd arduino-ide-appdir
export APP_ROOT=$PWD
cd $APP_ROOT/usr/bin
unzip unzip ~/src/binaries/arduino-ide_2.0.0-beta.5-snapshot.0dd1e45_Linux_ARM64.zip
cp *.so $APP_ROOT/usr/lib/
cd $APP_ROOT
cat >arduino-ide.desktop <<EOD
[Desktop Entry]
Type=Application
Version=1.0
Name=Arduino IDE
Icon=arduino-ide
Exec=arduino-ide %f
MimeType=text/x-arduino;
Categories=Development;
Keywords=Arduino;Programming;Development;
X-AppImage-Version=2.0.0-beta5
EOD
# use icon that is not copyrighted
wget -O arduino-ide.svg http://halley.cc/paste/arduino.svg 
wget https://github.com/AppImage/AppImageKit/releases/download/continuous/AppRun-aarch64
mv AppRun-aarch64 AppRun
chmod +x ./AppRun
```
Create the appimage. For aarch64, type:
```  
cd  $APP_ROOT/..
wget https://github.com/AppImage/AppImageKit/releases/download/continuous/appimagetool-aarch64.AppImage
chmod +x ./appimagetool-aarch64.AppImage
ARCH=arm_aarch64 ~/Downloads/appimagetool-aarch64.AppImage ./arduino-ide-appdir
```

For arm 32-bit, type:

```
cd  $APP_ROOT/..
ARCH=arm ~/Downloads/appimagetool-aarch64.AppImage ./arduino-ide-appdir
```

not truncated.
