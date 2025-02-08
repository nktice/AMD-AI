# Ollama for Aider, and CrewAI
2024-08-21 - Here's a supplement for Aider with Ollama install instructions.  
In order, Conda, Ollama, LiteLLM, and last it's Aider - 
there are some issues I note for all these sysstems, but get them running.   
This is the first draft, after getting to to work, and may revise more.  
2024-08-22 - I have now added details about CrewAI... added at bottom. 

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
sudo apt install -y curl
curl -fsSL https://ollama.com/install.sh | sh
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
sudo mkdir /home/ollama/models
sudo mkdir /home/ollama/tmp
sudo chown -R ollama:ollama /home/ollama
chmod 775 -R /home/ollama
```

Edit systemd service config 
```bash
sudo systemctl edit ollama.service
```

In this you will add to the section `[Service]` like the following : 
```
[Service]
Environment="OLLAMA_MODELS=/home/ollama/models"
Environment="TMPDIR=/home/ollama/tmp" 
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

#### Converting models...

Here's an example for MistralAi's Mistral-Small-24B-Base-2501
```
cd deepseekai_DeepSeek-R1-Distill-Qwen-14B
echo "FROM ." > modelfile 
create dsr1q14b -f modelfile 
```
This then created a new model to access through the interface...
```
ollama list
ollama run dsr1q14b
```
That responds to queries.


#### More... 

That is the basics of preparing Ollama, to get more info...
```
ollama help
```


I have had issues after a system restart with it not starting...  in the system logs there were messages about like the following : `ollama.service: Service has more than one ExecStart= setting, which is only allowed for Type=oneshot services. Refusing.` I found that I had to move /etc/systemd/system/ollama.service.d/override.conf and it worked again.  Another option, if the daemon doesn't start, is manual start - opening a shell, and running the following commands at the shell... 
```bash
## manual loading of ollama if needed ... 
# conda activate ollama
# ollama serve 
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


## CrewAI 
CrewAI is an agent based AI framework that uses python.  
Here is their website : https://www.crewai.com/  
Here is their github : https://github.com/crewAIInc/crewAI

CrewAI can use Ollama as we've installed earlier / above.  
Here is some info on how to use CrewAI calling Ollama
https://docs.crewai.com/how-to/LLM-Connections/#connect-crewai-to-llms

The basics of this involve installing python modules as follows...
```bash
## in case it's ont active activate ollama conda env
# conda activate ollama
pip install langchain-ollama
pip install crewai crewai-tools
```

This was sufficient for me to run the test script on CrewAI's site. 
I'll paraphrase here, to streamline people testing, and getting started.
```bash
cd
mkdir crewai
cd crewai
tee --append crewaidemo.py <<EOF
# CrewAI with Ollama demo script
from crewai import Agent, Task, Crew
from langchain_ollama import ChatOllama
import os
os.environ["OPENAI_API_KEY"] = "NA"

llm = ChatOllama(
    model = "llama3.1:70b",
    base_url = "http://localhost:11434")

general_agent = Agent(role = "Math Professor",
                      goal = """Provide the solution to the students that are asking mathematical questions and give them the answer.""",
                      backstory = """You are an excellent math professor that likes to solve math questions in a way that everyone can understand your solution""",
                      allow_delegation = False,
                      verbose = True,
                      llm = llm)

task = Task(description="""what is 3 + 5""",
             agent = general_agent,
             expected_output="A numerical answer.")

crew = Crew(
            agents=[general_agent],
            tasks=[task],
            verbose=True
        )

result = crew.kickoff()

print(result)
EOF
# now test that it's working...
python crewaidemo.py
```
Nothe the selection of the model may vary based on your setup.  For me with it pulled, the script functions properly, and demonstrates work. 
