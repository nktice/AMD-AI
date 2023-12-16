# Mixtral on Oobabooga [ ROCm ]
Mixtral is a model from Mistral AI - more info here : https://www.youtube.com/watch?v=WjiX3lCnwUI

## AutoGPTQ
2023-12-15 - AutoGPTQ has added support for Mixtral models ( GPTQ only ) 
Here is the github page for AutoGPTQ where there's more details. https://github.com/PanQiWei/AutoGPTQ - If you need other architectures that's the place to look for what to do...


Activate conda environment
```bash
conda activate textgen
```

Search for and uninstall old llama-cpp-python packages
```bash
pip list | grep auto
```

Remove if needed...
```bash
pip uninstall auto-gptq
```

Install Auto-GPTQ...
```bash
## ROCm 5.6.1
# pip install auto-gptq --extra-index-url https://huggingface.github.io/autogptq-index/whl/rocm561/
# ROCm 5.7.1
pip install auto-gptq --extra-index-url https://huggingface.github.io/autogptq-index/whl/rocm571/
```

Now run textgen webui as normal, select the Auto-GPTQ loader on the models tab...  This appears to work wihout trust-remote-code.  
[ Triton isn't supported for ROCm apparently... ] 
AutoGPTQ is much faster than the llama.cpp loader below... ( using the GPU for most of it's work, and keeping the model loaded ).  This also means that one needs to be careful to not load a model that is too large, as loading oversized models can lead to a system lock (crash) - be sure to save work.  
As such I have found this model works as it's smaller than the others...
- TheBloke_mixtralnt-4x7b-test-GPTQ - loaded it uses 14+ GB  ( on 24GB 7900XTX ), with an increase in memory usage during queries.
https://huggingface.co/TheBloke/mixtralnt-4x7b-test-GPTQ 

-------------


## llama.cpp 
2023-12-13 - As of writing there is not standard support for loading it built
 into Oobabooga, but there is a workaround...
https://www.reddit.com/r/Oobabooga/comments/18gijyx/simple_tutorial_using_mixtra
l_8x7b_gguf_in_ooba/
As the workaround takes adaptation to work, I've composed this guide.


Activate conda environment
```bash
conda activate textgen
```

Search for and uninstall old llama-cpp-python packages
```bash
pip list | grep llama
```

look for anything starting with llama_cpp_python and uninstall all
```bash
pip uninstall llama_cpp_python
pip uninstall llama_cpp_python_cuda
```

Get llama.cpp from : 
https://github.com/ggerganov/llama.cpp
Includes instructions for how to install it - so if you're looking for how to compile for other architectures, that's where to look.  

```bash
# Go to repositories folder
cd ~/text-generation-webui/repositories
# Clone llama-cpp-python into repositories,
git clone https://github.com/abetlen/llama-cpp-python
cd llama-cpp-python/vendor
# move old llama.cpp,
mv llama.cpp llama.cpp.1
# then clone Mixtral branch into vendor
git clone https://github.com/ggerganov/llama.cpp.git
# cd make llama.cpp for ROCm
cd llama.cpp
# note that it's this line that you'd change for other architectures... 
make LLAMA_HIPBLAS=1
# install
cd ../..
python -m pip install -e .
cd ../..
```

Once you have done this, the llama.cpp for mixtral should be installed, and show
 up here...
```bash
pip list | grep llama
```

Using this you can now use the mixtral _GGUF_ models with Oobabooga ( use llama.cpp loader ) ... 
It does appear able to load models that are larger than my video card - although it also appears that it only uses the GPU a little, and most of the processing is done on CPU.  
I've tested with these ( with Sally Riddle answers... https://github.com/nktice/AMD-AI/blob/main/SallyAIRiddle.md ) : 
- https://huggingface.co/TheBloke/Mixtral-8x7B-v0.1-GGUF [ pass / pass ]
- https://huggingface.co/TheBloke/dolphin-2.5-mixtral-8x7b-GGUF [ pass / pass ]
- https://huggingface.co/TheBloke/openbuddy-mixtral-8x7b-v15.1-GGUF [ pass / pass ] 
- https://huggingface.co/TheBloke/Mixtral-8x7B-Instruct-v0.1-GGUF [ fail / pass ]  
- https://huggingface.co/TheBloke/mixtralnt-4x7b-test-GGUF  [ fail / pass ]
- https://huggingface.co/TheBloke/Mixtral-SlimOrca-8x7B-GGUF [ fail / pass ]
- https://huggingface.co/TheBloke/Synthia-MoE-v3-Mixtral-8x7B-GGUF [ fail / pass ]
- https://huggingface.co/TheBloke/Mixtral-8x7B-MoE-RP-Story-GGUF [ fail / pass ]

