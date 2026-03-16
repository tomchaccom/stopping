# Stalking Case Similarity Finder

### RAG 기반 법률 질의응답 시스템

이 프로젝트는 사용자의 질문을 입력받아 **벡터 DB(Qdrant)에 저장된 스토킹 관련 판례 중 유사한 사례를 검색**하고, 해당 판례를 기반으로 **GPT가 법률적 판단을 생성하는 법률 상담형 RAG 시스템**입니다.

사용자의 질문 → 벡터 검색 → 판례 검색 → GPT 답변 생성의 흐름으로 동작합니다.

---

# 주요 기능

- 사용자 질문을 **OpenAI Embedding API**를 통해 벡터화
- **Qdrant 벡터 DB**에서 유사한 판례 검색
- 검색된 판례를 기반으로 **GPT가 법률적 판단 및 요약 생성**
- REST API 기반으로 **질문 → 판례 검색 → 결과 반환** 처리

---

# 사용 기술 스택

| 기술 | 설명 |
| --- | --- |
| **Java 21** | 백엔드 개발 언어 |
| **Spring Boot** | REST API 서버 구현 |
| **WebClient** | OpenAI / Qdrant API 호출 |
| **OpenAI API** | 질문 및 판례 Embedding 생성 |
| **Qdrant** | 판례 벡터 저장 및 유사도 검색 |
| **GPT-3.5-turbo** | 유사 판례 기반 법률 답변 생성 |

---

# 시스템 구조

```
User Question
      │
      ▼
OpenAI Embedding API
      │
      ▼
Vector Search (Qdrant)
      │
      ▼
Similar Case Retrieval
      │
      ▼
GPT Prompt 구성
      │
      ▼
Legal Response Generation
```

---

# API 명세

| 기능 | Method | URI | 설명 |
| --- | --- | --- | --- |
| 회원가입 | POST | `/sign` | 신규 사용자 등록 |
| 로그인 | POST | `/login` | 사용자 로그인 (JWT 발급) |
| AI 판결 질의 | GET | `/chat?prompt=` | 사용자 질문 기반 AI 판결 |
| 유사 판례 검색 | POST | `/search/simple` | 질문과 유사한 판례 검색 |
| 지원 제도 조회 | GET | `/supports/{id}` | 특정 지원 제도 기본 정보 |
| 지원 제도 상세 조회 | GET | `/detail/{id}` | 특정 지원 제도 상세 정보 |

---

# API 엔드포인트 상세

---

# 1. 회원가입

### Endpoint

```
POST http://localhost:8080/sign
```

### Request

```
{
  "email":"암호화 테스트",
  "password":"이래 저장되면 안됨"
}
```

### Response

```
{
  "id":6,
  "message":"User signed successfully"
}
```

---

# 2. 로그인

### Endpoint

```
POST http://localhost:8080/login
```

### Request

```
{
  "email":"암호화 테스트",
  "password":"이래 저장되면 안됨"
}
```

### Response

```
{
  "id":"1",
  "token":"JWT_TOKEN"
}
```

### 설명

- 로그인 성공 시 **JWT 토큰 발급**
- 로그인 및 회원가입을 제외한 모든 API 요청 시 **Authorization 헤더에 토큰 포함 필요**

```
Authorization: Bearer {JWT_TOKEN}
```

---

# 3. AI 법률 판결

### Endpoint

```
GET http://localhost:8080/chat?prompt=모르는 사람이 따라옴
```

### Request

```
empty
```

### Response

```
{
  "stalkingType":"물건 전달형 스토킹",
  "isStalkingConvicted":true,
  "physicalApproachOrContact":false,
  "harassmentViaCommunicationOrItems":true,
  "invasionOfPrivacyOrIdentity":false,
  "persistenceOrRepetition":true
}
```

### Response 필드 설명

| 필드 | 설명 |
| --- | --- |
| stalkingType | 스토킹 유형 |
| isStalkingConvicted | 스토킹 성립 여부 |
| physicalApproachOrContact | 물리적 접근 여부 |
| harassmentViaCommunicationOrItems | 통신/물건 전달 괴롭힘 |
| invasionOfPrivacyOrIdentity | 개인정보 침해 여부 |
| persistenceOrRepetition | 지속적 반복 행위 여부 |

---

# 4. 유사 판례 조회

### Endpoint

```
POST http://localhost:8080/search/simple
```

### Request

```
{
  "question":"모르는 사람이 계속 쫓아와요"
}
```

### Response

```
[
  {
    "title":"대법원 2021",
    "content":"2020도6085 전원합의체 판결...",
    "similarity":"79.43%"
  }
]
```

### 설명

- 질문을 벡터화하여 **Qdrant에서 유사 판례 검색**
- 유사도 기준으로 판례 반환
- 결과는 **리스트 형태로 반환**

---

# 5. 지원 제도 조회

### Endpoint

```
GET http://localhost:8080/supports/{id}
```

### Example

```
GET http://localhost:8080/supports/1
```

### Response

```
{
  "category":"신변보호",
  "programName":"임시숙소 지원",
  "description":"범죄피해자에게 임시숙소 제공",
  "contact":"182"
}
```

---

# 6. 지원 제도 상세 조회

### Endpoint

```
GET http://localhost:8080/detail/{id}
```

### Response

```
{
  "programName":"(사)한국여성변호사회",
  "description":"...",
  "target":"...",
  "content":"...",
  "procedureInfo":"...",
  "contact":"02-595-2097"
}
```

### 설명

- 스토킹 피해자를 위한 **법률 지원 프로그램 상세 정보 제공**
