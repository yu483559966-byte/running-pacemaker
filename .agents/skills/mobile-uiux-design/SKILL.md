---
name: mobile-uiux-design
description: >-
  프리미엄 모바일 웹 및 웹 앱을 위한 모던 UI/UX 디자인 시스템과 인터랙션 구현 가이드.
  고대비 다크모드(OLED 블랙&화이트), 스포츠/헬스케어 대시보드, 타이포그래피 조합,
  음성/사운드 피드백, 실시간 지도, 모바일 오터치 방지 및 시각화 디자인 요청 시 사용합니다.
---

# 📱 모던 모바일 UI/UX 디자인 스킬 가이드 (Mobile UI/UX Design System)

이 스킬은 단순한 화면을 **나이키(Nike), 스트라바(Strava), 애플 피트니스(Apple Fitness) 수준의 프리미엄 하이엔드 모바일 UI/UX**로 구축하고 고도화하기 위한 핵심 디자인 원칙, 타이포그래피 규칙, 마이크로 인터랙션 및 프론트엔드 코드 패턴을 제공합니다.

---

## 1. 비주얼 테마 & 컬러 팔레트 (Visual Theme & Palette)

야외 강한 햇빛 아래에서도 극도의 가독성을 유지하며, 스마트폰 OLED 디스플레이의 배터리를 극대화하는 **고대비 모노크롬(High-Contrast Pure Dark & White)** 시스템을 기본으로 합니다.

### CSS 토큰 정의
```css
:root {
  /* 배경 & 레이어 */
  --bg: #000000;              /* 순수 블랙 (OLED 친화적) */
  --card-bg: #0c0c0c;         /* 1차 카드 컨테이너 */
  --card-inner: #151515;      /* 내부 입력폼 및 스핀 박스 */
  --card-border: #1e1e1e;     /* 절제된 1px 외곽선 */
  --card-border-active: #383838;

  /* 텍스트 & 계층 구조 */
  --fg: #ffffff;              /* 1차 주요 텍스트 (순백색) */
  --fg-muted: #8e8e93;        /* 2차 서브 텍스트 및 라벨 */
  --fg-dim: #48484a;          /* 비활성화 및 배경선 */
  
  /* 라운딩 시스템 */
  --radius-card: 20px;
  --radius-btn: 16px;
  --radius-pill: 999px;
}
```

---

## 2. 타이포그래피 황금 조합 (Typography Pairing)

모바일 화면, 특히 운동이나 이동 중 흔들리는 환경에서는 **숫자가 밀리지 않는 고정폭 폰트**와 **높은 가독성의 한글 폰트**의 조합이 필수적입니다.

### 폰트 임포트
```html
<!-- 한글 본문 최적화 Pretendard + 스포츠 대형 숫자 Barlow Condensed -->
<link rel="stylesheet" as="style" crossorigin href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.min.css" />
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Barlow+Condensed:ital,wght@0,700;0,800;0,900;1,800&family=Chakra+Petch:wght@700;800&display=swap" rel="stylesheet">
```

### 타이포그래피 규칙
1. **일반 UI 및 한글 텍스트**: `'Pretendard', -apple-system, sans-serif`  
   자간 `-0.2px`, 행간 `1.4~1.5`로 설정하여 뭉개짐 없는 선명한 텍스트 렌더링.
2. **지표/숫자/타이머 (Numbers & Metrics)**: `'Barlow Condensed', 'Chakra Petch', monospace`  
   반드시 `font-variant-numeric: tabular-nums;` 속성을 부여하여 1초 단위로 숫자가 바뀔 때 글자가 좌우로 떨리는 현상(Jitter)을 방지.

---

## 3. 인터랙션 및 모션 디자인 (Micro-Interactions)

### A. 시네마틱 인트로 스플래시 (Cinematic Intro)
* 앱 시작 시 레이더 펄스 링(`radarPulse`)과 프로그레스 로딩 바를 통해 시스템 초기화(GPS 탐색, 오디오 엔진 언락 등)를 감각적으로 표현.
* 1.5~1.8초 후 `opacity: 0; transform: scale(1.06);`로 자연스럽게 페이드아웃.

### B. 상태 반전 배너 (Dynamic Contrast Feedback)
* 상태(적정 / 느림 / 빠름 / 경고)에 따라 배너 테마를 극적으로 반전:
  * **정상/적정**: 블랙 배경에 2px 화이트 실선 테두리 (`#111111`, `#ffffff`)
  * **빠름/경고**: 화이트 배경에 블랙 텍스트로 고대비 반전 (`#ffffff`, `#000000`)
  * **느림/주의**: 블랙 배경에 점선 화이트 테두리 (`border: 2px dashed #ffffff`)
* 음성이 말하고 있을 때 배너 내부에 **음성 이퀄라이저 파형 바(Wave Bar)** 애니메이션 노출.

---

## 4. 다감각 피드백 설계 (Audio, TTS & Haptics)

모바일 앱은 시각뿐만 아니라 청각과 촉각을 통해 사용자에게 즉각적인 피드백을 주어야 합니다.

### A. 브라우저 오디오 자동재생 언락 (Audio Autoplay Policy)
모바일 브라우저는 사용자의 첫 터치 이전에 오디오 출력을 차단하므로, 초기 터치 시 전역 언락을 수행합니다:
```javascript
function unlockAudioContext() {
  if (!audioCtx) {
    const AudioContext = window.AudioContext || window.webkitAudioContext;
    if (AudioContext) audioCtx = new AudioContext();
  }
  if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
  if ('speechSynthesis' in window) window.speechSynthesis.resume();
}
window.addEventListener('click', unlockAudioContext, { once: true });
window.addEventListener('touchstart', unlockAudioContext, { once: true });
```

### B. Web Audio API 사운드 차임 (Acoustic Chimes)
외부 사운드 파일 의존성 없이 순수 브라우저 오실레이터로 깔끔한 신호음을 합성:
* **주의/느림**: 낮은 주파수 2단 알림 (420Hz → 320Hz)
* **빠름/조절**: 높은 주파수 2단 알림 (600Hz → 840Hz)
* **목표 달성/1km 돌파**: 4단 상승 화음 팡파레 (C5 → E5 → G5 → C6)

### C. 한국어 음성 합성 (Web Speech API TTS)
```javascript
const utter = new SpeechSynthesisUtterance(text);
utter.lang = 'ko-KR';
utter.rate = 1.1; // 운동/이동 시 늘어지지 않도록 1.1배속 권장
window.speechSynthesis.speak(utter);
```

### D. 햅틱 진동 피드백
```javascript
if ('vibrate' in navigator) {
  navigator.vibrate([200, 100, 200, 100, 400]); // 축하 또는 경고 패턴
}
```

---

## 5. 실전 모바일 사용성 & UX 디테일 (Ergonomics)

1. **화면 꺼짐 방지 (Screen Wake Lock API)**:
   이동 중 손에 쥐거나 거치대에 두었을 때 화면이 절전 모드로 들어가지 않도록 `navigator.wakeLock.request('screen')` 필수 적용.
2. **오터치 방지 잠금 (Lock Screen Overlay)**:
   달리거나 활동 중 땀이나 손 흔들림으로 중요한 버튼(종료/일시정지)이 눌리지 않도록 화면 잠금 모드 및 잠금 해제 인터랙션 제공.
3. **완료 리포트 모달 (Summary Popup)**:
   활동 종료 시 단순 텍스트 표시가 아닌 총 거리, 시간, 평균 페이스, 소모 칼로리 등을 요약하는 스포츠 모달 제공.
4. **빈 상태 (Empty State) 안내**:
   데이터가 0건일 때 테이블을 비워두지 않고, 시각적 아이콘과 함께 직관적인 시작 유도 메시지 표시.
5. **전체화면 지원**:
   상단에 전체화면(`requestFullscreen`) 버튼을 두어 브라우저 주소창을 숨기고 네이티브 앱처럼 몰입도 제공.

---

## 6. 점검 체크리스트 (UI/UX Checklist)

* [ ] 배경과 텍스트의 대비가 충분하여 야외에서도 시인성이 확보되는가?
* [ ] 대형 숫자 폰트에 `tabular-nums`가 적용되어 자릿수 흔들림이 없는가?
* [ ] 모바일 터치 타겟(버튼)의 최소 높이가 48px 이상인가?
* [ ] 브라우저 자동재생 제한을 해제하는 언락 로직이 터치 이벤트에 연결되어 있는가?
* [ ] 초기 설치 시 불필요한 더미 데이터 없이 0km/0건으로 시작하는가?
* [ ] 데이터가 없을 때 친절한 Empty State 화면을 렌더링하는가?

