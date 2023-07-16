1.核心服务安装
1.1安装kafua
wget http://ftp.meisei-u.ac.jp/mirror/apache/dist/zookeeper/zookeeper-3.4.12/zookeeper-3.4.12.tar.gz
(备用:https://apache.org/dist/zookeeper/zookeeper-3.4.12/zookeeper-3.4.12.tar.gz)
tar xvf kafka_2.12-2.0.0.tgz
tar zxvf zookeeper-3.4.12.tar.gz
cd kafka_2.12-2.0.0/config/

vi server.properties
broker.id=0
port = 9092
host.name = localhost

vi producer.properties
bootstrap.servers=localhost:9092
metadata.broker.listconnect=localhost:9092

vi zookeeper.properties
clientPort=2181
host.name = localhost

vim consumer.properties
zookeeper.connect=localhost:2181

# timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=6000

bin/zookeeper-server-start.sh config/zookeeper.properties &
bin/kafka-server-start.sh config/server.properties &
1.2安装Redis
wget http://download.redis.io/releases/redis-3.2.8.tar.gz
tar zxvf redis-3.2.8.tar.gz
cd redis-3.2.8/

make PREFIX=/usr/local/redis install

ln -s /usr/local/redis/bin/redis-cli /usr/local/bin/redis-cli
ln -s /usr/local/redis/bin/redis-sentinel /usr/local/bin/redis-sentinel
ln -s /usr/local/redis/bin/redis-server /usr/local/bin/redis-server
mkdir run

cd run/
mkdir data
mkdir log
mkdir tmp
..
redis-server redis_master_6301.conf
redis-server redis_slave_6315.conf
redis-server redis_slave_6316.conf
redis-server redis_slave_6317.conf

redis-sentinel sentinel_26301.conf &
redis-sentinel sentinel_26302.conf &
redis-sentinel sentinel_26303.conf &

1.3安装Mysql
apt install mysql-server

1.4安装viabtc_exchange_server
git clone https://github.com/viabtc/viabtc_exchange_server.git


cd viabtc_exchange_server/
l
make -C depends/hiredis
make -C network
#如果报错找不到mysql.h安卓编译开发库
#sudo apt-get install libmysql++-dev


vi utils/makefile
#modify INCS
# INCS = -I ../network -I ../depends
make -C utils

vi accesshttp/makefile
#modify INCS & LIBS
# INCS = -I ../network -I ../utils -I ../depends
# LIBS = -L ../utils -lutils -L ../network -lnetwork -L ../depends/hiredis -Wl,-Bstatic -lev -ljansson -lmpdec -lrdkafka -lz -lssl -lcrypto -lhiredis -lcurl -Wl,-Bdynamic -lm -lpthread -ldl -lssl -lldap -llber -lgss -lgnutls -lidn -lnettle -lrtmp -lsasl2 -lmysqlclient
make -C accesshttp

vi accessws/makefile
{modify INCS and LIBS like accesshttp/makefile}
make -C accessws

vi alertcenter/makefile
{modify INCS and LIBS like accesshttp/makefile}
make -C alertcenter

vi marketprice/makefile
{modify INCS and LIBS like accesshttp/makefile}
make -C marketprice


vi matchengine/makefile
{modify INCS and LIBS like accesshttp/makefile}
make -C matchengine

vi readhistory/makefile
{modify INCS and LIBS like accesshttp/makefile}
make -C readhistory

### create db
cd viabtc_exchange_server/
cd sql/
vi init_trade_history.sh

vim create_trade_history.sql
CREATE DATABASE `trade_history`;
USE `trade_history`;

vim create_trade_log.sql
CREATE DATABASE `trade_log`;
USE `trade_log`;

mysql -u root -p < create_trade_history.sql 
vi create_trade_log.sql 
mysql -u root -p < create_trade_log.sql 
./init_trade_history.sh

###
mkdir coin_exchange
cd coin_exchange
mkdir readhistory accesshttp accessws marketprice matchengine alertcenter
cd ..

cp accesshttp/accesshttp.exe accesshttp/config.json accesshttp/restart.sh coin_exchange/accesshttp/
cp accessws/accessws.exe accessws/config.json accessws/restart.sh coin_exchange/accessws/
cp alertcenter/alertcenter.exe alertcenter/config.json alertcenter/restart.sh coin_exchange/alertcenter/
cp marketprice/marketprice.exe marketprice/config.json marketprice/restart.sh coin_exchange/marketprice/
cp matchengine/matchengine.exe matchengine/config.json matchengine/restart.sh coin_exchange/matchengine/
cp readhistory/readhistory.exe readhistory/config.json readhistory/restart.sh coin_exchange/readhistory/

### start
cd /alidata/via/coin_exchange_server/


cd /root/viabtc_exchange_server/coin_exchange/matchengine
vi config.json 配数据库 建目录
./restart.sh 

cat config.json 
cd ../alertcenter/
./restart.sh 
cd ../readhistory/
./restart.sh 
ll
vi config.json 
./restart.sh 
cd ../accesshttp/
ll
cat config.json 
./restart.sh 
cd ../marketprice/
cat config.json 
./restart.sh 
cd ../accessws/
cat config.json 
tcp@0.0.0.0:8090
./restart.sh 
cd ../marketprice/
vi  config.json 
"127.0.0.1:26301",
"127.0.0.1:26302",
 "127.0.0.1:26303"

#### test
curl http://127.0.0.1:8080/ -d '{"method": "market.list", "params": [], "id": 1516681174}'

#### viaxchtest

git clone https://github.com/djpnewton/viaxchtest.git

cd viaxchtest/

vi src/index.js 

npm install browserify -g
npm install
apt install node
apt install nodejs
sudo ln -s /usr/bin/nodejs /usr/bin/node
npm run build
npm install http-server -g
apt install screen
screen #进入screen子界面，此时putty标题栏会指示处于子界面状态，然后运行你的程序
http-server -a 0.0.0.0 -p 8088
然后按下Ctrl+A后抬起，然后按下d键，此时切换回主界面
screen -ls
screen -r 子界面代号

1.5重启服务云创源码loowp.com
cd kafka_2.12-2.0.0
bin/zookeeper-server-start.sh config/zookeeper.properties &
bin/kafka-server-start.sh config/server.properties &

cd redis-3.2.8/
redis-server redis_master_6301.conf
redis-server redis_slave_6315.conf
redis-server redis_slave_6316.conf
redis-server redis_slave_6317.conf

redis-sentinel sentinel_26301.conf &
redis-sentinel sentinel_26302.conf &
redis-sentinel sentinel_26303.conf &

cd viabtc_exchange_server/
cd matchengine && ./restart.sh && cd ../
cd alertcenter && ./restart.sh && cd ../
cd readhistory && ./restart.sh && cd ../
cd accesshttp && ./restart.sh && cd ../
cd accessws && ./restart.sh && cd ../
cd marketprice && ./restart.sh && cd ../
cd matchengine/
nohup python -u update_coins.py >> up.log &

viaxchtest测试页面启动
cd viaxchtest/
screen #进入screen子界面，此时putty标题栏会指示处于子界面状态，然后运行你的程序
http-server -a 0.0.0.0 -p 8088
然后按下Ctrl+A后抬起，然后按下d键，此时切换回主界面

2.视图控制服务器安装
2.1安装环境
Apche 2.4
Mysql 5.6 
Php 7.2

建议安装宝塔集成环境，https://www.bt.cn/

2.2配置
2.2.1导入数据库
Jinglanex.sql
2.2.2配置网站域名
根目录为jinglanex\web
2.2.3配置数据库连接
配置文件为 jinglanex\common\config\main-local.php



3.区块链节点安装
3.1区块链节点安装
由于区块链节点客户端在不断更新中，建议在节点官网下载最新节点安装
Btc 
https://www.bitcoin.com/

Eth 
https://www.ethereum.org/

Usdt 
https://tether.to/

