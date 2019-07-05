## Logback을 이용한 Redis Appender

#### 1) logback과 연동
프로젝트의 logback Appender를 통해서 Redis에 데이터를 전송할 수 있습니다.

</br>

#### pom.xml
pom.xml에 **logback-redis-appender** 라이브러리가 꼭 추가되어 있어야 합니다. 

해당 라이브러리가 정상적으로 추가되지 않으면 kafka로 전송이 되지 않을 뿐만 아니라 별도의 에러가 나지 않아서 원인을 찾기도 쉽지 않습니다.

``` xml
<!-- logstash -->
<dependency>
    <groupId>com.cwbase</groupId>
    <artifactId>logback-redis-appender</artifactId>
    <version>1.1.5</version>
</dependency>
``` 
</br>

#### logback.xml
**logback-redis-appender** 라이브러리 추가 후 redis관련 설정을 logback에 추가해 줍니다.

``` xml
<appender name="redis" class="com.cwbase.logback.RedisAppender">
    <host>127.0.01</host>
    <port>6379</port>
    <key>tomcat</key>
    <type>tmsengine</type>
    <source>local</source>
</appender>
 
 
<appender name="redis-logstash" class="ch.qos.logback.classic.AsyncAppender">
    <appender-ref ref="redis"/>
</appender>
 
 
<logger name="access_log" level="INFO" additivity="false">
    <appender-ref ref="console"/>
    <appender-ref ref="FILE"/>
    <appender-ref ref="redis-logstash"/>
</logger>
<logger name="com.sjb.tms" level="DEBUG" additivity="false">
    <appender-ref ref="console"/>
    <appender-ref ref="FILE"/>
    <appender-ref ref="redis-logstash"/>
</logger>
<root level="WARN">
    <appender-ref ref="console"/>
    <appender-ref ref="FILE"/>
    <appender-ref ref="redis-logstash"/>
</root>
``` 

</br>

주요 정보로는 다음과 같습니다.

|태 그|설 명|
|------|---|
|host,port|정보를 전송할 redis 주소|
|key|redis 사용하는 key|
|type,source|logstash 설정파일에서 field로 사용됩니다.|

</br>

#### 3) Redis 데이터 확인

실제 request를 발생 시켜서 logback을 통해 redis에 정상적으로 저장되었는지 확인해보면 됩니다.

redis-cli를 이용해서 접속해서 확인가능합니다. 또는 **FastoRedis** 같은 GUI 툴을 사용해도 됩니다.

``` config
%REDIS_HOME%/redis-cli
```

</br>

redis에 데이터가 List 형태로 쌓이기 때문에 **[GET KEY]** 형태로 조회 하면 출력이 되지 않습니다.

**[LRANGE KEY START STOP]** 형태로 범위로 조회 해야 합니다.

저는 logback에 key를 tomcat으로 설정했기 때문에 **LRANGE tomcat 0 3** 으로 조회를 했고 정상적으로 데이터가 들어간 것을 확인했습니다.

![image2018-10-26_17-42-7](https://user-images.githubusercontent.com/7076334/60695976-c1c10b00-9f1e-11e9-9768-7131357960a0.png)

</br>
redis에 저장된 데이터 형태는 대략 다음과 같고 source, type 같은 컬럼은 현재 logstash에서 사용되고 있습니다.

#### json
``` json
{
  "source": "local",
  "host": "SIMJUNBO",
  "path": null,
  "type": "tmsengine",
  "tags": [],
  "message": "[200][GET][http://localhost:8080/api/routing/routing_plans][0:0:0:0:0:0:0:1][message={test2}][accept-language={ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7}&cookie={SCOUTER=xv5ev5sj7a4qq; Idea-8ec0b4c3=f7759d29-a224-42b0-8d0f-e11a3f6ef968}&host={localhost:8080}&upgrade-insecure-requests={1}&connection={keep-alive}&accept-encoding={gzip, deflate, br}&user-agent={Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36}&accept={text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8}][][4][test2]",
  "@timestamp": "2018-10-26T15:08:25.536+0900",
  "logger": "access_log",
  "level": "INFO",
  "thread": "http-nio-8080-exec-1"
}
``` 
