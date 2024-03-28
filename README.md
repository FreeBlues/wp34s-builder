# WP 34S 构建环境 Docker 镜像

这个镜像允许你以一种非常轻松的方式来编译构建 WP 34S的固件，只需要下载这个 `Docker` 镜像，然后用你的 `Docker` 加载它，

## 技能需求

需要你懂一点 `Docker` 的常用命令和基本操作，大致理解宿主机，镜像，容器这三个基本概念，然后按图索骥就可以了。

## 版本需求

说明：`Docker` 的版本要保证不低于下面所列，暂时只在 `macOS` 上运行成功，还没有在 `Windows` 和 `Linux` 上试过，理论上应该也可以。

###  Docker 版本需求

-  Docker Desktop：4.28.0(139021)
-  Engine：25.0.3

### 操作系统

-  macOS Ventura 13.4.1(c)
-  Windows 未测试
-  Linux 未测试

## 使用方法

首先要安装好 `Docker Desktop`

>注意：不要通过命令行安装，命令行安装的 `Docker` 版本偏低!

接着在你的用户目录下建立一个新目录，用来放置最新版本的 `wp34s` 源代码，命令如下：

```
host> cd ~/
host> mkdir WP34SSRC
```

>说明一下，这个目录是用来让你的宿主机跟容器交换文件用的，你可以把最新版的 `wp34s` 的源码放在这里，然后在容器启动后，在容器里访问这个目录，对于容器来说，它被设置为只读。

>这里属于可选项，因为我已经在 `Docker` 的镜像文件中内置了一套 `V3.3 r924` 的 `WP34S` 源码了

>host>表示你的宿主机提示符

然后下载 `Docker` 镜像文件 [wp34s-builder-src.tar](https://www.123pan.com/s/HQ3cVv-o1rch.html)保存到本地目录，大约 `2.38G`，对于 `github` 提供的免费空间来说太大了，所以放在网盘上。

执行：

```
host> docker load wp34s-builder-src.tar
```

成功后继续执行：

```
host> docker run -it --rm --name wp34s-builder-src -v ~/WP34SSRC:/wp-src:ro wp34s-builder-src
```

执行成功后会进入容器中，因为这个镜像已经内置了一个 `r924` 的源码，目录名为 `wp34s` 所以进入源码目录后直接就可以编译了，命令如下：

```
root@f993ed11dbba:/# cd wp34s/trunk
root@f993ed11dbba:/# make REALBUILD=1
```
>注意，这个路径中的数字 `f993ed11dbba` 是你的容器 `ID`，不一定是这里这个，你只要从命令行成功启动容器之后就会自动进入类似的一个路径下

然后你就会看到编译过程，最后会成功构建 `r924` 的 `firmware`，最终生成的文件是 `calc.bin`  显示信息如下：

```
root@f993ed11dbba:/# cd wp34s/trunk/
root@f993ed11dbba:/wp34s/trunk# make REALBUILD=1
mkdir -p -v Linux64_realbuild/obj
mkdir: created directory 'Linux64_realbuild'
mkdir: created directory 'Linux64_realbuild/obj'
cc -Wall -O1 -g -DHOSTBUILD=1 -m32 -DREALBUILD=1 -o Linux64_realbuild/genchars7.exe genchars7.c

......

arm-none-eabi-nm -n realbuild/calc >realbuild/symbols.txt
arm-none-eabi-nm -S realbuild/calc >>realbuild/symbols.txt
arm-none-eabi-objcopy -O binary --gap-fill 0xff realbuild/calc realbuild/calc.tmp 
Linux64_realbuild/post_process.exe realbuild/calc.tmp realbuild/calc.bin
Post-processing address tables
Monadic commands:    153
Dyadic commands:      47
Triadic commands:     12
Niladic commands:    206
Argument commands:   156
Multi word commands:  11
.fixed          0x00100000    0x1d6f8
.revision       0x0011d6f8        0x8
                0x0011d700                UserFlash = .
                0x00002100                UserFlashSize = (. - UserFlash)
.backupflash    0x0011f800      0x800
.cmdtab         0x000f0000     0x2066
.bss            0x00200000      0x180
.slcdcmem       0x00200180       0x30
.volatileram    0x002001b0      0x210
.persistentram  0x00300000      0x800
root@f993ed11dbba:/wp34s/trunk# 
```

编译生成的固件 `calc.bin` 放在 `/wp34s/trunk/realbuild`目录下，需要从容器中拷贝到宿主机，才能使用，这个操作要在宿主机上执行，命令如下：

```
host> docker cp wp34s-builder-src:/wp34s/trunk/realbuild/calc.bin .
```

如果你不想下载这么大的 `Docker` 镜像文件，也可以通过这里提供的 `Dockerfile` 去自己生成`Docker` 镜像（国内用户大概率会遇到 `winehq` 和 `raw.github` 无法访问的问题，或者访问速度很慢）。

自己创建 `Docker` 镜像的操作可以参考来自[SammysHP](https://github.com/SammysHP/wp34s-builder)的英文说明。

# WP 34S build environment with Docker

This image allows you to easily build WP 34S realcalc firmware images. It provides all tools necessary to compile the firmware as recommended in the official documentation.


## Example usage

```
host> docker run -it --rm --name wp34s-builder -v /path/to/source:/wp-src:ro wp34s-builder
container> cp -a /wp-src wp
container> cd wp/trunk
container> make REALBUILD=1
```

You can then copy the firmware image to the host by running:

```
host> docker cp wp34s-builder:/wp/trunk/realbuild/calc.bin .
```


## TODO

- Automate build by combining copy, make and returning calc.bin
- Fix revision parsing (it might be enough to install svn and use the official SVN repository)


## Credits

Installing Wine inside a Docker image is loosely based on [GitHub:scottyhardy/docker-wine](https://github.com/scottyhardy/docker-wine). Information about the Yagarto compiler was taken from the official "Compiling WP 34S / 31S on Linux" documentation.
