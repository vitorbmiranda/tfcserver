# tfcserver

## AWS EC2 Setup (web console)

- Create new EC2 instance (Ubuntu)
- On Security Groups, add the following rules for Inbound

```
SSH 22 0.0.0.0/0
TCP 27010 - 27015 0.0.0.0/0
UDP 27010 - 27015 0.0.0.0/0
```

- Create a new .pem file or use an existing one

## Configuring the server

### Log into the instance 

On AWS Console, go to EC2 page, find your running instance and click on `Connect` to get your ssh command handy, such as:

`ssh -i "tfc-server.pem" ubuntu@ec2-54-232-45-230.sa-east-1.compute.amazonaws.com`

### Set up TFC server

**PS**: assuming a server.cfg is in the user home (`/home/ubuntu` on this case).

```
# 32 bit setup
sudo add-apt-repository multiverse
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install lib32gcc1 -y

# this makes the steamcmd installer skip the license agreement - you'd have to manually press <TAB> and <ENTER> otherwise
echo steamcmd steam/question select "I AGREE" | sudo debconf-set-selections
echo steamcmd steam/license note "" | sudo debconf-set-selections

sudo apt install steamcmd

# sets up the script that will be executed using steamcmd - this way you can run everything at once
echo "login anonymous" >> ~/steamcmd_tfc.txt
echo "force_install_dir ./tfc/" >> ~/steamcmd_tfc.txt
echo "app_set_config 90 mod tfc" >> ~/steamcmd_tfc.txt
echo "app_update 90 validate" >> ~/steamcmd_tfc.txt
echo "app_update 90 validate" >> ~/steamcmd_tfc.txt
echo "app_update 90 validate" >> ~/steamcmd_tfc.txt
echo "app_update 90 validate" >> ~/steamcmd_tfc.txt
echo "quit" >> ~/steamcmd_tfc.txt

# runs the script
# notice that app_update 90 step may fail a few times
steamcmd +runscript ~/steamcmd_tfc.txt

# sets up the app id
echo 90 > /home/ubuntu/.steam/steamcmd/tfc/tfc/steam_appid.txt

# HERE you need your custom server.cfg in the user home
cp ~/server.cfg /home/ubuntu/.steam/steamcmd/tfc/tfc/server.cfg

# Trying to avoid a library lookup error, but it still happens anyway
mkdir ~/.steam/sdk32
ln -s steamcmd/linux32/steamclient.so ~/.steam/sdk32/steamclient.so

# goes to tfc folder
cd /home/ubuntu/.steam/steamcmd/tfc

# runs a screen in detached mode
# NOTE: for some reason using nohup doesn't make the server start properly, but screen works fine
screen -d -m -S tfc ./hlds_run -game tfc -sys_ticrate 1100 -pingboost 3 +map 2fort +maxplayers 32
```
