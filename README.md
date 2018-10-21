# DXVK

A Vulkan-based translation layer for Direct3D 10/11 which allows running 3D applications on Linux using Wine.

For the current status of the project, please refer to the [project wiki](https://github.com/doitsujin/dxvk/wiki).


## How to use
In order to install DXVK package run the following commands. `~/.wine` stands for the default wine prefix in your home folder you may change that to another path if you know what you are doing. Also be sure the version is up to date with version on top of the [release page](https://github.com/doitsujin/dxvk/releases). If its not please do a pull request for this readme to change it.
```
cd /tmp
export WINEPREFIX=~/.wine
wget https://github.com/doitsujin/dxvk/releases/download/v0.90/dxvk-0.90.tar.gz
tar -xvf dxvk-0.90.tar.gz
winetricks --force dxvk-0.90/setup_dxvk.verb
```

This will **copy** the DLLs into the `system32` and `syswow64` directories of your wine prefix and set up the required DLL overrides. Pure 32-bit prefixes are also supported.

Verify that your application uses DXVK instead of wined3d by checking for the presence of the log files `d3d11.log` and `dxgi.log` in the application's directory, or by enabling the HUD (see notes below).


## Build instructions

### Requirements:
- [wine 3.10](https://www.winehq.org/) or newer
- [Meson](http://mesonbuild.com/) build system (at least version 0.43)
- [MinGW64](http://mingw-w64.org/) compiler and headers (requires threading support)
- [glslang](https://github.com/KhronosGroup/glslang) compile

### Building DLLs

#### The simple way
Inside the DXVK directory, run:
```
./package-release.sh master /your/target/directory --no-package
```

This will create a folder `dxvk-master` in `/your/target/directory`, which contains both 32-bit and 64-bit versions of DXVK, which can be set up in the same way as the release versions as noted above.

#### Compiling manually
```
# 64-bit build. For 32-bit builds, replace
# build-win64.txt with build-win32.txt
meson --cross-file build-win64.txt --buildtype release --prefix /your/dxvk/directory build.w64
cd build.w64
ninja
ninja install
```

The D3D10, D3D11 and DXGI DLLs as well as a shell script to set up DXVK for a specific wine prefix will be located in `/your/dxvk/directory/bin`.

### Notes on Vulkan drivers
Before reporting an issue, please check the [Wiki](https://github.com/doitsujin/dxvk/wiki/Driver-support) page on the current driver status and make sure you run a recent enough driver version for your hardware.

### Online multi-player games
Manipulation of Direct3D libraries in multi-player games may be considered cheating and can get your account **banned**. This may also apply to single-player games with an embedded or dedicated multiplayer portion. **Use at your own risk.**

### HUD
The `DXVK_HUD` environment variable controls a HUD which can display the framerate and some stat counters. It accepts a comma-separated list of the following options:
- `devinfo`: Displays the name of the GPU and the driver version.
- `fps`: Shows the current frame rate.
- `frametimes`: Shows a frame time graph.
- `submissions`: Shows the number of command buffers submitted per frame.
- `drawcalls`: Shows the number of draw calls and render passes per frame.
- `pipelines`: Shows the total number of graphics and compute pipelines.
- `memory`: Shows the amount of device memory allocated and used.
- `version`: Shows DXVK version.

Additionally, `DXVK_HUD=1` has the same effect as `DXVK_HUD=devinfo,fps`.

### Device filter
Some applications do not provide a method to select a different GPU. In that case, DXVK can be forced to use a given device:
- `DXVK_FILTER_DEVICE_NAME="Device Name"` Selects devices with a matching Vulkan device name, which can be retrieved with tools such as `vulkaninfo`.

**Note:** If the device filter is configured incorrectly, it may filter out all devices and applications will be unable to create a D3D device.

### State cache
DXVK caches pipeline state by default, so that shaders can be recompiled ahead of time on subsequent runs of an application, even if the driver's own shader cache got invalidated in the meantime. This cache is enabled by default, and generally reduces stuttering.

The following environment variables can be used to control the cache:
- `DXVK_STATE_CACHE=0` Disables the state cache.
- `DXVK_STATE_CACHE_PATH=/some/directory` Specifies a directory where to put the cache files. Defaults to the current working directory of the application.

### Debugging
The following environment variables can be used for **debugging** purposes.
- `VK_INSTANCE_LAYERS=VK_LAYER_LUNARG_standard_validation` Enables Vulkan debug layers. Highly recommended for troubleshooting rendering issues and driver crashes. Requires the Vulkan SDK to be installed on the host system.
- `DXVK_LOG_LEVEL=none|error|warn|info|debug` Controls message logging.
- `DXVK_LOG_PATH=/some/directory` Changes path where log files are stored.
- `DXVK_CONFIG_FILE=/xxx/dxvk.conf` Sets path to the configuration file.

## Troubleshooting
DXVK requires threading support from your mingw-w64 build environment. If you
are missing this, you may see "error: 'mutex' is not a member of 'std'". On
Debian and Ubuntu, this can usually be resolved by using the posix alternate, which
supports threading. For example, choose the posix alternate from these
commands (use i686 for 32-bit):
```
sudo update-alternatives --set x86_64-w64-mingw32-gcc `which x86_64-w64-mingw32-gcc-posix`
sudo update-alternatives --set x86_64-w64-mingw32-g++ `which x86_64-w64-mingw32-g++-posix`
sudo update-alternatives --set i686-w64-mingw32-gcc `which i686-w64-mingw32-gcc-posix`
sudo update-alternatives --set i686-w64-mingw32-g++ `which i686-w64-mingw32-g++-posix`
```
