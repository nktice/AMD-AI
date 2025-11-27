# AMD-AI - A choose your own adventure how-to user guide...
Tested on hardware : AMD Radeon 7900XTX and 6900XT GPUs ( including dual cards), and the Ryzen AI Max 395+ ( Strix Halo ). 
# Ubuntu Linux 24.04.3  / 25.10
# ROCm 7.1.1
# Stable Diffusion (Automatic1111 and ForgeUI for AMDGPUs ) + ComfyUI  ( venv ) 
# Oobabooga - Text Generation WebUI ( conda ) 

## Introduction 
Introduction note : I started writing this guide 2023, because at the time I had a lot of trouble getting stuff running.  There was a range of partial or out of date guides that I came across...  So I set out to write a fairly complete guide to help fill the void.  Many things have changed, and there's lots of small details that have come and gone.  Generally things have improved over the months I've been doing this.  As much as I'd hoped all the issues would resolve, and it'd be easy to do everything making such guide redundant, that's yet to happen.  There appear to be a lot of fiddly bits that need attention, simple workarounds to make things work together that aren't well explained. 

Please note that there is another supplemental set of instructions to use Ollama, and related tools kept in a separate page for simplicity - https://github.com/nktice/AMD-AI/blob/main/ollama-litellm-aider.md

## Install notes / instructions / changelog 

2025-09-17 - Started new draft for Ubunutu 24.04.3 / ROCm 7.0.2 ...
2025-10-17 - Working with GMKTec Evo-X2... 
2025-10-19 - added ForgeUI ( for AMDGPUs )

2025-10-22 - Testing functionality on newest version of Ubuntu 25.10 - requires some changes...

2025-10-23 - Appears AMD has released some new drivers for the Strix Halo - alas this series might not support older cards, and is a "prevew".  There are components, such as with Oobabooga, that are setup with versions of ROCm that don't support Strix Halo ( Ryzen AI Max ) series processors, but should work with older stuff through default install.  So here is a link to the Strix Halo drivers for those who want them... https://rocm.docs.amd.com/en/7.9.0-preview/install/rocm.html  I may include notes I find related to them where appropriate.  

2025-11-27 - I had been having crahes with my Strix Halo that delayed updates.  I managed to find this page with a workaround that avoids crashes - https://github.com/ROCm/ROCm/issues/5590#issuecomment-3573570390 - So if that's soemthing that is relevant to you, review details there. 

--------


# Ubuntu 24.04.3 - Base system install 
This is all tested on Ubuntu Linux 24.04.3 - this appears to be now fairly well supported by drivers and tools. 

This has been tested on 25.10 too...

At this point we assume you've done the system install
and you know what that is, have a user, root, etc. 

```bash
# update system packages 
sudo apt update -y && sudo apt upgrade -y 
```


## Add AMD GPU package sources 
Make the directory if it doesn't exist yet.
This location is recommended by the distribution maintainers.
https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/install-methods/package-manager/package-manager-ubuntu.html

```bash
# Make the directory if it doesn't exist yet.
# This location is recommended by the distribution maintainers.
sudo mkdir --parents --mode=0755 /etc/apt/keyrings

# Download the key, convert the signing-key to a full
# keyring required by apt and store in the keyring directory
wget https://repo.radeon.com/rocm/rocm.gpg.key -O - | \
    gpg --dearmor | sudo tee /etc/apt/keyrings/rocm.gpg > /dev/null
```

```bash
sudo tee /etc/apt/sources.list.d/rocm.list << EOF
deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/7.1.1 noble main
deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/graphics/7.1.1/ubuntu noble main
EOF

sudo tee /etc/apt/preferences.d/rocm-pin-600 << EOF
Package: *
Pin: release o=repo.radeon.com
Pin-Priority: 600
EOF

sudo apt update
```

# More AMD ROCm related packages 
Here's a complete list of packages they offer and what they include...
https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/install-methods/package-manager/package-manager-ubuntu.html#install-rocm
```bash
# ROCm...
# sudo apt install -y rocm 
sudo apt install -y rocm rocm-dev rocm-libs rocm-hip-sdk rocm-libs
```

# Post install setup 
From : https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/post-install.html
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

```bash
export LD_LIBRARY_PATH=/opt/rocm-7.1.1/lib
```


## Find graphics device
```bash
sudo /opt/rocm/bin/rocminfo | grep gfx
```
Examples of things you'd see... 
Found : gfx1030 [ Radeon 6900 ]
Found : gfx1100 [ Radeon 7900 ] 
Found : gfx1151 [ Ryzen AI Max 395+ ( Strix Halo ) ] 

## Add user to groups
Of course note to change the user name to match your user. 
```bash
sudo adduser `whoami` video
sudo adduser `whoami` render
```

## Useful packages --
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

An alternative that seems worth mentioning here is Mission Center : https://missioncenter.io/ 
It doesn't use apt, so we won't install it here, folks can look it up. 


## Radeon specific tools...
```bash
sudo apt install -y radeontop 
```

## and now we reboot...
```bash
reboot
```

## End of OS / base setup

--------

# Stable Diffusion 
Stable Diffusion is an amazing system to make AI art.  These open source tools are rather outdated now, and things have evolved.  It's been a challenge to get the python version that they will work with.  Newer versions break various dependencies, and things don't go well... 

## How to install Python 3.10 - 
These instructions are taken from this Q and A up on a forum : https://askubuntu.com/questions/1539019/how-to-install-python-3-10-on-ubuntu-24-10

Dependencies
```bash
sudo apt install -y build-essential libssl-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libffi-dev zlib1g-dev
```

Download and extract the sources :
```bash
cd /opt
sudo wget https://www.python.org/ftp/python/3.10.12/Python-3.10.12.tgz
sudo tar -xvf Python-3.10.12.tgz
```

Compile and Install
```bash
cd Python-3.10.12
sudo ./configure --enable-optimizations
sudo make altinstall -j
```
Now we have python3.10 installed and it is ready to use for SD programs... 


That is the magic to get that python version onto the system - that is essential for to get A1111 or forge UI up. 


## Automatic1111 
https://github.com/AUTOMATIC1111/stable-diffusion-webui
Get the files...
```bash
cd
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
cd stable-diffusion-webui
```

## Requisites : 
Automatic1111 / Stable Diffusion doesn't work with newer versions of python, so we specify one it works with. 
```bash
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
# export TORCH_COMMAND="pip install torch torchvision --index-url https://download.pytorch.org/whl/nightly/rocm7.1"
export TORCH_COMMAND="pip install --pre torch==2.10.0.dev20251123+rocm7.1 torchvision==0.25.0.dev20251124+rocm7.1 --extra-index-url https://download.pytorch.org/whl/nightly/rocm7.1"
 ## And if you want to call this from other programs...
 export COMMANDLINE_ARGS="--api"
 ## crashes with 2 cards, so to get it to run on the second card (only), unremark the following 
 # export CUDA_VISIBLE_DEVICES="1"
EOF
```


## Models 
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



## end A1111

## Stable Diffusion WebUI AMDGPU Forge ( Forge UI ) 

2025-10-19 - In trying SD on a "Strix Halo" system it runs very slow...  What I'm guessing is it's using the CPU and not GPU.  A1111 predates current GPUs and processor types.   So I went in search of things that may run better, and came across this.  As it's derived from the old Stable Diffusion it works similar... So I was able to get it up and functioning with some adjustments.  It also runs slow, but since I got it working, I'll include it here.

Stable Diffusion WebUI AMDGPU Forge github : https://github.com/lshqqytiger/stable-diffusion-webui-amdgpu-forge
See section above on installing Python 3.10 ... 

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
 export TORCH_COMMAND="pip install --pre torch==2.10.0.dev20251123+rocm7.1 torchvision==0.25.0.dev20251124+rocm7.1 --extra-index-url https://download.pytorch.org/whl/nightly/rocm7.1"
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

Here's two packages it didn't seem to install just using the scripts...
```bash
source venv/bin/activate
pip install "optimum[onnxruntime-gpu]" joblib --extra-index-url https://download.pytorch.org/whl/nightly/rocm7.1
deactivate
```

And with those in place, I ran the program again, and it worked for me. 
```bash
./webui.sh
```

## End Forge UI 


## SDNext
Here are instructions for for setting up SD Next a descendent of Stable Diffusion that looks like it is maintained at the present time.  Project page : https://github.com/vladmandic/sdnext 
2025-11-03 - Added these instructions...

```bash
sudo apt install python3 python3-venv git git-lfs
```

First we download the latest from GitHub...
```bash
cd
git clone https://github.com/vladmandic/sdnext.git
cd sdnext
```

SDNext is descended from Stable Diffusion such as seen above... so there is a lot of similar config, such as with venv... we'll want to pre-empt the default install methods and get torch installed...
```bash
python3 -m venv venv
source venv/bin/activate
# upgrade pip
python3 -m pip install -U pip
# If you want to pre-install torch and torchvision from nightlies 
python3 -m pip install --pre torch==2.10.0.dev20251123+rocm7.1 torchvision==0.25.0.dev20251124+rocm7.1  --extra-index-url https://download.pytorch.org/whl/nightly/rocm7.1
deactivate 
```

We may also want to prep our models folder to cut down on dupes...
```bash
# mv models models.1
# ln -s ../models/ models
```

If you want to set configuration options it also can use the same file name as the older sd versions - alas they haven't offered examples, so I will offer one here for something I want.
This parameter allows JavaScript access to the API... I've remarked this out, as it may not be secure for other users.
```bash
tee --append webui-user.sh <<EOF
# export COMMANDLINE_ARGS="--cors-origins=*"
EOF
```


Now we run the script that goes and sets it all up as they would expect... 
Sometimes it has errored, and needed running again to get all it wants.  
Once it is setup then this command will run what is setup / installed. 
```bash
./webui.sh
```



## End SDNext

# End of Stable Diffusion 

---



--- 

# ComfyUI 
- variation of https://raw.githubusercontent.com/ltdrdata/ComfyUI-Manager/main/scripts/install-comfyui-venv-linux.sh 
Includes ComfyUI-Manager
2025-10-22 - ComfyUI has been actively developed, and as such it can use modern python that comes in the packages ( old one also works... )   It's also quite fast ( compared to the old SD systems from above... ).  


```bash
cd 
git clone https://github.com/comfyanonymous/ComfyUI
cd ComfyUI/custom_nodes
git clone https://github.com/ltdrdata/ComfyUI-Manager
cd ..
python3 -m venv venv
source venv/bin/activate
python3 -m pip install -U pip 
## pre-install torch and torchvision from nightlies - note you may want to update versions... 
python3 -m pip install --pre torch==2.10.0.dev20251123+rocm7.1 torchvision==0.25.0.dev20251124+rocm7.1  torchsde torchaudio einops transformers\>=4.25.1 safetensors\>=0.4.2 aiohttp pyyaml Pillow scipy tqdm psutil av  --extra-index-url https://download.pytorch.org/whl/nightly/rocm7.1
## Note the following manually includes the contents of requirements.txt - because otherwise attempting to install the requirements goes and reinstalls torch over again. 
python3 -m pip install -r requirements.txt  --extra-index-url https://download.pytorch.org/whl/nightly/rocm7.1

python3 -m pip install -r custom_nodes/ComfyUI-Manager/requirements.txt --extra-index-url https://download.pytorch.org/whl/nightly/rocm7.1

# end vend if needed...
deactivate
```

Scripts for running the program...
Note that " TORCH_BLAS_PREFER_HIPBLASLT=0 " was needed as explained here - https://github.com/comfyanonymous/ComfyUI/issues/3698

```bash
# run_gpu.sh
tee --append run_gpu.sh <<EOF
#!/bin/bash
source venv/bin/activate
TORCH_BLAS_PREFER_HIPBLASLT=0 python3 main.py --preview-method auto
EOF
chmod +x run_gpu.sh

#run_cpu.sh
tee --append run_cpu.sh <<EOF
#!/bin/bash
source venv/bin/activate
TORCH_BLAS_PREFER_HIPBLASLT=0 python3 main.py --preview-method auto --cpu
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

Note with the models, it's looking for models in models/checkpoints - so you'll need to put models into that folder to get it to work, or configure things so that it can find the parts that you want to use. 

# End ComfyUI install


---

#  Oobabooga - Text Generation WebUI - ROCm 
Project Website : https://github.com/oobabooga/text-generation-webui.git

2025-11-27 - At the present time I cannot get this working - unfortunately.  

## Conda
2025-10-23 - In working with Ubuntu 25.10 I found there's an issue with Conda in various forms.  Turns out Ubuntu is shipping with a version of md5sum that makes different results from standard version, thus causes messes... there's a work around, as I'll get to below... but in my review, I found that there is not need for a bunch of stuff that there used to be... Oobabooga now has a working installer that is usable.  [ It used to be that their installer didn't work, and so we needed to setup ourselves with the whole environment and dependencies... that appears over, so we can slim this all down to a few commands. ] 

Here are the details of the work-around to use for new Ubuntu ( 25.10 ) :
https://forum.anaconda.com/t/critical-installation-failure-persistent-internal-md5-mismatch-anaconda-miniconda-2025-06-on-ubuntu-25-10/107525

Here are the commands to switch to GNU version of md5sum :
```bash
sudo apt install coreutils-from-gnu coreutils-from-uutils- --allow-remove-essential
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


