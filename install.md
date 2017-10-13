# 开发环境安装手册
1. 工程文件统一安装路径为:D:\java-develop\project 源代码环境
2. 网页访问 https://github.com/chenyanxu/admin-parent，
   顺序是：
   - kalix-parent
   - framework-parent
   - middleware-parent
   - admin-parent
   - oa-parent
   - common-parent
   - schedule-parent
   - tools-parent
   依次下载和编译。
3. 下载过程
   以oa-parent为例，打开 https://github.com/chenyanxu/oa-parent，点击clone，复制到剪贴板。桌面 IntelliJ IDEA 15.0.2 点击checkout from Version Control，打开github，打开 https://github.com/chenyanxu/oa-parent.git，提示框 点击NO。
4. 编译过程，打开 D:\java-develop\project\oa-parent\pom.xml，右侧maven setting 打开设置，按照如下方法
倒数第三行，修改成
 D:/java-develop/tools/apache-maven-3.3.9
倒数第二行，Override勾选，修改成
D:\java-develop\tools\apache-maven-3.3.9\conf\setting.xml
点击右上角齿轮图标group Modules，选择Lifecycle下install 双击。

5. 更新时，VCS-->git --》pull ，然后如上install。上面过程是初次机器配置，以后只要更新即可。

6. karaf统一部署在：D:\java-develop\tools\apache-karaf-4.0.5路径下。
7. 安装运行环境：feature:repo-add mvn:com.kalix.tools/tools-karaf-features/1.0.0-SNAPSHOT/xml/features
8. 输入feature:install -v kalix-base activiti couchdb，此时开发环境以及安装成功
    ********************************************************************
    *** WARNING: Wicket is running in DEVELOPMENT mode.              ***
    ***                               ^^^^^^^^^^^                    ***
    *** Do NOT deploy to your live server(s) without changing this.  ***
    *** See Application#getConfigurationType() for more information. ***
    ********************************************************************
