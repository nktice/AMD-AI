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

 Updates 2023-08-29 - ROCm 5.6 appears to be functioning now, so this guide is updated to refer to that... older versions may be mentioned in notes.  Oobabooga changed some command line arguments so there's changes there. 

 2023-09-09 - I had a report that this doesn't work in virtual machines (virtualbox) as the system there cannot see the hardware, it can't load drivers, etc.  Windows users may find it more helpful to try DirectML - https://rocm.docs.amd.com/en/latest/deploy/windows/quick_start.html / https://github.com/lshqqytiger/stable-diffusion-webui-directml

2023-09-30 - Added new version for ROCm 5.7 -> https://github.com/nktice/AMD-AI/blob/main/ROCm-5.7.md  

2023-11-30 - Updated for ROCm 5.7.2 and PyTorch from Nightlies, and simplification of process -> https://github.com/nktice/AMD-AI/blob/main/ROCm-5.7.md  


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
Some things may require older versions of python, so we need to add
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
Keyring required by apt and store in the keyring directory
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
Note : This commonly produces warning message about 'Possible missing firmware' these are just wanrings and things work anyway, they can be ignored. 

# ROCm repositories for jammy
https://rocmdocs.amd.com/en/latest/deploy/linux/os-native/install.html
Note : 2023-09-11 Support for newer version of BitsAndBytes(0.41 !) made for 5.6 - Project website : https://github.com/arlo-phoenix/bitsandbytes-rocm-5.6

```bash
#echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/5.4.2 jammy main" \
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/5.6 jammy main" \
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
sudo apt install -y rocm-dev rocm-libs rocm-hip-sdk rocm-dkms rocm-libs
#sudo apt install -y rocm-opencl rocm-opencl-dev
#sudo apt install -y hipsparse hipblas hipblas-dev hipcub
#sudo apt isntall -y rocblas rocblas-dev rccl rocthrust roctracer-dev 
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
sudo adduser `whoami` video
sudo adduser `whoami` render
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
This section is optional, and as such has been moved to [performance-tuning](https://github.com/nktice/AMD-AI/blob/main/performance-tuning.md)

## Top for video memory and usage
nvtop 
Note : I have had issues with the distro version crashes with 2 GPUs, installing new version from sources works fine.  Project website : https://github.com/Syllo/nvtop 
```bash
sudo apt install -y nvtop 
```

## Radeon specific tools...
```bash
sudo apt install -y radeontop rovclock
```

## and now we reboot...
```bash
sudo reboot
```

## End of OS / base setup

---

# Conda 
Required for pytorch... Conda provides virtual environments for python, so that programs
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
#wget https://repo.anaconda.com/archive/Anaconda3-2023.07-1-Linux-x86_64.sh
wget https://repo.anaconda.com/archive/Anaconda3-2023.07-2-Linux-x86_64.sh
```

```bash
#  Note versions may have changed... 
bash Anaconda3-2023.07-2-Linux-x86_64.sh -b
#bash Anaconda3-2023.07-1-Linux-x86_64.sh -b
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

Project Website: https://github.com/AUTOMATIC1111/stable-diffusion-webui


## Stable Diffusion likes TCMalloc... 
 [ mentioned earlier as fresh install may require reboot ] 
```bash
sudo apt install -y libtcmalloc-minimal4
```

## image related packages that may come in handy...
```bash
sudo apt install -y imagemagick ffmpeg
```

## Switch to conda environment... and freshen pip
```bash
conda create -y -n sd python=3.10.6
conda activate sd
```

# PyTorch repo related parts :
note : we're getting things from the nightly repos as the stable doesn't support 7900XTX. 
6900XT works fine with stable - 5.4.2 on Ubuntu 22.04

```bash
# pre-install 
pip install cmake colorama filelock lit numpy Pillow Jinja2 \
	mpmath fsspec MarkupSafe certifi filelock networkx \
	sympy packaging requests \
         --index-url https://download.pytorch.org/whl/nightly/rocm5.6
#         --index-url https://download.pytorch.org/whl/rocm5.4.2
```

```bash
# install
pip install torch torchvision torchtext torchaudio torchdata \
	triton pytorch-triton pytorch-triton-rocm \
         --index-url https://download.pytorch.org/whl/nightly/rocm5.6
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
pip install -r requirements.txt --extra-index-url https://download.pytorch.org/whl/nightly/rocm5.6
```

## Edit environment settings...
```bash
tee --append webui-user.sh <<EOF
 ## Torch for ROCm
 export TORCH_COMMAND="pip install torch torchvision --index-url https://download.pytorch.org/whl/nightly/rocm5.6"
 ## And if you want to call this from other programs...
 export COMMANDLINE_ARGS="--api"
 ## crashes with 2 cards, so to get it to run on the second card (only), unremark the following 
 # export CUDA_VISIBLE_DEVICES="1"
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
Project website : https://github.com/comfyanonymous/ComfyUI

## environment...
if it's not already active - activate the sd environment... or make a new one if you prefer...
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
git clone https://github.com/comfyanonymous/ComfyUI
cd ComfyUI
```

## packages
```bash
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/rocm5.6 -r requirements.txt
```

## setup config...
```bash
cp extra_model_paths.yaml.example extra_model_paths.yaml
## set path in file
vi extra_model_paths.yaml
"base_path: ~/stable-diffusion/"
```

# prep models
This will vary depending on where you've put your models and how you arrange them...
```bash
mv models models.1
ln -s /path/to/models/ models
## a1111 uses different name for folder with models... link
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

To start the next section fresh, you may want to deactivate the sd conda environment. 
```bash
conda deactivate
```

---

#  Oobabooga - Text Generation WebUI - ROCm 
Project Website : https://github.com/oobabooga/text-generation-webui.git

## setup conda and freshen pip
```bash
# make env
conda create -y -n textgen python=3.10.9
conda activate textgen
```


Note : As of writing this (2023-09) the most recent 'stable' supported version is ROCm 5.4.2
 to see packages available : https://download.pytorch.org/whl/rocm5.4.2/
 nightly : https://download.pytorch.org/whl/nightly/rocm5.6/
 7900XTX requires nightly builds... 

```bash
# pre-install
pip install cmake colorama filelock lit numpy \
         --index-url https://download.pytorch.org/whl/nightly/rocm5.6
#         --index-url https://download.pytorch.org/whl/rocm5.4.2
```

```bash
# main pytorch parts...
pip install torch torchvision torchtext torchaudio torchdata \
	triton pytorch-triton pytorch-triton-rocm \
         --index-url https://download.pytorch.org/whl/nightly/rocm5.6
#       --index-url https://download.pytorch.org/whl/rocm5.4.2
```

## bitsandbytes rocm
 video guide : https://www.youtube.com/watch?v=2cPsvwONnL8 
[ - depricated - https://git.ecker.tech/mrq/bitsandbytes-rocm - requires rocm 5.4.2 ]
  https://github.com/0cc4m/bitsandbytes-rocm
Older system with support for 5.6 : https://github.com/RockeyCoss/bitsandbytes-rocm
Note : 2023-09-11 New version of BitsAndBytes(0.41 !) made for 5.6 
Project website : https://github.com/arlo-phoenix/bitsandbytes-rocm-5.6
Found newer version that supports ROCm 5.6 -

```bash
cd
git clone https://github.com/arlo-phoenix/bitsandbytes-rocm-5.6.git
cd bitsandbytes-rocm-5.6/
BUILD_CUDA_EXT=0 pip install -r requirements.txt --extra-index-url https://download.pytorch.org/whl/nightly/rocm5.6
# 7900XTX
#make hip ROCM_TARGET=gfx1100 ROCM_HOME=/opt/rocm-5.6.0/
# 6900XT
#make hip ROCM_TARGET=gfx1030 ROCM_HOME=/opt/rocm-5.6.0/
# both...
make hip ROCM_TARGET=gfx1100,gfx1030 ROCM_HOME=/opt/rocm-5.6.0/
pip install . --extra-index-url https://download.pytorch.org/whl/nightly/rocm5.6
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

## torch grammar ( requirement added to Oobabooga 1.7 on 2023-10-08) 
```bash
pip install torch-grammar 
```


2023-09-11 : 
Usually we get Triton from the PyTorch nightly build files (included above) 
but I had some errors [akin to these](https://github.com/openai/triton/issues/2002) 
and found getting it fresh from the nightly build resovled them.
```bash
pip install -U --index-url https://aiinfra.pkgs.visualstudio.com/PublicPackages/_packaging/Triton-Nightly/pypi/simple/ triton-nightly
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
pip install -r requirements.txt --extra-index-url https://download.pytorch.org/whl/nightly/rocm5.6
cd ~/text-generation-webui
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
If you're new to this - new models can be downloaded from the shell via a python script, or from a form in the interface.
There are lots of them - http://huggingface.co 
Generally the GPTQ models by TheBloke are likely to load.  The 30B/33B models will load on 24GB of VRAM, but may error, or run out of memory depending on usage and parameters.  

If you have old models,  link pre-stored models into the models
```bash
# cd ~/text-generation-webui
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

The exllama loader works with most GPTQ models.  This is the best choice as it is fast.   
Some models that won't load that way will load with AutoGPTQ - but without Triton ( triton seems to break things ). 
Also worth noting, I've had things work on one card or the other, but not on both cards, 
loading on both cards causes LLMs to spit out gibberish.  

## End - Oobabooga - Text-Generation-WebUI

# nvtop from source
project website : https://github.com/Syllo/nvtop
optional - tool for displaying gpu / memory usage info
The package for this crashes with 2 gpu's, here it is from source.
```bash
sudo apt install -y libdrm-dev libsystemd-dev libudev-dev
cd 
git clone https://github.com/Syllo/nvtop.git
mkdir -p nvtop/build && cd nvtop/build
cmake .. -DNVIDIA_SUPPORT=OFF -DAMDGPU_SUPPORT=ON -DINTEL_SUPPORT=OFF
make
sudo make install
```
