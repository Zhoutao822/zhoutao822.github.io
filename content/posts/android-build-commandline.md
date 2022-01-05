---
title: "Build Android App with Commandline"
date: 2021-12-20T21:32:24+08:00
tags: ["android", "sdkmanager", "ndk", "docker", "jenkins"]
categories: ["tools"]
series: [""]
summary: "Jenkins上没有Android Studio，只能通过命令行工具编译Android项目"
draft: true
editPost:
  URL: "https://github.com/Zhoutao822/zhoutao822.github.io/tree/main/content/"
  Text: "Suggest Changes"
  appendFilePath: true 
---

以Ubuntu20.04为例，使用[android](https://github.com/android)/[ndk-samples](https://github.com/android/ndk-samples)项目进行测试

## 1. java

首先需要配置Java环境，如果是使用docker可以选一个自带java环境的docker镜像就行，java版本一般选java8。如果使用最新的java17，后面编译会不通过。

Ubuntu下安装openjdk命令如下，通过apt安装的jdk会被放在`/usr/lib/jvm`目录下，名称为`java-1.8.0-openjdk-amd64`：

```shell
sudo apt install openjdk-8-jdk    
```

如果之前安装过其他版本的jdk，那么java不会自动切换到新安装的这个版本，需要执行`sudo update-alternatives --config java`

```
sudo update-alternatives --config java                                                                             
There are 4 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                             Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/zulu-17-amd64/bin/java              2173000   auto mode
  1            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode
  2            /usr/lib/jvm/zulu-17-amd64/bin/java              2173000   manual mode

Press <enter> to keep the current choice[*], or type selection number: 
```

这里可以看到系统之安装了多个版本的jdk，默认会使用优先级最大的那个版本，除非我们**输入编号手动指定**，一般来说新安装的jdk并不会出现在这个列表里，需要手动加入，执行以下命令

```shell
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-1.8.0-openjdk-amd64/bin/java 300
sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/java-1.8.0-openjdk-amd64/bin/javac 300
```

```shell
# 同理javac执行sudo update-alternatives --config javac 
sudo update-alternatives --config java               
There are 3 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/zulu-17-amd64/bin/java              2173000   auto mode
  1            /usr/lib/jvm/java-1.8.0-openjdk-amd64/bin/java   300       manual mode
  2            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode
  3            /usr/lib/jvm/zulu-17-amd64/bin/java              2173000   manual mode

Press <enter> to keep the current choice[*], or type selection number: 
```

切换完成后运行`java -version`来确认是否成功。

## 2. sdkmanager 

### 2.1 安装与配置

[Android Studio](https://developer.android.com/studio)官网给出了sdkmanager命令行工具的下载地址，截至2022.01.04，可以通过https://dl.google.com/android/repository/commandlinetools-linux-7583922_latest.zip链接下载。执行以下命令

```shell
wget https://dl.google.com/android/repository/commandlinetools-linux-7583922_latest.zip -O cmd.zip
unzip cmd.zip -d android_sdk
cd android_sdk
mv cmdline-tools latest; mkdir cmdline-tools; mv latest cmdline-tools
```

最终文件路径如下

```shell
~/android_sdk/cmdline-tools/latest > ls
bin  lib  NOTICE.txt  source.properties
```

这里有一个很神奇的地方就是需要把解压后的cmdline-tools下的所有文件移动到cmdline-tools/latest目录下，不然后续无法使用sdkmanager。然后输出一下sdk的路径

```shell
~/android_sdk ❯ pwd
/root/sdk
```

最后需要设置环境变量以及ANDROID_SDK_ROOT

```shell
PATH=$PATH:/root/android_sdk/cmdline-tools/latest/bin
export ANDROID_SDK_ROOT=/root/android_sdk
```

Android Gradle 插件 4.2.0 及更高版本可在您首次构建项目时自动安装所需的 NDK 和 CMake，前提是已预先接受 NDK 和 CMake 的许可。执行下面这个命令配置licenses，后续sdkmanager才能自动下载需要的工具

```shell
yes | sdkmanager --licenses
```

使用[android](https://github.com/android)/[ndk-samples](https://github.com/android/ndk-samples)项目进行测试，clone后进入到`hello-jni`目录中，直接执行以下命令，会自动安装对应的gradle、ndk、cmake、platforms、build-tools之类的编译工具

```shell
./gradlew build
```

### 2.2 其他问题

在使用[android](https://github.com/android)/[ndk-samples](https://github.com/android/ndk-samples)项目进行测试时发现，某些项目指定了cmake的版本为`3.18.1`这是截至目前为止sdkmanager支持的最新版本的cmake。但是这里不能通过`sudo apt install cmake`或者cmake官网安装，因为这些版本的cmake中缺少了ninja工具，也是会导致测试项目无法编译通过，所以需要执行以下命令安装sdkmanager提供的cmake，并且还需要将其添加到环境变量中

```shell
sdkmanager "cmake;3.18.1"
```

最终环境变量如下

```bash
PATH=$PATH:/root/android_sdk/cmdline-tools/latest/bin
PATH=$PATH:/root/android_sdk/cmake/3.18.1/bin
export ANDROID_SDK_ROOT=/root/android_sdk
```

## 3. dockerfile

在没有设置代理的情况下这个dockerfile的环境可以编译通过[android](https://github.com/android)/[ndk-samples](https://github.com/android/ndk-samples)项目所有工程，如果需要设置代理，需要考虑sdkmanager的代理以及gradle中的仓库代理，这里不做过多展开，仅给出部分sdkmanager参数

| 选项                                      | 说明                                                         |
| :---------------------------------------- | :----------------------------------------------------------- |
| `--sdk_root=path`                         | 使用指定的 SDK 路径而不是包含此工具的 SDK                    |
| `--channel=channel_id`                    | 纳入从 channel_0 到 channel_id（含）的所有渠道中的软件包。可用的渠道包括：`0`（稳定版）、`1`（Beta 版）、`2`（开发版）和 `3`（Canary 版）。 |
| `--include_obsolete`                      | 在列出或更新软件包时纳入已过时的软件包。 仅适用于 `--list` 和 `--update`。 |
| `--no_https`                              | 强制所有连接使用 HTTP 而不是 HTTPS。                         |
| `--verbose`                               | 详细输出模式。该模式会输出错误、警告和参考性消息。           |
| `--proxy={http | socks}`                  | 通过给定类型的代理建立连接：用 `http` 指定一个高层级协议（如 HTTP 或 FTP）的代理，或用 `socks` 指定一个 SOCKS（V4 或 V5）代理。 |
| `--proxy_host={IP_address | DNS_address}` | 要使用的代理的 IP 或 DNS 地址。                              |
| `--proxy_port=port_number`                | 要连接到的代理端口号。                                       |

`ARG DEBIAN_FRONTEND=noninteractive`避免`apt update/install`时弹出选择框

```dockerfile
FROM ubuntu
ARG DOWNLOAD_URL=https://dl.google.com/android/repository/commandlinetools-linux-7583922_latest.zip
ARG SDK_PATH=/root/android_sdk
ARG CMAKE_VERSION=3.18.1
ARG DEBIAN_FRONTEND=noninteractive

ENV PATH=$PATH:$SDK_PATH/cmdline-tools/latest/bin
ENV PATH=$PATH:$SDK_PATH/cmake/$CMAKE_VERSION/bin
ENV ANDROID_SDK_ROOT=$SDK_PATH

RUN apt update -y \
    && apt install -y openjdk-8-jdk unzip wget git \
    && wget $DOWNLOAD_URL -O cmd.zip \
    && unzip cmd.zip -d $SDK_PATH \
    && rm cmd.zip \
    && cd $SDK_PATH \
    && mv cmdline-tools latest; mkdir cmdline-tools; mv latest cmdline-tools \
    && yes | sdkmanager --licenses \
    && sdkmanager "cmake;$CMAKE_VERSION"
```

## 参考

1. [sdkmanager](https://developer.android.com/studio/command-line/sdkmanager)
2. [Command line tools only](https://developer.android.com/studio)
3. [android](https://github.com/android)/[ndk-samples](https://github.com/android/ndk-samples)
4. [ubuntu下优雅的切换JDK版本](https://zhuanlan.zhihu.com/p/25896283)

