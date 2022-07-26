# REST\_API

## 사용자 확인하기&#x20;

### 기본 정보&#x20;

사용자가 게임 서비스에 가입되어 있는지 확인합니다.

엑세스 토큰을 헤더에 담아 `GET`으로 요청합니다. 현재 로그인한 사용자가 게임 서비스에 가입되어 있는지 묻는 API이므로 파라미터는 필요하지 않습니다.

요청은 JSON 형식과 키 `isGameUser`로 구성되어 있습니다. 해당 키 값이 `true`라면 게임 서비스 이용을 허가합니다. 만약 `false`일 때는 다른 서비스 API를 호출하지 않도록 예외처리 합니다.

서버는 카카오의 서버(kapi.kakao.com)를 사용합니다.

### Request

#### URL

```
GET /v1/api/minesweeper/users/isgameuser HTTP/1.1
Host: kapi.kakao.com
Authorization: Bearer ${ACCESS_TOKEN}
```

### Response

| 이름         | 유형      | 설명                                  | 필수 여부 |
| ---------- | ------- | ----------------------------------- | ----- |
| isGameUser | Boolean | <p>게임 사용자 여부 </p><p>true이면 사용자 </p> | Yes   |

### Sample

#### Request

```
curl -v -X GET "https://kapi.kakao.com/v1/api/minesweeper/users/isgameuser" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

#### Response

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=UTF-8
{
    "isGameUser": true
}
```

## 프로필 가져오기&#x20;

### 기본 정보&#x20;

사용자의 프로필 정보를 요청합니다. 프로필 정보로는 닉네임, 프로필 사진 등이 있습니다. 액세스 토큰을 헤더에 담아 GET으로 요청하며, 이를 JSON 형식으로 응답받습니다.&#x20;

### Request

#### URL

```
GET /v1/api/minesweeper/profile HTTP/1.1
Host: kapi.kakao.com
Authorization: Bearer ${ACCESS_TOKEN}
```

### Response

| 이름                  | 유형     | 설명               | 필수 여부 |
| ------------------- | ------ | ---------------- | ----- |
| nickname            | String | 게임 닉네임           | Yes   |
| profileInmageUrl    | String | 프로필 이미지 url      | No    |
| profileThumbnailUrl | String | 프로필 썸네일 이미지 url  | No    |

### Sample

#### Request

```
curl -v -X GET "https://kapi.kakao.com/v1/api/minesweeper/profile" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}"
```

#### Response: 성공, 프로필 사진이 있을 경우&#x20;

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=UTF-8
{
    "nickName": "김연우",
    "profileImageUrl": "http://xxx.kakao.com/.../aaa.jpeg",
    "thumbnailUrl": "http://xxx.kakao.com/.../bbb.jpeg"
}
```

#### Response: 성공, 프로필 사진이 없을 경우&#x20;

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=UTF-8
{
    "nickName": "김연우"
}
```

#### Response: 실패, 해당 사용자가 게임 서비스에 가입하지 않은 경우&#x20;

```
HTTP/1.1 400 Bad Request
Content-Type: application/json; charset=UTF-8
{
    "msg": "NotExistGameUserException",
    "code": -101
}
```

## 게임하기&#x20;

### 기본 정보&#x20;

현재 로그인한 사용자가 게임을 실행합니다. 지뢰찾기 게임은 플레이 시간이 기록되며, 지뢰를 선택했을 때 게임이 종료됩니다. 시간 기록과 게임의 성공 여부는 서버에 기록됩니다.&#x20;

성공 시 HTTP 상태 코드 200에 해당 승전 기록 ID가 반환됩니다.&#x20;

### Request

#### URL: 성공 여부 기록&#x20;

```
POST /v1/api/minesweeper/log HTTP/1.1
Host: kapi.kakao.com
Authorization: Bearer ${ACCESS_TOKEN}
Content-Type: application/x-form-urlencoded
```

#### Parameter

| 이름        | 유형      | 설명                                          | 필수 여부 |
| --------- | ------- | ------------------------------------------- | ----- |
| isSuccess | String  | <p>게임 승리 여부 </p><p>승리: W, 패배: L, 중지: S </p> | Yes   |
| gameTime  | Integer | 게임 수행 시간 (초)                                | Yes   |

### Response

| 이름 | 유형     | 설명           | 필수 여부 |
| -- | ------ | ------------ | ----- |
| id | String | 승전 기록 고유 번호  | Yes   |

### Sample

#### Request

```
curl -v -X POST "https://kapi.kakao.com/v1/api/minesweeper/log" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    --data-urlencoded "isSuccess = W" \
    --data-urlencoded "gameTime = 630"
```

#### Response

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=UTF-8
{
    "id": "AAAAA.BBBBBB"
}
```

## 내 게임이력 조회하기&#x20;

### 기본 정보&#x20;

현재 로그인한 사용자의 게임 이력을 불러옵니다. 최근 기록을 기본적로 요청하며, 특정 ID로 요청하면 해당 내역을 불러옵니다.

게임 이력을 조회할 때 특정 내역을 요청하는지, 여러 개의 최근 내역을 요청하는지에 따라 전달해야 할 파라미터가 다릅니다. 특정 내역을 요청할 때는 게임 이력 ID는 필수적으로 전달해야 합니다. 여러 개의 최근 내역을 요청할 때는 파라미터가 필요하지 않으며, 이전 검색 내역을 받을 수 있는 lastId 파라미터를 사용할 수 있습니다.

### Request

#### URL: 특정 내역 요청&#x20;

```
GET /v1/api/minesweeper/mylog HTTP/1.1
Host: kapi.kakao.com
Authorization: Bearer ${ACCESS_TOKEN}
```

#### URL: 최근 내역 요청&#x20;

```
GET /v1/api/minesweeper/mylogs HTTP/1.1
Host: kapi.kakao.com
Authorization: Bearer ${ACCESS_TOKEN}
```

#### Parameter

특정 내역 요청&#x20;

| 이름 | 유형     | 설명           | 필수 여부 |
| -- | ------ | ------------ | ----- |
| id | String | 승전 기록 고유 번호  | Yes   |

최근 내역 요청&#x20;

| 이름     | 유형     | 설명                                                                    | 필수 여부 |
| ------ | ------ | --------------------------------------------------------------------- | ----- |
| lastId | String | <p>마지막 검색 기록 해당 이력을 제외한, 그 이전에 기록된 정보를 수집함.</p><p>기본값은 가장 최근 이력임.</p> | No    |

### Response

| 이름   | 유형       | 설명           | 필수 여부 |
| ---- | -------- | ------------ | ----- |
| id   | String   | 승전 기록 ID     | Yes   |
| date | Datetime | 게임 수행 날짜     | Yes   |
| time | Integer  | 게임 수행 시간 (초) | Yes   |

### Sample

#### Request: 특정 내역 조회&#x20;

```
curl -v -G GET "https://kapi.kakao.com/v1/api/minesweeper/mylog" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    -d "id = ${LOG_ID}"
```

#### Request: 최근 내역 여러 개 조회&#x20;

```
url -v -G GET "https://kapi.kakao.com/v1/api/minesweeper/mylogS" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    -d "id = ${LOGS_ID}"
```

#### Response: 특정 내역 조회&#x20;

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=UTF-8
{
    "id": "AAAAA.BBBBBB",
    "date": "2002-07-26T14:12:20Z",
    "time": 630
}
```

#### Response: 최근 내역 여러 개 조회&#x20;

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=UTF-8
[
    {
        "id": "AAAAA.BBBBBB",
        "date": "2002-07-26T14:12:20Z",
        "time": 630
    },
    {
        "id": "AAAAA.CCCCCC",
        "date": "2002-07-24T08:14:36Z",
        "time": 471
    },
    ...
]
```
