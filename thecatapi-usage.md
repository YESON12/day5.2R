# The Cat API 사용법

2026-11-09에 공식 문서와 OpenAPI 명세(v1.7.1)를 읽고 정리했습니다. "직접 확인"이라고 적은 내용은 같은 날 API 키 없이 호출해 본 결과입니다.

## 기본 정보

- **Base URL:** `https://api.thecatapi.com/v1`
- **인증:** 요청 헤더에 `x-api-key: YOUR-API-KEY`
- **키 발급:** https://thecatapi.com/signup 에서 가입하면 이메일로 받습니다.

```bash
curl -H "x-api-key: YOUR-API-KEY" https://api.thecatapi.com/v1/breeds/abys
```

### 요금제

| 요금제 | 월 요금 | 월 요청 수 |
|---|---|---|
| Free | $0 | 10,000 |
| Startup | $99 | 100,000 |
| Enterprise | 문의 | 무제한 |

공식 페이지에는 초당, 분당 요청 제한 수치가 나와 있지 않습니다.

### API 키 없이 쓸 수 있는 범위 (직접 확인)

| 요청 | 결과 |
|---|---|
| `GET /images/search` | 200, 기본 1장 |
| `GET /images/search?limit=10` | 200, 10장 |
| `GET /images/search?limit=100` | 200, 10장만 반환 (키 없이는 10장이 상한으로 보임) |
| `GET /images/{id}` | 200 |
| `GET /breeds/abys`, `GET /facts/random` | 403 |
| `GET /favourites`, `GET /votes` | 401 |

키 없이 `breed_ids`나 `mime_types` 같은 필터를 보내도 200이 오지만, 필터가 실제로 적용되는지는 확인하지 못했습니다. 명세에는 `mime_types`가 키 유무와 상관없이 동작한다고 적혀 있습니다.

## 이미지

### 검색: `GET /images/search`

| 파라미터 | 설명 |
|---|---|
| `limit` | 결과 수. 기본 1, 키가 있으면 1~100 |
| `page` | 0부터 시작하는 페이지 번호 |
| `order` | `ASC`, `DESC`, 무작위 (아래 "문서끼리 다른 부분" 참고) |
| `breed_ids` | 쉼표로 구분한 품종 ID 필터 (예: `abys,beng`) |
| `has_breeds` | 품종 정보가 있는 이미지만 |
| `include_breeds` | 응답에 품종 객체 포함 |
| `include_categories` | 응답에 카테고리 객체 포함 |
| `mime_types` | `jpg`, `png`, `gif`를 쉼표로 구분 |
| `sub_id` | 이 `sub_id`로 업로드한 이미지만 |

`size`도 받지만 명세에 따르면 호환성을 위한 값이라, 어떤 값을 보내도 같은 URL이 옵니다.

```json
[{ "id": "MTgzNTE4NA", "url": "https://cdn2.thecatapi.com/images/MTgzNTE4NA.jpg", "width": 2048, "height": 1517 }]
```

### 그 외 이미지 기능

| 요청 | 설명 | 키 |
|---|---|---|
| `GET /images/{id}` | 이미지 1장 조회 (`created_at`, `breeds`, `categories`, `colours`, `tags` 포함) | 불필요 |
| `POST /images/upload` | 업로드 (multipart 필드 `file`) | 필요 |
| `POST https://upload.thecatapi.com/v1/images/upload-sync` | 분석이 끝날 때까지 기다리는 업로드 (전용 호스트) | 필요 |
| `GET /images/{id}/status` | 업로드 처리 상태 | 필요 |
| `GET /images/{id}/labels` | AI가 판별한 품종, 카테고리 라벨 | 필요 |
| `DELETE /images/{id}` | 업로드한 이미지 삭제 | 필요 |

업로드 이미지는 내 계정에서만 보입니다. 공개하려면 `POST /images/{id}/public-requests`로 심사를 요청합니다.

## 품종 (키 필요)

| 요청 | 설명 |
|---|---|
| `GET /breeds` | 품종 목록 |
| `GET /breeds/search?q=siamese` | 이름으로 검색 |
| `GET /breeds/{id}` | 품종 1개 조회 (예: `abys`) |
| `GET /breeds/breed-groups` | 품종 그룹 목록 |

- **필터:** `breed_groups`, `exclude_breed_groups`에 그룹 이름을 쉼표로 구분해 넣습니다 (예: `Short-haired,Rex`).
- **Free 요금제 응답:** 이름, 성격, 원산지, 수명, 설명, 이미지만 옵니다.
- **Startup 이상 응답:** 성향 점수(1~5), 특징 값(`indoor`, `hairless` 등)을 포함한 전체 필드가 오고, `lang`으로 번역된 데이터도 받을 수 있습니다.

## 상식 (유료 요금제 필요)

Free 요금제 키로 `GET /facts/random`을 호출하면 403과 `not available on free plan, upgrade for access`가 옵니다 (2026-11-09 직접 확인). 공식 페이지의 요금제 표에는 Free에도 상식이 포함된 것처럼 나와 있어 실제 동작과 다릅니다.

| 요청 | 설명 |
|---|---|
| `GET /facts/random` | 무작위 상식 1개 |
| `GET /facts` | 목록 (`limit`, `offset`, `breed_id`, `category`, `lang`) |
| `GET /facts/breed/{breedId}` | 품종별 상식 |
| `GET /facts/breed/{breedId}/random` | 품종별 무작위 상식 |

## 즐겨찾기 (키 필요)

| 요청 | 설명 |
|---|---|
| `GET /favourites` | 목록 (`?sub_id=`로 필터) |
| `POST /favourites` | 추가. 본문 `{"image_id": "...", "sub_id": "..."}` (`sub_id`는 선택) |
| `GET /favourites/{id}` | 1개 조회 |
| `DELETE /favourites/{id}` | 삭제 |

`sub_id`는 내 서비스의 사용자를 구분하는 값입니다.

## 투표 (키 필요)

| 요청 | 설명 |
|---|---|
| `GET /votes` | 목록 |
| `POST /votes` | 투표. 본문 `{"image_id": "...", "value": 1}` |
| `GET /votes/{vote_id}` | 1개 조회 |

투표는 삭제할 수 없습니다. 취소하려면 반대 `value`로 새로 투표합니다. 문서에 나온 `value` 예시는 `1`뿐입니다.

## 오류 처리

오류 응답은 JSON이고 `statusCode`, `timestamp`, `path`, `message`, `error` 필드가 있습니다. 일부에는 `code`도 있습니다. `message`는 문구가 바뀔 수 있으니 `statusCode`와 `code`로 분기합니다.

| 코드 | 의미 |
|---|---|
| 400 | 잘못된 요청, 월간 업로드 한도 초과 |
| 401 | 계정 비활성 |
| 403 | 키 없음 또는 잘못된 키, 사용할 수 없는 기능, 다른 계정의 리소스 |
| 404 | 없는 ID |
| 409 | 이미 존재함 |
| 415 | 지원하지 않는 업로드 형식 |
| 422 | 요청 형식은 맞지만 처리할 입력이 부족함 |
| 429 | 요청 또는 업로드가 너무 많음 |
| 500 / 503 | 서버 오류 / 사진 분석 일시 중단 |

**재시도:** 429, 500, 503만 재시도합니다. 약 1, 2, 4, 8초로 간격을 늘리고 임의의 시간을 조금 더합니다. 4~5번 실패하면 멈추고 사용자에게 오류를 보여 줍니다.

## 이 프로젝트에서 쓸 때 주의할 점

- **키 노출:** 이 프로젝트는 서버 없이 `index.html`만 쓰므로, 키를 코드에 넣으면 페이지를 보는 누구나 키를 볼 수 있습니다. 키 없이 되는 기능을 먼저 쓰고, 키가 필요한 기능을 공개할 때는 키를 숨길 서버가 필요합니다.
- **키를 넣는 방법:** `fetch`의 `headers`로 보냅니다.

```js
const response = await fetch('https://api.thecatapi.com/v1/images/search?limit=5', {
  headers: { 'x-api-key': apiKey },
});
```

## 문서끼리 다른 부분

| 항목 | 내용 | 정리한 기준 |
|---|---|---|
| Base URL | 명세의 서버 주소는 `beta-api.thecatapi.com`, 가이드는 `api.thecatapi.com` | `api.thecatapi.com` (가이드 기준, 호출 확인) |
| `order` 무작위 값 | 가이드는 `RAND`, 명세는 `RANDOM` | 키를 넣고 둘 다 200, 5번 호출에 5장 모두 다름. `order`를 빼도 같은 결과 (직접 확인) |
| `limit` 기본값 | 명세 설명은 10, 명세 스키마는 1 | 1 (직접 확인) |
| 키 없는 요청의 상태 코드 | 오류 문서는 403, `/favourites`와 `/votes`는 실제로 401 | 둘 다 처리 |

## 다루지 않은 기능

반려동물 등록과 건강 분석(`/pets`), 웹훅(`/webhooks`), 계정 정보(`/accounts`), 칼로리 계산(`/calculators`), 건강 팁(`/health-tips`)은 이 프로젝트와 거리가 있어 뺐습니다. 필요하면 아래 레퍼런스를 참고하세요.

## 출처

- 시작 가이드: https://docs.thecatapi.com/docs/intro
- API 레퍼런스: https://docs.thecatapi.com/reference
- 이미지: https://docs.thecatapi.com/docs/examples/images
- 품종: https://docs.thecatapi.com/docs/examples/breeds
- 투표, 즐겨찾기: https://docs.thecatapi.com/docs/examples/votes-favourites
- 오류와 제한: https://docs.thecatapi.com/docs/errors
- OpenAPI 명세: https://api.thecatapi.com/openapi-json
- 요금제: https://thecatapi.com/
