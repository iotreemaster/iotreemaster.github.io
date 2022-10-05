# Push 발송 요청 API

## 1. 요청

- 요청 API 주소

  - **http(s)://`host`:`port`/message/send**
    - 내부망에서 운영되는 경우에는 https를 구축할 필요가 없지만 외부망으로 분리되어 운영되는 경우에는 https 구성을 이용하는 게 안전하다.
    - 서버의 주소(host, port)의 경우에는 운영환경에 맞는 값을 사용한다.

- 요청 샘플
  - HTTP 프로토콜을 이용하는 POST 요청으로 아래와 같은 형태로 요청한다.

```
POST /message/send HTTP/1.1
...
Accept: application/json, text/plain, */*
Content-Type: application/json;charset=UTF-8
..
app-key: 608efc0f8f9e8aa11caed834203d7e03bba2456824405310bf876ae8268375cc

{
    "target":{
        "userIds":["yjmoon"],
        "topic":null,
        "groups":null
    },
    "message":{
        "title":"제목입니다.",
        "body":"안녕하세요.",
        "image":null,
        "sound":null,
        "clickAction":null,
        "category":null,
        "customData":{},
        "ttl":86400,
        "priorityHigh":true
    }
}
```

- 요청 개요

  - POST 요청의 데이터는 모두 JSON 포맷으로 이루어진다. 샘플은 참고용이므로 값이 null인 필드들이 모두 표시되어 있지만 몇몇 필수 필드를 제외하고 모두 생략 가능하다.
  - 요청의 헤더에는 `app-key`가 반드시 포함되어 있어야 하며 관리자에게 문의하면 받을 수 있다.

- `target`
  `userIds`, `topic`, `group` 중 하나의 필드만 값이 있어야 한다. 둘 이상 지정된 경우 `userIds` > `topic` > `group` 순으로 우선 적용된다.

  - `userIds`: 푸시를 수신할 사용자 ID의 목록이다. 1개의 ID라도 배열 형태로 전달되어야 한다.
    - 예: ["yjmoon", "sampleid"]
  - `topic`: 푸시를 수신할 topic을 지정한다. topic은 하나만 지정할 수 있다.
    - 예: "everyone"
  - `groups`: 푸시를 수신할 group을 지정한다. 여러 그룹을 동시에 지정할 수 있다.
    - topic과 비숫한 기능이지만 topic과 달리 각 그룹원들에게 각각 발송하여 발송 이력이 기록된다.
    - 참고로 지정된 그룹 중에 2개 이상 포함된 사용자가 있더라도 메시지는 하나만 받게된다.
    - 예: ["AndroidGroup", "iOSGroup"]

- `message`
  아래와 같은 필드들을 담을 수 있으며 `title`, `body`를 제외하고 나머지 필드는 모두 선택사항이므로 생략 가능하다.
  - [필수] `title`: 푸시 메시지의 제목을 지정한다. 최대 255글자까지 가능하지만 푸시 메시지 전체 payload의 크기가 4000 byte로 제한되므로 짧고 명료한 제목으로 지정한다.
  - [필수] `body`: 푸시 메시지의 본문 내용을 지정한다. 최대 4000글자까지 가능하지만 푸시 메시지 전체 payload의 크기가 4000 byte로 제한되므로 실제 발송 가능한 길이는 한글 기준으로 1000자 내외이며 제한을 넘어가는 경우 발송 실패 처리된다.
  - `image`: Push 메시지에 이미지를 담아 보내고자 하는 경우 표시할 이미지의 URL을 지정한다.
  - `sound`: Push 메시지 수신 시 재생할 Sound 리소스의 이름을 지정한다. 재생할 Sound 리소스는 푸시를 수신하는 앱에 이미 탑재가 되어 있어야 한다.
  - [안드로이드] `clickAction`: 안드로이드 옵션으로 클릭 시 동작시킬 인텐트의 이름을 지정한다.
  - [iOS] `category`: iOS용 옵션으로 위 clickAction의 iOS 버전이다. 등록되어 있는 `UNNotificationCategory`의 ID값을 지정한다.
  - `customData`: key:value 형태의 데이터를 탑재하기위한 필드로서 key와 value 모두 문자열 데이터 형태로 전송된다. 참고로 Android의 경우 `data` 필드 밑에 key:value 형태로 전달되며 iOS의 경우 payload 최상단에 바로 key:value 형태로 포함된다.
  - `ttl`: 단말이 여러가지 이유로 메시지 수신이 불가능한 상황에 있는 경우 서버가 일정 시간동안 메시지를 보관하고 있다가 수신이 가능한 상황이 되면 전송을 해주는데 이 때 메시지를 서버가 보관하는 시간을 초단위로 설정한다. 기본적으로 24시간(86400초) 동안 유효하다. Firebase에 전달되는 값이다.
  - `priorityHigh`: 즉시 전송되도록 할 것인지 설정한다. true, false 값으로 설정하며 false인 경우 단말이 절전 모드 상태 유지 등의 이유로 수신을 지연시킬 수 있다.

## 2. 응답

- 응답 개요

  - HTTP 응답 코드가 200 OK인 경우 일단 요청 자체는 성공한 것이다.
  - 응답 코드가 200 이외의 값인 경우 모두 실패이며 app-key 헤더가 없거나 기타 이유로 HTTP 요청 자체가 실패할 수 있다.

- 응답 샘플
  200 OK를 받은 경우에도 응답에 포함된 `code` 값을 검사하여 실패 여부를 검사해야 한다. 실패한 경우 `message` 필드를 통해서 적당한 에러메시지를 받을 수 있다.

  - 성공

  ```
  {
    "code": 0,
    "results": [
      {
        "userId": "yjmoon",
        "token": "fCC_ZGspReq33WNq_9LeIj:APA91bGpDJpDcBPXdU0CV_UTGmmTOu8q0kcEg9xrfXWf1LgQyZsgmrrTqx6eTHDe5TYea4cOlHwC1Gqgdjzk5xvphhQ7TFNQ3VTlIPmiNgnzlcLdiueKitvb5Te7KHBIjUTBsXzp_4g2",
        "deviceType": "A",
        "requestUid": 329
      },
      {
        ...
      }
    ]
  }
  ```

  - 실패

  ```
  {
    "code":-2,
    "message":"No token is found."
  }
  ```

- `code`: 0보다 크거나 같은 경우 성공(일반적으로 0). 0 보다 작은 경우 실패.

  | code | 사유                        |
  | ---- | --------------------------- |
  | 0    | 성공                        |
  | -1   | 기타 실패. 메시지 참고.     |
  | -2   | 타겟이 유효하지 않음        |
  | -3   | 요청 데이터가 유효하지 않음 |
  | -4   | DB Access 에러              |

- `message`: 코드값이 실패인 경우 실패 원인에 대한 메시지를 갖는다. 특별한 포맷은 없으며 에러 추적을 위한 시스템 메시지를 포함하는 경우도 있으므로 로그 기록 이외의 용도로는 부적합하다.

- `results`: 전송 요청 결과를 담고 있는 배열. 타겟이 userId, group인 경우에만 값이 존재하고 topic으로 보낸 경우에는 값이 존재하지 않는다. topic으로 전송을한 경우에는 특정 단말이 대상이 아니므로 '수신', '수신 확인' 같은 상태값이 존재하지 않는다. 따라서 requestUid 값을 이용해서 상태조회를 할 필요가 없으므로 topic으로 요청한 경우에는 code값 외에 추가 데이터를 반환하지 않는다. 또한 그룹으로 전송한 경우 그룹에 포함된 토큰 수만큼 결과값이 내려오기 때문에 요청에 비해 상당히 큰 데이터량이 내려온다.
  - `userId`: 푸시를 수신할 사용자 ID.
  - `group`: 푸시를 수신할 그룹 ID.
  - `token`: 위 userId, group에 등록된 단말의 Token값. 참고로 하나의 userId에 여러 Token이 등록될 수 있다.
  - `deviceType`: 발송된 token의 단말 타입. Android: "A", iOS: "I".
  - `requestUid`: 요청이 접수되면 발급되는 요청 ID. 전송 결과/상태를 조회할 때 사용할 수 있다.
