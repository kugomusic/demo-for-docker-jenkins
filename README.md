## demo

本项目旨在以spring boot项目为例，演示如何使用jenkins和docker进行持续集成；

配置完成后，使用web访问远程命令即可触发项目的构建：

`10.108.211.136:12312/job/springbootdemo4docker/build?token=pcncad`



### 1、配置Dockerfile

在项目中创建Dockerfile，并上传项目至git托管平台。

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gegujra1moj30r80bx760.jpg)

Dockerfile代码如下：

```dockerfile
#项目所依赖的镜像
FROM java:8
# 将maven构建好的jar添加到镜像中
ADD *.jar app.jar
# 暴露的端口号
EXPOSE 8072
# 镜像所执行的命令
ENTRYPOINT ["java","-jar","/app.jar"]
```





### 2、配置jenkins

> jenkins和docker环境均已安装在master节点上，ip：10.108.211.136

##### 基本信息：

* jenkins地址：10.108.211.136:12312/jenkins

* 账号：admin，密码：pcncad123456



##### 2.1、创建项目

点击 “New 任务” 新建jenkins项目。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gegtrhcv2aj30a80bfgmf.jpg" style="zoom: 67%;" />

输出名称，选择构建类型（这里选择maven）。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gegttsl4m8j31590n6gq1.jpg"  />



##### 2.2、配置git仓库信息

填写仓库地址，根据需求添加令牌。

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gegu1ul7g2j313p0fpgn2.jpg)



##### 2.3、构建触发器

选择远程构建触发器。

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gegu3airsjj313o09lwg4.jpg)

Token设置完成后，使用web访问 `10.108.211.136:12312/job/springbootdemo4docker/build?token=pcncad` 即可触发构建任务。



##### 2.4、build前工作

选择shell命令，填写在maven执行build任务之前需要执行的工作。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gegu90ah20j308d05v0sx.jpg" style="zoom: 67%;" />

如果项目已经部署完成，即docker中存在已构建完成并在运行中的container，那么需要停止该container的运行并将其删除，以便后续部署新版本的代码。

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gegu6zt3blj30hl0dxmye.jpg)

shell代码如下：

```shell
# !/bin/sh -ilex
# 获取容器
containers=$(docker ps -a  -q -f name=demo_container)
# 停止并删除原有容器（如果存在）
echo $containers
if [ ! "$containers" ] ; then
        echo "不存在容器"
else
		# 停止容器
        docker stop $containers
        # 删除容器
        docker rm $containers
fi
```



##### 2.5、Maven build

填写build指令。

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gegubdg7nwj30e704b0sr.jpg)



##### 2.6、build后工作

同样选择shell命令，添加Maven build任务执行成功后需要执行的任务。

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gegucr0npwj30os0bh3zy.jpg)

shell代码如下：

```shell
# !/bin/sh -l
# 切换至源码的docker目录下
cd src/docker
# 将build完成的工程拷贝到当前目录，供docker调取
cp ../../target/*.jar ./
# 创建镜像
docker build -t springbootdemo4docker .
# 创建容器并运行
docker run -d -p 8072:8072 --name demo_container springbootdemo4docker
```



##### 2.7、启动构建

点击 “Save” 保存配置项后，选择 “立即构建” 即可触发本项目的构建任务。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gegufuo0owj309c0agaas.jpg" style="zoom: 67%;" />

也可以选择远程触发，使用web访问`10.108.211.136:12312/job/springbootdemo4docker/build?token=pcncad`

> 参考2.3 触发构建器



### 3、其他补充说明

3.1、jenkins工作目录

jenkins的工作目录是 `/var/lib/jenkins`

触发构建任务之后，jenkins会将代码拉取到 `/var/lib/jenkins/workspace` ，如有bug可在此处进行排查。

3.2、上传项目版本问题

要求上传可以稳定运行的代码，不一定是毕设的最新版本。

**一定要保证能够正常运行！**。

3.3、示例数据

最好在git仓库中上传示例数据，以便进行演示，如果数据在数据库中，请配置好数据库的路径，并在项目文档中进行说明。









