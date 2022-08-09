## 프로필 가져오기
> 현재 로그인한 사용자의 프로필 정보(닉네임, 프로필 이미지)을 불러옵니다. 사용자가 SCSC 홈페이지 회원이 아니라면 오류가 발생합니다.

------------
### 요청
> 요청자가 SCSC 회원인지 확인합니다.

#### URL
* GET /users/{id}

#### URL 파라미터
* id: 회원 아이디 [string]

#### 데이터 파라미터
* nickname: 닉네임 정보 [string]
* name: 사용자의 실명 [string]
* profileImageUrl: 프로필 이미지 URL [string]
* thumbnailImageUrl: 프로필 썸네일 이미지 URL [string]
* userState: 회원 등급(동아리원, 졸업생, 일반회원) [string]
* email: 사용자의 이메일 주소 [string]

------------
### 성공 시 응답

#### 응답 코드 및 내용
* 응답 코드: 200(OK)
* 내용
``` Javascript
{
  nickname: [string],
  name: [string],
  profileImageUrl: [string],
  thumbnailImageUrl: [string],
  userState: [string],
  email: [string]
}
```

#### 응답 예시
``` Javascript
{
  "nickname": 'GilDong',
  "name": '홍길동',
  "profileImageUrl": 'https://server_domain/.../aaa.jpeg',
  "thumbnailImageUrl": 'https://server_domain/.../bbb.jpeg',
  "userState": '동아리원',
  "email": 'gildonghong@snu.ac.kr'
}
```

------------
### 실패 시 응답
> SCSC에 회원가입하지 않았거나, 로그인하지 않은 회원이 접근할 때 오류가 발생합니다.

#### 응답 코드 및 내용
* 응답 코드: 401(Unauthorized)
* 내용
``` Javascript
{
  error: {
    id: ERROR_UNAUTHORIZED,
    message:[string]
  }
}
```

#### 응답 예시
``` Javascript
{
  error: {
    id: ERROR_UNAUTHORIZED,
    message:'접근 권한이 없습니다.'
  }
}
```
------------
