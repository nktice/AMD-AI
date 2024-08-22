# Ollama and Aider
2024-08-21 - Here's a supplement for Aider with Ollama install instructions.  
In order, Conda, Ollama, LiteLLM, and last it's Aider - 
there are some issues I note for all these sysstems, but get them running.   
This is the first draft, after getting to to work, and may revise more.  

This guide is based on the first part of the guide as a foundation. 
https://github.com/nktice/AMD-AI 
It will likely work after the with ROCm drivers installed under Ubuntu, after the reboot, so if you've not done those parts, please go do them first. 


## Conda
First we'll need Conda ... this will keep python's installs and it's packages versioned together, as some versions may conflict and cause frustrations. 
This is similar to instructions in guide, skip parts if needed. 
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

## Ollama
Ollama provides a model loading service... Here's the project's page : https://github.com/ollama/ollama
Surprisingly it now has support for ROCm ( as of version 6... ) 
Linux install guide : https://github.com/ollama/ollama/blob/main/docs/linux.md

### setup a conda environment...
```bash
conda create -n ollama python=3.11 -y
conda activate ollama
```

### Install Ollama
```bash
pip install ollama
```

### Move where models go...
Here's a video 'where does Ollama store models?' - about it's storage : https://www.youtube.com/watch?v=6bF1uCHTFyk
By default Ollama likes to put it's models in /usr/share/ollama/.ollama 
If you want to move where ollama stores its files, that is an issue... here's a support thread : https://github.com/ollama/ollama/pull/897
How to place models also gets some discusion in Ollama's FAQ : https://github.com/ollama/ollama/blob/main/docs/faq.md

Make home directory...
```bash
# Ollama's install, because it runs as a daemon / service, makes it's own user...
sudo mkdir /home/ollama
sudo chown -R ollama:ollama /home/ollama
```

Edit systemd service config 
```bash
sudo systemctl edit ollama.service
```

In this you will add to the section `[Service]` like the following : 
```
[Service]
Environment="OLLAMA_MODELS=/home/ollama"
```
This lets Ollama know that is where you want it to put its models. 

Now use systemd to restart Ollama. 
```bash
sudo systemctl daemon-reload
sudo systemctl restart ollama
```
Note that these commands for managing Ollama through systemctl may be useful to consult if you have issues or want to manage. 
As it's a service, it does include notes in logs in `/var/log/syslog` 

### Getting models 
Here is a list of Ollama's models : https://ollama.com/library
To download a model with thier system we can use the following 
```bash
# ollama pull <name-of-model:size>
# ollama pull codellama:70b
```

That is the basics of preparing Ollama, to get more info...
```
ollama help
```

## LiteLLM
LiteLLM is the API Aider likes to communicate.  As such it functions to interface with models, via Ollama.
Here's their github page : https://github.com/BerriAI/litellm
It appears that my issue that inspired this part have now been resolved : https://github.com/paul-gauthier/aider/issues/1140 - so this section is now kept for reference, but remarked out. 

### Installation

Theoretically you can simply install it using pip...
```bash
## conda activate ollama
#pip install litellm
```

Or if you prefer, it can be installed from sources, from their github site.
```bash
# conda activate ollama
# cd
# git clone https://github.com/BerriAI/litellm.git
# cd litellm
# pip install -r requirements.txt
# pip install . 
```

Automatically works with a range of models pulled with Ollama.  Unfortunately LiteLLM gave warnings using some models, noting that it didn't know cost for tokens, but how it does this, was interrupting the flow of using Aider.  I have however figured out a workaround, it is as follows.

### LiteLLM config
Here's an example litellm config file...
```bash
# cd
# tee --append litellm.config.yaml <<EOF
# model_list:
#  - model_name: deepseek-coder:33b 
#    litellm_params:
#      model: ollama/deepseek-coder:33b 
#      input_cost_per_second: 0
#  - model_name: deepseek-coder-v2:16b
#    litellm_params:
#      model: ollama/deepseek-coder-v2:16b
#      input_cost_per_second: 0
#EOF
```

### Running LiteLLM 
Of course you can choose other models you may have using such format.  
And now to run it... 
```bash
## conda activate ollama 
# litellm -c litellm.config.yaml
```
Then LiteLLM will be will be running with config from specified file.  Now we can run Aider, it'll use LiteLLM as we've configured.



## Aider 
Aider is a tool that is used for programming - here is the github : https://github.com/paul-gauthier/aider
Here is their website : http://aider.chat 
Here's their leaderboards covering best LLMs for use with Aider - https://aider.chat/docs/leaderboards/


To get it installed 
```bash
# in case we're not in Ollama's conda environment... 
conda activate ollama

## To install from pip / packages...
# pip install aider-chat
## or if you prefer sources...
git clone --recurse-submodules https://github.com/paul-gauthier/aider.git
cd aider
pip install -r requirements.txt
pip install . 
```
That should get Aider ( aka aider-chat ) installed on your system. 

For running Aider...
```bash
## if you don't have conda running ... 
# conda activate ollama
## starting Aider... you can load model once it's going too... and you will likely want other options, this is just a start...
# aider --model ollama:codellama:70b
```
