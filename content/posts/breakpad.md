---
title: "Breakpad"
date: 2022-01-10T10:21:00+08:00
tags: ["breakpad", "wsl"]
categories: ["tools"]
series: [""]
summary: "Breakpad是一个库和工具套件可以让你发布的应用程序（把编译器提供的调试信息剥离掉的）给用户，记录了崩溃紧凑的“dump”文件，发送回您的服务器，并从这些minidump产生C和C++堆栈踪迹。"
draft: false
editPost:
  URL: "https://github.com/Zhoutao822/zhoutao822.github.io/tree/main/content/"
  Text: "Suggest Changes"
  appendFilePath: true 
---

[Google Breakpad](https://links.jianshu.com/go?to=https%3A%2F%2Fchromium.googlesource.com%2Fbreakpad%2Fbreakpad)是一套完整的工具集，从crash的捕获到crash的dump，都提供了相对应的工具。

## 1. 编译与安装

项目中要用到breakpad编译后的两个可执行文件`src/processor/minidump_stackwalk`和`src/tools/linux/dump_syms/dump_syms`，前者其实Android Studio已经提供了Windows版的，在`Android Studio/bin/lldb/bin`下，但是`dump_syms`只能从源码编译得到，而且最好是在Linux环境下编译及使用，所以可以使用WSL进行，WSL在Windows Store上下载即可使用，可选Ubuntu-20.04、Ubuntu-18.04等等。

### 1.1  depot_tools安装

depot_tools是用于管理Google开源项目，不仅仅有breakpad，还可以获取Chromium、WebRTC等源码，这里使用depot_tools拉取breakpad的源码，如果是从Github上clone的breakpad源码可能会缺失`linux_syscall_support.h`头文件

```sh
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

添加环境变量并`source ~/.bashrc`，如果用的是zsh，同理修改`.zshrc`

```sh
export PATH=/path/to/depot_tools:$PATH
```

### 1.2 breakpad编译与安装

首先需要安装python 2，否则`fetch breakpad`会失败；并且安装基础编译工具

```shell
sudo apt-get update
sudo apt install python

```

按照Github指示一步一步执行即可：

```shell
mkdir breakpad && cd breakpad

fetch breakpad
cd src

./configure && make

sudo make install
```

其中`fetch breakpad`可能需要10分钟以上，成功的结果

```shell
tao@LTSZ-TaoZhou:~/breakpad$ fetch breakpad
Running: gclient root
Running: gclient config --spec 'solutions = [
  {
    "name": "src",
    "url": "https://chromium.googlesource.com/breakpad/breakpad.git",
    "managed": False,
    "custom_deps": {},
  },
]
'
Running: gclient sync

________ running 'git -c core.deltaBaseCacheLimit=2g clone --no-checkout --progress https://chromium.googlesource.com/breakpad/breakpad.git /home/tao/breakpad/_gclient_src_a_m392sy' in '/home/tao/breakpad'
Cloning into '/home/tao/breakpad/_gclient_src_a_m392sy'...
remote: Sending approximately 40.06 MiB ...
remote: Total 19092 (delta 14865), reused 19092 (delta 14865)
Receiving objects: 100% (19092/19092), 40.06 MiB | 4.63 MiB/s, done.
Resolving deltas: 100% (14865/14865), done.
Syncing projects:  80% ( 4/ 5) src/src/testing
[0:01:44] Still working on:
[0:01:44]   src/src/third_party/protobuf/protobuf

[0:01:54] Still working on:
[0:01:54]   src/src/third_party/protobuf/protobuf

[0:02:04] Still working on:
[0:02:04]   src/src/third_party/protobuf/protobuf

[0:02:14] Still working on:
[0:02:14]   src/src/third_party/protobuf/protobuf

[0:02:24] Still working on:
[0:02:24]   src/src/third_party/protobuf/protobuf

[0:02:34] Still working on:
[0:02:34]   src/src/third_party/protobuf/protobuf

[0:02:44] Still working on:
[0:02:44]   src/src/third_party/protobuf/protobuf

[0:02:54] Still working on:
[0:02:54]   src/src/third_party/protobuf/protobuf

[0:02:55] Still working on:
[0:02:55]   src/src/third_party/protobuf/protobuf
Syncing projects: 100% (5/5), done.
Running hooks: 100% (3/3), done.
Running: git submodule foreach 'git config -f $toplevel/.git/config submodule.$name.ignore all'
Running: git config --add remote.origin.fetch '+refs/tags/*:refs/tags/*'
Running: git config diff.ignoreSubmodules all
```

`./configure && make`结果，没有显示error即可，最后`sudo make install`，install需要root权限

```shell
g++ -I./src/third_party/mac_headers  -DHAVE_MACH_O_NLIST_H -g -O2   -o src/tools/mac/dump_syms/dump_syms_mac src/common/src_tools_mac_dump_syms_dump_syms_mac-dwarf_cfi_to_module.o src/common/src_tools_mac_dump_syms_dump_syms_mac-dwarf_cu_to_module.o src/common/src_tools_mac_dump_syms_dump_syms_mac-dwarf_line_to_module.o src/common/src_tools_mac_dump_syms_dump_syms_mac-dwarf_range_list_handler.o src/common/src_tools_mac_dump_syms_dump_syms_mac-language.o src/common/src_tools_mac_dump_syms_dump_syms_mac-md5.o src/common/src_tools_mac_dump_syms_dump_syms_mac-module.o src/common/src_tools_mac_dump_syms_dump_syms_mac-path_helper.o src/common/src_tools_mac_dump_syms_dump_syms_mac-stabs_reader.o src/common/src_tools_mac_dump_syms_dump_syms_mac-stabs_to_module.o src/common/dwarf/src_tools_mac_dump_syms_dump_syms_mac-bytereader.o src/common/dwarf/src_tools_mac_dump_syms_dump_syms_mac-dwarf2diehandler.o src/common/dwarf/src_tools_mac_dump_syms_dump_syms_mac-dwarf2reader.o src/common/dwarf/src_tools_mac_dump_syms_dump_syms_mac-elf_reader.o src/common/mac/src_tools_mac_dump_syms_dump_syms_mac-arch_utilities.o src/common/mac/src_tools_mac_dump_syms_dump_syms_mac-dump_syms.o src/common/mac/src_tools_mac_dump_syms_dump_syms_mac-file_id.o src/common/mac/src_tools_mac_dump_syms_dump_syms_mac-macho_id.o src/common/mac/src_tools_mac_dump_syms_dump_syms_mac-macho_reader.o src/common/mac/src_tools_mac_dump_syms_dump_syms_mac-macho_utilities.o src/common/mac/src_tools_mac_dump_syms_dump_syms_mac-macho_walker.o src/tools/mac/dump_syms/src_tools_mac_dump_syms_dump_syms_mac-dump_syms_tool.o
```

最后测试一下

```shell
tao@LTSZ-TaoZhou:~$ dump_syms -V
2.4 -V
Usage: dump_syms [OPTION] <binary-with-debugging-info> [directories-for-debug-file]

Options:
  -i:         Output module header information only.
  -c          Do not generate CFI section
  -r          Do not handle inter-compilation unit references
  -v          Print all warnings to stderr
  -n <name>   Use specified name for name of the object
  -o <os>     Use specified name for the operating system
```

```shell
tao@LTSZ-TaoZhou:~$ minidump_stackwalk -V
minidump_stackwalk: invalid option -- 'V'
Usage: minidump_stackwalk [options] <minidump-file> [symbol-path ...]

Output a stack trace for the provided minidump

Options:

  -m         Output in machine-readable format
  -s         Output stack contents
```

## 参考：

1. [Breakpad Github](https://github.com/google/breakpad)
2. [depot_tools_tutorial](https://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up)