topic: https://devtalk.nvidia.com/default/topic/1055434/jetson-nano/after-i2cset-test-nano-can-t-boot/1



because the eeprom checksum failed, nvboot or cboot run failed.

use the rescue image and do update command:

```
cd ~/nvidia/nvidia_sdk/JetPack_4.2_Linux_P3448/Linux_for_Tegra
sudo ./flash.sh jetson-nano-qspi-sd mmcblk0p1
```



