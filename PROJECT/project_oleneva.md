#deploy for single vm
  yc compute instance create \
  --name mongonao1 \
  --hostname mongonao1 \
  --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts \
  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
  --zone ru-central1-a \
  --metadata-from-file ssh-keys="/c/Users/naoleneva/.ssh/mng.txt"
  

#deploy VMs
for i in {1..4} 
do  yc compute instance create  --name mongonao$i   --hostname mongonao$i   --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts   --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4   --zone ru-central1-a   --metadata-from-file ssh-keys="/c/Users/naoleneva/.ssh/mng.txt"  
done


#check vms
yc compute instance list
+----------------------+------------+---------------+---------+----------------+--------------+
|          ID          |    NAME    |    ZONE ID    | STATUS  |  EXTERNAL IP   | INTERNAL IP  |
+----------------------+------------+---------------+---------+----------------+--------------+
| fhm2beho2cgcnvmpjmc5 | mongonao3  | ru-central1-a | RUNNING | 84.201.175.43  | 192.168.1.15 |
| fhmen8ueig05ug5lpdno | mongonao2  | ru-central1-a | RUNNING | 84.201.173.60  | 192.168.1.6  |
| fhmfm7epniv5vcgh0lmo | mongonao4  | ru-central1-a | RUNNING | 158.160.42.235 | 192.168.1.16 |
| fhmmtcedot2bduehi77m | mongonao1  | ru-central1-a | RUNNING | 178.154.203.34 | 192.168.1.10 |
| fhmqg3joireqf5e32pcd | mongonao01 | ru-central1-a | STOPPED |                | 192.168.1.4  |
+----------------------+------------+---------------+---------+----------------+--------------+

#allow ssh
ssh-keygen
ssh-copy-id ubuntu@mongonao2
ssh-copy-id ubuntu@mongonao3
ssh-copy-id ubuntu@mongonao4

#install percona mongo & backup 1-4

sshpass -p nao ssh -p 22 -i "C:\Users\naoleneva\.ssh\mng" ubuntu@mongonao1 
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-60 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb

sshpass -p nao ssh -p 22 -i "C:\Users\naoleneva\.ssh\mng" ubuntu@mongonao2
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-60 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb

sshpass -p nao ssh -p 22 -i "C:\Users\naoleneva\.ssh\mng" ubuntu@mongonao3
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-60 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb

sshpass -p nao ssh -p 22 -i "C:\Users\naoleneva\.ssh\mng" ubuntu@mongonao4
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo  wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb && sudo dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb && sudo percona-release enable psmdb-60 release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-server-mongodb && sudo percona-release enable pbm release && sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y percona-backup-mongodb

#check installation (percona mongodb starts by default) 1-4
ps -aef | grep mongo | grep -v grep
ssh -p 22 ubuntu@mongonao1 'ps -aef | grep mongo | grep -v grep'
ssh -p 22 ubuntu@mongonao2 'ps -aef | grep mongo | grep -v grep'
ssh -p 22 ubuntu@mongonao3 'ps -aef | grep mongo | grep -v grep'
ssh -p 22 ubuntu@mongonao4 'ps -aef | grep mongo | grep -v grep'


for i in {1..4}
do ssh -p 22 ubuntu@mongonao$i "hostname; ps -aef | grep mongo | grep -v grep"
done

#stop mongo service localy
ps -aef | grep mongo | grep -v grep |awk {'print $2'} | sudo xargs kill -9

#stop mongo service 1-4
for i in {1..4}
do ssh -p 22 ubuntu@mongonao$i "hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print \$2}' | sudo xargs kill -9"
done

#create folders 1-4
for i in {1..4}
do ssh -p 22 ubuntu@mongonao$i "hostname; sudo rm -rf /home/mongo && sudo mkdir -p /home/mongo/{dbc1,db1,db2,db3} && sudo chmod 777 /home/mongo/{dbc1,db1,db2,db3}"
done

#check existing folders
for i in {1..4}
do ssh -p 22 ubuntu@mongonao$i "ls -la /home/mongo"
done

#run config servers 1-3
#ok
for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i "sudo mongod --configsvr --bind_ip '0.0.0.0' --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid"
done

#check
for i in {1..4}
do ssh -p 22 ubuntu@mongonao$i "hostname; ps -aef | grep mongo | grep -v grep"
done


						netstat -tlpn

#initiate config srv on 1 (config server doensn't have an arbitr)
sudo su 
mongosh --port 27001 

rs.initiate({"_id" : "RScfg", configsvr: true, members : [{"_id" : 0, host : "mongonao1:27001"},{"_id" : 1, host : "mongonao2:27001"},{"_id" : 2, host : "mongonao3:27001"}]});

						#rs.add( { host: "mongonao1:27001" } )

use admin
db.createUser({user: "UserClusterAdmin",pwd: "nao", roles: [ "clusterAdmin" ]})
--Владелец всех БД
db.createUser({user: "UserdbOwner",pwd: "nao", roles: [ { role: "dbOwner", db: "*" } ]})
-- Супер ROOT
db.createUser({user: "UserRoot",pwd: "nao", roles: [ "root" ]})

#stop mongo service 1-3
#ok
for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i "hostname; sudo ps -aef | grep mongo | grep -v grep |awk '{print \$2}' | sudo xargs kill -9"
done

#try run mongo with auth
for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i "hostname; mongod --auth --configsvr --bind_ip --bind_ip 0.0.0.0 --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid"
done
						-- BadValue: security.keyFile is required when authorization is enabled with replica sets
						
#create folders for mongo-security
for i in {1..4}; do gcloud compute ssh mongo$i --command='sudo mkdir /home/mongo/mongo-security && sudo chmod 777 /home/mongo/mongo-security' & done;

for i in {1..4}
do ssh -p 22 ubuntu@mongonao$i "hostname; sudo mkdir /home/mongo/mongo-security && sudo chmod 777 /home/mongo/mongo-security"
done
 
#generate keyfile on 1-st instance
openssl rand -base64 756 > /home/mongo/mongo-security/keyfile
chmod 400 /home/mongo/mongo-security/keyfile

scp /home/mongo/mongo-security/keyfile ubuntu@mongonao2:/home/mongo/mongo-security/keyfile
scp /home/mongo/mongo-security/keyfile ubuntu@mongonao3:/home/mongo/mongo-security/keyfile
scp /home/mongo/mongo-security/keyfile ubuntu@mongonao4:/home/mongo/mongo-security/keyfile

for i in {2..4}
do ssh -p 22 ubuntu@mongonao$i "hostname; chmod 400 /home/mongo/mongo-security/keyfile"
done

#run mongo with auth
for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i "hostname; sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --configsvr --bind_ip '0.0.0.0' --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid"
done

#test connection
mongosh --port 27001 -u "UserClusterAdmin" -p nao --authenticationDatabase "admin"


rs.config()
rs.status()

##########################################################################################################################################################

#create replicas on different nodes (no auth)
for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i 'sudo mongod --shardsvr --dbpath /home/mongo/db1 --bind_ip '0.0.0.0' --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/dbrs1.log --pidfilepath /home/mongo/db1/dbrs1.pid;'
done

for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i 'sudo mongod --shardsvr --dbpath /home/mongo/db2 --bind_ip '0.0.0.0' --port 27021 --replSet RS2 --fork --logpath /home/mongo/db2/dbrs2.log --pidfilepath /home/mongo/db2/dbrs2.pid;'
done

for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i 'sudo mongod --shardsvr --dbpath /home/mongo/db3 --bind_ip '0.0.0.0' --port 27031 --replSet RS3 --fork --logpath /home/mongo/db3/dbrs3.log --pidfilepath /home/mongo/db3/dbrs3.pid;'
done

							##kill incorrect mongo pocesses on specific ports netstat -tlpn
							for i in {1..3}
							do ssh -p 22 ubuntu@mongonao$i "hostname; sudo ps -aef | grep  27011| grep -v grep |awk '{print \$2}' | sudo xargs kill -9"
							done

							for i in {1..3}
							do ssh -p 22 ubuntu@mongonao$i "hostname; ps -aef | grep  27021| grep -v grep |awk '{print \$2}' | sudo xargs kill -9"
							done

							for i in {1..3}
							do ssh -p 22 ubuntu@mongonao$i "hostname; ps -aef | grep  27031| grep -v grep |awk '{print \$2}' | sudo xargs kill -9"
							done


#create replicasets on mastert nodes
mongosh --host mongonao1 --port 27011
rs.initiate({"_id" : "RS1", members : [{"_id" : 0, priority : 3, host : "mongonao1:27011"},{"_id" : 1, host : "mongonao2:27011"},{"_id" : 2, host : "mongonao3:27011"}]});
use admin
db.createUser({user: "UserDBAdmin",pwd: "nao", roles: [ { role: "dbAdmin", db: "*" } ]})
db.createUser({user: "UserDBRoot",pwd: "nao", roles: [ "root" ]})

mongosh --host mongonao2 --port 27021
rs.initiate({"_id" : "RS2", members : [{"_id" : 0, host : "mongonao1:27021"},{"_id" : 1, priority : 3, host : "mongonao2:27021"},{"_id" : 2, host : "mongonao3:27021"}]});
use admin
db.createUser({user: "UserDBAdmin",pwd: "nao", roles: [ { role: "dbAdmin", db: "*" } ]})
db.createUser({user: "UserDBRoot",pwd: "nao", roles: [ "root" ]})

mongosh --host mongonao3 --port 27031
rs.initiate({"_id" : "RS3", members : [{"_id" : 0, host : "mongonao1:27031"},{"_id" : 1, host : "mongonao2:27031"},{"_id" : 2, priority : 3, host : "mongonao3:27031"}]});
use admin
db.createUser({user: "UserDBAdmin",pwd: "nao", roles: [ { role: "dbAdmin", db: "*" } ]})
db.createUser({user: "UserDBRoot",pwd: "nao", roles: [ "root" ]})

#stop replicas with data
for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i "ps -aef | grep shardsvr | grep -v grep | awk '{print \$2}'| sudo xargs kill -9"
done

#start replicas with auth
for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i 'sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db1 --bind_ip '0.0.0.0' --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/dbrs1.log --pidfilepath /home/mongo/db1/dbrs1.pid;'
done

mongosh --port 27011 -u "UserDBRoot" -p nao --authenticationDatabase "admin"

for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i 'sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db2 --bind_ip '0.0.0.0' --port 27021 --replSet RS2 --fork --logpath /home/mongo/db2/dbrs2.log --pidfilepath /home/mongo/db2/dbrs2.pid;'
done

mongosh --port 27021 -u "UserDBRoot" -p nao --authenticationDatabase "admin"

for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i 'sudo mongod --auth --keyFile /home/mongo/mongo-security/keyfile --shardsvr --dbpath /home/mongo/db3 --bind_ip '0.0.0.0' --port 27031 --replSet RS3 --fork --logpath /home/mongo/db3/dbrs3.log --pidfilepath /home/mongo/db3/dbrs3.pid;'
done

mongosh --port 27011 -u "UserDBRoot" -p nao --authenticationDatabase "admin"

###############################################################################################################################################################################

#create mongos with auth
for i in 3 4
do ssh -p 22 ubuntu@mongonao$i 'sudo mkdir -p /home/mongo/dbms && sudo chmod 777 /home/mongo/dbms'
done

for i in 3 4
do ssh -p 22 ubuntu@mongonao$i 'mongos --keyFile /home/mongo/mongo-security/keyfile --configdb RScfg/mongonao1:27001,mongonao2:27001,mongonao3:27001 --bind_ip '0.0.0.0' --port 27000 --fork --logpath /home/mongo/dbms/dbs.log --pidfilepath /home/mongo/dbms/dbs.pid'
done

#####sharded cluster is ready


mongosh --port 27000 --host mongonao4 -u "UserRoot" -p nao --authenticationDatabase "admin"
db.a.insert({"a":1})

MongoBulkWriteError: Database test could not be created :: caused by :: No shards found
##add shards
use admin
sh.addShard("RS1/mongonao1:27011,mongonao2:27011,mongonao3:27011")
sh.addShard("RS2/mongonao1:27021,mongonao2:27021,mongonao3:27021")
sh.addShard("RS3/mongonao1:27031,mongonao2:27031,mongonao3:27031")
sh.status()

						-- Для того чтобы реплика была доступна на чтение:
						rs.secondaryOk(true)


#get db size
#under ubuntu user
for i in {1..4}
do ssh -p 22 ubuntu@mongonao$i 'hostname; sudo du -m -s /home/mongo/*'
done



ssh mongonao4
#download collection
wget https://dl.dropboxusercontent.com/s/p75zp1karqg6nnn/stocks.zip 
sudo apt install unzip
unzip -qo stocks.zip 
cd dump/stocks/
ls -l

#restore collection
mongorestore --port 27000 -u "UserRoot" -p nao --authenticationDatabase "admin" values.bson

mongosh --port 27000 -u "UserRoot" -p nao --authenticationDatabase "admin"
db.values.find().limit(1)



db.values.count()
show databases


-- запустим долгий запрос
-- db.values.find({$where: '(this.open - this.close > 100)'},{"stock_symbol":1,"open":1,"close":1})
-- через 20 минут надоело) посмотрим загрузку - так как уехало на 2 RS - то его мастер и будет страдать
-- посмотрим план
use test
db.values.find({$where: '(this.open - this.close > 100)'}).explain("executionStats")
-- и его тоже не смогли

use test
db.values.createIndex({stock_symbol: 1})
-- меньше 1 минуты индекс
-- сделал машинки мощнее и теперь 6 секунд 0_0, была e2_small
sh.enableSharding("test")
sh.shardCollection("test.values",{ stock_symbol: 1 })
sh.status()

-- Тоже самое, но через mapreduce.
-- Работает за 10 секунд
var map=function(){
   if (this.open - this.close > 100)  {
    emit(this.stock_symbol, this.open - this.close);
   }
};

var reduce=function(key, values) {
     return Array.sum(values);
};
db.values.mapReduce(map, reduce, {out: 'o' })
db.o.find()

##############################################################################################################################################

#percona dump
pbm

-- https://docs.percona.com/percona-backup-mongodb/initial-setup.html
#add user for every shard and config server
mongosh --port 27001 -u "UserRoot" -p nao --authenticationDatabase "admin"
mongosh --host mongonao1 --port 27011 -u "UserDBRoot" -p nao --authenticationDatabase "admin"
mongosh --host mongonao2 --port 27021 -u "UserDBRoot" -p nao --authenticationDatabase "admin"
mongosh --host mongonao3 --port 27031 -u "UserDBRoot" -p nao --authenticationDatabase "admin"

use admin
db.getSiblingDB("admin").createRole({ "role": "pbmAnyAction",
      "privileges": [
         { "resource": { "anyResource": true },
           "actions": [ "anyAction" ]
         }
      ],
      "roles": []
   });

db.getSiblingDB("admin").createUser({user: "pbmuser",
       "pwd": "secretpwd",
       "roles" : [
          { "db" : "admin", "role" : "readWrite", "collection": "" },
          { "db" : "admin", "role" : "backup" },
          { "db" : "admin", "role" : "clusterMonitor" },
          { "db" : "admin", "role" : "restore" },
          { "db" : "admin", "role" : "pbmAnyAction" }
       ]
    });
exit

#check file
sudo nano /etc/default/pbm-agent


############################################################################################################################################

#check file
sudo nano /etc/default/pbm-agent

#create backup folders
for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i 'sudo mkdir -p /home/mongo/backups && sudo chmod 777 /home/mongo/backups && sudo mkdir -p /home/mongo/pbm && sudo chmod 777 /home/mongo/pbm'
done

#create config file
for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i 'cat > temp.cfg << EOF 
storage:
  type: filesystem
  filesystem:
    path: /home/mongo/backups
EOF
cat temp.cfg | sudo tee -a /home/ubuntu/pbm_config.yaml
' 
done

#apply configs
for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i 'pbm config --file /home/ubuntu/pbm_config.yaml --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27001/?authSource=admin&replicaSet=RScfg"'
done

									-- стандартный путь, описанный в документации как обычно не работает
									-- sudo systemctl start pbm-agent
									-- sudo systemctl start pbm-agent
									-- sudo systemctl status pbm-agent
									-- sudo systemctl stop pbm-agent
									-- journalctl -u pbm-agent.service

ssh mongonao1
export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27001/?authSource=admin&replicaSet=RScfg"
pbm list

#run pbm agent
6mhZ57dK2YDi2Z
for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i 'nohup pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27001/?authSource=admin&replicaSet=RScfg" > /home/mongo/pbm/agent.$(hostname -s).27001.log 2>&1 &'
done;
-- запускаем агент бэкап для остальных реплик
for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i 'nohup pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27011/?authSource=admin&replicaSet=RS1" > /home/mongo/pbm/agent.$(hostname -s).27011.log 2>&1 &'
done
for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i 'nohup pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27021/?authSource=admin&replicaSet=RS2" > /home/mongo/pbm/agent.$(hostname -s).27021.log 2>&1 &'
done
for i in {1..3}
do ssh -p 22 ubuntu@mongonao$i 'nohup pbm-agent --mongodb-uri "mongodb://pbmuser:secretpwd@localhost:27031/?authSource=admin&replicaSet=RS3" > /home/mongo/pbm/agent.$(hostname -s).27031.log 2>&1 &'
done


ps -xf

pbm status
pbm logs
pbm backup 
pbm list



https://docs.percona.com/percona-backup-mongodb/running.html#pbm-restore-new-env

pbm restore --help

								-- для восстановления БД рекомендуется отключить всех юзеров и остановить балансировщик
								https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/#disable-the-balancer
								mongo --host mongo4 --port 27000 -u "UserRoot" -p Otus\$123 --authenticationDatabase "admin"
mongosh --host mongonao4 --port 27000 -u "UserRoot" -p nao --authenticationDatabase "admin"
sh.stopBalancer()
show databases
use test
db.dropDatabase()

						-- disable mongos
						-- for i in {3..4}; do gcloud compute ssh mongo$i --command="hostname; ps -aef | grep mongos | grep -v grep | awk {'print \$2'}| sudo xargs kill -9"; done
						-- on 1
						
						pbm list
						-- pbm restore 2022-01-11T18:06:37Z
#disable mongos					
for i in {3..4}
do ssh -p 22 ubuntu@mongonao$i "hostname; sudo ps -aef | grep mongos | grep -v grep |awk '{print \$2}' | sudo xargs kill -9"
done

						-- !!! все бэкапы должны быть в этом каталоге на всех ВМ !!!


#on mongonao4
6mhZ57dK2YDi2Z
sudo mkdir -p /mnt/download/mongo
sudo scp -r ubuntu@mongonao1:/home/mongo/backups /mnt/download/mongo
sudo scp -r ubuntu@mongonao2:/home/mongo/backups /mnt/download/mongo
sudo scp -r ubuntu@mongonao3:/home/mongo/backups /mnt/download/mongo

scp -r /mnt/download/mongo ubuntu@mongonao1:/home
scp -r /mnt/download/mongo ubuntu@mongonao2:/home
scp -r /mnt/download/mongo ubuntu@mongonao3:/home



-- чтобы не мучаться
-- https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04-ru

#on mongonao1
pbm restore 2024-02-17T17:01:48Z
export PBM_MONGODB_URI="mongodb://pbmuser:secretpwd@localhost:27001/?authSource=admin&replicaSet=RScfg"

-- sudo chown -R pbm:pbm /home/mongo/backups
-- sudo chmod -R 777 /home/mongo/backups

pbm status
pbm logs

New in version 1.3.2: The Percona Backup for MongoDB config includes the restore options 
to adjust the memory consumption by the pbm-agent in environments with tight memory bounds. 
This allows preventing out of memory errors during the restore operation.

restore:
  batchSize: 500
  numInsertionWorkers: 10

db.values.count()

############################################################################################################################################
######Monitoring
######prometheus installation
https://www.cherryservers.com/blog/install-prometheus-ubuntu

#Update System Packages
sudo apt update

# Create a System User for Prometheus
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus

#Create Directories for Prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

#Download Prometheus and Extract Files
wget https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz
tar vxf prometheus*.tar.gz
cd prometheus*/

#Move the  Files & Set Owner
sudo mv prometheus /usr/local/bin
sudo mv promtool /usr/local/bin
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo mv consoles /etc/prometheus
sudo mv console_libraries /etc/prometheus
sudo mv prometheus.yml /etc/prometheus
sudo chown prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
sudo chown -R prometheus:prometheus /var/lib/prometheus
sudo nano /etc/prometheus/prometheus.yml

#Create Prometheus Systemd Service
sudo nano /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target

#Reload Systemd
sudo systemctl daemon-reload

#Start Prometheus Service
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
#Access Prometheus Web Interface
http://178.154.203.73:9090


############################################################################################################################################
######MongoDB Exporter installation
https://gist.github.com/rbudiharso/3ca2bc41d73a0e9e3511805b8708c345
https://www.digitalocean.com/community/tutorials/how-to-monitor-mongodb-with-grafana-and-prometheus-on-ubuntu-20-04


mkdir mongodb-exporter
cd mongodb-exporter
cd /tmp
tar xvzf mongodb_exporter-0.7.1.linux-amd64.tar.gz
curl -Ls -o ./mongodb_exporter.tar.gz https://github.com/percona/mongodb_exporter/releases/download/v0.40.0/mongodb_exporter-0.40.0.linux-amd64.tar.gz
tar xvzpf mongodb_exporter.tar.gz
chmod 755 ./mongodb_exporter
mv ./mongodb_exporter /usr/local/bin/mongodb_exporter

systemctl daemon-reload
systemctl enable mongodb_exporter.service
systemctl start mongodb_exporter.service

systemctl status mongodb_exporter.service
export MONGODB_URI=mongodb://mongomon:mongomon@158.160.127.145:27011
systemctl restart mongodb_exporter.service
systemctl status mongodb_exporter.service


http://178.154.203.73:9090
http://178.154.203.73:3000 #grafana
http://178.154.203.73:9090/targets
http://178.154.203.73:9090/metrics
http://178.154.203.73:9216/metrics

############################################################################################################################################
######install grafana
https://grafana.com/grafana/download?edition=oss

sudo apt-get install -y adduser libfontconfig1 musl
wget https://dl.grafana.com/oss/release/grafana_10.3.3_amd64.deb

root@mongonao01:/home/ubuntu# sudo dpkg -i grafana_10.3.3_amd64.deb
Selecting previously unselected package grafana.
(Reading database ... 111049 files and directories currently installed.)
Preparing to unpack grafana_10.3.3_amd64.deb ...
Unpacking grafana (10.3.3) ...
Setting up grafana (10.3.3) ...
Adding system user `grafana' (UID 115) ...
Adding new user `grafana' (UID 115) with group `grafana' ...
Not creating home directory `/usr/share/grafana'.
### NOT starting on installation, please execute the following statements to configure grafana to start automatically using systemd
 sudo /bin/systemctl daemon-reload
 sudo /bin/systemctl enable grafana-server
### You can start grafana-server by executing
 sudo /bin/systemctl start grafana-server

#http://178.154.203.73:3000 #grafana


##secret
6mhZ57dK2YDi2Z
#service check
netstat -tlpn




#delete VMs
for i in {1..4} 
do yc compute instance delete mongonao$i
done

