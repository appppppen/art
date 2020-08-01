### 起因

今天遇到 tomcat 手动启动不成功，但是再 eclipse 里面是可以启动成功的。我一直开始以为是端口被占用了，最后才发现是这么一回事。首先说下我的 tomcat 是安装版的。免安装的 tomcat 双击 startup.bat 后，启动窗口一闪而过，而且 tomcat 服务未启动。这个原因就是：在启动 tomcat 是，需要读取环境变量和配置信息，缺少了这些信息，就不能登记环境变量，导致了 tomcat 的闪退。

- 1：在已解压的 tomcat 的 bin 文件夹下找到 startup.bat，右击->编辑。在文件头加入下面两行：

* SET JAVA_HOME=D:\Java\jdk1.7 （java jdk 目录）
* SET TOMCAT_HOME=E:\tomcat-7.0 （解压后的 tomcat 文件目录）

- 2.在已解压的 tomcat 的 bin 文件夹下找到 shutdown.bat，右击->编辑。在文件头加入下面两行：

* SET JAVA_HOME=D:\Java\jdk1.7 （java jdk 目录）
* SET TOMCAT_HOME=E:\tomcat-7.0 （解压后的 tomcat 文件目录）
