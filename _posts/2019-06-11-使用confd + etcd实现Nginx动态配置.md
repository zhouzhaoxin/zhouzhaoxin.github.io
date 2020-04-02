---
layout: post
title: "使用confd + etcd实现Nginx动态配置"
author: zhouzhaoxin
categories: ["后端"]
image: assets/images/fantasy-4956789_1920.jpg
comments: false
---
## etcd
`etcd`是一个高可用的分布式键值(key-value)**数据库**。内部采用`raft`协议作为一致性算法，基于Go语言实现。`etcd`数据库与`redis`类似，其独特性在于：
1. 分布式部署，扩展性强，且数据和事务保持一致
2. 提供`watch`接口，可监听多个键的变化
3. 对于单个键而言，每次更新其值都会保留上一个版本，可以对键进行版本回溯
4. `ttl`使用租约实现

> **`etcd`**更强调的是各个节点之间的通信，同步，确保各个节点上数据和事务的一致性，使得服务更稳定，本身单节点的写入能力并不强。 
**`redis`**更像是内存型缓存，虽然也有`cluster`做主从同步和读写分离，但节点间的一致性主要强调的是数据，并不在乎事务，因此读写能力很强，`qps`甚至可以达到10万+


#### 安装
```
$ mkdir etcd
$ cd etcd
$ curl -L https://github.com/etcd-io/etcd/releases/download/v3.3.13/etcd-v3.3.13-linux-amd64.tar.gz -o 
$ ./etcd-v3.3.13-linux-amd64.tar.gz
$ tar xzvf etcd-v3.3.13-linux-amd64.tar.gz --strip-components=1
$ ./etcd -version
$ ./etcdctl -version
```

#### 本地多成员集群
针对单机用户，开启多进程，模拟多机器集群(本次模拟开启三个`etcd`集群)

###### 1. 安装`go`

###### 2. 安装`goreman`（进程管理工具）
```shell
$ go get github.com/mattn/goreman
```
###### 3. 查看`gopath`
```shell
$ go env
GOPATH="/home/apple/go"
```
**所以`goreman`运行路径为`home/apple/go/bin/goreman`**
###### 4. 编写`goroman`配置文件
`goroman`配置文件默认名称为Procfile,可以更换，但启动时，需要通过`-c`指定配置文件
`goroman`管理进程的配置文件由 `进程名:执行命令`组成
```
$ vim Procfile
```
编辑内容如下
```
etcd1: ./etcd --name infra1 --listen-client-urls http://127.0.0.1:2379 --advertise-client-urls http://127.0.0.1:2379 --listen-peer-urls http://127.0.0.1:12380 --initial-advertise-peer-urls http://127.0.0.1:12    380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof
etcd2: ./etcd --name infra2 --listen-client-urls http://127.0.0.1:22379 --advertise-client-urls http://127.0.0.1:22379 --listen-peer-urls http://127.0.0.1:22380 --initial-advertise-peer-urls http://127.0.0.1:    22380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof
etcd3: ./etcd --name infra3 --listen-client-urls http://127.0.0.1:32379 --advertise-client-urls http://127.0.0.1:32379 --listen-peer-urls http://127.0.0.1:32380 --initial-advertise-peer-urls http://127.0.0.1:    32380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof
```
###### 5. 运行
```shell
$ /home/apple/go/bin/goreman -f  Procfile start
```
此时`etcd`集群被开启
###### 6.查看集群列表
此时代表集群正确安装并启动
```shell
$ export ETCDCTL_API=3
$ ./etcdctl member list
8211f1d0f64f3269: name=infra1 peerURLs=http://127.0.0.1:12380 clientURLs=http://127.0.0.1:2379 isLeader=false
91bc3c398fb3c146: name=infra2 peerURLs=http://127.0.0.1:22380 clientURLs=http://127.0.0.1:22379 isLeader=true
fd422379fda50e48: name=infra3 peerURLs=http://127.0.0.1:32380 clientURLs=http://127.0.0.1:32379 isLeader=false
```
#### etcd基本使用
`API`地址`https://godoc.org/github.com/coreos/etcd/client`
官方包提供了对于`etcd`所有操作的`API`

###### 创建用于操作`etcd`键值的`KeysAPI`对象`kv`
```golang
var cli client.Client
var kv client.KeysAPI

func handleError(e error, msg string) {
	if e != nil {
		log.Fatal(e)
	}
	if msg != "" {
		log.Println(msg)
	}
}

func init(){
	cfg := client.Config{
		Endpoints: []string{"http://127.0.0.1:2379", "http://127.0.0.1:22379", "http://127.0.0.1:32379"},
	}
	var e error

	cli, e = client.New(cfg)
	handleError(e, "")
	kv = client.NewKeysAPI(cli)
}
```
###### 设置键
```golang
func setVal(kv client.KeysAPI, key string, val string) {
	log.Printf("设置键%s值%s\n", key, val)
	_, e := kv.Set(context.Background(), key, val, nil)
	if e != nil {
		log.Println(e)
	}
}
```
###### 获取键值
```golang
func getVal(kv client.KeysAPI, key string) string {
	// 获取键
	log.Printf("开始获取键%s \n", key)
	resp, e := kv.Get(context.TODO(), key, nil)
	handleError(e, "")
	index, value := resp.Index, resp.Node.Value
	log.Printf("获取当前版本:%d 值:%s", index, resp.Node.Value)
	return string(value)
}
```

###### 创建文件夹
`etcd`的键值对存储可以理解为文件存储在目录中，键为目录，值为文件
创建文件夹的目的为：使用`etcd`提供的`watch`方法可以监控整个文件夹中键（即文件）的变化
```golang
func mkdir(kv client.KeysAPI, dir string) {
	o := client.SetOptions{Dir: true}
	_, e := kv.Set(context.Background(), dir, "", &o)
	handleError(e, "创建完成")
}
```

###### 调用
```golang
func TestSetVal(t *testing.T) {
	setVal(kv, "/nginx/foo", "bar")
}

func TestGetVal(t *testing.T) {
	getVal(kv, "/nginx/foo")
}

func TestMkdir(t *testing.T) {
	mkdir(kv, "/nginx")
}
```
## confd
轻量级的配置管理工具，主要有两个目的
1. 读取etcd保存的配置信息，同步到本地配置文件中，并保证本地配置文件是最新的
2. 同步配置文件之后可以指定命令使配置生效

#### 安装
https://github.com/kelseyhightower/confd/blob/master/docs/installation.md

#### 使用
把核心信息，比如`upstream`、`server_name`等存储在`etcd`中，使用`confd`来自动生成`nginx`配置文件,并`reload`使配置生效
```
upstream www_test {
    server 196.75.121.112:443;     （动态生成）
}

server {
    listen       443 ssl; （动态生成）
    server_name  www.test.com; (动态生成)
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;; 
    ssl_certificate             /home/build/openresty/nginx/cert/dealssl/www.bestenover.com.crt; (动态生成)

    location / { 
        proxy_pass https://www_test; (动态生成)
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_redirect off;
    }   
}
```
要实现动态配置首先要把核心信息存储到etcd中
```
func TestNginxMkdir(t *testing.T) {
	mkdir(kv, "/nginx")
	mkdir(kv, "/nginx/https")
	mkdir(kv, "/nginx/http")
	mkdir(kv, "/nginx/ssl")
	mkdir(kv, "/nginx/https/www")
	mkdir(kv, "/nginx/https/www/server")
	mkdir(kv, "/nginx/https/www/upstream")
	mkdir(kv, "/nginx/https/www/server/location")
}
```
`confd`注册监控`etcd`的`key`为`/nginx/`，只要发生变化就通知`confd`根据模板生成配置。`confd`默认的配置路径为`/etc/confd/`，创建`conf.d`和`template`两个目录，分别存放配置资源和配置模板。

nginx的配置资源如下所示：test.conf.toml
```
[template]
src = "test.conf.tmpl"
dest = "/tmp/test.conf"
keys = [
    "/nginx",
]
check_cmd = "echo a"
reload_cmd = "echo b"
```
`nginx`的配置模板如下所示：`test.conf.tmpl`
```
upstream www_{{getv "/nginx/https/www/server/server_name"}} {
    {{range getvs "/nginx/https/www/upstream/*"}}server {{.}};
    {{end}}
}

server {
    server_name         {{getv "/nginx/https/www/server/server_name"}}:443;
    ssl on
    ssl_certificate     {{getv "/nginx/https/www/server/ssl_certificate"}};
    ssl_certificate_key {{getv "/nginx/https/www/server/ssl_certificate_key"}};
    location / {
        proxy_pass        http://www_{{getv "/nginx/https/www/server/server_name"}};
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_redirect    off;
    }
}
```
开启`confd` : `confd -watch -backend etcd -node http://127.0.0.1:2379`
设置内容
```
func TestNginxSetVal(t *testing.T) {
	setVal(kv, "/nginx/https/www/server/server_name", "test.com")
	setVal(kv, "/nginx/https/www/server/ssl_certificate", "client.crt")
	setVal(kv, "/nginx/https/www/server/ssl_certificate_key", "client.key")
	setVal(kv, "/nginx/https/www/upstream/server1", "192.168.4.2:443")
	setVal(kv, "/nginx/https/www/upstream/server2", "192.168.5.2:443")
}
```
生成结果
```
  1 upstream www_test.com {                                                                                                                                                                                         
  2     server 192.168.4.2:443;
  3     server 192.168.5.2:443;
  4 
  5 }
  6 
  7 server {
  8     server_name         test.com:443;
  9     ssl on
 10     ssl_certificate     client.crt;
 11     ssl_certificate_key client.key;
 12     location / {
 13         proxy_pass        http://www_test.com;
 14         proxy_set_header Host $host;
 15         proxy_set_header X-Real-IP $remote_addr;
 16         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 17         proxy_set_header X-Forwarded-Proto https;
 18         proxy_redirect    off;
 19     }
 20 }
```