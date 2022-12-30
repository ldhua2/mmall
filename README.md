

# 项目性能测试报告

## 01-测试目的

主要是让开发者对hero_mall项目的性能负载和容量有个准确的认知。同时，协助技术管理者更好的管理业务系统性能质量，科学评估业务系统的负荷，拒绝盲目上线。

## 02-测试工具

![image-20220806185532353](项目性能测试报告/image-20220806185532353.png)

## 03-测试环境

### 3.1 环境

| 指标              | 参数 |
| ----------------- | ---- |
| 机器              | 4C8G |
| 集群规模          | 单机 |
| hero_mall_one版本 | 1.0  |
| 数据库            | 4C8G |

![image](https://user-images.githubusercontent.com/28390845/210051050-8c4749fb-e7b8-4f77-a6e9-8b4ed9929819.png)


### 3.1 设置启动参数

```bash
#!/bin/sh
export JAVA_HOME=/usr/local/hero/jdk1.8.0_261
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
#===========================================================================================
# init
#===========================================================================================
export SERVER="hero_web"
export JAVA_HOME
export JAVA="$JAVA_HOME/bin/java"
# 获取当前目录
export BASE_DIR=`cd $(dirname $0)/.; pwd`
# 默认加载路径
export DEFAULT_SEARCH_LOCATIONS="classpath:/,classpath:/config/,file:./,file:./config/"
# 自定义默认加载配置文件路径
export CUSTOM_SEARCH_LOCATIONS=${DEFAULT_SEARCH_LOCATIONS},file:${BASE_DIR}/conf/
#===========================================================================================
# JVM Configuration
#===========================================================================================
JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256 -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=${BASE_DIR}/logs/java_heapdump.hprof"
JAVA_OPT="${JAVA_OPT} -XX:-UseLargePages"
JAVA_OPT="${JAVA_OPT} -jar ${BASE_DIR}/${SERVER}-*.jar"
JAVA_OPT="${JAVA_OPT} ${JAVA_OPT_EXT}"
JAVA_OPT="${JAVA_OPT} --spring.config.location=${CUSTOM_SEARCH_LOCATIONS}"
# 创建日志文件目录
if [ ! -d "${BASE_DIR}/logs" ]; then
  mkdir ${BASE_DIR}/logs
fi
# 输出变量
echo "$JAVA ${JAVA_OPT}"
# 检查start.out日志输出文件
if [ ! -f "${BASE_DIR}/logs/${SERVER}.out" ]; then
  touch "${BASE_DIR}/logs/${SERVER}.out"
fi
#===========================================================================================
# 启动服务
#===========================================================================================
# 启动服务
echo "$JAVA ${JAVA_OPT}" > ${BASE_DIR}/logs/${SERVER}.out 2>&1 &
nohup $JAVA ${JAVA_OPT} hero_web.hero_web >> ${BASE_DIR}/logs/${SERVER}.out 2>&1 &
echo "server is starting，you can check the ${BASE_DIR}/logs/${SERVER}.out"
```



## 04-测试场景

测试场景一般情况下是都是最重要接口：验证hero_mall服务获取商品信息接口在不同并发规模的表现

**情况01-模拟低延时场景，**用户访问接口并发逐渐增加的过程。接口的响应时间为20ms，线程梯度：5、10、15、20、25、30、35、40个线程，5000次;

- 时间设置：Ramp-up period(inseconds)的值设为对应线程数
- 测试总时长：约等于20ms x 5000次 x 8 = 800s = 13分

**情况02-模拟高延时场景，**用户访问接口并发逐渐增加的过程。接口的响应时间为500ms，线程梯度：100、200、300、400、500、600、700、800个线程，200次; 

- 时间设置：Ramp-up period(inseconds)的值设为对应线程数的1/10；
- 测试总时长：约等于500ms x 200次 x 8 = 800s = 13分



## 05-测试结论

hero_web性能测试是针对重点功能，单机单节点服务进行压测，可以看到各个接口容量。本测试供给大家作为参考，如有不足或偏差，请指正！如果对性能有其他需求，可以进行集群扩容。例如:3节点、10节点、100节点...
