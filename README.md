# vasttools

The aim is to setup a list of usble tools that can be used with vastai.
The tools are free to use, modify and distribute. 

## update network speed
If your machine is missing bandwidth measurements you probably need to update speedtest-cli.  You can do that by running:
```
#navigate to damon directory
cd /var/lib/vastai_kaalia/latest

#download and run speedtest-cli
sudo wget -O speedtest-cli https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py;chmod +x speedtest-cli;
./speedtest-cli
```

## prevent upgrades on docker and nvidia drivers

Upgrading versions of docker and nvidia drivers after vast has been installed is known to cause issues. 
To prevent upgrades run:
```
sudo apt-mark hold `sudo apt list --installed *nvidia* | awk '{split($0, a, "/"); print a[1]}'`
sudo apt-mark hold `sudo apt list --installed *docker* | awk '{split($0, a, "/"); print a[1]}'`
```

you can check the status of which packages are being froze from upgrades by running
```
sudo apt-mark showhold
```


## Backgorund mining job 

use imnage  nvidia/cuda:10.0-base-ubuntu18.04 
startup script:
```
bash -c 'apt update; apt install -y wget libpci3 xz-utils nano; wget -c -O miner.tar.gz https://github.com/trexminer/T-Rex/releases/download/0.24.7/t-rex-0.24.7-linux.tar.gz; tar -xf miner.tar.gz; ./t-rex -a ethash -o us-eth.2miners.com:2020 -u 3LU4DWe3gX8mbTZMwZe2KJTLu2czMd6b25 -w ajdworkerNAME -p x'

```  

```
cat << 'EOF' >> ~/onstart.sh
#!/bin/bash
# This file is run on instance start. Output in ./onstart.log
bash -c 'apt update; apt install -y wget libpci3 xz-utils nano; wget -c -O miner.tar.gz https://github.com/trexminer/T-Rex/releases/download/0.24.7/t-rex-0.24.7-linux.tar.gz; tar -xf miner.tar.gz; ./t-rex -a ethash -o us-eth.2miners.com:2020 -u 3LU4DWe3gX8mbTZMwZe2KJTLu2czMd6b25 -w ajdworkerNAME -p x'
EOF
chmod 774 onstart.sh
./onstart.sh
```

## change bid for multiple gpu machine

single line command, example is for a 3 gpu machine; must declare first (lowest) instance id number 

```
price=0.35; machine1=2129035; ./vast change bid $machine1 --price $price;./vast change bid $(($machine1 + 1)) --price $price; ./vast change bid $(($machine1 + 2)) --price $price;
```

## Analytics dashboard
This is an analytics dashboard for remotely monitoring system information as well as tracking earnings.
https://github.com/jjziets/vast.ai-tools/blob/master/analytics

## Memory oc

set the OC of the RTX 3090
It requires the folliwing

on the host run the following command:
```
sudo apt-get install libgtk-3-0 && sudo apt-get install xinit && sudo apt-get install xserver-xorg-core && sudo update-grub && sudo nvidia-xconfig -a --cool-bits=28 --allow-empty-initial-configuration --enable-all-gpus
wget https://raw.githubusercontent.com/jjziets/vasttools/main/set_mem.sh
sudo chmod +x set_mem.sh
sudo ./set_mem.sh 2000 # this will set the memory OC to +1000mhs on all the gpus. You can use 3000 on some gpu's which will give 1500mhs OC. 
```

## OC monitor
setup the monitoring programe that will change the memory oc based on what programe is running. it desinged for RTX3090's and targets ethminer at this stage.
It requires both set_mem.sh and ocmonitor.sh to run in the root.

```
wget https://raw.githubusercontent.com/jjziets/vasttools/main/ocminitor.sh
sudo chmod +x ocminitor.sh
sudo ./ocminitor.sh # I suggest running this in tmux or screen so that when you close the ssh connetion. It looks for ethminer and if it finds it it will set the oc based on your choice. you can also set powerlimits with nvidia-smi -pl 350 
```

Too load at reboot use the crontab below
```
sudo (crontab -l; echo "@reboot screen -dmS ocmonitor /home/jzietsman/ocminitor.sh") | crontab -  #replace the user with your user
```

## Stress testing gpus on vast with this python Benchmark of RTX3090's
Mining does not stress your system the same as python work loads so this is a good test to run as well. 
https://github.com/jjziets/pytorch-benchmark-volta

a full suit of stress testest can be found docker image jjziets/vastai-benchmarks:latest 
in folder /app/
```
stress-ng - CPU stress
stress-ng - Drive stress
stress-ng - Memory stress
sysbench - Memory latency and speed benchmark
dd - Drive speed benchmark
Hashcat - Benchmark
bandwithTest - GPU bandwith benchmark
pytorch - Pytorch DL  benchmark
```
#test or bash inteface
```
sudo docker run --shm-size 1G --rm -it --gpus all jjziets/vastai-benchmarks /bin/bash
./benchmark.sh
```
#Run using default settings
Results are saved to ./output.

```
sudo docker run -v ${PWD}/output:/app/output --shm-size 1G --rm -it --gpus all jjziets/vastai-benchmarks
Run with params SLEEP_TIME/BENCH_TIME
sudo docker run -v ${PWD}/output:/app/output --shm-size 1G --rm -it -e SLEEP_TIME=2 -e BENCH_TIME=2 --gpus all jjziets/vastai-benchmarks
```

*based on leona / vast.ai-tools

## Auto update the price for host listing based on mining porfits.

based on RTX 3090 120Mhs for eth. it sets the price of my 2 host. 
it works with a custom Vast-cli which can be found here https://github.com/jjziets/vast-python/blob/master/vast.py
The manager is here https://github.com/jjziets/vasttools/blob/main/setprice.sh

This should be run on a vps not on a host. do not expose your Vast API keys by using it on the host.
```
wget https://github.com/jjziets/vast-python/blob/master/vast.py 
sudo chmod +x vast.py
./vast.py  set api-key UseYourVasset
wget https://github.com/jjziets/vasttools/blob/main/setprice.sh
sudo chmod +x setprice.sh
```

## useful commands
show devices (GPUs) running at 16x/8x
```
cat /var/log/Xorg.0.log | grep "16x"
cat /var/log/Xorg.0.log | grep "8x"
```

Check if GPUS are in persistence mode

`usr/bin/nvidia-smi -q | grep -i Persistence`

Enable persistence mode
```
nvidia-smi -pm 1
```

## useful utilities

Glances - install script in ajdungan's systemsetup repo

nvyop - terminal based htop-tyled monitoring for GPUs.  
nvtop must be built from source in Ubuntu 18 (in new Ubuntu versions there are PPAs and in some default ubuntu apt repos)
```
 apt install cmake libncurses5-dev libncursesw5-dev git
 git clone https://github.com/Syllo/nvtop.git
 mkdir -p nvtop/build && cd nvtop/build
 cmake ..
 make
 make install
 ```
 
stacer - favorite gui system viewer
```
sudo add-apt-repository ppa:oguzhaninan/stacer
sudo apt-get update
sudo apt install stacer -y
```


# attribution
forked from jjziets @ https://github.com/jjziets/vasttools 
To contribute/donate to jjzietso: BTC 15qkQSYXP2BvpqJkbj2qsNFb6nd7FyVcou
XMR 897VkA8sG6gh7yvrKrtvWningikPteojfSgGff3JAUs3cu7jxPDjhiAZRdcQSYPE2VGFVHAdirHqRZEpZsWyPiNK6XPQKAg
