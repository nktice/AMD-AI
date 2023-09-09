# How to install and run Flowise on Ubuntu (22.04/23.04)
project website : https://github.com/FlowiseAI/Flowise

The following are my notes from installing flowise (2023-08) - nktice : https://github.com/nktice

# localAI
project site : https://localai.io/basics/build/
```bash
cd
git clone https://github.com/go-skynet/LocalAI.git
cd LocalAI

sudo apt install libclblast1 libclblast-dev
sudo apt install -y golang-go
make BUILD_TYPE=clblas build
```

# here is where you'd like to link your stash of models...

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


