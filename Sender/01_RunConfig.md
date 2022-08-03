# 서버 실행 및 설정

## 실행

일반 java 실행 명령과 같은 방식으로 실행한다.

```
java -jar sender.jar &
```

다른 java 실행 옵션들도 모두 유효하다.

```
java -Xms1024m -Xmx1024m -jar sender.jar &
```

### 설정

일반적인 스프링 부트 설정 방식과 동일하게 application.yml 파일을 실행하는 폴더와 같은 폴더에 생성해서 사용자 설정을 적용할 수 있다.

```
iotree:
  push:
    use-custom-notification: false
    android-channel-id: itp-default-channel

spring:
  cloud:
    discovery:
      client:
        simple:
          instances:
            core:
              - uri: http://localhost:8093

server:
  port: 8094

logging:
  config: classpath:logback.xml
```

- **iotree.push.use-custom-notification**  
  앱으로 보내지는 Push 메세지에 표준 notification 필드를 사용할 것인지 아니면 커스텀 notification 필드를 사용할 것인지 지정한다. 일반적으로는 false가 맞지만 안드로이드의 '수신'기능 구현을 위해서는 true로 설정해야 한다.
- **iotree.push.android-channel-id**  
  안드로이드로 보내는 Push에만 유효한 옵션이다. Android에는 앱에서 생성한 NotificationChannel로 Push를 보내야 Channel에 적용된 옵션들이 동작하기 때문에 사전에 정의된 Channel로 Push를 보내야 한다. 기본 채널의 ID는 `itp-default-channel`이다.
- **spring.cloud.discovery.client.simple.instances.core**  
  Core 서버의 URI를 지정한다. 배열 형태로 `uri`를 지정하여 Core 서버가 다중화되어 있는 경우 모두 기록해준다.
- **server.port**  
  서버의 포트를 설정한다.
- **logging.config**  
  logback 설정 파일을 지정한다. 따로 지정하지 않으면 현재 폴더의 아래에 log 폴더를 만들고 그 안에 매일 Rolling되는 로그 파일을 생성된다.
