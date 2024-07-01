# Devuan的Jenkins部署

## 大致情况

 Jenkinsfile配置文件位于下方

 https://git.devuan.org/devuan/package-pipeline

 存在测试和实际部署的分支

 https://git.devuan.org/devuan/package-pipeline/src/branch/deployment

 https://git.devuan.org/devuan/package-pipeline/src/branch/testing



 ## 安装Jenkins

 https://www.jenkins.io/doc/book/installing/linux/

 目标平台 Debian amd64 x64 12

 

 添加证书/软件源

```
 sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update

sudo apt-get install jenkins
```

安装Java组件

```
sudo apt update

sudo apt install fontconfig openjdk-17-jre
```

测试Java安装
```
java -version
```
启动/启用Jenkins服务
```
sudo systemctl enable jenkins

sudo systemctl start jenkins

sudo systemctl status jenkins
```

访问目标平台的8080端口，下面是一个Demo地址

http://215.10.1.134:8080/


初始化登录时需要使用一个生成的密码，在下列位置查询
```
cat /var/lib/jenkins/secrets/initialAdminPassword

```

如果包来源是Github 且使用国内没有打洞特殊线路支持的 需要在目标主机上给git挂个代理

## 如何使构建错误后重新构建

在 Jenkins 控制台中，Manage Jenkins > Plugins

搜索 Naginator 插件并安装。

然后前往所在项目的配置页面 例如：mesa/configure 划到最下面

Post-build Actions 选择 Retry build after failure

大概是下面这样的表单，按需填写即可


