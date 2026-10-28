# Editable Install — `src` 레이아웃 파이썬 프로젝트에서 PYTHONPATH 없애기

> 목적: 여러 파이썬 프로젝트·가상환경을 다루는 개발자가 `PYTHONPATH=src python ...`처럼 import 경로를 매 호출마다 주입하지 않고도 모듈을 찾게 만드는 방법을 정리한다. 배경 개념(빌드 백엔드·wheel·editable install)과 가상환경 격리 원리를 함께 다룬다.

## 목차

1. [문제 정의](#1-문제-정의)
2. [왜 PYTHONPATH 주입이 필요한가 — `src` 레이아웃의 구조](#2-왜-pythonpath-주입이-필요한가--src-레이아웃의-구조)
3. [Editable install이란](#3-editable-install이란)
4. [pyproject.toml · 빌드 백엔드 · wheel 의 관계](#4-pyprojecttoml--빌드-백엔드--wheel-의-관계)
5. [설정 예시 (uv + hatchling)](#5-설정-예시-uv--hatchling)
6. [`.pth` 파일과 `sys.path` 주입 메커니즘](#6-pth-파일과-syspath-주입-메커니즘)
7. [가상환경 격리 — 여러 venv에서의 동작](#7-가상환경-격리--여러-venv에서의-동작)
8. [네 가지 경로 주입 방식 비교](#8-네-가지-경로-주입-방식-비교)
9. [명시형 vs 자동 탐색 — 새 서브패키지 추가 시 운영비용](#9-명시형-vs-자동-탐색--새-서브패키지-추가-시-운영비용)
10. [흔한 오해와 주의점](#10-흔한-오해와-주의점)

---

## 1. 문제 정의

전형적인 파이썬 프로젝트에서 다음과 같은 호출을 반복하게 된다.

```bash
PYTHONPATH=src python -m my_package.cli ...
PYTHONPATH=src pytest tests/
```

- 쉘 스크립트, README 예시, CI 스텝 어디에나 같은 접두어가 반복된다.
- 쉘 세션마다 다시 붙여야 한다 (깜빡하면 `ModuleNotFoundError`).
- 서브셸·IDE 테스트 러너·도커 실행마다 재설정이 필요하다.

근본 원인은 파이썬이 프로젝트 소스 디렉토리를 자동으로 import 경로에 포함하지 않는다는 점이다. **editable install**은 이 문제를 표준 파이썬 패키징 메커니즘으로 해결한다.

---

## 2. 왜 PYTHONPATH 주입이 필요한가 — `src` 레이아웃의 구조

많은 프로젝트는 "flat 레이아웃"과 "src 레이아웃" 중 후자를 선호한다.

```
# src 레이아웃
project/
├── pyproject.toml
├── src/
│   ├── my_package/
│   │   └── __init__.py
│   └── my_other_package/
│       └── __init__.py
└── tests/
```

파이썬은 현재 디렉토리(`''`)를 `sys.path`에 자동으로 넣지만 `src/`는 넣지 않는다. `project/` 루트에서 `python -c "import my_package"`를 실행하면 `my_package`가 있는 곳은 `src/my_package/`이므로 찾지 못한다.

**src 레이아웃의 장점:**
- 소스 루트에서 실수로 import하는 문제 방지 (개발 중인 코드와 설치된 코드의 경계가 명확)
- 테스트가 "설치된 버전"을 사용하므로 배포 상태와 일치

**대가:** 설치 없이 import가 불가능 — 그래서 `PYTHONPATH=src`로 강제 주입하거나 **editable install**을 한다.

---

## 3. Editable install이란

> **핵심:** 프로젝트를 설치된 패키지로 등록하되 Python 코드를 소스 트리에서 불러오도록 연결한다. PEP 660은 editable wheel 인터페이스를 정의하며, 연결 방식은 빌드 백엔드가 정한다. Python 소스 수정은 재설치 없이 반영되지만 의존성·진입점 변경이나 확장 모듈은 별도 설치·빌드가 필요할 수 있다.

### 일반 install vs editable install

| | 일반 install (`pip install .`) | editable install (`pip install -e .`) |
|---|---|---|
| site-packages에 들어가는 것 | 소스 파일 복사본 | 메타데이터와 소스 연결 장치(`.pth`·프록시 등) |
| 소스 수정 반영 | 재설치 필요 | 즉시 반영 |
| 용도 | 배포/운영 | 개발 |
| 디스크 공간 | 복사본만큼 추가 | 미미 |

### 무엇이 설치되는가

이 문서의 Hatchling 기본 개발 모드와 `src` 레이아웃 예시에서는 가상환경의 `site-packages/` 안에 다음과 같은 파일이 생긴다. 다른 백엔드나 개발 모드는 프록시·심볼릭 링크 등 다른 연결 방식을 사용할 수 있다.

```
venv/lib/pythonX.Y/site-packages/
├── my_project-0.1.0.dist-info/       # 메타데이터
├── _my_project.pth                    # ← 포인터 파일
└── (기타 의존성들)
```

이 예시의 `.pth` 파일은 프로젝트 소스 디렉토리의 **절대경로**를 담는다.

```
/abs/path/to/project/src
```

이 경로는 파이썬 인터프리터가 기동할 때 자동으로 `sys.path`에 추가된다. 자세한 메커니즘은 §6 참조.

---

## 4. pyproject.toml · 빌드 백엔드 · wheel 의 관계

editable install을 이해하려면 현대 파이썬 패키징의 3계층을 알아야 한다.

```
패키지 매니저 (uv, pip)
        ↓
pyproject.toml  ── [build-system] ──→  빌드 백엔드 (hatchling, setuptools, poetry-core, ...)
        ↓                                      ↓
    의존성 resolve                      "내 소스를 어떻게 wheel로 만들지"
        ↓                                      ↓
     설치 실행 ← ← ← ← ← ← ← ← ← ← ←  wheel (.whl) 또는 editable proxy
        ↓
  가상환경 site-packages
```

| 레이어 | 역할 |
|--------|------|
| **패키지 매니저** | 의존성 그래프 해결, lock 파일 관리, 가상환경 관리 |
| **pyproject.toml** | 프로젝트 메타데이터 + 빌드 설정 (PEP 621) |
| **빌드 백엔드** | 소스 트리를 설치 가능한 아티팩트(wheel)로 변환 |
| **wheel** | 설치 가능한 ZIP 포맷 파일 — 파이썬 표준 배포 포맷 |

### 빌드 백엔드 선택지

| 백엔드 | 특징 |
|--------|------|
| **hatchling** | 경량, 설정 간결, PyPA 공식 권장 중 하나 |
| **setuptools** | 가장 오래됨, 방대한 기능, 복잡한 설정 |
| **poetry-core** | Poetry의 빌드 백엔드이며 다른 PEP 517 frontend에서도 독립적으로 사용 가능 |
| **flit-core** | 순수 파이썬 단일 패키지에 최적 |

본 문서는 hatchling을 예로 든다. 개념은 어느 백엔드든 동일하다.

### 패키지 매니저와 빌드 백엔드의 분리

`uv`, `pip` 같은 도구는 직접 wheel을 만들지 않고 **`pyproject.toml`의 `[build-system]` 섹션을 읽어 해당 백엔드에게 위임**한다.

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

이 분리 덕분에 같은 프로젝트를 uv·pip·poetry 어떤 도구로 설치해도 동일한 결과가 나온다.

---

## 5. 설정 예시 (uv + hatchling)

src 레이아웃 프로젝트를 editable install 가능하게 만드는 최소 설정:

```toml
[project]
name = "my-project"
version = "0.1.0"
dependencies = [...]

[tool.uv]
package = true                          # uv에게 "이 프로젝트 자체도 설치 대상"이라고 알림

[tool.hatch.build.targets.wheel]
packages = [
    "src/my_package",
    "src/my_other_package",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

설치:

```bash
uv sync
# 또는
uv pip install -e .
```

설치한 환경의 Python을 사용하면 PYTHONPATH 없이 import할 수 있다. 셸 활성화 없이 프로젝트 환경을 선택하려면 `uv run`을 사용한다.

```bash
uv run python -c "import my_package"
uv run python -m my_package.cli ...
uv run pytest tests/
```

### `[tool.uv] package` 설정의 의미

| 값 | 의미 |
|----|------|
| `false` | "이 `pyproject.toml`은 의존성 선언용일 뿐, 프로젝트 자체는 설치하지 않음" |
| `true` | "이 프로젝트도 패키지다. `uv sync` 시 editable install 수행" |

### `[tool.hatch.build.targets.wheel]` 핵심 키

| 키 | 역할 |
|----|------|
| `packages` | 어떤 디렉토리를 파이썬 패키지로 포함할지 명시 |
| `only-include` | wheel에 포함할 경로 화이트리스트 |
| `sources` | 경로 매핑 시 벗길 접두어 (`src/my_package` → `my_package`) |

`packages = ["src/my_package"]`를 쓰면 hatchling이 자동으로 `src/` 접두어를 벗겨 wheel 안에서는 `my_package/`로 배치한다.

---

## 6. `.pth` 파일과 `sys.path` 주입 메커니즘

Hatchling 기본 개발 모드의 경로 연결에는 파이썬 표준 기능인 `.pth` 파일이 쓰인다. 이것이 PEP 660 전체의 유일한 구현 방식은 아니다.

### `.pth` 파일 규약

- 위치: 가상환경의 `site-packages/` 아래
- 내용: 추가할 경로를 줄 단위로 나열하며, 이 예시에서는 절대경로를 사용한다.
- 파이썬 인터프리터가 기동할 때 `site` 모듈이 `site-packages/*.pth`를 자동 스캔
- 존재하는 경로를 `sys.path`에 추가하며 빈 줄과 주석은 건너뛴다. `site` 자동 처리를 끄는 `-S` 실행은 이 설명의 대상에서 제외한다.

예시:

```
# /venv/lib/python3.13/site-packages/_my_project.pth
/abs/path/to/project/src
```

결과적으로 인터프리터가 시작된 직후의 `sys.path`는 다음과 같다.

```python
import sys
print(sys.path)
# ['', '/venv/lib/python3.13/site-packages',
#  '/abs/path/to/project/src', ...]
#   ↑ .pth로 추가됨
```

이 상태에서 `import my_package`를 하면 파이썬이 `sys.path`를 순회하며 `/abs/path/to/project/src/my_package/__init__.py`를 찾아낸다.

### 왜 PYTHONPATH 없이 동작하는가

- PYTHONPATH는 **쉘 환경변수 → `sys.path` 주입** 경로
- `.pth`는 **site-packages 파일 → `sys.path` 주입** 경로
- 두 방법 모두 파이썬의 표준 import 메커니즘에 훅을 건 것
- 이 문서의 Hatchling 기본 개발 모드는 후자를 자동으로 설정한다.

---

## 7. 가상환경 격리 — 여러 venv에서의 동작

> 자주 나오는 우려: "editable install을 하면 호스트 전체에 경로가 고정되는 건 아닌가?"
> **답: 아니다. `.pth` 파일이 venv의 site-packages에 들어가므로 영향은 해당 venv로 한정된다.**

### 시나리오 1 — 같은 프로젝트를 여러 venv에 설치

```
프로젝트 루트: /work/my-project/

venv-A/lib/python3.13/site-packages/_my_project.pth
    → /work/my-project/src

venv-B/lib/python3.13/site-packages/_my_project.pth
    → /work/my-project/src
```

두 venv 모두 같은 경로를 가리킨다. 소스 수정은 양쪽에서 즉시 보인다.

### 시나리오 2 — 다른 프로젝트에 설치된 다른 venv

```
venv-A (my-project): /work/my-project/src를 가리키는 .pth
venv-C (other-project): .pth 없음 — my-project와 무관
```

venv-C를 activate해도 my-project 코드는 `sys.path`에 **안 들어간다**. venv-C의 site-packages에는 my-project `.pth`가 없기 때문.

### 시나리오 3 — 여러 클론을 여러 venv에 설치

```
/work/my-project-dev/    + venv-dev   → .pth에 /work/my-project-dev/src
/work/my-project-rc/     + venv-rc    → .pth에 /work/my-project-rc/src
```

각 venv가 자기 클론을 가리킨다. 서로 간섭 없음. 같은 패키지 이름이라도 상관 없다.

### 핵심 원리

- `.pth` 파일은 **절대경로**를 담지만 **자기가 속한 venv 내부에서만 효력**을 갖는다.
- 어떤 환경의 `site-packages`를 읽는지는 실행한 Python 인터프리터가 정한다. 셸 활성화는 PATH를 바꾸는 편의 기능이며 `uv run`이나 venv Python의 절대경로로도 그 환경을 사용할 수 있다.
- 결과적으로 editable install은 venv 격리 경계를 깨지 않는다.

### 전역 파이썬(venv 밖)에 editable install하면?

- `.pth`가 **시스템 파이썬의 site-packages**에 들어간다 → 해당 파이썬을 쓰는 모든 프로세스가 영향받는다.
- **이것은 venv를 쓰지 않은 사용자의 실수이지 editable install 자체의 결함은 아니다.**
- 개발 환경에서는 설치 대상과 실행 인터프리터를 같은 venv로 맞춘다. 셸 활성화는 필수가 아니다.

---

## 8. 네 가지 경로 주입 방식 비교

| 방식 | 영향 범위 | 다른 venv에 영향? | 설정 난이도 |
|------|-----------|-------------------|-------------|
| **`PYTHONPATH=...` 1회 호출** | 해당 쉘 명령 한 번 | ❌ | 낮음 (매번 입력) |
| **`export PYTHONPATH=...` in shell rc** | 쉘 로그인 전체 | ✅ **오염 큼** | 낮음 (1회 설정) |
| **`.envrc` + direnv** | 해당 디렉토리 진입 시 | ❌ | 중간 (direnv 설치) |
| **editable install (`.pth`)** | 해당 venv 내부 | ❌ | 중간 (pyproject 설정 1회) |

### 왜 editable install이 권장되는가

- **표준 파이썬 메커니즘** — 추가 도구 불필요 (direnv 등)
- **venv 격리 준수** — 다른 프로젝트 오염 없음
- **의도 명시** — `pip list`에 프로젝트가 나타나므로 "이 환경이 이 프로젝트를 다룬다"는 사실이 드러남
- **CI/배포와 같은 패키징 설정 활용** — editable 설치와 일반 wheel의 파일 노출은 다를 수 있으므로 배포 검증에서는 일반 설치도 확인한다.
- **동료 온보딩 단순** — `git clone` 후 `uv sync` 한 줄이면 끝

### 언제 PYTHONPATH가 여전히 적절한가

- 일회성 디버깅 (패키지 설치 없이 특정 소스를 잠깐 import)
- 패키지로 구조화되지 않은 스크립트 모음 (단일 폴더에 독립 `.py` 파일들)
- 빌드 가능한 `pyproject.toml`이 없거나 작성할 수 없는 환경

---

## 9. 명시형 vs 자동 탐색 — 새 서브패키지 추가 시 운영비용

`[tool.hatch.build.targets.wheel]`의 두 가지 접근.

### 명시형 (`packages = [...]`)

```toml
[tool.hatch.build.targets.wheel]
packages = [
    "src/my_package",
    "src/my_other_package",
]
```

- 장점: 포함 대상이 명확, 패키지 아닌 디렉토리(`docs/`, `scripts/` 등)가 혼입될 위험 없음
- 단점: **새 최상위 패키지 추가 시 한 줄 추가 필요** (서브패키지 추가는 무관)

### 자동 탐색 (`only-include` + `sources`)

```toml
[tool.hatch.build.targets.wheel]
only-include = ["src"]
sources = ["src"]
```

- 장점: `src/` 아래 모든 패키지 자동 포함, 새 최상위 패키지 추가 시 설정 변경 불필요
- 단점: `src/` 아래 **패키지 아닌 디렉토리**(예: `src/scripts/`, `src/docs/`)까지 wheel에 들어갈 수 있음. 실수로 데이터·스크립트가 배포됨

### 선택 기준

- `src/` 아래 **모든 것이 패키지**이고 앞으로도 그럴 계획 → 자동 탐색
- `src/` 아래 **비패키지 리소스가 섞여 있음** → 명시형 (혼입 방지)

위 파일 선택 방식의 변경이나 내부 Python 패키지 추가 자체는 보통 `uv.lock` 갱신을 요구하지 않는다. 의존성이나 다른 프로젝트 메타데이터가 함께 바뀌었다면 별도로 확인한다.

---

## 10. 흔한 오해와 주의점

### 오해 1: "editable install하면 다른 venv에도 영향이 간다"

`.pth` 파일은 venv의 `site-packages`에 들어간다. 다른 venv의 site-packages에는 없으므로 영향 없다. §7 참조.

### 오해 2: "패키지 하나 추가하면 `uv.lock`을 재생성해야 한다"

내부 Python 서브패키지 추가 자체는 보통 lock 파일 변경을 요구하지 않는다. 그러나 lock 최신 여부는 프로젝트 메타데이터에도 의존하며, `uv lock --upgrade` 같은 명시적 갱신도 가능하다. `dependencies` 목록 변경만이 갱신 조건은 아니다.

### 오해 3: "editable install은 성능이 느리다"

Hatchling 기본 경로 방식에서는 파이썬이 `.pth`로 추가된 경로를 평범한 `sys.path` 항목으로 취급한다. 이 설명을 프록시 등 다른 editable 구현의 성능 보장으로 일반화하지 않는다.

### 오해 4: "editable install을 한 venv에서 `import`하면 영원히 그 경로에 고정된다"

새로 실행한 Python은 자신이 속한 환경의 `.pth`를 읽는다. 셸 활성화·해제는 이후 명령의 인터프리터 선택에 영향을 주며, 이미 실행 중인 Python의 import 경로나 모듈을 바꾸지 않는다.

### 주의 1: 전역 파이썬에 editable install하지 말 것

시스템 Python에 editable 설치하면 그 Python이 검색하는 설치 위치에 소스 연결이 추가될 수 있다. 모든 Python 인터프리터에 적용되는 것은 아니지만 환경 간 충돌을 피하려면 설치 대상과 실행 인터프리터를 개발용 venv로 맞춘다.

### 주의 2: `__init__.py` 없는 디렉토리

일반 패키지는 `__init__.py`를 사용한다. PEP 420의 암묵적 namespace package는 `__init__.py` 없이도 import할 수 있으므로, 의도한 패키지 유형과 빌드 백엔드의 배포 파일 선택을 함께 확인한다.

### 주의 3: 개발 의존성이 사라질 수 있음

`uv sync`는 기본적으로 설치 대상으로 선택되지 않은 기존 패키지를 제거한다. 개발 도구는 `[dependency-groups] dev` 등에 선언하고 그룹 선택 옵션을 확인한다. `dev` 그룹은 기본으로 동기화되며, `package = false` → `true` 전환 자체나 프로덕션 의존성에 없다는 사실만으로 제거를 단정하지 않는다.

### 주의 4: wheel 경로와 sources 접두어

`packages = ["src/my_package"]`만 쓰면 hatchling은 자동으로 `src/` 접두어를 벗긴다. 하지만 `only-include = ["src"]`를 쓸 때는 `sources = ["src"]`를 함께 지정하지 않으면 wheel 안에 `src/my_package/`로 들어가 `import src.my_package` 같은 이상한 경로가 된다.

---

## 요약

1. src 레이아웃 프로젝트에서 `PYTHONPATH=src`를 매번 주입하는 것은 editable install로 영구 해결할 수 있다.
2. PEP 660의 연결 방식은 백엔드가 정하며, 이 문서의 Hatchling 기본 개발 모드는 `.pth`로 소스 경로를 연결한다.
3. 환경 격리는 실행한 인터프리터를 기준으로 이해하며, 셸 활성화 없이 `uv run`으로도 프로젝트 환경을 선택할 수 있다.
4. Hatchling 파일 선택과 `[build-system]`을 설정하면 uv가 기본적으로 프로젝트를 editable 설치한다. `package = true`만으로 파일 선택 설정을 대신하지는 못한다.
5. 내부 Python 패키지 추가 자체는 보통 lock 갱신을 요구하지 않지만, 메타데이터 변경과 명시적 갱신은 별도로 확인한다.
6. 개발 도구는 필요한 의존성 그룹에 선언하고 `uv sync`의 패키지 선택·제거 동작을 확인한다.

---

## 출처

- `sources/env/python-editable-install-research.md` — PEP 660 스펙, hatchling 기본 `.pth` 메커니즘, uv의 자동 editable 처리 동작을 통합 정리한 조사 노트.
