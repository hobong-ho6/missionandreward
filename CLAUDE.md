# CLAUDE.md

# CLAUDE.md - 프로젝트 가이드라인

## 🌐 언어 및 응답 규칙 (Language & Response Rules)
- **모든 대화는 반드시 한국어**로 진행한다.
- 코드에 대한 설명, 분석 결과, 제안 사항 등 모든 텍스트 응답은 한국어를 사용한다.
- **새로 생성하는 모든 문서(Markdown, 주석 등)**의 텍스트 내용은 한국어로 작성한다.
- 기술 용어는 가급적 표준 용어를 사용하되, 필요한 경우 영문을 병기한다.

## 🛠️ 개발 지침 (Development Guidelines)
- 코드를 수정하거나 파일을 생성할 때, 변경 이유와 작동 원리를 한국어로 상세히 설명한다.
- 프로젝트 분석 결과 보고서나 가이드 문서를 작성할 때 한국어 문법과 가독성을 최우선으로 한다.

이 파일은 Claude Code (claude.ai/code)가 이 저장소에서 작업할 때 필요한 가이드를 제공합니다.

## 아키텍처 개요

이 프로젝트는 빌드 과정이 없는 **정적 웹 애플리케이션**으로, USDT 리워드 기반 게임 플랫폼입니다. 여러 HTML 페이지로 구성되어 있으며 인라인 JavaScript를 사용하고 localStorage를 통해 상태를 지속적으로 관리합니다.

### 핵심 상태 관리

모든 애플리케이션 상태는 localStorage를 통해 글로벌 상태 패턴으로 관리됩니다:

```javascript
// localStorage에 'unifi_state'로 저장되는 글로벌 상태 구조
{
  usdt: number,           // 사용자의 USDT 잔고
  tickets: number,        // USDT 뽑기권 개수
  history: array,         // 리워드 내역 엔트리
  gameStates: [0,0,0,0,0,0], // 6개 게임의 진행률 (0-4 상태)
  heartStates: [false,false,false,false,false,false] // 하트 리워드 수령 여부
}
```

상태 접근 함수들이 페이지 간에 중복되어 있습니다:
- `getGlobalState()` - localStorage에서 상태를 가져오고 초기화
- `saveGlobalState(state)` - 상태를 저장하고 재렌더링 트리거
- `addReward(amount)` - USDT 리워드를 추가하고 내역 업데이트

### 페이지 아키텍처

**주요 페이지:**
- `index.html` - USDT 잔고와 게임 캐러셀이 있는 홈 대시보드
- `daily_mission.html` - 일일 미션과 뽑기 시스템  
- `reward_history.html` - USDT/티켓 잔고와 리워드 내역
- `apps.html` - 게임 실행기 (style.css 대신 Tailwind CSS 사용)

**게임 상세 페이지:** (6개의 동일한 패턴)
- `game_detail_*.html` - 각각 `GAME_INDEX` 상수를 정의하고 동일한 UI/로직 패턴을 따름
- 게임: squishy_cat, tab_tab_jello, merge_cat, hook_and_gold, rich_match, soda_king

### 스타일링 시스템

- **style.css**: apps.html을 제외한 모든 페이지의 통합 스타일시트
  - 테마용 CSS 변수 (`--brand-green`, `--text-main` 등)
  - 모바일 우선 반응형 디자인 (최대 너비: 480px)
  - 공유 컴포넌트 클래스 (`.btn-black`, `.game-card` 등)

- **apps.html**: CDN을 통한 Tailwind CSS와 커스텀 설정 사용

## 개발 명령어

### 로컬 개발
```bash
# 로컬 서버 실행 (임의의 정적 서버)
python -m http.server 8000
# 또는
npx http-server
```

### 배포
```bash
# Vercel 프로덕션 배포
vercel --prod

# 미리보기 배포
vercel
```

### 테스트 및 개발
- 자동화된 테스트 없음 - 브라우저를 통한 수동 테스트
- 각 페이지에 재설정/디버그 버튼이 있는 테스트 패널 포함
- 상태 디버깅을 위해 브라우저 개발자 도구의 localStorage 인스펙터 사용

## 주요 패턴

### 새로운 게임 상세 페이지 추가
1. 기존 `game_detail_*.html` 복사 
2. `GAME_INDEX` 상수를 다음 사용 가능한 인덱스로 업데이트
3. 게임별 콘텐츠 업데이트 (제목, 이미지, 미션)
4. `index.html`의 홈페이지 게임 캐러셀에 항목 추가

### 상태 디버깅
브라우저 콘솔에서 다음 코드 사용:
```javascript
// 현재 상태 조회
JSON.parse(localStorage.getItem('unifi_state'))

// 모든 상태 재설정  
localStorage.removeItem('unifi_state')

// 특정 값 업데이트
let state = JSON.parse(localStorage.getItem('unifi_state'))
state.usdt = 1000
localStorage.setItem('unifi_state', JSON.stringify(state))
```

### 공통 UI 컴포넌트
- **LINE 알림**: 모든 페이지에서 공유되는 알림 시스템
- **USDT 리워드 오버레이**: 일관된 리워드 표시 패턴  
- **진행률 추적**: 5개 상태(0-4)를 가진 미션 진행률
- **하단 네비게이션**: 주요 페이지의 고정 네비게이션

## 파일 구조 고려사항

- 모든 JavaScript는 HTML 파일에 인라인으로 작성 - 별도 JS 파일 없음
- 외부 이미지는 `obs.line-scdn.net` CDN에서 제공
- 전체 UI가 한국어로 구성
- 배포 설정을 위한 `.vercel` 폴더 하나 (gitignore됨)

## Vercel 설정

프로젝트는 GitHub에서 자동 배포되도록 설정되어 있습니다. `.vercel/project.json`에 포함된 내용:
- 프로젝트 ID: `prj_i0pyxCD7bCJvILaoFPuxjXCDlqRw`  
- 배포 주소: `https://missionandreward.vercel.app`