# SD — Sequence Diagram

## AI Companion Robot in Apartment

팀이 제안한 로봇의 역할을 바탕으로 주요 서비스의 메시지 전달 순서를 정리한 초기 시퀀스 다이어그램입니다. 내부 서버·DB·API 구조는 정하지 않고 **사용자, 로봇, 플랫폼, 관리사무소 및 외부 설비**의 상호작용에 집중합니다.

- 실선 화살표는 요청·전달, 점선 화살표는 응답·결과를 나타냅니다.
- `alt / else`는 조건에 따른 분기입니다.
- 거실 컴패니언 로봇·월패드는 세대 알림 접점이며, 실제 구현·연동 방식은 미정입니다.
- 인증 확인, 미응답 처리, 위험 상황 분기는 흐름을 명확히 하기 위한 설계 제안입니다.
- 아래 흐름은 구현 완료나 실제 장비 연동을 의미하지 않습니다. 출입·엘리베이터·보관함 연동 가능 여부는 별도 확인이 필요합니다.

## SD-01. 민원 접수 및 처리 현황 조회

**목적:** 입주민의 민원을 관리사무소에 전달하고 처리 결과를 안내합니다.

```mermaid
sequenceDiagram
    autonumber
    actor Resident as 입주민
    participant Robot as 행정 지원 로봇
    participant Platform as 플랫폼
    actor Staff as 관리사무소 직원

    Resident->>Robot: 민원 내용 입력
    Robot->>Platform: 본인·세대 확인 요청
    Platform-->>Robot: 확인 결과
    alt 확인 성공
        Robot->>Platform: 민원 요약·유형 제안 요청
        Platform-->>Robot: 정리된 민원 내용
        Robot-->>Resident: 접수할 내용 확인 요청
        Resident->>Robot: 내용 확인 및 제출
        Robot->>Platform: 민원 접수 요청
        Platform-->>Robot: 접수 번호 및 상태
        Robot-->>Resident: 접수 완료 안내
        Platform->>Staff: 민원 전달
        Staff->>Platform: 담당 배정 및 처리 상태 등록
        Resident->>Robot: 민원 처리 현황 조회
        Robot->>Platform: 본인 민원 조회 요청
        Platform-->>Robot: 조회 권한 확인 후 상태·결과 반환
        Robot-->>Resident: 처리 현황 안내
    else 확인 실패
        Robot-->>Resident: 재확인 또는 직원 연결 안내
    end
```

**보완 사항:** 동·호수 입력만으로 본인 확인을 완료하지 않습니다. 확인 방식은 추후 정합니다. 민원 접수는 저장 성공 후 완료로 안내하며, 질문 요약을 할 수 없는 경우 입력 내용을 직접 확인받아 접수할 수 있습니다. 공개 공지·행정 안내는 개인 민원 조회와 구분합니다.

## SD-02. 안심 귀가 동행

**목적:** 입주민의 귀가 요청을 순찰 로봇에 연결하고 동행 결과를 관리합니다.

```mermaid
sequenceDiagram
    autonumber
    actor Resident as 입주민
    participant Platform as 플랫폼
    participant Patrol as 안심 귀가·순찰 로봇
    actor Staff as 관리·안전 담당자

    Resident->>Platform: 귀가 동행 요청 및 만남 위치 전달
    Platform->>Platform: 요청 권한 및 서비스 가능 구역 확인
    Platform->>Patrol: 동행 가능 여부 확인
    Patrol-->>Platform: 현재 위치·상태 및 가능 여부
    alt 동행 가능
        Platform->>Patrol: 동행 요청 배정
        Platform-->>Resident: 만남 위치 및 배정 결과 안내
        Patrol-->>Resident: 도착 및 동행 시작 안내
        loop 동행 중
            Patrol->>Platform: 위치·진행 상태 보고
        end
        alt 이상 없이 동행 종료
            Patrol->>Platform: 동행 완료 보고
            Platform-->>Resident: 완료 안내
        else 위험 또는 운행 문제 발생
            Patrol->>Platform: 이상 상황 및 위치 보고
            Platform->>Staff: 대응 요청
            Patrol-->>Resident: 상황 및 안전 안내
        end
    else 동행 불가
        Platform-->>Resident: 이용 불가 또는 대기 안내
    end
```

**보완 사항:** 호출 수단과 동행 종료 지점은 미정입니다. 실제 운행에서는 로컬 안전 기능이 우선하며, 플랫폼 응답을 기다리느라 안전 정지를 지연하지 않습니다. 세대 내부 진입과 침입자 직접 제압은 하지 않습니다.

## SD-03. 방문객 도착 및 출입 승인

**목적:** 방문객의 도착을 세대에 알리고 입주민의 승인 결과를 출입 시스템에 전달합니다.

```mermaid
sequenceDiagram
    autonumber
    actor Visitor as 방문객
    participant Lobby as 현관·로비 안내 로봇
    participant Platform as 플랫폼
    participant Home as 세대 컴패니언·월패드
    actor Resident as 입주민
    participant Door as 출입 시스템

    Visitor->>Lobby: 방문 세대 및 목적 전달
    Lobby->>Platform: 세대 호출 요청
    Platform->>Home: 방문객 도착 및 승인 요청 전달
    Home-->>Resident: 방문 안내
    alt 승인권 있는 입주민이 유효 시간 내 승인
        Resident->>Home: 출입 승인
        Home->>Platform: 승인 결과 전달
        Platform->>Platform: 승인자 권한·대상 방문·유효성 확인
        Platform->>Door: 검증된 승인에 따른 문 개방 요청
        Door-->>Platform: 개방 처리 결과
        Platform-->>Lobby: 처리 결과 전달
        Lobby-->>Visitor: 개방 성공 또는 실패 안내
    else 거절·미응답·승인 검증 실패
        Platform-->>Lobby: 승인 미완료 안내
        Lobby-->>Visitor: 출입 불가 또는 경비 문의 안내
    end
```

**보완 사항:** 음성 응답만으로 승인자의 권한을 확정하지 않습니다. 승인 없는 문 개방은 하지 않으며, 승인 접수와 문 개방 성공을 구분합니다. 출입 시스템의 응답이 없으면 개방 여부 확인이 필요하다고 안내합니다. 로비 로봇이 방문객 신원을 보증하거나 독자적으로 문을 여는 흐름이 아닙니다.

## SD-04. 순찰 감지와 청소 로봇 협업

**목적:** 순찰 로봇이 발견한 일반 오염을 청소 작업으로 연결하고, 위험 상황은 사람에게 전달합니다.

```mermaid
sequenceDiagram
    autonumber
    participant Patrol as 순찰 로봇
    participant Platform as 플랫폼
    participant Cleaner as 바닥 청소 로봇
    actor Staff as 관리·안전 담당자

    Patrol->>Platform: 이상 징후 유형 및 위치 보고
    Platform->>Platform: 청소 가능 오염과 위험·불명 상황 구분
    alt 청소 가능한 일반 오염
        Platform->>Cleaner: 해당 구역 작업 가능 여부 확인
        Cleaner-->>Platform: 상태 및 작업 가능 여부
        alt 작업 가능
            Platform->>Cleaner: 청소 작업 요청
            Cleaner-->>Platform: 작업 수락 및 시작 보고
            alt 청소 완료
                Cleaner->>Platform: 청소 완료 보고
                Platform-->>Staff: 완료 결과 전달
            else 장애물·고장 등 작업 중단
                Cleaner->>Platform: 중단 사유 보고
                Platform->>Staff: 현장 확인 요청
            end
        else 작업 불가
            Platform->>Staff: 미처리 오염 및 확인 요청
        end
    else 화재·침입·불명 액체 등 위험 또는 판단 불가
        Platform->>Staff: 위치·상황 전달 및 대응 요청
        Note over Platform,Cleaner: 청소 로봇 출동으로 자동 연결하지 않음
    end
```

**보완 사항:** 사건 분류 기준과 사람의 확인이 필요한 범위는 추후 정합니다. 쏟아진 물도 해당 로봇이 처리할 수 있는지 확인해야 합니다. 입주민 안전 알림은 필요한 대상과 전달 기준을 정한 후 세대 접점으로 전달합니다.

## SD-05. 주차 문제 감지 및 차량 이동 알림

**목적:** 차량 이동이 필요한 상황을 해당 차량과 연결된 세대에 알립니다.

```mermaid
sequenceDiagram
    autonumber
    participant Parking as 주차 관리 로봇
    participant Platform as 플랫폼
    participant Home as 세대 컴패니언·월패드
    actor Resident as 입주민
    actor Staff as 관리사무소 직원

    Parking->>Platform: 이중주차 등 의심 상황 보고
    Note over Parking,Platform: 차량번호·위치·발견 시각 전달
    Platform->>Platform: 등록 차량과 연결된 세대 확인
    alt 알림 대상 확인 가능
        Platform->>Home: 해당 세대에 차량 이동 알림 요청
        Home-->>Resident: 차량 위치 및 이동 요청 안내
        Resident->>Home: 알림 확인
        Home->>Platform: 확인 결과 전달
        Note over Platform,Resident: 알림 확인은 차량 이동 완료와 다름
    else 대상 확인 불가 또는 오인식 의심
        Platform->>Staff: 차량·위치 확인 요청
    end
```

**보완 사항:** 실제 차량 이동 여부를 확인하는 방식과 재알림 기준은 미정입니다. 이 흐름은 감지·알림에 한정하며, 견인·과태료·결제는 포함하지 않습니다. 빈자리 안내와 전기차 충전 지원은 세부 이용 흐름이 정해진 후 추가합니다.

## SD-06. 택배 운반 및 수령

**목적:** 택배를 세대 문 앞까지 운반하고 수령 또는 부재에 따른 처리 결과를 알립니다.

```mermaid
sequenceDiagram
    autonumber
    actor Requester as 입주민·택배 기사
    participant Platform as 플랫폼
    participant Delivery as 짐꾼·택배 운반 로봇
    participant Elevator as 엘리베이터
    participant Home as 세대 컴패니언·월패드
    actor Resident as 수령 입주민

    Requester->>Platform: 운반 요청 및 수령 세대 전달
    Platform->>Delivery: 배송 요청·경로·수령 방식 전달
    Delivery-->>Platform: 작업 수락 및 운반 시작
    opt 층간 이동 필요
        Delivery->>Platform: 엘리베이터 사용 요청
        Platform->>Elevator: 사용 허가 확인 및 호출 요청
        Elevator-->>Platform: 사용 가능 여부
        Platform-->>Delivery: 탑승 가능 또는 대기 안내
    end
    alt 경로 확보 및 문 앞 도착
        Delivery->>Platform: 도착 보고
        Platform->>Home: 배송 도착 알림 요청
        Home-->>Resident: 택배 도착 안내
        alt 입주민 수령
            Resident->>Delivery: 물품 수령 및 수령 확인
            Delivery->>Platform: 수령 확인·배송 완료 보고
            Platform-->>Requester: 배송 완료 안내
        else 부재 또는 수령 불가
            Delivery->>Platform: 수령 실패 보고
            Platform->>Delivery: 정해진 보관 장소로 복귀 요청
            Delivery->>Platform: 보관 성공 또는 실패 보고
            Platform->>Home: 보관 위치·재배송 안내 또는 확인 필요 알림
        end
    else 이동 불가 또는 운반 중단
        Delivery->>Platform: 실패 사유 및 현재 위치 보고
        Platform-->>Requester: 운반 중단 및 담당자 확인 안내
    end
```

**보완 사항:** 요청자 권한, 엘리베이터 연동, 적재·수령 확인 방법, 부재 시 보관 장소는 추후 정합니다. 실제 수령·보관이 확인되기 전에는 완료로 표시하지 않습니다. 물품 내용물 대신 운송에 필요한 최소 정보만 사용하며 세대 내부에는 진입하지 않습니다.
