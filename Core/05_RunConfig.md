# 서버 실행 및 설정

### 실행

일반 java 실행 명령과 같은 방식으로 실행한다.

```
java -jar core.jar &
```

다른 java 실행 옵션들도 모두 유효하다.

```
java -Xms1024m -Xmx1024m -jar core.jar &
```

### 설정

일반적인 스프링 부트 설정 방식과 동일하게 application.yml 파일을 실행하는 폴더와 같은 폴더에 생성해서 사용자 설정을 적용할 수 있다.

```yml
iotree:
  push:
    token-expiration-days: 60
    pull:
      enabled: true
      max-count: 100
    statistics:
      enabled: true
    reservation:
      enabled: true
      expiration-time: 3600

spring:
  cloud:
    discovery:
      client:
        simple:
          instances:
            sender:
              - uri: http://devsvr:8094
              - uri: http://devsvr2:8094
  datasource:
    driver-class-name: oracle.jdbc.driver.OracleDriver
    url: jdbc:oracle:thin:@devsvr:1521:ORCLCDB
    username: USERID
    password: SOMEPASSWD

server:
  port: 8093

logging:
  config: classpath:logback.xml
```

- **iotree.push.token-expiration-days**  
  사용되지 않는 Token의 만료 기간(단위: 일). 여기서 지정된 기간동안 사용되지 않는 토큰은 DB에서 삭제된다.
- **iotree.push.pull.enabled**  
  DB에서 푸시 요청을 가져와서 Sender로 전달하는 기능을 Enable할 것인지 지정. 다중화 되어있는 경우 각 인스턴스마다 선택적으로 설정할 수 있다. 여러 대가 enable 되어도 상관없다.
- **iotree.push.pull.max-count**  
  DB에서 푸시 요청을 가져올 때 한 번에 처리하는 요청 갯수. 일반적으로 100이면 충분하며 최대 500까지 지정가능하다.
- **iotree.push.statistics.enabled**  
  통계 집계 프로세스를 enable 시킬 것인지 지정. 다중화 되어있다면 그 중에 1대만 enable 시키고 나머지는 disable 시킨다.
- **iotree.push.reservation.enabled**  
  예약 처리 프로세스를 enable 시킬 것인지 지정. 다중화 되어있다면 그 중에 1대만 enable 시키고 나머지는 disable 시킨다.
- **iotree.push.reservation.expiration-time**  
  예약된 메시지의 만료 시간(단위: 초). 예약된 시간 이후로 여기서 지정된 시간만큼 지났으나 발송되지 못한 경우 발송 시도를 하지 않고 무시한다.
- **spring.cloud.discovery.client.simple.instances.sender**  
  Sender 서버의 URI를 지정한다. 배열 형태로 `uri`를 지정하여 Sender 서버가 다중화되어 있는 경우 모두 기록해준다.
- **spring.datasource.xxx**  
  DB 데이터 소스 설정이다. 예는 일반적인 Oracle DB의 경우에 대한 샘플이다.
- **server.port**  
  서버의 포트를 설정한다.
- **logging.config**  
  logback 설정 파일을 지정한다. 따로 지정하지 않으면 현재 폴더의 아래에 log 폴더를 만들고 그 안에 매일 Rolling되는 로그 파일을 생성된다.
