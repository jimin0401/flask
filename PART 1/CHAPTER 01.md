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
$env:FLASK_ENV="development"
```
<br></br>
```
# 환경 변수 설정 없이 실행
if __name__ == "__main__":
    app.run(debug=True)
```
4. flask run 명령어 실행
