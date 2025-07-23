# 플라스크의 개요와 환경 구축
<br></br>

## 파이썬 설치하기
[링크](https://www.python.org/downloads/)
두 항목 체크: add python to PATH, Install launcher for all users
명령 프롬프트(파워셸)에 설치 확인
```
python --version
```

## 가상 환경 만들기
### 1. 로컬 환경
1. 정책 변경(가상환경 활성화 스크립트 실행)
```
PowerShell Set-ExecutionPolicy RemoteSigned CurrentUser
```
2. 가상 환경 생성
```
py -m venv venv
```
3. 가상 환경 활성화(명령 프롬프트 : activate.bat)
```
.\venv\Scripts\Activate.ps1
```
4. 가상 환경 비활성화
```
deactivate
```
### 2. 아나콘다
1. 다운로드
[링크](https://www.anaconda.com/download/success)
2. 가상 환경 생성(Anaconda Prompt)
```
conda create -n myenv python=3.10
```
3. 가상 환경 활성화
```
conda activate myenv
```
4. conda deactivate

## 플라스크 설치하기
```
mkdir flask
cd flask
venv\Scripts\Activate.ps1
pip install flask //anaconda사용시 conda install flask
```

