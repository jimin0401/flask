# 3.2 앱에 인증 기능 등록하기
## Blueprint에서 사용자 인증 기능 등록하기
app.py
```
from flask import Flask
from flask migrate import Migrate
from flask_sqlalchemy import SQLAlchemy
from flask_wtf.csrf import CSRFProtect

from apps.config import config

db = SQLAlchemy ()
csrf = CSRFProtect)

def create_app(config_key) :
    app = Flask (__name__)

    # 이제부터 작성하는 auth 패키지로부터 views를 import한다
    from apps.auth import views as auth_views
    #register_blueprint를 사용해 views의 auth를 앱에 등록한다
    app. register_blueprint(auth_views.auth, url_prefix="/auth")
  return app
```
## 사용자 인증 기능 엔드포인트 만들기
views.py
```
from flask import Blueprint, render_template
    # Blueprint를 사용해서 auth를 생성한다
    auth = Blueprint(
        "auth",
        __name_,
        template_ folder="templates",
        static_folder="static"
    )

# index 엔드포인트를 작성한다
@auth. route("/")
def index():
    return render_template("auth/index.html")
```
## 인증 기능의 확인용 템플릿 만들기
base.html
```
<! DOCTYPE html >
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <title>{% block title %}{% endblock %}</title>
  </head>
  <body>
    {% block title %}{% endblock %}
  </body>
</html>
```
## 인증 페이지 표시 확인 화면 만들기
index.html
```
{% extends "auth/base.html"%}
{% block title%} 인증 페이지 {% endblock %}
{% block content %} 인증 페이지 표시 확인 {% endblock %}
```
# 3.3 회원가입 기능 만들기
flask login 패키지 다운
```
pip install flask-login
```
app.py
```
from flask import Flask
from flask_login import LoginManager

# LoginManager를 인스턴스화한다
login_manager = LoginManager()
# Login_view 속성에 미로그인 시 리다이렉트하는 엔드포인트를 지정한다
login_ manager. login_view = "auth.signup"
# login_message 속성에 로그인 후에 표시할 메시지를 지정한다
# 여기에서는 아무것도 표시하지 않도록 공백을 지정한다
Login manager. login_message = ""
def create_app(config_key):
    app = Flask(__ name__)

    # login_manager를 애플리케이션과 연계한다
    login manager.init_app(app)

    from apps.crud import views as crud_views
```
## 회원가입 기능의 폼 클래스 만들기
forms.py
```
from flask wtf import FlaskForm
from wtforms import PasswordField, StringField, SubmitField
from wtforms.validators import DataRequired, Email, Length

class SignUpForm(FlaskForm) :
    username = StringField(
        "사용자명"
        validators= [
            DataRequired("사용자명은 필수입니다."),
            Length(1, 30, "30문자 이내로 입력해 주세요."),
        ],
    )
    email = StringField(
        " 메일 주소",
        validators=[
            DataRequired("메일 주소는 필수입니다. "),
            Email("메일 주소의 형식으로 입력해 주세요. "),
        ],
    )
    password = PasswordFieLd("비밀번호",
        vaLidators=[DataRequired("비밀번호는 필수입니다. ")1)
    submit = SubmitField("신규 등록")
```
## User 모델 갱신하기
apps/crud/models.py
```
from datetime import datetime
from apps. app import db, login manager
from flask_login import UserMixin
from werkzeug security import generate_password_hash, check_password _hash

# User 클래스를 db.ModeL에 더해서 UserMixin을 상속한다
class User (db.Model, UserMixin):
    __tablename__ = "users"
    id = db. Column (db.Integer, primary_key=True)
    username = db. Column(db.String, index=True)
    email = db. Column(db.String, unique=True, index=True)
    password_hash = db.Column(db.String)
    created_at = db. Column (db.DateTime, default=datetime.now)
    updated_at = db. Column(db.DateTime, default=datetime. now,
                            onupdate=datetime.now)
@property
def password (self) :
    raise AttributeError("읽어 들일 수 없음")
@password.setter
def password (self, password):
    self. password_hash = generate_password_hash (password)

# 비밀번호 체크를 한다
def verify_password(self, password):
    return check_ password_hash(self-password _hash, password)

# 이메일 주소 중복 체크를 한다
def is_duplicate_email(self):
    return User. query. filter_by(email=self.email). first() is not None
# 로그인하고 있는 사용자 정보를 취득하는 함수를 작성한다
@login _manager.user_loader def load_user (user_id):
    return User. query get (user_id)
```
## 회원가입 기능의 엔드포인트 만들기
views.py
```
from apps.app import db
from apps.auth.forms import SignUpForm
from apps.crud.models import User
from flask import Blueprint, render_template, flash, url_for, redirect, request
from flask_login import login_user

@auth.route("/signup", methods=["GET", "POST"])
def signup() :
# SignUpForm을 인스턴스화한다
form = SignUpForm()
if form. validate_on_submit():
    user = User(
        username=form.username.data,
        email=form.email.data,
        password=form. password.data,
    )

    # 이메일 주소 중복 체크를 한다
    if user.is_duplicate_email() :
        flash("지정 이메일 주소는 이미 등록되어 있습니다")
        return redirect(url_for ("auth.signup"))

    # 사용자 정보를 등록한다
    db.session.add (user)
    db.session.commit()
    # 사용자 정보를 세션에 저장한다
    login_user(user)
    # GET 파라미터에 next 키가 존재하고, 값이 없는 경우는 사용자의 일람 페이지로 리다이렉트한다
    next_ = request. args. get ("next")
    if next_ is None or not next_ startswith("/"):
        next_ = urL_for ("crud.users")
    return redirect (next_)
return render_template("auth/signup.html", form=form)
```
## 회원가입 템플릿
singup.html
```
% extends "auth/base.html" %}
{% block title %}사용자 신규 등록{% endblock %}
{% block content %}
<h2> 사용자 신규 등록</h2>
<form
  action="{{ url_for('auth signup', next-request.args. get ('next')) }}"
  method="POST"
  novalidate="novalidate"
>
  {% for message in get_flashed_ messages () %}
  <p style="color: red;">{{ message }}</p>
  {% endfor %}
  {{ form. csrf_token }}
  ‹p>
    {{ form. username. label }}
    {{ formusername(size=30, placehoLder="사용자명") }}
  </p>
  {% for error in form.username. errors %}
  <span style="color: red;"›{{ error }}</ span›
  {% endfor %}
  <p>{{ form. email. label }} {{ form. email (placeholder="메일 주소") }}</p›
  {% for error in form.email.errors %}
  <span style="color: red;"> {{error }}</span>
  {% endfor %}
  <p>{{ form.password. label }} {{ form.password (placeholder="비밀번호") }}</р>
  {% for error in form.password. errors %}
  <span style="color: red;">{{ error }}</span>
  {% endfor %}
  <p>{{ form. submit() }}</p>
</form>
{% endblock %}
```
# 3.4 로그인 기능 만들기
forms.py
```
class LoginForm(FlaskForm):
    email = StringField(
        "메일 주소",
        validators=[
            DataRequired("이메일 주소는 필수입니다."),
            Email("이메일 주소의 형식으로 입력하세요.),
        ],
    )

password = PasswordFieLd("비밀번호", validators=[DataRequired("비밀번호는 필수입니다.")1)
submit = SubmitField("로그인")
```
## 로그인 기능의 엔드포인트 만들기 
views.py
```
from apps.app import db
from apps.auth.forms import SignUpForm, LoginForm

@auth.route("/login", methods=["GET", "POST"])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        # 이메일 주소로 데이터베이스에 사용자가 있는지 검사한다
        user = User.query.filter_by(email=form.email.data).first()

        # 사용자가 존재하고 비밀번호가 일치하면 로그인을 허가한다
        if user is not None and user.verify_password(form.password.data):
            login_user(user)
            return redirect(url_for("crud.users"))

        # 로그인 실패 메시지를 설정한다
        flash("메일 주소 또는 비밀번호가 일치하지 않습니다.")
    return render_template("auth/login.html", form=form)
```
## 로그인 템플릿
login.html
```
{% extends "auth/base.html" %}
{% block title %}로그인{% endblock %}

{% block content %}
<h2>로그인</h2>
<form action="{{ url_for('auth.login') }}"
      method="post"
      novalidate="novalidate">
  
  {% for message in get_flashed_messages() %}
    <p style="color: red;">{{ message }}</p>
  {% endfor %}

  {{ form.csrf_token }}

  <p>
    {{ form.email.label }}
    {{ form.email(placeholder="메일주소") }}
  </p>
  {% for error in form.email.errors %}
    <span style="color: red;">{{ error }}</span>
  {% endfor %}

  <p>
    {{ form.password.label }}
    {{ form.password(placeholder="비밀번호") }}
  </p>
  {% for error in form.password.errors %}
    <span style="color: red;">{{ error }}</span>
  {% endfor %}

  <p>{{ form.submit() }}</p>
</form>
{% endblock %}

```

## 로그아웃 엔드포인트(auth/views.py)
```
from flask_login import login_user, logout_user
@auth.route("/logout")
def logout():
    logout_user()
    return redirect(url_for("auth.login"))
```
## 공통 템플릿 갱신(base.html)
```
<!DOCTYPE html >
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <title>{% block title %}{% endblock %}</title>
  </head>
  <body>
    <div>
        {% if current_user.is_authenticated %}
        <p>
            <span>{{current_user.username}}</span>
            <span><a href="{{url_for('auth.logout')}"}></span>

        </p>
        {%endif%}
    </div>
    {% block content %}{% endblock %}
  </body>
</html>
```
## forms.py
```
from flask_wtf import FlaskForm
from wtforms import PasswordField, StringField, SubmitField
from wtforms.validators import DataRequired, Email, Length

class SignUpForm(FlaskForm) :
    username = StringField(
        "사용자명",
        validators= [
            DataRequired("사용자명은 필수입니다."),
            Length(1, 30, "30문자 이내로 입력해 주세요."),
        ],
    )
    email = StringField(
        " 메일 주소",
        validators=[
            DataRequired("메일 주소는 필수입니다. "),
            Email("메일 주소의 형식으로 입력해 주세요. "),
        ],
    )
    password = PasswordField("비밀번호",
        validators=[DataRequired("비밀번호는 필수입니다. ")])
    submit = SubmitField("신규 등록")

class LoginForm(FlaskForm):
    email = StringField(
        "메일 주소",
        validators=[
            DataRequired("이메일 주소는 필수입니다."),
            Email("이메일 주소의 형식으로 입력하세요."),
        ],
    )

    password = PasswordField("비밀번호", validators=[DataRequired("비밀번호는 필수입니다.")])
    submit = SubmitField("로그인")
```
## base.index
```
<!DOCTYPE html >
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <title>{% block title %}{% endblock %}</title>
  </head>
  <body>
    <div>
        {% if current_user.is_authenticated %}
        <p>
            <span>{{current_user.username}}</span>
            <span><a href="{{url_for('auth.logout')}}"></span>

        </p>
        {%endif%}
    </div>
    {% block content %}{% endblock %}
  </body>
</html>
```
## login.html
```
{% extends "auth/base.html" %}
{% block title %}로그인{% endblock %}

{% block content %}
<h2>로그인</h2>
<form action="{{ url_for('auth.login') }}"
      method="post"
      novalidate="novalidate">
  
  {% for message in get_flashed_messages() %}
    <p style="color: red;">{{ message }}</p>
  {% endfor %}

  {{ form.csrf_token }}

  <p>
    {{ form.email.label }}
    {{ form.email(placeholder="메일주소") }}
  </p>
  {% for error in form.email.errors %}
    <span style="color: red;">{{ error }}</span>
  {% endfor %}

  <p>
    {{ form.password.label }}
    {{ form.password(placeholder="비밀번호") }}
  </p>
  {% for error in form.password.errors %}
    <span style="color: red;">{{ error }}</span>
  {% endfor %}

  <p>{{ form.submit() }}</p>
</form>
{% endblock %}

```
