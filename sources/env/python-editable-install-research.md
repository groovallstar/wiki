# Python Editable Install — PYTHONPATH 주입을 대체하는 uv/hatchling 설정 조사

- **조사 주제**: src 레이아웃 파이썬 프로젝트에서 쉘 스크립트·CI·IDE 실행마다 `PYTHONPATH=src ...`를 주입하는 관행을 없애고, 표준 패키징 메커니즘으로 대체하는 방법
- **조사 경로**: 공식 스펙(PEP) → 빌드 백엔드(hatchling) 문서 → 패키지 매니저(uv) 문서 → 관련 이슈·토론
- **유형**: 2차 자료 (복수 1차 문헌을 통합한 조사 노트)

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

표준 해법은 **PEP 660 기반 editable install**이다. `pip install -e .` 또는 `uv sync` 가 빌드 백엔드에 위임해 site-packages에 `.pth` 파일을 남기면, 파이썬 인터프리터가 기동할 때 `site.py`가 그 경로를 `sys.path`에 자동 추가한다. 결과적으로 PYTHONPATH 주입이 불필요해진다.

## 주요 내용 — 1차 문헌별 근거

### PEP 660 (2021) — Editable installs for pyproject.toml based builds

- 링크: https://peps.python.org/pep-0660/
- 저자: Daniel Holth, Stéphane Bidoul
- 빌드 백엔드가 구현해야 하는 `build_editable(wheel_directory, ...)` 훅을 정의.
- **세 가지 허용 메커니즘**을 명시하며 선택은 백엔드 재량:
  1. `.pth` 파일 — 소스 디렉토리 절대경로를 site-packages의 `.pth`에 기재
  2. 프록시 모듈 — 런타임 import redirect (PEP 660 import hook)
  3. 심볼릭 링크
- 트레이드오프: `.pth`는 단순·빠르고 도구 호환 양호. 프록시/import hook은 엄격한 경계 제공하나 정적 분석기와 충돌.
- 기존 `setup.py develop` 의 비표준 관행을 대체하는 빌드 백엔드 중립 표준.

### Hatchling — Build Configuration

- 링크: https://hatch.pypa.io/latest/config/build/
- 보조 링크: https://github.com/pypa/hatch/discussions/827 (editable 기본 메커니즘 메인테이너 확인)
- `[tool.hatch.build.targets.wheel]` 의 `packages = ["src/foo"]` 는 `only-include = ["src/foo"]` + `sources = ["src"]` 로 확장된다. 즉 **`src/` 접두어를 자동으로 벗겨** wheel 안에서는 `foo/` 로 배치되고, `import foo` 로 사용된다.
- editable install 기본 산출물: site-packages에 **`.pth` 파일 1개**가 생성되며 프로젝트의 `src/` 디렉토리 절대경로를 담는다 (discussion #827에서 메인테이너 확인).
- `dev-mode-exact = true` 옵션은 PEP 660 import-hook 방식으로 전환하나, **Pylance/pyright 등 정적 분석기가 경로를 해석하지 못하는** 이슈로 기본값이 아니다.
- `dev-mode-dirs` 는 editable 모드에서 `sys.path`에 추가할 디렉토리를 명시적으로 지정하는 키.

### uv — Project Configuration

- 링크: https://docs.astral.sh/uv/concepts/projects/config/
- 보조 링크: https://github.com/astral-sh/uv/issues/3898 (PEP 660 import hook과 Pylance 비호환)
- `[build-system]` 섹션이 존재하면 uv는 프로젝트를 **자동으로 editable install** 한다. 따라서 `[tool.uv] package = true` 는 해당 상황에서 **중복(no-op)** 이다.
- `[build-system]` 이 없는데 `tool.uv.package = true` 로 지정하면 uv는 `setuptools` 레거시 백엔드를 사용한다.
- `tool.uv.package = false` 는 프로젝트 자체의 설치를 **억제** 한다 (순수 의존성 선언용 `pyproject.toml` 로 취급).
- `uv sync` / `uv pip install -e .` 둘 다 PEP 660 훅을 통해 동일한 editable 산출물을 생성한다. 권장 워크플로는 `uv.lock` 기반 `uv sync`.
- PEP 660 import hook 방식은 Pylance/pyright 호환성 이슈가 보고돼 있다 (uv#3898, pylance#3407). 기본 `.pth` 방식은 영향 없음.

## 결론 — 권장 구성

`pyproject.toml`에 다음 몇 줄로 PYTHONPATH 주입 관행을 완전히 제거할 수 있다.

```toml
[tool.hatch.build.targets.wheel]
packages = ["src/my_package", "src/my_other_package"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

- `uv sync` 한 번이면 editable install 완료.
- `.pth` 방식이 기본이라 런타임 오버헤드 사실상 0, 정적 분석기 호환.
- `[tool.uv] package = true` 는 `[build-system]` 이 있으면 생략 가능.
- `dev-mode-exact = true` 는 IDE 호환성 때문에 권장하지 않는다.

## 인용하는 위키 페이지

- `concepts/src-layout-packaging.md` — editable install 메커니즘과 `src` 레이아웃 운영 원리
