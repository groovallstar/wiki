# Python Editable Install — PYTHONPATH 주입을 대체하는 uv/hatchling 설정 조사

- **조사 주제**: src 레이아웃 파이썬 프로젝트에서 쉘 스크립트·CI·IDE 실행마다 `PYTHONPATH=src ...`를 주입하는 관행을 없애고, 표준 패키징 메커니즘으로 대체하는 방법
- **조사 경로**: 공식 스펙(PEP) → 빌드 백엔드(hatchling) 문서 → 패키지 매니저(uv) 문서 → 관련 이슈·토론
- **유형**: 2차 자료 (복수 1차 문헌을 통합한 조사 노트)
- **보완 확인일**: 2026-10-28 (아래 명시한 공식 문서·규약 기준이며, 최초 조사일은 기록되어 있지 않다.)

## 조사 동기

전형적인 src 레이아웃 프로젝트는 다음 호출을 반복한다.

```bash
PYTHONPATH=src python -m my_package.cli ...
PYTHONPATH=src pytest tests/
```

- 쉘 세션·서브셸·IDE·도커마다 재설정 필요
- 깜빡하면 `ModuleNotFoundError`
- README·CI 스텝·스크립트 전반에 같은 접두어가 증식

이 불편을 "쉘 단에서 매번 주입"이 아니라 **`pyproject.toml` 한 번 설정으로 영속적으로 해결**할 수 있는지 조사했다.

## 조사 결과 요약

표준 해법은 **PEP 660 기반 editable install**이다. PEP 660은 editable wheel 인터페이스를 정의하며 실제 소스 연결 방식은 백엔드가 정한다. 이 노트의 Hatchling 기본 개발 모드와 `src` 레이아웃 예시는 `.pth`로 소스 경로를 연결한다. 설치한 환경의 Python을 사용하면 PYTHONPATH 주입이 불필요해진다.

## 주요 내용 — 1차 문헌별 근거

### PEP 660 (2021) — Editable installs for pyproject.toml based builds

- 링크: https://peps.python.org/pep-0660/ (2026-10-28 확인)
- 저자: Daniel Holth, Stéphane Bidoul
- 빌드 백엔드가 구현해야 하는 `build_editable(wheel_directory, ...)` 훅을 정의.
- 다음 세 가지 구현 예시를 설명하며 선택은 백엔드 재량이다. 이 목록은 규범적인 제한이 아니다.
  1. `.pth` 파일 — 소스 디렉토리 절대경로를 site-packages의 `.pth`에 기재
  2. 프록시 모듈 — 런타임 import redirect (PEP 660 import hook)
  3. 심볼릭 링크
- 트레이드오프: `.pth`는 단순·빠르고 도구 호환 양호. 프록시/import hook은 엄격한 경계 제공하나 정적 분석기와 충돌.
- 기존 `setup.py develop` 의 비표준 관행을 대체하는 빌드 백엔드 중립 표준.
- Python 소스 수정은 재설치 없이 반영되지만 의존성·진입점 변경은 재설치가, 확장 모듈 변경은 별도 빌드가 필요할 수 있다. editable 설치에서 일반 wheel에 없는 파일이 노출될 수도 있다.

### Hatchling — Build Configuration

- 링크: https://hatch.pypa.io/latest/config/build/ (2026-10-28 확인, 특히 `#dev-mode`)
- 보조 링크: https://github.com/pypa/hatch/discussions/827 (editable 기본 메커니즘 메인테이너 확인)
- `[tool.hatch.build.targets.wheel]` 의 `packages = ["src/foo"]` 는 `only-include = ["src/foo"]` + `sources = ["src"]` 로 확장된다. 즉 **`src/` 접두어를 자동으로 벗겨** wheel 안에서는 `foo/` 로 배치되고, `import foo` 로 사용된다.
- editable install 기본 산출물: site-packages에 **`.pth` 파일 1개**가 생성되며 프로젝트의 `src/` 디렉토리 절대경로를 담는다 (discussion #827에서 메인테이너 확인).
- `dev-mode-exact = true` 옵션은 PEP 660 import-hook 방식으로 전환하나, **Pylance/pyright 등 정적 분석기가 경로를 해석하지 못하는** 이슈로 기본값이 아니다.
- `dev-mode-dirs` 는 editable 모드에서 `sys.path`에 추가할 디렉토리를 명시적으로 지정하는 키.

### uv — Project Configuration

- 링크: https://docs.astral.sh/uv/concepts/projects/config/ (2026-10-28 확인)
- 동기화·lock 검사: https://docs.astral.sh/uv/concepts/projects/sync/ (2026-10-28 확인)
- 프로젝트 환경·lock 파일: https://docs.astral.sh/uv/concepts/projects/layout/ (2026-10-28 확인)
- 보조 링크: https://github.com/astral-sh/uv/issues/3898 (PEP 660 import hook과 Pylance 비호환)
- `[build-system]` 섹션이 존재하면 uv는 프로젝트를 **자동으로 editable install** 한다. 따라서 `[tool.uv] package = true` 는 해당 상황에서 **중복(no-op)** 이다.
- `[build-system]` 이 없는데 `tool.uv.package = true` 로 지정하면 uv는 `setuptools` 레거시 백엔드를 사용한다.
- `tool.uv.package = false` 는 프로젝트 자체의 설치를 **억제** 한다 (순수 의존성 선언용 `pyproject.toml` 로 취급).
- `uv sync` / `uv pip install -e .` 둘 다 PEP 660 훅을 통해 동일한 editable 산출물을 생성한다. 권장 워크플로는 `uv.lock` 기반 `uv sync`.
- PEP 660 import hook 방식은 Pylance/pyright 호환성 이슈가 보고돼 있다 (uv#3898, pylance#3407). 기본 `.pth` 방식은 영향 없음.
- `uv sync`는 기본적으로 선택되지 않은 기존 설치 패키지를 제거한다. `dev` 그룹은 기본 포함되며 그룹 제외 옵션을 줄 수 있다. `package = false`에서 `true`로 바꿨다는 사실 자체가 개발 도구 제거 원인은 아니다.
- 내부 Python 패키지 추가 자체는 보통 lock 변경을 요구하지 않는다. lock 최신 여부는 프로젝트 메타데이터에도 의존하며 명시적 upgrade로 갱신할 수도 있으므로, `dependencies` 목록 변경만이 갱신 조건은 아니다.

### Python — 가상환경과 namespace package

- venv: https://docs.python.org/3/library/venv.html#how-venvs-work (2026-10-28 확인)
- site: https://docs.python.org/3/library/site.html (2026-10-28 확인)
- PEP 420: https://peps.python.org/pep-0420/#specification (2026-10-28 확인)
- 환경의 Python을 절대경로로 실행하면 셸 활성화 없이도 해당 venv를 사용할 수 있다. 활성화는 PATH를 바꾸는 편의 기능이며, 이미 실행 중인 Python의 import 경로를 바꾸지 않는다. uv 프로젝트에서는 `uv run`으로 환경을 선택할 수 있다.
- `.pth`의 경로 줄은 해당 인터프리터의 `site` 처리로 반영된다. 존재하지 않는 경로·빈 줄·주석은 제외되며, `-S`는 `site` 자동 처리를 끈다.
- 일반 패키지와 달리 암묵적 namespace package는 `__init__.py`를 요구하지 않는다. import 가능 여부와 빌드 백엔드의 배포 파일 선택은 구분하여 확인한다.

### poetry-core — 빌드 백엔드의 독립성

- 링크: https://github.com/python-poetry/poetry-core#usage (2026-10-28 확인)
- `poetry-core`는 Poetry에서 분리된 PEP 517 빌드 백엔드다. `pip` 등 호환 frontend가 Poetry 자체 없이 사용할 수 있으므로, 백엔드를 lock 파일과 강결합한 것으로 설명하지 않는다.

## 결론 — 권장 구성

`pyproject.toml`에 다음 몇 줄로 PYTHONPATH 주입 관행을 완전히 제거할 수 있다.

```toml
[tool.hatch.build.targets.wheel]
packages = ["src/my_package", "src/my_other_package"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

- 이 구성에서 `uv sync`는 프로젝트를 기본적으로 editable 설치한다. 실행 시에는 `uv run` 등으로 같은 환경의 Python을 선택한다.
- 이 Hatchling 기본 경로 방식은 `.pth`를 사용한다. 다른 백엔드·개발 모드의 성능이나 정적 분석기 호환성까지 보장하는 설명은 아니다.
- `[tool.uv] package = true` 는 `[build-system]` 이 있으면 생략 가능.
- `dev-mode-exact = true` 는 IDE 호환성 때문에 권장하지 않는다.

## 인용하는 위키 페이지

- `concepts/src-layout-packaging.md` — editable install 메커니즘과 `src` 레이아웃 운영 원리
