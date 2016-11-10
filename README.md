## 安装 Java 语言的软件开发工具包
```
brew cask install java
```
或者在 [Oracle官网](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 中选择 Mac 版本 `jdk-8u111-macosx-x64.dmg` 下载并安装。

## 安装 Solr
安装的版本 >= 6.1.0:
```
brew install solr
```
## 启动 Solr
```
solr start
```
返回以下文字提示，则表示 solr 服务器安装成功，默认监听的端口号为 8983:
```
Waiting up to 30 seconds to see Solr running on port 8983 [\]
Started Solr server on port 8983 (pid=890). Happy searching!
```

## 在浏览器中访问可视化管理界面`Solr Admin`

solr 默认的访问URL为： [http://localhost:8983/solr/](http://localhost:8983/solr/)

## 创建一个名为 test 的 `core`
```
solr create -c test
```


## 安装 MySQL 数据库
```
brew install mysql
```
将root的密码修改为123456。或者其他你喜欢的密码。
```
create database solrdata;
use solrdata;
create table goods(id int not null auto_increment, name varchar(20) not null default '', number varchar(20) not null default '', updateTime timestamp not null default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP, primary key(id));
insert into goods(name, number)  values('鞋子', 100);
insert into goods(name, number)  values('衣服', 200);
insert into goods(name, number)  values('裤子', 300);
```

## 下载 MySQL 驱动

从 [MySQL 官方地址](http://dev.mysql.com/downloads/connector/j/) 下载 `mysql-connector-java` 驱动。或者直接运行一下命令获取 5.1.40 版本的驱动：
```
wget http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.40.tar.gz
```
将 jar 包复制到 `libexec/dist/`目录中：

```
tar -zxvf mysql-connector-java-5.1.40.tar.gz
cd mysql-connector-java-5.1.40
cp mysql-connector-java-5.1.40-bin.jar /usr/local/Cellar/solr/6.1.0/libexec/dist/
```

## 修改 solrconfig.xml 文件中的配置信息：

```
vi /usr/local/Cellar/solr/6.1.0/server/solr/test/conf/solrconfig.xml
```
* 将 `libexec/dist/` 中的 3 个相关 jar 包进入进来：
```
  <lib dir="${solr.install.dir}/libexec/dist/" regex="mysql-connector-java-5.1.40-bin.jar" />
  <lib dir="${solr.install.dir}/libexec/dist/" regex="solr-dataimporthandler-.*\.jar" />
```

* 将 连接 mysql 的相关配置信息添加进来：
```
<requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">  
　     <lst name="defaults">  
　        <str name="config">data-config.xml</str>  
　     </lst>  
　</requestHandler>
```

* 在同目录下新建data-config.xml文件：
```
<?xml version="1.0" encoding="UTF-8"?>  
<dataConfig>  
    <dataSource name="source1" type="JdbcDataSource" driver="com.mysql.jdbc.Driver" url="jdbc:mysql://localhost:3306/solrdata" user="root" password="123456" batchSize="-1" />  
　　<document>  
        <entity name="goods" pk="id"  dataSource="source1"   
                query="select * from  goods"  
                deltaImportQuery="select * from goods where id='${dih.delta.id}'"  
                deltaQuery="select id from goods where updateTime> '${dataimporter.last_index_time}'">  
  
　　　      <field column="id" name="id"/>  
　　　      <field column="name" name="name"/>  
           <field column="number" name="number"/>  
           <field column="updateTime" name="updateTime"/>  
　　　  </entity>  
　　</document>  
</dataConfig>

```


##  managed-schema配置field信息
```
vi /usr/local/Cellar/solr/6.1.0/server/solr/test/conf/managed-schema
```

新增以下信息：
```
    <field name="name" type="string" indexed="true" stored="false" />
    <field name="number" type="int" indexed="true" stored="false" />
    <field name="updateTime" type="date" indexed="true" stored="false" />
```

##  重启 solr 服务
```
solr restart
```