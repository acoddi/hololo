# 코딩 컨벤션

## 1. 기본 원칙

- 읽기 쉬운 코드가 우선이다.
- 한 클래스와 한 파일은 하나의 핵심 책임을 가진다.
- UI, 상태 관리, 장비 통신, 저장 로직을 섞지 않는다.
- 중복보다 명확한 책임 분리가 더 중요하다.
- 규칙은 문서와 실제 코드에서 함께 유지한다.

## 2. 디렉터리 및 파일 배치 규칙

- QML 화면은 `qml/pages/` 아래에 둔다.
- 재사용 가능한 QML 컴포넌트는 `qml/components/` 아래에 둔다.
- 데이터 구조는 `src/models/`에 둔다.
- 비즈니스 처리 로직은 `src/services/`에 둔다.
- 화면 상태 중계 계층은 `src/viewmodels/`에 둔다.
- 공통 상수와 순수 유틸리티는 `src/utils/`에 둔다.

규칙:

- 하나의 주요 클래스는 하나의 헤더와 하나의 소스 파일로 분리한다.
- 파일명과 클래스명은 동일하게 맞춘다.
- ViewModel과 Service는 폴더 이름만 봐도 역할이 드러나야 한다.

## 3. C++ 네이밍 규칙

### 3.1 타입 이름

- 클래스, 구조체, enum, typedef는 `PascalCase`
- 예: `SensorData`, `DeviceService`, `ThresholdConfig`

### 3.2 함수 이름

- 함수와 메서드는 `camelCase`
- 예: `openPort()`, `startPump()`, `parseSensorPacket()`

### 3.3 멤버 변수

- 일반 멤버 변수는 `m_` 접두사 + `camelCase`
- 예: `m_serialPort`, `m_isConnected`, `m_warningMessage`

### 3.4 지역 변수와 인자

- 지역 변수와 함수 인자는 `camelCase`
- 예: `portName`, `baudRate`, `sensorData`

### 3.5 상수

- `constexpr` 또는 `const` 상수는 `kPascalCase`
- 예: `kDefaultBaudRate`, `kSoilMoistureWarningLevel`

### 3.6 enum 값

- enum 타입은 `PascalCase`
- enum 항목도 `PascalCase`
- 예: `ConnectionState::Connected`

## 4. Qt / QObject 규칙

- `QObject` 기반 클래스는 필요할 때만 사용한다.
- 소유권이 명확하면 Qt 부모-자식 관계를 사용한다.
- 소유권이 불명확한 raw pointer 남용을 피한다.
- `Q_PROPERTY`는 UI에 실제로 필요한 상태만 노출한다.
- signal 이름은 상태 변화 의미가 드러나야 한다.

예:

- `temperatureChanged()`
- `connectionStateChanged()`
- `warningMessageChanged()`

권장:

- signal은 "무슨 일이 바뀌었는지"를 표현한다.
- slot은 외부 입력을 처리하는 진입점으로 사용한다.

## 5. 헤더 / 소스 구성 규칙

### 5.1 헤더 파일

- 가능하면 `#pragma once`를 사용한다.
- 공개 API만 헤더에 둔다.
- 불필요한 include 대신 전방 선언을 우선 고려한다.

### 5.2 include 순서

1. 자기 자신의 헤더
2. Qt 헤더
3. C++ 표준 라이브러리 헤더
4. 프로젝트 내부 헤더

### 5.3 구현 파일

- 함수 구현 순서는 헤더 선언 순서와 최대한 맞춘다.
- 한 함수가 너무 길어지면 private helper로 분리한다.
- 설명이 필요한 복잡한 분기만 짧게 주석을 단다.

## 6. QML 규칙

### 6.1 파일 및 컴포넌트 이름

- QML 파일명은 `PascalCase`
- 예: `DashboardPage.qml`, `SensorCard.qml`

### 6.2 id 및 속성 이름

- `id`는 `camelCase`
- QML 속성 이름도 `camelCase`
- 예: `dashboardPage`, `sensorCard`, `warningText`

### 6.3 QML 책임 범위

QML이 해야 하는 일:

- 레이아웃
- 스타일링
- 사용자 입력 처리
- ViewModel 프로퍼티 표시

QML이 하지 말아야 하는 일:

- 시리얼 패킷 해석
- 데이터베이스 직접 접근
- 복잡한 자동 제어 로직 작성
- 서비스 계층 직접 생성 및 관리

원칙:

- QML은 ViewModel만 바라보도록 설계한다.
- QML 내부 JavaScript는 간단한 표현 처리 정도로 제한한다.

## 7. ViewModel 규칙

- ViewModel은 화면 상태의 단일 창구 역할을 한다.
- QML에 필요한 값만 `Q_PROPERTY`로 노출한다.
- 사용자 입력은 명확한 메서드 이름으로 서비스에 전달한다.
- ViewModel 안에 시리얼 프로토콜 문자열 생성 로직을 넣지 않는다.
- ViewModel 안에 SQL 쿼리를 작성하지 않는다.

예:

- `startPump()`
- `stopPump()`
- `setAutoMode(bool enabled)`
- `refreshLogs()`

## 8. Service 규칙

- 서비스는 실제 처리 로직을 담당한다.
- 서비스는 UI 의존성이 없어야 한다.
- 서로 다른 책임을 가진 서비스는 분리한다.
- 프로토콜 파싱과 명령 생성 책임은 분리한다.

권장 책임 분리:

- `SerialManager`: 입출력 전담
- `SensorDataParser`: 파싱 전담
- `DeviceService`: 제어 명령 전담
- `AutomationService`: 자동 판단 전담
- `LogService`: 로그 기록 전담
- `DatabaseManager`: DB 접근 전담

## 9. 통신 프로토콜 작성 규칙

- 패킷 타입은 첫 토큰으로 식별한다.
- 명령 키워드는 대문자를 사용한다.
- 값은 가능한 한 `KEY=VALUE` 형식을 유지한다.
- 프로토콜 상수는 코드 여러 곳에 하드코딩하지 않는다.
- 잘못된 패킷은 무시하지 말고 로그를 남긴다.

예:

- `SENSOR,TEMP=24.1,HUM=51,SOIL=32`
- `CMD,PUMP,ON`
- `ACK,MODE,AUTO`

## 10. 로그 및 에러 처리 규칙

- 사용자에게 보여줄 메시지와 개발자 로그를 구분한다.
- 통신 실패, 파싱 실패, DB 실패는 각각 구분해서 기록한다.
- recover 가능한 오류는 경고로 처리하고, 복구 불가 오류만 치명 오류로 취급한다.

권장 로그 예시:

- `qInfo()` : 정상 상태 변화
- `qWarning()` : 복구 가능한 문제
- `qCritical()` : 치명적인 실패

## 11. 데이터 및 DB 네이밍 규칙

- C++ 모델 이름은 `PascalCase`
- DB 테이블과 컬럼 이름은 `snake_case`
- 설정 key 값은 의미가 분명한 문자열을 사용한다.

예:

- 모델: `SensorData`
- 테이블: `sensor_logs`
- 컬럼: `soil_moisture`
- 설정 키: `auto_watering_threshold`

## 12. 금지 사항

- QML에서 시리얼 포트를 직접 제어하지 않는다.
- ViewModel에서 DB 쿼리를 직접 실행하지 않는다.
- Service가 QML 타입을 직접 참조하지 않는다.
- 문자열 프로토콜을 여러 클래스에서 중복 조립하지 않는다.
- 센서값, 모드값, 상태값을 매직 넘버와 매직 문자열로 남발하지 않는다.
