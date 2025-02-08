# Docker Images Pusher

使用Github Action将国外的Docker镜像转存到阿里云私有仓库，供国内服务器使用，免费易用<br>
- 支持DockerHub, gcr.io, k8s.io, ghcr.io等任意仓库<br>
- 支持最大40GB的大型镜像<br>
- 使用阿里云的官方线路，速度快<br>

视频教程：https://www.bilibili.com/video/BV1Zn4y19743/

作者：**[技术爬爬虾](https://github.com/tech-shrimp/me)**<br>
B站，抖音，Youtube全网同名，转载请注明作者<br>

## 使用方式


### 配置阿里云
登录阿里云容器镜像服务<br>
https://cr.console.aliyun.com/<br>
启用个人实例，创建一个命名空间（**ALIYUN_NAME_SPACE**）
![](/doc/命名空间.png)

访问凭证–>获取环境变量<br>
用户名（**ALIYUN_REGISTRY_USER**)<br>
密码（**ALIYUN_REGISTRY_PASSWORD**)<br>
仓库地址（**ALIYUN_REGISTRY**）<br>

![](/doc/用户名密码.png)


### Fork本项目
Fork本项目<br>
#### 启动Action
进入您自己的项目，点击Action，启用Github Action功能<br>
#### 配置环境变量
进入Settings->Secret and variables->Actions->New Repository secret
![](doc/配置环境变量.png)
将上一步的**四个值**<br>
ALIYUN_NAME_SPACE,ALIYUN_REGISTRY_USER，ALIYUN_REGISTRY_PASSWORD，ALIYUN_REGISTRY<br>
配置成环境变量

### 添加镜像
打开images.txt文件，添加你想要的镜像 
可以加tag，也可以不用(默认latest)<br>
可添加 --platform=xxxxx 的参数指定镜像架构<br>
可使用 k8s.gcr.io/kube-state-metrics/kube-state-metrics 格式指定私库<br>
可使用 #开头作为注释<br>
![](doc/images.png)
文件提交后，自动进入Github Action构建

### 使用镜像
回到阿里云，镜像仓库，点击任意镜像，可查看镜像状态。(可以改成公开，拉取镜像免登录)
![](doc/开始使用.png)

在国内服务器pull镜像, 例如：<br>
```
docker pull registry.cn-hangzhou.aliyuncs.com/shrimp-images/alpine
```
registry.cn-hangzhou.aliyuncs.com 即 ALIYUN_REGISTRY(阿里云仓库地址)<br>
shrimp-images 即 ALIYUN_NAME_SPACE(阿里云命名空间)<br>
alpine 即 阿里云中显示的镜像名<br>

### 多架构
需要在images.txt中用 --platform=xxxxx手动指定镜像架构
指定后的架构会以前缀的形式放在镜像名字前面
![](doc/多架构.png)

### 镜像重名
程序自动判断是否存在名称相同, 但是属于不同命名空间的情况。
如果存在，会把命名空间作为前缀加在镜像名称前。
例如:
```
xhofe/alist
xiaoyaliu/alist
```
![](doc/镜像重名.png)

### 定时执行
修改/.github/workflows/docker.yaml文件
添加 schedule即可定时执行(此处cron使用UTC时区)
![](doc/定时执行.png)


我来用中文分析这段 GitHub Actions 工作流配置文件：

这是一个用于 Docker 镜像处理的工作流，主要功能是从公共仓库拉取镜像并推送到阿里云私有仓库。让我们分部分详细分析：

1. **触发条件**:
```yaml
on:
  workflow_dispatch:  # 手动触发
  push:
    branches: [ main ]  # main 分支推送时触发
```

2. **环境变量**:
```yaml
env:
  ALIYUN_REGISTRY: # 阿里云容器镜像服务地址
  ALIYUN_NAME_SPACE: # 阿里云命名空间
  ALIYUN_REGISTRY_USER: # 阿里云登录用户
  ALIYUN_REGISTRY_PASSWORD: # 阿里云登录密码
```

3. **主要工作流程**:
- 准备工作：
  1. 显示并清理磁盘空间
  2. 使用 `maximize-build-space` 动作优化构建空间
  3. 重启 Docker 服务
  4. 检出代码
  5. 设置 Docker Buildx

- 核心处理逻辑：
  1. 登录阿里云容器镜像服务
  2. 处理 `images.txt` 文件中的镜像列表，主要包含两个阶段：
     
     **第一阶段**（重名检测）：
     - 遍历文件检测重名镜像
     - 解析镜像名称、命名空间等信息
     - 使用关联数组记录重名情况

     **第二阶段**（镜像处理）：
     - 拉取原始镜像
     - 处理平台架构信息
     - 构建新的镜像名称（包含平台和命名空间信息）
     - 标记并推送到阿里云
     - 清理本地镜像以释放空间

4. **特殊处理**:
- 支持带平台参数的镜像拉取
- 处理重名镜像的情况，通过添加命名空间前缀区分
- 处理特殊格式的镜像名（如 docker.io/nginx）
- 定期清理本地镜像以节省空间

5. **主要功能点**:
- 镜像名称解析和重命名
- 平台架构信息处理
- 重名镜像处理
- 磁盘空间管理
- 错误处理和日志输出

6. **建议改进**:
1. 可以添加错误处理机制
2. 考虑添加并行处理以提高效率
3. 可以添加镜像处理结果的汇总报告
4. 建议添加重试机制处理网络问题
5. 可以考虑缓存机制优化重复镜像的处理

这个工作流的主要用途是自动化镜像同步，将公共镜像同步到私有仓库，并且处理了重名、多架构等复杂情况。这对于需要在私有环境使用公共镜像的团队来说非常有用。








