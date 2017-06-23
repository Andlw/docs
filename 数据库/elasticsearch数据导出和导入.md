一、安装npm 和elasticdump
  # yum install npm -y
  # npm install elasticdump

二、查询ealsticsearch数据索引
  # curl XGET http://127.0.0.1:9200/_cat/indices

三、用elasticdump工具把索引和数据导出成json文件
  # cd /data/node_modules/elasticdump/bin
  # ./elasticdump --input=http://127.0.0.1:9200/store-2017.05.21 \
    --output=/data/back/521_mapping --type=mapping
  # ./elasticdump --input=http://127.0.0.1:9200/store-2017.05.21 \
    --output=/data/back/521_data --type=data

四、把导出的数据文件传到另外一个elasticsearch集群上，用elasticdump工具导入数据
  # cd /data/node_modules/elasticdump/bin
  # ./elasticdump --input=/data/back/521_mapping \
    --output=http://127.0.0.1:9200/store-2017.05.21  --type=mapping
  # ./elasticdump --input=/data/back/521_data \
    --output=http://127.0.0.1:9200/store-2017.05.21  --type=data

五、把一个elasticsearch集群的数据直接导入到另外一个elasticsearch集群
  # cd /data/node_modules/elasticdump/bin
  # ./elasticdump --input=http://172.26.223.100:9200/store-2017.05.25 \
    --output=http://172.26.223.110:9200/store-2017.05.25 --type=mapping
  # ./elasticdump --input=http://172.26.223.100:9200/store-2017.05.25 \
    --output=http://172.26.223.110:9200/store-2017.05.25 --type=data
