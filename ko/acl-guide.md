## Storage > Object Storage > 접근 정책 설정 가이드

콘솔 또는 API를 사용해 다른 사용자에게 컨테이너의 읽기/쓰기 접근 권한을 부여할 수 있습니다.

## 콘솔
콘솔에서는 [컨테이너 생성](/Storage/Object%20Storage/ko/console-guide/#_2) 또는 [컨테이너 설정](/Storage/Object%20Storage/ko/console-guide/#_5) 창에서 컨테이너 접근 정책을 선택할 수 있습니다. 선택할 수 있는 정책은 `PRIVATE`과 `PUBLIC` 두 가지로 제한됩니다.

### PRIVATE
`PRIVATE`은 컨테이너가 속한 프로젝트의 사용자에게만 접근 권한을 부여하는 기본 접근 정책입니다. 콘솔을 이용하거나, 인증 토큰을 발급받아 API로 컨테이너에 접근할 수 있습니다. API 섹션의 [컨테이너가 속한 프로젝트의 사용자에게만 읽기/쓰기 허용](/Storage/Object%20Storage/ko/acl-guide/#_2) 항목과 같은 정책입니다.
<br/>

### PUBLIC
`PUBLIC`은 누구에게나 읽기와 오브젝트 목록 조회를 허용하는 정책입니다. 컨테이너를 PUBLIC으로 설정하면 콘솔에서 URL을 얻을 수 있습니다. 이 URL을 통해 누구나 컨테이너에 접근할 수 있습니다. API 섹션의 [모든 사용자에게 읽기 허용](/Storage/Object%20Storage/ko/acl-guide/#_2) 항목과 같은 정책입니다.
<br/>

## API
API를 사용해 컨테이너의 `X-Container-Read`, `X-Container-Write` 속성에 ACL 정책 요소를 입력하면 여러 가지 상황에 맞게 접근 정책을 설정할 수 있습니다.
<br/>

### ACL 정책 요소

설정할 수 있는 ACL 정책 요소는 다음과 같습니다. 모든 정책 요소는 콤마(`,`)로 구분해 조합할 수 있습니다.

| 정책 요소 | 설명 |
| --- | --- |
| `.r:*` | 누구나 인증 토큰 없이 오브젝트에 접근할 수 있습니다. |
| `.rlistings` | 읽기 권한이 있는 사용자에게 컨테이너 조회(GET 또는 HEAD 요청)를 허용합니다.<br/>이 정책 요소가 없으면 오브젝트 목록을 조회할 수 없습니다.<br/>이 정책 요소는 단독으로 설정할 수 없습니다. |
| `.r:<referrer>` | 요청 헤더를 참조하여 설정된 HTTP 리퍼러(HTTP Referer)에게 접근을 허용합니다.<br/>인증 토큰은 필요하지 않습니다. |
| `.r:-<referrer>` | 요청 헤더를 참조하여 설정된 HTTP 리퍼러의 접근을 제한합니다.<br/>리퍼러 앞에 마이너스 기호(-)를 붙여 설정합니다. |
| `<tenant-id>:<user-uuid>` | 특정 프로젝트에 속한 특정 사용자에게 발급된 인증 토큰으로 오브젝트에 접근할 수 있습니다.<br/>읽기, 쓰기 권한을 모두 부여할 수 있습니다. |
| `<tenant-id>:*` | 특정 프로젝트에 속한 모든 사용자에게 발급된 인증 토큰으로 오브젝트에 접근할 수 있습니다.<br/>읽기, 쓰기 권한을 모두 부여할 수 있습니다. |
| `*:<user-uuid>` | 프로젝트와 관계없이 특정 사용자에게 발급된 인증 토큰으로 오브젝트에 접근할 수 있습니다.<br/>읽기, 쓰기 권한을 모두 부여할 수 있습니다. |
| `*:*` | 프로젝트와 관계없이 인증 토큰을 발급받을 수 있는 사용자라면 누구나 오브젝트에 접근할 수 있습니다.<br/>읽기, 쓰기 권한을 모두 부여할 수 있습니다. |

> [참고]
> 사용자 UUID는 NHN Cloud 사용자 ID가 아닙니다. 인증 토큰 발급 요청의 응답 본문에 포함되어 있습니다. (access.user.id)
> API 가이드의 [인증 토큰 발급](/Storage/Object%20Storage/ko/api-guide/#_2) 항목을 참조하세요.

<br/>

### 컨테이너가 속한 프로젝트의 사용자에게만 읽기/쓰기 허용
ACL 정책 요소를 설정하지 않았을 때 사용되는 기본 접근 정책입니다. API를 사용해 컨테이너에 접근하려면 반드시 유효한 인증 토큰이 필요합니다.
컨테이너의 `X-Container-Read`, `X-Container-Write` 속성값을 모두 삭제하면 컨테이너가 속한 프로젝트의 사용자에게만 접근을 허용하는 [PRIVATE](/Storage/Object%20Storage/ko/acl-guide/#private) 컨테이너가 됩니다.

<br/>

<details>
<summary>모든 ACL 정책 요소 제거 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read;' \
  -H 'X-Container-Write;' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

<blockquote>
<p>[참고]
curl을 이용해 값이 없는 헤더를 보낼 때는 헤더 이름에 세미콜론(;)을 붙여야 합니다.</p>
</blockquote>

유효한 인증 토큰 없이 요청하면 에러 메시지를 응답합니다.

```
$ curl -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

반드시 요청 헤더에 유효한 인증 토큰이 있어야 원하는 응답을 받을 수 있습니다.

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container

[컨테이너의 오브젝트 목록]
```
</details>
<br/>

### 모든 사용자에게 읽기/목록 조회 허용
컨테이너의 `X-Container-Read` 속성을 `.r:*, .rlistings`로 설정하면 모든 사용자에게 오브젝트 읽기와 목록 조회를 허용합니다. 인증 토큰은 필요하지 않습니다. 콘솔 섹션의 [PUBLIC](/Storage/Object%20Storage/ko/acl-guide/#public) 항목과 같은 정책입니다.
<br/>

<details>
<summary>전체 사용자에 대해 오브젝트 읽기 및 목록 조회 허용 설정 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .rlistings' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container

[컨테이너의 오브젝트 목록]
```

<code>.r:*</code>만 설정하면 컨테이너의 오브젝트에는 접근할 수 있지만, 오브젝트 목록은 조회할 수 없습니다.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br/>


### 특정 HTTP 리퍼러 요청에 읽기 허용/거부
HTTP 리퍼러(HTTP Referer)는 하이퍼링크를 통해 요청하는 웹 페이지의 주소 정보입니다. 요청 헤더에 포함되어 있습니다.
컨테이너의 `X-Container-Read` 속성에 `.r:<referrer>` 또는 `.r:-<referrer>` 형태의 ACL 정책 요소를 설정하면, 특정 리퍼러의 접근 요청을 허용하거나 차단할 수 있습니다. ACL 정책 요소로 HTTP 리퍼러를 설정할 때는 프로토콜과 하위 경로를 제외한 도메인 이름을 입력해야 합니다.

> [주의]
> HTTP 리퍼러는 헤더 변조를 통해 사용자가 언제든지 변경할 수 있습니다. HTTP 리퍼러를 이용한 접근 정책은 보안에 취약하기 때문에 권장하지 않습니다.

<details>
<summary>특정 HTTP 리퍼러 읽기 요청 허용 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

API 요청 헤더에 허용된 HTTP 리퍼러 주소를 명시해 요청하면 오브젝트에 접근할 수 있습니다.

```
$ curl -O -X GET \
  -H 'Referer: https://cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -O -X GET \
  -H 'Referer: https://cloud.nhn.com/some/path' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]
```

API 요청 헤더에 허가된 리퍼러 주소가 없거나, 리퍼러 주소에 프로토콜이 포함되어 있지 않으면 접근이 차단됩니다.

```
$ curl -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>


$ curl -X GET \
  -H 'Referer: https://example.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>


$ curl -X GET \
  -H 'Referer: cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

다음과 같이 HTTP 리퍼러 설정에 <code>.</code>으로 시작하는 도메인 이름을 입력하면, 설정된 도메인의 모든 서브 도메인 주소를 포함하는 리퍼러에 읽기를 허용합니다.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -O -X GET \
  -H 'Referer: https://guide.docs.nhn.com/some/path' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]
```

서브 도메인이 포함되어 있지 않은 요청은 차단됩니다.

```
$ curl -X GET \
  -H 'Referer: https://nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

특정 도메인 이름을 가진 모든 리퍼러의 접근 요청을 허용하려면 다음과 같이 콤마 리스트를 이용해 설정합니다.

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:nhn.com, .r:.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  -H 'Referer: https://nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -O -X GET \
  -H 'Referer: https://container.nhn.com/some/path' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]
```
</details>

<details>
<summary>특정 HTTP 리퍼러의 읽기 요청 차단 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

HTTP 리퍼러 도메인 이름 앞에 마이너스 기호를 붙여 설정하면, 설정된 HTTP 리퍼러 요청이 차단됩니다.

```
$ curl -X GET -H 'Referer: https://cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```

</details>
<br/>

HTTP 리퍼러에 대한 접근 허용/차단 정책은 입력하는 순서에 따라 적용됩니다. 예를 들어, 리퍼러 차단 정책 요소 뒤에 모두에게 접근을 허용하는 `.r:*` 정책 요소를 입력했다면 리퍼러 차단 정책은 무시됩니다. 반대로 모두에게 접근을 허용하는 정책 요소를 먼저 입력하고 특정 리퍼러 차단 정책 요소를 뒤에 입력했다면, 설정된 리퍼러의 접근 요청을 제외한 모든 접근 요청이 허용됩니다.
<br/>

<details>
<summary>HTTP 리퍼러 차단이 무시되는 잘못된 정책 설정 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:-cloud.nhn.com, .r:*' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -O -X GET -H 'Referer: https://cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]
```
</details>

<details>
<summary>특정 HTTP 리퍼러의 접근 요청을 제외한 모든 접근 요청을 허용하는 정책 설정 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: .r:*, .r:-cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

```
$ curl -O -X GET \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]


$ curl -X GET -H 'Referer: https://cloud.nhn.com' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

<html><h1>Unauthorized</h1><p>This server could not verify that you are authorized to access the document you requested.</p></html>
```
</details>
<br/>

### 특정 프로젝트 또는 특정 사용자에게 읽기/쓰기 허용
컨테이너의 `X-Container-Read`와 `X-Container-Write` 속성에 `<tenant-id>:<user-uuid>` 형태의 ACL 정책 요소를 설정하면, 특정 프로젝트 또는 특정 사용자에게 읽기/쓰기 권한을 각각 부여할 수 있습니다. 테넌트 ID 또는 사용자 UUID 대신 와일드카드 문자 `*`를 입력하면 모든 프로젝트 또는 모든 사용자에게 접근 권한을 부여합니다. 접근 요청을 할 때는 반드시 유효한 인증 토큰이 필요합니다.

> [참고]
> 인증 토큰이 필요한 ACL 정책으로 부여된 읽기 권한에는 오브젝트 목록 조회 권한이 포함되어 있습니다.

<details>
<summary>특정 프로젝트의 특정 사용자에게 읽기/쓰기 권한 부여 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:{user-uuid}' \
  -H 'X-Container-Write: {tenant-id}:{user-uuid}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

오브젝트에 접근 요청을 할 때는 반드시 허가된 테넌트 ID와 NHN Cloud 사용자 ID로 발급받은 유효한 인증 토큰이 필요합니다.

```
$ curl -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container

[컨테이너의 오브젝트 목록]


$ curl -O -X GET \
  -H 'X-Auth-Token: ${token-id}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container/object

[오브젝트 다운로드]
```
</details>

<details>
<summary>특정 프로젝트의 모든 사용자에게 읽기/쓰기 권한 부여 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: {tenant-id}:*' \
  -H 'X-Container-Write: {tenant-id}:*' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

오브젝트에 접근 요청을 할 때는 반드시 허가된 테넌트 ID와 해당하는 프로젝트에 속한 NHN Cloud 사용자 ID로 발급받은 유효한 인증 토큰이 필요합니다.
<br/><br/>
</details>

<details>
<summary>프로젝트와 관계없이 특정 사용자에게 읽기/쓰기 권한 부여 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:{user-uuid}' \
  -H 'X-Container-Write: *:{user-uuid}' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

오브젝트에 접근 요청을 할 때는 반드시 허가된 NHN Cloud 사용자 ID로 발급받은 유효한 인증 토큰이 필요합니다.
<br/><br/>
</details>

<details>
<summary>모든 NHN Cloud 사용자에게 읽기/쓰기 권한 부여 예시</summary>

```
$ curl -i -X POST \
  -H 'X-Auth-Token: ${token-id}' \
  -H 'X-Container-Read: *:*' \
  -H 'X-Container-Write: *:*' \
  https://api-storage.cloud.toast.com/v1/AUTH_*****/container
```

오브젝트에 접근 요청을 할 때는 반드시 유효한 인증 토큰이 필요합니다.
</details>
<br/>

### 접근 정책 삭제
빈 헤더를 입력하면 설정된 ACL 정책 요소를 모두 삭제할 수 있습니다. ACL 정책 요소가 없는 컨테이너는 허가된 사용자만 접근할 수 있는 **PRIVATE** 컨테이너가 됩니다. [컨테이너가 속한 프로젝트의 사용자에게만 읽기/쓰기 허용](/Storage/Object%20Storage/ko/acl-guide/#_2) 항목을 참고하세요.


## References
Swift Access Control Lists (ACLs) - [https://docs.openstack.org/swift/latest/overview_acl.html](https://docs.openstack.org/swift/latest/overview_acl.html)
