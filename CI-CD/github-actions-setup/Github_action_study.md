# GitHub Actions 입문 - 팀 프로젝트 CI/CD 세팅

## 1. 목적

팀 프로젝트 협업을 위한 GitHub Actions 기초 학습 및 브랜치 정책 수립

---

## 2. 개념 정리

| Jenkins (구) | GitHub Actions (현) |
|---|---|
| Jenkinsfile | ci.yml |
| Pipeline | workflow |
| Gerrit | Pull Request |
| Jenkins UI | Actions 탭 |

GitHub Actions는 GitHub 서버(ubuntu)에서 코드를 자동으로 내려받아 테스트까지 실행해주는 CI 환경이다.  
Microsoft가 GitHub을 인수한 만큼 Azure와 네이티브 연동이 강점이다.

---

## 3. 기본 디렉토리 구조

```
project/
├── .github/
│   └── workflows/
│       └── ci.yml       ← GitHub Actions 핵심 파일
├── src/
│   ├── train.py
│   ├── model.py
│   └── utils.py
├── tests/
│   └── test_smoke.py
├── data/
├── notebooks/
├── requirements.txt
└── README.md
```

`.github/workflows/` 폴더는 아래 명령어로 생성한다.

```bash
mkdir -p .github/workflows
```

> ⚠️ `.github` 앞에 `.` 이 있어야 한다. GitHub가 이 경로를 자동으로 감지한다.

---

## 4. ci.yml 기본 형태

```yaml
name: CI

on:
  push:
    branches: ['**']        # 모든 브랜치 push 시 트리거
  pull_request:
    branches: ['main']      # main 대상 PR 시 트리거

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4        # GitHub 서버에 코드 내려받기
      - uses: actions/setup-python@v5    # Python 설치
        with:
          python-version: '3.14'         # 로컬 버전과 달라도 무관
      - run: pip install -r requirements.txt
      - run: pip install pytest
      - run: pytest tests/ -v
```

---

## 5. 전체 흐름

```
feature 브랜치에 commit & push
        ↓
GitHub Actions 자동 실행 (pytest)
        ↓
실패 → 빨간불 ❌ (PR 머지 차단)
성공 → 초록불 ✅ (PR 머지 가능)
        ↓
팀원 코드 리뷰 (LGTM)
        ↓
main 머지
```

### 커밋 & 푸시

```bash
git add .
git commit -m "feat: 기능명"
git push origin feature/이름-기능명
```

### PR 생성

push 후 GitHub 상단에 뜨는 **Compare & pull request** 클릭 → **Create pull request**

---

## 6. 브랜치 정책

### 브랜치 구조

```
main
├── feature/이름-기능A    ← 작업 시 생성, 머지 후 삭제
├── feature/이름-기능B
└── feature/이름-기능C
```

> `/` 는 디렉토리가 아니다. 브랜치 이름에 슬래시를 포함하면 GitHub이 시각적으로 그룹핑해줄 뿐이다.

### feature 브랜치 생성 방법

```bash
# 항상 main 최신 상태로 먼저 동기화
git checkout main
git pull origin main

# 그 다음 분기
git checkout -b feature/이름-기능명
```

> ⚠️ `git pull` 먼저 안 하면 옛날 main 기준으로 브랜치가 생성되어 나중에 충돌 발생

### main 최신 상태 유지 (작업 중 매일)

```bash
git checkout main
git pull origin main
git checkout feature/내브랜치
git merge origin/main
```

### 커밋 메시지 컨벤션

```
feat: 새 기능 추가
fix: 버그 수정
docs: 문서 수정
test: 테스트 추가
chore: 설정 변경 (ci.yml 수정 등)
```

---

## 7. main 브랜치 보호 설정

**Settings → Branches → Add branch ruleset**

| 설정 항목 | 내용 |
|---|---|
| Restrict deletions | main 브랜치 삭제 방지 |
| Require a pull request before merging | 직접 push 금지, PR 필수 |
| Required approvals: 2 | 2명 이상 리뷰 후 머지 가능 |
| Require status checks to pass | CI 통과 필수 |
| Block force pushes | 강제 push 방지 |

설정 후 main에 직접 push 시도하면 아래처럼 차단된다.

```bash
To https://github.com/username/repo.git
 ! [remote rejected] main -> main (push declined due to repository rule violations)
error: failed to push some refs to 'https://github.com/username/repo.git'
```

---

## 8. 발생한 에러 모음

### 에러 1: Private repo에서 CI 실패

```
fatal: could not read Username for 'https://github.com': terminal prompts disabled
```

- **원인**: Private repo라 GitHub Actions 서버가 코드를 읽지 못함
- **해결**: Settings → Danger Zone → Change visibility → **Public** 으로 변경
- **실무 대안**: Repository secret에 Personal Access Token 등록

### 에러 2: import 누락으로 pytest 실패

```
NameError: name 'say_bye' is not defined
test_hello.py:8: NameError
FAILED test_hello.py::test_say_bye
```

```python
# 잘못된 것
from hello import say_hello

# 고친 것
from hello import say_hello, say_bye
```

- **원인**: `say_bye` 함수를 추가했는데 import를 누락
- **해결**: import 수정 후 재커밋하면 CI 자동 재실행
- **교훈**: CI가 이런 실수를 자동으로 잡아준다. 로컬에서 pytest 먼저 돌리는 습관을 들이면 더 좋다.

---

## 9. 다음 단계 (팀 프로젝트 적용 시)

```yaml
# Azure 배포까지 붙이면 CD 완성
- uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}

- uses: azure/webapps-deploy@v2
  with:
    app-name: '앱이름'
```

Azure 세팅이 완료된 후 추가한다.
