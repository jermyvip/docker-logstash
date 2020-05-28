# docker-logstash

docker 构建镜像 同步mysql 数据至ES

目录说明：
config --覆盖logstash 自带的yml配置文件 
lastvalue --增量数据实时同步技术文件
Dcokerfile -- 构建镜像文件 
logstash.conf -- 同步数据配置文件
mysql-connector-java-5.1.47.jar --链接mysql驱动、根据不同版本可自行下载

一、构建dockerfile：

#自行对照es版本选择logstash版本

FROM logstash:6.6.1

#使用root权限 否则没有权限修改增量数据计数文件

USER root

#删除logstash pipeline 自带的配置文档

RUN rm -f /usr/share/logstash/pipeline/logstash.conf

#覆盖logstash 自带的yml配置文件

ADD ./config/ /usr/share/logstash/config/

#拷贝自行编写的配置文件至pipeline文件下

ADD logstash.conf /usr/share/logstash/pipeline/logstash.conf

#拷贝增量计数文件至logstash文件下下

ADD ./lastvalue/ /usr/share/logstash/lastvalue/

#拷贝mysql驱动至lostash容器目录下

ADD mysql-connector-java-5.1.47.jar /usr/share/logstash/logstash-core/lib/jars/mysql/mysql-connector-java-5.1.47.jar

#设置增量同步计数文件权限

RUN chmod 0777 /usr/share/logstash/lastvalue && chmod 0777 /usr/share/logstash/lastvalue/customer.txt

#安装input插件

RUN logstash-plugin install logstash-input-jdbc

#安装output插件

RUN logstash-plugin install logstash-output-elasticsearch

#容器启动时执行的命令.(CMD 能够被 docker run 后面跟的命令行参数替换/运行配置文件根据自行拷贝的配置文件路径)

CMD ["-f", "/usr/share/logstash/pipeline/logstash.conf"]


2 构建镜像：

docker build -t logstashv1 .

3 运行镜像 

docker run logstashv1

