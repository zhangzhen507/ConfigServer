# ConfigServer
This is a demo configserver based on Spring Cloud Config Server.
It is for applications configurations management.

# How to use this demo configserver
http://blog.csdn.net/comeonzz/article/details/79122744

# Config Server Introduction
Config Server
Spring Cloud Config Server 作为配置中心服务端

拉取配置时更新 git 仓库副本，保证是最新结果 
支持数据结构丰富，yml, json, properties 等 
配合 eureke 可实现服务发现，配合 cloud bus 可实现配置推送更新 
配置存储基于 git 仓库，可进行版本管理 
简单可靠，有丰富的配套方案

建立一个简单的Spring Boot工程：ConfigServer 
和其他Spring Boot工程一样，只是ConfigServer稍加配置就可以使用Spring Cloud Config Server的功能了。 
pom.xml:

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.smart.configserver</groupId>
    <artifactId>ConfigServer</artifactId>
    <version>0.0.1</version>
    <packaging>jar</packaging>

    <name>ConfigServer</name>
    <description>Demo config server </description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Edgware.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>
配置Config Server的端口和配置仓库的路径 
这里配置仓库的路径是local目录git仓库

vi application.properties
server.port: 8888
spring.cloud.config.server.git.uri: file://home/zzhen/gitRepository/demoConfig
security.user.name: zzhen
security.user.password: 12345678
如果配置了security就需要导入maven依赖配置

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
在本地仓库有一个配置文件billing.properties：

cat /home/zzhen/gitRepository/demoConfig/billing.properties
spring.application.name = billing
spring.jdbc.host = localhost
spring.jdbc.port = 3306
spring.jdbc.user = root
spring.jdbc.password = root
启动ConfigServer程序然后在浏览器访问下面的URL可以获得相应的配置信息：可以是json格式也可以是yml格式。

curl http://192.168.137.11:8888/master/billing-profile.json
{
    "spring": {
        "application": {
            "name": "billing"
        },
        "jdbc": {
            "host": "localhost",
            "password": "root",
            "port": "3306",
            "user": "root"
        }
    }
}

curl http://192.168.137.11:8888/master/billing-profile.yml
spring:
  application:
    name: billing
  jdbc:
    host: localhost
    password: root
    port: '3306'
    user: root


curl http://192.168.137.11:8888/billing/profile/master
{
    "name": "billing",
    "profiles": ["profile"],
    "label": "master",
    "version": "de015cae2b021b8ac6f0d3e9ff273d3e9ac076aa",
    "state": null,
    "propertySources": [{
        "name": "file://home/zzhen/gitRepository/demoConfig/billing.properties",
        "source": {
            "spring.jdbc.password": "root",
            "spring.jdbc.host": "localhost",
            "spring.jdbc.user": "root",
            "spring.jdbc.port": "3306",
            "spring.application.name": "billing"
        }
    }]
}
注：访问启动的项目，通过URL路径匹配相应的配置文件，URL与配置文件的映射关系如下： 
/{application}/{profile}[/{label}] 
/{application}-{profile}.yml 
/{label}/{application}-{profile}.yml 
/{application}-{profile}.properties 
/{label}/{application}-{profile}.properties
