---
title: "Stable Diffusion 使用 AMD ROCm 显卡加速 "
date: 2023-03-20T12:12:15+08:00
draft: false
toc: false
images:
tags: 
  - pytorch
  - stable diffusion
  - AI
slug: stable-diffusion-amd-gpu-rocm-ubuntu
---

<https://www.reddit.com/r/StableDiffusion/comments/zu9w40/novices_guide_to_automatic1111_on_linux_with_amd/>

<https://bbs.ruliweb.com/userboard/board/700315/read/3180>

<https://github.com/XiaozhouTAT/stable-diffusion-webui-use-amd-gpu>

## Ubuntu
- 本篇文章使用 Ubuntu 22.04 LTS，理论上 Ubuntu 20.04 LTS 也可以，[下载地址](https://ubuntu.com/download/desktop)。
- 使用 Linux 和 Python 难度最大的不是安装，而是依赖问题，请做好心理准备。
- Ubuntu 可以安装在电脑上任意分区，会自动添加引导到 Windows 的引导上。
- RX6700XT 按照 [Tom's Hardware](https://www.tomshardware.com/news/stable-diffusion-gpu-benchmarks)文章的测试参数，跑512x512, 100it 约16秒，6.25it/s.

### Windows?
- 由于 ROCm 版本的 Pytorch 不支持 Windows，你一定要用 Windows 请使用下面的方法，并做好性能减半，或者完全不能用的准备。
- https://github.com/lshqqytiger/stable-diffusion-webui-directml
- https://github.com/nod-ai/SHARK/

### 显卡是否支持
- [官方ROCm支持列表](https://docs.amd.com/bundle/Hardware_and_Software_Reference_Guide/page/Hardware_and_Software_Support.html)支持 GFX9, CDNA, RDNA显卡。
    - https://llvm.org/docs/AMDGPUUsage.html#processors
    - https://github.com/RadeonOpenCompute/rocminfo/blob/master/rocm_agent_enumerator
> 以上是官方支持的显卡，同架构的显卡可以添加环境变量来支持。

### RX580
- 网友说[官方ROCm](https://docs.amd.com/bundle/Hardware_and_Software_Reference_Guide/page/Hardware_and_Software_Support.html)对 RX580 只支持到 Rocm3.7，之后的版本官方不做可用性认证，导致新版本ROCm支持 RX580 有很多bug,但是 ROCm3.7 只支持Python3.6-3.9
- 幸好网友对官方 ROCm 进行了修改，添加了 RX580 的支持，下面会用到 <https://github.com/XiaozhouTAT/stable-diffusion-webui-use-amd-gpu> 库里的修改版 rocblas, pytorch, torchvision, tensorflow.
> 请注意这个库使用的是 Ubuntu 20.04 LTS 进行的验证。

## 安装Python
```sh
sudo apt install wget git python3 python3-venv python3-pip -y
```
> 就上面的命令也有可能存在依赖问题，祝你好运。

## 安装ROCm
```sh
# https://docs.amd.com/category/Release%20Documentation
# https://www.amd.com/zh-hans/support/graphics/radeon-500-series/radeon-rx-500-series/radeon-rx-580
# https://www.amd.com/zh-hans/support/graphics/amd-radeon-6000-series/amd-radeon-6700-series/amd-radeon-rx-6700-xt
wget https://repo.radeon.com/amdgpu-install/22.40.3/ubuntu/jammy/amdgpu-install_5.4.50403-1_all.deb # Rocm 5.4.3 on Ubuntu 22.04 LTS (jammy)
sudo apt install ./amdgpu-install_5.4.50403-1_all.deb
sudo apt update && sudo apt upgrade -y
sudo amdgpu-install --usecase=rocm --no-dkms
sudo usermod -a -G video,render $LOGNAME
sudo reboot
```
> 上面的包的作用就是 添加 https://repo.radeon.com/ 到你的 Ubuntu 源，并将可执行添加到$PATH，Ubuntu CN 源的速度其实还行，但是 Radeon 的速度看人品了，自己加速器吧。

### 显卡驱动状态
```sh
ls -l /dev/dri/render*  # 看显卡在Linux下可见不可见
watch -n 1 rocm-smi     # 每1秒刷新 rocm-smi 显卡状态
/opt/rocm/bin/rocminfo  # 详细显卡信息
rocminfo | grep 'Name'  # 显卡名称，gfx代号
/opt/rocm/opencl/bin/clinfo # 显示支持 OpenCL 显卡
```

## 安装 PyTorch
- [PyTorch官方安装命令](https://pytorch.org/get-started/locally/) 是 `pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/rocm5.4.2`
- 但是他们的库速度很慢，国内的[mirror](https://mirror.sjtu.edu.cn/pytorch-wheels/)是爬虫抓的，不是官方支持，所有安装兼容性可能有问题，有加速器的尽量使用加速器吧。
> 请注意你安装的 Python 的快捷方式是 `python` `python3` `python3.10` 中的哪一个，还有 `pip` `pip3` 要注意。

```sh
# https://download.pytorch.org/whl/ -> https://mirror.sjtu.edu.cn/pytorch-wheels/
# ALL pytorch version https://download.pytorch.org/whl/torch_stable.html or https://mirror.sjtu.edu.cn/pytorch-wheels/torch_stable.html search ROCm
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
python -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade pip wheel
#pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/rocm5.4.2
pip install torch torchvision torchaudio -f https://mirror.sjtu.edu.cn/pytorch-wheels/rocm5.4.2/torch_stable.html
pip list | grep 'torch'
#pip install torch==2.0.0+rocm5.4.2 torchvision==0.15.1+rocm5.4.2 torchaudio==2.0.1+rocm5.4.2 -f https://mirror.sjtu.edu.cn/pytorch-wheels/rocm5.4.2/torch_stable.html
# 下面命令可能能会帮你解决依赖问题，也可能不会。
pip freeze > req.txt
pip install -r req.txt
```
> 要注意自己下载安装的是否是带有 ROCm 标识的 PyTorch.

## 下载 Stable Diffusion WebUI
```sh
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui
cd stable-diffusion-webui
#pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```
> 从这里开始会下载很多 GitHub 的文件，可能会下载失败，请加速器吧。或者了解 <https://ghproxy.com> 的用法。

### 修改 webui-user.sh
```sh
# 添加进 webui-user.sh
export HSA_OVERRIDE_GFX_VERSION=10.3.0 # gfx1030 Radeon RX 6800 6800 XT 6900 XT, gfx1031 Radeon RX 6700 XT.
#export HSA_OVERRIDE_GFX_VERSION=8.0.3 # gfx803 Radeon RX580
# RX5000 系列需要在 COMMANDLINE_ARGS 添加 --precision full --no-half，RX500 和 RX6000 系列不需要
# --medvram --lowvram --opt-sub-quad-attention --disable-nan-check https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Command-Line-Arguments-and-Settings
export COMMANDLINE_ARGS="--skip-torch-cuda-test --autolaunch --listen"
export TORCH_COMMAND="pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/rocm5.4.2"
#export TORCH_COMMAND="pip install torch torchvision torchaudio -f https://mirror.sjtu.edu.cn/pytorch-wheels/rocm5.4.2/torch_stable.html"
```

### 下载Model
- civitai.com
- huggingface.co
- 推荐下载 fp16 版本
- 将下载的 .safetensors 放到 `stable-diffusion-webui/models/Stable-diffusion` 目录下。

## 运行 webui.sh

```sh
./webui.sh
```
> 在实际运行时，你添加修改某些功能会自动从GitHub下载某些文件，不用加速器，你遇到的问题会很多。

> 请注意，PyTorch 版本必须和 ROCm 版本一致，比如 5.4.x 5.2.x，否则你会在运行某些功能时报 PyTorch 版本不匹配的错误。