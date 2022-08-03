# 서버 실행 및 설정

### 실행

일반 java 실행 명령과 같은 방식으로 실행한다.

```
java -Djava.security.egd=file:/dev/./urandom -jar feedback.jar &
```

다른 java 실행 옵션들도 모두 유효하다.

```
java -Djava.security.egd=file:/dev/./urandom -Xms1024m -Xmx1024m -jar feedback.jar &
```

### 설정

일반적인 스프링 부트 설정 방식과 동일하게 application.yml 파일을 실행하는 폴더와 같은 폴더에 생성해서 사용자 설정을 적용할 수 있다.

```yml
spring:
  cloud:
    discovery:
      client:
        simple:
          instances:
            core:
              - uri: http://devsvr:8093
              - uri: http://devsvr2:8093

server:
  port: 8092
  ssl:
    enabled: true
    key-store: keystore.p12
    key-store-type: PKCS12
    key-store-password: sompasswd
    key-alias: iotree

# Logback Config
logging:
  config: classpath:logback.xml
```

- **spring.cloud.discovery.client.simple.instances.core**  
  Core 서버의 URI를 지정한다. 배열 형태로 `uri`를 지정하여 Core 서버가 다중화되어 있는 경우 모두 기록해준다.
- **server.port**  
  서버의 포트를 설정한다.
- **server.ssl.xxx**  
  https 접속을 위한 인증서 설정이다.
- **logging.config**  
  logback 설정 파일을 지정한다. 따로 지정하지 않으면 현재 폴더의 아래에 log 폴더를 만들고 그 안에 매일 Rolling되는 로그 파일을 생성된다.
