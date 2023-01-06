---
title: "搭建SOLANA集群"
date: 2022-08-11T13:26:57+08:00
draft: true
---

# 搭建SOLANA集群

启动solana集群需要四个程序,分别是faucet,genesis,keygen,validator.   
其中,genesis负责创建创世块,faucet负责提供代币的空投,keygen用来生成用户账户,validator提供链的基础服务.

## 安装solana
```bash
sh -c "$(curl -sSfL https://release.solana.com/v1.10.32/install)"
```

## 安装CUDA
```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID | sed -e 's/\.//g')
wget https://developer.download.nvidia.com/compute/cuda/repos/$distribution/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
#Update the APT repository cache and install the driver using the cuda-drivers meta-package. Use the --no-install-recommends option for a lean driver install without any dependencies on X packages. This is particularly useful for headless installations on cloud instances.
sudo apt-get update
sudo apt-get -y install cuda-drivers
```
⚠️: 如果你的电脑开启了secure boot,那么安装时会让你设置secure boot的密码,在重启后,你会进入蓝色界面的“Perform MOK Management”,这里选择 “Enroll Key”后,填写你设置的密码,continue boot的话nvidia-smi会找不到显卡的,CUDA也不会转.
```bash
uname -m && cat /etc/*release
sudo apt-key del 7fa2af80
# Install the new cuda-keyring package:
wget https://developer.download.nvidia.com/compute/cuda/repos/$distro/$arch/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
sudo apt-get update
#Install CUDA SDK:
#Note: These two commands must be executed separately.
sudo apt-get install cuda # 这里solana我们要安装CUDA10.1,可以先search后安装指定版本
#To include all GDS packages:
sudo apt-get install nvidia-gds
```

## 操作系统调优

solana有一个守护进程用以自动化优化系统,每次升级后可以让其以守护的形式运行在后台.
```
sudo $(command -v solana-sys-tuner) --user $(whoami) > sys-tuner.log 2>&1 &
```
### 或者你也可以选择如下的手动调优
Optimize sysctl knobs
```
sudo bash -c "cat >/etc/sysctl.d/21-solana-validator.conf <<EOF
# Increase UDP buffer sizes UDP缓冲
net.core.rmem_default = 134217728
net.core.rmem_max = 134217728
net.core.wmem_default = 134217728
net.core.wmem_max = 134217728

# Increase memory mapped files limit 内存映射文件限制
vm.max_map_count = 1000000

# Increase number of allowed open file descriptors 最大文件描述符
fs.nr_open = 1000000
EOF"
```
`sudo sysctl -p /etc/sysctl.d/21-solana-validator.conf`

Increase systemd and session file limits#
Add
```
LimitNOFILE=1000000
```
to the [Service] section of your systemd service file, if you use one, otherwise add
```
DefaultLimitNOFILE=1000000
```
to the [Manager] section of /etc/systemd/system.conf.
```
sudo systemctl daemon-reload
```

增加流程文件描述器的计数上限
```
sudo bash -c "cat >/etc/security/limits.d/90-solana-nofiles.conf <<EOF
# Increase process file descriptor count limit
* - nofile 1000000
EOF"
```
重启或者重新登陆   
开放防火墙
```
sudo ufw allow 8000:8013/tcp
sudo ufw allow 8000:8013/udp
sudo ufw allow 8899
sudo ufw allow 8900
```

## 开辟创世,启动最基础服务
这里见 [run.sh](run.sh) 和 [fetch-spl.sh](fetch-spl.sh)
## 验证器启动
```bash
solana-keygen new -o ~/vote-account-keypair.json
solana create-vote-account ~/vote-account-keypair.json ~/validator-keypair.json ~/authorized-withdrawer-keypair.json
```
```bash
solana-validator  --identity validator-keypair.json \ 
    --vote-account vote-account-keypair.json \
    --trusted-validator 6D3nasDwfvkEvWbNP1b5dkc8S7buytqhyRNjUbcAJUjD \ # 已有且相信的节点的验证器密钥
    --known-validator 6D3nasDwfvkEvWbNP1b5dkc8S7buytqhyRNjUbcAJUjD \ # 已有节点的验证器密钥
    --only-known-rpc \
    --ledger ~/validator-ledger \ # 存储文件所在位置
    --gosip-host 84.11.2.23 \ # 本机节点的ip,gossip广播会用到
    --gossip-port 8001 \
    --full-rpc-api \
    --rpc-port 8899 \
    --dynamic-port-range 8000-8013 \
    --entrypoint 47.95.28.45:8001 \ # 已有节点的网络地址
    --expected-genesis-hash 4cq4jta2YjDHSHkxV5rPqycpfZMpV7f3cravqcPm1NSV \ #创世区块的哈希
    --wal-recovery-mode skip_any_corrupted_record \
    --limit-ledger-size 50000000 \ #Keep this amount of shreds in root slots. 存储shreds的数量限制
    --cuda \ # 如果配置了CUDA加速,开启CUDA
    --log - 
```
## 自动重启和监控
自动重启: 建议将进程设置为一个`systemd service`.

监控:
```
solana-watchtower --validator-identity <YOUR VALIDATOR IDENTITY>
```
当solana节点unhealthy的时候,可以通过直接通过slack/telegram/discord等直接通知.

## 单节点性能测试参考
[网友提供](https://forums.solana.com/t/benchmark-performance-testing-aka-individual-time-trials/744/7)  

> ## Ryzen 3700X,16GB RAM, 1TB SSD Nvme, 1x NVIDIA 2080TI 
```
Only CPU:

Average max TPS: 43961.43, 0 nodes had 0 TPS
Highest TPS: 43961.43 sampling period 1s max transactions: 3392569 clients: 1 drop rate: 0.76
Average TPS: 37507.06

With GPU

Average max TPS: 114793.67, 0 nodes had 0 TPS
Highest TPS: 114793.67 sampling period 1s max transactions: 7336420 clients: 1 drop rate: 0.57
Average TPS: 80683.016

```

> ## VMware running on 1x Intel® Core™ i7-4770 CPU @ 3.40GHz
> ## Virtual machine has 24GB of RAM. 8 cores.
> ## VM is running on Raid 5 10GB NFS Shared Storage using 3x Intel SSD S3700
> ## NVIDIA T4 GPU Enabled
> ## Local Network
```
bench-tps output:
Sampler 76259.93 TPS, Transactions: 76305, Total transactions: 9468865 over 301 s
Max TPS | Total Transactions
76259.93 | 9468865

Average TPS: 31374.154
```


> ### 2x Intel Xeon E5-2620v4 @ 2.1Ghz (8 core/16 HT, 20MB cache each CPU)
> ### 64 GB RAM (8x 8GB DD4 2133)
> ### 4x 300GB 10K SAS HDD in RAID 5
> ### OS & Account drive separate LVM LVs in same PV
> ### No GPUs
> ### TdS
```
With default trace level logging on validator
[2020-08-17T18:55:16.858430309Z INFO solana_bench_tps::bench] Average max TPS: 95511.38, 0 nodes had 0 TPS
[2020-08-17T18:55:16.858445036Z INFO solana_bench_tps::bench] Highest TPS: 95511.38 sampling period 1s max transactions: 4197121 clients: 1 drop rate: 0.60
[2020-08-17T18:55:16.858465790Z INFO solana_bench_tps::bench] Average TPS: 46093.523

With RUST_LOG=warn on validator
[2020-08-17T19:12:00.660520328Z INFO solana_bench_tps::bench] Average max TPS: 103514.71, 0 nodes had 0 TPS
[2020-08-17T19:12:00.660528112Z INFO solana_bench_tps::bench] Highest TPS: 103514.71 sampling period 1s max transactions: 4583238 clients: 1 drop rate: 0.55
[2020-08-17T19:12:00.660540872Z INFO solana_bench_tps::bench] Average TPS: 50689.96

Average max TPS: 228k
```

> ## https://www.amd.com/es/products/cpu/amd-epyc-7502p 9
> ## NVme SSD
```
2020-08-21T00:26:09.227006768Z INFO
solana bench tps::bench
Average max TPS: 228234.86, © nodes had © TPS
2020-08-21T00:26:09.227011116Z INFO solan bench ps: : bench]
Highest TPS: 228234.86 sampling period 1s max transactions: 12364998 clients: 1 drop rate: 0.51
2020-08-21T00:26:09.227016256Z INFO
solana bench tos::bench]
Average TPS: 136086.31
```

> ## Dedicated Root Server SB145
> ## * AMD EPYC 7401P
> ## * 2x SSD U.2 NVMe 960 GB Datacenter
> ## * 4x RAM 32768 MB DDR4 ECC reg.
> ## * NIC 1 Gbit
> ## * - Intel I350
> ## * Location: HEL1
```
Node address | Max TPS | Total Transactions
---------------------±--------------±-------------------
127.0.0.1:8003 | 74717.75 | 4796383

Average max TPS: 74717.75, 0 nodes had 0 TPS
[2020-08-28T09:51:14.265313651Z INFO solana_bench_tps::bench]
Highest TPS: 74717.75 sampling period 1s max transactions: 4796383 clients: 1 drop rate: 0.74
[2020-08-28T09:51:14.265320651Z INFO solana_bench_tps::bench] Average TPS: 52885.4
```

自测
> ## Intel(R) Xeon(R) CPU E3-1230 v5 @ 3.40GHz / 16G DDR4 / GTX980Ti 6G
```bash
Sampler   5506.08 TPS, Transactions:   5508, Total 
 Token balance: 1001781760
  Node address        |       Max TPS | Total Transactions
 ---------------------+---------------+--------------------
 127.0.0.1:8003       |      11962.19 | 616063 
 
 616063 clients: 1 drop rate: 0.92
 	Average TPS: 6742.964
 datapoint: bench-tps-lamport_balance balance=1001781760i
```

solana CUDA bench mark ubuntu 20.04
```bash
atapoint: bench-tps-lamport_balance balance=1001781760i
oken balance: 1001781760
Node address        |       Max TPS | Total Transactions
--------------------+---------------+--------------------
atapoint: bench-tps-lamport_balance balance=1001781760i
27.0.0.1:8003       |       9723.19 | 514874 

    Average max TPS: 9723.19, 0 nodes had 0 TPS
    Highest TPS: 9723.19 sampling period 1s max transactions: 514874 clients: 1 drop rate: 0.94
	Average TPS: 5692.3276
```

> ## 8核 Intel(R) Xeon(R) Gold 6240 CPU @ 2.60GHz 32G + 1V00 jiutian.10086.cn
```
[2022-08-03T08:08:44.287651184Z INFO  solana_bench_tps::bench] Waiting for sampler threads...
[2022-08-03T08:08:44.475730394Z INFO  solana_bench_tps::bench] Token balance: 497201305426094000
[2022-08-03T08:08:44.475793960Z INFO  solana_metrics::metrics] datapoint: bench-tps-lamport_balance balance=497201305426094000i
[2022-08-03T08:08:45.202427001Z INFO  solana_bench_tps::perf_utils] Sampler    732.68 TPS, Transactions:    733, Total transactions: 594636 over 96 s
[2022-08-03T08:08:45.202563725Z INFO  solana_bench_tps::bench] Waiting for transmit threads...
[2022-08-03T08:08:45.202597542Z INFO  solana_bench_tps::bench] Waiting for blockhash thread...
[2022-08-03T08:08:45.202948931Z INFO  solana_bench_tps::bench] Token balance: 497201305426094000
[2022-08-03T08:08:45.202970550Z INFO  solana_bench_tps::bench]  Node address        |       Max TPS | Total Transactions
[2022-08-03T08:08:45.202976974Z INFO  solana_bench_tps::bench] ---------------------+---------------+--------------------
[2022-08-03T08:08:45.202983157Z INFO  solana_bench_tps::bench] http://127.0.0.1:8899 |       9654.77 | 594636 
[2022-08-03T08:08:45.202994788Z INFO  solana_bench_tps::bench] 
    Average max TPS: 9654.77, 0 nodes had 0 TPS
[2022-08-03T08:08:45.202980061Z INFO  solana_metrics::metrics] datapoint: bench-tps-lamport_balance balance=497201305426094000i
[2022-08-03T08:08:45.203016071Z INFO  solana_bench_tps::bench] 
    Highest TPS: 9654.77 sampling period 1s max transactions: 594636 clients: 1 drop rate: 0.01
[2022-08-03T08:08:45.203064076Z INFO  solana_bench_tps::bench]  Average TPS: 6188.3247
root@dl-220803122842849-pod-jupyter-55dcdf5d77-46v4c:~# 
```

## solana 建议配置
### solana 验证器硬件建议
#### CPU
* 12 核/ 24线程 或者更多
* 2.8Ghz 或者更快
* 支持 AVX2 指令集
* 支持 AVX512f 或和 SHA-NI指令集更佳
#### RAM
* 128GB 内存或更多
* 推荐主板能支持到256GB的内存
#### Disk
* PCIe Gen3 x4 NVME SSD 或者更快
* 账户存储: 500GB容量或者更大, 高TBW
* 账本储存: 1TB 容量或者更大,高TBW
* 推荐OS / 账户 / 账本分盘存储
#### GPUs
* 目前非必须
* 结合目前性能测试来看,性能瓶颈主要为CPU

### RPC 节点硬件推荐
#### CPU
* 16核/32线程或者更多
#### RAM
* 256GB 或者更多
#### Disk
* 如果要存储交易历史记录,建议使用较大的磁盘
* 账户和账单不能存储在相同的磁盘上

### 软件
* ubuntu 20.04
### 网络
* 各个节点之间有300Mbit/s的带宽,推荐有1GBit/s
* 8000 - 10000 TCP/UDP P2P端口, 这些可以被限制在任何13个端口中.
* 8899 8900 TCP端口
### GPU
ubuntu 20.4, CUDA 10.1 update 1
## 参考:
https://www.2331314.xyz/276.html
https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html


http://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git

https://docs.solana.com/integrations/exchange