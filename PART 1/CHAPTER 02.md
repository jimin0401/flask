# 2.1 디렉터리 구성
## CRUD 앱의 디렉터리 구성
CRUD : 데이터 베이스를 조작하는데 있어서 최소한 필요한 기능인 Create(데이터 작성), Read(읽어 들이기), Update(갱신), Delete(삭제)의 약칭

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
