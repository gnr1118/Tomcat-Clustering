# Tomcat clustering on Windows-Apache Http Server as load balancer 

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

