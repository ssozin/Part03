---
description: 개발서버 자동 시작 (Windows 환경)
---

# 🖥️ 개발서버 설정

> 이 워크플로우는 Windows 로컬 개발서버 실행을 자동화합니다.
> 프로젝트 폴더를 루트로 하여 정적 파일을 서빙합니다.
> **npx live-server**를 기본으로 사용하여 Live Reload를 지원합니다.

## 포트 구분
| 환경 | 포트 | 비고 |
|------|------|------|
| **AI IDE** | 9000 | AI 에이전트가 자동 실행 |
| **수동 (VSCode 등)** | 8000 | 사용자가 직접 실행 |

> [!NOTE]
> 포트 번호는 프로젝트에 맞게 자유롭게 변경 가능합니다.
> 다른 서비스와 충돌하지 않는 포트를 선택하세요.

---

// turbo-all

## 실행 순서

### 1단계: 기존 서버 종료 (PowerShell)
```powershell
$proc = Get-NetTCPConnection -LocalPort 9000 -ErrorAction SilentlyContinue | Select-Object -ExpandProperty OwningProcess -Unique
if ($proc) { $proc | ForEach-Object { Stop-Process -Id $_ -Force -ErrorAction SilentlyContinue } }
Start-Sleep -Seconds 1
```

### 2단계: 새 서버 시작
```powershell
npx -y live-server --port=9000 --no-browser
```

서버 URL: http://localhost:9000
장점: 파일 변경 시 자동 새로고침(Live Reload) 지원

---

## 수동 실행 (터미널에서 직접)
```powershell
# 프로젝트 루트 폴더로 이동 후
cd D:\your\project\root
npx -y live-server --port=8000 --no-browser
```
서버 URL: http://localhost:8000

---

## 커스터마이징 가이드

### 포트 변경
위 명령어에서 `9000` 또는 `8000`을 원하는 포트 번호로 변경하세요.

### Python 서버 사용 시 (대안)
```powershell
python -m http.server 9000
```
> [!NOTE]
> Python 서버는 Live Reload를 지원하지 않습니다.
> 정적 파일 확인 용도로만 사용하세요.

### 특정 디렉토리만 서빙
```powershell
npx -y live-server --port=9000 --no-browser --entry-file=index.html ./dist
```

### CMD(명령 프롬프트)에서 기존 서버 종료
```cmd
for /f "tokens=5" %a in ('netstat -ano ^| findstr :9000') do taskkill /PID %a /F 2>nul
```
