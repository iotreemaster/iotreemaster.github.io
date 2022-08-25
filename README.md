# Android SDK 사용 가이드

## 구현 가이드

SDK 사용을 위해서 다음 순서대로 구현을 진행한다.

1. 프로젝트 생성 후 아래의 **라이브러리 설정**을 수행.
2. **알림 기본 옵션 설정** 수행. (Optional)
   - AndroidManifest.xml 파일에서 설정.
   - ~~또는 코드로 설정 가능.~~
3. **SDK 초기화**를 수행.
4. **토큰 발급**을 수행.
5. **토픽 구독**을 수행. (Optional)
6. launcher activity의 onCreate()에서 **수신 메시지 사후 처리** 수행.
7. Custom Receiver 구현. (Optional)
   - 기본 알림(Notification)을 사용하지 않고 기본 동작의 수정을 원하는 경우 Custom Receiver를 구현한다.
8. 기타 API의 경우 설명을 참조하여 필요 시 사용하면 된다.

각 항목의 자세한 사항은 아래 내용 참조.

## 라이브러리 설정

1. Firebase 콘솔에 접속하여 Firebase 프로젝트를 생성한다.
2. 푸시 알림을 받을 안드로이드 앱 프로젝트를 로컬에 생성한다.
3. 1번에서 생성한 Firebase 프로젝트에 2번에서 생성한 안드로이드 앱을 등록한다.

   - 콘솔 메인 화면이나 프로젝트 설정에서 앱 등록을 클릭한다.
   - 안드로이드 패키지 이름(필수), 앱 닉네임, SHA-1(디버그 서명 인증서)을 입력한다.
   - google-services.json 파일을 다운로드 하고 안드로이드 앱 프로젝트의 app 폴더에 저장한다.
   - build.gradle(project)에 google service plugin을 추가한다.(버전은 최신 버전으로 적용)

   ```
   buildscript {
       repositories {
           google()
           mavenCentral()
       }
       dependencies {
           classpath "com.android.tools.build:gradle:4.2.2"
           classpath "com.google.gms:google-services:4.3.8"
       }
   }

   allprojects {
       repositories {
           google()
           mavenCentral()
       }
   }
   ```

   - build.gradle(app)에 google service gradle plugin을 추가한다.
   - dependencies에는 firebase SDK를 추가한다.(버전은 최신 버전으로 적용)

   ```
   plugins {
       id 'com.android.application'
       id 'com.google.gms.google-services'
   }

   dependencies {
       ...
       implementation platform('com.google.firebase:firebase-bom:28.2.1')
       implementation 'com.google.firebase:firebase-messaging'
   }
   ```

   - 자세한 사항은 https://firebase.google.com/docs/cloud-messaging/android/client?hl=ko 참고!

4. aar 파일을 다운로드 한다.
   - https://github.com/iotreemaster/iotreemaster.github.io/raw/main/iotreePush.aar
5. app/libs에 다운로드한 aar 파일을 저장하고 build.gradle(app)에 라이브러리를 추가해준다.

   ```
   repositories {
       flatDir {
           dirs 'libs'
       }
   }

   dependencies {
       ...
       implementation name: 'iotreePush', ext: 'aar'
   }
   ```

## API 사용 방법

### SDK 초기화

- 푸시 알림을 받기 위해서 가장 먼저 필수적으로 해야하는 과정으로 push SDK를 초기화한다.
- IoTreePush.initialize()를 호출하여 초기화하고 context, appKey, feedbackBaseUrl을 매개변수로 넘겨준다.

  - context: Android Context.
  - appKey: ioTree Push 서버에서 발급하는 고유한 키 값(String).
  - feedbackBaseUrl: 서버 url.

  ```java
  IoTreePush.initialize(context, appKey, feedbackBaseUrl);
  ```

### 토큰 발급

- Push 수신을 위한 토큰을 발급 받아서 등록한다.
- 비동기 방식으로 동작하며 완료 시 전달된 callback의 메소드를 호출해준다.

  - context: Android Context.
  - userId: 서비스에 따른 사용자 식별자 값. (서비스 구현에 따라 어떤 값이든 될 수 있다. 예를 들어 서비스 Login ID 같은 값을 사용할 수 있다.)
  - groups
    - 사용자가 속한 그룹
    - 없으면 null, 속한 그룹의 알림을 받기 위해서는 Set으로 그룹 값을 넘겨줘야한다. ex) Android, Ios, sales 등
  - callback: 완료 시 호출되는 callback 객체(발급된 토큰을 확인할 수 있다.)

  ```java
  interface RegisterTokenCallback {
      void onRegister(IoTreePushResult result, String token);
  }

  IoTreePush.registerToken(context, userId, groups, new RegisterTokenCallback() {
      @Override
      public void onRegister(IoTreePushResult result, String token) {
          if (!result.isSuccess()) {
              int code = result.getCode();
          }

          Log.d("Push", token);
     }
  });
  ```

### 토큰 해제 (Optional)

- push 서버에 등록된 토큰을 해제한다.
- 비동기 방식으로 동작하며 완료 시 전달된 callback의 메소드를 호출해준다.

  - callback: 완료 시 호출되는 callback 객체.

  ```java
  interface UnregisterTokenCallback {
      void onUnregister(IoTreePushResult result);
  }

  IoTreePush.unregisterToken(new UnregisterTokenCallback() {
      @Override
      public void onUnregister(IoTreePushResult result) {
          if (!result.isSuccess()) {
              int code = result.getCode();
          }
      }
  });
  ```

### 토픽 구독 (Optional)

- 원하는 토픽을 선택하여 구독 요청을 한다. 토픽으로 전송된 모든 메시지를 수신할 수 있다.

  - topic: 토픽
  - callback: 완료 시 호출되는 callback 객체.

  ```java
  interface SubscribeTopicCallback {
      void onSubscribe(IoTreePushResult result);
  }

  IoTreePush.subscribeTopic(topic, new SubscribeTopicCallback() {
      @Override
      public void onSubscribe(IoTreePushResult result) {
          if (!result.isSuccess()) {
              int code = result.getCode();
          }
      }
  });
  ```

### 토픽 구독 해제 (Optional)

- 지정된 토픽 구독을 해제한다. 토픽 구독을 해제하면 토픽으로 메세지를 발송해도 수신이 되지 않는다.

  - topic: 토픽
  - callback: 완료 시 호출되는 callback 객체.

  ```java
  interface UnsubscribeTopicCallback {
      void onUnsubscribe(IoTreePushResult result);
  }

  IoTreePush.unsubscribeTopic(topic, new UnsubscribeTopicCallback() {
      @Override
      public void onUnsubscribe(IoTreePushResult result) {
          if (!result.isSuccess()) {
              int code = result.getCode();
          }
      }
  });
  ```

### 수신 메시지 사후 처리 (Optional)

- 앱이 백그라운드 상태에서 수신된 알림(Notification) 클릭 시 지정된 launcher activity가 실행되는데 이 activity의 onCreate()에서 호출해준다.
- '수신확인' 처리를 하지 않는다면 호출하지 않아도 되지만 무조건 호출해도 무방하기 때문에 가능하면 호출해주도록 한다.
  ```java
  IoTreePush.handleNotification(this);
  ```

### 알림 기본 옵션 설정 (Optional)

- Notification과 NotificationChannel에 대한 기본 옵션을 설정한다.
- 필수는 아니며 설정하지 않으면 기본 설정이 사용된다. 부분적으로 필요한 옵션만 설정 가능하다.
- 기본으로 제공하는 NotificationChannel의 이름과 설명을 설정한다.

- Notification의 아이콘, 색상, 알림음, 포그라운드 알림 여부를 설정한다.
- 기본 값(기기나 버전의 시스템에 따라 기본 값이 다를 수 있다.)
    - 아이콘 : 앱 아이콘(Androidmanifest의 application 태그에 설정된 icon)

    - 색상 : 시스템에 설정된 색상

    - 알림음 : 시스템 기본 알림 소리

    - 포그라운드 알림 : false

- 알림음의 경우, res/raw에 저장된 로컬 리소스만 사용가능 하고 value는 파일의 이름으로 설정한다
- 기본적으로 앱이 포그라운드 상태이면 Notification을 띄우지 않지만 SDK로 하여금 Notification을 띄우도록 설정하려면 enableForeground(true)로 설정한다.
  - enable_foreground가 false인 경우 custom receiver를 구현하지 않았다면 앱에서는 메시지가 수신된 걸 알 수 없으므로 enable_foreground를 false로 
  설정한다면 custom receiver 구현을 권장한다.
- Androidmanifest.xml 파일에서 설정한다. (Runtime에 설정할 수 없다.)
- ~~Runtime에 코드로 설정하거나 AndroidManifest.xml 파일에 메타 데이터로 정의할 수 있다. 만약 둘 다 설정된 경우 코드로 설정된 값이 우선 적용된다.~~

  ```xml
  <meta-data android:name="com.google.firebase.messaging.default_notification_icon"
              android:resource="@drawable/default_notification_icon" />
  <meta-data android:name="com.google.firebase.messaging.default_notification_color"
              android:resource="@color/default_notification_color" />
  <meta-data android:name="kr.co.iotree.push.notification.default_notification_sound"
              android:value="noti" />
  <meta-data
  android:name="kr.co.iotree.push.notification.foreground_enabled"
              android:value="false" />
  ```

  ~~또는~~

  ```
  IoTreeNotificationOptions options =
      new IoTreeNotificationOptions.Builder(context)
          .setChannelName(channelName) // 기본 채널 아이디
          .setChannelDesc(channelDesc) // 기본 채널 설명
          .setIcon(R.drawable.default_notification_icon) // 알림 아이콘
          .setColor(R.color.default_notification_color) // 알림 색상
          .setSound(rawSoundName) // 알림음
          .enableForeground(true) // 포그라운드 알림 노출
          .build();

  IoTreePush.setDefaultOptions(context, options);
  ```

### Custom Receiver 구현

직접 알림을 생성해야 하는 경우 또는 수신된 메시지를 수정하려고 하면 IoTreePushMessageReceiver를 상속하는 리시버를 구현해야한다.

- IoTreePushMessageReceiver를 상속받은 Custom 리시버를 구현한 후 onMessageReceived()를 override한다.
- 포그라운드 알림 노출을 true로 설정했다 하더라도 Custom 리시버를 구현했다면 Custom 리시버가 우선적으로 적용되므로 포그라운드 알림 노출 설정는 무시된다.
- onMessageReceived()에서 수신된 메세지를 확인 및 처리할 수 있다.

  - Custom 리시버로 메시지를 받지만 Notification은 SDK가 생성하는 경우

    ```java
    public class IoTreePushSampleReceiver extends IoTreePushMessageReceiver {
        @Override
        public void onMessageReceived(Context context, IoTreeRemoteMessage remoteMessage) {
            IoTreePushMessage message = remoteMessage.getMessage();

            notify(context, remoteMessage); // 알림을 띄워줌
    }
    ```

  - Notification은 SDK가 생성하도록 하지만 메시지를 수정하려고하는 경우

    ```java
    public class IoTreePushSampleReceiver extends IoTreePushMessageReceiver {
        @Override
        public void onMessageReceived(Context context, IoTreeRemoteMessage remoteMessage) {
            IoTreePushMessage message = remoteMessage.getMessage();

            IoTreePushMessage message = remoteMessage.getMessage();
            message.setTitle("리시버에서 수정함"); // 메세지 타이틀 수정

            notify(context, remoteMessage);
    }
    ```

  - Notification을 직접 생성하는 경우 (수신확인 설정 무시)

    ```java
    public class IoTreePushSampleReceiver extends IoTreePushMessageReceiver {
        @Override
        public void onMessageReceived(Context context, IoTreeRemoteMessage remoteMessage) {
            IoTreePushMessage message = remoteMessage.getMessage();

            Intent intent = new Intent(context, MainActivity.class);

            PendingIntent contentIntent = PendingIntent.getActivity(
                    context,
                    0, // 매번 다른 값이어야 함.
                    intent,
                    PendingIntent.FLAG_UPDATE_CURRENT);

            NotificationCompat.Builder builder = new NotificationCompat.Builder(context, remoteMessage.getChannelId())
                    .setSmallIcon(android.R.drawable.star_big_on)
                    .setContentTitle(message.getTitle())
                    .setContentText(message.getBody())
                    .setAutoCancel(true)
                    .setPriority(NotificationCompat.PRIORITY_HIGH)
                    .setContentIntent(contentIntent);

            notify(context, builder.build());
    }
    ```

  - Notification을 직접 생성하는 경우 (수신확인 설정 동작함)

    ```java
    public class IoTreePushSampleReceiver extends IoTreePushMessageReceiver {
        @Override
        public void onMessageReceived(Context context, IoTreeRemoteMessage remoteMessage) {
            IoTreePushMessage message = remoteMessage.getMessage();

            Intent intent = new Intent(context, MainActivity.class);

            PendingIntent contentIntent = PendingIntent.getActivity(
                    context,
                    100,
                    intent,
                    PendingIntent.FLAG_UPDATE_CURRENT);

            // 알림을 클릭했을 때 수신확인 설정이 동작하려면 getNotificationIntent()를 호출하여 PendingIntent를 생성하고 Notificaion을 생성할 때 지정한다.
            PendingIntent serviceIntent = getNotificationIntent(context, remoteMessage, contentIntent);

            NotificationCompat.Builder builder = new NotificationCompat.Builder(context, remoteMessage.getChannelId())
                    .setSmallIcon(android.R.drawable.star_big_on)
                    .setContentTitle(message.getTitle())
                    .setContentText(message.getBody())
                    .setAutoCancel(true)
                    .setPriority(NotificationCompat.PRIORITY_HIGH)
                    .setContentIntent(serviceIntent);

            notify(context, builder.build());
        }
    }
    ```

- 새로 구현된 Custom 리시버는 반드시 AndroidManifest.xml 파일에 등록해야한다.

  ```xml
  <receiver android:name=".IoTreePushSampleReceiver" android:permission="${applicationId}.iotree.push.permission.RECEIVE">
      <intent-filter>
          <action android:name="kr.co.iotree.push.MESSAGE_EVENT" />
      </intent-filter>
  </receiver>
  ```

### Custom Data(사용자 데이터) 사용법

- 콘솔에서 메세지를 보낼 때, title, body 등 정해진 필드 이외에 custom data를 담아서 보낼 수 있다.
- custom data는 (key : value) 형식으로 알림 클릭 시 intent에 데이터를 담아서 보낸다.
- 알림 클릭 시 이동할 activity에서 아래와 같이 custom data를 꺼내서 사용할 수 있다.

```java
Intent intent = getIntent();
Bundle bundle = intent.getExtras();

if (bundle != null) {
    // 지정된 키가 있다면 바로 bundle에서 꺼내서 사용한다.
    String customData = bundle.getString("customData");
}
```