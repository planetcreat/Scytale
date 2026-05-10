# 수원대 스마트 시간표 — API 명세서

> **Version** 1.0  
> **Base URL** `https://api.suwon-timetable.kr/v1`  
> **인증** Bearer Token (JWT)  
> **Content-Type** `application/json`

---

## 목차

1. [학과 정보 조회](#1-학과-정보-조회)
2. [학생 프로필 등록/수정](#2-학생-프로필-등록수정)
3. [학생 프로필 조회](#3-학생-프로필-조회)
4. [이수 학점 저장](#4-이수-학점-저장)
5. [시간표 추천 요청](#5-시간표-추천-요청)
6. [시간표 추천 결과 조회](#6-시간표-추천-결과-조회)
7. [시간표 저장](#7-시간표-저장)
8. [저장된 시간표 목록 조회](#8-저장된-시간표-목록-조회)
9. [빈 강의실 조회](#9-빈-강의실-조회)
10. [건물 목록 조회](#10-건물-목록-조회)
11. [졸업요건 분석 조회](#11-졸업요건-분석-조회)
12. [필수과목 이수현황 조회](#12-필수과목-이수현황-조회)
13. [추천 과목 조회](#13-추천-과목-조회)
14. [선수과목 조회](#14-선수과목-조회)
15. [과목 검색](#15-과목-검색)

---

## 1. 학과 정보 조회

대학 → 학부/학과 → 전공 3단계 캐스케이딩 드롭다운에 사용되는 학과 계층 데이터를 조회합니다.

| 항목 | 내용 |
|------|------|
| **Endpoint** | `GET /departments` |
| **Method** | GET |
| **설명** | 대학 · 학부 · 전공 계층 구조를 조회한다. query parameter로 특정 대학 또는 학부 하위만 필터링할 수 있다. |

### Request

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `college` | string | X | 대학명 필터 (예: `지능형SW융합대학`) |
| `department` | string | X | 학부/학과명 필터 (예: `컴퓨터학부`) |

**요청 예시**

```
GET /v1/departments?college=지능형SW융합대학
Authorization: Bearer {token}
```

### Response

**Status 200 OK**

```json
{
  "success": true,
  "data": [
    {
      "college": "지능형SW융합대학",
      "departments": [
        {
          "name": "데이터과학부",
          "majors": []
        },
        {
          "name": "컴퓨터학부",
          "majors": ["컴퓨터SW전공", "미디어SW전공"]
        },
        {
          "name": "정보통신학부",
          "majors": ["정보통신전공", "정보보호전공"]
        }
      ]
    }
  ]
}
```

---

## 2. 학생 프로필 등록/수정

학생의 기본 정보(대학, 학부, 전공, 학년, 입학유형 등)를 등록하거나 수정합니다.

| 항목 | 내용 |
|------|------|
| **Endpoint** | `PUT /students/me/profile` |
| **Method** | PUT |
| **설명** | 현재 로그인한 학생의 프로필을 등록 또는 수정한다. (멱등성 보장) |

### Request

**Request Body**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `college` | string | O | 대학명 |
| `department` | string | O | 학부/학과명 |
| `major` | string | X | 전공명 (단일학과인 경우 null) |
| `grade` | integer | O | 학년 (1~4) |
| `admission_type` | string | O | 입학유형: `normal` 또는 `transfer` |
| `transfer_grade` | integer | X | 편입 학년 (편입생일 경우, 예: 3) |
| `transfer_credits` | integer | X | 편입 전 인정학점 (편입생일 경우, 0~100) |

**요청 예시**

```json
PUT /v1/students/me/profile
Authorization: Bearer {token}

{
  "college": "지능형SW융합대학",
  "department": "컴퓨터학부",
  "major": "컴퓨터SW전공",
  "grade": 3,
  "admission_type": "normal",
  "transfer_grade": null,
  "transfer_credits": null
}
```

### Response

**Status 200 OK**

```json
{
  "success": true,
  "data": {
    "student_id": "STU-20230001",
    "college": "지능형SW융합대학",
    "department": "컴퓨터학부",
    "major": "컴퓨터SW전공",
    "grade": 3,
    "admission_type": "normal",
    "transfer_grade": null,
    "transfer_credits": null,
    "updated_at": "2026-05-06T09:30:00Z"
  }
}
```

---

## 3. 학생 프로필 조회

| 항목 | 내용 |
|------|------|
| **Endpoint** | `GET /students/me/profile` |
| **Method** | GET |
| **설명** | 현재 로그인한 학생의 프로필 정보를 조회한다. |

### Request

파라미터 없음.

```
GET /v1/students/me/profile
Authorization: Bearer {token}
```

### Response

**Status 200 OK**

```json
{
  "success": true,
  "data": {
    "student_id": "STU-20230001",
    "college": "지능형SW융합대학",
    "department": "컴퓨터학부",
    "major": "컴퓨터SW전공",
    "grade": 3,
    "admission_type": "normal",
    "transfer_grade": null,
    "transfer_credits": null,
    "created_at": "2026-03-01T00:00:00Z",
    "updated_at": "2026-05-06T09:30:00Z"
  }
}
```

---

## 4. 이수 학점 저장

학생의 현재까지 이수 완료한 학점(전공필수, 전공핵심, 전공선택, 교양)을 저장합니다.

| 항목 | 내용 |
|------|------|
| **Endpoint** | `PUT /students/me/credits` |
| **Method** | PUT |
| **설명** | 학생의 영역별 이수 학점을 저장한다. |

### Request

**Request Body**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `major_required` | integer | O | 전공필수 이수 학점 (0~50) |
| `major_core` | integer | O | 전공핵심 이수 학점 (0~50) |
| `major_elective` | integer | O | 전공선택 이수 학점 (0~50) |
| `general` | integer | O | 교양 이수 학점 (0~50) |

**요청 예시**

```json
PUT /v1/students/me/credits
Authorization: Bearer {token}

{
  "major_required": 33,
  "major_core": 12,
  "major_elective": 18,
  "general": 22
}
```

### Response

**Status 200 OK**

```json
{
  "success": true,
  "data": {
    "major_required": 33,
    "major_core": 12,
    "major_elective": 18,
    "general": 22,
    "total": 85,
    "updated_at": "2026-05-06T09:35:00Z"
  }
}
```

---

## 5. 시간표 추천 요청

입력된 조건(시작 시간, 선호 시간대, 공강 요일, 목표 학점, 관심 분야 등)을 기반으로 최적의 시간표를 추천받습니다.

| 항목 | 내용 |
|------|------|
| **Endpoint** | `POST /timetables/recommend` |
| **Method** | POST |
| **설명** | 설정된 조건을 기반으로 시간표 추천을 요청한다. 비동기 처리 시 `recommendation_id`를 반환한다. |

### Request

**Request Body**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `start_hour` | integer | O | 수업 시작 가능 시간 (8~12) |
| `time_preference` | string | O | 선호 시간대: `morning`, `afternoon`, `any` |
| `free_days` | string[] | X | 공강 원하는 요일: `mon`, `tue`, `wed`, `thu`, `fri` |
| `target_credits` | integer | O | 이번 학기 목표 학점 (12~21) |
| `plan_period` | string | O | 계획 기간: `single` (이번 학기만), `year` (1년 계획) |
| `interests` | string[] | X | 교양 관심 분야: `music`, `sports`, `language`, `it`, `humanities`, `science`, `business`, `design` |
| `plan_count` | integer | O | 추천 시간표 수: 1 또는 3 |

**요청 예시**

```json
POST /v1/timetables/recommend
Authorization: Bearer {token}

{
  "start_hour": 9,
  "time_preference": "any",
  "free_days": ["tue", "fri"],
  "target_credits": 18,
  "plan_period": "single",
  "interests": ["it", "science"],
  "plan_count": 3
}
```

### Response

**Status 201 Created**

```json
{
  "success": true,
  "data": {
    "recommendation_id": "REC-20260506-001",
    "status": "completed",
    "created_at": "2026-05-06T09:40:00Z",
    "plans": [
      {
        "plan_id": "PLAN-001",
        "tag": "화·금 공강",
        "free_days": ["화요일", "금요일"],
        "reason": "오전 9시 시작, 월·수·목만 등교하는 균형 잡힌 주간 구성",
        "total_credits": 14,
        "courses": [
          {
            "course_id": "CSE3001",
            "name": "선형대수학",
            "professor": "박서연",
            "credits": 3,
            "type": "major_required",
            "type_label": "전공필수",
            "schedule": [
              {
                "day": "mon",
                "start_period": 1,
                "end_period": 2,
                "start_time": "09:00",
                "end_time": "10:50"
              },
              {
                "day": "wed",
                "start_period": 1,
                "end_period": 2,
                "start_time": "09:00",
                "end_time": "10:50"
              }
            ],
            "room": "공학관 304"
          },
          {
            "course_id": "CSE3002",
            "name": "운영체제",
            "professor": "김민준",
            "credits": 3,
            "type": "major_required",
            "type_label": "전공필수",
            "schedule": [
              {
                "day": "mon",
                "start_period": 3,
                "end_period": 4,
                "start_time": "11:00",
                "end_time": "12:50"
              },
              {
                "day": "wed",
                "start_period": 3,
                "end_period": 4,
                "start_time": "11:00",
                "end_time": "12:50"
              }
            ],
            "room": "정보과학관 301"
          }
        ]
      }
    ]
  }
}
```

---

## 6. 시간표 추천 결과 조회

이전에 생성된 추천 시간표를 다시 조회합니다.

| 항목 | 내용 |
|------|------|
| **Endpoint** | `GET /timetables/recommend/{recommendation_id}` |
| **Method** | GET |
| **설명** | 추천 결과를 recommendation_id로 조회한다. |

### Request

**Path Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `recommendation_id` | string | O | 추천 요청 ID |

```
GET /v1/timetables/recommend/REC-20260506-001
Authorization: Bearer {token}
```

### Response

**Status 200 OK**

```json
{
  "success": true,
  "data": {
    "recommendation_id": "REC-20260506-001",
    "status": "completed",
    "created_at": "2026-05-06T09:40:00Z",
    "plans": [
      {
        "plan_id": "PLAN-001",
        "tag": "화·금 공강",
        "free_days": ["화요일", "금요일"],
        "reason": "오전 9시 시작, 월·수·목만 등교하는 균형 잡힌 주간 구성",
        "total_credits": 14,
        "credit_breakdown": {
          "major_required": 9,
          "major_elective": 3,
          "general_required": 2,
          "general_elective": 0
        },
        "course_count": 5,
        "courses": ["...과목 배열 (5번 API와 동일 구조)"]
      }
    ]
  }
}
```

---

## 7. 시간표 저장

추천된 시간표 중 하나를 선택하여 저장합니다.

| 항목 | 내용 |
|------|------|
| **Endpoint** | `POST /timetables/saved` |
| **Method** | POST |
| **설명** | 추천 플랜 중 하나를 선택하여 내 시간표로 저장한다. |

### Request

**Request Body**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `recommendation_id` | string | O | 추천 요청 ID |
| `plan_id` | string | O | 선택한 플랜 ID |
| `semester` | string | X | 학기 (예: `2026-1`). 미입력 시 현재 학기 자동 적용 |

**요청 예시**

```json
POST /v1/timetables/saved
Authorization: Bearer {token}

{
  "recommendation_id": "REC-20260506-001",
  "plan_id": "PLAN-001",
  "semester": "2026-1"
}
```

### Response

**Status 201 Created**

```json
{
  "success": true,
  "data": {
    "timetable_id": "TT-20260506-001",
    "semester": "2026-1",
    "plan_id": "PLAN-001",
    "tag": "화·금 공강",
    "total_credits": 14,
    "course_count": 5,
    "saved_at": "2026-05-06T10:00:00Z"
  },
  "message": "시간표가 저장되었습니다."
}
```

---

## 8. 저장된 시간표 목록 조회

| 항목 | 내용 |
|------|------|
| **Endpoint** | `GET /timetables/saved` |
| **Method** | GET |
| **설명** | 학생이 저장한 시간표 목록을 조회한다. |

### Request

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `semester` | string | X | 학기 필터 (예: `2026-1`) |

```
GET /v1/timetables/saved?semester=2026-1
Authorization: Bearer {token}
```

### Response

**Status 200 OK**

```json
{
  "success": true,
  "data": [
    {
      "timetable_id": "TT-20260506-001",
      "semester": "2026-1",
      "tag": "화·금 공강",
      "total_credits": 14,
      "course_count": 5,
      "saved_at": "2026-05-06T10:00:00Z"
    }
  ]
}
```

---

## 9. 빈 강의실 조회

특정 교시와 건물에 대해 강의실 사용 가능 여부를 조회합니다.

| 항목 | 내용 |
|------|------|
| **Endpoint** | `GET /rooms/availability` |
| **Method** | GET |
| **설명** | 지정된 건물 · 교시에 대한 강의실 사용 가능 현황을 조회한다. 교시를 지정하지 않으면 현재 시간 기준으로 자동 판별한다. |

### Request

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `building_id` | string | X | 건물 ID (예: `it`). 미입력 시 전체 건물 |
| `period` | integer | X | 교시 번호 (1~9). 미입력 시 현재 교시 자동 감지 |
| `day` | string | X | 요일: `mon`~`fri`. 미입력 시 오늘 |

**요청 예시**

```
GET /v1/rooms/availability?building_id=it&period=3
Authorization: Bearer {token}
```

### Response

**Status 200 OK**

```json
{
  "success": true,
  "data": {
    "building_id": "it",
    "building_name": "정보과학관",
    "queried_period": 3,
    "queried_day": "wed",
    "summary": {
      "total": 7,
      "available": 4,
      "busy": 3
    },
    "rooms": [
      {
        "room_id": "it-101",
        "name": "101호",
        "capacity": 60,
        "tags": ["빔프로젝터", "에어컨"],
        "is_available": true,
        "schedule": [
          { "period": 1, "status": "free" },
          { "period": 2, "status": "occupied" },
          { "period": 3, "status": "free" },
          { "period": 4, "status": "free" },
          { "period": 5, "status": "free" },
          { "period": 6, "status": "occupied" },
          { "period": 7, "status": "occupied" },
          { "period": 8, "status": "free" },
          { "period": 9, "status": "free" }
        ]
      },
      {
        "room_id": "it-201",
        "name": "201호",
        "capacity": 40,
        "tags": ["빔프로젝터", "에어컨", "PC실"],
        "is_available": false,
        "current_course": "자료구조 (오민성 교수)",
        "schedule": [
          { "period": 1, "status": "occupied" },
          { "period": 2, "status": "occupied" },
          { "period": 3, "status": "occupied" },
          { "period": 4, "status": "free" },
          { "period": 5, "status": "occupied" },
          { "period": 6, "status": "occupied" },
          { "period": 7, "status": "free" },
          { "period": 8, "status": "free" },
          { "period": 9, "status": "free" }
        ]
      }
    ]
  }
}
```

---

## 10. 건물 목록 조회

| 항목 | 내용 |
|------|------|
| **Endpoint** | `GET /buildings` |
| **Method** | GET |
| **설명** | 캠퍼스 내 건물 목록과 각 건물의 총 강의실 수를 조회한다. |

### Request

파라미터 없음.

```
GET /v1/buildings
Authorization: Bearer {token}
```

### Response

**Status 200 OK**

```json
{
  "success": true,
  "data": [
    {
      "building_id": "it",
      "name": "정보과학관",
      "icon": "💻",
      "total_rooms": 7
    },
    {
      "building_id": "eng",
      "name": "공학관",
      "icon": "⚙️",
      "total_rooms": 7
    },
    {
      "building_id": "hum",
      "name": "인문사회관",
      "icon": "📚",
      "total_rooms": 5
    },
    {
      "building_id": "biz",
      "name": "경영관",
      "icon": "💼",
      "total_rooms": 4
    }
  ]
}
```

---

## 11. 졸업요건 분석 조회

학생의 프로필과 이수 학점을 기반으로 졸업요건 달성 현황을 분석합니다.

| 항목 | 내용 |
|------|------|
| **Endpoint** | `GET /graduation/analysis` |
| **Method** | GET |
| **설명** | 현재 학생의 졸업요건 달성률을 영역별로 분석하여 반환한다. |

### Request

파라미터 없음. (인증 토큰으로 학생 식별)

```
GET /v1/graduation/analysis
Authorization: Bearer {token}
```

### Response

**Status 200 OK**

```json
{
  "success": true,
  "data": {
    "student_summary": {
      "college": "지능형SW융합대학",
      "department": "컴퓨터학부",
      "major": "컴퓨터SW전공",
      "grade": 3,
      "semester": "2학기"
    },
    "overall": {
      "done_credits": 90,
      "total_required": 130,
      "remaining": 40,
      "percentage": 69
    },
    "categories": [
      {
        "name": "전공필수",
        "done": 33,
        "total": 45,
        "remaining": 12,
        "percentage": 73,
        "color": "#3B82F6"
      },
      {
        "name": "전공선택",
        "done": 18,
        "total": 30,
        "remaining": 12,
        "percentage": 60,
        "color": "#7C3AED"
      },
      {
        "name": "교양필수",
        "done": 14,
        "total": 20,
        "remaining": 6,
        "percentage": 70,
        "color": "#16A34A"
      },
      {
        "name": "교양선택",
        "done": 8,
        "total": 10,
        "remaining": 2,
        "percentage": 80,
        "color": "#D97706"
      },
      {
        "name": "기타·자유",
        "done": 17,
        "total": 25,
        "remaining": 8,
        "percentage": 68,
        "color": "#EC4899"
      }
    ]
  }
}
```

---

## 12. 필수과목 이수현황 조회

전공필수 · 교양필수 과목에 대한 이수 여부를 과목 단위로 조회합니다.

| 항목 | 내용 |
|------|------|
| **Endpoint** | `GET /graduation/checklist` |
| **Method** | GET |
| **설명** | 필수과목 목록과 각 과목의 이수 상태(이수완료, 수강중, 미이수)를 반환한다. |

### Request

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `category` | string | X | 필터: `major_required`, `general_required`. 미입력 시 전체 |

```
GET /v1/graduation/checklist
Authorization: Bearer {token}
```

### Response

**Status 200 OK**

```json
{
  "success": true,
  "data": [
    {
      "category": "전공필수",
      "color": "#3B82F6",
      "items": [
        { "course_id": "CSE1001", "name": "이산수학",       "credits": 3, "status": "done" },
        { "course_id": "CSE1002", "name": "자료구조",       "credits": 3, "status": "done" },
        { "course_id": "CSE2001", "name": "알고리즘",       "credits": 3, "status": "done" },
        { "course_id": "CSE2002", "name": "운영체제",       "credits": 3, "status": "done" },
        { "course_id": "CSE2003", "name": "데이터베이스",   "credits": 3, "status": "done" },
        { "course_id": "CSE3001", "name": "선형대수학",     "credits": 3, "status": "in_progress" },
        { "course_id": "CSE3002", "name": "컴퓨터구조",     "credits": 3, "status": "not_taken" },
        { "course_id": "CSE4001", "name": "캡스톤디자인",   "credits": 3, "status": "not_taken" },
        { "course_id": "CSE3003", "name": "소프트웨어공학", "credits": 3, "status": "not_taken" },
        { "course_id": "CSE3004", "name": "컴파일러이론",   "credits": 3, "status": "not_taken" }
      ]
    },
    {
      "category": "교양필수",
      "color": "#16A34A",
      "items": [
        { "course_id": "GEN1001", "name": "글쓰기와소통",      "credits": 2, "status": "done" },
        { "course_id": "GEN1002", "name": "기술영어",          "credits": 2, "status": "done" },
        { "course_id": "GEN1003", "name": "창의적사고와문제",  "credits": 2, "status": "done" },
        { "course_id": "GEN1004", "name": "소양과인성",        "credits": 2, "status": "done" },
        { "course_id": "GEN1005", "name": "영어회화 1",        "credits": 2, "status": "done" },
        { "course_id": "GEN1006", "name": "영어회화 2",        "credits": 2, "status": "done" },
        { "course_id": "GEN2001", "name": "취업전략세미나",    "credits": 2, "status": "in_progress" }
      ]
    }
  ]
}
```

> **status 값**: `done` (이수 완료), `in_progress` (수강 중), `not_taken` (미이수)

---

## 13. 추천 과목 조회

졸업요건 분석 결과를 기반으로 이번 학기에 수강이 추천되는 과목을 조회합니다.

| 항목 | 내용 |
|------|------|
| **Endpoint** | `GET /graduation/recommendations` |
| **Method** | GET |
| **설명** | 미이수 필수과목과 졸업요건을 고려하여 이번 학기 수강 추천 과목을 반환한다. |

### Request

파라미터 없음.

```
GET /v1/graduation/recommendations
Authorization: Bearer {token}
```

### Response

**Status 200 OK**

```json
{
  "success": true,
  "data": [
    {
      "course_id": "CSE3002",
      "name": "컴퓨터구조",
      "professor": "이승훈",
      "credits": 3,
      "type": "major_required",
      "type_label": "전공필수",
      "color": "#3B82F6",
      "reason": "졸업필수 미이수",
      "schedule_text": "월수 11:00~12:50"
    },
    {
      "course_id": "CSE4001",
      "name": "캡스톤디자인",
      "professor": "김태훈",
      "credits": 3,
      "type": "major_required",
      "type_label": "전공필수",
      "color": "#3B82F6",
      "reason": "졸업필수 미이수",
      "schedule_text": "월 14:00~16:50"
    },
    {
      "course_id": "CSE3003",
      "name": "소프트웨어공학",
      "professor": "윤소희",
      "credits": 3,
      "type": "major_required",
      "type_label": "전공필수",
      "color": "#3B82F6",
      "reason": "졸업필수 미이수",
      "schedule_text": "화목 13:00~14:50"
    },
    {
      "course_id": "CSE3004",
      "name": "컴파일러이론",
      "professor": "배지수",
      "credits": 3,
      "type": "major_elective",
      "type_label": "전공선택",
      "color": "#7C3AED",
      "reason": "전공선택 12학점 미달",
      "schedule_text": "월화 11:00~12:50"
    },
    {
      "course_id": "GEN2001",
      "name": "취업전략세미나",
      "professor": "강인서",
      "credits": 2,
      "type": "general_required",
      "type_label": "교양필수",
      "color": "#16A34A",
      "reason": "교양필수 미이수",
      "schedule_text": "금 10:00~11:50"
    },
    {
      "course_id": "CSE3005",
      "name": "인공지능개론",
      "professor": "강도현",
      "credits": 3,
      "type": "major_elective",
      "type_label": "전공선택",
      "color": "#7C3AED",
      "reason": "전공선택 채우기 추천",
      "schedule_text": "수 10:00~12:50"
    }
  ]
}
```

---

## 14. 선수과목 조회

학생의 전공(또는 학부)에 해당하는 과목들의 선수과목 관계와 수강 가능 여부를 조회합니다.

| 항목 | 내용 |
|------|------|
| **Endpoint** | `GET /graduation/prerequisites` |
| **Method** | GET |
| **설명** | 전공/학부 기준 과목별 선수과목 이수 여부와 수강 가능 상태를 반환한다. |

### Request

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `scope` | string | X | 조회 범위: `department` (학부 공통), `major` (전공). 미입력 시 `major` 우선 |

```
GET /v1/graduation/prerequisites?scope=department
Authorization: Bearer {token}
```

### Response

**Status 200 OK**

```json
{
  "success": true,
  "data": {
    "scope": "컴퓨터학부",
    "courses": [
      {
        "course_id": "CSE1001",
        "name": "C프로그래밍",
        "credits": 3,
        "status": "done",
        "can_take": true,
        "prerequisites": []
      },
      {
        "course_id": "CSE3002",
        "name": "컴퓨터구조",
        "credits": 3,
        "status": "not_taken",
        "can_take": true,
        "prerequisites": [
          { "course_id": "CSE1001-2", "name": "이산수학", "is_completed": true }
        ]
      },
      {
        "course_id": "CSE3005",
        "name": "인공지능개론",
        "credits": 3,
        "status": "not_taken",
        "can_take": false,
        "prerequisites": [
          { "course_id": "CSE3001", "name": "선형대수학", "is_completed": false },
          { "course_id": "STAT2001", "name": "확률과통계", "is_completed": false }
        ]
      },
      {
        "course_id": "CSE4001",
        "name": "캡스톤디자인",
        "credits": 3,
        "status": "not_taken",
        "can_take": false,
        "prerequisites": [
          { "course_id": "CSE3003", "name": "소프트웨어공학", "is_completed": false }
        ]
      }
    ]
  }
}
```

> **status 값**: `done` (이수 완료), `taking` (수강 중), `not_taken` (미이수)  
> **can_take**: 선수과목이 모두 이수되었으면 `true`, 아니면 `false`

---

## 15. 과목 검색

개설 과목을 키워드, 교수명, 과목 유형, 요일 등으로 검색합니다.

| 항목 | 내용 |
|------|------|
| **Endpoint** | `GET /courses/search` |
| **Method** | GET |
| **설명** | 현재 학기 개설 과목을 다양한 조건으로 검색한다. |

### Request

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `keyword` | string | X | 과목명 또는 교수명 검색어 |
| `type` | string | X | 과목 유형: `major_required`, `major_elective`, `general_required`, `general_elective` |
| `day` | string | X | 요일 필터: `mon`~`fri` |
| `department` | string | X | 학부/학과 필터 |
| `page` | integer | X | 페이지 번호 (기본값: 1) |
| `size` | integer | X | 페이지 크기 (기본값: 20, 최대: 50) |

**요청 예시**

```
GET /v1/courses/search?keyword=알고리즘&type=major_required&page=1&size=20
Authorization: Bearer {token}
```

### Response

**Status 200 OK**

```json
{
  "success": true,
  "data": {
    "total": 2,
    "page": 1,
    "size": 20,
    "items": [
      {
        "course_id": "CSE2001",
        "name": "알고리즘",
        "professor": "정하준",
        "credits": 3,
        "type": "major_required",
        "type_label": "전공필수",
        "department": "컴퓨터학부",
        "schedule": [
          {
            "day": "tue",
            "start_period": 2,
            "end_period": 3,
            "start_time": "10:00",
            "end_time": "11:50"
          },
          {
            "day": "thu",
            "start_period": 2,
            "end_period": 3,
            "start_time": "10:00",
            "end_time": "11:50"
          }
        ],
        "room": "정보과학관 201",
        "current_enrollment": 45,
        "max_enrollment": 60
      }
    ]
  }
}
```

---

## 공통 에러 응답

모든 API는 아래 형식으로 에러를 반환합니다.

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "학년은 1~4 사이의 값이어야 합니다.",
    "field": "grade"
  }
}
```

### 에러 코드 목록

| HTTP Status | 에러 코드 | 설명 |
|-------------|-----------|------|
| 400 | `VALIDATION_ERROR` | 요청 파라미터 유효성 검증 실패 |
| 401 | `UNAUTHORIZED` | 인증 토큰이 없거나 만료됨 |
| 403 | `FORBIDDEN` | 해당 리소스에 대한 접근 권한 없음 |
| 404 | `NOT_FOUND` | 요청한 리소스를 찾을 수 없음 |
| 409 | `CONFLICT` | 중복 저장 등 충돌 발생 |
| 429 | `RATE_LIMITED` | 요청 횟수 제한 초과 |
| 500 | `INTERNAL_ERROR` | 서버 내부 오류 |

---

## 교시 시간표

| 교시 | 시작 | 종료 |
|------|------|------|
| 1 | 09:00 | 09:50 |
| 2 | 10:00 | 10:50 |
| 3 | 11:00 | 11:50 |
| 4 | 12:00 | 12:50 |
| 5 | 13:00 | 13:50 |
| 6 | 14:00 | 14:50 |
| 7 | 15:00 | 15:50 |
| 8 | 16:00 | 16:50 |
| 9 | 17:00 | 17:50 |

---

## Enum 정의

### 입학 유형 (`admission_type`)
- `normal` — 일반입학
- `transfer` — 편입생

### 과목 유형 (`type`)
- `major_required` — 전공필수
- `major_elective` — 전공선택
- `general_required` — 교양필수
- `general_elective` — 교양선택

### 시간 선호 (`time_preference`)
- `morning` — 오전 집중형 (오후 3시 이전 마무리)
- `afternoon` — 오후 집중형 (오전 11시 이후 시작)
- `any` — 상관없음

### 계획 기간 (`plan_period`)
- `single` — 이번 학기만
- `year` — 1년 계획

### 요일 (`day`)
- `mon`, `tue`, `wed`, `thu`, `fri`

### 이수 상태 (`status`)
- `done` — 이수 완료
- `in_progress` / `taking` — 수강 중
- `not_taken` — 미이수

### 강의실 상태
- `free` — 사용 가능
- `occupied` — 사용 중

### 교양 관심 분야 (`interests`)
- `music` — 음악·예술
- `sports` — 스포츠·체육
- `language` — 언어·외국어
- `it` — IT·정보기술
- `humanities` — 인문학·사회
- `science` — 과학·환경
- `business` — 경제·경영
- `design` — 창작·디자인
