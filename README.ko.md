<div align="center">

<img src="https://github.com/user-attachments/assets/b50a50bb-2404-4153-a312-aa5784a16928" alt="Claude Cowork for Linux (비공식)" width="800">

 # Claude Cowork on Linux
 ### macOS도, VM도 필요 없습니다.

<br>

![Platform](https://img.shields.io/badge/platform-Linux%20x86__64-blue?style=flat-square)
![Version](https://img.shields.io/badge/version-v4.0.0-brightgreen?style=flat-square)
![Tested](https://img.shields.io/badge/tested-Arch%20Linux-1793D1?style=flat-square&logo=archlinux&logoColor=white)
![Status](https://img.shields.io/badge/status-Working-success?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)

**[빠른 시작](#-빠른-시작)** · **[작동 원리](#-작동-원리)** · **[수동 설치](#-수동-설치)** · **[문제 해결](#-문제-해결)**

</div>

---

## ![](.github/assets/icons/info-24x24.png) 개요

Claude Cowork는 사용자가 지정한 폴더 안에서 파일을 읽고, 쓰고, 정리하면서 작업 계획을 실행하는 특별한 Claude Desktop 빌드입니다. Cowork는 현재 **macOS 전용 프리뷰**로, 샌드박스된 Linux VM을 기반으로 동작합니다. 이 레포지토리는 macOS 전용 네이티브 부분을 역공학(reverse-engineering)하여 stub으로 대체함으로써, **VM도 macOS도 없이** Linux(x86_64)에서 직접 Cowork를 실행할 수 있게 합니다. stub은 VM 경로를 호스트 경로로 변환하여 Linux에서도 올바른 파일을 가리키도록 합니다.

**작동 방식:**

| 단계 | 설명 |
|:-----|:-----|
| ![](.github/assets/icons/script-24x24.png) **Stub 교체** | macOS 전용 네이티브 모듈(`@ant/claude-swift`, `@ant/claude-native`)을 JavaScript로 대체 |
| ![](.github/assets/icons/console-24x24.png) **직접 실행** | Claude Code 바이너리를 직접 실행 (Linux이니까 VM 불필요!) |
| ![](.github/assets/icons/translation-24x24.png) **경로 변환** | VM 경로를 호스트 경로로 투명하게 변환 |
| ![](.github/assets/icons/platform-24x24.png) **플랫폼 위장** | 서버에 macOS 헤더를 전송하여 Cowork 기능 활성화 |

---

## ![](.github/assets/icons/status-24x24.png) 상태

- **비공식 리서치 프리뷰**: 역공학으로 구현되었으며 Claude Desktop 업데이트 시 작동이 중단될 수 있습니다.
- **Linux 지원**: 현재 **Linux x86_64** 대상. Wayland는 `$WAYLAND_DISPLAY` / `$XDG_SESSION_TYPE`로 자동 감지 (Ozone 백엔드).
- **접근**: Claude 계정이 필요합니다. 설치 프로그램이 Claude Desktop DMG를 자동 다운로드하므로 macOS 기기가 없어도 됩니다.
- **테스트**: 18개 테스트 파일에 215개 이상의 테스트 케이스 (IPC, 경로 변환, 보안, 세션 지속성 검증).

---

## ![](.github/assets/icons/platform-24x24.png) 호환성

| 배포판 | 데스크톱 | 상태 | 비고 |
|:-------|:---------|:-----|:-----|
| **Arch Linux** | Hyprland (Wayland) | 테스트됨 | 주 개발 환경 |
| **Arch Linux** | KDE Plasma (Wayland) | 예상됨 | KDE Wallet은 SecretService D-Bus로 노출됨 |
| **Arch Linux** | GNOME (Wayland) | 예상됨 | 글로벌 단축키는 DE 수동 설정 필요 (GNOME은 포털 미지원) |
| **Ubuntu 22.04+** | GNOME / X11 | 예상됨 | gnome-keyring이 SecretService 제공 |
| **Fedora 39+** | GNOME / KDE | 예상됨 | DMG 추출에 `p7zip-plugins` 필요할 수 있음 |
| **Debian 12+** | 모두 | 예상됨 | apt에서 `p7zip-full` 설치 |
| **NixOS** | 모두 | 미테스트 | Electron + bwrap 샌드박싱에 추가 설정 필요할 수 있음 |
| **openSUSE** | 모두 | 테스트됨 | `7zip` 패키지 사용 (p7zip 아님); Node.js는 `nodejs-default` |

**알려진 주의사항:**
- `GlobalShortcuts` 포털을 구현하지 않는 Wayland 컴포지터(GNOME)에서는 글로벌 단축키가 동작하지 않습니다. DE 설정에서 직접 단축키를 지정하세요.
- `gnome-keyring` 또는 다른 SecretService 제공자가 실행 중이지 않으면, 런처가 `--password-store=basic`으로 대체됩니다 (인증 정보가 키링이 아닌 디스크에 저장됨).
- `/sessions` 루트 심볼릭 링크는 설치 시 `sudo`가 한 번 필요합니다. 배포판이 루트 심볼릭 링크를 제한하는 경우 수동으로 생성하세요: `sudo ln -s "$HOME/.config/Claude/local-agent-mode-sessions/sessions" /sessions`.

설치 후 `./install.sh --doctor` (또는 `claude-desktop --doctor`)로 환경을 검증하세요.

---

## ![](.github/assets/icons/checkbox-24x24.png) 요구 사항

- **Linux x86_64** (Arch Linux, 커널 6.18.13에서 테스트됨)
- **Node.js 18+** / npm
- **Electron** (시스템 패키지 또는 npm 전역 설치)
- **asar** (`npm install -g @electron/asar`)
- **p7zip** (macOS DMG 추출용; openSUSE는 `7zip` 사용)
- **bubblewrap** (샌드박스 격리)
- **Python 3.11+** (선택사항, `enable-cowork.py` 패치용 — 설치 프로그램은 Node.js로 DMG 다운로드)
- **Claude Pro** 이상 구독 (Cowork 기능 접근에 필요)
- **Secret service 제공자** (선택사항) — 안전한 인증 정보 저장을 위한 gnome-keyring, KDE Wallet, 또는 KeePassXC. 없는 경우 `--password-store=basic`으로 대체됨.

---

## ![](.github/assets/icons/rocket-24x24.png) 빠른 시작

### 방법 1: install.sh (권장)

```bash
git clone https://github.com/byounghak36/claude-cowork-linux.git
cd claude-cowork-linux
./install.sh          # Node.js로 최신 DMG 자동 다운로드
claude-desktop
```

### 방법 2: AUR (Arch Linux)

```bash
yay -S claude-cowork-linux       # 최신 DMG 자동 다운로드
```

### 방법 3: curl 파이프

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/byounghak36/claude-cowork-linux/master/install.sh)
```

설치 프로그램이 Node.js(`scripts/fetch-dmg.js`)를 사용해 최신 Claude Desktop DMG를 자동 다운로드합니다. DMG를 수동으로 제공할 수도 있습니다:

```bash
./install.sh ~/Downloads/Claude-*.dmg
# 또는
CLAUDE_DMG=~/Downloads/Claude-1.1.4010.dmg ./install.sh
```

> [!IMPORTANT]
> 이 레포지토리에는 Anthropic의 독점 코드가 포함되어 있지 않습니다. 설치 프로그램이 Anthropic CDN에서 직접 다운로드합니다.

---

## ![](.github/assets/icons/architecture-24x24.png) 아키텍처

```
┌─────────────────────────────────────────────────────────────────┐
│                     Claude Desktop (Electron)                   │
├─────────────────────────────────────────────────────────────────┤
│  @ant/claude-swift (STUB 처리됨)                                │
│  ├── vm.setEventCallbacks() → 프로세스 이벤트 핸들러 등록       │
│  ├── vm.startVM() → No-op (이미 Linux이므로 VM 불필요)          │
│  ├── vm.spawn() → session orchestrator에 위임                   │
│  ├── vm.kill() → 스폰된 프로세스 종료                           │
│  └── vm.writeStdin() → 프로세스 stdin에 쓰기                    │
├─────────────────────────────────────────────────────────────────┤
│  @ant/claude-native (STUB 처리됨)                               │
│  ├── AuthRequest → 시스템 브라우저 열기 (xdg-open)             │
│  └── Platform helpers → 최소한의 호환성 shim                   │
├─────────────────────────────────────────────────────────────────┤
│  stubs/cowork/ — 오케스트레이션 레이어 (15개 모듈)              │
│  ├── session_orchestrator.js   → 스폰 라이프사이클 조율        │
│  ├── asar_adapter.js            → Asar IPC API 호환성           │
│  ├── process_manager.js         → 프로세스 라이프사이클 & I/O  │
│  ├── resume_coordinator.js      → 세션 재개 로직               │
│  ├── sessions_api.js            → 세션 CRUD 작업               │
│  ├── session_store.js           → 인메모리 세션 상태            │
│  ├── transcript_store.js        → 대화 내용 지속성             │
│  ├── file_registry.js           → 작업 디렉토리 추적           │
│  ├── file_watch_manager.js      → 파일 변경 감지               │
│  ├── stream_protocol.js         → JSON-RPC 스트림 파싱         │
│  ├── credential_classifier.js   → 토큰 유출 방지               │
│  ├── eipc_channel.js            → EIPC 메시지 프로토콜         │
│  ├── ipc_tap.js                 → IPC 채널 탐색                │
│  ├── dirs.js                    → XDG 디렉토리 해석            │
│  └── file_identity.js           → 경로 정규화                  │
├─────────────────────────────────────────────────────────────────┤
│  Claude Code 바이너리                                           │
│  └── ~/.local/bin, mise/asdf shims, PATH 등에서 해석            │
│      (launch.sh가 macOS Mach-O 바이너리를 Linux 심볼릭 링크로  │
│       자동 교체)                                                │
└─────────────────────────────────────────────────────────────────┘
```

### 경로 변환

stub은 VM 경로를 호스트 경로로 변환합니다:

| VM 경로 | 호스트 경로 |
|:--------|:-----------|
| `/usr/local/bin/claude` 또는 `claude` | `~/.local/bin/claude`, `~/.config/Claude/claude-code-vm/{version}/claude`, 또는 PATH에서 해석 |
| `/sessions/...` | `~/.config/Claude/local-agent-mode-sessions/sessions/...` |

### 마운트 심볼릭 링크

Cowork에서 폴더를 선택하면, stub이 심볼릭 링크를 생성하여 VM 경로에서 접근 가능하게 합니다:

```
~/.config/Claude/local-agent-mode-sessions/sessions/<session-name>/mnt/
├── <folder>  → /home/user/path/to/selected/folder (심볼릭 링크)
├── .claude   → ~/.config/Claude/local-agent-mode-sessions/.../session/.claude (심볼릭 링크)
├── .skills   → ~/.config/Claude/local-agent-mode-sessions/skills-plugin/... (심볼릭 링크)
└── uploads/  (파일 업로드용 디렉토리)
```

`additionalMounts` 파라미터가 마운트 이름과 호스트 경로 간의 매핑을 제공합니다.

> [!NOTE]
> Claude Code 바이너리는 `/sessions`가 존재해야 합니다. `install.sh`가 `/sessions`를 `~/.config/Claude/local-agent-mode-sessions/sessions`의 심볼릭 링크로 한 번 생성합니다 (`sudo` 한 번 필요). 루트에 쓰기 권한이 있는 디렉토리는 필요하지 않습니다.

---

## ![](.github/assets/icons/how-it-works-24x24.png) 작동 원리

<details>
<summary><strong>1. 플랫폼 위장 (Platform Spoofing)</strong></summary>

앱이 Anthropic 서버에 다음 헤더를 전송합니다:

```javascript
'Anthropic-Client-OS-Platform': 'darwin'
'Anthropic-Client-OS-Version': '14.0'
```

이로 인해 서버는 macOS 14 (Sonoma)로 인식하여 Cowork 기능을 활성화합니다.

</details>

<details>
<summary><strong>2. 플랫폼 게이트 우회 (Platform Gate Bypass)</strong></summary>

플랫폼 게이트 함수(미니파이된 이름은 빌드마다 변경 — v1.1.3963에서는 `xPt()`, 이전 빌드에서는 `wj()`)가 Cowork 지원 여부를 확인합니다. `enable-cowork.py`가 이 함수를 자동으로 찾아 `{status: "supported"}`를 무조건 반환하도록 교체합니다.

</details>

<details>
<summary><strong>3. Swift 애드온 Stub</strong></summary>

원본 `@ant/claude-swift`는 Apple의 Virtualization Framework를 사용합니다. 우리의 stub은:

- 동일한 API 표면을 구현
- spawn 로직을 `session_orchestrator.js`에 위임하여 라이프사이클 관리
- JSON 출력을 라인 단위로 버퍼링하여 올바른 스트림 파싱
- VM 경로를 호스트 경로로 변환

핵심 포인트: 앱이 `Si()`를 호출하여 `module.default.vm`을 반환받으므로, 메서드는 `vm` 객체에 있어야 합니다.

</details>

<details>
<summary><strong>4. 네이티브 유틸리티 Stub</strong></summary>

앱은 `@ant/claude-native`(macOS 전용 네이티브 모듈)도 필요로 합니다. 우리의 stub이 최소한의 호환성을 제공하여 Linux에서 앱이 시작될 수 있게 합니다. 예를 들어 OAuth 흐름은 `xdg-open`으로 시스템 브라우저를 여는 방식으로 대체됩니다.

</details>

<details>
<summary><strong>5. 세션 오케스트레이션 레이어</strong></summary>

`stubs/cowork/` 오케스트레이션 레이어는 세션 라이프사이클, IPC 통신, 대화 내용 지속성, 보안을 처리하는 15개 모듈을 제공합니다:

- **session_orchestrator.js** — 모든 spawn 작업, 마운트 심볼릭 링크, 프로세스 정리를 조율
- **credential_classifier.js** — 스폰된 프로세스로의 인증 토큰 유출 방지
- **ipc_tap.js** — `ipcMain._invokeHandlers.set()` 탭핑으로 런타임에 EIPC 채널 탐색
- **transcript_store.js** — 대화 기록을 `~/.config/Claude/local-agent-mode-sessions/`에 지속
- **file_watch_manager.js** — 작업 디렉토리의 파일 변경 감지

모든 모듈은 XDG 기본 디렉토리 규칙을 따르며 215개 이상의 테스트 케이스로 검증됩니다.

</details>

<details>
<summary><strong>6. 직접 실행 (Direct Execution)</strong></summary>

macOS에서 Cowork는 Linux VM을 실행합니다. Linux에서는 VM을 완전히 건너뛰고 호스트에서 Claude Code 바이너리를 직접 실행합니다. 실제로 더 단순하고 빠릅니다!

stub은 다음 우선순위로 바이너리를 해석합니다:
```
$CLAUDE_CODE_PATH                                    (명시적 재정의)
~/.config/Claude/claude-code-vm/{version}/claude    (Desktop에서 다운로드)
~/.local/bin/claude                                  (npm/bun 전역)
~/.npm-global/bin/claude
/usr/local/bin/claude
/usr/bin/claude
/home/linuxbrew/.linuxbrew/bin/claude               (Linuxbrew 시스템)
~/.linuxbrew/bin/claude                              (Linuxbrew 사용자)
~/.local/share/mise/shims/claude                     (mise 버전 매니저)
~/.asdf/shims/claude                                 (asdf 버전 매니저)
```

**Code 탭 바이너리 교체**: `launch.sh`가 asar 내의 Claude Code 바이너리가 macOS Mach-O인지 자동 감지하여 호스트 Linux 바이너리의 심볼릭 링크로 교체합니다. 이로 인해 Code 탭이 원활하게 동작합니다.

</details>

---

## ![](.github/assets/icons/folder-24x24.png) 프로젝트 구조

```
claude-cowork-linux/
├── stubs/
│   ├── @ant/claude-swift/js/index.js   # 주 stub: vm.spawn() → orchestrator에 위임
│   ├── @ant/claude-native/index.js     # 인증(xdg-open), 키보드 상수, 플랫폼 헬퍼
│   ├── cowork/                         # 오케스트레이션 레이어 (15개 모듈)
│   │   ├── session_orchestrator.js     # 스폰 라이프사이클 조율자
│   │   ├── asar_adapter.js             # Asar IPC API 호환성
│   │   ├── process_manager.js          # 프로세스 라이프사이클 & I/O
│   │   ├── resume_coordinator.js       # 세션 재개 로직
│   │   ├── sessions_api.js             # 세션 CRUD 작업
│   │   ├── session_store.js            # 인메모리 세션 상태
│   │   ├── transcript_store.js         # 대화 내용 지속성
│   │   ├── file_registry.js            # 작업 디렉토리 추적
│   │   ├── file_watch_manager.js       # 파일 변경 감지
│   │   ├── stream_protocol.js          # JSON-RPC 스트림 파싱
│   │   ├── credential_classifier.js    # 토큰 유출 방지
│   │   ├── eipc_channel.js             # EIPC 메시지 프로토콜
│   │   ├── ipc_tap.js                  # IPC 채널 탐색
│   │   ├── dirs.js                     # XDG 디렉토리 해석
│   │   └── file_identity.js            # 경로 정규화
│   └── frame-fix/
│       ├── frame-fix-wrapper.js        # 초기 부트스트랩: TMPDIR 수정, 플랫폼 위장, 정상 종료
│       └── frame-fix-entry.js          # 진입점: frame-fix-wrapper 로드 후 main index.js 실행
├── tests/
│   ├── node/current-path/             # 18개 테스트 파일, 215개 이상 node:test 케이스
│   └── ...
├── scripts/
│   ├── fetch-dmg.js                   # Node.js fetch로 Claude DMG 자동 다운로드
│   └── enable-cowork.py               # 플랫폼 게이트를 {status:"supported"} 반환으로 패치
├── docs/
│   ├── FAQ.md                         # 상세 문제 해결 가이드
│   ├── extensions.md                  # MCP 및 Chrome 확장 통합 개요
│   ├── known-issues.md                # Safe Storage 암호화, 키링 설정
│   └── safestorage-tokens.md          # 재시작 후 토큰 지속 방법
├── config/
│   └── hyprland/claude.conf           # 선택사항: Hyprland 윈도우 규칙
├── install.sh                         # 설치 프로그램 + --doctor 사전 진단
├── launch.sh                          # 런처: stub 동기화, asar 재패킹, electron 실행
├── launch-devtools.sh                 # --inspect 포함 런처 (Node.js DevTools)
├── validate.sh                        # 환경 변수 확인, stub URL 검증, 로그 스캔
└── PKGBUILD                           # Arch Linux AUR 패키지 정의
```

`install.sh` 실행 후 `linux-app-extracted/` 디렉토리에 추출된 Claude Desktop이 생성됩니다.

---

## ![](.github/assets/icons/console-24x24.png) 수동 설치

자동 설치 프로그램이 동작하지 않는 경우, 다음 단계를 따르세요:

<details>
<summary><strong>1. DMG에서 Claude Desktop 추출</strong></summary>

```bash
# 7z으로 DMG 추출
7z x Claude-*.dmg -o/tmp/claude-extract

# 앱 디렉토리 생성
mkdir -p linux-app-extracted

# 최신 버전 (app.asar):
if [ -f "/tmp/claude-extract/Claude/Claude.app/Contents/Resources/app.asar" ]; then
    npx --yes asar extract "/tmp/claude-extract/Claude/Claude.app/Contents/Resources/app.asar" linux-app-extracted
    [ -d "/tmp/claude-extract/Claude/Claude.app/Contents/Resources/app.asar.unpacked" ] && \
        cp -r "/tmp/claude-extract/Claude/Claude.app/Contents/Resources/app.asar.unpacked/"* linux-app-extracted/
elif [ -d "/tmp/claude-extract/Claude/Claude.app/Contents/Resources/app" ]; then
    cp -r "/tmp/claude-extract/Claude/Claude.app/Contents/Resources/app/"* linux-app-extracted/
fi

rm -rf /tmp/claude-extract
```

</details>

<details>
<summary><strong>2. Stub 모듈 설치</strong></summary>

```bash
cp -r stubs/@ant/* linux-app-extracted/node_modules/@ant/
cp -r stubs/cowork linux-app-extracted/node_modules/
```

</details>

<details>
<summary><strong>3. index.js 패치</strong></summary>

```bash
python3 scripts/enable-cowork.py linux-app-extracted/.vite/build/index.js
```

</details>

<details>
<summary><strong>4. 필수 디렉토리 생성</strong></summary>

```bash
mkdir -p "$HOME/.config/Claude/local-agent-mode-sessions/sessions"
chmod 700 "$HOME/.config/Claude/local-agent-mode-sessions/sessions"

# 심볼릭 링크 생성 (sudo 한 번 필요)
sudo ln -s "$HOME/.config/Claude/local-agent-mode-sessions/sessions" /sessions
```

</details>

<details>
<summary><strong>5. Electron 및 asar 설치</strong></summary>

```bash
# 시스템 패키지 (권장)
# Arch: pacman -S electron
# Ubuntu/Debian: apt install electron
# 또는 npm으로:
npm install -g electron @electron/asar
```

</details>

---

## ![](.github/assets/icons/warning-24x24.png) 문제 해결

상세한 문제 해결 가이드는 **[docs/FAQ.md](docs/FAQ.md)**를 참조하세요.

<details>
<summary><strong>패치 적용 확인</strong></summary>

```bash
grep -q 'cowork-patched' linux-app-extracted/.vite/build/index.js && echo "✓ Cowork 패치 적용됨" || echo "✗ 패치 없음 - ./install.sh 실행하세요"
```

</details>

<details>
<summary><strong>EACCES: permission denied, mkdir '/sessions'</strong></summary>

```bash
mkdir -p "$HOME/.config/Claude/local-agent-mode-sessions/sessions"
sudo ln -s "$HOME/.config/Claude/local-agent-mode-sessions/sessions" /sessions
```

</details>

<details>
<summary><strong>JSON 파싱 오류 (Unexpected non-whitespace character after JSON)</strong></summary>

stub이 라인 단위 버퍼링으로 완전한 JSON 객체를 전송합니다. 지속된다면 트레이스 로그를 확인하세요:

```bash
cat ~/.local/state/claude-cowork/logs/claude-swift-trace.log
```

</details>

<details>
<summary><strong>작업 공간 시작 실패 (Failed to start Claude's workspace)</strong></summary>

먼저 `claude-desktop --doctor`로 환경을 확인하세요. 그리고:

1. swift stub이 제대로 로드되었는지 확인 (로그에서 `[claude-swift-stub] LOADING MODULE` 확인)
2. Claude 바이너리가 해석된 경로 중 하나에 존재하는지 확인
3. 유효한 Claude 계정이 있는지 확인

</details>

<details>
<summary><strong>프로세스가 즉시 종료 (code=1)</strong></summary>

```bash
tail -50 ~/.local/state/claude-cowork/logs/claude-swift-trace.log
```

주요 원인:
- `/sessions` 심볼릭 링크 없음
- 바이너리를 찾을 수 없음
- 권한 문제

</details>

<details>
<summary><strong>앱이 재실행되지 않음 / 아무 반응 없음</strong></summary>

이전 인스턴스가 비정상 종료되어 잠금 파일이 남아 있을 수 있습니다:

```bash
rm -f ~/.config/Claude/SingletonLock ~/.config/Claude/SingletonSocket ~/.config/Claude/SingletonCookie
claude-desktop
```

</details>

<details>
<summary><strong>Wayland에서 글로벌 단축키 동작 안 함 (GNOME)</strong></summary>

앱이 Wayland 글로벌 단축키를 위해 `GlobalShortcutsPortal`을 활성화하지만, **KDE Plasma**와 **Hyprland**에서만 동작하며 **GNOME**에서는 동작하지 않습니다 (`xdg-desktop-portal-gnome`이 GlobalShortcuts 포털을 아직 구현하지 않음).

**GNOME Wayland 사용자 대안:** GNOME 설정 > 키보드 > 사용자 지정 단축키에서 `claude-desktop` 실행 단축키를 직접 등록하세요.

</details>

---

## ![](.github/assets/icons/console-24x24.png) 개발

```bash
./launch.sh                   # stub 변경 시 asar 자동 재패킹
./launch-devtools.sh          # Node.js inspector 포함
./validate.sh                 # 환경 변수 확인, stub URL 검증, 로그 오류 확인
./install.sh --doctor         # 사전 확인: 바이너리, node, CLI, /sessions, secret service, 패치

# 테스트 실행
node --test tests/node/current-path/*.test.cjs
```

### 디버그 로깅

```bash
export CLAUDE_COWORK_TRACE_IO=1    # Claude Code stdout/stderr를 트레이스 로그에 포함
export CLAUDE_COWORK_DEBUG=1       # 디버그 모드 활성화
export ELECTRON_ENABLE_LOGGING=1   # Electron 로깅 활성화

rm -f ~/.local/state/claude-cowork/logs/claude-swift-trace.log

./launch.sh 2>&1 | tee /tmp/claude-full.log

# 다른 터미널에서 트레이스 실시간 확인
tail -f ~/.local/state/claude-cowork/logs/claude-swift-trace.log
```

---

## ![](.github/assets/icons/shield-security-protection-24x24.png) 보안

이 프로젝트에는 다음 보안 강화 사항이 포함되어 있습니다:

- **커맨드 허용 목록** — `vm.spawn()`은 검증된 바이너리 경로만 허용; 알 수 없는 명령은 거부
- **커맨드 인젝션 방지** — `exec()` 대신 `execFile()` 사용
- **경로 탐색 방지** — `isPathSafe()`로 세션 경로 검증
- **환경 변수 필터링** — 안전한 환경 변수 허용 목록
- **안전한 권한** — 세션 디렉토리는 777이 아닌 700 사용
- **/sessions 심볼릭 링크** — 월드 쓰기 가능 디렉토리 없음
- **URL 출처 검증** — Anthropic 도메인만 허용
- **OAuth 규정 준수** — 서브프로세스로의 토큰 유출 방지
- **인증 정보 분류** — `credential_classifier.js`로 엄격한 토큰 유출 방지
- **CRLF 가드** — 스트림 파서가 JSON-RPC 메시지의 CRLF 인젝션 거부
- **FD 범위 확인** — 스폰된 프로세스의 파일 디스크립터 한도 강제

---

## 법적 고지

> [!CAUTION]
> 이 프로젝트는 **교육 및 연구 목적**으로 제작되었습니다. Claude Desktop은 Anthropic PBC가 소유한 독점 소프트웨어입니다. Cowork 사용에는 유효한 Claude 계정이 필요합니다.
>
> 이 레포지토리에는 stub 구현과 패치만 포함되어 있으며, Claude Desktop 애플리케이션 자체는 포함되지 **않습니다**. Claude Desktop은 반드시 Anthropic에서 직접 다운로드해야 합니다.
>
> 이 프로젝트는 **Anthropic과 무관하며, Anthropic의 승인이나 후원을 받지 않습니다**. "Claude"는 Anthropic PBC의 상표입니다.

---

## 크레딧

Claude Desktop Electron 앱 구조 분석, pyghidra-lite를 이용한 바이너리 분석, 반복적인 디버깅을 통해 역공학으로 구현되었습니다.

**기여자:**
- [@Boermt-die-Buse](https://github.com/Boermt-die-Buse) — Linux UI 수정: 네이티브 윈도우 프레임, 타이틀바 패치, 아이콘 추출
- [@JaPossert](https://github.com/JaPossert) — 리소스 복사 수정, Wayland 글로벌 단축키 보고
- [@alpham8](https://github.com/alpham8) — openSUSE 호환성 수정, 바이너리 해석 경로, Swift stub 메서드 stub
- [@matiasandina](https://github.com/matiasandina) — 아이콘 수정 및 터미널 분리 제안 (이슈 #37)

---

<div align="center">

**MIT 라이선스** · 자세한 내용은 [LICENSE](LICENSE) 참조

</div>
