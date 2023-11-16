## Домашнее задание 3
#Done in YC

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
ssh -i "C:\Users\naoleneva\.ssh\mng" ubuntu@158.160.109.77


#install docker
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && sudo apt-key fingerprint 0EBFCD88 && sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && sudo apt-get update && sudo apt-get install -y docker-ce docker-ce-cli containerd.io

#create docker network
sudo docker network create mongo-net

#add mongo net to MongoDB container
sudo docker run --name mongo-server --network mongo-net -d -p 27017:27017 -v /home/ubuntu/mongo:/data/db -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=otus123 mongo:5.0.4

#get containered Mongo ip
sudo docker inspect mongo-server | grep IPAddress 
172.18.0.2

#run container with docker client
sudo docker run -it --rm --name mongo-client --network mongo-net mongo:5.0.4 mongosh --host 172.18.0.2 -u root -p otus123 --authenticationDatabase admin

#add test data
show databases
use test
db.peoples.insert({'Name':'Mickey'})
db.peoples.find()

#remote connection
mongosh "mongodb://158.160.109.77:27017/" -u root -p otus123 --authenticationDatabase admin

#recreate container
docker ps
docker stop 07a08bc8016e
docker rm 07a08bc8016e
sudo docker run --name mongo-server --network mongo-net -d -p 27017:27017 -v /home/ubuntu/mongo:/data/db -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=otus123 mongo:5.0.4

#check that data are in place
sudo docker run -it --rm --name mongo-client --network mongo-net mongo:5.0.4 mongosh --host 172.18.0.2 -u root -p otus123 --authenticationDatabase admin
use test
db.peoples.find()
