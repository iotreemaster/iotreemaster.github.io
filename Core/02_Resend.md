# Push 재전송 요청 API

## 1. 재전송 요청

- 결과/상태 조회 API 주소

  - **http(s)://`host`:`port`/message/resend**
    - 내부망에서 운영되는 경우에는 https를 구축할 필요가 없지만 외부망으로 분리되어 운영되는 경우에는 https 구성을 이용하는 게 안전하다.
    - 서버의 주소(host, port)의 경우에는 운영환경에 맞는 값을 사용한다.

- 요청 샘플
  - HTTP 프로토콜을 이용하는 POST 요청으로 아래와 같은 형태로 요청한다.

```
POST /message/resend HTTP/1.1
...
Accept: application/json, text/plain, */*
Content-Type: application/json;charset=UTF-8
...
app-key: 608efc0f8f9e8aa11caed834203d7e03bba2456824405310bf876ae8268375cc

{
    "requestUids": [294, 295]
}
```

- 요청 개요

  - POST 요청의 데이터는 모두 JSON 포맷으로 이루어진다.

- `requestUids`
  `/message/send` 요청의 결과로 받은 requestUid를 전달한다. 단 1건만 요청을 하더라도 배열 형태로 전달해야 한다.

## 2. 응답

- 응답 개요

  - HTTP 응답 코드가 200 OK인 경우 일단 요청 자체는 성공한 것이다.
  - 응답 코드가 200 이외의 값인 경우 모두 실패이며 여러가지 이유로 HTTP 요청 자체가 실패할 수 있다.

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

- `results`: 전송 요청 결과를 담고 있는 배열. 타겟이 userId인 경우에만 값이 존재하고 topic으로 보낸 경우에는 값이 존재하지 않는다. topic으로 전송을한 경우에는 특정 단말이 대상이 아니므로 '수신', '수신 확인' 같은 상태값이 존재하지 않는다. 따라서 requestUid 값을 이용해서 상태조회를 할 필요가 없으므로 topic으로 요청한 경우에는 code값 외에 추가 데이터를 반환하지 않는다.
  - `userId`: 푸시를 수신할 사용자 ID.
  - `group`: 푸시를 수신할 그룹 ID.
  - `token`: 위 userId에 등록된 단말의 Token값. 하나의 userId에 여러 Token이 등록될 수 있다.
  - `deviceType`: 발송된 token의 단말 타입. Android: "A", iOS: "I".
  - `requestUid`: 요청이 접수되면 발급되는 요청 ID. 전송 결과/상태를 조회할 때 사용할 수 있다.
