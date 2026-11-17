# PoE 시세 오버레이 (카카오 서버)

패스 오브 엑자일 카카오 서버용 비공식 무료 시세 확인 도구입니다. 아이템에 마우스를 올리고 `Ctrl+D`를 누르면 시세 창이 뜹니다.

> This product isn't affiliated with or endorsed by Grinding Gear Games or Kakao Games in any way.
> Grinding Gear Games·카카오게임즈와 무관한 비공식 도구입니다. 게임 파일·메모리를 읽거나 바꾸지 않으며,
> 클립보드 복사(키 한 번에 한 번)와 공식 거래소 웹 API만 사용합니다. 사용에 따른 책임은 사용자에게 있습니다.

## 설치

1. [Releases](https://github.com/Sulhwa99/poe-price-overlay-releases/releases)에서 최신 `PoEPriceOverlay-Setup-x.y.z.exe`를 받습니다.
2. 실행하면 **"Windows의 PC 보호"** 파란 창이 뜰 수 있습니다. 코드 서명이 없는 개인 프로그램이라 나오는 경고입니다.
   **[추가 정보] → [실행]**을 누르면 설치됩니다. 
3. 게임이 관리자 권한으로 실행되므로 오버레이도 관리자 권한을 요청합니다 (그래야 게임 안에서 단축키가 동작합니다).
4. 이후 업데이트는 자동으로 받습니다.

게임 화면은 **창 모드 또는 테두리 없는 창 모드**여야 합니다 (전체 화면 독점 모드는 오버레이를 띄울 수 없음).

## 문제 신고

시세 창의 **[신고]** 버튼을 누르면 아이템 텍스트와 검색 조건만 개발자에게 보내집니다 (계정·캐릭터 이름, PC 정보는 보내지 않음).

---

아래는 개발 문서입니다.

## 지금 되는 것

| 단축키 | 기능 | 게임에 보내는 입력 | 거래소 호출 |
| :--- | :--- | :--- | :--- |
| `Ctrl+D` | 시세 창 (고유·화폐는 poe.ninja, 그 외 거래소) | Ctrl+Alt+C 1회 | 고유·화폐 0회 / 그 외 search 1 + fetch ≤2 |
| `Ctrl+Alt+D` | 거래소 웹 페이지로 바로 열기 | Ctrl+Alt+C 1회 | 0회 (검색 조건을 주소에 담아 엶) |
| `F5` | 은신처 이동 | `/hideout` 채팅 1건 | 0회 |
| `F6` | 가격 급상승 TOP 20 (poe.ninja) | 없음 | 0회 |
| `Esc` | 열린 창 닫기 (게임에도 그대로 전달) | 없음 | 0회 |

- 시세 창에서 옵션 체크와 최솟값(−/+)을 바꾸고 **[다시 검색]**을 누르면 그 조건으로 1회 검색합니다.
- 같은 조건은 10분 동안 저장된 결과를 보여 주고 거래소를 다시 부르지 않습니다.
- UI는 기능 확인용 최소 화면입니다. Figma 디자인이 나오면 `app/src/`를 교체합니다.

## 다음 차수 (설계서 9장 4단계~)
핀 모드 + 등급 판정, 히스토리·즐겨찾기·알림, 맵 위험 체크·정규식, 거래 귓속말·창고 하이라이트·빠른 채팅, 상위 베이스, 통계, 화면 설정.

## 폴더 구조
```
core/   Python Core (단축키, 복사, 파싱, 시세, WebSocket)
  poe_core/input/     키 판정(KeyGate), 복사 1회, 채팅 1건 — 게임 입력은 win_input.py에서만
  poe_core/parser/    클립보드 파서, DPS, 합산 스탯, 스탯 매칭, 언어 표기(locale.py)
  poe_core/pricing/   Rate Limit, 거래소 클라이언트, poe.ninja 캐시, 이름 변환, 시장 분석
  tests/              정책 준수 테스트 포함 (pytest)
app/    Electron + React (투명 오버레이, 트레이)
scripts/build-release.ps1   설치 파일 빌드
```

## 1. 개발 PC에서 바로 실행해 보기 (Windows)
준비: Python 3.11+, Node.js 20+
```powershell
cd core
python -m venv .venv
.\.venv\Scripts\pip install -r requirements-dev.txt
.\.venv\Scripts\python -m pytest -q          # 테스트

cd ..\app
npm ci
$env:POE_PYTHON = (Resolve-Path ..\core\.venv\Scripts\python.exe).Path   # 개발 모드에서 쓸 파이썬
npm run dev
```
게임을 테두리 없는 창 모드로 켜고 아이템 위에서 `Ctrl+D`를 눌러 봅니다.
> ⚠ 카카오 PoE가 관리자 권한으로 실행 중이면 **cmd도 관리자 권한으로 열어서** `npm run dev`를 해야 단축키가 동작합니다.
> 설치판은 자동으로 관리자 권한으로 실행됩니다 (실행할 때 Windows 확인 창이 한 번 뜸). 트레이 아이콘에서 로그 폴더를 열 수 있습니다.

## 2. 설치 파일 만들기
```powershell
powershell -ExecutionPolicy Bypass -File scripts\build-release.ps1
```
결과: `app\dist\PoEPriceOverlay-Setup-2.2.0.exe` — 이 파일 하나를 친구에게 전달합니다.
친구 PC에는 Python·Node.js가 필요 없습니다 (Python 런타임 내장).

**친구 안내 문구**
> "Windows의 PC 보호" 창이 뜨면 [추가 정보] → [실행]을 누르세요. 서명이 없는 개인 프로그램이라 뜨는 경고입니다.
> 프로그램을 켤 때마다 "이 앱이 디바이스를 변경하도록 허용하시겠어요?" 창이 뜹니다. 게임이 관리자 권한으로 돌아서
> 오버레이도 같은 권한이 있어야 단축키가 동작하기 때문입니다. [예]를 누르세요.
> 채팅창에 글을 쓰는 중에 F5를 누르면 쓰던 글이 먼저 전송될 수 있으니 주의하세요.

**자동 업데이트 배포**는 [RELEASE.md](RELEASE.md)를 보세요. 처음 한 번 저장소·토큰을 설정하면, 이후에는 태그만 푸시하면
GitHub Actions가 테스트 → 빌드 → 배포하고 친구들 프로그램이 알아서 업데이트합니다.

## 3. 첫 테스트 때 꼭 확인할 것 (설계서 7장)
1. **게임 감지:** 우상단에 `● 게임 감지됨`이 뜨는지. 안 뜨면 작업 관리자 > 세부 정보에서 게임 실행 파일 이름을 확인해
   `%APPDATA%\PoEPriceOverlay\config.json`의 `game.process_names`에 추가합니다.
2. **리그:** `config.json`의 `league`를 현재 리그 ID로 바꿉니다 (기본 `Standard`). 리그 ID는 거래소 `/api/trade/data/leagues`의 `id`.
3. **연락처:** `config.json`의 `contact`에 연락처(이메일 등)를 넣으면 User-Agent에 들어갑니다 (GGG·poe.ninja 권장 사항).
4. **클립보드 샘플:** 아이템 위에서 `Ctrl+Alt+C` → 메모장에 붙여 넣어 `core/tests/fixtures/ko_종류_번호.txt`로 저장 (30종 이상).
   `pytest tests/test_real_fixtures.py -v -s`로 인식 못 한 줄을 확인하고 `core/poe_core/parser/locale.py`를 고칩니다.
5. **거래소 웹 링크:** `Ctrl+Alt+D`로 열린 페이지에 검색 조건이 제대로 들어가는지 (`?q=` 방식 동작 확인).

## 알려진 제한
- 한국어 클립보드 표기 일부(분열/제작 꼬리표, 티어 표기, 아이템 종류 이름)는 추정값입니다 → 3-4번으로 확정.
- 갑충석·점술 카드·에센스 등은 poe.ninja 문서의 지원 목록에 없어 poe.ninja 가격 대신 거래소로 조회합니다.
- 오버레이는 게임 포커스를 빼앗지 않도록 키보드 입력을 받지 않습니다 (최솟값은 −/+ 버튼으로 조정).
- 게임 화면 모드: **창 모드, 테두리 없는 창 모드** 지원 (오버레이가 게임 창 위치·크기를 따라감, 다른 모니터 포함).
  **전체 화면(독점) 모드는 지원하지 않습니다** — Windows 특성상 다른 창을 그 위에 띄울 수 없습니다.
- 시세 창·급상승 목록은 게임 쪽(창 바깥)을 클릭하면 닫힙니다. 클릭은 읽기만 하고 게임에는 그대로 전달됩니다.

## 정책 준수 (테스트로 강제)
`core/tests/test_compliance.py`, `test_input_compliance.py`, `test_key_gate.py`가 다음을 검사하고, 실패하면 빌드가 중단됩니다.
- 키 입력 1회당 복사 1회, 키 반복 무시, 프로그램이 보낸 입력에 반응하지 않음
- 채팅은 1회 1건, 줄바꿈 거부, 0.5초 간격
- SendInput은 `win_input.py`에서만, 마우스 입력·화면 캡처·프로세스 메모리 API 사용 없음
- 쿠키·브라우저 저장소 사용 없음
- 거래소 검색은 엔진(수동 조회)에서만, 백그라운드 루프는 poe.ninja 갱신뿐
- Rate Limit: 헤더 기준 75% 안전선 + 검색 최소 2.5초 간격, 429 시 정지

## 사용한 외부 자료

- 게임 문장 → 거래소 스탯 표(`core/poe_core/data/client_stats_ko.json.gz`)는
  [Awakened PoE Trade](https://github.com/SnosMe/awakened-poe-trade)의 한국어 스탯 자료로 만들었습니다
  (MIT License, Copyright (c) 2020 Alexander Drozdov, 전문: `core/poe_core/data/LICENSE-awakened-poe-trade.txt`).
  시즌이 바뀌면 `core/tools/build_client_stats.py` 안내대로 다시 만듭니다.
- 고양이 테마 그림(`app/src/assets/cat/`)은 [Microsoft Fluent Emoji](https://github.com/microsoft/fluentui-emoji)의 3D 그림입니다
  (MIT License, Copyright (c) Microsoft Corporation, 전문: `app/src/assets/cat/LICENSE-fluentui-emoji.txt`).

- "꽁꽁 얼어붙은 고양이" 테마 그림(`app/src/assets/frozen/`)은 프로젝트 관리자가 직접 준비한 그림입니다.

## 라이선스

이 프로그램의 코드는 [MIT License](LICENSE)입니다. 포함된 외부 자료(옵션 문장 표, 고양이 그림, 글꼴)는 각자의 라이선스를 따릅니다 (위 "사용한 외부 자료"와 `app/src/fonts/LICENSE-fonts.txt`).
