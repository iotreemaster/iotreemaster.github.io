
# Android SDK 사용 가이드

## 라이브러리 설정

---

1. Firebase 콘솔에 접속하여 Firebase 프로젝트를 생성한다.
2. 푸시 알림을 받을 안드로이드 앱 프로젝트를 로컬에 생성한다.
3. 1번에서 생성한 Firebase 프로젝트에 2번에서 생성한 안드로이드 앱을 등록한다.

    * 콘솔 메인 화면이나 프로젝트 설정에서 앱 등록을 클릭한다.
    * 안드로이드 패키지 이름(필수), 앱 닉네임, SHA-1(디버그 서명 인증서)을 입력한다.
    * google-services.json 파일을 다운로드 하고 안드로이드 앱 프로젝트의 app 폴더에 저장한다.
    * build.gradle(project)에 google service plugin을 추가한다.(버전은 최신 버전으로 적용)

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

    * build.gradle(app)에 google service gradle plugin을 추가한다.
    * dependencies에는 firebase SDK를 추가한다.(버전은 최신 버전으로 적용)

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

    * 자세한 사항은 https://firebase.google.com/docs/cloud-messaging/android/client?hl=ko 참고!

4. aar 파일을 다운로드 한다.
    * https://github.com/iotreemaster/iotreemaster.github.io/raw/main/iotreePush.aar
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

---

### SDK 초기화
   * 푸시 알림을 받기 위해서 가장 먼저 필수적으로 해야하는 과정으로 push SDK를 초기화한다.
   * IoTreePush.initialize()를 호출하여 초기화하고 context와 appKey를 매개변수로 넘겨준다.
     * context: Android Context.
     * appKey: ioTree Push 서버에서 발급하는 고유한 키 값(String).

     ```
     IoTreePush.initialize(context, appKey);
     ```

### 토큰 발급
   * Push 수신을 위한 토큰을 발급 받아서 등록한다.
   * 비동기 방식으로 동작하며 완료 시 전달된 callback의 메소드를 호출해준다.
     * context: Android Context.
     * userId: 서비스에 따른 사용자 식별자 값. (서비스 구현에 따라 어떤 값이든 될 수 있다. 예를 들어 서비스 Login ID 같은 값을 사용할 수 있다.)
     * callback: 완료 시 호출되는 callback 객체. 

     ```
     interface RegisterTokenCallback {
         void onRegister(IoTreePushResult result, String token);
     }

     IoTreePush.registerToken(context, userId, new RegisterTokenCallback() {
         @Override
         public void onRegister(IoTreePushResult result, String token) {
             if (!result.isSuccess()) {
                 int code = result.getCode();
             }

             Log.d("Push", token);
        }
     });
     ```

### 토큰 해제
   * push 서버에 등록된 토큰을 해제한다.
   * 비동기 방식으로 동작하며 완료 시 전달된 callback의 메소드를 호출해준다.
     * callback: 완료 시 호출되는 callback 객체. 

     ```
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

### 토픽 구독
   * 원하는 토픽을 선택하여 구독 요청을 한다. 토픽으로 전송된 모든 메시지를 수신할 수 있다.
     * topic: 토픽
     * callback:완료 시 호출되는 callback 객체.

     ```
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

### 토픽 구독 해제
   * 지정된 토픽 구독을 해제한다. 토픽 구독을 해제하면 토픽으로 메세지를 발송해도 수신이 되지 않는다.
     * topic: 토픽
     * callback:완료 시 호출되는 callback 객체.

     ```
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

### 알림 기본 옵션 설정
   * Notification과 NotificationChannel에 대한 기본 옵션을 설정한다.
   * 필수는 아니며 설정하지 않으면 기본 설정이 사용된다. 부분적으로 필요한 옵션만 설정 가능하다.
   * 기본으로 제공하는 NotificationChannel의 이름과 설명을 설정한다.
   * Notification의 아이콘, 색상, 알림음을 설정한다.
   * 알림음의 경우, res/raw에 저장된 로컬 리소스만 사용가능 하다.
   * 기본적으로 앱이 포그라운드 상태이면 Notification을 띄우지 않지만 SDK로 하여금 Notification을 띄우도록 설정하려면 enableForeground(true)로 설정한다. (default: false)
   * Runtime에 코드로 설정하거나 AndroidManifest.xml 파일에 메타 데이터로 정의할 수 있다. 만약 둘 다 설정된 경우 코드로 설정된 값이 우선 적용된다.

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
     또는

     ```
     <meta-data android:name="kr.co.iotree.push.notification.default_notification_icon"
                 android:resource="@drawable/default_notification_icon" />
     <meta-data android:name="kr.co.iotree.push.notification.default_notification_color"
                 android:resource="@color/default_notification_color" />
     <meta-data android:name="kr.co.iotree.push.notification.default_notification_sound"
                 android:value="noti" />
     <meta-data
     android:name="kr.co.iotree.push.notification.foreground_Enabled"
                 android:value="false" />
     ```

### Custom Receiver 구현

직접 알림을 생성해야 하는 경우 또는 수신된 메시지를 수정하려고 하면 IoTreePushMessageReceiver를 상속하는 리시버를 구현해야한다.

   * IoTreePushMessageReceiver를 상속받은 Custom 리시버를 구현한 후 onMessageReceived()를 override한다. 
   * 포그라운드 알림 노출을 true로 설정했다 하더라도 Custom 리시버를 구현했다면 Custom 리시버가 우선적으로 적용되므로 포그라운드 알림 노출 설정는 무시된다.
   * onMessageReceived()에서 수신된 메세지를 확인 및 처리할 수 있다.

     * Custom 리시버로 메시지를 받지만 Notification은 SDK가 생성하는 경우

       ```
       public class IoTreePushSampleReceiver extends IoTreePushMessageReceiver {
           @Override
           public void onMessageReceived(Context context, IoTreeRemoteMessage remoteMessage) {
               IoTreePushMessage message = remoteMessage.getMessage();
       
               notify(context, remoteMessage); // 알림을 띄워줌
       }
       ```
     * Notification은 SDK가 생성하도록 하지만 메시지를 수정하려고하는 경우

       ```
       public class IoTreePushSampleReceiver extends IoTreePushMessageReceiver {
           @Override
           public void onMessageReceived(Context context, IoTreeRemoteMessage remoteMessage) {
               IoTreePushMessage message = remoteMessage.getMessage();
       
               IoTreePushMessage message = remoteMessage.getMessage();
               message.setTitle("리시버에서 수정함"); // 메세지 타이틀 수정
       
               notify(context, remoteMessage);
       }
       ```

     * Notification을 직접 생성하는 경우 (수신확인 설정 무시)

       ```
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

     * Notification을 직접 생성하는 경우 (수신확인 설정 동작함)

       ```
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

   * 새로 구현된 Custom 리시버는 반드시 AndroidManifest.xml 파일에 등록해야한다.

     ```
     <receiver android:name=".IoTreePushSampleReceiver" android:permission="${applicationId}.iotree.push.permission.RECEIVE">
         <intent-filter>
             <action android:name="kr.co.iotree.push.MESSAGE_EVENT" />
         </intent-filter>
     </receiver> 
     ```
