# Stalking Case Similarity Finder (RAG 기반 법률 질의 응답 시스템)

이 프로젝트는 사용자의 질문을 받아, 벡터 DB(Qdrant)에 저장된 스토킹 관련 판례들 중 유사한 사례를 검색하고, 이를 기반으로 GPT를 활용하여 법률적 답변을 생성하는 **법률 상담형 RAG(검색-생성) 시스템**입니다.

---

##  주요 기능

- ✅ 사용자의 질문을 OpenAI Embedding API를 통해 벡터화
- ✅ Qdrant에 저장된 판례 벡터들과 유사도 기반 검색
- ✅ 검색된 유사 판례들을 GPT 프롬프트에 포함해 요약 및 법률적 판단 제공
- ✅ REST API(`/search`)로 질문 → 결과 반환까지 처리

---

##  사용 기술

| 기술               | 설명 |
|--------------------|------|
| **Java 21**        | 백엔드 언어 |
| **Spring Boot**    | 백엔드 프레임워크 |
| **WebClient**      | 외부 API 호출(OpenAI, Qdrant) |
| **OpenAI API**     | 질문 및 판례 Embedding 생성 (`text-embedding-ada-002`) |
| **Qdrant**         | 벡터 DB로 판례 벡터 저장 및 유사도 검색 |
| **GPT-3.5-turbo**  | 유사 판례 기반 답변 생성 (향후 프롬프트 연동 예정) |

---

# 시스템 구조

## API 명세

| 기능                | HTTP Method | URI                  | 설명 |
|---------------------|-------------|----------------------|------|
| 회원가입            | POST        | `/sign`              | 신규 사용자 등록 |
| 로그인              | POST        | `/login`             | 사용자 로그인 (JWT 발급) |
| AI를 통한 판결 질의 | GET         | `/chat?prompt=`      | 사용자의 질문을 받아 AI가 답변 생성 |
| 유사 판례 조회      | POST        | `/search/simple`     | 질문과 유사한 판례 검색 (RAG 기반) |
| 지원 제도 조회(기본)| GET         | `/supports/{id}`     | 특정 지원 제도 기본 정보 조회 |
| 지원 제도 조회(상세)| GET         | `/detail/{id}`       | 특정 지원 제도 상세 정보 조회 |

#  API 엔드포인트 

## 회원가입

`http://localhost:8080/sign`

## @RequestBody

```json
{
    "email":"암호화 테스트",
    "password":"이래 저장되면 안됨"
}
```

## @ResponseBody

```json
{
    "id": 6,
    "message": "User signed successfully"
}
```
##  로그인

`http://localhost:8080/login`

## @RequestBody

```json
{
    "email":"암호화 테스트",
    "password":"이래 저장되면 안됨"
}
```

## @ResponseBody

```json
{
    "id":"1",
    "token":"eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiLslZTtmLjtmZQg7YWM7Iqk7Yq4IiwiaWF0IjoxNzQ4NTAwMDEwLCJleHAiOjE3NDg1MDcyMTB9.lLXlhK-45vYMw9zL-3l3aSayAUPiuqmINeojJDP7"
}
```

- 응답으로 JWT 토큰을 발급해줌. 로그인 회원가입 외의 API 요청에는 반드시 포함시킬 것

![image.png](attachment:f9026eae-0197-40b9-a575-1d9738b66444:image.png)


## AI 법률 판결

`http://localhost:8080/chat?prompt=모르는 사람이 따라옴`

## @RequestBody

```json
// empty
```

## @ResponseBody

```json
{
    "stalkingType": "물건 전달형 스토킹",
    "isStalkingConvicted": true,
    "physicalApproachOrContact": false,
    "harassmentViaCommunicationOrItems": true,
    "invasionOfPrivacyOrIdentity": false,
    "persistenceOrRepetition": true
}
```

- 스토킹 유형
- 스토킹 여부
- 물리적접근 및 접촉
- 통신 기기를 통한 괴롭힘
- 개인정보 침해
- 지속성 반복성

출력 결과 - postman

![image.png](attachment:ea0b3f0d-59f0-4e8c-b040-6cf031502995:image.png)



## 유사 판례 조회

`http://localhost:8080/search/simple`

## @RequestBody

```jsx
{
    "question" : "모르는 사람이 계속 쫓아와요"
}
```

## @ResponseBody

```jsx
[
    {
        "title": "대법원 2021.",
        "content": "9. 9. 선고 2020도6085 전원합의체 판결 가정불화로 처와 일시 별거 중인 남편이 그의 부모와 함께 주거지에 들어가려고 하는데 처로부터 집을 돌보아 달라는 부탁을 받은 처제가 출입을 못하게 하자, 출입문에 설치된 잠금장치를 손괴하고 주거지에 출입하여 폭력행위 등 처벌에 관한 법률위반(공동주거침입)죄 등으로 기소된 사안",
        "similarity": "79.43%"
    }
]
```

- 기존에 리스트 형식으로 반환했기 때문에 파싱 시 리스트 내부 요소 접근이 요구될 수도 있음

## 지원 제도 조회

`http://localhost:8080/supports/1`

## @RequestBody

```json
// empty
```

## @ResponseBody

```json
{
    "category": "신변보호",
    "programName": "임시숙소 지원",
    "description": "가해자에게 주거지가 노출되어 보복범죄 등 추가 피해가 우려되거나 방화 등 범죄피해로 당장 거주할 곳이 없는 범죄피해자에게 임시숙소를 제공하는 제도",
    "contact": "182"
}
```

![image.png](attachment:e43b38c4-aa3b-4f1b-b815-97e7b265368b:image.png)

## 지원 제도 조회(상세)

`http://localhost:8080/detail/1`

## @RequestBody

```json

```

## @ResponseBody

```json
{
    "programName": "(사)한국여성변호사회",
    "description": "여성가족부 위탁사업으로 스토킹 및 교제폭력 등 신종폭력 피해자에게 형사, 민사, 가사 소송 등 법률지원(소송대리)을 무료로 제공",
    "target": "스토킹범죄의 처벌 등에 관한 법률 제2조 제1호 각호에 따른 스토킹 피해자, 단 채권·채무관계·층간소음 등의 갈등으로 발생하는 스토킹 피해자는 제외\n    성별을 기반으로 한 교제하는 사이에서 발생한 성폭력, 사이버 성폭력, 폭행, 상해, 명예훼손, 모욕, 강요, 협박 등 피해자\n    피해자를 지원하는 정당한 업무수행과정 중에 법적분쟁이 발생한 스토킹·교제폭력 및 성폭력 피해자 지원시설 종사자. 단 고의 또는 중과실이 아닌 경우에 한한다.\n    디지털콘텐츠 및 기사 등 사건 처리과정에서 2차 피해를 당한 피해자\n    기타 운영위원회가 구조함이 상당하다고 인정한 신종 폭력 피해자",
    "content": "- 법률상담\n     · 법률지원 요청 피해자를 대상으로 소송지원 여부 판단을 위한 초기 상담 진행\n    - 형사 소송지원\n     · 피해자의 형사사건과 관련하여 무료변호, 수사의뢰, 수사기관 사건 조사 동행, 고소대리\n    - 민사, 가사 소송지원\n     · 스토킹·교제폭력과 관련하여 계속 중이거나 진행이 필요한 소송의 지원(소장, 답변서, 의견서 작성 및 변론기일, 조정기일 출석 등 일체의 소송대리 지원)",
    "procedureInfo": "무료법률지원신청은 스토킹 및 교제폭력 등 피해자 대상 무료법률지원사업 홈페이지 (https://help.kwla.or.kr)를 통한 상담신청서 접수 또는 이메일 접수(help-kwla@naver.com)를 통해서 가능",
    "contact": "02-595-2097"
}
```

![image.png](attachment:ad416881-640d-4592-a7dd-c0724c25d840:image.png)
