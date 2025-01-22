# Создание веб-приложения с помощью Flask в Python 3 в инфраструктуре {{ yandex-cloud }}

[Flask](http://flask.pocoo.org/) — это небольшой фреймворк для создания веб-приложений на языке программирования Python. Он является наиболее доступным фреймворком для новых разработчиков, так как позволяет создать веб-приложение, используя только один файл Python. Flask — это расширяемая система, которая не обязывает использовать конкретную структуру директорий и не требует сложного шаблонного кода перед началом использования. Flask использует набор инструментов [Werkzeug](https://werkzeug.palletsprojects.com/en/latest/) и [механизм шаблонов Jinja](http://jinja.palletsprojects.com/) для динамического создания HTML-страниц с использованием знакомых понятий в Python, таких как переменные, циклы, списки и т. д.

В рамках этого руководства вы будете использовать [инструментарий Bootstrap](https://getbootstrap.com/), чтобы сделать ваше приложение визуально привлекательным. Bootstrap включает в себя HTML и CSS шаблоны оформления для типографики, веб форм, кнопок, меток, блоков навигации и прочих компонентов веб-интерфейса, включая JavaScript расширения.

С помощью этого руководства вы создадите небольшой блог с использованием Flask и [SQLite](https://sqlite.org/) в Python 3. Пользователи приложения могут видеть все посты в вашей базе данных и нажимать на заголовки постов для просмотра их содержания. Кроме того, у вас будет возможность добавлять новый пост в базу данных и редактировать или удалять существующий пост.

Чтобы создать веб-приложение с помощью Flask:

1. [Установите Flask](#install-flask).
1. [Создайте и запустите приложение](#create-app).
1. [Создайте и настройте шаблоны HTML](#html-templates).
1. [Настройте базу данных](#configure-data-base).
1. [Настройте отображение постов блога](#display-posts).
1. [Добавьте действия с постами](#add-post-actions).
1. [Подведите итоги](#results).

Если веб-приложение вам больше не нужно, [удалите все используемые им ресурсы](#clear-out).

## Перед началом работы {#before-you-begin}

### Создайте и настройте виртуальную машину {#create-setup-vm}

Веб-приложение будет размещаться на виртуальной машине в {{ yandex-cloud }}.

1. [Создайте ВМ](../../compute/operations/vm-create/create-linux-vm.md) на основе [Ubuntu](/marketplace?tab=software&search=Ubuntu&categories=os) с публичным доступом, например, [Ubuntu 22.04 LTS](/marketplace/products/yc/ubuntu-22-04-lts).

1. Убедитесь, что можете [подключиться](../../compute/operations/vm-connect/ssh.md) к ВМ по [SSH](../../glossary/ssh-keygen.md).

1. [Добавьте правило](../../vpc/operations/security-group-add-rule.md), разрешающее входящий HTTP-трафик на порт `5000`, к [группе безопасности](../../vpc/concepts/security-groups.md) ВМ. На этом порту по умолчанию работает сервер приложения Flask.

### Создайте и активируйте виртуальное окружение {#create-activate-venv}

Виртуальное окружение обеспечивает изолированное пространство для проектов Python на сервере, то есть все необходимые зависимости — исполняемые файлы, библиотеки и прочие файлы копируются в некоторый выбранный каталог, а приложение использует их, а не установленные в системе. Это позволяет обеспечить стабильность среды разработки и чистоту основной системы.

Для создания виртуального окружения используйте стандартной модуль библиотеки Python 3 `venv`.

1. [Подключитесь](../../compute/operations/vm-connect/ssh.md) к ВМ.

1. Создайте и перейдите в директорию проекта `flask_blog`, в которой будет находиться приложение:

    ```bash
    mkdir flask_blog && \
    cd flask_blog
    ```

1. Создайте виртуальное окружение `env`:

    ```bash
    python3 -m venv env
    ```

1. Активируйте созданное виртуальное окружение:

    ```bash
    source env/bin/activate
    ```

    После активации виртуального окружения в командной строке появится префикс с именем окружения:

    ```text
    (env) yandexcloud@ubuntu:~/flask_blog$
    ```

    {% note info %}

    Вы можете использовать систему контроля версий [Git](https://git-scm.com/) для эффективного управления и отслеживания процесса разработки проекта. В таком случае добавьте директорию `env` в файл `.gitignore`, чтобы не отслеживать файлы, не связанные с проектом.

    {% endnote %}

    Чтобы деактивировать виртуальное окружение, выполните команду:

    ```text
    deactivate
    ```

## Установите Flask {#install-flask}

1. Чтобы установить Flask, выполните команду:

    ```bash
    pip install flask
    ```

1. Убедитесь, что Flask установлен:

    ```bash
    python3 -c "import flask; print(flask.__version__)"
    ```

    В результате вы увидите версию установленного пакета `flask`.

## Создайте и запустите приложение {#create-app}

Создайте небольшое веб-приложение внутри файла Python и запустите его для начала работы сервера, который отобразит определенную информацию в браузере.

1. Создайте и откройте файл с именем `app.py`. Используйте `vim` или любой другой текстовый редактор:

    ```bash
    vim app.py
    ```

    Этот файл будет служить минимальным примером того, как обрабатывать HTTP-запросы.

1. Добавьте в файл следующий код:

    ```python
    from flask import Flask

    app = Flask(__name__)


    @app.route('/')
    def hello():
        return 'Hello, World!'
    ```

    В этом коде:

    * Импортируется [объект `Flask`](https://flask.palletsprojects.com/en/latest/api/#flask.Flask) из пакета `flask`.
    * Создается экземпляр приложения Flask с именем `app`. Специальная переменная `__name__` содержит имя текущего модуля Python и указывает экземпляру его расположение. Это необходимо, так как Flask устанавливает ряд путей внутри приложения.
    * С помощью [декоратора](https://en.wikipedia.org/wiki/Python_syntax_and_semantics#Decorators) `@app.route('/')` стандартная функция Python превращается в функцию просмотра Flask, конвертирующую возвращаемое значение в HTTP-ответ, который отображается клиентом HTTP, например, веб-браузером. Значение `'/'` в `@app.route()​​` означает, что эта функция будет отвечать на веб-запросы для URL `/`, который является основным URL-адресом.
    * Описана простейшая функция `hello()`, которая возвращает строку `Hello, World!` в качестве ответа.

    Сохраните и закройте файл.

1. Настройте переменные окружения `Flask`:

    ```bash
    export FLASK_APP=app && \
    export FLASK_DEBUG=true
    ```

    `FLASK_APP=app` указывает, где находится приложение — файл `app.py`.
    `FLASK_DEBUG=true` указывает, что приложение необходимо запустить в режиме разработки.

1. Запустите приложение:

    ```bash
    flask run --host=0.0.0.0
    ```

    Параметр `--host=0.0.0.0` запускает сервер приложения на всех IP-адресах вашей ВМ. Без указания этого параметра, вы не сможете обратиться к приложению по публичному IP-адресу ВМ.

    Результат:

    ```text
     * Serving Flask app 'app'
     * Debug mode: on
    WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
     * Running on all addresses (0.0.0.0)
     * Running on http://127.0.0.1:5000
     * Running on http://<внутренний IP-адрес ВМ>:5000
    Press CTRL+C to quit
     * Restarting with stat
     * Debugger is active!
     * Debugger PIN: 884-371-225
    ```

    Для нас важна следующая информация:

    * Название работающего приложения — `app`.
    * `Debug mode: on` означает, что отладчик Flask работает. Эта функция полезна при разработке, так как при возникновении проблем она выдает детализированные сообщения об ошибках, что упрощает работу по их устранению.
    * Приложение работает на всех адресах.

1. Откройте браузер и введите URL `​​​​​​http://<публичный IP-адрес ВМ>:5000/​​​​​`​. Вы увидите строку `Hello, World!` в качестве ответа. Это подтверждает, что ваше приложение успешно работает и к нему можно обращаться из интернета.

{% note warning %}

Flask использует простой веб-сервер для обслуживания приложения в среде разработки. Он предназначен для тестирования и обнаружения ошибок и не должен использоваться при развертывании в производственной среде. Подробнее см. в [документации Flask](https://flask.palletsprojects.com/en/latest/deploying/).

{% endnote %}

Теперь вы можете оставить сервер разработки работать и открыть другое окно терминала для дальнейшей настройки приложения.

При открытии нового окна терминала:

1. [Подключитесь](../../compute/operations/vm-connect/ssh.md) к ВМ.
1. Перейдите в директорию проекта `flask_blog`.
1. Активируйте виртуальное окружение.
1. Настройте переменные окружения `FLASK_APP` и `FLASK_DEBUG`.

{% note info %}

Если сервер разработки приложения Flask работает, невозможно запустить еще одно приложение Flask с помощью такой же команды `flask run`. Это связано с тем, что `flask run` по умолчанию использует номер порта `5000`, следовательно, когда он занят, невозможно запустить другое приложение. Поэтому вы увидите ошибку:

```text
Address already in use
Port 5000 is in use by another program. Either identify and stop that program, or start the server with a different port.
```

Для решения этой проблемы остановите работающий в настоящий момент сервер с помощью сочетания клавиш `CTRL+C`, или, если вам нужно, чтобы оба сервера работали одновременно, передайте другой номер порта в аргументе `--port` при запуске приложения, например, `5001`:

```bash
flask run --host=0.0.0.0 --port=5001
```

При этом к группе безопасности ВМ [добавьте правило](../../vpc/operations/security-group-add-rule.md), разрешающее входящий HTTP-трафик на указанный порт.

{% endnote %}

## Создайте и настройте шаблоны HTML {#html-templates}

Сейчас приложение отображает только простое сообщение, однако большинство веб-приложений используют HTML для отображения информации для посетителей. Добавьте в приложение файлы-шаблоны HTML, которые можно отобразить в браузере.

Flask предлагает [вспомогательную функцию `render_template()`](https://flask.palletsprojects.com/en/latest/api/#flask.render_template), которая позволяет использовать [механизм шаблонов Jinja](https://jinja.palletsprojects.com/en/latest/templates/). С помощью этой функции вы сможете настроить шаблоны HTML в файлах `.html`, а также применять логику в коде HTML. Вы будете использовать шаблоны HTML для создания всех страниц приложения, например, главной страницы, где будут отображаться текущие посты блога, страницы поста блога, страницы, на которой пользователь сможет добавить новый пост и т. д.

### Создайте шаблон HTML {#create-html-template}

1. Откройте файл `app.py`:

    ```bash
    vim app.py
    ```

1. Замените его содержимое на следующий код:

    ```python
    from flask import Flask, render_template

    app = Flask(__name__)

    @app.route('/')
    def index():
        return render_template('index.html')
    ```

    В этом коде:

    * Дополнительно импортируется вспомогательная функция `render_template()` для визуализации шаблонов HTML.
    * Простейшая функция `hello()` заменена на функцию `index()`, которая возвращает результат вызова вспомогательной функции `render_template()` с `index.html​`​ в качестве аргумента. Аргумент указывает функции `render_template()` искать файл с именем `index.html` в директории шаблонов. И директория, и файл еще отсутствуют. Вы получите сообщение об ошибке, если запустите приложение на этом этапе. Так вы столкнетесь с этой часто встречающейся ошибкой и научитесь ее исправлять, создав необходимые директорию и файл.

    Сохраните и закройте файл.

1. Обновите главную страницу приложения `​​​​​​http://<публичный IP-адрес ВМ>:5000/​​​​​`.

    Отладчик сообщит об ошибке:

    ```text
    TemplateNotFound
    jinja2.exceptions.TemplateNotFound: index.html
    ```

    А ниже укажет файлы и конкретные строки кода с ошибкой. Следуют обратить внимание на:

    ```text
    File "/home/<имя пользователя ВМ>/flask_blog/app.py", line 7, in index
      return render_template('index.html')
    ```

    Теперь добавьте шаблон в проект и устраните ошибку.

1. Создайте директорию с именем `templates` в директории проекта `flask_blog`:

    ```bash
    mkdir templates
    ```

1. Создайте и откройте файл-шаблон с именем `index.html` в директории `templates`:

    ```bash
    vim templates/index.html
    ```

1. Добавьте в файл следующий код:

    ```html
    <!DOCTYPE html>
    <html lang="en">
      <head>
        <meta charset="UTF-8">
        <title>FlaskBlog</title>
      </head>
      <body>
        <h1>Welcome to FlaskBlog</h1>
      </body>
    </html>
    ```

    В этом HTML коде описана простейшая страница с заголовком `FlaskBlog` и содержимым `Welcome to FlaskBlog` в заголовке первого уровня.

    Сохраните и закройте файл.

1. Обновите главную страницу приложения `​​​​​​http://<публичный IP-адрес ВМ>:5000/​​​​​`.

    Браузер должен отобразить текст `Welcome to FlaskBlog` в виде заголовка.

1. Помимо папки `templates` веб-приложения Flask также обычно имеют папку `static` для хостинга статичных файлов, таких как файлы CSS, JavaScript и изображений, которые использует приложение. Создайте директорию с именем `static` в директории проекта `flask_blog`:

    ```bash
    mkdir static
    ```

1. Создайте директорию с именем `css` в директории `static`:

    ```bash
    mkdir static/css
    ```

    {% note info %}

    Обычно это делается для упорядочивания статичных файлов в специальных папках. Например, файлы JavaScript обычно находятся в директории с именем `js`, изображения — в директории `images` (или `img`) и т. д.

    {% endnote %}

1. Создайте и откройте файл с именем `style.css` в директории `static/css`:

    ```bash
    vim static/css/style.css
    ```

1. Добавьте в файл следующий код:

    ```css
    h1 {
        border: 2px #eee solid;
        color: #fc3d17;
        text-align: center;
        padding: 10px;
    }
    ```

    Этот код CSS добавит границу, изменит цвет на красный, выравняет текст по центру и добавит небольшое дополнение к меткам `<h1>`.

    Сохраните и закройте файл.

1. Откройте шаблон `index.html`:

    ```bash
    vim templates/index.html
    ```

1. Добавьте ссылку на файл `style.css` внутри раздела `<head>`:

    ```html
    ...
      <head>
        <meta charset="UTF-8">
        <link rel="stylesheet" href="{{ url_for('static', filename= 'css/style.css') }}">
        <title>FlaskBlog</title>
      </head>
    ...
    ```

    В ссылке используется [вспомогательная функция `url_for()`](https://flask.palletsprojects.com/en/latest/api/#flask.url_for)​​​ для генерации адреса расположения файла. Первый аргумент указывает, что ссылка связана со статическими файлами, а второй — путь к файлу в директории для статических файлов, по умолчанию это директория `static`.

    Сохраните и закройте файл.

1. Обновите главную страницу приложения `​​​​​​http://<публичный IP-адрес ВМ>:5000/​​​​​`.

    Вы увидите, что цвет текста `Welcome to FlaskBlog`​​​ теперь красный и расположен по центру внутри рамки.

Используйте язык CSS для оформления стиля приложения и придавайте ему более привлекательный вид с помощью своего собственного дизайна. Однако, если вы не веб-дизайнер или не знакомы с CSS, воспользуйтесь [инструментарием Bootstrap](https://getbootstrap.com/), который предлагает простые в использовании компоненты для оформления приложения. Дальше в проекте мы будем использовать Bootstrap.

### Настройте другие шаблоны HTML {#configure-html-templates}

Вы могли догадаться, что создание другого шаблона HTML будет означать повторение основной части кода HTML, который уже написан в шаблоне `index.html`. Избежать ненужного повторения кода можно благодаря файлу _базового шаблона_, из которого наследуются все остальные шаблоны HTML. Подробнее см. в [документации Jinja](https://jinja.palletsprojects.com/en/latest/templates/#template-inheritance).

1. Создайте и откройте файл-шаблон с именем `base.html` в директории `templates`:

    ```bash
    vim templates/base.html
    ```

    Этот файл будет _базовым шаблоном_ для всех остальных шаблонов HTML.

1. Добавьте в файл следующий код:

    ```html
    <!doctype html>
    <html lang="en">
      <head>
        <!-- Required meta tags -->
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Bootstrap CSS -->
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-rbsA2VBKQhggwzxH7pPCaAqO46MgnOM80zW1RWuH61DGLwZJEdK2Kadq2F9CUG65" crossorigin="anonymous">

        <title>{% block title %} {% endblock %}</title>
      </head>
      <body>
        <nav class="navbar navbar-expand-lg bg-light">
          <div class="container-fluid">
            <a class="navbar-brand" href="{{ url_for('index')}}">FlaskBlog</a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
              <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarSupportedContent">
              <ul class="navbar-nav me-auto mb-2 mb-lg-0">
                <li class="nav-item">
                  <a class="nav-link active" href="#">About</a>
                </li>
              </ul>
            </div>
          </div>
        </nav>
        <div class="container">
          {% block content %} {% endblock %}
        </div>

        <!-- Optional JavaScript -->
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-kenU1KFdBIe4zVF0s0G1M5b4hcpxyD9F7jL+jjXkk+Q2h455rYXK/7HAuoJl+0I4" crossorigin="anonymous"></script>
      </body>
    </html>
    ```

    Этот код состоит из стандартного HTML и кода, необходимого для Bootstrap. Теги `<meta>` содержат информацию для веб-браузера, тег `<link>` привязывает файлы CSS Bootstrap, а тег `<script>` является ссылкой на код JavaScript, который позволяет выполнить ряд дополнительных функций Bootstrap. Подробнее см. в [документации Bootstrap](https://getbootstrap.com/docs/).

    Однако, следующие части относятся к механизму шаблонов Jinja:

    * `{% block title %} {% endblock %}` — [блок](https://jinja.palletsprojects.com/en/latest/templates/#blocks), замещающий заголовок. Вы будете использовать его в других шаблонах для написания пользовательского заголовка каждой страницы в приложении без перезаписи раздела `<head>`.
    * `{{ url_for('index')}}` — вызов функции, который возвращает URL для функции просмотра `index()`. Этот вариант отличается от прошлого вызова `url_for()`, который использовался для привязки файла CSS. Сейчас он использует только один аргумент — имя функции просмотра, и привязывается к маршруту, связанному с функцией, вместо файла.
    * `{% block content %} {% endblock %}` — еще один блок, который будет замещен контентом в зависимости от дочернего шаблона.

    Сохраните и закройте файл.

1. Теперь, когда у вас есть _базовый шаблон_, воспользуйтесь наследованием. Откройте шаблон `index.html`:

    ```bash
    vim templates/index.html
    ```

1. Замените его содержимое на следующий код:

    ```html
    {% extends 'base.html' %}

    {% block title %} Welcome to FlaskBlog {% endblock %}

    {% block content %}
      <h1> Welcome to FlaskBlog </h1>
    {% endblock %}
    ```

    В этом коде:

    * `{% extends 'base.html' %}` — указывает, что текущий шаблон используется для наследования из шаблона `base.html`.
    * `{% block title %} ... {% endblock %}` — содержит заголовок для замены соответствующего блока в `base.html`.
    * `{% block content %} ... {% endblock %}` — содержит контент для замены соответствующего блока в `base.html`.

    Блоки с заголовком и контентом своего рода переменные в _базовом шаблоне_. И для его использования передаются значения этих переменных.

    Так как один и тот же текст `Welcome to FlaskBlog` используется и в качестве названия страницы, и в качестве заголовка, который появляется под панелью навигации, то этот код можно сократить, избежав повторения:

    ```html
    {% extends 'base.html' %}

    {% block content %}
      <h1>{% block title %} Welcome to FlaskBlog {% endblock %}</h1>
    {% endblock %}
    ```

    Сохраните и закройте файл.

1. Обновите главную страницу приложения `​​​​​​http://<публичный IP-адрес ВМ>:5000/​​​​​`.

    Вы увидите страницу с панелью навигации и оформленным заголовком.

## Настройте базу данных {#configure-data-base}

Настройте базу данных для хранения данных, в случае с вашим приложением — это посты блога. Также наполните базу несколькими примерами.

Вы будете использовать файл [базы данных SQLite](https://sqlite.org/) для хранения ваших данных, так как модуль [sqlite3](https://docs.python.org/3/library/sqlite3.html), который мы будем использовать для взаимодействия с базой, уже присутствует в стандартной библиотеке Python.

Поскольку данные в SQLite хранятся в таблицах и столбцах, а ваши данные состоят из постов блога, вам понадобится создать таблицу с именем `posts` и необходимыми столбцами. Создайте файл `.sql`, содержащий команды SQL для создания таблицы `posts` с несколькими столбцами, и используйте его для создания базы данных.

1. Создайте и откройте файл с именем `schema.sql` в директории проекта `flask_blog`:

    ```bash
    vim schema.sql
    ```

    Этот файл будет схемой SQL для базы данных.

1. Добавьте в файл следующий код:

    ```sql
    DROP TABLE IF EXISTS posts;

    CREATE TABLE posts (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        created TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
        title TEXT NOT NULL,
        content TEXT NOT NULL
    );
    ```

    В этом коде две SQL-команды:

    * `DROP TABLE IF EXISTS posts;` — удаляет таблицу с именем `posts`, если она существует. Это стандартный шаг перед созданием таблицы для избежания ошибок.
    * `CREATE TABLE posts` — создает таблицу posts со следующими столбцами:
        * `id` — целое число, которое представляет собой _первичный ключ_ (`PRIMARY KEY`). `AUTOINCREMENT` означает, что значением будет уникальное число, сгенерированное автоматически при вставке новой записи (поста блога) в таблицу.
        * `created` — время создания новой записи. `NOT NULL` означает, что значение в столбце не может быть пустым, а `DEFAULT CURRENT_TIMESTAMP`​​​ — что по умолчанию значением будет время добавления новой записи в таблицу. Благодаря этому, вам не нужно будет вручную указывать время создания постов — оно будет вставляться автоматически.
        * `title` — заголовок поста.
        * `content` — содержание поста.

    Сохраните и закройте файл.

    {% note info %}

    При использовании этих SQL-команд будет удалено все содержимое базы данных, поэтому не записывайте ничего важного в веб-приложении до окончания изучения этого руководства.

    {% endnote %}

1. Создайте и откройте файл с именем `init_db.py` в директории проекта `flask_blog`:

    ```bash
    vim init_db.py
    ```

    С помощью этого файла сформируется файл базы данных SQLite для приложения.

1. Добавьте в файл следующий код:

    ```python
    import sqlite3

    connection = sqlite3.connect('database.db')


    with open('schema.sql') as f:
        connection.executescript(f.read())

    cur = connection.cursor()

    cur.execute("INSERT INTO posts (title, content) VALUES (?, ?)",
                ('First Post', 'Content for the first post')
                )

    cur.execute("INSERT INTO posts (title, content) VALUES (?, ?)",
                ('Second Post', 'Content for the second post')
                )

    connection.commit()
    connection.close()
    ```

    В этом коде:
    * Импортируется модуль `sqlite3`.
    * Открывается соединение с файлом базы данных `database.db`, который будет создан сразу после запуска файла Python.
    * С помощью функции `open()` открывается схема SQL `schema.sql`. Оформляется содержание схемы с помощью метода `executescript()`, при котором выполняется несколько операторов SQL одновременно, что формирует таблицу `posts`.
    * С помощью метода `cursor()` создается объект курсор, для отправки запросов к базе данных.
    * С помощью методов `execute()` выполняются два оператора SQL `INSERT`, которые добавляют два поста блога в таблицу `posts`.
    * Фиксируются изменения.
    * Закрывается соединение.

    Сохраните и закройте файл.

1. Запустите выполнение файла с помощью `python`:

    ```bash
    python init_db.py
    ```

1. Убедитесь, что в директории проекта `flask_blog` появился файл с именем `database.db`.

## Настройте отображение постов блога {#display-posts}

Настройте отображение всех постов блога на главной странице приложения и отдельного поста блога по его ID.

### Отобразите все посты блога {#display-all-posts}

Измените функцию просмотра `index()` и шаблон `index.html`, чтобы отобразить все посты, внесенные в базу данных.

1. Откройте файл `app.py`:

    ```bash
    vim app.py
    ```

1. Замените его содержимое на следующий код:

    ```python
    import sqlite3
    from flask import Flask, render_template

    def get_db_connection():
        conn = sqlite3.connect('database.db')
        conn.row_factory = sqlite3.Row
        return conn

    app = Flask(__name__)

    @app.route('/')
    def index():
        conn = get_db_connection()
        posts = conn.execute('SELECT * FROM posts').fetchall()
        conn.close()
        return render_template('index.html', posts=posts)
    ```

    В этом коде дополнительно:

    * Импортируется модуль `sqlite3`.
    * Создается функция `get_db_connection()`, которая:
        * создает объект подключения `conn`, соединяющийся с файлом базы данных `database.db`;
        * получает доступ к данным базы и присваивает их атрибуту объекта `conn`;
        * возвращает объект подключения `conn`, который будет использоваться для доступа к данным базы;
    * В измененной функции `index()`:
        * объекту `conn` присваивается результат выполнения функции `get_db_connection(`);
        * объекту `posts` присваивается результат выполнения SQL-запроса для выбора всех записей из таблицы `posts` с помощью метода `fetchall();
        * закрывается подключение к базе данных с помощью метода `close()`;
        * возвращается результат отображения шаблона `index.html`, в который в качестве аргумента передается объект `posts`.

    Сохраните и закройте файл.

1. Откройте шаблон `index.html`:

    ```bash
    vim templates/index.html
    ```

1. Замените его содержимое на следующий код:

    ```html
    {% extends 'base.html' %}

    {% block content %}
      <h1>{% block title %} Welcome to FlaskBlog {% endblock %}</h1>
      {% for post in posts %}
        <a href="#">
          <h2>{{ post['title'] }}</h2>
        </a>
        <span class="badge text-bg-primary">{{ post['created'] }}</span>
        <hr>
      {% endfor %}
    {% endblock %}
    ```

    В этом коде добавляются:

    * ```{% for post in posts %}``` — [цикл `for` Jinja](https://jinja.palletsprojects.com/en/latest/templates/#for), который аналогичен циклу `for` в Python, за исключением того, что позже он закрывается с помощью синтаксиса ```{% endfor %}```. Он используется для отображения каждого элемента в списке `posts`, который был передан функцией `index()` в файле `app.py`. Внутри этого цикла отображается название поста в заголовке `<h2>​​`​ в теге ссылки `<a>`. Позже эти ссылки будут использоваться для перехода к каждому посту в отдельности.
    * `not_var{{ ... }}​​​​​​` — разделитель переменной, с помощью которого отображается заголовок. Так как `post` представляет собой словарь, можно получить доступ к заголовку поста с помощью выражения `post['title']`.
    * Аналогично отображается дата создания поста `post['created']`, оформленная в классе `badge`.

    Сохраните и закройте файл.

1. Обновите главную страницу приложения  `​​​​​​http://<публичный IP-адрес ВМ>:5000/​​​​​`.

    Вы увидите два поста, которые вы добавили в базу данных.

### Отобразите отдельные посты блога {#display-individual-posts}

Создайте новый маршрут Flask с функцией просмотра и новым шаблоном HTML для отображения отдельного поста блога по его ID. Благодаря этому, на URL `http://<публичный IP-адрес ВМ>:5000/ID` будет отображаться пост с соответствующим номером `ID`, если такой существует.

1. Откройте файл `app.py`:

    ```bash
    vim app.py
    ```

1. Замените его содержимое на следующий код:

    ```python
    import sqlite3
    from flask import Flask, render_template
    from werkzeug.exceptions import abort

    def get_db_connection():
        conn = sqlite3.connect('database.db')
        conn.row_factory = sqlite3.Row
        return conn

    def get_post(post_id):
        conn = get_db_connection()
        post = conn.execute('SELECT * FROM posts WHERE id = ?',
                            (post_id,)).fetchone()
        conn.close()
        if post is None:
            abort(404)
        return post

    app = Flask(__name__)

    @app.route('/')
    def index():
        conn = get_db_connection()
        posts = conn.execute('SELECT * FROM posts').fetchall()
        conn.close()
        return render_template('index.html', posts=posts)

    @app.route('/<int:post_id>')
    def post(post_id):
        post = get_post(post_id)
        return render_template('post.html', post=post)
    ```

    В этом коде дополнительно:

    * Импортируется функция [abort()](https://werkzeug.palletsprojects.com/en/latest/exceptions/#werkzeug.exceptions.abort) из библиотеки [Werkzeug](https://werkzeug.palletsprojects.com/en/latest/). С ее помощью будет создаваться ответ Flask с сообщением `404 Not Found​​`​, если пост блога с указанным ID не существует.
    * Создается функция `get_post()` с аргументом `post_id`, который определяет, какой пост блога предназначен для возврата. В этой функции:
        * объекту `conn` присваивается результат выполнения функции `get_db_connection(`);
        * объекту `post` присваивается результат выполнения SQL-запроса для выбора поста с указанным значением `post_id` с помощью метода `fetchone()`;
        * закрывается подключение к базе данных с помощью метода `close()`;
        * если переменная `post` имеет значение `None`​​​, т. е. в базе данных результат не найден, используется функция `abort()` для ответа с помощью кода ошибки 404, и выполнение функция завершается;
        * если же пост был найден — возвращается значение переменной `post`;
    * Создается функция просмотра `post(post_id)`. В этой функции:
        * декоратором добавляется [правило переменной](https://flask.palletsprojects.com/en/latest/quickstart/#variable-rules) `<int:post_id>`, которое указывает, что часть после `/` представляет собой положительное целое число (отмеченное конвертером int), использующееся функцией просмотра.
        * объекту `post` с помощью функции `get_post(post_id)` присваивается содержимое поста блога с указанным ID.
        * возвращается результат отображения шаблона `post.html`, в который в качестве аргумента передается объект `post`.

    Сохраните и закройте файл.

1. Создайте и откройте файл-шаблон с именем `post.html` в директории `templates`:

    ```bash
    vim templates/post.html
    ```

1. Добавьте в файл следующий код:

    ```html
    {% extends 'base.html' %}

    {% block content %}
        <h2>{% block title %} {{ post['title'] }} {% endblock %}</h2>
        <p>{{ post['content'] }}</p>
        <span class="badge text-bg-primary">{{ post['created'] }}</span>
    {% endblock %}
    ```

    Этот код аналогичен коду в шаблоне `index.html`, за исключением того, что он будет отображать единичный пост с его содержимым `post['content']`.

    Сохраните и закройте файл.

1. Перейдите к следующим URL-адресам для просмотра двух постов из базы данных и страницы, которая указывает пользователю, что запрашиваемый пост блога не обнаружен, так как пост с номером ID `3` не существует:

    ```text
    http://<публичный IP-адрес ВМ>:5000/1
    http://<публичный IP-адрес ВМ>:5000/2
    http://<публичный IP-адрес ВМ>:5000/3
    ```

    Теперь можно привязать заголовок каждого поста к соответствующей странице с его содержимым.

1. Откройте шаблон `index.html`:

    ```bash
    vim templates/index.html
    ```

1. Замените значение атрибута `href` с `#` на `{{ url_for('post', post_id=post['id']) }}`, чтобы цикл `for` выглядел следующим образом:

    ```html
    ...
      {% for post in posts %}
        <a href="{{ url_for('post', post_id=post['id']) }}">
          <h2>{{ post['title'] }}</h2>
        </a>
        <span class="badge text-bg-primary">{{ post['created'] }}</span>
        <hr>
      {% endfor %}
    ...
    ```

    Функция `url_for()` принимает в качестве аргументов `'post'` и `post_id` и возвращает правильный URL для каждого поста на основании его ID.

    Сохраните и закройте файл.

1. Перейдите к главной странице приложения `​​​​​​http://<публичный IP-адрес ВМ>:5000/​​​​​` и убедитесь, что заголовки постов работают как ссылки для перехода к их содержимому.

## Добавьте действия с постами {#add-post-actions}

Добавьте возможность писать новые посты в блог, добавляя их в базу данных, редактировать существующие посты, а также удалять ненужные.

### Создание нового поста {#create-post}

Создайте страницу, на которой можно создать пост, указав его заголовок и содержание.

1. Откройте файл `app.py`:

    ```bash
    vim app.py
    ```

1. Замените его содержимое на следующий код:

    ```python
    import sqlite3
    from flask import Flask, render_template, request, url_for, flash, redirect
    from werkzeug.exceptions import abort

    def get_db_connection():
        conn = sqlite3.connect('database.db')
        conn.row_factory = sqlite3.Row
        return conn

    def get_post(post_id):
        conn = get_db_connection()
        post = conn.execute('SELECT * FROM posts WHERE id = ?',
                            (post_id,)).fetchone()
        conn.close()
        if post is None:
            abort(404)
        return post

    app = Flask(__name__)
    app.config['SECRET_KEY'] = 'your secret key'

    @app.route('/')
    def index():
        conn = get_db_connection()
        posts = conn.execute('SELECT * FROM posts').fetchall()
        conn.close()
        return render_template('index.html', posts=posts)

    @app.route('/<int:post_id>')
    def post(post_id):
        post = get_post(post_id)
        return render_template('post.html', post=post)

    @app.route('/create', methods=('GET', 'POST'))
    def create():
        return render_template('create.html')
    ```

    В этом коде дополнительно:

    * Импортируются:
        * [глобальный объект `request`](https://flask.palletsprojects.com/en/latest/api/#flask.request) для доступа к входящим данным запроса, которые будут подаваться через форму HTML;
        * [функция `url_for()`](https://flask.palletsprojects.com/en/latest/api/#flask.url_for) для генерирования URL-адресов;
        * [функция `flash()`](https://flask.palletsprojects.com/en/latest/api/#flask.flash) для появления сообщения при обработке запроса;
        * [функция `redirect()`](https://flask.palletsprojects.com/en/latest/api/#flask.redirect) для перенаправления клиента в другое расположение;
    * Добавляется конфигурация секретного ключа `SECRET_KEY` через объект `app.config`. Она нужна для функции `flash()`, которая хранит всплывающие сообщения в сеансе браузера клиента. Секретный ключ используется для защищенных сеансов, что позволяет Flask запоминать информацию от одного запроса к другому, например, переходить от страницы нового поста к главной странице приложения. Пользователь может получить доступ к информации, хранящейся в сеансе, но не может изменить ее без секретного ключа. Поэтому никогда никому не передавайте доступ к вашему секретному ключу. Секретный ключ должен представлять собой длинную случайную строку. Подробнее см. в [документации Flask](https://flask.palletsprojects.com/en/latest/api/#sessions).
    * Функция просмотра `create()`, которая отображает шаблон `create.html`, использующий для создания нового поста в блоге. Ее декоратор создает маршрут `/create`, который принимает GET и POST запросы. GET запросы принимаются по умолчанию. Для того чтобы также принимать POST запросы, которые посылаются браузером при подаче форм, передается кортеж с приемлемыми типами запросов в аргумент `methods`.

    Сохраните и закройте файл.

1. Создайте и откройте файл-шаблон с именем `create.html` в директории `templates`:

    ```bash
    vim templates/create.html
    ```

1. Добавьте в файл следующий код:

    ```html
    {% extends 'base.html' %}

    {% block content %}
    <h1>{% block title %} Create a New Post {% endblock %}</h1>

    <form method="post">
      <div class="mb-3">
        <label for="title" class="col-sm-2 col-form-label">Title</label>
        <input type="text" name="title"
               placeholder="Post title" class="form-control"
               value="{{ request.form['title'] }}">
      </div>

      <div class="mb-3">
        <label for="content" class="col-sm-2 col-form-label">Content</label>
        <textarea name="content" placeholder="Post content"
                  class="form-control" rows="3">{{ request.form['content'] }}</textarea>
      </div>
      <div class="mb-3">
        <button type="submit" class="btn btn-primary">Submit</button>
      </div>
    </form>
    {% endblock %}
    ```

    Большая часть этого кода — стандартный HTML. Он будет отображать поля ввода для заголовка, содержания поста и кнопку для отправки формы.

    Значением заголовка поста является `{{ request.form['title'] }}`, а содержания — `{{ request.form['content'] }}`. Это сделано для того, чтобы вводимые вами данные не потерялись, если что-то пойдет не по плану. Например, если вы напишете длинный пост и забудете дать ему заголовок, отобразится сообщение, информирующее вас о том, что указание заголовка является обязательным. Это произойдет без потери уже написанного поста, так как он будет сохранен в глобальном объекте `request`, к которому у вас есть доступ в шаблонах.

    Сохраните и закройте файл.

1. Перейдите к URL `http://<публичный IP-адрес ВМ>:5000/create` и убедитесь, что отображается страница **Create a New Post** с полями для заголовка, содержания поста и кнопкой для отправки формы.

    Эта форма отправляет POST запрос в функцию просмотра `create()`. Но в функции пока отсутствует код обработки POST запроса, поэтому после заполнения формы и ее отправки ничего не произойдет. Исправьте это, изменив функцию просмотра `create()`.

1. Откройте файл `app.py`:

    ```bash
    vim app.py
    ```

1. Замените содержимое функции просмотра `create()` на следующий код:

    ```python
    ...
    @app.route('/create', methods=('GET', 'POST'))
    def create():
        if request.method == 'POST':
            title = request.form['title']
            content = request.form['content']

            if not title:
                flash('Title is required!')
            else:
                conn = get_db_connection()
                conn.execute('INSERT INTO posts (title, content) VALUES (?, ?)',
                             (title, content))
                conn.commit()
                conn.close()
                return redirect(url_for('index'))

        return render_template('create.html')
    ```

    В этой функции:

    * Выражением `if` обеспечивается, что следующий за ним код выполняется только в случае, если запрос является POST запросом через сравнение `request.method == 'POST'`.
    * Из объекта `request.form`, который дает доступ к данным формы в запросе, извлекается отправленные заголовок и содержание.
    * Если заголовок не указан, будет выполнено условие `if not title`, и отобразится сообщение для пользователя с информацией о необходимости заголовка.
    * Если заголовок указан:
        * открывается подключение с помощью функции `get_db_connection()` и вставляются полученные заголовок и содержание в таблицу `posts`;
        * фиксируются изменения;
        * закрывается соединение;
        * происходит перенаправление на главную страницу приложения с помощью функции `redirect()`.

    Сохраните и закройте файл.

1. Перейдите к URL `http://<публичный IP-адрес ВМ>:5000/create`, введите любой заголовок и контент и отправьте форму. Вас перенаправит на главную страницу приложения, где вы увидите все посты, включая новый.

    Теперь отобразите всплывающее сообщение и добавьте ссылку на панель навигации в шаблоне `base.html` для добавления новых постов.

1. Откройте шаблон `base.html`:

    ```bash
    vim templates/base.html
    ```

1. Добавьте тег `<li>` перед ссылкой `About`​​ внутри тега `<nav>` и цикл `for` непосредственно над блоком `content` для отображения всплывающих сообщений под панелью навигации:

    ```html
    ...
        <nav class="navbar navbar-expand-lg bg-light">
          <div class="container-fluid">
            <a class="navbar-brand" href="{{ url_for('index')}}">FlaskBlog</a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
              <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarSupportedContent">
              <ul class="navbar-nav me-auto mb-2 mb-lg-0">
                <li class="nav-item">
                  <a class="nav-link active" href="{{url_for('create')}}">New Post</a>
                </li>
                <li class="nav-item">
                  <a class="nav-link active" href="#">About</a>
                </li>
              </ul>
            </div>
          </div>
        </nav>
        <div class="container">
          {% for message in get_flashed_messages() %}
            <div class="alert alert-danger">not_var{{ message }}</div>
          {% endfor %}
          {% block content %} {% endblock %}
        </div>
    ...
    ```

    Всплывающие сообщения доступны в специальной функции `get_flashed_messages()`​​​​​​, имеющейся в Flask.

    Сохраните и закройте файл.

    Теперь на панели навигации будет присутствовать элемент **New Post**, который связан с маршрутом `/create`.

1. Перейдите на главную страницу приложения  `http://<публичный IP-адрес ВМ>:5000` и убедитесь, что с помощью ссылки на панели навигации **New Post** можно создать новый пост.

### Редактирование поста {#edit-post}

Создайте страницу, на которой можно отредактировать существующий пост.

1. Откройте файл `app.py`:

    ```bash
    vim app.py
    ```

1. Добавьте функцию просмотра `edit()`:

    ```python
    ...
    @app.route('/<int:id>/edit', methods=('GET', 'POST'))
    def edit(id):
        post = get_post(id)

        if request.method == 'POST':
            title = request.form['title']
            content = request.form['content']

            if not title:
                flash('Title is required!')
            else:
                conn = get_db_connection()
                conn.execute('UPDATE posts SET title = ?, content = ?'
                            ' WHERE id = ?',
                            (title, content, id))
                conn.commit()
                conn.close()
                return redirect(url_for('index'))

        return render_template('edit.html', post=post)
    ```

    Редактирование существующего поста аналогично созданию нового, поэтому эта функция просмотра будет почти такой же, как и функция просмотра `create()`. Редактируемый пост, определяется с помощью URL, а Flask будет передавать функции `edit()` номер ID через аргумент `id`. Это же значение передается функции `get_post()`, чтобы получить пост из базы данных по указанному ID. Новые данные будут поступать в POST запросе, который обрабатывается внутри условия `if request.method == 'POST'`.

    Как и при создании нового поста, сначала извлекаются данные из объекта `request.form`, затем, если заголовок имеет пустое значение, показывается всплывающее сообщение, в противном случае открывается подключение к базе данных. После этого обновляется таблица `posts` — указывается новый заголовок и содержание, где ID поста в базе данных соответствует ID, указанному в URL.

    В случае GET запроса используется шаблон `edit.html`, в который передается объект `post`. Он нужен для отображения существующих заголовка и содержания на странице редактирования.

    Сохраните и закройте файл.

1. Создайте и откройте файл-шаблон с именем `edit.html` в директории `templates`:

    ```bash
    vim templates/edit.html
    ```

1. Добавьте в файл следующий код:

    ```html
    {% extends 'base.html' %}

    {% block content %}
    <h1>{% block title %} Edit "{{ post['title'] }}" {% endblock %}</h1>

    <form method="post">
      <div class="mb-3">
        <label for="title" class="col-sm-2 col-form-label">Title</label>
        <input type="text" name="title" placeholder="Post title"
               class="form-control"
               value="{{ request.form['title'] or post['title'] }}">
      </div>

      <div class="mb-3">
        <label for="content" class="col-sm-2 col-form-label">Content</label>
        <textarea name="content" placeholder="Post content"
                  class="form-control" rows="3">{{ request.form['content'] or post['content'] }}</textarea>
      </div>
      <div class="mb-3">
        <button type="submit" class="btn btn-primary">Submit</button>
      </div>
    </form>
    <hr>
    {% endblock %}
    ```

    Код аналогичен коду в шаблоне `create.html` за исключением выражений `{{ request.form['title'] or post['title'] }}` и `{{ request.form['content'] or post['content'] }}`. Они отображают данные, сохраненные в запросе, если он существует. В противном случае отображаются данные из переменной `post`, которая была передана в шаблон, содержащий данные текущей базы данных.

    Сохраните и закройте файл.

1. Перейдите к URL `http://<публичный IP-адрес ВМ>:5000/1/edit` для редактирования первого поста. Вы увидите страницу **Edit "First Post"** с данными поста. Отредактируйте пост, отправьте форму и убедитесь, что пост был обновлен.

    Теперь добавьте ссылку, указывающую на страницу редактирования каждого поста.

1. Откройте шаблон `index.html`:

    ```bash
    vim templates/index.html
    ```

1. Добавьте ссылку для редактирования поста после даты создания:

    ```html
    ...
      {% for post in posts %}
        <a href="{{ url_for('post', post_id=post['id']) }}">
          <h2>{{ post['title'] }}</h2>
        </a>
        <span class="badge text-bg-primary">{{ post['created'] }}</span>
        <a href="{{ url_for('edit', id=post['id']) }}">
          <span class="badge text-bg-warning">Edit</span>
        </a>
        <hr>
      {% endfor %}
    ...
    ```

    В теге ссылки `<a>` добавляется ссылка **Edit**, которая указывает на функцию просмотра `edit()` и передает значение ID из `post['id']`.

    Сохраните и закройте файл

1. Перейдите на главную страницу приложения `http://<публичный IP-адрес ВМ>:5000` и убедитесь, что на ней появились ссылки **Edit**, с помощью которых можно изменить содержимое поста.

### Удаление поста {#delete-post}

Удаление будет происходить с помощью функции просмотра `delete()`, которая будет получать ID поста из URL, и маршрута `/ID/delete`, который принимает POST запросы, аналогично функции просмотра `edit()`.

1. Откройте файл `app.py`:

    ```bash
    vim app.py
    ```

1. Добавьте функцию просмотра `delete()`:

    ```python
    ...
    @app.route('/<int:id>/delete', methods=('POST',))
    def delete(id):
        post = get_post(id)
        conn = get_db_connection()
        conn.execute('DELETE FROM posts WHERE id = ?', (id,))
        conn.commit()
        conn.close()
        flash('"{}" was successfully deleted!'.format(post['title']))
        return redirect(url_for('index'))
    ```

    Эта функция принимает только POST запросы. Это значит, что навигация по маршруту `/ID/delete` в браузере вернет ошибку, так как веб-браузеры по умолчанию настроены на GET запросы.

    Но можно получить доступ к этому маршруту через форму, которая отправляет POST запрос, передающий ID поста, необходимого для удаления. Используя значение ID, функция получит пост из базы данных с помощью функции `get_post()`.

    Затем открывается подключение к базе данных и выполняется команда SQL `DELETE FROM`. Изменения фиксируются и подключение закрывается. Для пользователя отображается сообщение, что пост был успешно удален. В конце происходит перенаправление на главную страницу приложения.

    Данная функция не использует шаблон, так как кнопка **Delete** просто добавляется на страницу редактирования.

    Сохраните и закройте файл.

1. Откройте шаблон `edit.html`:

    ```bash
    vim templates/edit.html
    ```

1. Добавьте тег `<form>` после тега `<hr>` и непосредственно перед строкой `{% endblock %}`:

    ```html
    ...
    <hr>
    <form action="{{ url_for('delete', id=post['id']) }}" method="POST">
      <input type="submit" value="Delete Post"
             class="btn btn-danger btn-sm"
             onclick="return confirm('Are you sure you want to delete this post?')">
    </form>
    {% endblock %}
    ```

    Здесь используется [метод `confirm()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/confirm) для отображения сообщения подтверждения перед отправкой запроса.

    Сохраните и закройте файл.

1. Перейдите на главную страницу приложения  `http://<публичный IP-адрес ВМ>:5000` и убедитесь, что при редактировании поста появилась ссылка **Edit**, удаляющая пост.

В итоге исходный код вашего проекта будет выглядеть, как [код на этой странице](https://github.com/).

## Подведите итоги {#results}

В руководстве были представлены важные концепции фреймворка Flask Python. Вы научились создавать небольшое веб-приложение, запускать его на сервере разработки и предоставлять пользователю возможность вносить свои данные через параметры URL и веб-формы. Вы также использовали [механизм шаблонов Jinja](http://jinja.palletsprojects.com/) для повторного использования файлов HTML и применения логики в них. В итоге вы получили полностью функционирующий веб-блог, взаимодействующий с [базой данных SQLite](https://sqlite.org/) для создания, отображения, редактирования и удаления постов с использованием языка Python и запросов SQL.

Вы можете продолжить разработку приложения, добавив аутентификацию, позволяющую только зарегистрированным пользователям создавать и менять посты в блоге. А также можете добавить комментарии и теги для каждого поста блога или разрешить загрузку файлов, чтобы пользователи могли прикреплять изображения к посту. Подробнее см. в [документации Flask](https://flask.palletsprojects.com/en/latest/).

У Flask есть множество [расширений](http://flask.palletsprojects.com/en/latest/extensions/), созданных сообществом. Ниже представлен список расширений, которые вы можете использовать для упрощения процесса разработки:

* [Flask-Login](https://flask-login.readthedocs.io/en/latest/) — управляет сеансом пользователя и обрабатывает вход и выход, а также запоминает вошедших в систему пользователей.
* [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/latest/) — упрощает использование Flask благодаря поддержки [SQLAlchemy](https://www.sqlalchemy.org/). Это инструментарий Python SQL и Object Relational Mapper для взаимодействия с базами данных.
* [Flask-Mail](https://pythonhosted.org/Flask-Mail/) — помогает отправлять сообщения электронной почты в приложении Flask.

## Удалите созданные ресурсы {#clear-out}

Чтобы перестать платить за развернутый сервер, [удалите ВМ](../../compute/operations/vm-control/vm-delete.md).

Если вы зарезервировали для ВМ статический публичный IP-адрес, [удалите его](../../vpc/operations/address-delete.md).
