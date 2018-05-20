## 修改两个地址源

```
sudo nano /etc/apt/sources.list
```

```
deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ stretch main non-free contrib
deb-src http://mirrors.ustc.edu.cn/raspbian/raspbian/ stretch main non-free contrib
```


```
sudo nano /etc/apt/sources.list.d/raspi.list
```


```
deb http://mirrors.ustc.edu.cn/archive.raspberrypi.org/debian/ stretch main ui
```


**解决的问题是：**

 * 1、HomeAssistant的安装
 * 2、Samba的设置
 * 3、MQTT的安装

 
## 一、HomeAssistant的安装

首先改一下sudoer设置，省得sudo时总要求输密码

```
sudo nano /etc/sudoers
```

在最下面（看好，是此文件的最下方，否则会被下面的设置所覆盖而无效！）添加以下内容（xxxxxxxx改为你的用户名）：

```
xxxxxxxx ALL=NOPASSWD: ALL

```

可选项，如果在安装ubuntu时没有更改时区的，使用下面的代码更改时区

```
sudo dpkg-reconfigure tzdata

```

必选项！更换国内源！如果不更换，sudo apt-get update会非常慢
首先是备份原源地址，然后将虚线间的代码加入sources.list，contrl + x, y 退出


```
sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak
sudo nano /etc/apt/sources.list
```

```
--------------------------------------------------------------------------------------------------
deb http://mirrors.aliyun.com/ubuntu/ yakkety main restricted
deb http://mirrors.aliyun.com/ubuntu/ yakkety-updates main restricted
deb http://mirrors.aliyun.com/ubuntu/ yakkety universe
deb http://mirrors.aliyun.com/ubuntu/ yakkety-updates universe
deb http://mirrors.aliyun.com/ubuntu/ yakkety multiverse
deb http://mirrors.aliyun.com/ubuntu/ yakkety-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ yakkety-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ yakkety-security main restricted
deb http://mirrors.aliyun.com/ubuntu/ yakkety-security universe
deb http://mirrors.aliyun.com/ubuntu/ yakkety-security multiverse
--------------------------------------------------------------------------------------------------
```

更新源信息，安装更新


```
sudo apt-get update 
sudo apt-get upgrade -y
```

做一些清理工作，安装python3，默认应该是已经安装的


```
sudo apt-get autoclean
sudo apt-get clean
sudo apt-get purge -y python3-pip
sudo apt-get install python3
```

安装PIP

```
wget https://bootstrap.pypa.io/get-pip.py
sudo python3 ./get-pip.py
sudo apt-get install python3-pip 
```

## 安装Python3虚拟环境


```
sudo apt-get install python3-venv

```
添加一个名为homeassistant的用户

```
sudo useradd -rm homeassistant

```

转到/srv目录，建立homeassistant文件夹

```
cd /srv
sudo mkdir homeassistant

```

更改此文件夹的所有者和所属组

```
sudo chown homeassistant:homeassistant homeassistant
```

更换用户

```
sudo su -s /bin/bash homeassistant
```

切换目录，创建并进入虚拟环境

```
cd /srv/homeassistant
python3 -m venv homeassistant_venv
source /srv/homeassistant/homeassistant_venv/bin/activate
```

虚拟环境下安装pip

```
pip install --upgrade pip

```
安装依赖netdisco，理论上直接默认安装即可，但有的Hass版本需要指定1.0.0rc3，则按下面的命令输入

```
pip3 install netdisco
pip3 install netdisco==1.0.0rc3

```
正式安装HomeAssistant，速度会非常快


```
pip3 install homeassistant

```
安装完毕，退出虚拟环境

```
exit
```

设置开机启动，建立service文件，将#中间的部分拷入，按ctrl + x, y 退出。

```
sudo nano /etc/systemd/system/home-assistant@homeassistant.service

```
####################################################################

```
[Unit]
Description=Home Assistant
After=network.target

[Service]
Type=simple
User=homeassistant
Environment=PATH="$VIRTUAL_ENV/bin:$PATH"
ExecStart=/srv/homeassistant/homeassistant_venv/bin/hass -c "/home/homeassistant/.homeassistant"

[Install]
WantedBy=multi-user.target

```
####################################################################

更新系统设置

```
sudo systemctl daemon-reload
```

设置HomeAssistant开机启动

```
sudo systemctl enable home-assistant@homeassistant.service

```

启动HomeAssistant

```
sudo systemctl start home-assistant@homeassistant.service

```
重新启动HomeAssistant

```
sudo systemctl restart home-assistant@homeassistant.service

```
查看HomeAssistant状态

```
sudo systemctl status home-assistant@homeassistant.service
```

编辑SAMBA配置文件
## 二、Samba的设置

#### 设置SAMBA共享路径、编辑SAMBA配置文件

```
sudo nano /etc/samba/smb.conf
```

#### 在文件最后加入

```
##############
[HOME ASSISTANT]
path = /home/homeassistant/.homeassistant
comment = No comment
browsable = yes
read only = no
valid users =
writable = yes
guest ok = yes
public = yes
create mask = 0777
directory mask = 0777
force user = root
force create mode = 0777
force directory mode = 0777
hosts allow =
########################

```

### 重启SAMBA服务

```
sudo service smbd restart
```

三、MQTT的安装

##安装MQTT服务

#### 安装3个MAKE所需的工具

```
编译找不到openssl/ssl.h
sudo apt-get install libssl-dev

编译过程找不到ares.h
sudo apt-get install libc-ares-dev

编译过程找不到uuid/uuid.h
sudo apt-get install uuid-dev
```

**安装**

```
sudo apt-get install mosquitto
```


#### 开启服务

```
sudo systemctl start mosquitto

```
#### 查看服务状态

```
sudo systemctl status mosquitto
```


#####################################################################


#####################################################################


```
sudo nano /etc/systemd/system/homebridge.service
```	

```
cd /
sudo useradd --system homebridge
sudo mkdir /var/homebridge
sudo cp ~/.homebridge/config.json /var/homebridge/
sudo cp -r ~/.homebridge/persist /var/homebridge
sudo chmod -R 0777 /var/homebridge
cd /etc/default
```


```
sudo nano homebridge
```

复制粘贴

```
# Defaults / Configuration options for homebridge
# The following settings tells homebridge where to find the config.json file and where to persist the data (i.e. pairing and others)
HOMEBRIDGE_OPTS=-U /var/homebridge
# If you uncomment the following line, homebridge will log more 
# You can display this via systemd's journalctl: journalctl -f -u homebridge
# DEBUG=*
```

ctrl+x,y,回车

```
cd /etc/systemd/system
sudo nano homebridge.service
```

复制粘贴

```
[Unit]
Description=Node.js HomeKit Server 
After=syslog.target network-online.target
[Service]
Type=simple
User=homebridge
EnvironmentFile=/etc/default/homebridge
ExecStart=/usr/lib/node_modules/homebridge/bin/homebridge $HOMEBRIDGE_OPTS
Restart=on-failure
RestartSec=10
KillMode=process
[Install]
WantedBy=multi-user.target
```

ctrl+x,y,回车

```
sudo systemctl daemon-reload
sudo systemctl enable homebridge.service
sudo systemctl restart homebridge.service
sudo systemctl status homebridge.service

```

```
homebridge -U /home/pi/homeassistant/.homebridge

```

```
sudo nano /etc/systemd/system/mopidy-mpd.service
sudo systemctl daemon-reload
sudo systemctl enable mopidy-mpd.service
sudo systemctl start mopidy-mpd.service
```


```
[Unit]
Description=Home Mopidy
After=network.target

[Service]
Type=simple
User=homeassistant
ExecStart=/usr/bin/mopidy --config /home/pi/.config/mopidy/

[Install]
WantedBy=multi-user.target

```

## github 备份配置

```
  git config --global user.email "fushenghua2010@gmail.com"
  git config --global user.name "Silver"
```


