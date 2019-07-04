i2c-tool由这几个工具组成：i2cdetect, i2cdump, i2cget, i2cset



(1)i2cdetect -l 得到所有总线

	root@nv-desktop:~# i2cdetect -l
	i2c-3	i2c       	7000c700.i2c                    	I2C adapter
	i2c-1	i2c       	7000c400.i2c                    	I2C adapter
	i2c-6	i2c       	Tegra I2C adapter               	I2C adapter
	i2c-4	i2c       	7000d000.i2c                    	I2C adapter
	i2c-2	i2c       	7000c500.i2c                    	I2C adapter
	i2c-0	i2c       	7000c000.i2c                    	I2C adapter
	i2c-5	i2c       	7000d100.i2c                    	I2C adapter



(2)

i2cdetect -r -y 6

i2cdetect -r -y 2

发现：总线6上由设备6a；总线2上有  50 和  57 

总线2上的50是个e2prom，存储mac地址



(3)读取一系列地址（比如nano的mac地址）

i2cdump  -f -y 2 0x50

i2cdump -y -r 172-177 2 0x50 b

i2cdump -y -r 0xac-0xb1 2 0x50 b



(4)i2cget和i2cset

i2cset -f -y 2 0x50 0xac 0x3f （设置i2c-2上0x50器件的0xac寄存器值为0x3f）

i2cset -f -y 2 0x50 0xac 0xc2 （设置i2c-2上0x50器件的0xac寄存器值为0x3f）

i2cget -f -y 2 0x50 0xac     （读取i2c-2上0x50器件的0xac寄存器值）