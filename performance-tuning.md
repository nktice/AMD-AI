## Performance Tuning 
This optional section is taken from the main [AMD-AI guide](https://github.com/nktice/AMD-AI/)
You can skip this section... altering settings like hardware overclocking must be done with care,
it's 'at your own risk', and your milage may vary, regarding results. 

Here are instructions for turning on control of GPU clocks..
https://www.reddit.com/r/linux_gaming/comments/xm2goe/help_with_overclocking_rx_6600_xt_on_unmodified/
* Step 1: (sudo) Open this file with an editor - [sub for editor of choice...]
  ```bash
  sudo vi /etc/default/grub
  ``` 
* Step 2: Add this Kernel parameter
  ```
  amdgpu.ppfeaturemask=0xffffffff
  ```
  to GRUB_CMDLINE_LINUX_DEFAULT 
* Step 3: Save and run this command install it if you don't have it -
  ```bash
  sudo grub-mkconfig -o /boot/grub/grub.cfg
  ```
Note : Restart is required for this to be effective. Do that when ready. 

### corectrl 
project website : https://gitlab.com/corectrl/corectrl
 for controlling and monitoring graphics hardware settings...
 it's not in the mainline packages, 
 so we need to add a PPA for it... this of course is known
 to have disasterous consequences if done the wrong way.
 so the instructions for how to do this are broke.
 thus we'll do them here. You have been warned...
 more details here : https://gitlab.com/corectrl/corectrl/-/wikis/Setup

```bash
# Protection... avoid other packages from this repo.
sudo tee --append /etc/apt/preferences.d/corectl <<EOF
# Never prefer packages from the ernstp repository
Package: *
Pin: release o=LP-PPA-ernstp-mesarc
Pin-Priority: 1

# Allow upgrading only corectrl from LP-PPA-ernstp-mesarc
Package: corectrl
Pin: release o=LP-PPA-ernstp-mesarc
Pin-Priority: 500
EOF
```

#### Add PPA - https://launchpad.net/~ernstp/+archive/ubuntu/mesarc 
```bash
sudo add-apt-repository -y ppa:ernstp/mesarc
sudo apt update -y
```
#### install corectrl
```bash
sudo apt install -y corectrl
```
#### end corectrl

### End performance tuning 
