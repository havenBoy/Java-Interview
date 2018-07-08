# maven
> 主要介绍maven优缺点，以及maven的生命周期；

# maven是什么
> 是一个跨平台的项目管理工具，为开发人员提供项目完整的生命周期框架

## maven优缺点
- 优点：
  * 项目管理 管理项目的生命周期
  * jar包依赖 舍弃了以往臃肿的jar包的管理方式，极大的减少了项目大小
  * 提高项目的可复用性；
- 缺点
  * 学习的成本较高 
  * 以插件的方式提供，会影响开发集成环境的流畅度（ps:实在找不到其他的缺点了。。。）

  ## 相关命令
  - mvn clean  删除之前编译的文件
  - mvn compile 
  - mvn test
  - mvn package 
  - mvn install 把项目部署到本地仓库

  ## 构建步骤
  >  清除(clean) -> 编译(compile) -> 测试(test) -> 打包(package(jarwar)) -> 安装 -> 部署(install)

  ## maven 生命周期(三个)
  - clean 删除上次编译文件，即target目录下的文件
  - 默认 
    * 把resource下的文件输出在classpath下的目录下
    * 把java文件进行编译(compile)，生成相应的class文件
    * test阶段，运行测试用例
    * package阶段，把项目打成war jar包
    * install 把项目部署在本地仓库
    * deploy  把项目部署在远程仓库
  - site周期
    * 生成项目的站点文件
    * 把项目的站点文件部署在远程仓库