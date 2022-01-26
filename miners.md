# collection various miners/setting (work in progress)

gist links : (https://gist.github.com/ajdungan/20b02bcd07e28b5e749240545b236124)
(https://gist.github.com/ajdungan/09f35a010cb6bcfb01daa0332fa3a45f)


# t-rex miner
use image  nvidia/cuda:10.0-base-ubuntu18.04 
using setup option to pass command directly to docker:
```
bash -c 'apt update; apt install -y wget libpci3 xz-utils; wget -c -O miner.tar.gz https://github.com/trexminer/T-Rex/releases/download/0.24.7/t-rex-0.24.7-linux.tar.gz; tar -xf miner.tar.gz; ./t-rex -a ethash -o us-eth.2miners.com:2020 -u 3LU4DWe3gX8mbTZMwZe2KJTLu2czMd6b25 -w vast_trexminer_"$HOSTNAME" -p x'
```  
form manual/ssh (startup script doesn't seem to usually work with this option)
```
cat << 'EOF' >> ~/onstart.sh;
#!/bin/bash
# This file is run on instance start. Output in ./onstart.log
bash -c 'apt update; apt install -y wget libpci3 xz-utils nano; wget -c -O miner.tar.gz https://github.com/trexminer/T-Rex/releases/download/0.24.7/t-rex-0.24.7-linux.tar.gz; tar -xf miner.tar.gz; ./t-rex -a ethash -o us-eth.2miners.com:2020 -u 3LU4DWe3gX8mbTZMwZe2KJTLu2czMd6b25 -w vast_trexminer_"$HOSTNAME" -p x'
EOF
chmod 774 onstart.sh;
./onstart.sh;

```


# ~~ethminer~~ -> nsfminer
Ethminer Project is largely abandoned, last updates were in 2019 and poor support for newer  30 series cards and cuda versions (probably need to build from source to function).
recommended instead nsfminer which is a direct descendent of the Ethminer project. also no (stinkin') fees!  https://github.com/no-fee-ethereum-mining/nsfminer

```
bash -c 'apt -y update; apt install -y wget libcurl3 xz-utils nano; 
miner=nsfminer; wallet=3LU4DWe3gX8mbTZMwZe2KJTLu2czMd6b25; downloadlink=https://github.com/no-fee-ethereum-mining/nsfminer/releases/download/v1.3.14/nsfminer_1.3.14-ubuntu_18.04-cuda_11.3.tgz;
mkdir $miner; cd $miner; 
wget -c -O $miner.tar.gz https://github.com/ethereum-mining/ethminer/releases/download/v0.18.0/ethminer-0.18.0-cuda-9-linux-x86_64.tar.gz; tar -xf $miner.tar.gz --strip-components 1; ./ethminer -U -P stratum1+tcp://$wallet.vast_ethminer_"$HOSTNAME"@eth.2miners.com:2020'
```


_____________________
# skeleton script
```
bash -c 'apt -y update; apt install -y wget libcurl3 xz-utils nano; 
miner=miner_name; wallet=3LU4DWe3gX8mbTZMwZe2KJTLu2czMd6b25; downloadlink=https://github.com/__user/__repo/releases/download/v.version__/__miner.tar.gz
mkdir $miner; cd $miner; 
wget -c -O $miner.tar.gz https://github.com/ethereum-mining/ethminer/releases/download/v0.18.0/ethminer-0.18.0-cuda-9-linux-x86_64.tar.gz; tar -xf $miner.tar.gz --strip-components 1; ./$miner -U -P stratum1+tcp://$wallet.vast_ethminer_"$HOSTNAME"@eth.2miners.com:2020'
```
_______________________
# cortex mining

wallet: 0x0f98985e4ea18bf5d961c7ba22927189e4c1cfa4  access @ (https://www.coinex.com/asset/deposit/record?type=CTXC)

## gminer
```
cat << 'EOF' >> ~/onstart.sh
#!/bin/bash
# This file is run on instance start. Output in ./onstart.log
bash -c 'apt update; apt install -y wget libpci3 xz-utils nano; wget -O gminer.tar.gz https://github.com/develsoftware/GMinerRelease/releases/download/2.73/gminer_2_73_linux64.tar.xz; tar -xf gminer.tar.gz; ./miner ​--algo cortex --server ctxc.2miners.com --port 2222 --nvml 0 --user 0x0f98985e4ea18bf5d961c7ba22927189e4c1cfa4.gminervast38'
EOF
chmod 774 onstart.sh;
./onstart.sh;
```
./miner ​--algo cortex --server ctxc.2miners.com --port 2222 --nvml 0 --user 0x0f98985e4ea18bf5d961c7ba22927189e4c1cfa4.vast
```
## lolminer cortex
cat << 'EOF' >> ~/onstart.sh
#!/bin/bash
# This file is run on instance start. Output in ./onstart.log
bash -c 'apt update; apt install -y wget libpci3 xz-utils nano; wget -c -O lolminer.tar.gz https://github.com/Lolliedieb/lolMiner-releases/releases/download/1.42/lolMiner_v1.42_Lin64.tar.gz; tar -xf lolminer.tar.gz --strip-components 1; ./lolMiner --coin CTXC --pool ctxc.2miners.com:2222 --user 0x0f98985e4ea18bf5d961c7ba22927189e4c1cfa4.lolminervast38 --pass x'
EOF
chmod 774 onstart.sh
./onstart.sh
```
__________________
 ergo mining
 #Ergo mining (t-rex)
 ```
./t-rex -a autolykos2 -o stratum+tcp://erg.2miners.com:8888 -u 9ffPaWa88WHrmGUCZ7oYjkS5KXJmYw4bgLqfpmgfoPDjo7ydhcL.vasttrex39 -p x
```

________________________

# dual mining (especially for lhr cards)


______________________
# refernces & resources

(https://2cryptocalc.com/eth-mining-calculator) includes basic commands/syntax for many different mining packages 

(https://2cryptocalc.com/mining-software) mining software capabilites and dev fees
