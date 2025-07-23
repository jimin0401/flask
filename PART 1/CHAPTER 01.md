# 1.2 최소한의 기능 앱 만들기

pwd //현재의 작업 디렉터리를 표시

ls //디렉터리 내의 정보를 표시

## 작업 디렉터리 만들기
```
mkdir -p apps/minimalapp
```
## 애플리케이션 실행절차
1. 파이썬 코드 작성
apps/minimalapp폴더에 app.py생성
```
# app.py
# flask 클래스 import
from flask import Flask

# flask 클래스 인스턴스화
app = Flask(__name__)

# URL과 실행할 함수를 매핑
@app.route("/")
def index():
  return "Hello, flask"
```
3. 환경 변수 설정
```
cd apps/minimalapp
$env:FLASK_APP="app.py"
$env:FLASK_ENV="development" # $env:FLASK_DEBUG="1" 디버그 모드 안될시 사용, 디버그모드 사용시 코드 수정 후 재시작할 필요 없음
```
<br></br>
```
# 환경 변수 설정 없이 실행
if __name__ == "__main__":
    app.run(host='0.0.0.0' port=5000 debug=True)
```
## .env활용
.env파일 생성후
```
FLASK_APP=app.py
FLASK_ENV=production
```
```
pip install python-dotenv
```

4. flask run 명령어 실행

## 라우팅 활용하기
```
@app.route("/hello")
def hello():
  return "Hello, World!"
```
/hello로 접속해보기

$flask routes 를 이용하여 라우팅 정보를 확인할 수 있음
