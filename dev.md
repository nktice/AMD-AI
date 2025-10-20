# AMD-AI - A choose your own adventure how-to user guide...
Tested on hardware : AMD Radeon 7900XTX and 6900XT GPUs ( including dual cards), and the Ryzen AI Max 395+ ( Strix Halo ). 
# Ubuntu Linux 24.04.3  
# ROCm 7.0.2
# Stable Diffusion (Automatic1111 and ForgeUI for AMDGPUs ) + ComfyUI  ( venv ) 
# Oobabooga - Text Generation WebUI ( conda ) 

## Introduction 
Introduction note : I started writing this guide 2023, because at the time I had a lot of trouble getting stuff running.  There was a range of partial or out of date guides that I came across...  So I set out to write a fairly complete guide to help fill the void.  Many things have changed, and there's lots of small details that have come and gone.  Generally things have improved over the months I've been doing this.  As much as I'd hoped all the issues would resolve, and it'd be easy to do everything making such guide redundant, that's yet to happen.  There appear to be a lot of fiddly bits that need attention, simple workarounds to make things work together that aren't well explained. 

Please note that there is another supplemental set of instructions to use Ollama, and related tools kept in a separate page for simplicity - https://github.com/nktice/AMD-AI/blob/main/ollama-litellm-aider.md

## Install notes / instructions / changelog 

2025-09-17 - Started new draft for Ubunutu 24.04.3 / ROCm 7.0.2 ...

2025-10-17 - Working with GMKTec Evo-X2... 

2025-10-19 - added ForgeUI ( for AMDGPUs )... 

2025-10-20 - Replaced main dev.md with this file, old dev instructions can be found here : 
https://github.com/nktice/AMD-AI/blob/main/ROCm-6.2.3-Dev.md 

--------


# Ubuntu 24.04.3 - Base system install 
This is all tested on Ubuntu Linux 24.04.3 - this appears to be now fairly well supported by drivers and tools. 

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

#### [ for Ubuntu 23.04, 23.10, and 24.04 ... ]
Some things may require various old packages so we need to add
jammy packages, so that they can be installed, on lunar systems.
```bash
sudo add-apt-repository -y -s deb http://security.ubuntu.com/ubuntu jammy main universe
```

#### [ For Ubuntu 24.04 ... ] 
This allows calls to older versions of Python by using "deadsnakes" 
```bash
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update -y 
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
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/amdgpu/7.0.2/ubuntu jammy main' \
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

```bash
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/7.0.2/ jammy main" \
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
# RIP rocm-dkms
# sudo apt install -y rocm-dev rocm-libs rocm-hip-sdk rocm-dkms rocm-libs
sudo apt install -y rocm-dev rocm-libs rocm-hip-sdk rocm-libs
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
Note : I have had issues with the distro version crashes with 2 GPUs, installing new version from sources works fine.  Instructions for that are included at the bottom, as they depend on things installed between here and there.   Project website : https://github.com/Syllo/nvtop 
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

--------


# Stable Diffusion (Automatic1111) 
This system is built to use its own venv ( rather than Conda )...

## Download Stable Diffusion ( Automatic1111 webui ) 
https://github.com/AUTOMATIC1111/stable-diffusion-webui
Get the files...
```bash
cd
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
cd stable-diffusion-webui
```

# Requisites : 
Automatic1111 / Stable Diffusion doesn't work with newer versions of python, so we specify one it works with. 
```bash
sudo apt install -y wget git python3.10 python3.10-venv libgl1
python3.10 -m venv venv
source venv/bin/activate
python3.10 -m pip install -U pip
deactivate
```

## Edit environment settings...
```bash
tee --append webui-user.sh <<EOF
# specify compatible python version
python_cmd="python3.10"
 ## Torch for ROCm
# generic import...
# export TORCH_COMMAND="pip install torch torchvision --index-url https://download.pytorch.org/whl/nightly"
# workaround for ROCm + Torch > 2.4.x - https://github.com/comfyanonymous/ComfyUI/issues/3698
 export TORCH_BLAS_PREFER_HIPBLASLT=0
# use specific versions to avoid downloading all the nightlies... ( update dates as needed )
# Note that torchvision sometimes needs the previous night's version of torch, so their dates are sequential
 export TORCH_COMMAND="pip install --pre torch==2.10.0.dev20251016+rocm7.0 torchvision==0.25.0.dev20251017+rocm7.0 --extra-index-url https://download.pytorch.org/whl/nightly/rocm7.0"
 ## And if you want to call this from other programs...
 export COMMANDLINE_ARGS="--api"
 ## crashes with 2 cards, so to get it to run on the second card (only), unremark the following 
 # export CUDA_VISIBLE_DEVICES="1"
EOF
```

## Models...
If you keep models for SD somewhere, this is where you'd link them up...
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

The first time this is run it will install the requirements.  May take a few tries for it to get everything the way it likes it. 



## end Stable Diffusion 


## Stable Diffusion WebUI AMDGPU Forge ( Forge UI ) 

2025-10-19 - In trying SD on a "Strix Halo" system it runs very slow...  What I'm guessing is it's using the CPU and not GPU.  A1111 predates current GPUs and processor types.   So I went in search of things that may run better, and came across this.  As it's derived from the old Stable Diffusion it works similar... So I was able to get it up and functioning with some adjustments.  

Stable Diffusion WebUI AMDGPU Forge github : https://github.com/lshqqytiger/stable-diffusion-webui-amdgpu-forge

Git from github - 
```bash
cd
git clone https://github.com/lshqqytiger/stable-diffusion-webui-amdgpu-forge.git
cd stable-diffusion-webui-amdgpu-forge
```

Update the config file...
```bash
tee --append webui-user.sh <<EOF
# specify compatible python version
python_cmd="python3.10"
# use specific versions to avoid downloading all the nightlies... ( update dates as needed )
# Note that torchvision sometimes needs the previous night's version of torch, so their dates are sequential
 export TORCH_COMMAND="pip install --pre torch==2.10.0.dev20251016+rocm7.0 torchvision==0.25.0.dev20251017+rocm7.0 --extra-index-url https://download.pytorch.org/whl/nightly/rocm7.0"
 ## And if you want to call this from other programs...
 export COMMANDLINE_ARGS="--api"
EOF
```

If you have models stored in some directory, you could link them now...
```bash
#mv models models.1
#ln -s ~/models/  models
```

Initial setup of program environment, that gave me issues... 
```bash
./webui.sh
```

That went and got things... of which there wwre some issues, so I closed it out, and then manually ran program environment, and got what it wants...
```bash
source venv/bin/activate
pip install "optimum[onnxruntime-gpu]" joblib --extra-index-url https://download.pytorch.org/whl/nightly/rocm7.0
deactivate
```

And with those in place, I ran the program again, and it worked for me. 
```bash
./webui.sh
```


## End of Stable Diffusion WebUI AMDGPU Forge


--- 

# ComfyUI install script 
- variation of https://raw.githubusercontent.com/ltdrdata/ComfyUI-Manager/main/scripts/install-comfyui-venv-linux.sh 
Includes ComfyUI-Manager

Install of packages like for Stable Diffusion ( included here in case you're not installed SD and just want ComfyUI... ) - but it's not bothered by Python 3.12... 
```bash
sudo apt install -y wget git python3.10 python3.10-venv libgl1 sqlite
```

```bash
cd 
git clone https://github.com/comfyanonymous/ComfyUI
cd ComfyUI/custom_nodes
git clone https://github.com/ltdrdata/ComfyUI-Manager
cd ..
python3.10 -m venv venv
source venv/bin/activate
python3.10 -m pip install -U pip 
## pre-install torch and torchvision from nightlies - note you may want to update versions... 
python3.10 -m pip install --pre torch==2.10.0.dev20251016+rocm7.0 torchvision==0.25.0.dev20251017+rocm7.0  torchsde torchaudio einops transformers\>=4.25.1 safetensors\>=0.4.2 aiohttp pyyaml Pillow scipy tqdm psutil av  --extra-index-url https://download.pytorch.org/whl/nightly/rocm7.0
## Note the following manually includes the contents of requirements.txt - because otherwise attempting to install the requirements goes and reinstalls torch over again. 
python3.10 -m pip install -r /home/n/ComfyUI/requirements.txt  --extra-index-url https://download.pytorch.org/whl/nightly/rocm7.0

python3.10 -m pip install -r custom_nodes/ComfyUI-Manager/requirements.txt --extra-index-url https://download.pytorch.org/whl/nightly/rocm7.0

# end vend if needed...
deactivate
```

Scripts for running the program...
Note that " TORCH_BLAS_PREFER_HIPBLASLT=0 " is needed as explained here - https://github.com/comfyanonymous/ComfyUI/issues/3698

```bash
# run_gpu.sh
tee --append run_gpu.sh <<EOF
#!/bin/bash
source venv/bin/activate
TORCH_BLAS_PREFER_HIPBLASLT=0 python3.10 main.py --preview-method auto
EOF
chmod +x run_gpu.sh

#run_cpu.sh
tee --append run_cpu.sh <<EOF
#!/bin/bash
source venv/bin/activate
TORCH_BLAS_PREFER_HIPBLASLT=0 python3.10 main.py --preview-method auto --cpu
EOF
chmod +x run_cpu.sh
```

Update the config file to point to Stable Diffusion (presuming it's installed...)
2025-10-17 - The format of this file has been completely changed, so the following code that used to work doesn't anymore...  This does give a clue as to what the config file is named and what you might want to do with it. 
```bash
## config file - connecto stable-diffusion-webui 
#cp extra_model_paths.yaml.example extra_model_paths.yaml
#sed -i "s@path/to@`echo ~`@g" extra_model_paths.yaml
## edit config file to point to your checkpoints etc 
##vi extra_model_paths.yaml
```

## End ComfyUI install


---

#  Oobabooga - Text Generation WebUI - ROCm 
Project Website : https://github.com/oobabooga/text-generation-webui.git


## Conda
First we'll need Conda ... Required for pytorch... Conda provides virtual environments for python, so that programs with different dependencies can have different environments.
Here is more info on managing conda : https://docs.conda.io/projects/conda/en/latest/user-guide/getting-started.html#
Other notes : https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html
Download info : https://www.anaconda.com/download/


Anaconda ( if you prefer this to miniconda below ) 
```bash
#cd ~/Downloads/
#wget https://repo.anaconda.com/archive/Anaconda3-2023.09-0-Linux-x86_64.sh
#bash Anaconda3-2023.09-0-Linux-x86_64.sh -b
#cd ~
#ln -s anaconda3 conda
```

Miniconda ( if you prefer this to Anaconda above... ) 
[ https://docs.conda.io/projects/miniconda/en/latest/ ] 
```bash
cd ~/Downloads/
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh -b -c
cd ~
ln -s miniconda3 conda
```


```bash
echo "PATH=~/conda/bin:$PATH" >> ~/.profile
source ~/.profile
# Accept Conda's TOS 
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r
conda update -y -n base -c defaults conda
```


```bash
conda install -y cmake ninja
```

### conda is now active...

### install pip
```bash
sudo apt install -y pip
pip3 install --upgrade pip
```

#### useful pip stuff to know ... 
```bash
## show outdated packages...
#pip list --outdated
## check dependencies 
#pip check
## install specified bersion 
#pip install <packagename>==<version>
```

### End conda and pip setup.

## Oobabooga / Textgen webui 
- https://github.com/oobabooga/text-generation-webui

```bash
conda create -n textgen python=3.11 -y
conda activate textgen
```



### bitsandbytes rocm 
2024-04-24 - AMD's own ROCm version of bitsandbytes has been updated! - https://github.com/ROCm/bitsandbytes  ( ver 0.44.0.dev0 at time of writing ) 
2025-10-17 - Looks like this has been updated in ways that are dysfunctional so this has been remarked out for now. 

```bash
## conda activate textgen
#cd
#git clone https://github.com/ROCm/bitsandbytes.git
#cd bitsandbytes
#pip install .
```


## Oobabooga / Text-generation-webui - Install webui...
```bash
cd
git clone https://github.com/oobabooga/text-generation-webui
cd text-generation-webui
```

Models 
If you're new to this - new models can be downloaded from the shell via a python script, or from a form in the interface.
There are lots of them - http://huggingface.co 
Generally the GPTQ models by TheBloke are likely to load... https://huggingface.co/TheBloke  The 30B/33B models will load on 24GB of VRAM, but may error, or run out of memory depending on usage and parameters.  
Worthy of mention, TurboDerp ( author of the exllama loaders ) has been posting exllamav2 ( exl2 ) processed versions of models - https://huggingface.co/turboderp ( for use with exllamav2 loader ) - when downloading, note the --branch option.  

To get new models note the ~/text-generation-webui directory has a program " download-model.py " that is made for downloading models from HuggingFace's collection.  

If you have old models,  link pre-stored models into the models
```bash
# cd ~/text-generation-webui/user_data
# mv models models.1
# ln -s /path/to/models models
```


### Many things have changed so we're trying to use Oobabooga's installer 
- it now uses Vulkan instead of ROCm... so some of the previous instructions may be redundant... working on it. 
```bash
sudo apt install curl
./start_linux.sh 
```

2025-10-17 - Image generation interaction is now somewhat more limited, that needs options for sd_api_pictures and send_pictures need to be turned on, and Stable Diffusion (started first) running.  


## End - Oobabooga - Text-Generation-WebUI

2024-08-17 - 
Here's an example, nvtop, sd console, tgw console... 
this screencap taken using ROCm 6.1.3 - under this config : https://github.com/nktice/AMD-AI/blob/main/ROCm-6.1.3-Dev.md
![Alt text](Screenshot-2024-08-17.png)

-------

# nvtop from source
( As one from packages crashes on 2 GPUs, while this never version from sources works fine. ) 
project website : https://github.com/Syllo/nvtop
optional - tool for displaying gpu / memory usage info
The package for this crashes with 2 gpu's, here it is from source.
```bash
sudo apt install -y libdrm-dev libsystemd-dev libudev-dev cmake
cd 
git clone https://github.com/Syllo/nvtop.git
mkdir -p nvtop/build && cd nvtop/build
cmake .. -DNVIDIA_SUPPORT=OFF -DAMDGPU_SUPPORT=ON -DINTEL_SUPPORT=OFF
make
sudo make install
```
# end nvtop


