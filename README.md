# AMD / Radeon 7900XTX 6900XT GPU ROCm install / setup / config 
# Ubuntu 22.04 / 23.04  
# Conda / PyTorch (ROCm) / EXLLAMA / BitsAndBytes-ROCm
# Automatic1111 Stable Diffusion + ComfyUI 
# Oobabooga - Text Generation WebUI

## Install notes / instructions ##

 I have composed this collection of instructions as they are my notes.
 I use to setup my own Linux system with AMD parts.
 I've gone over these doing many re-installs to get them all right.
 This is what I had hoped to find when I had search for install instructions -
 so I'm sharing them in the hopes that they save time for other people. 
 There may be in here extra parts that aren't needed but this works for me.
 Originally text, with comments like a shell script that I cut and paste - 2023-07 - nktice

---


# Ubuntu 22.04 / 23.04 - Base system install 
Ubuntu 22.04 works great on Radeon 6900 XT video cards, 
but does not support 7900XTX cards as they came out later 
Ubuntu 23.04 is newer but has issues with some of the tools. 
So the notes below should work on either system, unless commented.

At this point we assume you've done the system install
and you know what that is, have a user, root, etc. 

```bash
# update system packages 
sudo apt update -y && sudo apt upgrade -y 
```

```bash 
#turn on devel and sources.
sudo apt-add-repository -y -s -s
sudo apt install -y "linux-headers-$(uname -r)" \
	"linux-modules-extra-$(uname -r)"
```

#### [ for Ubuntu 23.04 - lunar ]
some things may require older versions of python, so we need to add
jammy packages, so that they can be installed, on lunar systems.
```bash
sudo add-apt-repository -y -s deb http://security.ubuntu.com/ubuntu jammy main universe
```

## Add AMD GPU package sources 
Make the directory if it doesn't exist yet.
This location is recommended by the distribution maintainers.

```bash
sudo mkdir --parents --mode=0755 /etc/apt/keyrings
```

Download the key, convert the signing-key to a full
keyring required by apt and store in the keyring directory
```bash
wget https://repo.radeon.com/rocm/rocm.gpg.key -O - | \
    gpg --dearmor | sudo tee /etc/apt/keyrings/rocm.gpg > /dev/null
```
amdgpu repository for jammy
```bash
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/amdgpu/latest/ubuntu jammy main' \
    | sudo tee /etc/apt/sources.list.d/amdgpu.list
sudo apt update -y 
```

AMDGPU DKMS
```bash
sudo apt install -y amdgpu-dkms
```


# ROCm repositories for jammy
https://rocmdocs.amd.com/en/latest/deploy/linux/os-native/install.html
whereas 5.4.2 is the stable version supported by pytorch let's us that...
note : bitsandbytes-rocm requires 5.4.2
```bash
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/5.4.2 jammy main" \
    | sudo tee --append /etc/apt/sources.list.d/rocm.list
echo -e 'Package: *\nPin: release o=repo.radeon.com\nPin-Priority: 600' \
    | sudo tee /etc/apt/preferences.d/rocm-pin-600
sudo apt update -y
```

# More AMD ROCm related packages 
This is lots of stuff, but comparatively small so worth including,
as some stuff later may want as dependencies without much notice.
```bash
# ROCm...
sudo apt install -y rocm-dev rocm-libs rocm-hip-sdk rock-dkms rocm-libs \ 
	hipsparse hipblas hipblas-dev rocblas rocblas-dev rccl rocthrust \
	hipcub roctracer-dev rocm-opencl rocm-opencl-dev
```

```bash
# ld.so.conf update 
sudo tee --append /etc/ld.so.conf.d/rocm.conf <<EOF
/opt/rocm/lib
/opt/rocm/lib64
EOF
sudo ldconfig
```

```bash
# update path
echo "PATH=/opt/rocm/bin:/opt/rocm/opencl/bin:$PATH" >> ~/.profile
```

## Find graphics device
```bash
sudo /opt/rocm/bin/rocminfo | grep gfx
```
Found : gfx1030 [ Radeon 6900 ]
Found : gfx1100 [ Radeon 7900 ] 

## Add user to groups
Of course note to change the user name to match your user. 
```bash
sudo adduser n video
sudo adduser n render
```

```bash
# git and git-lfs (large file support
sudo apt install -y git git-lfs
# development tool may be required later...
sudo apt install -y libstdc++-12-dev
# stable diffusion likes TCMalloc...
sudo apt install -y libtcmalloc-minimal4
```

## Performance Tuning 
You can skip this section...
Here are instructions for turning on control of GPU clocks..
https://www.reddit.com/r/linux_gaming/comments/xm2goe/help_with_overclocking_rx_6600_xt_on_unmodified/
* Step 1: (sudo) Open this file with an editor - [sub for editor of choice...]
  ```bash
  sudo vi /etc/default/grub
  ``` 
* Step 2: Add this Kernel parameter
  ```
  amdgpu.ppfeaturemask=0xffffffff
  ```
  to GRUB_CMDLINE_LINUX_DEFAULT 
* Step 3: Save and run this command install it if you don't have it -
  ```bash
  sudo grub-mkconfig -o /boot/grub/grub.cfg
  ```


### corectrl 
- for controlling settings...
 https://gitlab.com/corectrl/corectrl
 apparently it's not in the mainline packages, 
 so we need to add a PPA for it... this of course is known
 to have disasterous consequences if done the wrong way.
 so the instructions for how to do this are broke.
 thus we'll do them here. You have been warned...
 more details here : https://gitlab.com/corectrl/corectrl/-/wikis/Setup

```bash
# Protection... avoid other packages from this repo.
sudo tee --append /etc/apt/preferences.d/corectl <<EOF
# Never prefer packages from the ernstp repository
Package: *
Pin: release o=LP-PPA-ernstp-mesarc
Pin-Priority: 1

# Allow upgrading only corectrl from LP-PPA-ernstp-mesarc
Package: corectrl
Pin: release o=LP-PPA-ernstp-mesarc
Pin-Priority: 500
EOF
```

#### Add PPA - https://launchpad.net/~ernstp/+archive/ubuntu/mesarc 
```bash
sudo add-apt-repository -y ppa:ernstp/mesarc
sudo apt update -y
```
#### install corectrl
```bash
sudo apt install -y corectrl
```
#### end corectrl
### End performance tuning 


## Top for video memory and usage 
```bash
sudo apt install -y nvtop 
```

## Radeon specific tools...
```bash
sudo apt install -y radeontop rovclock
```

## Environmental variables...
```bash
echo "HIP_PATH=/opt/rocm/" >> ~/.profile
echo "HIP_PLATFORM='amd'" >> ~/.profile
echo "USE_ROCM=1" >> ~/.profile
echo "BUILD_CUDA_EXT=0" >> ~/.profile
# Radeon 6900
#echo "PYTORCH_ROCM_ARCH=gfx1030" >> ~/.profile 
# Radeon 7900
echo "PYTORCH_ROCM_ARCH=gfx1100" >> ~/.profile 
```


## and now we reboot...
```bash
sudo reboot
```

## End of OS / base setup

---

# Conda 
- required for pytorch
Conda provides virtual environments for python, so that programs
with different dependencies can have different environments.
Here is more info on managing conda :
https://docs.conda.io/projects/conda/en/latest/user-guide/getting-started.html#
Other notes :
https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html
Download info :
https://www.anaconda.com/download/

```bash
cd ~/Downloads/
#wget https://repo.anaconda.com/archive/Anaconda3-2023.03-1-Linux-x86_64.sh
wget https://repo.anaconda.com/archive/Anaconda3-2023.07-1-Linux-x86_64.sh
```

```bash
#  Note versions may have changed... 
bash Anaconda3-2023.07-1-Linux-x86_64.sh -b
#bash Anaconda3-2023.03-1-Linux-x86_64.sh -b 
#bash Anaconda-latest-Linux-x86_64.sh
```

```bash
cd ~
ln -s anaconda3 conda

echo "PATH=~/conda/bin:$PATH" >> ~/.profile
source ~/.profile 
conda update -y -n base -c defaults conda
```

```bash
conda install -y cmake ninja
```

```bash
conda init
source ~/.profile 
```
## conda is now active...

## install pip
```bash
sudo apt install -y pip
pip3 install --upgrade pip
```

### useful pip stuff to know ... 
```bash
## show outdated packages...
#pip list --outdated
## check dependencies 
#pip check
## install specified bersion 
#pip install <packagename>==<version>
```

##
## End conda and pip setup.
##

---

# Stable Diffusion WebUI / Automatic1111
For [ Radeon 6900 / 7900 Ubuntu 23.04 / ROCm / AMD ]
Website: https://github.com/AUTOMATIC1111/stable-diffusion-webui


## Stable Diffusion likes TCMalloc... 
 [ mentioned earlier as fresh install may require reboot ] 
```bash
sudo apt install -y libtcmalloc-minimal4
```

## image related packages that may come in handy...
```bash
sudo apt install -y imagemagick ffmpeg
```
## SD likes this...
```bash
sudo apt install -y libtcmalloc-minimal4
```

## Switch to conda environment... and freshen pip
```bash
conda create -y -n sd python=3.10.6
conda activate sd

sudo apt install -y pip
pip3 install --upgrade pip
```

# PyTorch repo related parts :
note : we're getting things from the nightly repos as the stable doesn't support 7900XTX. 
6900XT works fine with stable - 5.4.2 on Ubuntu 22.04

```bash
# pre-install 
pip install cmake colorama filelock lit numpy Pillow Jinja2 \
	mpmath fsspec MarkupSafe certifi filelock networkx \
	sympy packaging requests \
         --index-url https://download.pytorch.org/whl/nightly/rocm5.5
#         --index-url https://download.pytorch.org/whl/rocm5.4.2
```

```bash
# install
pip install torch torchvision torchtext torchaudio torchdata \
	pytorch-triton pytorch-triton-rocm \
         --index-url https://download.pytorch.org/whl/nightly/rocm5.5
#       --index-url https://download.pytorch.org/whl/rocm5.4.2
```

## rocm related...
```bash
#pip install jaxlib-rocm
pip install gensim tables -U 
pip install tensorflow-rocm
pip install cupy-rocm-4-3 cupy-rocm-5-0
```
## other packages
```bash
pip install accelerate -U
```

## other parts
```bash
pip install onnx
pip install super-gradients
```

## Get the files...
```bash
cd
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
cd stable-diffusion-webui
```

## Install requisites...
```bash
pip install -r requirements.txt --extra-index-url https://download.pytorch.org/whl/nightly/rocm5.5
```

## Edit environment settings...
```bash
tee --append webui-user.sh <<EOF
 ## Torch for ROCm
 export TORCH_COMMAND="pip install torch torchvision --index-url https://download.pytorch.org/whl/nightly/rocm5.5"
 ## And if you want to call this from other programs...
 export COMMANDLINE_ARGS="--api"
EOF
```

## If you keep models for SD somewhere, this is where you'd like them up...
If you don't do this, it will install a default to get you going. 
Note that these start files do include things that it needs you'll want to copy
into the folder where you have other models ( to avoid issues ) 
```bash
#mv models models.1
#ln -s /path/to/models models 
```

## Run SD...
 Note that the first time it starts it may take it a while to go and get things
 it's not always good about saying what it's up to. 
```bash
./webui.sh 
```

## End - Stable Diffusion

## ComfyUI# ComfyUI
https://github.com/comfyanonymous/ComfyUI

## environment...

```bash
## If you want a new one...
#conda create -n comfy -y
#conda activate comfy
## Reuse one from sd...
conda activate sd
```

## git files...
```bash
cd
hgit clone https://github.com/comfyanonymous/ComfyUI
cd ComfyUI
```

## packages
```bash
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/rocm5.5 -r requirements.txt
```

## setup config...
```bash
cp extra_model_paths.yaml.example extra_model_paths.yaml
## set path in file
vi extra_model_paths.yaml
"base_path: ~/stable-diffusion/"
```

# prep models
```bash
mv models models.1
ln -s /path/to/models/ models
# a1111 uses different name for folder with models... link
mv models/checkpoints models/checkpoints.1
ln -s models/Stable-diffusion models/checkpoints
```

### call program...
```bash
# conda activate 
python main.py
```

### ComfyUI references...
Intro info : https://www.youtube.com/watch?v=AbB33AxrcZo

## End ComfyUI


---

#  Oobabooga - Text Generation WebUI - ROCm 
https://github.com/oobabooga/text-generation-webui.git

## setup conda and freshen pip
```bash
# make env
conda create -y -n textgen python=3.10.9
conda activate textgen

#update pip
sudo apt install -y pip
pip3 install --upgrade pip
```


Note : As of writing this the most recxent supported version is ROCm 5.4.2
 to see packages available : https://download.pytorch.org/whl/rocm5.4.2/
 nightly : https://download.pytorch.org/whl/nightly/rocm5.5/
 7900XTX requires nightly builds...

```bash
# pre-install
pip install cmake colorama filelock lit numpy \
         --index-url https://download.pytorch.org/whl/nightly/rocm5.5
#         --index-url https://download.pytorch.org/whl/rocm5.4.2
```

```bash
# main pytorch parts...
pip install torch torchvision torchtext torchaudio torchdata pytorch-triton-rocm \
         --index-url https://download.pytorch.org/whl/nightly/rocm5.5
#       --index-url https://download.pytorch.org/whl/rocm5.4.2
```

## bitsandbytes rocm
 video guide : https://www.youtube.com/watch?v=2cPsvwONnL8
 https://git.ecker.tech/mrq/bitsandbytes-rocm
 https://github.com/0cc4m/bitsandbytes-rocm
```bash
cd
git clone https://git.ecker.tech/mrq/bitsandbytes-rocm.git
cd bitsandbytes-rocm/
```

```bash
BUILD_CUDA_EXT=0 pip install -r requirements.txt --extra-index-url https://download.pytorch.org/whl/nightly/rocm5.5
```

```bash
make hip
```

```bash
#CUDA_VERSION=gfx1030 python setup.py install
CUDA_VERSION=gfx1100 python setup.py install
``` 

### end bitsandbytes-rocm

## Packages for text-generation UI
various changes have made the default requirements.txt and dependencies make a mess
so we're going to go and get those parts manually / independently. 

### Base pieces to get the UI working 
```bash
pip install gradio psutil markdown transformers accelerate datasets peft
```

### part for the model loaders 
```bash
pip install gensim tables blosc2 cython
pip install spyder python-lsp-black 
pip install jaxlib-rocm
# rocm version isn't up to date... make jax match the rocm version...
pip install jax==0.4.6
```

```bash
pip install tensorflow-rocm
pip install cupy-rocm-4-3 cupy-rocm-5-0
pip install loader
pip install logger
pip install -U scikit-learn
```


##
## Oobabooga install / text-generation-webui
##
###### Get files...
```bash
cd
git clone https://github.com/oobabooga/text-generation-webui.git
cd text-generation-webui
```

### Get requirements 

### exllama : https://github.com/turboderp/exllama
Fastest module loadter... 
 How-to : https://www.youtube.com/watch?v=gicpKGo1hW0
 more info : https://www.reddit.com/r/LocalLLaMA/comments/14btvqs/7900xtx_linux_exllama_gptq/
```bash
cd ~/text-generation-webui
mkdir repositories
cd ~/text-generation-webui/repositories
git clone https://github.com/turboderp/exllama
```

```bash
cd exllama
pip install -r requirements.txt --extra-index-url https://download.pytorch.org/whl/nightly/rocm5.5
```

```bash
### exllama does have some features, like it's own webui ...
### we'll not use them here, but here's a few clues.
# python test_benchmark_inference.py -d <path_to_model_files> -p -ppl
#pip install -r requirements-web.txt
# python webui/app.py -d <path_to_model_files>
#pip install flask waitress
```

### image interaction related packages...
```bash
pip install bs4
```

### Models 
They have to come from /somewhere/ so, this is where you'd 
link pre-stored models into the models
If you don't have some, there are tools to download them, 
they're often changing so check : huggingface.co 
```bash
# mv models models.1
# ln -s /path/to/models models
```

### quick script to call program... 
Made some changes here as Oobabooga has changed some parameters. 
```bash
tee --append run.sh <<EOF
#!/bin/bash
## activate conda
conda activate textgen
## command to run server... 
python server.py --listen --loader=exllama  \
  --auto-devices --extensions sd_api_pictures send_pictures gallery 
conda deactivate
EOF
chmod u+x run.sh
```

Note we haven't called Oobabooga's requirements.txt - it is out of date, and will break things. 

## running the server...
note that the script must be run via `source` rather than normal execution.
```bash
source run.sh
```

## End - Oobabooga - Text-Generation-WebUI


