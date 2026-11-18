# pyenv / pyenv-virtualenv 사용법

- **pyenv**: Python 자체의 버전(3.12, 3.13 …)을 설치하고 고른다.
- **pyenv-virtualenv**: 그 Python 위에 프로젝트별 패키지 공간(가상환경)을 만든다.

## 1. Python 버전 관리 (pyenv)

| 명령어 | 설명 |
| --- | --- |
| `pyenv install --list` | 설치 가능한 버전 목록 |
| `pyenv install 3.13` | 3.13 계열 최신 패치 설치 |
| `pyenv uninstall 3.13.18` | 버전 삭제 |
| `pyenv versions` | 설치된 버전·가상환경 목록 (`*`가 현재 적용) |
| `pyenv version` | 현재 적용된 버전과 그 근거 파일 |

## 2. 어떤 버전을 쓸지 지정

| 명령어 | 범위 | 저장 위치 |
| --- | --- | --- |
| `pyenv global 3.13.18` | 전체 기본값 | `~/.pyenv/version` |
| `pyenv local rag-study` | 현재 폴더와 하위 폴더 | `./.python-version` |
| `pyenv shell 3.12.7` | 현재 터미널만 | 환경변수 `PYENV_VERSION` |

우선순위: `shell` > `local` > `global`

해제는 `--unset`을 붙인다. 예: `pyenv local --unset`

## 3. 가상환경 (pyenv-virtualenv)

| 명령어 | 설명 |
| --- | --- |
| `pyenv virtualenv 3.13.18 rag-study` | 3.13.18 기반 가상환경 `rag-study` 생성 |
| `pyenv virtualenvs` | 가상환경 목록 |
| `pyenv local rag-study` | 이 폴더에 들어오면 자동 활성화 |
| `pyenv activate rag-study` | 수동 활성화 |
| `pyenv deactivate` | 현재 터미널에서 비활성화 |
| `pyenv virtualenv-delete rag-study` | 가상환경 삭제 |

## 4. 새 프로젝트 시작 순서

```bash
pyenv install 3.13                      # 필요한 버전이 없을 때만
pyenv virtualenv 3.13.18 my-project
cd my-project
pyenv local my-project
pip install -r requirements.txt
```

## 5. 적용 여부 확인

```bash
pyenv version                                  # rag-study (set by .../.python-version)
python -c "import sys; print(sys.prefix)"      # ~/.pyenv/versions/rag-study
```

프롬프트에 `🐍 rag-study`가 보이면 활성화된 상태다.

## 6. 패키지 관리

| 명령어 | 설명 |
| --- | --- |
| `pip install <패키지>` | 현재 가상환경에 설치 |
| `pip list` | 설치된 패키지 목록 |
| `pip freeze > requirements.txt` | 현재 패키지 목록을 파일로 저장 |
| `pip install -r requirements.txt` | 파일 기준으로 일괄 설치 |

## 7. 자주 겪는 문제

- **`pyenv local <이름>`이 안 됨**: 그 이름의 버전·가상환경이 없다. `pyenv versions`로 확인하고 먼저 `pyenv virtualenv`로 만든다.
- **`python`이 다른 버전으로 잡힘**: `pyenv version`으로 어느 파일이 지정했는지 확인한다. 상위 폴더의 `.python-version`이 적용됐을 수 있다.
- **설정을 바꿨는데 반영 안 됨**: `source ~/.zshrc`를 실행하거나 터미널을 새로 연다.
- **셸 설정 위치**: `~/.zshrc`의 `pyenv init`, `pyenv virtualenv-init` 두 줄이 있어야 자동 활성화가 동작한다.
