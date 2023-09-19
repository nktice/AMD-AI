# How to install and run Flowise on Ubuntu (22.04/23.04)
project website : https://github.com/FlowiseAI/Flowise

The following are my notes from installing flowise (2023-08) - nktice : https://github.com/nktice

# localAI
project site : https://localai.io/basics/build/

Depends on go, but the current distro version is insuffieient... Here is Go's website with install instructions - https://go.dev/doc/install
```bash
cd ~/Downloads/
wget https://go.dev/dl/go1.21.1.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.21.1.linux-amd64.tar.gz 
export PATH=$PATH:/usr/local/go/bin
export PATH=$PATH:/usr/local/go/bin >> ~/.profile
go version
```

```bash
cd
git clone https://github.com/go-skynet/LocalAI.git
cd LocalAI

sudo apt install -y libclblast1 libclblast-dev
sudo apt install -y rocm-opencl rocm-opencl-dev rocm-opencl-runtime rocm-opencl-sdk rocm-clang-ocl 
#sudo apt install -y golang-go
make BUILD_TYPE=clblas build
```

Here is where you'd like to link your stash of models...
```bash
# mv models models.1
# ln -s /path/to/models models 
```

```bash
./local-ai
```

# nodejs
```bash
sudo apt install -y npm
sudo npm install n -g
sudo n latest
```
# yarn
```bash
sudo npm i -g yarn
```

# Flowise
```bash
git clone https://github.com/FlowiseAI/Flowise.git
cd Flowise
yarn install
yarn build
yarn start
```


