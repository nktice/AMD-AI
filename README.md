# AMD / Radeon 7900XTX 6900XT GPU ROCm install / setup / config 
# Ubuntu 22.04 / 23.04  
# AMD ROCm 5.6.1 
# Automatic1111 Stable Diffusion + ComfyUI ( venv ) 
# Oobabooga - Text Generation WebUI ( Conda / PyTorch (ROCm) / BitsAndBytes-ROCm ( 0.41.1 ) / ExLlama + ExLlamav2 ) 

## Install notes / instructions ##
2023-07 - I have composed this collection of instructions as they are my notes.  I use to setup my own Linux system with AMD parts. I've gone over these doing many re-installs to get them all right.  This is what I had hoped to find when I had search for install instructions -  so I'm sharing them in the hopes that they save time for other people.   There may be in here extra parts that aren't needed but this works for me.  Originally text, with comments like a shell script that I cut and paste 

2023-09-09 - I had a report that this doesn't work in virtual machines (virtualbox) as the system there cannot see the hardware, it can't load drivers, etc.  Windows users may find it more helpful to try DirectML - https://rocm.docs.amd.com/en/latest/deploy/windows/quick_start.html / https://github.com/lshqqytiger/stable-diffusion-webui-directml

2023-09-30 - Added new version for ROCm 5.7 -> https://github.com/nktice/AMD-AI/blob/main/ROCm-5.7.md  
2023-11-30 - Updated for ROCm 5.7.2 and PyTorch from Nightlies, and simplification of process -> https://github.com/nktice/AMD-AI/blob/main/ROCm-5.7.md  

2023-11-30 - Rewrite of this document to include updates... ROCm 5.6.1,  Stable Diffusion and ComfyUI sections rewritten for venv,  Textgen webui section updated to use more of the project requirements as designed.  Overall greatly simplified.  

2023-12-13 - Added supplement for those who want to use Mixtral models ( uses llama.cpp ) - https://github.com/nktice/AMD-AI/blob/main/Mixtral.md

2023-12-18 - ROCm 6.0 is out, so there's an updated guide for that here - https://github.com/nktice/AMD-AI/blob/main/ROCm6.0.md

2023-12-18 - Update this document ( for ROCm 5.6.1 ) as it is the current 'stable' version supported by PyTorch.  Test showed it was able to load the modest Mixtral variant TheBloke_mixtralnt-4x7b-test-GPTQ using ExLlamav2 - takes ~18GB out of VRAM to hold this model.  

2023-12-23 - Update to use miniconda instead of anaconda.  Exllamav2 improved details.  Looks like FA2 works.  Minor revisions.

-----


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

```bash
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/5.6.1 jammy main" \
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
```bash
sudo apt install -y wget git python3 python3-venv libgl1 libglib2.0-0
```

## Edit environment settings...
```bash
tee --append webui-user.sh <<EOF
 ## Torch for ROCm
# generic import...
export TORCH_COMMAND="pip install torch torchvision --index-url https://download.pytorch.org/whl/rocm5.6"
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
Note that the first time it starts it may take it a while to go and get things it's not always good about saying what it's up to. 
```bash
./webui.sh 
```

## end Stable Diffusion 

--- 

# ComfyUI install script 
- variation of https://raw.githubusercontent.com/ltdrdata/ComfyUI-Manager/main/scripts/install-comfyui-venv-linux.sh 
Includes ComfyUI-Manager

Same install of packages here as for Stable Diffusion ( included here in case you're not installed SD and just want ComfyUI... ) 
```bash
sudo apt install -y wget git python3 python3-venv libgl1 libglib2.0-0
```

```bash
cd 
git clone https://github.com/comfyanonymous/ComfyUI
cd ComfyUI/custom_nodes
git clone https://github.com/ltdrdata/ComfyUI-Manager
cd ..
python3 -m venv venv
source venv/bin/activate
# pre-install torch and torchvision from nightlies - note you may want to update versions...
python3 -m pip install --pre torch torchvision --index-url https://download.pytorch.org/whl/rocm5.6
python3 -m pip install -r requirements.txt  --extra-index-url https://download.pytorch.org/whl/rocm5.6
python3 -m pip install -r custom_nodes/ComfyUI-Manager/requirements.txt --extra-index-url https://download.pytorch.org/whl/rocm5.6

# end vend if needed...
deactivate
```

Scripts for running the program...
```bash
# run_gpu.sh
tee --append run_gpu.sh <<EOF
#!/bin/bash
source venv/bin/activate
python3 main.py --preview-method auto
EOF
chmod +x run_gpu.sh

#run_cpu.sh
tee --append run_cpu.sh <<EOF
#!/bin/bash
source venv/bin/activate
python3 main.py --preview-method auto --cpu
EOF
chmod +x run_cpu.sh
```

Update the config file to point to Stable Diffusion (presuming it's installed...)
```bash
# config file - connecto stable-diffusion-webui 
cp extra_model_paths.yaml.example extra_model_paths.yaml
sed -i "s@path/to@`echo ~`@g" extra_model_paths.yaml
# edit config file to point to your checkpoints etc 
#vi extra_model_paths.yaml
```

Now you can call ComfyUI through the script created above.

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
bash Miniconda3-latest-Linux-x86_64.sh -b
cd ~
ln -s miniconda3 conda
```

```bash
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

## PyTorch install...

```bash
# pre-install 
pip install --pre cmake colorama filelock lit numpy Pillow Jinja2 \
	mpmath fsspec MarkupSafe certifi filelock networkx \
	sympy packaging requests \
         --index-url https://download.pytorch.org/whl/rocm5.6
```

```bash
# install
pip install torch torchvision torchtext torchaudio torchdata  \
 --index-url https://download.pytorch.org/whl/rocm5.6     
```



### bitsandbytes rocm 
2023-09-11 New version of BitsAndBytes(0.41 !) made for 5.6 
Project website : https://github.com/arlo-phoenix/bitsandbytes-rocm-5.6

```bash
cd
git clone https://github.com/arlo-phoenix/bitsandbytes-rocm-5.6.git
cd bitsandbytes-rocm-5.6/
BUILD_CUDA_EXT=0 pip install -r requirements.txt --extra-index-url https://download.pytorch.org/whl/rocm5.6
# 7900XTX
#make hip ROCM_TARGET=gfx1100 ROCM_HOME=/opt/rocm-5.6.1/
# 6900XT
#make hip ROCM_TARGET=gfx1030 ROCM_HOME=/opt/rocm-5.6.1/
# both...
make hip ROCM_TARGET=gfx1100,gfx1030 ROCM_HOME=/opt/rocm-5.6.1/
pip install . --extra-index-url https://download.pytorch.org/whl/rocm5.6
```


### Flash-Attention 2 :
Install may take a few mins ( takes author close to 5 minutes at time of writing )...
```bash
cd
git clone https://github.com/ROCmSoftwarePlatform/flash-attention.git
cd flash-attention
pip install .
```
2023-11-30 - Note it appears PyTorch for ROCm doesn't include FA support at this time... as there's a warning : "UserWarning: 1Torch was not compiled with memory efficient attention. " Further this issue is noted here : https://github.com/pytorch/pytorch/issues/112997 - So while the above runs, it isn't operating at the present time. 
2023-12-23 - FA2 appears to be working.  YMMV. 

## Oobabooga / Text-generation-webui - Install webui...
```bash
cd
git clone https://github.com/oobabooga/text-generation-webui
cd text-generation-webui
```

### Oobabooga's 'requirements'
The default bitsandbytes for AMD is out of date and doesn't support GPU.  So we installed one earlier ( may be unsupported... ) we'll run sed first to adjust that line of the requirements...
```bash
sed -i "s@bitsandbytes==@bitsandbytes>=@g" requirements_amd.txt 
pip install -r requirements_amd.txt 
```

Exllama and Exllamav2 loaders ...
exllama isn't being maintained, but exllamav2 is...
2023-12-23 - After many tests, it appears that the exllamav2 that's installed above gives an error, so we're compiling and reinstalling exllama here as when we do that it does work.  
2024-01-19 - Something has broken and exllamav2 won't compile directly from checkout, 
The package version for 5.6 may be functional.  However, for those who want to compile the latest, or for their hardware, 
I've added a line to reset the checkout to the last known good / compiling version 0.0.11
```bash
## install exllama
##git clone https://github.com/turboderp/exllama repositories/exllama
# install exllamav2
git clone https://github.com/turboderp/exllamav2 repositories/exllamav2
cd repositories/exllamav2
# Force collection back to base 0.0.11 
git reset --hard a4ecea6
pip install .   
cd ../..
```

Let's create a script (run.sh) to run the program...
```bash
tee --append run.sh <<EOF
#!/bin/bash
## activate conda
conda activate textgen
## command to run server...
# python server.py
# preferred configuration... note that --listen makes it accessible on the local network.
python server.py --listen --extensions sd_api_pictures send_pictures gallery
conda deactivate
EOF
chmod u+x run.sh
```


Models 
If you're new to this - new models can be downloaded from the shell via a python script, or from a form in the interface.
There are lots of them - http://huggingface.co 
Generally the GPTQ models by TheBloke are likely to load.  The 30B/33B models will load on 24GB of VRAM, but may error, or run out of memory depending on usage and parameters.  

To get new models note the ~/text-generation-webui directory has a program " download-model.py " that is made for downloading models from HuggingFace's collection.  

If you have old models,  link pre-stored models into the models
```bash
# cd ~/text-generation-webui
# mv models models.1
# ln -s /path/to/models models
```

And here's the command to start the interface... note that it takes some time to download components the first time it runs. 
```bash
source run.sh
```


The exllamav2 loader works with most GPTQ models.  This is the best choice as it is fast.   
Some models that won't load that way will load with AutoGPTQ - but without Triton ( triton seems to break things ). 
Also worth noting, I've had things work on one card or the other, but not on both cards, 
loading on both cards causes LLMs to spit out gibberish.  

## End - Oobabooga - Text-Generation-WebUI


---

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

