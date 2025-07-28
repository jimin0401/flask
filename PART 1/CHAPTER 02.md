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
    ur l_prefix="/sample",
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

