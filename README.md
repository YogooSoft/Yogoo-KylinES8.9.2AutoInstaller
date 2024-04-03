# Ansible 部署 ES-8.9.2-x86 
# 简介
   KylinESAutoInstaller
   该项目使用 Ansible 自动化工具，实现在银河麒麟V10-SP3 x86架构版本上对 Elasticsearch 的一键式离线环境安装。该项目包含所有必要的安装文件，使安装过程简单快速。

# 主要特点
* 一键式安装：使用 Ansible 简化 Elasticsearch 安装流程。
* 集成所有安装文件：将所有必需的安装文件（例如 Asnsble、Elasticsearch 和 Supervisor）集成在一起。
* 简化配置：提供简单的配置选项，以适应各种环境。
* 自动化：自动化安装过程，减少手动工作量。
* 常用功能：集群扩容节点、滚动重启、全面重启、节点下线、节点上线、节点销毁、插件安装等。

# 优点
* 快速安装，节省时间。
* 简化管理，降低成本。
* 可扩展性，可以扩展到大型环境。
* 可维护性，易于维护。
  
# 前期准备
* 服务器要求：银河麒麟V10-SP3 X86架构
* ansible服务器1台【172.168.0.125】
* ES master节点服务器3台
* ES data节点服务器4台
* ES coordinating节点服务器2台
* root权限

# 服务器节点分配示例
  角色    | ip | 名称
|:--------| :--------| :--------|
 master节点  | 172.168.0.126 | node-126-master-8x
 master节点  | 172.168.0.127 | node-127-master-8x
 master节点  | 172.168.0.128 | node-128-master-8x
 data节点  | 172.168.0.129 | node-129-data-8x
 data节点  | 172.168.0.130 | node-130-data-8x
 data节点  | 172.168.0.131 | node-131-data-8x
 data节点  | 172.168.0.132 | node-132-data-8x
 coordinating节点  | 172.168.0.133 | node-133-coord-8x
 coordinating节点  | 172.168.0.134 | node-134-coord-8x

# 使用说明
## 步骤1：工作目录规范
* 创建目录/root/yogoo_es_ansible
* 工作目录结构
```text
Yogoo-KylinES8.9.2AutoInstaller
├─ ansible_install.sh                       # ansible一键安装脚本
├─ elk_pkg                                  # ES部署相关安装包
│  └─ elk
│     └─ README.txt                         # 需要的安装包列表，可前往交流群下载
├─ playbook                                 # ansible剧本文件
│  ├─ inventory                             # ansible剧本配置文件
│  │  ├─ all                                # 配置节点ip及别名
│  │  ├─ es_8.9.2                           # ES安装基础配置文件
│  │  ├─ es_8.9.2_add                       # ES安装扩容配置文件
│  │  ├─ es_8.9.2_more                      # ES安装单机多实例配置文件
│  │  └─ kibana_8.9.2                       # kibana安装配置文件
│  ├─ playbooks                             # ansible剧本目录
│  │  ├─ roles                              # 各安装项剧本
│  │  │  ├─ elasticsearch                   # ES安装相关剧本
│  │  │  ├─ kibana                          # kibana安装相关剧本
│  │  │  └─ supervisor                      # supervisor安装相关剧本
│  └─ script_bash                           # 剧本一键执行脚本
├─ portable-ansible-v0.4.0-py3.tar.bz2      # ansible离线包
└─ soft_pkg                                 # 部署所需相关程序安装包
```
    
## 步骤2：文件下载服务配置
关闭防火墙，启动文件下载服务
```shell
# 查看防火墙
systemctl status firewalld
# 关闭防火墙
systemctl stop firewalld
# 启动文件下载服务
cd /root/yogoo_es_ansible/elk_pkg   
nohup python3 -m http.server 8080 &
# 浏览器访问服务器ip对应地址，即可查看到/root/yogoo_es_ansible/elk_pkg/elk目录下文件
http://172.168.0.125:8080/elk
```
## 步骤3：机器之间的免密配置
### 3.1 hosts文件编辑
```shell
# 备份 /etc/hosts.bk
cp /etc/hosts /etc/hosts.bk
# 编辑hosts增加ES节点服务器ip 别名
vi /etc/hosts
# 内容为 IP地址 别名，如下所示配置所有es服务器的别名
172.168.0.126 es_126
172.168.0.127 es_127
172.168.0.128 es_128
172.168.0.129 es_129
172.168.0.130 es_130
172.168.0.131 es_131
172.168.0.132 es_132
172.168.0.133 es_133
172.168.0.134 es_134
```
### 3.2 sshpass 离线安装
安装sshpass 用于配置服务免密登录
```shell
cd /root/yogoo_es_ansible/soft_pkg   
yum localinstall -y sshpass-1.06-8.ky10.x86_64.rpm
sshpass -V
```
### 3.3 配置免密登录
```shell
# 生成公钥，执行命令，回车直到结束
ssh-keygen

# 通过以下方式指定每台es服务器的别名配置免密登录
sshpass -p '服务器登录密码' ssh-copy-id -o StrictHostKeyChecking=no es_126
sshpass -p '服务器登录密码' ssh-copy-id -o StrictHostKeyChecking=no es_127
sshpass -p '服务器登录密码' ssh-copy-id -o StrictHostKeyChecking=no es_128
sshpass -p '服务器登录密码' ssh-copy-id -o StrictHostKeyChecking=no es_129
sshpass -p '服务器登录密码' ssh-copy-id -o StrictHostKeyChecking=no es_130
sshpass -p '服务器登录密码' ssh-copy-id -o StrictHostKeyChecking=no es_131
sshpass -p '服务器登录密码' ssh-copy-id -o StrictHostKeyChecking=no es_132
sshpass -p '服务器登录密码' ssh-copy-id -o StrictHostKeyChecking=no es_133
sshpass -p '服务器登录密码' ssh-copy-id -o StrictHostKeyChecking=no es_134
```
### 3.4 免密登录测试
安装sshpass 用于配置服务免密登录
```shell
# 连接远程
ssh es_134
#退出远程
logout
```
## 步骤4：安装ansible
### 4.1 执行安装脚本
```shell
cd /root/yogoo_es_ansible
# ansible_install.sh 增加可执行权限
chmod +x ansible_install.sh
# 执行安装ansible脚本
./ansible_install.sh
source /etc/profile
```
### 4.2 验证版本
```shell
# 版本：2.9.0
ansible --version
```
## 步骤5：ES部署
### 5.1 剧本执行脚本增加执行权限
```shell
cd /root/yogoo_es_ansible/playbook/script_bash
chmod +x do_elasticsearch-setup.sh
chmod +x do_elasticsearch-rolling-reconfigure.sh
chmod +x do_elasticsearch-full-reconfigure.sh
chmod +x do_elasticsearch-stop-all.sh
chmod +x do_elasticsearch-start-all.sh
chmod +x do_elasticsearch-plugin.sh
chmod +x do_destroy.sh
chmod +x do_elasticsearch-add-cert.sh
chmod +x do_kibana-setup.sh
```
### 5.2 ES配置文件编辑
ES配置文件位置：/root/yogoo_es_ansible/playbook/inventory
```shell
cd /root/yogoo_es_ansible/playbook/inventory
#修改配置文件中的url地址为elk地址：http://172.168.0.125:8080/elk。
vi es_8.9.2
vi es_8.9.2_add
# 插件配置，根据实际需要增加或删减
plugins=[{"name": "analysis-ik", "url": "file:///opt/elk/tmp/elasticsearch-analysis-ik-8.9.2.zip"}]
# 节点服务器配置
根据实际情况配置服务器ip、节点名称
```
### 5.3 执行剧本
执行脚本安装ES-8.9.2，安装结束务必记录输出的es相关登录账号和密码

脚本位置：/root/yogoo_es_ansible/playbook/script_bash
```shell
# 执行完成后记录es登录账号和密码，后续登录需要使用elastic账号
cd /root/yogoo_es_ansible/playbook/script_bash
bash do_elasticsearch-setup.sh -i es_8.9.2

# 安装完成输出内容如下：
ok: [node-126-master-8x] => {
    "output": [
        "",
        "### ### ### ### ### ### #",
        "# es coordinating hosts #",
        "### ### ### ### ### ### #",
        "http://172.168.0.129:9200",
        "",
        "### ### ### ### ### ### ### ### ##",
        "# username and passwords #",
        "### ### ### ### ### ### ### ### ##",
        " user: beats_system    password: jQd******2qg ",
        " user: logstash_system    password: I1N******duT ",
        " user: kibana    password: B6M******XnG ",
        " user: elastic    password: Oju******Ev0 ",
        "",
        "### ### ### ### ### ### #",
        "# Running Status #",
        "### ### ### ### ### ### #",
        "cluster_testarm8-node-126-master-8x   RUNNING   pid 263104, uptime 0:00:47",
        "cluster_testarm8-node-127-master-8x   RUNNING   pid 260663, uptime 0:00:47",
        "cluster_testarm8-node-128-master-8x   RUNNING   pid 289121, uptime 0:00:47",
        "cluster_testarm8-node-129-data-8x    RUNNING   pid 310241, uptime 0:00:47",
        "cluster_testarm8-node-130-data-8x    RUNNING   pid 322487, uptime 0:00:47",
        "cluster_testarm8-node-131-data-8x    RUNNING   pid 341429, uptime 0:00:47",
        "cluster_testarm8-node-132-data-8x    RUNNING   pid 362148, uptime 0:00:47",
        "cluster_testarm8-node-133-coord-8x   RUNNING   pid 472872, uptime 0:00:47",
        "cluster_testarm8-node-134-coord-8x   RUNNING   pid 435521, uptime 0:00:47"
    ]
}
```
### 5.4 查看节点
```shell
# 浏览器访问查看节点
# 账号密码：elastic/Oju******Ev0
http://172.168.0.126:9200/_cat/nodes
```
### 5.5 注意事项
各服务器确保系统时间一致，否则会引发服务器证书错误。
## 步骤6：常用剧本测试
脚本参数说明
* -i 指定使用的ES配置文件
* -s 指定需要执行节点名称，多个节点名称使用英文逗号隔开
* -h 指定host读取文件，仅用于集群销毁脚本

### 6.1 测试扩容集群
```shell
# 注意：扩容的集群需要在ES配置文件中配置
cd /root/yogoo_es_ansible/playbook/script_bash
bash do_elasticsearch-setup.sh -i es_8.9.2_add -s node-132-data-8x,node-133-coord-8x,node-134-coord-8x
```
### 6.2 测试滚动重启
```shell
cd /root/yogoo_es_ansible/playbook/script_bash
bash do_elasticsearch-rolling-reconfigure.sh -i es_8.9.2_add
```
### 6.3 测试全面重启
```shell
cd /root/yogoo_es_ansible/playbook/script_bash
bash do_elasticsearch-full-reconfigure.sh -i es_8.9.2_add
```
### 6.4 测试仅停止集群
```shell
cd /root/yogoo_es_ansible/playbook/script_bash
bash do_elasticsearch-stop-all.sh -i es_8.9.2_add
```
### 6.5 测试仅启动集群
```shell
cd /root/yogoo_es_ansible/playbook/script_bash
bash do_elasticsearch-start-all.sh -i es_8.9.2_add
```
### 6.6 测试节点下线
```shell
# 将节点从集群路由策略中排除（适用于持续写入ES集群）
curl -X PUT -H "Content-Type: application/json" -u "elastic:Oju******Ev0"  http://172.168.0.126:9200/_cluster/settings -d '
{"transient" : {"cluster.routing.allocation.exclude._ip" : "172.168.0.132,172.168.0.130"}}';
# 查看配置
curl -X GET -u "elastic:Oju******Ev0" http://172.168.0.126:9200/_cluster/settings
# 等待所有分区与数据迁移完成 等待集群green/验证集群状态是否green
curl -u elastic:Oju******Ev0 172.168.0.132:9200/_cat/health
# 登录对应节点所在服务器，使用supervisord工具停止es节点服务[目标节点服务器下执行]
cd /opt/elk/supervisor
#查看当前es服务状态及es服务名[目标节点服务器下执行]
supervisorctl status
#停止es节点服务[目标节点服务器下执行]
supervisorctl stop [es服务名] 
# 等待集群green/验证集群状态是否green
curl -u elastic:Oju******Ev0 172.168.0.126:9200/_cat/health
# 查看节点是否下线，也可通过浏览器查看
curl -X GET -u "elastic:Oju******Ev0"  http://172.168.0.126:9200/_cat/nodes?
# 恢复集群路由策略（适用于持续写入ES集群）
curl -X PUT -H "Content-Type: application/json" -u "elastic:Oju******Ev0"  http://172.168.0.126:9200/_cluster/settings -d '
{"transient" : {"cluster.routing.allocation.exclude._ip" : null}}';
```
### 6.7 测试节点重新上线
```shell
# 登录对应节点所在服务器，使用supervisord工具启动es节点服务 查看当前es服务状态及es服务名
supervisorctl status
#启动es节点服务
supervisorctl start [es服务名]
# 等待集群green/验证集群状态是否green
curl -u elastic:Oju******Ev0 172.168.0.126:9200/_cat/health
```
### 6.8 测试插件安装
```shell
# 确认es配置文件中的配置
vi /root/yogoo_es_ansible/playbook/inventory/es_8.9.2_add

# plugins下载剧本确认，确保需要下载的插件都在/root/yogoo_es_ansible/elk_pkg/elk目录下
vi /root/yogoo_es_ansible/playbook/playbooks/roles/elasticsearch/tasks/plugin/download.yml

# 执行剧本 
bash do_elasticsearch-plugin.sh -i es_8.9.2_add
# 验证 访问http://172.168.0.126:9200/_cat/plugins 查看各节点插件版本
```
### 6.9 测试节点销毁
```shell
# 编辑all文件
vi /root/yogoo_es_ansible/playbook/inventory/all
# 编辑内容如下，将所有es节点服务器按照 服务器别名 服务器ip 形式配置
es126 ansible_host=172.168.0.126
es127 ansible_host=172.168.0.127

# 将节点从集群路由策略中排除（适用于持续写入ES集群）
curl -X PUT -H "Content-Type: application/json" -u "elastic:Oju******Ev0"  http://172.168.0.126:9200/_cluster/settings -d '
{"transient" : {"cluster.routing.allocation.exclude._ip" : "172.168.0.134"}}';
# 执行销毁
bash do_destroy.sh -h all -s es134
# 恢复集群路由策略配置
curl -X PUT -H "Content-Type: application/json" -u "elastic:Oju******Ev0"  http://172.168.0.126:9200/_cluster/settings -d '
{"transient" : {"cluster.routing.allocation.exclude._ip" : null}}';

```
### 6.10 测试证书更新
```shell
cd /root/yogoo_es_ansible/playbook/script_bash
bash do_elasticsearch-add-cert.sh -i es_8.9.2_add
# 结果查看[在节点服务器查看]
cd /opt/elk/cluster_testarm8/elasticsearch/node-134-coord-8x/config/certs
openssl pkcs12 -in elastic-certificates.p12 -clcerts -nodes | openssl x509 -noout -enddate
# 浏览器访问查看
http://172.168.0.126:9200/_ssl/certificates
```
### 6.11 单服务器部署多节点
```shell
# 编辑配置文件  例：es_8.9.2_more
vi /root/yogoo_es_ansible/playbook/inventory/es_8.9.2_more
# 在节点配置中增加节点信息
# ansible_host指定为同一个ip；节点名称为不同名称；分别指定http_port、transport_port为不同的端口；jvm_heap_memory根据具体情况分配，如合计超过服务器内存则会导致节点无法启动。示例如下：
node-129-data-8x-01 ansible_host=172.168.0.129 node_name=node-129-data-8x-01 path_data=/data/elasticsearch/node-129-data-8x-01  path_logs=/data/elasticsearch/logs/node-129-data-8x-01 http_port=9201 transport_port=9301 jvm_heap_memory=1g
node-129-data-8x-02 ansible_host=172.168.0.129 node_name=node-129-data-8x-02 path_data=/data/elasticsearch/node-129-data-8x-02  path_logs=/data/elasticsearch/logs/node-129-data-8x-02 http_port=9202 transport_port=9302 jvm_heap_memory=1g
# 如果是新的节点直接执行扩容剧本
bash do_elasticsearch-setup.sh -i es_8.9.2_more -s node-129-data-8x-01,node-129-data-8x-02
# 如果是在已部署节点服务器上新增，则需销毁服务器节点后，再执行扩容
```
## 步骤7：Kibana部署
### 7.1 kibana配置文件
```shell
# 配置kibana_8.9.2
vi /root/yogoo_es_ansible/playbook/inventory/kibana_8.9.2
# 主要修改内容：
修改download_url、supervisor_download_url的ip地址为elk地址：172.168.0.125:8080
修改kibana_password为部署的ES kibana账号密码
修改es_hosts指定多台节点服务器
修改kibana ansible_host 指定一台master节点服务器安装
# 内容如下：
[all:vars]
download_url=http://172.168.0.125:8080/elk
supervisor_download_url=http://172.168.0.125:8080/elk
version=8.9.2
project_name=cluster_testarm8
path_base=/opt/elk
supervisor_use_systemd=true
running_user=elk
url_username=elk
url_password=e******01
kibana_username=kibana
kibana_password=B6M******XnG
xpack_security=true

[kibana:vars]
es_hosts=['http://172.168.0.133:9200', 'http://172.168.0.134:9200']
server_host=0.0.0.0
server_port=15601

[kibana]
kibana ansible_host=172.168.0.126
```
### 7.2 执行kibana部署脚本
```shell
cd /root/yogoo_es_ansible/playbook/script_bash
bash do_kibana-setup.sh -i kibana_8.9.2
# 访问地址：http://172.168.0.126:15601/
```
### 7.3 配置writer账号
```shell
# 进入kibana 的开发工具页面执行以下命令
PUT _security/role/writer 
{  
  "cluster": ["monitor","read_ilm","manage_ingest_pipelines","manage_ilm","manage_index_templates"],  
  "indices": [  
    {  
      "names": [ "*" ],  
      "privileges": ["create","create_index","index","write","manage_ilm","view_index_metadata","manage"]  
    }  
  ]  
} 
PUT _security/user/writer
{  
  "password" : "wr******01",  
  "roles" : [ "writer"],  
  "full_name" : "writer",  
  "email" : "w****f@bitian.cn"  
}
```
### 7.4 启用monitoring
```shell
# 进入kibana 的开发工具页面执行以下命令
PUT _cluster/settings  
{  
    "persistent": {  
        "xpack": {  
            "monitoring": {  
                "collection": {  
                    "enabled": "true"  
              }  
            }  
        }  
    }  
}
```
### 7.5 启用monitoring
```shell
# 进入kibana 的开发工具页面执行以下命令
PUT _cluster/settings
{  
  "persistent" : {  
    "cluster.routing.allocation.awareness.attributes": "null"  
  }  
}

```
# ES-Ansible 交流QQ群
<img src="images/QQ-ES-ANSIBLE.png" width=30%>