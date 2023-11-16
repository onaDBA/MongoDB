## Домашнее задание 2
#Done in YC

#generate keys
ssh-keygen -t ed25519

#install cli for windows
iex (New-Object System.Net.WebClient).DownloadString('https://storage.yandexcloud.net/yandexcloud-yc/install.ps1')
yc init

#create vpc in yc
yc vpc network create \
  --name my-yc-network \
  --labels my-label=my-value \
  --description "NataNet"

#create subnet
yc vpc subnet create --name default-ru-central1-a --network-name my-yc-network --zone 'ru-central1-a' --range 192.168.1.0/24

#create VM
  yc compute instance create \
  --name mongo2023-21071993 \
  --hostname mongo2023-21071993 \
  --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts \
  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
  --zone ru-central1-a \
  --metadata-from-file ssh-keys="/c/Users/naoleneva/.ssh/mng.txt"

#check VM
yc compute instance get mongo2023-21071993
yc compute instance get --full mongo2023-21071993

#connect to VM
ssh -i "C:\Users\naoleneva\.ssh\mng" ubuntu@158.160.109.44

#install mongo
wget -qO - https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -  && echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt-get install -y mongodb-org

#create mongo directory and start mongod with defaults
sudo mkdir /home/mongo && sudo mkdir /home/mongo/db1 && sudo chmod 777 /home/mongo/db1
mongod --dbpath /home/mongo/db1 --port 27001 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid

#connect to mongo
mongosh --port 27001

show databases
use test
db.peoples.insert({'Name':'Mickey'})
db.peoples.find()

#configure external access
use admin;
db.createUser( { user: "naoleneva", pwd: "*********", roles: [ "userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase" ] } )
ps -xf
#bind_ip_all & --auth
kill 2176
mongod --dbpath /home/mongo/db1 --port 27001 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid --bind_ip_all --auth

#test remote connect
mongosh "mongodb://158.160.109.44:27001/" -u naoleneva -p ********* --authenticationDatabase admin