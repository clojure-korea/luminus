## 방명록 애플리케이션

이 튜토리얼은 루미너스로 간단한 방명록 애플리케이션을 만드는 방법을 알려줍니다.
방명록은 사용자가 메시지를 남기고 다른 사람들이 메시지 목록을 볼 수 있는 애플리케이션입니다.
이 애플리케이션을 만들어 보면서 기본적인 HTML 템플릿, 데이터베이스 접근, 프로젝트 구조를 설명하려고 합니다.

아직 클로저 에디터가 없다면 [Light Table](http://www.lighttable.com/)로 튜토리얼을 따라서 만들어 보는 것을 추천합니다.

### 도커 이미지 사용하기

도커를 사용한다면 다음과 같이 실행합니다:

1. `docker pull danboykis/luminus-guestbook`
2. `docker run -p 3000:3000 -p 7000:7000 -it danboykis/luminus-guestbook`

도커 이미지를 만들어서 빌드하려면 [여기](https://github.com/luminus-framework/luminus-docker)를 참조하세요

### JDK 설치

클로저는 JVM에서 동작하기 때문에 JDK가 설치되어 있어야 합니다. JDK가 없다면 [여기](http://www.azul.com/downloads/zulu/)
에서 OpenJDK를 다운로드 받아서 설치하는 것을 추천합니다. Luminus는 기본적으로 JDK 8에 동작합니다.

### Leiningen 설치

Luminus를 사용하려면 [Leiningen](http://leiningen.org/)이 설치되어 있어야 합니다.
Leiningen은 다음과 같이 설치합니다.

1. 스크립트 파일 다운로드
3. 스크립트를 실행 가능 하도록 권한 설정 (예: chmod +x lein)
2. 어디서나 실행 가능한 $PATH에 위치 시키기 (예: ~/bin)
4. `lein` 명령어를 실행하고 설치가 되기를 기다린다.

```
wget https://raw.github.com/technomancy/leiningen/stable/bin/lein
chmod +x lein
mv lein ~/bin
lein
```

### 새로운 애플리케이션 만들기

Leiningen을 설치하고 터미널에 다음과 같이 입력하면 애플리케이션을 생성할 수 있습니다:

```
lein new luminus guestbook +h2
cd guestbook
```

위 명령어는 [H2 임베디드 데이터베이스](http://www.h2database.com/html/main.html) 엔진을 지원하는
탬플릿 프로젝트를 만드는 예제 입니다.

### Luminus 애플리케이션 구조

생성된 애플리케이션은 아래와 같은 구조 입니다:

```
guestbook
├── Capstanfile
├── Dockerfile
├── Procfile
├── README.md
├── env
│   ├── dev
│   │   ├── clj
│   │   │   ├── guestbook
│   │   │   │   ├── dev_middleware.clj
│   │   │   │   └── env.clj
│   │   │   └── user.clj
│   │   └── resources
│   │       ├── config.edn
│   │       └── logback.xml
│   ├── prod
│   │   ├── clj
│   │   │   └── guestbook
│   │   │       └── env.clj
│   │   └── resources
│   │       ├── config.edn
│   │       └── logback.xml
│   └── test
│       └── resources
│           └── config.edn
├── profiles.clj
├── project.clj
├── resources
│   ├── docs
│   │   └── docs.md
│   ├── migrations
│   │   ├── 20160811175305-add-users-table.down.sql
│   │   └── 20160811175305-add-users-table.up.sql
│   ├── public
│   │   ├── css
│   │   │   └── screen.css
│   │   ├── favicon.ico
│   │   ├── img
│   │   └── js
│   ├── sql
│   │   └── queries.sql
│   └── templates
│       ├── about.html
│       ├── base.html
│       ├── error.html
│       └── home.html
├── src
│   └── clj
│       └── guestbook
│           ├── config.clj
│           ├── core.clj
│           ├── db
│           │   └── core.clj
│           ├── handler.clj
│           ├── layout.clj
│           ├── middleware.clj
│           └── routes
│               └── home.clj
└── test
    └── clj
        └── guestbook
            └── test
                ├── db
                │   └── core.clj
                └── handler.clj
```

최상위 폴더에 있는 파일 부터 살펴봅시다:

* `Procfile` - Heroku 배포에 사용
* `README.md` - 일반적인 애플리케이션 문서
* `project.clj` - Leiningen 파일로 프로젝트 설정이나 의존성을 관리
* `profiles.clj` - 리파지토리에 포함하지 않는 로컬 설정을 하기 위해 사용
* `.gitignore` - 빌드 생성 파일과 같은 Git에 포함하지 않는 파일 목록

### 소스 디렉토리

모든 소스 코드는 `src/clj` 폴더 아래 옵니다. 우리가 만드는 방명록 애플리케이션은 guestbook가 프로젝트의
최상위 네임스페이스입니다. 이 아래 생성된 다른 네임스페이스에 대해 살펴 봅시다.

#### guestbook

* `core.clj` - 서버 시작과 종료 로직을 가지고 있는 애플리케이션의 엔트리 포인트
* `handler.clj` - 애플리케이션의 기본 라우트를 정의
* `layout.clj` - 페이지에 내용을 렌더링 하기 위한 레이아웃 헬퍼 네임스페이스
* `middleware.clj` - 애플리케이션에 필요한 미들웨어를 가지고 있는 네임스페이스

#### guestbook.db

`db` 네임스페이스는 애플리케이션의 영속 레이어를 다루고 모델을 정의 합니다.

* `core.clj` - 데이터베이스를 조작하는 함수들이 위치

#### guestbook.routes

The `routes` namespace is where the routes and controllers for our home and about pages are located. When you add more routes,
such as authentication, or specific workflows you should create namespaces for them here.

* `home.clj` - a namespace that defines the home and about pages of the application

### The Env Directory

Environment specific code and resources are located under the `env/dev`, `env/test`, and the `env/prod` paths.
The `dev` configuration will be used during development, `test` during testing,
while the `prod` configuration will be used when the application is packaged for production.

#### `dev/clj`

* `user.clj` - a utility namespace for any code you wish to run during REPL development
* `guestbook/env.clj` - contains the development configuration defaults
* `guestbook/dev_middleware.clj` - contains middleware used for development that should not be compiled in production

#### `dev/resources`

* `config.edn` - default environment variables for the development
* `logback.xml` file used to configure the development logging profile

#### `test/resources`

* `config.edn` - default environment variables for testing

#### `prod/clj`

* `guestbook/env.clj` namespace with the production configuration

#### `prod/resources`

* `config.edn` - default environment variables that will be packaged with the application
* `logback.xml` - default production logging configuration

### The Test Directory

Here is where we put tests for our application, a couple of sample tests have already been defined for us.

### The Resources Directory

This is where we put all the static resources for our application. Content in the `public` directory under `resources` will be served to the clients by the server. We can see that some CSS resources have already been created for us.

#### HTML templates

The templates directory is reserved for the [Selmer](https://github.com/yogthos/Selmer) templates
that represent the application pages.

* `about.html` - about page
* `base.html` - base layout for the site
* `home.html` - home page
* `error.html` - error page template

#### SQL Queries

The SQL queries are found in the `resources/sql` folder.

* `queries.sql` - defines the SQL queries and their associated function names

#### The Migrations Directory

Luminus uses [Migratus](https://github.com/yogthos/migratus) for migrations. Migrations are managed using up and down SQL files.
The files are conventionally versioned using the date and will be applied in order of their creation.

* `20150718103127-add-users-table.up.sql` - migrations file to create the tables
* `20150718103127-add-users-table.down.sql` - migrations file to drop the tables


### The Project File

As was noted above, all the dependencies are managed via updating the `project.clj` file.
The project file of the application we've created is found in its root folder and should look as follows:

```clojure
(defproject guestbook "0.1.0-SNAPSHOT"

  :description "FIXME: write description"
  :url "http://example.com/FIXME"

  :dependencies [[org.clojure/clojure "1.8.0"]
                 [selmer "1.0.7"]
                 [markdown-clj "0.9.89"]
                 [ring-middleware-format "0.7.0"]
                 [metosin/ring-http-response "0.8.0"]
                 [bouncer "1.0.0"]
                 [org.webjars/bootstrap "4.0.0-alpha.3"]
                 [org.webjars/font-awesome "4.6.3"]
                 [org.webjars.bower/tether "1.3.3"]
                 [org.webjars/jquery "3.0.0"]
                 [org.clojure/tools.logging "0.3.1"]
                 [compojure "1.5.1"]
                 [ring-webjars "0.1.1"]
                 [ring/ring-defaults "0.2.1"]
                 [mount "0.1.10"]
                 [cprop "0.1.8"]
                 [org.clojure/tools.cli "0.3.5"]
                 [luminus-nrepl "0.1.4"]
                 [luminus-migrations "0.2.6"]
                 [conman "0.6.0"]
                 [com.h2database/h2 "1.4.192"]
                 [org.webjars/webjars-locator-jboss-vfs "0.1.0"]
                 [luminus-immutant "0.2.2"]]

  :min-lein-version "2.0.0"

  :jvm-opts ["-server" "-Dconf=.lein-env"]
  :source-paths ["src/clj"]
  :resource-paths ["resources"]
  :target-path "target/%s/"
  :main guestbook.core
  :migratus {:store :database :db ~(get (System/getenv) "DATABASE_URL")}

  :plugins [[lein-cprop "1.0.1"]
            [migratus-lein "0.4.1"]
            [lein-immutant "2.1.0"]]

  :profiles
  {:uberjar {:omit-source true
             :aot :all
             :uberjar-name "guestbook.jar"
             :source-paths ["env/prod/clj"]
             :resource-paths ["env/prod/resources"]}

   :dev           [:project/dev :profiles/dev]
   :test          [:project/test :profiles/test]

   :project/dev  {:dependencies [[prone "1.1.1"]
                                 [ring/ring-mock "0.3.0"]
                                 [ring/ring-devel "1.5.0"]
                                 [pjstadig/humane-test-output "0.8.1"]]
                  :plugins      [[com.jakemccrary/lein-test-refresh "0.14.0"]]

                  :source-paths ["env/dev/clj" "test/clj"]
                  :resource-paths ["env/dev/resources"]
                  :repl-options {:init-ns user}
                  :injections [(require 'pjstadig.humane-test-output)
                               (pjstadig.humane-test-output/activate!)]}
   :project/test {:resource-paths ["env/dev/resources" "env/test/resources"]}
   :profiles/dev {}
   :profiles/test {}})
```

As you can see the `project.clj` file is simply a Clojure list containing key/value pairs describing different aspects of the application.

The most common task is adding new libraries to the project. These libraries are specified using the `:dependencies` vector.
In order to use a new library in our project we simply have to add its dependency here.

The items in the `:plugins` vector can be used to provide additional functionality such as reading environment variables via `lein-cprop` plugin.

The `:profiles` contain a map of different project configurations that are used to initialize it for either development or production builds.

Note that the project sets up composite profiles for `:dev` and `:test`. These profiles contain the variables from `:project/dev` and `:project/test` profiles,
as well as from `:profiles/dev` and `:profiles/test` found in the `profiles.clj`. The latter should contain local environment variables that are not meant to be
checked into the shared code repository.

Please refer to the [official Leiningen documentation](http://leiningen.org/#docs) for further details on structuring the `project.clj` build file.


### Creating the Database

First, we will create a model for our application, to do that we'll open up the `<date>-add-users-table.up.sql`
file located under the `migrations` folder. The file has the following contents:

```sql
CREATE TABLE users
(id VARCHAR(20) PRIMARY KEY,
 first_name VARCHAR(30),
 last_name VARCHAR(30),
 email VARCHAR(30),
 admin BOOLEAN,
 last_login TIME,
 is_active BOOLEAN,
 pass VARCHAR(300));
```

We'll replace the `users` table with one that's more appropriate for our application:

```sql
CREATE TABLE guestbook
(id INTEGER PRIMARY KEY AUTO_INCREMENT,
 name VARCHAR(30),
 message VARCHAR(200),
 timestamp TIMESTAMP);
```

The guestbook table will store all the fields describing the message, such as the name of the
commenter, the content of the message and a timestamp.
Next, let's replace the contents of the `<date>-add-users-table.down.sql` file accordingly:

```sql
DROP TABLE guestbook;
```

We can now run the migrations using the following command from the root of our project:

```
lein run migrate
```

If everything went well we should now have our database initialized.

### Accessing The Database

Next, we'll take a look at the `src/clj/guestbook/db/core.clj` file.
Here, we can see that we already have the definition for our database connection.

```clojure
(ns guestbook.db.core
  (:require
    [conman.core :as conman]
    [mount.core :refer [defstate]]
    [guestbook.config :refer [env]]))

(defstate ^:dynamic *db*
           :start (conman/connect! {:jdbc-url (env :database-url)})
           :stop (conman/disconnect! *db*))

(conman/bind-connection *db* "sql/queries.sql")
```

The database connection is read from the environment map at runtime. By default, the `:database-url` key points to a
a string with the connection URL for the database.
 This variable is populated from the `profiles.clj` file during development and has to be set as an environment variable for production, e.g:

```
export DATABASE_URL="jdbc:h2:./guestbook.db"
```

Since we're using the embedded H2 database, the data is stored in a file specified in the URL that's found in the path relative to where the project is run.

The functions that map to database queries are generated when `bind-connection` is called. As we can see it references the `sql/queries.sql` file.
This location is found under the `resources` folder. Let's open up this file and take a look inside.

```sql
-- :name create-user! :! :n
-- :doc creates a new user record
INSERT INTO users
(id, first_name, last_name, email, pass)
VALUES (:id, :first_name, :last_name, :email, :pass)

-- :name update-user! :! :n
-- :doc update an existing user record
UPDATE users
SET first_name = :first_name, last_name = :last_name, email = :email
WHERE id = :id

-- :name get-user :? :1
-- :doc retrieve a user given the id.
SELECT * FROM users
WHERE id = :id
```

As we can see each function is defined using the comment that starts with `-- :name` followed by the name of the function.
The next comment provides the doc string for the function and finally we have the body that's plain SQL. The parameters are
denoted using `:` notation. Let's replace the existing queries with some of our own:


```sql
-- :name save-message! :! :n
-- :doc creates a new message
INSERT INTO guestbook
(name, message, timestamp)
VALUES (:name, :message, :timestamp)

-- :name get-messages :? :*
-- :doc selects all available messages
SELECT * FROM guestbook
```

Now that our model is all setup, let's start up the application.

### Running the Application

We can run our application in development mode as follows:

```
>lein run
[2016-02-28 15:05:34,970][DEBUG][org.jboss.logging] Logging Provider: org.jboss.logging.Log4jLoggerProvider
[2016-02-28 15:05:36,067][INFO][com.zaxxer.hikari.HikariDataSource] HikariPool-0 - is starting.
[2016-02-28 15:05:36,252][INFO][luminus.http-server] starting HTTP server on port 3000
[2016-02-28 15:05:36,294][INFO][org.xnio] XNIO version 3.4.0.Beta1
[2016-02-28 15:05:36,344][INFO][org.xnio.nio] XNIO NIO Implementation Version 3.4.0.Beta1
[2016-02-28 15:05:36,406][INFO][org.projectodd.wunderboss.web.Web] Registered web context /
[2016-02-28 15:05:36,407][INFO][luminus.repl-server] starting nREPL server on port 7000
[2016-02-28 15:05:36,422][INFO][guestbook.core] #'guestbook.config/env started
[2016-02-28 15:05:36,422][INFO][guestbook.core] #'guestbook.db.core/*db* started
[2016-02-28 15:05:36,422][INFO][guestbook.core] #'guestbook.core/http-server started
[2016-02-28 15:05:36,423][INFO][guestbook.core] #'guestbook.core/repl-server started
[2016-02-28 15:05:36,423][INFO][guestbook.env]
-=[guestbook started successfully using the development profile]=-
```

Once server starts, you should be able to navigate to [http://localhost:3000](http://localhost:3000) and see
the app running. The server can be started on an alternate port by either passing it as a parameter as seen below,
or setting the `PORT` environment variable.

```
lein run -p 8000
```

Note that the page is prompting us to run the migrations in order to initialize the database. However, we've already done that earlier, so we won't need to do that again.

### Creating Pages and Handling Form Input

Our routes are defined in the `guestbook.routes.home` namespace. Let's open it up and add the logic for
rendering the messages from the database. We'll first need to add a reference to our `db` namespace along with
references for [Bouncer](https://github.com/leonardoborges/bouncer) validators and [ring.util.response](http://ring-clojure.github.io/ring/ring.util.response.html)

```clojure
(ns guestbook.routes.home
  (:require
    ...
    [guestbook.db.core :as db]
    [bouncer.core :as b]
    [bouncer.validators :as v]
    [ring.util.response :refer [redirect]]))
```

Next, we'll create a function to validate the form parameters.

```clojure
(defn validate-message [params]
  (first
    (b/validate
      params
      :name v/required
      :message [v/required [v/min-count 10]])))
```

The function uses the Bouncer `validate` function to check that the `:name` and the `:message` keys
conform to the rules we specified. Specifically, the name is required and the message must contain at least
10 characters. Bouncer uses vector syntax to pass multiple rules to a validator as is the case with the message validator.

When a validator takes additional parameters as is the case with `min-count` the vector syntax is used as well. The value will be passed in implicitly as the first parameter to the validator.

We'll now add a function to validate and save messages:

```clojure
(defn save-message! [{:keys [params]}]
  (if-let [errors (validate-message params)]
    (-> (response/found "/")
        (assoc :flash (assoc params :errors errors)))
    (do
      (db/save-message!
       (assoc params :timestamp (java.util.Date.)))
      (response/found "/"))))
```

The function will grab the `:params` key from the request that contains the form parameters. When the `validate-message` functions returns errors we'll redirect back to `/`, we'll associate a `:flash` key with the response where we'll put the supplied parameters along with the errors. Otherwise, we'll save the message in our database and redirect.

We can now change the `home-page` handler function to look as follows:

```clojure
(defn home-page [{:keys [flash]}]
  (layout/render
   "home.html"
   (merge {:messages (db/get-messages)}
          (select-keys flash [:name :message :errors]))))
```

The function renders the home page template and passes it the currently stored messages along with any parameters from the `:flash` session, such as validation errors.

Recall that the database accessor functions were automatically generated for us by the `(conman/bind-connection *db* "sql/queries.sql")` statement ran in the `guestbook.db.core` namespace. The names of these functions are inferred from the `-- :name` comments in the SQL templates found in the `resources/sq/queries.sql` file.

Our routes will now have to pass the request to both the `home-page` and the `save-message!` handlers:

```clojure
(defroutes home-routes
  (GET "/" request (home-page request))
  (POST "/" request (save-message! request))
  (GET "/about" [] (about-page)))
```

Don't forget to refer `POST` from `compojure.core`

```
(ns guestbook.routes.home
  (:require ...
            [compojure.core :refer [defroutes GET POST]]
            ...))
```

Now that we have our controllers setup, let's open the `home.html` template located under the `resources/templates` directory. Currently, it simply renders the contents of the `content` variable inside the content block:

```xml
{% extends "base.html" %}
{% block content %}
  <div class="jumbotron">
    <h1>Welcome to guestbook</h1>
    <p>Time to start building your site!</p>
    <p><a class="btn btn-primary btn-lg" href="http://luminusweb.net">Learn more &raquo;</a></p>
  </div>

  <div class="row">
    <div class="span12">
    {{docs|markdown}}
    </div>
  </div>
{% endblock %}
```

We'll update our `content` block to iterate over the messages and print each one in a list:

```xml
{% extends "base.html" %}
{% block content %}
  <div class="jumbotron">
    <h1>Welcome to guestbook</h1>
    <p>Time to start building your site!</p>
    <p><a class="btn btn-primary btn-lg" href="http://luminusweb.net">Learn more &raquo;</a></p>
  </div>

  <div class="row">
    <div class="span12">
        <ul class="messages">
            {% for item in messages %}
            <li>
                <time>{{item.timestamp|date:"yyyy-MM-dd HH:mm"}}</time>
                <p>{{item.message}}</p>
                <p> - {{item.name}}</p>
            </li>
            {% endfor %}
        </ul>
    </div>
</div>
{% endblock %}
```

As you can see above, we use a `for` iterator to walk the messages.
Since each message is a map with the message, name, and timestamp keys, we can access them by name.
Also, notice the use of the `date` filter to format the timestamps into a human readable form.

Finally, we'll create a form to allow users to submit their messages. We'll populate the name and message values if they're supplied and render any errors associated with them. Note that the forms also uses the `csrf-field` tag that's required for cross-site request forgery protection.

```xml
<div class="row">
    <div class="span12">
        <form method="POST" action="/">
                {% csrf-field %}
                <p>
                    Name:
                    <input class="form-control"
                           type="text"
                           name="name"
                           value="{{name}}" />
                </p>
                {% if errors.name %}
                <div class="alert alert-danger">{{errors.name|join}}</div>
                {% endif %}
                <p>
                    Message:
                <textarea class="form-control"
                          rows="4"
                          cols="50"
                          name="message">{{message}}</textarea>
                </p>
                {% if errors.message %}
                <div class="alert alert-danger">{{errors.message|join}}</div>
                {% endif %}
                <input type="submit" class="btn btn-primary" value="comment" />
        </form>
    </div>
</div>
```

Our final `home.html` template should look as follows:

```xml
{% extends "base.html" %}
{% block content %}
<div class="row">
    <div class="span12">
        <ul class="messages">
            {% for item in messages %}
            <li>
                <time>{{item.timestamp|date:"yyyy-MM-dd HH:mm"}}</time>
                <p>{{item.message}}</p>
                <p> - {{item.name}}</p>
            </li>
            {% endfor %}
        </ul>
    </div>
</div>
<div class="row">
    <div class="span12">
        <form method="POST" action="/">
                {% csrf-field %}
                <p>
                    Name:
                    <input class="form-control"
                           type="text"
                           name="name"
                           value="{{name}}" />
                </p>
                {% if errors.name %}
                <div class="alert alert-danger">{{errors.name|join}}</div>
                {% endif %}
                <p>
                    Message:
                <textarea class="form-control"
                          rows="4"
                          cols="50"
                          name="message">{{message}}</textarea>
                </p>
                {% if errors.message %}
                <div class="alert alert-danger">{{errors.message|join}}</div>
                {% endif %}
                <input type="submit" class="btn btn-primary" value="comment" />
        </form>
    </div>
</div>
{% endblock %}
```

Finally, we can update the `screen.css` file located in the `resources/public/css` folder to format our form nicer:

```
html,
body {
	font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
  height: 100%;
  line-height: 1.4em;
	background: #eaeaea;
	width: 520px;
	margin: 0 auto;
}
.navbar {
  margin-bottom: 10px;
}
.navbar-brand {
  float: none;
}
.navbar-nav .nav-item {
  float: none;
}
.navbar-divider,
.navbar-nav .nav-item+.nav-item,
.navbar-nav .nav-link + .nav-link {
  margin-left: 0;
}
@media (min-width: 34em) {
  .navbar-brand {
    float: left;
  }
  .navbar-nav .nav-item {
    float: left;
  }
  .navbar-divider,
  .navbar-nav .nav-item+.nav-item,
  .navbar-nav .nav-link + .nav-link {
    margin-left: 1rem;
  }
}

.messages {
  background: white;
  width: 520px;
}
ul {
	list-style: none;
}

ul.messages li {
	position: relative;
	font-size: 16px;
	padding: 5px;
	border-bottom: 1px dotted #ccc;
}

li:last-child {
	border-bottom: none;
}

li time {
	font-size: 12px;
	padding-bottom: 20px;
}

form, .error {
	padding: 30px;
	margin-bottom: 50px;
	position: relative;
  background: white;
}
```

When we reload the page in the browser we should be greeted by the guestbook page.
We can test that everything is working as expected by adding a comment in our comment form.

## Adding some tests

Now that we have our application working we can add some tests for it.
Let's open up the `test/clj/guestbook/test/db/core.clj` namespace and update it as follows:

```clojure
(ns guestbook.test.db.core
  (:require [guestbook.db.core :refer [*db*] :as db]
            [luminus-migrations.core :as migrations]
            [clojure.test :refer :all]
            [clojure.java.jdbc :as jdbc]
            [guestbook.config :refer [env]]
            [mount.core :as mount]))

(use-fixtures
  :once
  (fn [f]
    (mount/start
      #'guestbook.config/env
      #'guestbook.db.core/*db*)
    (migrations/migrate ["migrate"] (select-keys env [:database-url]))
    (f)))

(deftest test-users
  (jdbc/with-db-transaction [t-conn *db*]
    (jdbc/db-set-rollback-only! t-conn)
    (let [timestamp (java.util.Date.)]
      (is (= 1 (db/save-message!
                t-conn
                {:name "Bob"
                 :message "Hello, World"
                 :timestamp timestamp}
                {:connection t-conn})))
      (is (=
           {:name "Bob"
            :message "Hello, World"
            :timestamp timestamp}
           (-> (db/get-messages t-conn {})
               (first)
               (select-keys [:name :message :timestamp])))))))
```

We can now run `lein test` in the terminal to see that our database interaction works as expected.

Luminus comes with [lein-test-refresh](https://github.com/jakemcc/lein-test-refresh) enabled by default. This plugin allows running tests continuously
whenever a change in a namespace is detected. We can start a test runner in a new terminal using the `lein test-refresh` command.

## Packaging the application

The application can be packaged for standalone deployment by running the following command:

```
lein uberjar
```

This will create a runnable jar that can be run as seen below:

```
export DATABASE_URL="jdbc:h2:./guestbook_dev.db"
java -jar target/uberjar/guestbook.jar
```

Note that we have to supply the `DATABASE_URL` environment variable when running as a jar, as
it's not packaged with the application.

***

Complete source listing for the tutorial is available [here](https://github.com/luminus-framework/examples/tree/master/guestbook).
