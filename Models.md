# Models 

Large Language Models aren't always easy downloads, so here are some clues...

The big collection that's presently popular is huggingface's site ( huggingface.co ). 
Using search tools there, one can find many models and all their details. 

Here is a page with some details on downloading from huggingface's site. 
https://huggingface.co/docs/hub/en/models-downloading

Example : 
```bash
# cd ~/models 
pip install "huggingface_hub[hf_transfer]"
# replace ... here with the user/model from huggingface's url... 
HF_HUB_ENABLE_HF_TRANSFER=1 huggingface-cli download ...
# HF_HUB_ENABLE_HF_TRANSFER=1 huggingface-cli download user/model-name 
```

-----

This page serves as a source of links to relevant models of interest.   Started 2024-03-10
Inspiration : Tom Jobbins aka TheBloke - https://huggingface.co/TheBloke - produced a great many models ready for easy use with Oobabooga.  He appears to be taking a break, with no activity for weeks now...  Looking for new models, I've been forced to look at other sources for the latest releases... So I'm composing this with links to direct sources of models worth watching. 

LLMs... ( no particular order... ) 
- TheBloke https://huggingface.co/TheBloke
- MistralAI - https://mistral.ai/ - https://huggingface.co/mistralai - source of popular Mistral and Mixtral models. 
- Gorilla LLM - https://gorilla.cs.berkeley.edu/ - https://huggingface.co/gorilla-llm - this is a function calling LLM, which as some different architecture and features worth considering.
- NousResearch - https://nousresearch.com/ - https://huggingface.co/NousResearch - Some great models, including variations of other popular models like Mixtral / Mistral.  Includes Hermes ( Nous' ultra-popular uncensored LLM ) and Capybera ( Un-aligned model for general use, leveraging Amplify-Instruct and novel quality curation techniques, made with a dataset of less than 20K examples. )  
- OpenBuddy - https://github.com/OpenBuddy - https://huggingface.co/OpenBuddy - Includes offerings of model variations like Mixtral and Mistral...
- WizardLM - https://huggingface.co/WizardLM
- Code Llama - https://huggingface.co/codellama
- Zain ul Abideen - https://huggingface.co/abideen - Model merger / quantizations ... NexoNimbus  DareVox 
- Cognitive Computations - https://erichartford.com/ - https://huggingface.co/cognitivecomputations - source of Dolphin ( and other ) models...
- Exllama's author Turboderp has been posting exl2 quants - https://huggingface.co/turboderp

Some published models aren't in Safetensors formats and need conversion if they are to work in Oobabooga's UI and other systems.  Here are some links to the related instructions to do conversions...
- PyTorch to Safetensors conversion - https://github.com/Silver267/pytorch-to-safetensor-converter
- Exllamav2's converter - https://github.com/turboderp/exllamav2/blob/master/doc/convert.md

-----

Stable Diffusion / ComfyUI  checkpoints...
- Civit.ai / https://civitai.com/
  
