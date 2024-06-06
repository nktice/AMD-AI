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
then you can download them with the following command.  Be sure to replace $MODEL with the name of the model you've copied. Note that $MODELNAME is what you want to call the folder for it to be in after download. ( If you 'just download' it hides it in some cache files that are hard to find...  be careful with the local-dir as it needs the full path of what you want it called, omitting the directory/model name will just plop it in the directory leading to a mess of files. )
```bash
huggingface-cli download  --local-dir ~/models/$MODELNAME $MODEL
```
