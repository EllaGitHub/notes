## ECE 复习整理

### Linux  常用命令

```shell
#删除当前目录下的 a 目录/文件
rm -rf a
#获得当前目录下 a目录/文件的绝对路径
pwd

#创建文件
touch file_name

#移动/重命名 a目录/文件 为 b目录/文件 
mv a b
#复制文件b 到 c主机上d用户的e目录中
scp file_name  d@c_ip:e_location
eg scp elastic.yml elastic@192.168.3.184:/usr/local/es/config/
#启动es
./bin/elasticsearch-7.13.4 -d -p pid
#获得当前用户es进程号
ps -ef -e | grep 'username' | grep elasticsearch
#关闭进程
kill -9 `cat pid`
kill -9  进程号



```



### 配置elasticsearch.yml

#### 1.集群节点属性基本配置----主节点,数据节点等

cluster.name

node.name

node.roles

network.host

http.port

trans.port

discovery.seed_hosts

cluster.initial_master_nodes

##### 单节点模式

discovery.type: single-node

##### dedicated master 配置

master , data , ingest , ml(机器学习), voting_only  这五个角色中只能有master,其他角色根据情况配置

因为master 与其他四个角色一起搭配,会使节点性质发生改变

##### 远程节点配置

角色: remote_cluster_client

#### 2.分片分配感知配置

node.attr.自定义属性名 : 自定义属性值

eg: node.attr.rack_id :  rack-1(/rack-2/......)

##### 冷热集群配置

 node.roles : [data_hot(/data_warm/data_cold)]

node.attr.hot_warm_cold : hot (/warm/cold)

注:node.roles 的值要与冷热属性值相互对应,即data_hot 对应 hot

#### 3.snapshot 仓库位置配置

path.repo:["自定义路径"]

eg:

path.repo:['/usr/local/es/backup']

#### 4.security 配置 [Secure the Elastic Stack](https://www.elastic.co/guide/en/elasticsearch/reference/7.13/secure-cluster.html)  

​    [Set up minimal security](https://www.elastic.co/guide/en/elasticsearch/reference/7.13/security-minimal-setup.html)    

```yaml
xpack.security.enabled: true
```

 [Set up basic security](https://www.elastic.co/guide/en/elasticsearch/reference/7.13/security-basic-setup.html)

```yaml
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate 
xpack.security.transport.ssl.client_authentication: required
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
```

### 分词器

Text analysis enables Elasticsearch to perform full-text search, where the search returns all *relevant* results rather than just exact matches.

备注: 分词器是对索引搜索时自动以什么样的方式来搜索数据, 对源数据无改变,只是搜索的方式

analyzer: 分析器

tokenizer :分词器

filter :过滤器

语法结构:





















### Snapshot 

Snapshot&Restore文档的目录即为顺序

1.创建repositary

```console
PUT /_snapshot/my_backup
{
  "type": "fs",
  "settings": {
    "location": "my_backup_location"----配置文件中配置的path.repo的值即仓库地址
  }
}
```

2.添加索引

3.索引可查

4.创建snapshot

```console
PUT /_snapshot/my_backup/snapshot_1?wait_for_completion=true
```

5.restore snapshot

```console
POST /_snapshot/my_backup/snapshot_1/_restore
```

6. mount  searchable snapshot(------searchable snapshot)

   ```console
   POST /_snapshot/my_repository/my_snapshot/_mount?wait_for_completion=true
   {
     "index": "my_docs", 
     "renamed_index": "docs", 
     "index_settings": { 
       "index.number_of_replicas": 0
     }
   }
   ```

   

### Security

1.配置elasticsearch.yml (见Secrity配置)

2.生成：elastic-stack-ca.p12 文件

```shell
./bin/elasticsearch-certutil ca
```

3.生成： elastic-certificates.p12 证书

```shell
./bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```

此证书需要存放在config/ 目录下,所以要执行

mv  elastic-certificates.p12   config/   命令

4.将config 目录中的 elastic-certificates.p12 证书 复制到其他节点的 config / 下

scp  elastic-certificates.p12   root@192.168.3.181:/usr/local/elasticsearch-7.13.0/config

### Templates

#### Index Template



#### Search Template



#### Dynamic Template (---mapping----)

rules:

"dynamic_templates":["my_template_name"]

"my_template_name":

 1.match_mapping_type:匹配的字段类型

2.match or unmatch:匹配的字段(名称或字段名称模式)

3.path_match or path_unmath

4.mapping:映射后的结果:

​    type:  映射后的类型

​    analyzer:分词器

​	doc_values

​	copy_to

#### Component Template

#### Script Template

### 跨集群操作

#### 配置

1.远程节点配置

2.如果任一集群开启security , 则需颁发相同证书(实为复制)给另一集群的各个节点

#### CCS搜索

step1 配置远程集群,保证集群彼此之间连接状态方式如下:

##### 方式一------命令

PUT _cluster/settings
{
  "transient": {
    "cluster": {
      "remote": {
        "my_cluster_name1": {
          "seeds": [
            "host1 : transport1","host2 : transport2"
          ]
        },
        "my_cluster_name2": {
          "seeds": [
            "host1 : transport1"
          ]
        }
      }
    }
  }
}

注意:

​	1>.理论可以跨n个集群,不止包含my_cluster_name1,2

​	2>.my_cluster_name1,my_cluster_name2 与配置文件的cluster_name 保持一致

​	3>.host 和 transpot  与每个集群各个节点的host ,transport 报持一致

##### 方式二------kibana ui 操作









3.搜索

GET /index_name,my_cluster_name1:index_name,my_cluster_name2:index_name/_search

#### 复制

1.远程节点配置

2.启动

3.运行  

```shell
./bin/elasticsearch-setup-passwords interactive
```



### DataStream

#### ilm policy

#### index template / component template

注意:  创建索引时将数据放在哪个节点上的配置

要用:

index.routing.allocation.**require**.{attribute}

插入数据一定在最开始多加一个@timestamp 字段

### Mapping



#### Runtime field 

  难点 & 注意:

1.[Retrieve Runtime mapping](https://www.elastic.co/guide/en/elasticsearch/reference/7.13/runtime-retrieving-fields.html#runtime-retrieving-fields)

Use the [`fields`](https://www.elastic.co/guide/en/elasticsearch/reference/7.13/search-fields.html) parameter on the `_search` API to retrieve the values of runtime fields. Runtime fields won’t display in `_source`, but the `fields` API works for all fields, even those that were not sent as part of the original `_source`.

 2 与 search template  结合:

格式:

PUT _scripts/my-search-template
{
  "script": {
    "lang": "mustache",
    "source": {
      "runtime_mappings": {
        "runtime_field_name": {
          "type": "",
          "script": {
            "source": """
            """
          }
        }
      },
     "fields":["*"],
      "query": {
      },
      "from": "{{from}}",
      "size": "{{size}}"
    }
  }
}





### 时间格式的处理

1.runtime mapping

2.时间的过滤

查询某一年或某一月

### 区别

查询的排序











