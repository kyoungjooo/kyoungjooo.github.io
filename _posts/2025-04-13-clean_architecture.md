---
title: "Clean Architecture/ DDD(도메인 주도 설계)"
layout: single
categories: database
tag: [클린 아키텍처, next.js]
toc: false
author_profile: false
search: true
---

## 클린 아키텍처 / DDD(도메인 주도 설계) 구조

Next.js 기반 프로젝트에서 클린 아키텍처(Clean Architecture) + 계층별 역할 분리하기

## 클린 아키텍처 핵심 2가지 원칙

## 1. 단방향 의존성 원칙 (Dependency Rule)

내부 계층은 외부 계층에 의존할 수 없으며, 외부 계층은 내부 계층에 의존해야한다.<br>
의존성 방향 = 바깥 → 안쪽만 허용

```jsx
//구조 흐름
UI → Controller → Service(UseCase) → Repository Interface → Infra 구현체
//내부 계층(Infra)은 상위 계층(Service, Controller)에 의존할 수 없다.
```

🤔 이 과정에서 내부계층의 infra를 상위계층에서 사용하기 위해 IoC + DI (제어 역전 + 의존성 주입)이 필요하다.

- IoC(Inversion of Control)제어 역전<br>
  "프레임워크"가 알아서 객체 생성 + 흐름 제어
- DI(Dependency Injection)의존성 주입<br>
  필요한 객체(Repository 등)를 직접 만들지 않고, 외부에서 대신 넣어주는 것

  (이부분은 다음에 정리해보자..)

## 2. 관심사의 분리 (Separation of Concerns)

서비스(Service) 계층 분리하기, 각 계층은 특정 역할 하나에만 집중해야한다.<br>

### 🎯 각 디렉토리의 계층과 역할

```jsx
// 클린 아키텍처 레이어 계층 구조
page → adapter → service → repository → entity(domain) 구조
```

| 계층       | 역할 (책임)                            | 핵심 개념                    |
| ---------- | -------------------------------------- | ---------------------------- |
| page       | UI / View (presentation Layer)         | 사용자 화면, 이벤트 발생     |
| adapter    | Controller (Interface Adapters Layer)  | API 요청 처리, 데이터 가공   |
| service    | UseCase (Application Layer)            | 서비스 로직, 규칙 처리       |
| repository | 데이터 처리 (Infra Layer)              | 외부 저장소(DB/API)에 접근   |
| entity     | 순수 비즈니스 규칙 처리 (Domain Layer) | 핵심 데이터 객체 & 규칙 처리 |

<small>\* Entity 계층 → Domain 계층 내부의 핵심 객체</small>

### 폴더구조 살펴보기

일단 이해가 어려우니.. 강사님이 만든 폴더구조를 살펴보기..

```jsx
app/ //사용자의 요청이 들어오는 Next.js의 페이지 컴포넌트
├── api/ //Controller (API 엔드포인트)
│   └── categories/
│       └── route.ts //client에게 요청 받고 UseCase 호출 후 응답 반환
│                    // Next에서 app/api/route.ts 구조를 보고 자동으로 API endpoint 생성
│
application/ //서비스, 비즈니스 로직 실행 계층
├── usecases/ //서비스 로직
│   └── category/
│       └── dto/ //화면(API)에 전달할 전용 데이터 구조 정의
│           └── CategoryDto.ts
│       └── GetCategoryListUsecase.ts //카테고리 리스트 조회 비즈니스 로직
│
domain/ //도메인, 업무 로직 계층
├── entities/ //도메인 모델
│   └── Category.ts //new로 인스턴스 만들어서 비즈니스 모델 (Entity) 정의
├── repository/ //데이터 접근 인터페이스, 필요한 데이터 목록 선언
│   └── CategoryRepository.ts //CategoryRepository interface 정의
│
infra/ //실제 구현 계층
└── repositories/
    └── supabase/
        └── SbUserRepository.ts //데이터 가져오는 구체적인 방법
```

위에서부터 순서대로 살펴보자..🫠

### 1. app/ (UI / page 계층)

### 2. api/ (controller / adapter 계층)

- 외부와 내부의 데이터를 연결하는 어댑터 역할
- 외부 요청을 받는 역할
- 요청 데이터 검증 (Request DTO 검증 + 전달)
- Response DTO 변환 (클라이언트가 보기 좋게 가공, Entity → DTO 변환)
- Service 호출, Service / UseCase에 있는 핵심 로직 실행

### 3. application/ (usecase / service 계층)

- 비즈니스 로직 처리
- Repository 호출
- 비즈니스 로직 처리 중 필요한 데이터 가공(API로 받은 데이터(DTO) → Entity 변환)
- Repository Interface를 호출해서 데이터를 가져오거나 저장

### 4. domain/ (Entity, Business Logic / entity 계층)

- 서비스 비즈니스에 대한 "핵심 개념"과 "규칙(룰)"만 정의
- entity : 핵심 비즈니스 데이터 모델
- repository interface : 데이터 접근 명세(필요한 메서드 정의)

### 5. infra/ (실제 구현 / repository 계층)

- 데이터베이스, 외부 API 등 구체적인 구현 담당

### 🤔 DTO 변환 왜 필요한가?

예를 들어 DB에 저장된 카테고리 데이터가 이렇게 생겼을 때

```jsx
// Entity
export class Category {
  constructor(
    public id: number,
    public name: string,
    public order: number,
    public createdAt: Date
  ){}
}
```

클라이언트에서 원하는 데이터가 아래와 같을 때 entity 데이터를 그대로 보내면 필요 없는 데이터도 보내진다.<br>
불필요한 데이터 제거, 구조를 맞추기 위해 필요

```jsx
[
  {
    id: 1,
    name: "americano",
    order: 1,
  },
];
```

<br>

글로 적으니 좀 정리가 되는 것 같지만 아직 잘 모르겠다,, 일단 만들어 보는 걸루
