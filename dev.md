# AMD / Radeon 7900XTX GPU ROCm install / setup / config 
# Ubuntu 24.04.1
# ROCm 6.2.3
# Automatic1111 Stable Diffusion + ComfyUI  ( venv ) 
# Oobabooga - Text Generation WebUI ( conda, Exllamav2, BitsAndBytes ) 

## Install notes / instructions / changelog 
This document features development / nightly versions, ymmv... - if you want the stable versions those are located here : 
https://github.com/nktice/AMD-AI/blob/main/README.md

2023-07 - I have composed this collection of instructions as they are my notes, from varied efforts at a configuration that is consistent.  I've gone over these doing many re-installs to get them all right. This is what I had hoped to find when I had search for install instructions - so I'm sharing them in the hopes that they save time for other people. There may be in here extra parts that aren't needed but this works for me.  Originally text, with comments like a shell script that I cut and paste.

2023-09-09 - I had a report that this doesn't work in virtual machines (virtualbox) as the system there cannot see the hardware, it can't load drivers, etc. 

[ various updates abridged... ] 

2024-10-16 - ROCm 6.2.3 is out...    
- I tested 24.10, and amdgpu-dkms gave errors, and it isn't supported by deadsnakes, so didn't proceed.  ( It also wiped out my entire home partition unexpectedly... )
- Note bug report filed on issues with TGW. oobabooga/text-generation-webui#6471
- PyTorch 20241014, and 20241015... gave errors with hipBLASLt, so I needed to specify previous versions to get things working.  Bug report here : https://github.com/ROCm/hipBLASLt/issues/1243  This has been resolved as of 20241021 


--------


# Ubuntu 24.04.1 - Base system install 
With ROCm 6.2.3 there is now official support for Ubuntu 24.04.1 ( noble ) 

At this point we assume you've done the system install
and you know what that is, have a user, root, etc. 


Update system software packages... 
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


#### Support for older versions of Python
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
amdgpu repository...
```bash
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/amdgpu/6.2.3/ubuntu noble main' \
    | sudo tee /etc/apt/sources.list.d/amdgpu.list
sudo apt update -y 
```


## AMDGPU DKMS

```bash
sudo apt install  amdgpu-dkms  -y
```

## ROCm repositories 
https://rocmdocs.amd.com/en/latest/deploy/linux/os-native/install.html

```bash
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/6.2.3/ noble main" \
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
reboot
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
## This command should work - but the new ( 20241014, 20241015 pytorch versions have errors with hipBLASLt - 
#export TORCH_COMMAND="pip install --pre torch torchvision --extra-index-url https://download.pytorch.org/whl/nightly/rocm6.2"
export TORCH_COMMAND="pip install --pre torch==2.6.0.dev20241021+rocm6.2  torchvision==0.20.0.dev20241021+rocm6.2 --extra-index-url https://download.pytorch.org/whl/nightly/rocm6.2"
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
 Note that the first time it starts it may take it a while to go and get things  it's not always good about saying what it's up to.  Sometiems it crashes due to trying to load something after it starts, but does load on further attempts once that's in place. 
```bash
./webui.sh 
```



## end Stable Diffusion 

--- 

# ComfyUI install script 
- variation of https://raw.githubusercontent.com/ltdrdata/ComfyUI-Manager/main/scripts/install-comfyui-venv-linux.sh 
Includes ComfyUI-Manager

Install of packages like for Stable Diffusion ( included here in case you're not installed SD and just want ComfyUI... ) - but it's not bothered by Python 3.12... 
```bash
sudo apt install -y wget git python3.10 python3.10-venv libgl1 
```

```bash
cd 
git clone https://github.com/comfyanonymous/ComfyUI
cd ComfyUI/custom_nodes
git clone https://github.com/ltdrdata/ComfyUI-Manager
cd ..
```

```bash
python3.10 -m venv venv
source venv/bin/activate
python3.10 -m pip install -U pip 
## pre-install torch and torchvision from nightlies - note you may want to update versions... 
## Note the following manually includes the contents of requirements.txt - because otherwise attempting to install the requirements goes and reinstalls torch over again. 
# python3.10 -m pip install --pre torch torchvision torchsde torchaudio einops transformers>=4.25.1 safetensors>=0.4.2 aiohttp pyyaml Pillow scipy tqdm psutil  --extra-index-url https://download.pytorch.org/whl/nightly/rocm6.2
python3.10 -m pip install --pre torch==2.6.0.dev20241021+rocm6.2  torchvision==0.20.0.dev20241021+rocm6.2  torchsde torchaudio einops transformers>=4.25.1 safetensors>=0.4.2 aiohttp pyyaml Pillow scipy tqdm psutil  --extra-index-url https://download.pytorch.org/whl/nightly/rocm6.2
python3.10 -m pip install -r custom_nodes/ComfyUI-Manager/requirements.txt --extra-index-url https://download.pytorch.org/whl/nightly/rocm6.2
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
```bash
# config file - connecto stable-diffusion-webui 
cp extra_model_paths.yaml.example extra_model_paths.yaml
sed -i "s@path/to@`echo ~`@g" extra_model_paths.yaml
# edit config file to point to your checkpoints etc 
#vi extra_model_paths.yaml
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

2024-10-16 - New version of TGW 1.15 has resolved some issues ( the requirements_amd.txt installs smoothly now... ) but has introduced issues with the locally built loaders ( exllamav2 and llamacpp ).  I've filed a bug report with them, but have remarked out those portions of these instructions for now, as their exllamav2 loader works from fresh install, but breaks after manual install.  
https://github.com/oobabooga/text-generation-webui/issues/6471


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
         --index-url https://download.pytorch.org/whl/nightly/rocm6.2
```


Here is old method that works - but tries to download things back to the start of these releases which may take a while... so it is remarked... 
```bash
## install
#pip install torch torchvision torchtext torchaudio torchdata \
#	triton pytorch-triton pytorch-triton-rocm \
#         --index-url https://download.pytorch.org/whl/nightly/rocm5.7
```
Instead of that we go and look through the files at https://download.pytorch.org/whl/nightly/rocm6.1/ (note trailing slash) and in the program directories there, we can see the individual nightly build files.  One has been chosen at the time of writing this, if you want newer, that is where you can find those details to update the file names / versions.  

Here we refer to specific nightly versions to keep things simple. 
```bash
#pip install --pre -U torch torchvision  \
pip install --pre -U torch==2.6.0.dev20241021+rocm6.2  torchvision==0.20.0.dev202410121+rocm6.2 \
	torchtext torchaudio pytorch-triton pytorch-triton-rocm  \
	 --index-url https://download.pytorch.org/whl/nightly

```

2024-05-12 For some odd reason, torchtext isn't recognized, even though it's there... so we specify it using it's URL to be explicit.  
```bash
# pip install https://download.pytorch.org/whl/cpu/torchtext-0.18.0%2Bcpu-cp311-cp311-linux_x86_64.whl#sha256=c760e672265cd6f3e4a7c8d4a78afe9e9617deacda926a743479ee0418d4207d
```

### Triton 
2023-09-11 : Usually we get Triton from the PyTorch nightly build files (included above)  but I had some errors [akin to these](https://github.com/openai/triton/issues/2002)  and found getting it fresh from the nightly build resovled them.
2023-12.17 : The issue appears to have been resolved, so I'm remarking this out, but leaving it here in case there are issues with Triton that may call for installing the nightly again.  
```bash
#pip install -U --index-url https://aiinfra.pkgs.visualstudio.com/PublicPackages/_packaging/Triton-Nightly/pypi/simple/ triton-nightly
```


### bitsandbytes rocm 
2024-04-24 - AMD's own ROCm version of bitsandbytes has been updated! - https://github.com/ROCm/bitsandbytes  ( ver 0.44.0.dev0 at time of writing ) 
```bash
# conda activate textgen
cd
git clone https://github.com/ROCm/bitsandbytes.git
cd bitsandbytes
pip install .
```


### Flash-Attention 2 :
Install may take a few mins ( takes author close to 5 minutes at time of writing )...
2024-01-18 - FA2 appears to be 'working' now... as in it compiles and installs normally. 
2024-04-03 - While this (2.0.4) version 'works' it isn't recent enough to be used by exllamav2 - Here is more info : https://github.com/turboderp/exllamav2/issues/397#issuecomment-2034652594 (2.2.1...) - it's a known issue and has had a support thread with the main authors - https://github.com/Dao-AILab/flash-attention/issues/707 
2024-08-04 - Isn't compiling... there are occasionally updates to the sources so it looks like they're working on it, but at a glacial pace... remarked out for now.
```bash
#cd
#git clone --recurse-submodules https://github.com/ROCmSoftwarePlatform/flash-attention.git
#cd flash-attention
#pip install . 
```

## Oobabooga / Text-generation-webui - Install webui...
```bash
cd
git clone https://github.com/oobabooga/text-generation-webui
cd text-generation-webui
```

### Oobabooga's 'requirements'

```bash
pip install -r requirements_amd.txt  --extra-index-url https://download.pytorch.org/whl/nightly/rocm6.2
```


#### Exllamav2 loader 
https://github.com/turboderp/exllamav2

```bash
# git clone https://github.com/turboderp/exllamav2 repositories/exllamav2
# cd repositories/exllamav2
# pip install -r requirements.txt  --extra-index-url https://download.pytorch.org/whl/nightly/rocm6.2
# pip install .   --extra-index-url https://download.pytorch.org/whl/nightly/rocm6.2
# cd ../..
```

#### Llama-cpp-python - 
- Another loader, that is highly efficient in resource use, but not very fast. https://github.com/abetlen/llama-cpp-python  It may need models in GGUF format ( and not other types ).
- Note compiling took 5:35 on my 5950x cpu... so be patient. 
```
## remove old versions
# pip uninstall llama_cpp_python -y 
# pip uninstall llama_cpp_python_cuda -y 
# ## install llama-cpp-python
# git clone  --recurse-submodules  https://github.com/abetlen/llama-cpp-python.git repositories/llama-cpp-python 
# cd repositories/llama-cpp-python
# CC='/opt/rocm/llvm/bin/clang' CXX='/opt/rocm/llvm/bin/clang++' CFLAGS='-fPIC' CXXFLAGS='-fPIC' CMAKE_PREFIX_PATH='/opt/rocm' ROCM_PATH="/opt/rocm" HIP_PATH="/opt/rocm" CMAKE_ARGS="-GNinja -DLLAMA_HIPBLAS=ON -DLLAMA_AVX2=on " pip install --no-cache-dir .
# cd ../.. 
```

Here's a supplement written when Mixtral was new and not supported for how to install Auto-GPTQ, and Llama.cpp at that time - deprecated now, but may be of interest for some explorers. 
https://github.com/nktice/AMD-AI/blob/main/Mixtral.md

Let's create a script (run.sh) to run the program... 
note "TORCH_BLAS_PREFER_HIPBLASLT=0" is to deal with the issue described here - https://github.com/comfyanonymous/ComfyUI/issues/3698
```bash
tee --append run.sh <<EOF
#!/bin/bash
## activate conda
conda activate textgen
## command to run server... 
TORCH_BLAS_PREFER_HIPBLASLT=0 python server.py --extensions sd_api_pictures send_pictures gallery
# if you want it to show up on your own network, add --listen 
#python server.py --listen --extensions sd_api_pictures send_pictures gallery 
conda deactivate
EOF
chmod u+x run.sh
```


Models 
If you're new to this - new models can be downloaded from the shell via a python script, or from a form in the interface.
There are lots of them - http://huggingface.co 
Generally the GPTQ models by TheBloke are likely to load... https://huggingface.co/TheBloke  The 30B/33B models will load on 24GB of VRAM, but may error, or run out of memory depending on usage and parameters.  
Worthy of mention, TurboDerp ( author of the exllama loaders ) has been posting exllamav2 ( exl2 ) processed versions of models - https://huggingface.co/turboderp ( for use with exllamav2 loader ) - when downloading, note the --branch option.  

To get new models note the ~/text-generation-webui directory has a program " download-model.py " that is made for downloading models from HuggingFace's collection.  

If you have old models,  link pre-stored models into the models
```bash
# cd ~/text-generation-webui
# mv models models.1
# ln -s /path/to/models models
```

Note that to run the script we need to use the source command as follows : 
```bash
# conda activate textgen
source run.sh
```

## End - Oobabooga - Text-Generation-WebUI



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


