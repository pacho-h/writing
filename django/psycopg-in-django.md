# psycopg3, binary 설치

Django 프로젝트에서 데이터베이스로 PostgreSQL을 사용하기위해 psycopg(PostgreSQL database adapter for Python)을 설치해야했다.

[Django 4.2 버전부터 psycopg 3버전을 지원하기 시작](https://docs.djangoproject.com/en/5.0/releases/4.2/#psycopg-3-support)했다고 한다. 새로 시작하는 프로젝트였기에 psycopg도 3버전을 사용하기로 했다.

[psycopg3](ihttps://pypi.org/project/psycopg/) 설치

```shell
poetry add psycopg
```

`pyproject.toml`


기본 Django 프로젝트를 생성하고 settins.py 에 database 설정값을 작성했다.

`project/settings.py`

```Python
# Database
# https://docs.djangoproject.com/en/5.0/ref/settings/#databases

DATABASES = {
    "default": {
        "ENGINE": f"django.db.backends.{os.environ.get('DATABASE_ENGINE', 'postgresql')}",
        "NAME": os.environ.get("DATABASE_NAME", "project_db"),
        "USER": os.environ.get("DATABASE_USER", "project_user"),
        "PASSWORD": os.environ.get("DATABASE_PASSWORD", "project_password"),
        "HOST": os.environ.get("DATABASE_HOST", "localhost"),
        "PORT": os.environ.get("DATABASE_PORT", "5432"),
    }
}
```

`INSTALLED_APPS` 리스트는 Django 프로젝트 생성 기본값 그대로 사용하고 makemigrations 커맨드를 실행했는데 다음과 같은 오류가 발생했다.

```shell
Traceback (most recent call last):
...
    import psycopg as Database
ModuleNotFoundError: No module named 'psycopg'
...

    import psycopg2 as Database
ModuleNotFoundError: No module named 'psycopg2'
...
```

Django는 PostgreSQL을 사용하면 psycopg 또는 psycopg2 module을 사용하는데 둘 다 찾지 못했다는 것이다. 하지만 분명히 psycopg (3버전)을 설치했고 `site-packages`에도 추가되어있는 것을 확인했다. 한참을 찾다가 다음과 같은 [팁](https://learndjango.com/tutorials/psycopg3-binary-and-django-42-installation-quick-t)를 발견했다.

내용을 보면 psycopg2를 사용할때 더 빠른 실행을 위해 binary 버전(`psycopg2-binary)만을 설치하는 경우가 있는데, psycopg (3버전)에서는 binary만을 따로 설치할 수 없고 psycopg 모듈에 선택적으로 포함시킬 수 있다고 한다.(psycopg-pool 모듈도 동일)
[psycopg-binary](https://pypi.org/project/psycopg-binary/)

더 자세한 원인은 찾지 못했지만 Django 5 버전에서 psycopg 모듈만 설치해서는 Django에서 PostgreSQL을 데이터베이스로 사용할 수 없었고, 아래처럼 binary를 함께 설치해주어야 정상적으로 동작했다.

```shell
poetry add psycopg[binary]
# pool도 설치하려면 psycopg[binary,pool]
```

`pyproject.toml`

```
[tool.poetry.dependencies]
python = "^3.12"
django = "^5.0.3"
psycopg = {extras = ["binary"], version = "^3.1.18"}

```

끝.
