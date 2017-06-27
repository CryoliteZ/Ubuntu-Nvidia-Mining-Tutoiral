# Ubuntu Nvidia 挖礦懶人包

由於眾多教學文章多為Windows，好像比較少看到Linux的，這邊寫一篇以Ubuntu 16.04為範例給新手看的N卡挖礦中文教學。以下內容將涵蓋

* Driver的安裝
* Cuda的安裝
* 執行miner
* 超頻/節能調整指令

## Nvidia Driver 安裝

1. 可以先去 Nvidia的[網站](http://www.nvidia.com/Download/index.aspx)看適合driver的版本，1060以上多用nvidia-375或是nvidia-381 
2. 將最新的drivers加到PPA
```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update 
``` 
3. 安裝指定的版本（這邊以375當作範例）

```sudo apt install nvidia-375 ```

或是去[Settings]>[Software&Updates]>[Additional Drivers]直接透過GUI安裝

![](https://i.imgur.com/1VpUVA4.png)


## 安裝Cuda

1. 去官方[[網站](https://developer.nvidia.com/cuda-downloads)]下載安裝檔 （Cuda 8.0）

![](https://i.imgur.com/mgIKVRR.png)

> Installer Type建議選deb會比runfile要容易成功xD

2. 切到該目錄下安裝

```
sudo dpkg -i cuda-repo-<distro>_<version>_<architecture>.deb
sudo apt update
sudo apt install cuda
sudo reboot
```

3. 檢查安裝是否成功

```
nvidia-smi 
```
![](https://i.imgur.com/u4m3Qrp.png)

> nvidia-smi查看顯卡和驅動資訊 

```
nvcc --version
```
![](https://i.imgur.com/sbz7c8r.png)

> 查看Cuda compiler version

如果出現 `The program 'nvcc' is currently not installed. You can install it by typing:apt-get install nvidia-cuda-toolkit`的error，有可能是path沒設定好，其解法如下

1. `nano ~/.bashrc` 打開.bashrc
2. 將下面兩行加在最下面
 ```
export PATH=/usr/local/cuda-8.0/bin:$PATH 
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:$LD_LIBRARY_PATH
```
3. `source ~/.bashrc` 就設定完成了

## Miner 介紹

### EWBF for Zcash [[下載連結](https://mega.nz/#F!aop0BLaR!qQUGG6C2ZhE2zAC0XAlMSw)]

N卡比較適合挖zec，EWBF其操作流程跟Windows上都差不多，先去[Poloniex](https://poloniex.com/)或是[Bittrex](https://bittrex.com/)申請錢包。礦池可以選擇[魚池](https://www.f2pool.com/help)、[Nanopool](https://nanopool.org/)、[flypool](https://zcash.flypool.org/)或是[Dwarfpool](https://dwarfpool.com/)。以下用Nanopool當作範例

```
./miner --server zec-asia1.nanopool.org --port 6666 --user t1TRH4eTRZYrKY43A4LMLkkmhnkYFAkHHwt.myworker/myemail --pass z --log 2 --templimit 75 --api --pec
--server 礦池位址
--user 錢包位址.礦工名稱/礦工信箱 (後兩項為optional)
--templimit 溫度上限
--log 產生miner.log,1只有error output，2有console output
--api 預設開在 http://127.0.0.1:42000/ 
--pec 顯示 power usage 跟 Sol/W 
```

![](https://i.imgur.com/6fHWhwi.png)

> zec miner 提供的 API

### Claymore for Ethereum/ Musicoin [[下載連結(linux)](https://drive.google.com/drive/folders/0B69wv2iqszefdFZUV2toUG5HdlU)]

操作方式大同小異。礦池Ethereum可以用[魚池](https://www.f2pool.com/help)，Musicoin可以用[Gpuminepool](http://www.gpumine.org/#/)。下面以挖Musicoin為範例

```
./ethdcrminer64 -epool gpumine.org:8508 -ewal 0x7647f80db8531cbde0562b46494a6bbb9c52ffff -epsw x -allpools 1 -allcoins 1 -eworker miner001
-epool 礦池
-ewal 錢包位址
-eworker 礦工名稱
```

## 超頻/節能調整指令

###  Script
1. 打開任何可以打字文字編輯器，貼上（可自行修改參數）
```
#!/bin/bash
# set power limit
sudo nvidia-smi --power-limit=130
nvidia-smi -i 0 --format=csv --query-gpu=power.limit

# core clock 
nvidia-settings -a "[gpu:0]/GPUGraphicsClockOffset[3]=100"
# memory clock
nvidia-settings -a "[gpu:0]/GPUMemoryTransferRateOffset[3]=350"
# enable fan control
nvidia-settings -a '[gpu:0]/GPUFanControlState=1'
# set fan speed
nvidia-settings -a '[fan:0]/GPUTargetFanSpeed=70'
```
2. 存成 `myscript.sh`，去terminal`chmod+x myscript.sh`開啟可執行權限
3. 執行`./myscript` 就設定完成了！

### 透過  Nvidia Control Center 調整

1. 打開Nvidia control center

![](https://i.imgur.com/3F0k3uV.png)

2. 到nvidia-settings configuration點選 [Save Current Configuration]，會將.nvidia-settings-rc存到Home

3. 打開Terminal，在Home下執行以下指令

```
sudo update-grub 
sudo nvidia-xconfig -a --cool-bits=28 --allow-empty-initial-configuration
reboot
```

4. Reboot後再打開Nvidia control center，就會在PowerMizer看到可以調整overclock的選項了

![](https://i.imgur.com/yA95V8H.png)

> 調整Graphics Clock Offset / Memory Transfer Rate Offset

![](https://i.imgur.com/BLTt3FG.png)

> 在Thermal Settings 可以條整 Fan Speed




## 補充

### 破圖

如果挖礦挖一挖發現UI邊框有**破圖**的情況（通常是休眠後喚醒會發生，但不常見），不用太擔心，那是Nvidia-375的Bug，將驅動更新到Nvidia-381就解決了。如果不想更新，重開機也可以解決問題。如果還是不想重開機，在terminal輸入unity也可以暫時解決問題。

### 瀏覽器
個人經驗挖礦的時候，Firefox會非常卡，使用者體驗極差。也有Chromium使用者說很鈍。可以改用充滿黑科技的 **Chrome** ，應該會順暢不少！



## 小結

筆者在Linux挖起來跟在Windows上算力感覺是差不多的，好像沒有Linux挖比較多的都市傳說xD(反而感覺Windows算力略高一些些) 但是挖起來蠻穩的，系統在操作上也很流暢。這篇算是簡單的教學，供主要工作環境是Linux但是又想同時挖礦的朋友參考。

```
打賞地址 
zec: t1TRH4eTRZYrKY43A4LMLkkmhnkYFAkHHwt
eth: 0xddaed2e6ae80862cc1084a8bc5816a28bbbc97f6
musicoin: 0x7647f80db8531cbde0562b46494a6bbb9c52ffff
```
