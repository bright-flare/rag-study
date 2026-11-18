# Jupyter Notebook 설치·사용법

가상환경 `rag-study`가 활성화된 상태(프롬프트에 `🐍 rag-study`)에서 진행한다. 가상환경 사용법은 [pyenv.md](pyenv.md) 참고.

## 1. 설치

```bash
pip install notebook
```

탭·파일 탐색기·터미널이 포함된 최신 UI를 쓰려면 JupyterLab을 설치한다. `notebook`도 함께 설치된다.

```bash
pip install jupyterlab
```

설치 후 `jupyter` 명령을 찾지 못하면 `pyenv rehash`를 실행한다.

## 2. 실행

| 명령어 | 설명 |
| --- | --- |
| `jupyter notebook` | 클래식 노트북 실행 (브라우저 자동 열림) |
| `jupyter lab` | JupyterLab 실행 |
| `jupyter notebook --port 8889` | 포트 지정 (기본 8888) |
| `jupyter notebook --no-browser` | 브라우저를 열지 않고 URL만 출력 |
| `jupyter server list` | 실행 중인 서버와 접속 URL |

- 실행한 폴더가 작업 루트가 되므로 프로젝트 폴더에서 실행한다.
- 종료는 터미널에서 `Ctrl + C` 두 번.

## 3. 가상환경을 커널로 등록

노트북이 어느 Python으로 코드를 실행할지 정하는 것이 커널이다. 가상환경 안에서 `jupyter`를 실행하면 기본 커널이 이미 그 가상환경이라 등록 없이 써도 된다. 커널 목록에 이름으로 보이게 하거나 IDE에서 고르려면 등록한다.

```bash
python -m ipykernel install --user --name rag-study --display-name "Python (rag-study)"
```

| 명령어 | 설명 |
| --- | --- |
| `jupyter kernelspec list` | 등록된 커널 목록 |
| `jupyter kernelspec uninstall rag-study` | 커널 등록 해제 |

## 4. 올바른 가상환경에서 도는지 확인

노트북 셀에서 실행한다. 경로가 `~/.pyenv/versions/rag-study`로 나와야 한다.

```python
import sys
print(sys.prefix)
```

## 5. 노트북 안에서 패키지 설치

`!pip` 대신 `%pip`를 쓴다. `%pip`는 현재 커널의 Python에 설치하는 것이 보장된다.

```python
%pip install langchain
```

## 6. 자주 쓰는 단축키

| 키 | 동작 |
| --- | --- |
| `Shift + Enter` | 셀 실행 후 다음 셀로 이동 |
| `Ctrl + Enter` | 셀 실행 (이동 없음) |
| `Esc` → `A` / `B` | 위 / 아래에 셀 추가 |
| `Esc` → `D` `D` | 셀 삭제 |
| `Esc` → `M` / `Y` | 마크다운 / 코드 셀로 전환 |

## 7. VS Code에서 사용

VS Code가 노트북 화면을 직접 그려 주므로 브라우저용 서버(`notebook`, `jupyterlab`)는 필요 없다. 코드를 실행할 `ipykernel`만 있으면 된다.

| 필요한 것 | 설치 위치 | 역할 |
| --- | --- | --- |
| Python 확장 (Microsoft) | VS Code | 가상환경 인식 |
| Jupyter 확장 (Microsoft) | VS Code | `.ipynb` 편집·실행 화면 |
| `ipykernel` | 가상환경 `rag-study` | 셀 코드 실행 |

### 설정 순서

1. VS Code 확장 탭(`Cmd + Shift + X`)에서 **Python**, **Jupyter** 설치
2. 가상환경에 `ipykernel` 설치

   ```bash
   pip install ipykernel
   ```

3. 프로젝트 폴더를 VS Code로 열기 (`code .`)
4. `.ipynb` 파일 생성: `Cmd + Shift + P` → **Create: New Jupyter Notebook**
5. 오른쪽 위 **Select Kernel** → **Python Environments** → **rag-study** 선택
6. 셀에 코드를 쓰고 `Shift + Enter`로 실행

`ipykernel` 없이 5번까지 진행하면 VS Code가 설치 여부를 물어보므로 그때 설치해도 된다.

### 브라우저 방식과 차이

| | 브라우저 | VS Code |
| --- | --- | --- |
| 필요한 패키지 | `notebook` 또는 `jupyterlab` | `ipykernel` |
| 실행 | 터미널에서 `jupyter notebook` | `.ipynb` 파일을 열면 끝 |
| 커널 선택 | 메뉴 Kernel → Change Kernel | 오른쪽 위 Select Kernel |

## 8. 자주 겪는 문제

- **VS Code 커널 목록에 `rag-study`가 안 보임**: `Cmd + Shift + P` → **Python: Select Interpreter**에서 `~/.pyenv/versions/rag-study/bin/python`을 직접 지정하거나, **Developer: Reload Window**로 창을 새로 고친다.

- **`ModuleNotFoundError`가 나는데 `pip list`에는 있음**: 커널이 다른 Python을 쓰고 있다. 4번으로 경로를 확인하고 커널을 `Python (rag-study)`로 바꾼다.
- **포트가 이미 사용 중**: 다른 서버가 떠 있다. `jupyter server list`로 확인하거나 `--port`로 다른 포트를 지정한다.
- **`.ipynb_checkpoints/` 폴더가 생김**: 자동 저장용이다. `.gitignore`에 추가해 커밋에서 제외한다.
- **변수 상태가 꼬임**: 메뉴에서 Kernel → Restart로 커널을 재시작한다.
