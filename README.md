# vasttools

The aim is to setup a list of usble tools that can be used for vast.ai hosts (not renters/clients)
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

## Prevent upgrades on docker and nvidia drivers

Upgrading versions of docker and nvidia drivers after vast has been installed is known to cause issues. 
To prevent upgrades run:
```
sudo apt-mark hold `sudo apt list --installed *nvidia* 2>/dev/null|awk '{split($0, a, "/");if(NR>1)print a[1]}'`
sudo apt-mark hold `sudo apt list --installed *docker* 2>/dev/null|awk '{split($0, a, "/");if(NR>1)print a[1]}'`
```

Coding notes:  *2>/dev/null* gets rid of pesky warning ("WARNING: apt does not have a stable CLI interface. Use with caution in scripts." Warning is standard error output, ie channel 2, and not standard, so the command will actually functionally properly without getting rid of it. Getting rid of it was just asthetic.

*if(NR>1)* is to skip the first line of the output from *apt list*. This first line of output is *loading...* and including it in our *apt-mark hold* command it will be read literally as regex expression which ends up identifying the package libfile-listing-perl to be held, which isn't wanted. 


you can check the status of which packages are being froze from upgrades by running
```
sudo apt-mark showhold
```
------

## Backgorund mining job/client

### Trex miner/ethereum
use image  nvidia/cuda:10.0-base-ubuntu18.04 
using setup option to pass command directly to docker:
```
bash -c 'apt update; apt install -y wget libpci3 xz-utils nano; echo export id="$(cat ~/.vast_containerlabel)"| cat >> /etc/environment; source /etc/environment; wget -c -O miner.tar.gz https://github.com/trexminer/T-Rex/releases/download/0.24.8/t-rex-0.24.8-linux.tar.gz; tar -xf miner.tar.gz; ./t-rex -a ethash -o us-eth.2miners.com:2020 -u 3LU4DWe3gX8mbTZMwZe2KJTLu2czMd6b25 -w vast_trexminer_"$id" -p x'
```  
from manual cli/ssh (startup script doesn't seem to usually work with this option)
```
cat << 'EOF' >> ~/onstart.sh;
#!/bin/bash
# This file is run on instance start. Output in ./onstart.log
bash -c 'apt update; apt install -y wget libpci3 xz-utils nano; echo export id="$(cat ~/.vast_containerlabel)"| cat >> /etc/environment; source /etc/environment; wget -c -O miner.tar.gz https://github.com/trexminer/T-Rex/releases/download/0.24.8/t-rex-0.24.8-linux.tar.gz; tar -xf miner.tar.gz; ./t-rex -a ethash -o us-eth.2miners.com:2020 -u 3LU4DWe3gX8mbTZMwZe2KJTLu2czMd6b25 -w vast_trexminer_"$id" -p x'
EOF
chmod 774 onstart.sh;
./onstart.sh;

```

### Set container label as a global variable
For naming a mining "worker" among other things it is advantageous to set the intance id (container label) as a global variable. 
Note in the CLI the container label appears as the machine id however it just an alias; `hostname` or `echo $HOSTNAME' commands will show an entirely different ID which is just random letters and numbers
```
echo export id="$(cat ~/.vast_containerlabel)"| cat >> /etc/environment; source /etc/environment;
```
________________

## Vast-cli commands

### change bid for multiple gpu machines

Single line command to change interruptable bid for background job, as it is often advantageous to change the bid price on all gpus on a machine. You must declare the variable 'machine1' with the first (i.e. lowest) instance id number (unique instance id numbers are in sequential order). Note: currently background job creates a single instance for each system gpu rather than a single instance with multiple GPUs

For 4 gpu machine
```
price=0.35; machine1=2129035; ./vast change bid $machine1 --price $price;./vast change bid $(($machine1 + 1)) --price $price; ./vast change bid $(($machine1 + 2)) --price $price; ./vast change bid $(($machine1 + 3)) --price $price; 
```
For 3 GPU machine
```
price=0.35; machine1=2129035; ./vast change bid $machine1 --price $price;./vast change bid $(($machine1 + 1)) --price $price; ./vast change bid $(($machine1 + 2)) --price $price;
```
For 2 GPU machine
```
price=0.35; machine1=2129035; ./vast change bid $machine1 --price $price;./vast change bid $(($machine1 + 1)) --price $price; 
```

### list current rentals on-demand and bid/interreptable
./vast show machines | grep -e current_rentals_running_on_demand -e gpu_name -e hostname -e mobo_name -e current_rentals_running


_________________
## Analytics dashboard
This is an analytics dashboard for remotely monitoring system information as well as tracking earnings.
https://github.com/jjziets/vast.ai-tools/blob/master/analytics


_________________
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
## useful utilities

### Glances 
Install script in ajdungan's [systemsetup repo] (https://github.com/ajdungan/systemsetup), it is helpful to add shortcut on your phone which essentially becomes an app to monitor your system including GPU temps

### nvtop
nvtop - terminal based htop-tyled monitoring for GPUs.  

nvtop must be built from source in Ubuntu 18 (in new Ubuntu versions there are PPAs and in some default ubuntu apt repos)
```
 apt install cmake libncurses5-dev libncursesw5-dev git -y
 cd/opt
 git clone https://github.com/Syllo/nvtop.git
 mkdir -p nvtop/build && cd nvtop/build
 cmake ..
 make
 make install
 ```
### nfancurve

nfancurve is a small and lightweight script for setting a custom fan curve, default curv in repo's conf has more agressive fan speeds than nvidia defaults (higher noise/lower temps) but is gihgly configurable.
(https://github.com/nan0s7/nfancurve)

### telegram bot - offline machine notification

TBD


### s-tui
Terminal based cpu monitoring and stress testing. nice visualizations
https://github.com/amanusk/s-tui

### android phone - connect bot - SSH terminal
nice ssh terminal for your phone 
dowload available from f-droid

### stacer - favorite gui system viewer
```
sudo add-apt-repository ppa:oguzhaninan/stacer
sudo apt-get update
sudo apt install stacer -y
```

### OC monitor
setup the monitoring programe that will change the memory oc based on what program is running. Currently configured for RTX3090's and targets ethminer at this stage.
It requires both set_mem.sh and ocmonitor.sh to run in the root.

```
wget https://raw.githubusercontent.com/jjziets/vasttools/main/ocminitor.sh
sudo chmod +x ocminitor.sh
sudo ./ocminitor.sh # I suggest running this in tmux or screen so that when you close the ssh connetion. It looks for ethminer and if it finds it it will set the oc based on your choice. you can also set powerlimits with nvidia-smi -pl 350 
```

To load at reboot use the crontab below
```
sudo (crontab -l; echo "@reboot screen -dmS ocmonitor /home/jzietsman/ocminitor.sh") | crontab -  #replace the user with your user
```

_________________
## Stress testing
Stress testing gpus on vast with this python Benchmark of RTX3090's
Mining does not stress your system the same as python work loads so this is a good test to run as well. 
https://github.com/jjziets/pytorch-benchmark-volta

A full suit of stress testest can be found docker image jjziets/vastai-benchmarks:latest 
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
*based on leona / vast.ai-tools*
____
## additional stress testing/benchmarking

'https://github.com/500farm/3090-burnin' 
_____________________
## adjusting fan speeds
Poorly programmed on the part of nvidia, very messy with fan sppeds being tied to X-sessions and Xorg configs. Easiet way to manually adjust fan speed (fixed only) is to be logged in an x session and open the nvidia-server application, but his is very limitted and can't be done in sshterminal

Nfancurve is a nice tool.

Arch wiki has helpful info (https://wiki.archlinux.org/title/NVIDIA/Tips_and_tricks#Set_fan_speed_at_login)


### setting fans speeds if you have headless system.
I have two scripts that you can use to set the fan speeds of all the gpus. Single or Dual fans use https://github.com/jjziets/test/blob/master/cool_gpu.sh and 
tripple fans use https://github.com/jjziets/test/blob/master/cool_gpu2.sh

to use run this command
```
sudo apt-get install libgtk-3-0 && sudo apt-get install xinit && sudo apt-get install xserver-xorg-core && sudo update-grub && sudo nvidia-xconfig -a 
--cool-bits=28 --allow-empty-initial-configuration --enable-all-gpus
wget https://raw.githubusercontent.com/jjziets/test/master/cool_gpu.sh
or wget https://raw.githubusercontent.com/jjziets/test/master/cool_gpu2.sh
sudo chmod +x cool_gpu.sh
sudo ./cool_gpu.sh 100 # this sets the fans to 100%
```


## Auto update the price for host listing based on mining profits.

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

Enable persistence mode
```
nvidia-smi -pm 1
```
Command on host that provides logs of the deamon running 
```
tail /var/lib/vastai_kaalia/kaalia.log -f 
```


## attribution
Forked from jjziets @ https://github.com/jjziets/vasttools 
To contribute/donate to jjzietso: BTC 15qkQSYXP2BvpqJkbj2qsNFb6nd7FyVcou
XMR 897VkA8sG6gh7yvrKrtvWningikPteojfSgGff3JAUs3cu7jxPDjhiAZRdcQSYPE2VGFVHAdirHqRZEpZsWyPiNK6XPQKAg
To contribute/donate to ajdungan: BTC 3LU4DWe3gX8mbTZMwZe2KJTLu2czMd6b25
