# 아키텍처 설계

## 1. 설계 원칙

이 프로젝트의 아키텍처는 아래 원칙을 따른다.

- 하드웨어 제어와 PC 관제 로직을 분리한다.
- UI와 비즈니스 로직을 분리한다.
- 데이터 수신, 파싱, 상태 관리, 저장을 서로 독립된 책임으로 나눈다.
- PC가 없어도 최소 안전 동작은 STM32에서 가능하도록 한다.
- 이후 통신 방식이나 센서 종류가 바뀌어도 큰 구조를 유지할 수 있어야 한다.

## 2. 시스템 역할 분리

### 2.1 STM32 책임

- 센서 주기 측정
- 펌프, 팬, 릴레이 등 장비 제어
- 기본 안전 조건 처리
- PC 명령 수신
- 현재 상태와 이벤트 전송

STM32는 "현장 제어기" 역할을 맡는다.

### 2.2 PC 애플리케이션 책임

- 시리얼 통신 연결 관리
- 수신 데이터 파싱
- 현재 상태 모델 유지
- 고급 자동 제어 판단
- 로그 저장 및 조회
- 사용자 설정 관리
- 실시간 모니터링 UI 제공

PC는 "관제 및 운영 애플리케이션" 역할을 맡는다.

## 3. PC 애플리케이션 구조

PC 애플리케이션은 `MVVM + Service` 구조를 기본으로 한다.

```text
QML View
   ↓
ViewModel
   ↓
Service / Manager
   ↓
Model / Storage / Serial
```

## 4. 계층별 책임

### 4.1 View

구성:

- `Main.qml`
- `DashboardPage.qml`
- `ControlPage.qml`
- `LogPage.qml`
- `SettingsPage.qml`
- 공통 컴포넌트

역할:

- 화면 배치
- 사용자 입력 수집
- 상태 표시
- 색상, 레이아웃, 시각적 피드백 처리

금지:

- 시리얼 통신 직접 호출
- 데이터 파싱
- SQL 처리
- 자동 제어 판단

QML은 가능한 한 "보여주기"에만 집중한다.

### 4.2 ViewModel

예시:

- `MainViewModel`
- `DashboardViewModel`
- `ControlViewModel`
- `LogViewModel`
- `SettingsViewModel`

역할:

- QML에서 사용할 상태를 `Q_PROPERTY`로 노출
- 사용자의 액션을 서비스 계층으로 전달
- 화면에서 필요한 형식으로 데이터를 가공
- 경고 메시지, 연결 상태, 현재 모드 같은 UI 상태 관리

예시 속성:

- `temperature`
- `humidity`
- `soilMoisture`
- `waterLevel`
- `flowRate`
- `isConnected`
- `warningMessage`

금지:

- 시리얼 패킷 파싱
- SQL 직접 실행
- STM32 프로토콜 문자열 조립

### 4.3 Service / Manager

서비스 계층은 실제 작업을 수행하는 핵심 계층이다.

#### `SerialManager`

- COM 포트 열기/닫기
- 송수신 처리
- 연결 상태 관리
- 수신 이벤트 발생

#### `SensorDataParser`

- 문자열 기반 패킷 파싱
- 센서/상태/응답/오류 패킷 분류
- 구조화된 모델 객체로 변환

#### `DeviceService`

- 펌프, 팬, 모드 변경 명령 생성
- 수동 제어 요청 처리
- 명령 전송 결과를 상태 계층에 반영

#### `AutomationService`

- 임계치 기반 자동 제어 판단
- 현재 센서값과 설정값 비교
- 필요 시 제어 명령 요청

#### `LogService`

- 이벤트 로그 생성
- 상태 변화 기록
- 로그 조회 요청 처리

#### `DatabaseManager`

- SQLite 연결 관리
- 테이블 생성
- 저장/조회 쿼리 수행

원칙:

- 서비스는 UI를 몰라야 한다.
- 서비스는 QML 객체를 직접 참조하지 않는다.
- 각 서비스는 하나의 핵심 책임을 가져야 한다.

### 4.4 Model

모델 계층은 데이터 구조를 정의한다.

예시:

- `SensorData`
- `DeviceStatus`
- `ControlCommand`
- `LogEntry`
- `ThresholdConfig`

원칙:

- 모델은 화면 표현보다 데이터 의미를 우선한다.
- 데이터 구조는 파서와 저장 계층에서 공통으로 사용할 수 있어야 한다.

## 5. 권장 의존성 규칙

아래 방향의 의존성만 허용한다.

- View -> ViewModel
- ViewModel -> Service
- Service -> Model / Manager / Storage

허용하지 않는 방향:

- View -> Service 직접 호출
- View -> SerialManager 직접 접근
- Service -> ViewModel 참조
- DatabaseManager -> QML 참조

즉, 위에서 아래로만 흐르게 만든다.

## 6. 권장 폴더 구조

현재 저장소는 초기 템플릿 구조지만, 구현이 확장되면 아래 구조로 정리하는 것을 권장한다.

```text
project/
├─ CMakeLists.txt
├─ main.cpp
├─ docs/
│  ├─ README.md
│  ├─ project-overview.md
│  ├─ architecture.md
│  └─ coding-convention.md
├─ qml/
│  ├─ Main.qml
│  ├─ pages/
│  │  ├─ DashboardPage.qml
│  │  ├─ ControlPage.qml
│  │  ├─ LogPage.qml
│  │  └─ SettingsPage.qml
│  └─ components/
│     ├─ SensorCard.qml
│     ├─ StatusBadge.qml
│     ├─ ControlButton.qml
│     └─ LogTable.qml
├─ src/
│  ├─ models/
│  │  ├─ SensorData.h
│  │  ├─ DeviceStatus.h
│  │  ├─ ControlCommand.h
│  │  ├─ LogEntry.h
│  │  └─ ThresholdConfig.h
│  ├─ services/
│  │  ├─ SerialManager.h
│  │  ├─ SerialManager.cpp
│  │  ├─ SensorDataParser.h
│  │  ├─ SensorDataParser.cpp
│  │  ├─ DeviceService.h
│  │  ├─ DeviceService.cpp
│  │  ├─ AutomationService.h
│  │  ├─ AutomationService.cpp
│  │  ├─ LogService.h
│  │  ├─ LogService.cpp
│  │  ├─ DatabaseManager.h
│  │  └─ DatabaseManager.cpp
│  ├─ viewmodels/
│  │  ├─ MainViewModel.h
│  │  ├─ MainViewModel.cpp
│  │  ├─ DashboardViewModel.h
│  │  ├─ DashboardViewModel.cpp
│  │  ├─ ControlViewModel.h
│  │  └─ ControlViewModel.cpp
│  └─ utils/
│     └─ constants.h
└─ resources/
```

## 7. 대표 런타임 흐름

### 7.1 센서 업데이트

1. `SerialManager`가 UART 데이터를 수신한다.
2. `SensorDataParser`가 패킷 종류를 판별한다.
3. `SensorData` 또는 `DeviceStatus` 모델을 생성한다.
4. 관련 서비스와 ViewModel이 상태를 갱신한다.
5. QML UI가 변경된 속성을 자동 반영한다.
6. `LogService`가 필요 시 데이터 또는 이벤트를 저장한다.

### 7.2 수동 제어

1. 사용자가 QML 버튼을 클릭한다.
2. ViewModel이 `DeviceService`에 제어 요청을 전달한다.
3. `DeviceService`가 프로토콜 문자열을 생성한다.
4. `SerialManager`가 STM32로 전송한다.
5. STM32가 장비를 제어하고 `ACK` 또는 `ERR`를 전송한다.
6. 결과가 다시 ViewModel과 UI에 반영된다.

### 7.3 자동 제어

1. 최신 센서값이 ViewModel 또는 상태 저장소에 반영된다.
2. `AutomationService`가 임계치와 현재 상태를 비교한다.
3. 제어가 필요하면 `DeviceService`에 명령 요청을 보낸다.
4. 명령 전송 후 결과를 로그와 UI에 반영한다.

## 8. 상태 관리 원칙

PC 애플리케이션은 최소한 아래 상태를 명확히 관리해야 한다.

- 연결 상태
- 현재 동작 모드
- 센서 최신값
- 장비 동작 상태
- 경고 상태
- 최근 명령 결과

권장 방식:

- 화면에 필요한 상태는 ViewModel에서 단일 진입점으로 노출한다.
- 원시 수신 문자열 자체를 UI가 직접 다루지 않는다.
- 경고 상태는 단순 문자열 하나보다 코드와 메시지를 함께 관리하는 것이 좋다.

## 9. 확장 포인트

향후 아래 기능을 붙이기 쉬운 구조를 목표로 한다.

- Wi-Fi 또는 BLE 통신 추가
- MQTT 기반 원격 모니터링
- 센서 종류 추가
- 다중 화분 또는 다중 채널 지원
- 임계치 프로필 저장
- 통계 그래프 및 이력 분석

핵심은 통신 방식과 화면이 바뀌더라도 서비스와 모델의 책임이 크게 흔들리지 않도록 만드는 것이다.
