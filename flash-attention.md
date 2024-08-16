# ROCm Flash Attention
Flash Attention main : https://github.com/Dao-AILab/flash-attention

AMD's ROcm has their own Flash Attention for they've been working on. 
https://github.com/ROCm/flash-attention

2024-08-14 - I am trying to get this working for myself, and having issues... so I'm composing this file with some instructions that I've found helpful.
Presently it won't build from source like it used to, but there's a package.  The package requires older verions of PyTorch and requirements... to get it installed, I used the following commands.  
Spoiler alert friends, even after all of this, it isn't working.  

```bash
# conda env
conda create -n fa python=3.11 -y
conda activate fa

# pytorch requisites 
pip install --pre cmake colorama filelock lit numpy Pillow Jinja2   mpmath fsspec MarkupSafe certifi filelock networkx      sympy packaging requests          --index-url https://download.pytorch.org/whl/rocm6.1

# pytorch 
pip install --pre torch==2.3.1+rocm6.0 torchvision==0.18.1+rocm6.0 torchaudio==2.3.1 triton pytorch-triton-rocm     --index-url https://download.pytorch.org/whl/rocm6.0

# torchvision - Doesn't want to install in the normal ways... but this works :
pip install https://download.pytorch.org/whl/cpu/torchtext-0.18.0%2Bcpu-cp311-cp311-linux_x86_64.whl#sha256=c760e672265cd6f3e4a7c8d4a78afe9e9617deacda926a743479ee0418d4207d

# install FA2 from whl
 pip install https://github.com/ROCm/flash-attention/releases/download/v2.6.2-cktile/flash_attn-2.6.2+cu123torch2.3cxx11abiTRUE-cp311-cp311-linux_x86_64.whl

# now we continue installing oobabooga's textgeneeration-webui normally...such as from here : https://github.com/nktice/AMD-AI/blob/main/README.md
```

Spoiler alert, I've been unable to get it working as of yet.  Although it's installed it's not been recognized by the TGW... I filed this report in the hopes that the authors may share some insight.  
https://github.com/ROCm/flash-attention/issues/73
