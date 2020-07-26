### 起因
今天遇到tomcat手动启动不成功，但是再eclipse里面是可以启动成功的。我一直开始以为是端口被占用了，最后才发现是这么一回事。首先说下我的tomcat是安装版的。免安装的tomcat双击startup.bat后，启动窗口一闪而过，而且tomcat服务未启动。这个原因就是：在启动tomcat是，需要读取环境变量和配置信息，缺少了这些信息，就不能登记环境变量，导致了tomcat的闪退。
* 1：在已解压的tomcat的bin文件夹下找到startup.bat，右击->编辑。在文件头加入下面两行：
- SET JAVA_HOME=D:\Java\jdk1.7 （java jdk目录）
- SET TOMCAT_HOME=E:\tomcat-7.0 （解压后的tomcat文件目录）

* 2.在已解压的tomcat的bin文件夹下找到shutdown.bat，右击->编辑。在文件头加入下面两行：
- SET JAVA_HOME=D:\Java\jdk1.7 （java jdk目录）
- SET TOMCAT_HOME=E:\tomcat-7.0 （解压后的tomcat文件目录）