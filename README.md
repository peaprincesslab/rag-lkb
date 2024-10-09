# rag-lkb
win系统使用RAGFlow + Ollama搭建LKB（Local knowledge base）本地知识库

## Ollama部署
### Ollama下载安装
[git地址](https://github.com/ollama/ollama)   
[下载地址](https://ollama.com/download)  

### 环境量配置
修改后要退出ollama重新启动才生效
#### 更改模型下载地址
```
ollama下载模型的默认地址：C:\Users\<用户名>\.ollama\models
在系统变量中，新增OLLAMA_MODELS变量，值填入想要存放模型的路径
```
#### 更改访问IP
```
在系统变量中，新增OLLAMA_HOST，值填入0.0.0.0。可以通过本机IP访问
```
#### 开放跨域权限
```
系统变量中，新增OLLAMA_ORIGINS，值填入*
```
### 模型下载
[模型列表](https://ollama.com/library) 
+ 下载命令例
```
ollama pull qwen2
```
+ 查看已下载的模型
```
ollama list
```

## RAGFlow部署
当前在win系统不支撑python源码部署，使用docker方式，[参考](https://github.com/infiniflow/ragflow/blob/main/README_zh.md)
### 环境配置
#### 子系统配置
```
# 1、查看windows下的子系统
wsl -l -v

# 2、进入docker-desktop子系统
wsl -d docker-desktop

# 3、修改内存映射配置。参考https://github.com/infiniflow/ragflow/blob/main/README_zh.md
```
#### RAGFlow web端口配置
web页默认使用80端口，避免端口被占用可以调整配置
```
# 在ragflow/docker目录下，修改启动时依赖的docker-compose-CN.yml文件
修改配置项路径services.ragflow.ports的80:80，端口映射改成<自定义端口>:80
```

#### RAGFlow es内存配置
运行依赖es组件
```
# 分配内存mem_limit: ${MEM_LIMIT}，默认8073741824，分配太多容易导致资源不足es退出
在docker\.env文件，修改MEM_LIMIT值调小，如512M
```

#### RAGFlow版本配置
```
核心镜像默认拉取dev版本，在脚本的RAGFLOW_VERSION环境量控制
拉取过程中可能会因为源问题镜像层损坏导致拉取失败，可以手动指定镜像版本
在docker\.env文件，修改RAGFLOW_VERSION值，填入如v0.10.0
```

### 拉取镜像
在使用docker compose部署前，可以先手动拉取依赖的镜像，在docker-compose-base.yml文件里有描述。
```
# 开启镜像拉取的断点续传
在DockerDesktop的docker引擎配置，features配置项里，增加"containerd-snapshotter": true
```

### 启动
高版本的DockerDesktop已集成Docker Compose
```
# 在ragflow源码下
cd ragflow/docker

# windows下部署跳过这一步
#chmod +x ./entrypoint.sh

docker compose -f docker-compose-CN.yml up -d
```
启动后，在浏览器访问http://<ragflow服务ip>:<自定义端口，默认80>

### 其他
#### RAGFlow服务日志查看
```
docker logs -f ragflow-server
```

#### DockerDesktop wsl镜像文件迁移
DockerDesktop的镜像数据在wsl的docker-desktop-data子系统
```
# 1、查看windows下的子系统。可以看到有docker-desktop-data
wsl -l -v

# 2、先停止wsl服务
wsl --shutdown

# 3、导出原数据
wsl --export docker-desktop-data "E:\\docker-desktop-data.tar"

# 4、取消注册。对应的.vhdx文件会被自动删除
wsl --unregister docker-desktop-data

# 5、导入新数据。目标路径和DockerDesktop里配置的镜像目录一致
wsl --import docker-desktop-data "E:\\docker\\wsl" "E:\\docker-desktop-data.tar" --version 2
```

## 创建知识库
浏览器访问RAGFlow服务web页
### 添加模型
在系统配置里添加模型。模型名称从ollama list获取，例qwen2。模型url设置为<ollama所在服务ip>:11434

### 数据集配置
知识库要至少上传一个文件到数据集并转换成功后才生效。

### 添加助理
配置关联知识库。