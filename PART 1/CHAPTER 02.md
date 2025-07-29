# 2.1 디렉터리 구성
## CRUD 앱의 디렉터리 구성
CRUD : 데이터 베이스를 조작하는데 있어서 최소한 필요한 기능인 Create(데이터 작성), Read(읽어 들이기), Update(갱신), Delete(삭제)의 약칭
```
📁 flaskbook
├── 📄 .env
├── 📁 apps
│ ├── 📄 app.py
│ ├── 📄 config.py
│ └── 📁 crud
│ ├── 📄 init.py
│ ├── 📄 forms.py
│ ├── 📄 models.py
│ ├── 📁 static
│ │ └── 📄 style.css
│ ├── 📁 templates
│ │ └── 📁 crud
│ │ ├── 📄 base.html
│ │ ├── 📄 create.html
│ │ ├── 📄 edit.html
│ │ └── 📄 index.html
│ └── 📄 views.py
├── 📄 local.sqlite
└── 📁 migrations
```

# 2.2 앱 실행하기 : Blueprint의 이용
<ol>
  <li>CRUD 앱의 모듈을 작성</li>
  <li>환경 변수 FLASK_APP의 경로를 변경</li>
  <li>엔드포인트를 만듦</li>
  <li>템플릿을 만듦</li>
  <li>정적 파일을 작성</li>
  <li>템플릿에 CSS를 읽어 들임</li>
  <li>동작을 확인</li>
</ol>

## 1. CRUD 앱의 모듈 작성하기
aaps/app.py
```
from flask import Flask
# create_app 함수를 작성한다
def create_app():
    # 플라스크 인스턴스 생성
    app = Flask(_ name__)

    # crud 패키지로부터 views를 import한다
    from apps.crud import views as crud_views

    # register_blueprint를 사용해 views의 crud를 앱에 등록한다
    app. register_blueprint(crud_views.crud, url_prefix="/crud"')
return app
```
Blueprint 객체 생성 예시
```
# Blueprint 객체를 생성한다
# template_folder와 static_folder를 지정하지 않는 경우는
# Blueprint 앱용 템플릿과 정적 파일을 이용할 수 없다
sample = Blueprint(
    __name__,
    "sample",
    static_folder="static",
    template_folder="templates",
    url_prefix="/sample",
    subdomain="example",
)
```

```
app.register_blueprin(sample, url_prefix="/sample", subdomain="example")
```

## 2. 환경 변수 FLASK_APP의 경로 변경하기
```
FLASK_APP=apps.app:create_app
FLASK_ENV=developement
```

## 3. 엔드포인트 만들기
```
from flask import Blueprint, render_template
# Blueprint로 Crud 앱을 생성한다
crud = Blueprint(
    "crud",
    __name__,
    template_folder="templates",
    static_folder-"static",
)

# index 엔드포인트를 작성하고 index.html을 반환한다
@crud, route("/")
def index:
    return render_template("crud/index.html.")
```

## 4. 템플릿 만들기
apps/crud/templates/crud/index.html
```
<! DOCTYPE html>
<html lang="ko">
‹head>
    ‹meta charset="UTF-8" />
    ‹title>CRUD</title>
</head>
< body>
    <p Class="crud">CRUD 애플리케이션</p〉
</body>
</html>
```
## 5. 정적 파일 작성하기
apps/crud/static/style.css
```
.crud {
    font-style: italic;
}
```
## 6. 템플릿에 css읽어 들이기
```
<! DOCTYPE html>
<html lang="ko">
‹head>
    ‹meta charset="UTF-8" />
    <link rel="stylesheet" href="{{ url_for('crud.static', file='style.css')}}"/>
    ‹title>CRUD</title>
</head>
< body>
    <p Class="crud">CRUD 애플리케이션</p〉
</body>
</html>
```
## 7. 동작 확인하기
http://127.0.0.1:5000, http://127.0.0.1:5000/crud/접속

# 2.3 SQLAlchemy 설정하기
확장 기능 설치하기
```
pip install flask-sqlalchemy
pip install flask-migrate
```
apps/app.py
```
from pathlib import Path
from flask import Flask
from flask migrate import Migrate
from flask_sqlalchemy import SQLAlchemy

# SOLALchemy를 인스턴스화한다
db = SQLAlchemy)

def create_app():
    app = Flask (__name__)
    # 앱의 config 설정을 한다
    app. config.from _mapping(
        SECRET_KEY="2AZSMss3p5QPbcY2hBsJ",
        SQLALCHEMY_ DATABASE_URI=
            f'sqlite:///{Path(__file__). parent. parent / 'local sqlite'}",
        SQLALCHEMY_TRACK_MODIFICATIONS=False
)
    # SOLAlchemy와 앱을 연계한다
    db.init_app(app)
    # Migrate와 앱을 연계한다
    Migrate(app, db)
```
# 2.4 데이터 베이스 조작하기

## 모델 정의하기
apps/crud/models.py
```
from datetime import datetime
from apps.app import db
from werkzeug. security import generate_password_hash

# db. Model을 상속한 User 클래스를 작성한다
class User (db.Model) :
    # 테이블명을 지정한다
    __tablename__ = "users"
    # 컬럼을 정의한다
    id = db.Column(db. Integer, primary_key=True)
    username = db.Column(db.String, index=True)
    email = db.Column(db.String, unique=True, index=True)
    password_hash = db.Column (db.String)
    created_at = db.Column (db.DateTime, default=datetime.now)
    updated_at = db.Column (
        db.DateTime, default=datetime.now, onupdate=datetime.now
    )
    # 비밀번호를 설정하기 위한 프로퍼티
    @property
    def password (self):
        raise AttributeError("읽어 들일 수 없음")
    # 비밀번호를 설정하기 위해 Setter 함수로 해시화한 비밀번호를 설정한다
    @password.setter
    def password(self, password) :
        self.password_hash = generate_password_hash(password)
```
모델 선언하기(apps/crud/__init__.py)
```
import apps.crud.models
```
## 데이터베이스 초기화와 마이그레이션 
```
flask db init
```
```
flask db migrate
```
```
flask db upgrade
```
```
flask db downgrade
```
## SQLAlchemy를 사용한 기본적인 데이터 조작
SQL 로그 출력하기(apps/app.py)
```
def create_app):
    app = Flask(__name__)
    # 앱의 config 설정
    app. config. from_mapping(
        SECRET_KEY="2AZSMss3p5QPbcY2hBsJ",
        SQLALCHEMY_DATABASE_URI="sqlite:////"
        + str (Path(Path(__file__). parent-parent, "local sqlite")),
        SQLALCHEMY_TRACK_MODIFICATIONS=False,
        #SOL을 콘솔 로그에 출력하는 설정
        SQLALCHEMY_ECHO=True
)
```
### query filter와 executer
query filter : 검색 조건을 좁히거나 정렬을 위해 이용

executer : SQL을 실행하고 결과를 취득하기 위해 이용
```
User.query
    .filter_by(id=2, username="admin") # query filter
    .all() # executer
```
### executer를 사용한 SELECT하기
모든 데이터 가져오기
```
from apps.app import db
from apps.crud.models import User

@crud.route("/sql")
def sql():
    db.session.query(User).all()
    return "콘솔 로그를 확인해 주세요"
```
레코드 1건 가져오기
```
db.session.query(User).first()
```
2건 가져오기
```
db.session.query(User).get(2)
```
레코드 개수 가져오기
```
db.session.query(User).count()
```
페이지네이션 객체 가져오기
```
paginate(page=None, per_page=None, error_out=Ture, max_per_page=None)
```
한 페이지에 10건 표시하고 두번쨰 페이지 표시하기
```
db.session.query(User).paginate(2, 10, False)
```
### query filter를 사용하여 SELECT하기
- where 구(filter_by)

id가 2이고 username이 admin인 레코드 가져오기
```
db.session.query(User).filter_by(id=2, username="admin").all
```
- where 구(filter)

id가 2이고 username이 admin인 레코드 가져오기
```
db.session.query(User).filter(User.id==2, User.username=="admin").all()
```
- limit 구

가져올 레코드의 개수를 1건으로 지정하기
```
db.session.query(User).limit(1).all()
```
- offset 구

3번째의 레코드로부터 1건 가져오기
```
db.session.query(User).limit(1).offset(1).all()
```
- order by 구

username을 정렬
```
db.session.query(User).order_by("username").all()
```
- group by 구

username을 그룹화
```
db.session.query(User).group_by("username").all()
```
###  ISERT하기
```
# 사용자 모델 객체를 작성한다
user = User
    username="사용자명",
    email="flaskbook@example.com"
    password="비밀번호"
)
# 사용자를 추가한다.
db.session.add(user)
# 커밋한다
db.session.commit()
```
### UPDATE 하기
```
user = db .session.query(User). filter_by(id=1). first()
user.username = "사용자명2"
user. email = "flaskbook2@example.com"
user.password = "비밀번호2"
db. session. add (user)
db. session.commit()
```
### DELETE 하기
```
user = db. session. query (User). filter_by (id=1) .delete()
db. session. commit()
```
# 2.5 데이터베이스를 사용한 CRUD앱 만들기
## 폼의 확장 기능 이용하기
```
pip install flask-wtf
```
CSRF대책을 위한 설정 추가하기(apps/app.py)
```
from flask_wtf.csrf import CSRFProtect

csrf = CSRFProtect()

def create_app():
    app = Flask(__name_)
    app. config. from_mapping(
        WTF_CSRF_SECRET_KEY="AuwzyszU5sugKN7KZs6f",
    )
csrf.init_app(app)
```
## tkdydwkfmf tlsrb wkrtjdgkrl
apps/crud/forms.py
```
from flask_wtf import FlaskForm
from wtforms import PasswordField, StringField, SubmitField
from wtforms.validators import DataRequired, Email, Length

# 사용자 신규 작성과 사용자 편집 폼 클래스
class UserForm(FlaskForm) :
    # 사용자 폼의 username 속성의 라벨과 검증을 설정한다
    username = StringField
        "사용자명"
        validators=[
            DataRequired(message="사용자명은 필수입니다. "),
            Length(max=30, message="38문자 이내로 입력해 주세요. "),
        },
    )

    # 사용자폼 email 속성의 라벨과 검증을 설정한다
    email = StringField(
        "메일 주소",
        validators=[
            DataRequired(message="메일주소는 필수입니다. "),
            Email(message="메일 주소의 형식으로 입력해 주세요. "),
        ],
    )

    # 사용자 폼 password 속성의 라벨과 검증을 설정한다
    password = PasswordField(
        "비밀번호",
        validators=[DataRequired(message="비밀번호는 필수입니다. ")]
    )
    # 사용자폼 Submit의 문구를 설정한다
    submit= SubmitFieLd("신규 등록")
```
## 사용자 신규 작성 화면의 엔드포인트 만들기
```
from apps.crud.forms import UserForm
from flask import Blueprint, render_template, redirect, url_for


@crud. route("/users/new", methods=["GET", "POST"])
def create_user():
    # UserForm을 인스턴스화한다
    form = UserForm()
    # 폼의 값을 검증한다
    if form. validate_on_submit):
        # 사용자를 작성한다
        user = User (
            username=form. username. data,
            email=form. email.data,
            password=form. password. data,
        )
    # 사용자를 추가하고 커밋한다
    db. session. add (user)
    db.session. commit()
    # 사용자의 일람 화면으로 리다이렉트한다
    return redirect(url_for("crud.users"))
return render_template("crud/create.html", form=form)
```
## 신규 작성 화면의 템플릿 만들기 (apps/crud/templates/crud/create.html)
```
<! DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <title>사용자 신규 작성 </title>
  </head>
‹body>
  <h2> 사용자 신규 작성</h2>
  <form
    action="{f url_for ('crud. create_user') }}"
    method="POST"
    novalidate="novalidate"
  >
      {{form. csrf_token }}
      <p>
        {{ form.username. label J} {{ form. username (placeholder="사용자명") }}
      </p>
      {% for error in form.username.errors %}
      <span style="color: red;">{{ error }}</span>
      {% endfor %}
      <p>
        {{form. email. label f} ff form. email (placeholder="메일 주소") }}
      </p>
      {% for error in form.email.errors %}
      <span style="color: red;">{{ error }}</span>
      {% endfor %}
      <p>
        {{ form-password. label }}{{ form-password(placeholder="비밀번호")}}
      </p>
      {% for error in form.password.errors %}
      <span style="color: red;">{{ error }</span>
      {% endfor %}
      <p>{{form.submit()}}</p>
    </form>
  </body>
</html>
```
## 사용자 일람 표시하기
### 엔드포인트 만들기(apps/crud/views.py)
```
@crud. route("/users")
def users:
    """사용자의 일람을 취득한다""
    users = User query .all)
    return render_template("crud/index.html", users-users)
```
템플릿 index.html
```
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset= "UTF-8" />
    <title>사용자의 일람</titLe>
    <Link rel="stylesheet" href="If url_for('crud static', filename='style.css') }}"/>
  </head>
  <body>
    <h2> 사용자의 일람</h2>
    <a href="{f url_for('crud.create_user') }}">사용자 신규 작성 </ a2
    <table>
      ‹tr›
        <th>사용자 ID</th>
        <th>사용자명</th>
        <th>메일 주소</th>
      </tr>
      {% for user in users %}
      <tr>
        <td>f user.id }}</td>
        <td>t user. username }}</td>
        <td>f user.email }}</td>
      </tr>
      {% endfor %}
    </table>
  </body>
</html>
```
## 스타일 시트 수정하기
```
table {
    /* 인접하는 border를 겹쳐서 1개로 해서 표시한다 */
    border-collapse: collapse;
}
table,
th,
td {
/* 1px의 실선으로 해서 색을 조절한다 */
    border: 1px solid #c0c0c0;
```
## 사용자 편집하기
views.py
```
# methods에 GET과 POST를 지정한다
@crud.route("/users/<user_id>", methods=["GET", "POST"])
def edit_user(user_id):
    form = UserForm)

    # User 모델을 이용하여 사용자를 취득한다
    user = User.query. filter_by(id=user_id). first()

    # form으로부터 제출된 경우는 사용자를 갱신하여 사용자의 일람 화면으로 리다이렉트한다
    if form.validate_on_submit():
        user. username = form.username.data
        user.email = form.email.data
        user.password = form.password.data
        db. session.add(user) db.session.commit()
    return redirect(url_for("crud.users"))

# GET의 경우는 HTML을 반환한다
return render_template("crud/edit.html", usereuser, form=form)
```
edit.html
```
<! DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" /›
    <title>사용자 편집</title>
    <link rel="stylesheet" href="[f url_for('crud.static', filename='style.css' )}}" />
  </head>
  <body>
    <h2>사용자 편집</h2>
    <form
      action="I url_for ('crud.edit_user', user_id-user.id) }}"
      method="POST"
      novalidate="novalidate"
    >
    {{ form.csrf_token }}
    <p>
      {{ form.username.label y} {{ form.username(placeholder="사용자명"
      value=user .username) }}
    </p>
    {% for error in form.username.errors %}
    <span style="color: red;"›{{ error }}</span>
    {% endfor %}
    <p>
      {{form. email. label }} {{ form. email (placeholder="메일 주소", value=user.email) }}
    </p>
    {% for error in form.email.errors %}
    <span style="color: red;">{{ error }}</span>
    {% endfor %}
    <p>
      {{ form.password. label th ft form.password(placeholder="비밀번호") }}
    </p>
    {% for error in form.password.errors %}
    ‹span style="color: red;">{ error }}</span>
    {% endfor %}
    <p>
      <input type="submit" value="갱신" />
    </p›
    </form>
  </body>
</html>
```
index.html 수정
```
<td>
<a href="{{ url_for('crud.edit_user', user_id=user.id) }}">{{ user.id}} </a> 추가
</td>
```
## 사용자 삭제하기
views.py
```
@crud. route("/users/<user_id>/delete"', methods=[" POST" ])
def delete_user(user_id):
    user=user. query. filter_by(id-user_id). first )
    db. session. delete(user)
    db. session. commit ( )
    return redirect(url_for("crud.users"))
```
edit.html
```
<form action="{f url_for('crud delete_user', user_id=user.id) }}" method="POST">
  {{ form. csrf_token }}
  <input type="submit" value="삭제">
‹/ form>
```

