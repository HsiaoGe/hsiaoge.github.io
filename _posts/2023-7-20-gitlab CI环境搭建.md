# Windows下gitlab CI环境搭建

公司的项目代码用gitlab管理，不过发布体系建立混乱，需要统一的发布出口，研究了一下gitlab的CI/CD，把搭建的过程记录了一下。

## Runner安装

​	Runner就是CI里Pipeline的job的执行者，一般情况下每一个项目都需要注册一个Runner。

​	Runner能够拉取代码（用的是Token的权限），然后在项目根目录下执行在.gitlab-ci.yml里的script脚本语句来实现编译、测试和部署等其他功能。

​	因为是编译windows下的exe和dll，打开[Install GitLab Runner on Windows | GitLab](https://docs.gitlab.com/runner/install/windows.html)，只需要下载Windows的64版本即可，下载完成之后无需安装，找到一个位置创建一个英文目录，把下载的exe复制过去即可，注意这个目录就是runner的工作目录，后续不能再动了。



## Runner注册

首先你的账户权限必须是Maintainer，否则无法获取Runner的注册token。

![image-20230720221358155](/assets/images/image-20230720221358155.png)

打开`设置`→`CI/CD`，展开`Runner`，找到`URL`和`注册令牌`，然后再Runner目录下执行：

```powershell
.\gitlab-runner-windows-amd64.exe register --url 项目的URL -r 项目的Token -c config.toml --name Runner名称 --tag-list build --shell pwsh --executor shell
```

执行之后，命令行会再次需要确认刚才的配置输入，直接一直enter即可，然后执行安装runner服务：

```powershell
.\gitlab-runner-windows-amd64.exe install
```

需要注意，此时启动Runner，会以默认`system`的身份执行，这会导致本来能编译的项目，再Runner里编译出错。gitlab-runner也支持指定用户，但是实测没有生效，原因未知。正确做法是`win+R`输入`services.msc`打开服务管理窗口，找到`gitlab-runner`服务，打开属性：

![image-20230720223724654](/assets/images/image-20230720223724654.png)

输入当前的用户名和密码，然后启动服务即可，此时再次打开你的项目CI/CD的Runner页面，可以看到:

![image-20230720224042170](/assets/images/image-20230720224042170.png)

说明已经注册成功。



## 编写CI流水线配置文件

在项目的根目录下创建`.gitlab-ci.yml`文件，复制如下内容到其中：

```yml
stages:
  - build
  - upload

cache:
  paths:
    - bin/Release/*.dll
    - bin/Debug/*.dll

build_job:
  tags:
    - build
  stage: build
  script:
    - devenv.com .\project.sln /rebuild Release
    - devenv.com .\project.sln /rebuild Debug

upload_job:
  tags:
    - build
  stage: upload
  script:
    - new-item -itemtype directory -path "$env:RELEASE_DIR/$CI_PROJECT_NAME/$CI_COMMIT_REF_NAME/Release" -force
    - new-item -itemtype directory -path "$env:RELEASE_DIR/$CI_PROJECT_NAME/$CI_COMMIT_REF_NAME/Debug" -force
    - copy-item -path bin/Release/tlmv.* -destination "$env:RELEASE_DIR/$CI_PROJECT_NAME/$CI_COMMIT_REF_NAME/Release"
    - copy-item -path bin/Debug/tlmvd.* -destination "$env:RELEASE_DIR/$CI_PROJECT_NAME/$CI_COMMIT_REF_NAME/Debug"
```

具体语法见[GitLab CI/CD | GitLab](https://docs.gitlab.com/ee/ci/index.html)。

​		只简单分成了build和upload阶段，编译阶段利用devenv.com直接在命令行编译sln工程文件，编译完成之后把生成的动态库文件进行cache。不cache的话，会在build的job完成之后，立即清理生成的所有文件，无法保存编译好的动态库。upload阶段就是把编译好的动态库复制到其他地方，测试人员从这个统一的地方拿动态库进行测试和发布。

​	然后提交这个.gitlab-ci.yml文件到仓库，打开项目的CI/CD页面就能看到这个流水线正在被执行了。



### git config -f ......错误

出现这个错误，升级最新的powershell即可。