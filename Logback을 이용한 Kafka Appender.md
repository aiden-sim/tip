## Logback을 이용한 Kafka Appender

#### 1) logback과 연동
프로젝트의 logback Appender를 통해서 Kafka Broker에 데이터를 전송할 수 있습니다.

</br>

#### pom.xml
pom.xml에 **logback-kafka-appender,  logstash-logback-encoder** 라이브러리가 꼭 추가되어 있어야 합니다. 해당 라이브러리가 정상적으로 추가되지 않으면 kafka로 전송이 되지 않을 뿐만 아니라 별도의 에러가 나지 않아서 원인을 찾기도 쉽지 않습니다.

``` xml
<!-- kafka -->
<dependency>
   <groupId>com.github.danielwegener</groupId>
   <artifactId>logback-kafka-appender</artifactId>
   <version>0.1.0</version>
</dependency>
 
<!-- logstash-encoder -->
<dependency>
   <groupId>net.logstash.logback</groupId>
   <artifactId>logstash-logback-encoder</artifactId>
   <version>4.6</version>
   <exclusions>
      <exclusion>
         <artifactId>logback-core</artifactId>
         <groupId>ch.qos.logback</groupId>
      </exclusion>
   </exclusions>
</dependency>
``` 

</br>

**logstash-logback-encoder** 라이브러리 내부에서 **logback-core**를 사용하고 있기 때문에 버전 충돌이 날 수도 있습니다.

[Intellij에서 Maven Helper Plugin을 이용한 의존 라이브러리 버전 관리 ](https://github.com/simjunbo/tip/blob/master/Intellij%EC%97%90%EC%84%9C%20Maven%20Helper%20Plugin%EC%9D%84%20%EC%9D%B4%EC%9A%A9%ED%95%9C%20%EC%9D%98%EC%A1%B4%20%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC%20%EB%B2%84%EC%A0%84%20%EA%B4%80%EB%A6%AC.md)를 참고하면 의존 라이브러리를 손쉽게 관리할 수 있습니다.


</br>

#### logback.xml
**logback-kafka-appender,  logstash-logback-encoder**  라이브러리 추가 후 관련 설정을 logback에 추가해 줍니다.

``` xml
<property name="APPENDER_KEY" value="tomcat"/>
<property name="APPENDER_TYPE" value="tdaapi"/>
<property name="APPENDER_SORUCE" value="dev"/>
 
<appender name="kafka" class="com.github.danielwegener.logback.kafka.KafkaAppender">
   <producerConfig>bootstrap.servers=localhost:9092</producerConfig>
   <topic>log-tomcat</topic>
   <encoder class="com.github.danielwegener.logback.kafka.encoding.LayoutKafkaMessageEncoder">
      <!-- JSON -->
      <layout class="net.logstash.logback.layout.LogstashLayout">
         <includeContext>false</includeContext>
         <includeCallerData>false</includeCallerData>
         <customFields>{"key":"${APPENDER_KEY}","type":"${APPENDER_TYPE}", "source":"${APPENDER_SORUCE}"}</customFields>
      </layout>
      <charset>UTF-8</charset>
      <!-- TEXT
           <layout class="ch.qos.logback.classic.PatternLayout">
               <pattern>[%d{yyyy-MM-dd HH:mm:ss}][%m%n]</pattern>
           </layout>
           -->
   </encoder>
</appender>
<appender name="kafka-async" class="ch.qos.logback.classic.AsyncAppender">
   <appender-ref ref="kafka"/>
</appender>
 
<logger name="access_log" level="INFO" additivity="false">
   <appender-ref ref="kafka-async"/>
</logger>
<logger name="com.sjb" level="DEBUG" additivity="false">
   <appender-ref ref="kafka-async"/>
</logger>
<root level="ERROR">
   <appender-ref ref="kafka-async"/>
</root>
``` 

</br>

주요 정보로는 다음과 같습니다.

</br>

|태 그|설 명|
|------|---|
|\<producerConfig\>bootstrap.servers=localhost:9092\<\/producerConfig\>|데이터를 전송할 kafka broker (여러개 인경우 콤마(,)로 구분)|
|\<topic\>log-tomcat\<\/topic\>|데이터를 전송할 topic|
|\<layout class="net.logstash.logback.layout.LogstashLayout"\>|Logstash의 json format 으로 메시지 출력|
|\<customFields\>\<\/customFields\>|customFields 태그를 사용하면 json에 특정 데이터 추가 가능|
|\<layout class="ch.qos.logback.classic.PatternLayout"\>|text format 으로 메시지 출력|
   
</br>

#### 2) kafka 데이터 확인

실제 request를 발생 시켜서 logback을 통해 kafka에 정상적으로 저장되었는지 확인해보면 됩니다.
쓸만한 GUI 툴이 없어서 kafka에서 제공해 주는 명령어를 사용했습니다.

``` config
%KAFKA_HOME%/bin/kafka-console-consumer.sh \
--bootstrap-server localhost:9092 \
--topic log-tomcat --from-beginning
```

</br>

kafka에 저장된 json format 데이터는 다음과 같고, layout을 **LogstashLayout**으로 사용하면 됩니다. 

**customFields**로 추가된 key, type, source 등이 같이 추가된 것을 확인하 수 있습니다.
#### json
``` json
{
  "@timestamp": "2019-06-04T15:39:55.269+09:00",
  "@version": 1,
  "message": "[200][POST][http://sjb.co.kr/api/event/list]",
  "logger_name": "access_log",
  "thread_name": "http-apr-9188-exec-5",
  "level": "INFO",
  "level_value": 20000,
  "req.requestURI": "/api/event/list",
  "req.remoteHost": "127.0.0.1",
  "req.requestURL": "http:///sjb.co.kr/api/event/list",
  "req.userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/57.0.2987.110 Safari/537.36",
  "key": "tomcat",
  "type": "wmsadmin",
  "source": "dev"
}
``` 

</br>

kafka에 저장된 text format (사용자가 지정한 패턴) 데이터는 다음과 같고, layout을 PatterLayout으로 사용하면 됩니다.
#### text
``` 
[2019-06-04 17:50:28][[404][GET][http://localhost:8080/][0:0:0:0:0:0:0:1][][accept-language={ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7}&cookie={SCOUTER=z3l276556f6qo0; Idea-8ec0b4c3=f7759d29-a224-42b0-8d0f-e11a3f6ef968}&host={localhost:8080}&upgrade-insecure-requests={1}&connection={keep-alive}&accept-encoding={gzip, deflate, br}&user-agent={Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.169 Safari/537.36}&accept={text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3}][][19][{"httpStatus":"NOT_FOUND","exceptionMessage":"Api is not found","exception":"com.sjb.exception.ApiException","httpCode":404}] ]
```
