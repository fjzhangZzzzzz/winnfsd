# WinNFSd

[![Build status](https://ci.appveyor.com/api/projects/status/github/winnfsd/winnfsd?svg=true)](https://ci.appveyor.com/project/MarcHarding/winnfsd-6y1xi/branch/master)

## Fork 与维护说明

本仓库为 [`winnfsd/winnfsd`](https://github.com/winnfsd/winnfsd) 的 fork，原仓库长期未维护，本 fork 主要目标是：

- 修复社区反馈的已知问题（例如 Linux 新内核下的 NFSv3 兼容性问题）；
- 保持与上游协议行为兼容，尽量不引入破坏性变更；
- 提供持续可用的 Windows 可执行文件（通过 GitHub Actions 自动构建并发布）。

## 已修复问题一览（节选）

| Issue                                                                                         | 现象                                                                                                                                 | 原因                                                                                                                                                                                     | 修复方式                                                                                                                       |
|----------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [#101 NFSv3 READ data length 错误](https://github.com/winnfsd/winnfsd/issues/101)            | 在 Linux 5.19+ 等新内核上，通过 NFSv3 挂载后 `cat` 小文件会报 `Input/output error`，日志中可见 `READ count doesn't match length of opaque`。 | READ 响应中，返回的 `count` 使用的是实际读取字节数，但 `opaque` 的 length 字段仍然是请求的大小（例如 4096），导致 NFS 客户端校验失败。                    | 在 `CNFS3Prog::ProcedureREAD` 中，`fread` 返回后使用实际读取的字节数再次调用 `data.SetSize(count)`，保证 `opaque` 长度与 `count` 一致。 |

后续新增修复会持续补充到本列表，必要时会拆分到独立的 `CHANGELOG.md` 中。

## 构建与下载

- **本地构建（示例）**：在安装了 Visual Studio（含 C++ 工具链）的环境中：
  - 使用 IDE 打开 `src/WinNFSd.sln`，选择 `Release` 配置后编译；或
  - 在命令行中执行：`msbuild src\WinNFSd.sln /p:Configuration=Release`。
- **预编译二进制**：在 GitHub Releases 页面可以下载由 GitHub Actions 自动构建的 `WinNFSd.exe`。tag 推送后会自动生成或更新对应版本的 Release 并上传 exe。

## CI 与测试

- 现有的 AppVeyor 配置会在 Windows 上通过 VirtualBox + Vagrant 运行 connectathon 测试集，对 NFS 行为做较完整验证（见 `appveyor.yml` 与 `tests/bash` 脚本）。
- 新增的 GitHub Actions workflow（`.github/workflows/build-and-release.yml`）主要负责：在 `windows-latest` 上编译 `WinNFSd.sln`、在每次 push/PR 校验能否成功构建，并在打 tag（`v*`）时自动创建 Release 并上传 `WinNFSd.exe`。\
  中短期内两套 CI 将并存：AppVeyor 偏向回归测试，GitHub Actions 负责快速构建与发布。

## 维护流程约定

- 每个 bug / 功能变更尽量对应一个 GitHub issue 与一个 PR，在 PR 描述中简要说明问题现象与修复思路。
- 合并后，将该问题的**现象 / 原因 / 修复方式**以 1–3 句的形式追加到“已修复问题一览”或单独的 `CHANGELOG.md` 中，便于后续回溯。
- 采用打 tag + GitHub Actions 的方式发布版本：
  - 在主分支合并稳定后打出语义化版本 tag（如 `v2.4.1`）；
  - GitHub Actions 自动构建并将 `WinNFSd.exe` 上传到对应 Release；
  - 如有需要，可在 Release 说明中附上该版本涉及的 issue/PR 列表。

## 原项目说明

Introduction
------------
* Fork of WinNFSd_edited by ZeWarden(http://github.com/ZeWaren/WinNFSd_edited), based on WinNFSd by vincentgao (http://sourceforge.net/projects/winnfsd/).
* License: GPL.
* Runs on all major versions of Windows.

Description
--------------------
WinNFSd is a Network File System V3 (NFS) server for Windows.

You can use any NFS client to mount a directory of Windows and read/write files via NFS v3 protocol. It is useful when you usually access files of Windows on Linux and for especially for virtual machines, since it is much faster than shared folders.

You can also export any folder with an additional alias.

The export of multiple folders is also possible. Just put the shared foldes and an optional alias into a simple text file:

```
C:\path\to\a\mount > /alias
C:\path\to\another\mount > /another-alias
```

Then start winnfsd.exe like this:
`WinNFSd.exe -pathFile C:\path\to\your\pathfile`


Usage
-------------------
```
=====================================================
WinNFSd 2.2.0
Network File System server for Windows
Copyright (C) 2005 Ming-Yang Kao
Edited in 2011 by ZeWaren
Edited in 2013 by Alexander Schneider (Jankowfsky AG)
Edited in 2014 2015 by Yann Schepens
Edited in 2016 by Peter Philipp (Cando Image GmbH), Marc Harding
=====================================================

Usage: WinNFSd.exe [-id <uid> <gid>] [-log on | off] [-pathFile <file>] [-addr <ip>] [export path] [alias path]

At least a file or a path is needed
For example:
On Windows> WinNFSd.exe d:\work
On Linux> mount -t nfs 192.168.12.34:/d/work mount

For another example:
On Windows> WinNFSd.exe d:\work /exports
On Linux> mount -t nfs 192.168.12.34:/exports

Another example where WinNFSd is only bound to a specific interface:
On Windows> WinNFSd.exe -addr 192.168.12.34 d:\work /exports
On Linux> mount - t nfs 192.168.12.34: / exports

Use "." to export the current directory (works also for -filePath):
On Windows> WinNFSd.exe . /exports
```
