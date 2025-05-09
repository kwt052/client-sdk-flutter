# LiveKit Flutter Example

This app implements a video room using LiveKit's Flutter SDK. Designed to run for iOS, Android, Web, Mac, and Windows.

## Quickstart

Run example:

```bash
flutter pub get
# Due to the inconvenience of typing on mobile devices, 
# you can autofill URL and TOKEN for first run in debug mode.
flutter run --dart-define=URL=wss://${LIVEKIT_SERVER_IP_OR_DOMAIN} --dart-define=TOKEN=${YOUR_TOKEN}
```

## End-to-End Encryption (E2EE)

The example app supports end-to-end encryption for audio and video tracks. To enable E2EE:

1. Toggle the "E2EE" switch in the connect screen
2. Enter a shared key that will be used for encryption
3. All participants must use the same shared key to communicate

For web support, you'll need to compile the E2EE web worker:

```bash
dart compile js web/e2ee.worker.dart -o example/web/e2ee.worker.dart.js -m
```

Note: All participants in the room must have E2EE enabled and use the same shared key to see and hear each other. If the keys don't match, participants won't be able to decode each other's audio and video.

## 애플리케이션 구조 및 기능

이 예제 애플리케이션은 LiveKit Flutter SDK를 사용하여 실시간 오디오/비디오 통신 기능을 시연합니다. 주요 구성 요소와 기능은 다음과 같습니다.

### 1. 연결 설정 (`lib/pages/connect.dart`)

*   **역할**: 사용자가 LiveKit 서버에 연결하기 위한 정보를 입력하고 관련 설정을 구성하는 진입점입니다.
*   **주요 기능**:
    *   LiveKit 서버 URL 및 인증 토큰 입력 필드 제공.
    *   E2EE (End-to-End Encryption) 사용 여부 및 공유 키 입력 기능.
    *   Simulcast, Adaptive Stream, Dynacast 등 고급 미디어 전송 옵션 설정.
    *   입력된 정보 및 설정을 로컬에 저장하고 다음 실행 시 자동으로 불러오는 기능 (`shared_preferences` 사용).
    *   필요한 경우 카메라, 마이크 등의 권한 요청.
    *   "Connect" 버튼을 통해 `PreJoinPage`로 이동하며 입력된 연결 정보를 전달합니다.

### 2. 통화방 입장 준비 (`lib/pages/prejoin.dart`)

*   **역할**: 실제 통화방에 참여하기 전에 사용자가 로컬 미디어 장치를 선택하고 미리보기를 통해 설정을 확인하는 화면입니다.
*   **주요 기능**:
    *   사용 가능한 오디오 입력(마이크) 및 비디오 입력(카메라) 장치 목록 표시 및 선택 기능.
    *   선택된 카메라의 실시간 미리보기 제공.
    *   오디오 및 비디오 활성화/비활성화 토글.
    *   `RoomOptions` (E2EE, Simulcast 등 포함)을 구성하고 `Room` 객체 생성.
    *   `room.connect()`를 호출하여 LiveKit 서버에 연결하고 통화방에 참여합니다. 이 과정에서 미리 생성된 로컬 오디오/비디오 트랙을 전달하여 빠른 미디어 발행을 지원합니다 (`FastConnectOptions`).
    *   연결 성공 시, 생성된 `Room` 객체와 이벤트 리스너(`EventsListener`)를 `RoomPage`로 전달합니다.

### 3. 실시간 통화방 (`lib/pages/room.dart`)

*   **역할**: 실제 오디오/비디오 통화가 이루어지는 메인 화면입니다. 참가자들의 미디어를 표시하고 다양한 통화 제어 기능을 제공합니다.
*   **주요 기능**:
    *   **참가자 관리 및 표시**:
        *   로컬 및 원격 참가자의 비디오 스트림과 화면 공유 스트림을 `ParticipantWidget`을 통해 화면에 렌더링합니다.
        *   참가자 정보(이름, 음소거 상태, 현재 발언자 표시 등)를 함께 보여줍니다.
        *   참가자 목록을 동적으로 정렬하여 (예: 현재 발언자 우선) 사용자 경험을 최적화합니다.
    *   **LiveKit 이벤트 처리**:
        *   `EventsListener`를 사용하여 방에서 발생하는 다양한 이벤트(참가자 입장/퇴장, 트랙 발행/구독 변경, 데이터 수신 등)를 실시간으로 감지하고 UI를 동기화합니다.
    *   **미디어 제어 (`ControlsWidget`)**:
        *   로컬 사용자가 자신의 마이크/카메라 켜고 끄기, 카메라 전환, 화면 공유 시작/중지 등의 기능을 제어할 수 있는 UI를 제공합니다.
        *   통화 종료 기능을 제공합니다.
    *   **데이터 채널**: `DataReceivedEvent`를 통해 텍스트 기반 데이터를 주고받을 수 있는 기능을 시연합니다 (예제에서는 수신 데이터를 다이얼로그로 표시).
    *   **UI 구성**: 일반적으로 주 발언자 또는 화면 공유를 크게 표시하고, 다른 참가자들은 작은 썸네일 형태로 나열하는 레이아웃을 사용합니다.

### 4. 공용 위젯 (`lib/widgets/`)

*   **`participant.dart` (`ParticipantWidget`)**: 각 참가자의 비디오/화면 공유 스트림을 렌더링하고 관련 정보를 표시하는 핵심 위젯입니다.
*   **`controls.dart` (`ControlsWidget`)**: 통화 중 사용되는 미디어 제어 버튼들을 모아놓은 위젯입니다.
*   **기타 위젯**: `participant_info.dart` (참가자 정보 표시), `participant_stats.dart` (미디어 통계), `no_video.dart` (비디오 꺼짐 상태 표시), `text_field.dart` (공용 텍스트 입력 필드) 등 앱 전반에서 재사용되는 다양한 UI 컴포넌트들이 포함되어 있습니다.

이러한 구성 요소들의 유기적인 상호작용을 통해, 본 예제 앱은 LiveKit Flutter SDK의 강력한 기능을 활용하여 다양한 플랫폼에서 동작하는 실시간 오디오/비디오 통신 애플리케이션을 효과적으로 구현하는 방법을 보여줍니다.
