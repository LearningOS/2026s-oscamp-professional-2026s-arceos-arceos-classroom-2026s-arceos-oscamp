# 开源操作系统训练营第三阶段——ArceOS

## 领取春夏季作业仓库

1. 加入 [2026 春夏季训练营](https://opencamp.cn/os2edu/camp/2026spring)，并绑定自己的 GitHub 账号。
2. 点击[领取作业仓库](https://github.com/LearningOS/2026s-enroll/issues/new?template=arceos.yml)，提交申请并接受仓库邀请。
3. 在回复的作业仓库中，按照下方教程完成实验并 push，在 Actions 和训练营网站查看成绩。

已领取过本课程的学员继续使用原作业仓库。

本仓库为 [ArceOS](https://github.com/arceos-org/arceos) 的一个剪裁版本，提供了更多初始化的组件与可用于训练的题目，作为开源操作系统第三阶段的训练题目。



## 目录结构

- arceos/：ArceOS 内核源码，它与上游[主线版本](https://github.com/arceos-org/arceos)有所差距，旨在通过剪裁版本让同学们更好理解代码
- course/：ArceOS 教学资料，配合[第三阶段课程](https://opencamp.cn/os2edu/camp/2026spring/stage/5?tab=video)进行学习
- crates/：ArceOS 所依赖并且由我们手动修改的模块，这里仅包括 kernel_guard 一个手动修改的模块
- scripts/：评测脚本，其中 `total-test.sh` 代表执行所有测试，其他脚本分别执行一个测例
- challenges/：往期内存分配器挑战题的设计与实验资料，见 [挑战说明](challenges/lab1.md)



## 环境配置

以下本地步骤面向 **x86_64 Linux / Ubuntu**；Windows 可在 WSL2 Ubuntu 中进行。Rust 安装原理可参考[环境配置教程](https://rcore-os.cn/arceos-tutorial-book/ch01-02.html)，编译版本使用本仓库 `arceos/rust-toolchain.toml` 中的 `nightly-2024-09-04`。

先通过 [rustup](https://rustup.rs/) 安装 Rust，再在作业仓库根目录准备系统工具：

```sh
sudo apt-get update
sudo apt-get install -y build-essential libclang-dev qemu-system-misc dosfstools mtools pkg-config libssl-dev zlib1g-dev wget xz-utils
```

依次更新包索引并安装编译、QEMU、磁盘镜像及依赖开发工具。

```sh
rustup toolchain install nightly-2024-09-04 --profile minimal --component rust-src --component llvm-tools-preview --component rustfmt --component clippy --target riscv64gc-unknown-none-elf
```

安装课程使用的 Rust、源码与 LLVM 工具，以及 RISC-V 裸机目标。

部分练习还需要 RISC-V musl C 工具链。首次安装时，使用与课程 CI 相同的预编译来源：

```sh
mkdir -p tmp/musl
wget -O tmp/musl/riscv64-linux-musl-cross.tgz https://github.com/arceos-org/setup-musl/releases/download/prebuilt/riscv64-linux-musl-cross.tgz
sudo mkdir -p /opt/musl
sudo tar -xzf tmp/musl/riscv64-linux-musl-cross.tgz -C /opt/musl
```

依次创建下载目录、下载工具链，并解压到 `/opt/musl`。该预编译工具链用于 x86_64 Linux 主机。

在练习终端中让构建工具可见：

```sh
mkdir -p tmp/tools
course_sysroot=$(rustc +nightly-2024-09-04 --print sysroot)
ln -sf "$course_sysroot/lib/rustlib/x86_64-unknown-linux-gnu/bin/llvm-objcopy" tmp/tools/rust-objcopy
ln -sf "$course_sysroot/lib/rustlib/x86_64-unknown-linux-gnu/bin/llvm-objdump" tmp/tools/rust-objdump
export PATH="$PWD/tmp/tools:/opt/musl/riscv64-linux-musl-cross/bin:$PATH"
qemu-system-riscv64 --version
riscv64-linux-musl-gcc --version
```

先在仓库的 `tmp/tools` 中建立课程版本的 LLVM 工具入口，再让当前终端找到它们和 musl 编译器，最后检查 QEMU 与 C 编译器。新开终端后，在仓库根目录重新执行其中的 `export PATH=...` 即可。

本期六项练习使用 RISC-V。学习其他架构时可继续参考 [ArceOS 上游构建说明](https://github.com/arceos-org/arceos) 配置 x86_64 或 AArch64 的工具链。

## 评测方式

### ArceOS 训练题

在`main`分支根目录下执行：

```shell
mkdir -p tmp
cd arceos
cargo update -p indexmap --precise 2.6.0
cd ..
./scripts/total-test.sh > tmp/local-test.log
cat tmp/local-test.log
cat test.output
```

首次运行前先按上述命令选择与春夏季 CI 相同的 `indexmap` 版本，再执行六项测试，查看完整日志与 `test.output`。每项测试通过获得 100 分。总测试脚本成功退出并不代表各项全部通过，请同时检查每项结果。

### 挑战题资料

继续阅读 [挑战题说明](challenges/lab1.md)，学习针对应用场景优化内存分配器的方法。该文档记录的是往期独立挑战，邮件、截止日期及 `lab1` 工程属于往期活动。当前作业仓库只提供 `main` 分支六项训练题的自动评测，因此不执行 `git switch lab1` 或 `verify_lab1.sh`。
