# AMD Radeon 7900XTX GPU ROCm install / setup / config 
# Ubuntu 24.04
# ROCm 6.2
# Automatic1111 Stable Diffusion + ComfyUI  ( venv ) 
# Oobabooga - Text Generation WebUI ( conda, Exllamav2, Llama-cpp-python, BitsAndBytes ) 

## Install notes / instructions 
This file is focused on the current stable version of PyTorch.  There is another variation of these instructions for the development / nightly version(s) here : https://github.com/nktice/AMD-AI/blob/main/dev.md

2023-07 - 
 I have composed this collection of instructions as they are my notes.
 I use this setup my own Linux system with AMD parts.
 I've gone over these doing many re-installs to get them all right.
 This is what I had hoped to find when I had search for install instructions -
 so I'm sharing them in the hopes that they save time for other people. 
 There may be in here extra parts that aren't needed but this works for me.
 Originally text, with comments like a shell script that I cut and paste.

2023-09-09 - I had a report that this doesn't work in virtual machines (virtualbox) as the system there cannot see the hardware, it can't load drivers, etc.  While this is not a guide about Windows, Windows users may find it more helpful to try DirectML - https://rocm.docs.amd.com/en/latest/deploy/windows/quick_start.html / https://github.com/lshqqytiger/stable-diffusion-webui-directml

[ ... updates abridged ... ] 

2024-07-24 - PyTorch has updated with 2.4 now stable and referring to ROCm 6.1, so there's updates here to reflect those changes. 

2024-08-04 - ROCm 6.2 is out, including support for the current version of Ubuntu (24.04 / Noble) so this revision includes changes to emphasize use of the new version.  Previous stable has been set aside here - https://github.com/nktice/AMD-AI/blob/main/ROCm-6.1.3-Stable.md - Note I'm getting errors with the 2nd GPU with the new ROCm, bug report is filed, here is a link to that thread so you can follow :  https://github.com/ROCm/ROCm/issues/3518   

2024-09-11 - Ubuntu 24.04 has introduced Linux Kernel 6.8.0-44 Generic, it turns out this kernel is incompatible with amdgpu-dkms . I did the normal (daily) sudo apt update -y && sudo apt upgrade -y and got errors about amdgpu-dkms not installing, and then in the next reboot Ubuntu wouldn't start (black screen at boot). So beware of this upgrade, as things are disasterously broken at the present time.  Bug report here : https://github.com/ROCm/ROCm/issues/3701

-----


# Ubuntu 24.04 - Base system install 
ROCm 6.2 includes support for Ubuntu 24.04 (noble). 

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


## Support older version of Python
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

amdgpu repository 
```bash
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/amdgpu/6.2/ubuntu noble main' \
    | sudo tee /etc/apt/sources.list.d/amdgpu.list
sudo apt update -y 
```

AMDGPU DKMS
```bash
sudo apt install -y amdgpu-dkms
```

# ROCm repositories...
https://rocmdocs.amd.com/en/latest/deploy/linux/os-native/install.html

```bash
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/6.2 noble main" \
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

My 6900 reported as gfx1030, and my 7900 XTX show up as gfx1100 

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

The 1.9.x+ release series breaks the API so that it won't work with Oobabooga's TGW - so the following resets to use the 1.8.0 relaase that does work with Oobabooga.  
2024-07-04 - Oobabooga 1.9 resolves this issue - these lines are remarked out for now, but preserved in case someone wants to see how to do something similar in the future...
```bash
# git checkout bef51ae
# git reset --hard
```


# Requisites : 
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
# workaround for ROCm + Torch > 2.4.x - https://github.com/comfyanonymous/ComfyUI/issues/3698
 export TORCH_BLAS_PREFER_HIPBLASLT=0
# generic import...
# export TORCH_COMMAND="pip install torch torchvision --index-url https://download.pytorch.org/whl/nightly/rocm6.1"
# use specific versions to avoid downloading all the nightlies... ( update dates as needed ) 
 export TORCH_COMMAND="pip install --pre torch torchvision --index-url https://download.pytorch.org/whl/rocm6.1"
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

## end Stable Diffusion 

-------

# ComfyUI install script 
- variation of https://raw.githubusercontent.com/ltdrdata/ComfyUI-Manager/main/scripts/install-comfyui-venv-linux.sh 
Includes ComfyUI-Manager

Same install of packages here as for Stable Diffusion ( included here in case you're not installed SD and just want ComfyUI... ) 
```bash
sudo apt install -y wget git python3 python3-venv libgl1 
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
python3 -m pip install --pre torch torchvision --index-url https://download.pytorch.org/whl/rocm6.1
python3 -m pip install -r requirements.txt  --extra-index-url https://download.pytorch.org/whl/rocm6.1
python3 -m pip install -r custom_nodes/ComfyUI-Manager/requirements.txt --extra-index-url https://download.pytorch.org/whl/rocm6.1

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
         --index-url https://download.pytorch.org/whl/rocm6.1
```

There's version conflicts, so we specify versions that we want installed - 
```bash
#pip install --pre torch torchvision torchtext torchaudio triton pytorch-triton-rocm \
#pip install --pre torch==2.3.1+rocm6.0 torchvision==0.18.1+rocm6.0 torchaudio==2.3.1 triton pytorch-triton-rocm   \
#   --index-url https://download.pytorch.org/whl/rocm6.0
pip install --pre torch==2.4.0+rocm6.1 torchvision==0.19.0+rocm6.1 torchaudio==2.4.0 triton pytorch-triton-rocm   \
  --index-url https://download.pytorch.org/whl/rocm6.1
```
2024-05-12 For some odd reason, torchtext isn't recognized, even though it's there... so we specify it using it's URL to be explicit. 
```bash
pip install https://download.pytorch.org/whl/cpu/torchtext-0.18.0%2Bcpu-cp311-cp311-linux_x86_64.whl#sha256=c760e672265cd6f3e4a7c8d4a78afe9e9617deacda926a743479ee0418d4207d
```




### bitsandbytes rocm 
2024-04-24 - AMD's own ROCm version of bitsandbytes has been updated! - https://github.com/ROCm/bitsandbytes ( ver 0.44.0.dev0 at time of writing )
```bash
cd
git clone https://github.com/ROCm/bitsandbytes.git
cd bitsandbytes
pip install .
```


## Oobabooga / Text-generation-webui - Install webui...
```bash
cd
git clone https://github.com/oobabooga/text-generation-webui
cd text-generation-webui
```

## Oobabooga / Text-generation-webui - Install webui...
```bash
cd
git clone https://github.com/oobabooga/text-generation-webui
cd text-generation-webui
```

### Oobabooga's 'requirements'
2024-07-26 Oobabooga release 1.12 changed how requirements are done, including calls that refer to old versions of PyTorch which didn't work for me... So the usual command here is remarked out, and I have instead offered a replacement requirements.txt with minimal includes, that combined with what else is here gets it up and running ( for me ), using more recent versions of packages. 

```bash
#pip install -r requirements_amd.txt 
```

```bash
tee --append requirements_amdai.txt <<EOF
# alternate simplified requirements from https://github.com/nktice/AMD-AI
accelerate>=0.32
colorama
datasets
einops
gradio>=4.26
hqq>=0.1.7.post3
jinja2>=3.1.4
lm_eval>=0.3.0
markdown
numba>=0.59
numpy>=1.26
optimum>=1.17
pandas
peft>=0.8
Pillow>=9.5.0
psutil
pyyaml
requests
rich
safetensors>=0.4
scipy
sentencepiece
tensorboard
transformers>=4.43
tqdm
wandb

# API
SpeechRecognition>=3.10.0
flask_cloudflared>=0.0.14
sse-starlette>=1.6.5
tiktoken

EOF
pip install -r requirements_amdai.txt  --extra-index-url https://download.pytorch.org/whl/nightly/rocm6.1
```


#### Exllamav2 loader
```bash
git clone https://github.com/turboderp/exllamav2 repositories/exllamav2
cd repositories/exllamav2
## Force collection back to base 0.0.11 
## git reset --hard a4ecea6
pip install -r requirements.txt  --extra-index-url https://download.pytorch.org/whl/rocm6.1
pip install .   --index-url https://download.pytorch.org/whl/rocm6.1
cd ../..
```


#### Llama-cpp-python 
2024-06-18 - Llama-cpp-python - Another loader, that is highly efficient in resource use, but not very fast. https://github.com/abetlen/llama-cpp-python  It may need models in GGUF format ( and not other types ).  
```
## remove old versions
pip uninstall llama_cpp_python -y 
pip uninstall llama_cpp_python_cuda -y
## install llama-cpp-python 
git clone  --recurse-submodules  https://github.com/abetlen/llama-cpp-python.git repositories/llama-cpp-python 
cd repositories/llama-cpp-python
CC='/opt/rocm/llvm/bin/clang' CXX='/opt/rocm/llvm/bin/clang++' CFLAGS='-fPIC' CXXFLAGS='-fPIC' CMAKE_PREFIX_PATH='/opt/rocm' ROCM_PATH="/opt/rocm" HIP_PATH="/opt/rocm" CMAKE_ARGS="-GNinja -DLLAMA_HIPBLAS=ON -DLLAMA_AVX2=on " pip install --no-cache-dir .
cd ../.. 
```


### Models 
Models : If you're new to this - new models can be downloaded from the shell via a python script, or from a form in the interface. There are lots of them - http://huggingface.co Generally the GPTQ models by TheBloke are likely to load... https://huggingface.co/TheBloke The 30B/33B models will load on 24GB of VRAM, but may error, or run out of memory depending on usage and parameters.
Worthy of mention, TurboDerp ( author of the exllama loaders ) has been posting exllamav2 ( exl2 ) processed versions of models - https://huggingface.co/turboderp ( for use with exllamav2 loader ) - when downloading, note the --branch option.

To get new models note the ~/text-generation-webui directory has a program " download-model.py " that is made for downloading models from HuggingFace's collection.  

If you have old models,  link pre-stored models into the models
```bash
# cd ~/text-generation-webui
# mv models models.1
# ln -s /path/to/models models
```


### Running TGW 

Let's create a script (run.sh) to run the program...
```bash
tee --append run.sh <<EOF
#!/bin/bash
## activate conda
conda activate textgen
## command to run server... 
python server.py --extensions sd_api_pictures send_pictures gallery
# if you want the server to listen on the local network so other machines can access it, add --listen.  
#python server.py --listen  --extensions sd_api_pictures send_pictures gallery 
conda deactivate
EOF
chmod u+x run.sh
```

Note that to run the script : 
```bash
source run.sh
```




## End - Oobabooga - Text-Generation-WebUI

Here's an example, nvtop, sd console, tgw console... 
this screencap taken using ROCm 6.1.3 - under this config : https://github.com/nktice/AMD-AI/blob/main/ROCm-6.1.3-Dev.md
![Alt text](Screenshot-2024-08-17.png)

--------

# nvtop from source
( As one from packages crashes on 2 GPUs, while this never version from sources works fine. ) 
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
# end nvtop
