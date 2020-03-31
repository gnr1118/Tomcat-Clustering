Tomcat clustering on Windows-Apache Http Server as load balancer 

There are three main steps for Tomcat clustering on Windows:

- Apache Http Server install & config
- Apache Tomcat install & config
- Apache Tomcat clustering-Apache Http Server as load balancer

## Apache Http Server install & config

### Install

- Download [ApacheHaus](https://www.apachehaus.com/cgi-bin/download.plx) version, you can choose the latest version 2.4.41
- If you haven't install the VC14 package, go to the [microsoft download page](https://www.microsoft.com/zh-tw/download/details.aspx?id=48145) and select the appropriate version to install
- unzip the Apache24 setup file and put the Apache24 folder whereever you want
- Open command line and change the directory to the yourowndirectory\Apache24\bin, then execute the httpd.exe

```
# change directory to Apache24\bin
$ cd Apache24\bin

# execute httpd.exe
$ httpd.exe
```

- If  port conflict occurs, it means one of port 80 or port 443 is currently using by other service, you can change the config in Apache24\conf\httpd.conf file and excute the httpd.exe again

```
# check 80 port use situation
$ netstat -nao |find "0.0.0.0:80"

# check 443 port use situation
$ netstat -nao |find "0.0.0.0:443"

# change the listening port in Apache24\conf\httpd.conf to 8080
Listen 8080

# execute httpd.exe
$ httpd.exe
```

- if you can see the [home page](http://localhost:8080) of you Apache Http Server, it means your Apache Http Server executes normally

### Config

- Change the **SRVROOT**  variable, add the **SRVNAME**  and the **DOCUMENTROOT**  variable to Apache24\conf\httpd.conf 

```
Define SRVROOT "yourdirectory/Apache24"
Define SRVNAME "localhost:yourportsetting"
Define DOCUMENTROOT "yourdirectory/Apache24/htdocs"
```

- if you encounter port conflict, or you just want to change the http listening port

```
# change the listen port in Apache24\conf\httpd.conf to 8080
Listen 8080
```

- Change the **ServerName** in Apache24\conf\httpd.conf using previous added **SRVNAME** variable

```
# Change the ServerName in Apache24\conf\httpd.conf
ServerName ${SRVNAME}
```

- Change the **DocumentRoot** in Apache24\conf\httpd.conf using previous added **DOCUMENTROOT** variable, **DocumentRoot** is used for setting the static resource 

```
# Change the DocumentRoot in Apache24\conf\httpd.conf
DocumentRoot ${DOCUMENTROOT}
```

- If you want to map the http URL path to your server's directory, you can use **Alias** 

```
# Use Alias to mapping URL path to real directory
Alias "/web" "D:/web" 
<Directory "/web">
	Options None
	AllowOverride None
	Require all granted
</Directory>
```

- Install, uninstall the Apache Http Server to Windows service

```
# use administrator to open the command line prompt
# change directory to Apache24\bin
$ cd Apache24\bin

# install the Apache Http Server to Windows service
$ httpd.exe -k install -n "MyServiceName"

# uninstall the Apache Http Server service
$ httpd.exe -k uninstall -n "MyServiceName"
```

- Test the service's configuration file, start and stop the service

```
# test the service's cinfiguration file
$ httpd.exe -n "MyServiceName" -t

# start the service
$ httpd.exe -k start -n "MyServiceName"

# stop the service
$ httpd.exe -k stop -n "MyServiceName"

# use Windows service command
# start service
$ net start MyServiceName
# stop service
$ net stop MyServiceName
```

## Apache Tomcat install & config

### prerequisite

- you need to install the JDK in your machine, and add environment variables such like JAVA_HOME,  CLASSPATH and Path, check [this website](https://caterpillar.gitbooks.io/javase6tutorial/content/c2.html)

### Install

- Download [Tomcat Version 9.0.21](https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.21/bin/apache-tomcat-9.0.21-windows-x64.zip) 
- Unzip the file and put the directory whereever you want, suggesting put the directory under D:\ and rename the directory to Tomcat9
- Set CATALINA_HOME environment variable to D:\Tomcat9

```
# using system administrator cmd to set CATALINA_HOME environment variable
$ setx /M CATALINA_HOME "D:\Tomcat9.0"
```

- Add D:\Tomcat9\bin to Path environment variable 

```
# using system administrator cmd to add path to Path environment variable
$ setx /M Path "%CATALINA_HOME%\bin;%Path%"
```

- Verify Tomcat install by executing startup.bat, if execute successfully, the Tomcat window would remain opening until execute shutdown.bat to close Tomcat

```
# using cmd to start Tomcat
$ startup.bat

# using cmd to close Tomcat
$ shutdown.bat
```

- if you can see the [home page](http://localhost:8080) of your Apache Tomcat Server, it means your Apache Tomcat Server executes normally 

### Config

- If you encounter port conflict, or you just want to change the tomcat listening port

```
# edit the D:\Tomcat9\conf\server.xml and change the port setting
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

- Install, Uninstall Tomcat to Windows Service

```
# install the Apache Tomcat Server to Windows service
$ service.bat install "MyServiceName"

# uninstall the Apache Tomcat Server service
$ service.bat uninstall "MyServiceName"
```

- Start and Stop the Windows Service

```
# using system administrator cmd

# start service
$ net start MyServiceName

# stop service
$ net stop MyServiceName
```

## Apache Tomcat clustering-Apache Http Server as load balancer

### Config

#### Apache Http Server

- Download [mod_jk](http://archive.apache.org/dist/tomcat/tomcat-connectors/jk/binaries/windows/tomcat-connectors-1.2.39-windows-x86_64-httpd-2.4.x.zip) and add it to yourowndirectory\Apache24\modules folder
- Add wrokers.properties to yourowndirectory\Apache24\conf folder and set tomcat workers

```
# wrokers.properties setting
worker.list=balancer,stat

worker.tomcat1.type=ajp13
worker.tomcat1.host=host1
worker.tomcat1.port=8009
worker.tomcat1.retries=1

worker.tomcat2.type=ajp13
worker.tomcat2.host=host2
worker.tomcat2.port=8009
worker.tomcat2.retries=1

worker.balancer.type=lb
worker.balancer.balance_workers=tomcat1,tomcat2

worker.stat.type=status
```

- Config yourowndirectory\Apache24\conf\httpd.conf adding mod_jk settings

```
# add these settings to the end of httpd.conf
LoadModule    jk_module  modules/mod_jk.so
JkWorkersFile conf/workers.properties
JkLogFile     logs/mod_jk.log
JkLogLevel    emerg
JkLogStampFormat "[%Y-%m-%d %H:%M:%S.%Q] "
JkRequestLogFormat     "%m %U %w %R %T"
JkOptions     +ForwardKeySize +ForwardURICompat -ForwardDirectories

JkMount  /jk-status  stat
JkMount  /*  balancer
```

#### Apache Tomcat Server

- Config D:\Tomcat9\conf\server.xml adding AJP connector and jvmRoute attribute

```
# add this tag under <Service name="Catalina"> tag
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

# add jvmRoute attribute to <Engine name="Catalina" defaultHost="localhost"> tag
# jvmRoute attribute value depend on which worker that it is
<Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat1">
```

