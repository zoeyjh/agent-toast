# GitHub Copilot CLI + Agent Toast 연동 가이드

GitHub Copilot CLI(`gh copilot`)는 Claude Code나 Codex CLI와 달리 내장 hook 시스템이 없습니다.
대신 셸 래퍼 함수를 작성해 작업 완료 시 Agent Toast에 수동으로 알림을 보낼 수 있습니다.

## 기본 명령

```powershell
agent-toast.exe --source copilot --event task_complete --pid $PID
```

- `--source copilot` : 토스트에 Copilot 아이콘 표시
- `--event task_complete` : 작업 완료 이벤트
- `--pid $PID` : 현재 터미널 창 자동 감지용 (생략 가능)

## PowerShell 래퍼 함수

`$PROFILE`에 추가하면 `copilot` 명령으로 사용 가능합니다.

```powershell
function copilot {
    gh copilot @args
    agent-toast.exe --source copilot --event task_complete --pid $PID
}
```

### 재사용 가능한 범용 래퍼

긴 작업 전반에 알림을 붙이고 싶을 때:

```powershell
function Invoke-WithToast {
    param([string]$Source = "copilot", [scriptblock]$Action)
    & $Action
    agent-toast.exe --source $Source --event task_complete --pid $PID
}

# 사용 예
Invoke-WithToast { gh copilot suggest "리팩토링 방법" }
```

## 이벤트 종류

`--event` 값에 따라 토스트 색상과 아이콘이 달라집니다.

| 값 | 의미 |
|---|---|
| `task_complete` | 작업 완료 (초록) |
| `user_input_required` | 입력 대기 (노랑) |
| `error` | 오류 발생 (빨강) |

## 메시지 커스텀

```powershell
agent-toast.exe --source copilot --event task_complete --message "Copilot 제안 완료" --pid $PID
```

## 설정 파일 위치

Agent Toast는 Copilot용 별도 설정 파일을 관리하지 않습니다.
위 명령을 `$PROFILE`의 래퍼 함수에 직접 작성하는 것이 전부입니다.

## 참고

- `$PROFILE` 파일 경로: `notepad $PROFILE` 로 열기
- 프로필이 없으면: `New-Item -Path $PROFILE -Force`
- 변경 후 적용: `. $PROFILE` (현재 세션에 즉시 반영)
