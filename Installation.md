# 설치

## Hive 1.2.2

Hive는 기본 `메타데이터 저장소`로 derby를 사용하지만, 외부 데이터 베이스로 MySQL, PostgreSQL를 사용할 수 있다.

MySQL은 메타데이터를 저장할 노드에만 설치하면 된다.

### MySQL 설치

```bash
yum -y install http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm

yum -y install mysql-community-server

systemctl start mysqld
systemctl enable mysqld
```

최초 실행시 아래처럼 임시 비밀번호를 조회하여 접속할 수 있다.

```bash
cat /var/log/mysqld.log

## [Note] A temporary password is generated for root@localhost: ******
```

접속 후 비밀번호를 재설정 해줘야 한다.

```bash
mysql -u root -p

ALTER USER 'root'@'localhost' IDENTIFIED BY '*****';
FLUSH PRIVILEGES;
SET GLOBAL validate_password_policy=LOW;
SET GLOBAL validate_password_length=4;

use mysql
CREATE USER 'hive'@'%' IDENTIFIED BY 'hive';
GRANT ALL ON *.* TO 'hive'@'%' IDENTIFIED BY 'hive';
FLUSH PRIVILEGES;
exit
```

hive 접속 테스트
```bash
mysql -u hive -p
```

### Hive 다운로드

```bash
wget http://mirror.apache-kr.org/hive/hive-1.2.2/apache-hive-1.2.2-bin.tar.gz
tar xvf apache-hive-1.2.2-bin.tar.gz
mv apache-hive-1.2.2-bin /opt/hive-1.2.2
chown -R hive:hadoop /opt/hive-1.2.2
chmod -R g+w /opt/hive-1.2.2
```

### 설정

```bash
vim /etc/profile.d/hive.sh
```
```bash
export HIVE_HOME=/opt/hive-1.2.2
export PATH=$PATH:$HIVE_HOME/bin:$HIVE_HOME/conf
```

```bash
source /etc/profile.d/hive.sh
cd $HIVE_HOME/conf

cp hive-default.xml.template hive-site.xml
cp hive-env.sh.template hive-env.sh
cp hive-exec-log4j.properties.template hive-exec-log4j.properties
cp hive-log4j.properties.template hive-log4j.properties
```

설정파일 수정 :

```bash
vim hive-env.sh
```
```bash
export HADOOP_HOME=/opt/hadoop-2.7.7
export HIVE_CONF_DIR=/opt/hive-1.2.2/conf
```

```bash
vim hive-site.xml
```
```xml
...
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://centos-namenode-1/hive?createDatabaseIfNotExist=true</value>
    <description>JDBC connect string for a JDBC metastore</description>
</property>
<property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
</property>
 <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>hive</value>
    <description>Username to use against metastore database</description>
</property>
<property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>hive</value>
    <description>password to use against metastore database</description>
</property>
...
```

JDBC 다운로드 :

```bash
cd $HIVE_HOME/lib
wget http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.27.tar.gz
tar xvf mysql*.gz
cp mysql-connector-java-5.1.27/*.jar $HIVE_HOME/lib
```

HDFS hive 계정 및 디렉토리 생성 :

```bash
hdfs dfs -mkdir -p /user/hive/warehouse
hdfs dfs -chown -R hive:hadoop /user/hive
hdfs dfs -chmod -R g+w /user/hive/
```

Mysql 메타데이터 테이블 생성 :

```bash
$HIVE_HOME/bin/schematool -initSchema -dbType mysql
```

접속 테스트 :

```bash
hive
```

## 에러해결

`Exception in thread "main" java.lang.RuntimeException: java.lang.IllegalArgumentException: java.net.URISyntaxException: Relative path in absolute URI: ${system:java.io.tmpdir%7D/$%7Bsystem:user.name%7D` :

다음을 `hive-site.xml` 끝에 추가한다 

```xml
<property>
    <name>system:java.io.tmpdir</name>
    <value>/tmp/hive/java</value>
</property>
<property>
    <name>system:user.name</name>
    <value>${user.name}</value>
</property>
```

`$SPARK_HOME/lib/spark-assembly-*.jar` 찾을 수 없다 에러, Spark 2.0 버전과의 호환성 문제, `$HIVE_HOME/bin/hive` 스크립트 수정

```vim
vim $HIVE_HOME/bin/hive
```

```bash
# add Spark assembly jar to the classpath
if [[ -n "$SPARK_HOME" ]]
then
  sparkAssemblyPath=`ls ${SPARK_HOME}/jar/*.jar`
  CLASSPATH="${CLASSPATH}:${sparkAssemblyPath}"
fi
```