# Push 예약 취소 요청 API

## 1. 요청

- 예약 취소 API 주소

  - **http(s)://`host`:`port`/message/cancelReserve**
    - 내부망에서 운영되는 경우에는 https를 구축할 필요가 없지만 외부망으로 분리되어 운영되는 경우에는 https 구성을 이용하는 게 안전하다.
    - 서버의 주소(host, port)의 경우에는 운영환경에 맞는 값을 사용한다.

- 요청 샘플
  - HTTP 프로토콜을 이용하는 POST 요청으로 아래와 같은 형태로 요청한다.

```
POST /message/cancelReserve HTTP/1.1
...
Accept: application/json, text/plain, */*
Content-Type: application/json;charset=UTF-8
...
app-key: 608efc0f8f9e8aa11caed834203d7e03bba2456824405310bf876ae8268375cc

{
    "reservationUids":[24, 26]
}
```

- 요청 개요

  - POST 요청의 데이터는 모두 JSON 포맷으로 이루어진다.
  - 요청의 헤더에는 `app-key`가 반드시 포함되어 있어야 하며 관리자에게 문의하면 받을 수 있다.
  - 요청된 uid 값이 (이미 처리된 경우와 같이) 실제로 DB에 없는 값이라도 문제없이 성공으로 처리된다.

- `reservationUids`
  취소하고자 하는 예약 요청의 uid 값. 예약을 요청할 때 응답으로 받을 수 있는 값이다. 예약이 이미 처리되어 종료된 요청을 취소하려고 하더라도 문제 없이 성공으로 처리된다.

## 2. 응답

- 응답 개요

  - HTTP 응답 코드가 200 OK인 경우 일단 요청 자체는 성공한 것이다.
  - 응답 코드가 200 이외의 값인 경우 모두 실패이며 app-key 헤더가 없거나 기타 이유로 HTTP 요청 자체가 실패할 수 있다.

- 응답 샘플
  200 OK를 받은 경우에도 응답에 포함된 `code` 값을 검사하여 실패 여부를 검사해야 한다. 실패한 경우 `message` 필드를 통해서 적당한 에러메시지를 받을 수 있다.

  - 성공

  ```
  {
    "code":0,
    "results": [
      {
        "userId": "yjmoon",
        "topic": null,
        "groups": null,
        "reservationUid": 24
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
    "code":-1,
    "message":"Invalid timezone offset."
  }
  ```

- `code`: 0보다 크거나 같은 경우 성공(일반적으로 0). 0 보다 작은 경우 실패.

| code | 사유                    |
| ---- | ----------------------- |
| 0    | 성공                    |
| -1   | 기타 실패. 메시지 참고. |
| -2   | 토큰이 존재하지 않음.   |
| -3   | DB Access 에러          |

- `message`: 코드값이 실패인 경우 실패 원인에 대한 메시지를 갖는다. 특별한 포맷은 없으며 에러 추적을 위한 시스템 메시지를 포함하는 경우도 있으므로 로그 기록 이외의 용도로는 부적합하다.

- `results`: 예약 요청 결과를 담고 있는 리스트(배열). 여러 사용자에게 예약을 요청한 경우 각 사용자별로 예약이 되기 때문에 각 사용자별 결과값을 리스트로 내려준다. 따라서 각 사용자별 예약 요청의 `reservationUid`를 사용하여 개별적으로 예약을 취소할 수 있다.

  - `userId`: 푸시를 수신할 사용자 ID.
  - `topic`: 푸시를 수신할 Topic.
  - `groups`: 푸시를 수신할 그룹들의 리스트.
  - `reservationUid`: 요청이 접수되면 발급되는 요청 ID. 전송 결과/상태를 조회할 때 사용할 수 있다.
