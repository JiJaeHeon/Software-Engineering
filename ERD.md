# ERD 초안 - 아파트 로봇 정보 공유 플랫폼

> 기준: 로봇 7종 md의 "플랫폼에서 받는 것 / 주는 것". 초안이며 CD 클래스명과 맞춰 수정한다.
> 🔒 = 개인정보·민감정보 컬럼

```mermaid
erDiagram
    COMPLEX ||--o{ HOUSEHOLD : has
    HOUSEHOLD ||--o{ RESIDENT : lives
    HOUSEHOLD ||--o{ VEHICLE : owns
    HOUSEHOLD ||--o{ PARCEL : receives
    HOUSEHOLD ||--o{ COMPLAINT : files
    HOUSEHOLD ||--o{ VISIT : visited
    VISITOR ||--o{ VISIT : makes
    RESIDENT ||--o{ CONVERSATION : talks
    ROBOT ||--o{ CONVERSATION : joins
    ROBOT ||--o{ TASK : performs
    ROBOT ||--o{ EVENT : emits
    ROBOT ||--o{ MESSAGE : sends
    ROBOT ||--o{ MESSAGE : receives
    ADMIN ||--o{ TASK : assigns
    ADMIN ||--o{ NOTICE : posts
    ADMIN ||--o{ COMPLAINT : handles
    COMPLAINT ||--o{ TASK : triggers
    EVENT ||--o| COMPLAINT : may_become
    ROBOT_TYPE ||--o{ ROBOT : classifies
    AREA ||--o{ ROBOT : assigned
    CONSENT }o--|| RESIDENT : by

    COMPLEX { int complex_id PK
              string name }
    HOUSEHOLD { int household_id PK
                int complex_id FK
                string building
                string unit }
    RESIDENT { int resident_id PK
               int household_id FK
               string name "🔒"
               string phone "🔒"
               string voice_profile "🔒" }
    CONSENT { int consent_id PK
              int resident_id FK
              string item "카메라/마이크/위치"
              bool agreed
              datetime updated_at }
    VISITOR { int visitor_id PK
              string name "🔒"
              string phone "🔒" }
    VISIT { int visit_id PK
            int visitor_id FK
            int household_id FK
            datetime arrived_at }
    ROBOT_TYPE { int type_id PK
                 string type_name "7종" }
    AREA { int area_id PK
           string name "로비/주차장/복도 등" }
    ROBOT { int robot_id PK
            int type_id FK
            int area_id FK
            string status
            string location }
    TASK { int task_id PK
           int robot_id FK
           int admin_id FK
           int complaint_id FK
           string task_type
           string state
           datetime created_at
           datetime done_at }
    EVENT { int event_id PK
            int robot_id FK
            string event_type
            string detail
            datetime occurred_at }
    MESSAGE { int message_id PK
              int from_robot_id FK
              int to_robot_id FK
              string content
              datetime sent_at }
    CONVERSATION { int conv_id PK
                   int robot_id FK
                   int resident_id FK
                   string content "🔒"
                   datetime talked_at
                   date delete_after }
    COMPLAINT { int complaint_id PK
                int household_id FK
                int admin_id FK
                int event_id FK
                string category
                string content
                string state
                datetime received_at }
    NOTICE { int notice_id PK
             int admin_id FK
             string title
             string body
             datetime posted_at }
    VEHICLE { int vehicle_id PK
              int household_id FK
              string plate_no "🔒" }
    PARCEL { int parcel_id PK
             int household_id FK
             string state
             datetime arrived_at }
    ADMIN { int admin_id PK
            string name
            string role }
```

## 설계 메모
- 로봇 7종은 ROBOT 하나에 ROBOT_TYPE으로 구분한다(공통 구조).
- 로봇 간 협업은 MESSAGE, 센서/이상 감지는 EVENT, 관리사무소 지시는 TASK에 저장한다.
- 이벤트가 민원이 될 수 있어 EVENT → COMPLAINT → TASK로 이어진다.
- CONVERSATION의 delete_after로 보관·삭제 정책을 반영한다.
- CONSENT로 카메라·마이크·위치 동의를 관리한다.

## 점검 체크리스트
- [ ] 로봇 md 7개의 "받는 것/주는 것"이 모두 엔티티·컬럼에 있는가
- [ ] CD 클래스명과 엔티티명이 같은가
- [ ] 🔒 컬럼에 접근 권한 정책이 SRS에 있는가
