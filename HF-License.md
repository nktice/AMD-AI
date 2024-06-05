# How to download Huggingface models that have licenses. 
Huggingface.co has lots of models some of which may have licenses.  As I didn't find a quick set of instructions for how to do it, I have written the following set of commands that have worked for me.


First we install huggingface_hub from pypi - 
```bash 
pip install huggingface_hub
```

Now once you've logged in on Huggingface, you can generate a token here : 
https://huggingface.co/settings/tokens
Once you have that, you can run this command, and paste the token in 
```bash
huggingface-cli login
```

Note that it's helpful to now cd to the directory where you keep your models...
```bash
cd ~/models
```

Now you can go to models on Huggingface that have licenses, accept those licenses, and copy their model code,
then you can download them with the following command.  Be sure to replace $MODEL with the name of the model you've copied. 
```bash
huggingface-cli download $MODEL
```

Alternatively if you want to specify the directory where the model is to be saved...
```bash
huggingface-cli download  --local-dir ~/models $MODEL
```
