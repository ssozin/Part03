---
description: Git 커밋 및 GitHub 푸시 자동 실행 (Windows 환경)
---

# 🚀 Git 자동 커밋 & 푸시 (Windows 환경)

> 이 워크플로우는 Git 커밋과 GitHub 푸시를 자동화합니다.
> 프로젝트 리포지토리 정보를 아래에 설정하고 사용하세요.

---

## 리포지토리 설정

> 프로젝트에 맞게 수정하세요.

| 항목 | 값 |
|------|-----|
| **리포 이름** | `my-project` |
| **GitHub URL** | `git@github.com:username/my-project.git` |
| **브랜치** | `main` |
| **배포 폴더** | `deploy\` (선택사항) |

---

// turbo-all

## 실행 순서

### 1단계: (선택) 빌드/배포 폴더 동기화
> deploy 폴더를 사용하는 경우에만 실행. 직접 커밋하는 경우 이 단계 생략.

```powershell
# 소스 → deploy 폴더 동기화 (Thumbs.db, desktop.ini 제외)
robocopy src\ deploy\src\ /MIR /XF Thumbs.db desktop.ini
robocopy public\ deploy\public\ /MIR /XF Thumbs.db desktop.ini
```

> [!NOTE]
> `robocopy`의 `/MIR` 옵션은 미러링(동기화)입니다.
> 종료 코드 0~3은 정상이므로 에러가 아닙니다.

### 2단계: 커밋 및 푸시
```powershell
# 변경사항 체크 후 커밋/푸시
$status = git status --porcelain
if ($status) {
    git add -A
    git commit -m "🔧 Auto commit"
    git push origin main 2>&1
} else {
    Write-Host "변경사항 없음"
}
```

---

## SSH 설정 (인증 팝업 방지)

### 문제
Windows에서 AI IDE 터미널로 `git push` 시 자격 증명 팝업이 뜨면서 대기 상태에 빠짐.

### 해결: SSH 키 사용

#### 1. SSH 키 생성
```powershell
New-Item -ItemType Directory -Force -Path .ssh_temp
ssh-keygen -t ed25519 -C "your-email@github" -f .ssh_temp\id_ed25519 -N '""'
```

#### 2. GitHub에 공개키 등록
```powershell
Get-Content .ssh_temp\id_ed25519.pub
# 출력된 키를 복사하여 GitHub > Settings > SSH Keys > New SSH key에 등록
```

#### 3. Remote URL을 SSH로 변경
```powershell
git remote set-url origin git@github.com:username/my-project.git
```

#### 4. SSH 키를 사용한 Push 명령
```powershell
$env:GIT_SSH_COMMAND = "ssh -i `"$PWD\.ssh_temp\id_ed25519`" -o IdentitiesOnly=yes -o StrictHostKeyChecking=no"
git push origin main 2>&1
```

> [!IMPORTANT]
> `2>&1`을 반드시 추가해야 AI IDE 터미널에서 출력이 표시되고 대기 현상이 방지됩니다.

### (대안) Windows 자격 증명 관리자 사용
HTTPS 방식을 사용하는 경우 Git Credential Manager가 자동으로 처리합니다:
```powershell
git config --global credential.helper manager
git remote set-url origin https://github.com/username/my-project.git
```

---

## 커밋 메시지 컨벤션

| 이모지 | 용도 | 예시 |
|--------|------|------|
| ✨ | 새 기능 | `✨ Add user profile page` |
| 🐛 | 버그 수정 | `🐛 Fix login redirect loop` |
| 🎨 | UI/스타일 | `🎨 Update button hover effect` |
| 🔧 | 설정/기타 | `🔧 Update config` |
| 📝 | 문서 | `📝 Update README` |
| 🚀 | 배포 | `🚀 Deploy v1.2.0` |
| ♻️ | 리팩토링 | `♻️ Refactor auth module` |
| 🗑️ | 삭제 | `🗑️ Remove unused files` |

> AI가 변경 내용에 맞는 커밋 메시지를 자동 작성합니다.

---

## 멀티 리포지토리 관리

여러 리포지토리를 관리하는 경우 아래 패턴을 복제하여 사용:

```powershell
# 리포 A
Push-Location project-a
$status = git status --porcelain
if ($status) { git add -A; git commit -m "🔧 Update"; git push origin main 2>&1 }
else { Write-Host "Project A: 변경사항 없음" }
Pop-Location

# 리포 B
Push-Location project-b
$status = git status --porcelain
if ($status) { git add -A; git commit -m "🔧 Update"; git push origin main 2>&1 }
else { Write-Host "Project B: 변경사항 없음" }
Pop-Location
```

---

## 주의사항
- ※ 커밋 메시지는 상황에 맞게 AI가 작성합니다.
- ※ 원격 리포지토리가 설정되지 않은 경우 먼저 `git remote add origin <URL>`을 실행합니다.
- ※ SSH 키 사용 시 `.ssh_temp\` 폴더를 `.gitignore`에 추가하세요.
- ※ PowerShell 실행 정책 오류 시 `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`를 실행하세요.
