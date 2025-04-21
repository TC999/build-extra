# 为 Git for Windows 构建环境

此 build-extra 仓库是 Git for Windows SDK 的核心部分，即 [Git for Windows](https://gitforwindows.org/) 的构建环境。任何问题应通过项目的主要 [Git 分支](https://github.com/git-for-windows/git) 上报。

安装 Git for Windows SDK 的最简单方式是通过 [Git SDK 安装程序](https://github.com/git-for-windows/build-extra/releases/latest)。该安装程序会克隆我们的[仓库](http://github.com/git-for-windows/)，包括构建 Git for Windows 所需的所有必要组件，并进行初始构建。同时会在桌面上安装一个指向 Git SDK Bash 的快捷方式。

要在 Git SDK 中检出 `build-extra` 项目，请在 Git SDK Bash 中执行以下命令：

```sh
cd /usr/src/build-extra
git fetch
git checkout main
```

# Git for Windows SDK 的组件

构建环境集成了构建 Git for Windows 安装程序或便携版 Git for Windows（"便携版" == "USB 驱动器版"，即无需安装即可从解压位置直接运行）所需的所有必要部分。

## Git for Windows

Git for Windows 最重要的部分显然是 [Git](https://git-scm.com/)。Git for Windows 项目维护了上游 [Git 项目](https://github.com/git/git) 的[友好分支](https://github.com/git-for-windows/git)。其理念是：Git for Windows 仓库作为测试平台，用于开发特定于 Windows 移植的补丁和补丁系列；一旦补丁稳定后，将[提交至上游](https://github.com/git-for-windows/git/tree/main/Documentation/SubmittingPatches)。

## MSYS2

Git 并非单一可执行文件，而是由多个 C 语言编写的可执行文件、Bash 脚本、Perl 脚本和 Tcl/Tk 脚本组成。部分组件（尚未被 Git for Windows 支持）仍使用其他脚本语言编写。

为支持这些脚本，Git for Windows 使用 [MSYS2](https://msys2.github.io/)。该项目提供了：
- 最小化的 POSIX 模拟层（基于 [Cygwin](https://cygwin.com)）
- 包管理系统（名为 "Pacman"，借鉴自 Arch Linux）
- 由活跃维护团队保持更新的众多软件包，包括 Bash、Perl、Subversion 等。

### MSYS2 与 MinGW 的区别

- **MSYS2** 指使用 POSIX 模拟层（"msys2 runtime"，衍生自 Cygwin 的 `cygwin1.dll`）的库和程序。由于大部分 POSIX 语义被合理模拟（例如 [`fork()` 函数](http://pubs.opengroup.org/onlinepubs/000095399/functions/fork.html)），从 Unix/Linux 移植库和程序非常容易。Bash 和 Perl 是典型的 MSYS2 程序。
- **MinGW** 指使用 GNU 工具编译但无需 POSIX 语义，直接依赖 Win32 API 和 C 运行时库的库和程序。MinGW 意为 "Minimal GNU for Windows"。例如：cURL（通过 HTTP(S)/(S)FTP 等协议与远程服务器通信的库）、emacs、Inkscape 等。

MSYS2 二进制文件的 POSIX 模拟层虽便利，但有代价：通常 MSYS2 程序比 MinGW 版本（如有对应版本）显著更慢。因此，Git for Windows 项目尽可能提供 MinGW 版本的组件。

### MinGW 软件包

MinGW 软件包从 `MINGW-packages` 仓库构建。在 Git SDK Bash 中通过以下命令初始化：

```sh
cd /usr/src/MINGW-packages
git fetch
git checkout main
```

进入 `/usr/src/MINGW-packages/` 对应子目录后，执行 `makepkg-mingw -s` 即可构建包。

通过安装两个工具链（`pacman -Sy mingw-w64-i686-toolchain mingw-w64-x86_64-toolchain`），可在运行 `makepkg-mingw` 时同时构建 `i686` 和 `x86_64` 架构的 MinGW 包。

### MSYS2 软件包

MSYS2 软件包从 `MSYS2-packages` 仓库构建。在 Git SDK Bash 中通过以下命令初始化：

```sh
cd /usr/src/MSYS2-packages
git fetch
git checkout main
```

要构建 `/usr/src/MSYS2-packages/` 中的包，需双击 Git SDK 顶层目录的 `msys2_shell.bat` 启动专用 shell，切换至 `/usr/src/MSYS2-packages/` 对应子目录后执行 `makepkg -s`。首次构建 MSYS2 包前，需通过 `pacman -Sy base-devel binutils` 安装必备开发包。

## 安装程序生成器

Git for Windows 项目提供三种安装程序：

- **Git for Windows**：面向终端用户。`installer/` 子目录包含生成此安装程序的文件。
- **便携版 Git for Windows**：面向终端用户的 "USB 驱动器版"。该安装程序实为自解压 `.7z` 压缩包，可通过 `portable/` 目录文件生成。
- **Git for Windows SDK**：面向项目贡献者的完整开发环境（包含 Git、Bash、cURL 等，自然包括这三个安装程序）。生成此安装程序的文件位于 `sdk-installer/`。

## 支持脚本/文件

`build-extra` 仓库还包含其他维护 Git for Windows 所需的资源。例如：
- [Git 花园修剪工具](https://github.com/git-for-windows/build-extra/blob/main/shears.sh)：帮助在上游 Git 新版本发布时更新 Git for Windows 源码（"合并变基"）。
