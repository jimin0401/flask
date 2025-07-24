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
```
$flask routes # 이 코드를 이용하여 라우팅 정보를 확인할 수 있음
```
### 엔드포인트에 이름 붙이기
URI와 연결된 함수명 
```
@app.route("/", endpoint="endpoint-name")
```
### 허가할 HTTP 메서드 지정하기
```
@app.route("/hello", methods=["GET", "POST"])
```
```
@app.get("/hello")
@app.post("/hello")
  return "Hello, World!"
```
동일한 코드

### Rule에 변수 지정
```
@app.route("/hello/<name>")
def hello(name):
  return f"Hello, {name}!"
```
/hello/xxx로 접속

<자료형: 변수명>으로 타입 정의 가능

## 템플릿 엔진 이용하기
reder_template 함수를 사용하여 템플릿을 이용

minimalapp에 templates폴더 생성 후 index.html생성
```
<!--index.html-->
<!DOCTYPE html>
<html lang="ko" >
    <head>
        <meta charset="UTF-8" />
        <title>Name</title>
    </head>
    <body>
        <h1>Name: {{ name }}</h1>
    </body>
</html>
```
```
# app.py
@app.route("/name/<name>")
def show_name(name):
  return render_template("index.html", name=name)
```
### Jinja2의 사용법
변수 : {{}}

if 문, for 문 : {% %}

## url_for 함수를 사요어해서 URL 생성하기
flask routes 명령어 사용하여 endpoint를 호출 
```
from flask import url_for

### 앱 동작 확인
with app.test_request_context():
  print(url_for("index"))
  print(url_for("hello-endpoint", name="world"))
  print(url_for("show_name", name="aa", page="1"))
```

## 정적 파일 이용하기
minimalapp에 static 폴더 생성 후 style.css 생성
```
/* style.css */
h1 {
  color: darkblue;
  font-size: 36px;
  text-align: center;
  font-family: 'Arial', sans-serif;
  text-transform: uppercase;
  border-bottom: 2px solid #ccc;
  padding-bottom: 10px;
}
```
index.html의 head안에 삽입
```
        <link 
            rel="stylesheet"
            href="{{url_for('static', filename= 'style.css')}}"
        />
```

## 애플리케이션 컨텍스트와 요청 컨텍스트
### 애플리케이션 컨텍스트
요청을 통해 앱 레벨의 데이터를 이용할 수 있도록 하는 것
- current_app : 액티브 앱(실행 중의 앱)의 인스턴스
- g : 요청을 통해 이용할 수 있는 전역 임시 영역. 요청마다 리셋


# 1.3 문의 폼 만들기
```
@app. route("/contact/complete", methods=["GET", "POST"])
def contact_complete():
    if request.method == "POST" :
    # 이메일을 보낸다(나중에 구현할 부분)
    
    # contact 엔드포인트로 리다이렉트한다
        return redirect(url_for ("contact_complete"))
    return render_template("contact_complete.html")
```
contact.html
```
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8" />
    <title>문의 폼</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}" />
</head>

<body>
    <h2>문의 폼</h2>
    <form
        action="{{url_for('contact_complete') }}"
        method="POST" 
        novalidate="novalidate"
    >
    <table>
        <tr>
            <td>사용자명</td>
            <td>
                <input
                    type="text"
                    name="username"
                    value="{{ username }}"
                    placeholder="사용자명"
                />
            </td>
        </tr>
        <tr>
            <td>메일 주소</td>        
            <td>
                <input
                    type="text"
                    name="email"
                    value="{{ email }}"
                    placeholder="메일 주소"
                />
            </td>
        </tr>
        <tr>
            <td>문의 내용</td>
            <td>
                <textarea name="description" placeholder="문의 내용">{{ description }}</textarea>
            </td>
        </tr>
        </table>
        <input type="submit" value="문의" />
    </form>
</body> 
</html>
```
contact_complete.html
```
<!DOCTYPE html>
<html lang="ko">
    <head>
    <meta charset="UTF-8" />
    <title>문의 완료</title>
    <link
        rel="stylesheet"
        href="{{ url_for('static', filename='style.css') }}"
    />
    </head>
    <body>
        <h2>문의 완료</h2>
    </body>
</html>
```
### POST된 폼의 값 얻기
```
@app. route("/contact/complete", methods=["GET", "POST"])
def contact_complete():
    if request.method == "POST" :
    # from값 취득
        username = request.form["username"]
        email = request.form["email"]
        description = request.form["description"]
        
    # contact 엔드포인트로 리다이렉트한다
        return redirect(url_for ("contact_complete"))
    return render_template("contact_complete.html")
```

## flash 메시지
flash 메시지를 사용하기 위해서 session이 필요함
```
app.config["SECRET_KEY"] = "2AFCV3HJdk"
```
이메일 주소 확인 패키지
```
pip install email-validator
```
입력 체크
```
from email_validator import validate_email, EmailNotValidError
from flask import flash

@app. route("/contact/complete", methods=["GET", "POST"])
def contact_complete():
    if request.method == "POST" :
    # from값 취득
        username = request.form["username"]
        email = request.form["email"]
        description = request.form["description"]

        is_valid = True
        
        if not username:
            flash("사용자명은 필수입니다")
            is_valid = False

        if not email:
            flash("메일 주소는 필수입니다 ")
            is_valid = False
        
        try:
            validate_email(email)
        
        except EmailNotValidError:
            flash("메일 주소의 형식으로 입력해 주세요")
            is_valid = False
        
        if not description:
            flash("문의 내용은 필수입니다")
            is_valid = False

        if not is_valid:
            return redirect(url_for("contact"))

        # 이메일을 보낸다(나중에 구현할 부분)

        # 문의 완료 엔드포이트로 리다이렉트한다
        flash("문의해 주셔서 감사합니다.")
        return redirect(url_for ("contact_complete"))
    return render_template("contact_complete.html")
```
contact.html
```
    <h2>문의 폼</h2>
    {% with messages = get_flashed_messages() %}
    {% if messages %}
    <ul>
        {% for message in messages %} 
        <li class="flash">{{ message }}</li>
        {% endfor %}
    </ul>
    {% endif %}
    {% endwith %}
```
contact_complete.html
```
        <h2>문의 완료</h2>
        {% with messages = get_flashed_messages() %}
        {% if messages %} 
        <ul>
            {% for message in messages %} 
            <li>{{ message }}</li>
            {% endfor %}
        </ul>
        {% endif %}
        {% endwith %}
```

## 이메일 보내기
```
pip install flask-mail
```
[Gmail 2단계 인증 프로세스](https://myaccount.google.com/signinoptions/two-step-verification/enroll-welcome)
[앱용 비밀번호](https://myaccount.google.com/apppasswords)
```
from flask_mail import Mail
import os

# Mail 클래스의 config
app.config["MAIL_SERVER"] = os.environ.get("MAIL_SERVER")
app.config["MAIL_PORT"] = os.environ.get("MAIL_PORT")
app.config["MAIL_USE_TSL"] = os.environ.get("MAIL_USE_TSL")
app.config["MAIL_USERNAME"] = os.environ.get("MAIL_USERNAME")
app.config["MAIL_PASSWORD"] = os.environ.get("MAIL_PASSWORD")
app.config["MAIL_DEFAULT_SENDER"] = os.environ.get("MAIL_DEFAULT_SENDER")

#flask-mail 확장 등록
mail = Mail(app)
```
.env 설정
```
MAIL_SERVER=smtp.gmail.com
MAIL_PORT=587
MAIL_USE_TLS=True
MAIL_USERNAME=schch0401@gmail.com
MAIL_PASSWORD=dctj acve jlbg cyji
MAIL_DEFAULT_SENDER=flask schch0401@gmail.com
```

### 이메일 보내기
```
from flask_mail import Mail, Message

@app. route("/contact/complete", methods=["GET", "POST"])
def contact_complete():
    if request.method == "POST" :
    # from값 취득
        username = request.form["username"]
        email = request.form["email"]
        description = request.form["description"]

        is_valid = True
        
        if not username:
            flash("사용자명은 필수입니다")
            is_valid = False

        if not email:
            flash("메일 주소는 필수입니다 ")
            is_valid = False
        
        try:
            validate_email(email)
        
        except EmailNotValidError:
            flash("메일 주소의 형식으로 입력해 주세요")
            is_valid = False
        
        if not description:
            flash("문의 내용은 필수입니다")
            is_valid = False

        if not is_valid:
            return redirect(url_for("contact"))

        # 이메일을 보낸다(나중에 구현할 부분)
        send_email(
           email,
           "문의 감사합니다.",
           "contact_mail",
           username=username,
           description=description
        )
        
        # 문의 완료 엔드포이트로 리다이렉트한다
        flash("문의해 주셔서 감사합니다.")
        return redirect(url_for ("contact_complete"))
    return render_template("contact_complete.html")

def send_email(to, subject, template, **kwargs):
   msg = Message(subject, recipients=[to])
   msg.body = render_template(template + ".txt", **kwargs)
   msg.html = render_template(template + ".html", **kwargs)
   mail.send(msg)
```
### 이메일 템플릿
contact_mail.txt
```
{{username}}씨

문의 감사합니다. 문의 내용은 다음과 같습니다.

문의 내용

{{ description }}
```
contact_mail.html
```
<!DOCTYPE html>
<html lang="ko">
    <head>
    <meta charset="UTF-8" />
    <title>문의 완료 </title>
    </head>
    
    <body>
        <p>{{ username }}</p>
        <p>문의 감사합니다. 문의 내용은 다음과 같습니다. </p> 
        <p>문의 내용</p>
        <p>{{ description }}</p>
    </body>
</html>
```

