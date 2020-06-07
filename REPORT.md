## Laboratory work VIII

<a href="https://yandex.ru/efir/?stream_id=v0mnBi_R2Ldw"><img src="https://raw.githubusercontent.com/tp-labs/lab08/master/preview.png" width="640"/></a>

Данная лабораторная работа посвещена изучению систем автоматизации развёртывания и управления приложениями на примере **Docker**

```sh
$ open https://docs.docker.com/get-started/
```

## Tasks

- [x] 1. Создать публичный репозиторий с названием **lab08** на сервисе **GitHub**
- [x] 2. Ознакомиться со ссылками учебного материала
- [x] 3. Выполнить инструкцию учебного материала
- [x] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

Создание переменных среды и установка их значений, а также связывание команд с их "новыми" названиями.
```sh
$ export GITHUB_USERNAME=nishaque
```
Начало работы в каталоге `workspace`
```sh
# Переход в  рабочую директорию
$ cd ${GITHUB_USERNAME}/workspace
$ pushd . # Сохранение текущего каталога в стек
# Активация Node.js и rvm
$ source scripts/activate
```
Настройка git-репозитория **lab07** для работы
```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab07 lab08
$ cd lab08
# Инциализация подмодуля
$ git submodule update --init

Submodule 'tools/polly' (https://github.com/ruslo/polly) registered for path 'tools/polly'
Cloning into '/Users/nishaque/nishaque/workspace/lab08/tools/polly'...
Submodule path 'tools/polly': checked out '0b3806e193b668fbb9b70c9aa70735b29396323d'


$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab08
```
Создаем `Dockerfile`. В самом начале указываем базовый образ (В нашем случае это Ubuntu 18.04), который определяет рабочую среду,пакеты и утилиты необходимые для запуска приложения в нашем контейнере. В Dockerfile содержатся инструкции по созданию образа.
```sh
$ cat > Dockerfile <<EOF
FROM ubuntu:18.04
EOF
```
Выполняем обновление списка пакетов **APT** в базовом образе - утилиты для управления пакетами. Затем с его помощью устанавливаем компиляторы и `CMake`.
```sh
$ cat >> Dockerfile <<EOF

RUN apt update
RUN apt install -yy gcc g++ cmake
EOF
```
Инструкция `COPY` сообщает Docker о том, что нужно взять файлы и папки из локального контекста сборки и добавить их в текущую рабочую директорию образа. Если целевая директория не существует, эта инструкция её создаст.
Инструкция `WORKDIR` позволяет изменить рабочую директорию контейнера.
```sh
$ cat >> Dockerfile <<EOF

COPY . print/
WORKDIR print
EOF
```
Инструкция `RUN` позволяет создать слой во время сборки образа. После её выполнения в образ добавляется новый слой, его состояние фиксируется. Инструкция RUN часто используется для установки в образы дополнительных пакетов.
```sh
# Выполняем сборку нашего проекта
$ cat >> Dockerfile <<EOF
RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install
EOF
```
Инструкция `ENV` позволяет задавать постоянные переменные среды, которые будут доступны в контейнере во время его выполнения.
```sh
# Задаем переменную LOG_PATH
$ cat >> Dockerfile <<EOF
ENV LOG_PATH /home/logs/log.txt
EOF
```
Инструкция `VOLUME` позволяет указать место, которое контейнер будет использовать для постоянного хранения файлов и для работы с такими файлами.
```sh
# Указываем в какой директории будут храниться файлы, которые останутся после работы с контейнером
$ cat >> Dockerfile <<EOF
VOLUME /home/logs
EOF
```
Переход в созданную в процессе сборки директорию `_install/bin`
```sh
$ cat >> Dockerfile <<EOF

WORKDIR _install/bin
EOF
```
Инструкция `ENTRYPOINT` позволяет задавать команду с аргументами, которая должна выполняться при запуске контейнера. Она похожа на команду `CMD`, но параметры, задаваемые в `ENTRYPOINT`, не перезаписываются в том случае, если контейнер запускают с параметрами командной строки.
```sh
# В нащем случае команда ENTRYPOINT опреедляет точку входа в приложение
$ cat >> Dockerfile <<EOF
ENTRYPOINT ./demo
EOF
```
Сборка образа с тегом `logger` и путем к `Dockerfile`
```sh
$ sudo docker build -t logger .
...
Successfully built ba3db56543df
Successfully tagged logger:latest
```
Команда, которая выводит всю информацию об образах
```sh
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
logger                    latest              3428bfcc1442        3 minutes ago       321MB
ubuntu                    18.04               c3c304cb4f22        6 weeks ago         64.2MB
rusdevops/bootstrap-cpp   latest              2596a89150d1        18 months ago       976MB
```
Запуск контейнера в интерактивном режиме. Флаг `-v` указывает куда сохранять получаемые в контейнере файлы.
```sh
$ mkdir logs
$ docker run -it -v "$(pwd)/logs/:/home/logs/" logger /bin/bash

text1
text2
text3
<C-D>
```
Вывод подробной информации о контейнере
```sh
$ docker inspect logger
```
Проверяем, что файлы, полученные во время работы в контейнере были сохранены
```sh
$ cat logs/log.txt
text1
text2
text3
```
Замена lab07 на lab08
```sh
$ gsed -i  's/lab07/lab08/g' README.md
```
Изменение `.travis.yml` для сборки в контейнере.
```sh
$ vim .travis.yml
/lang<CR>o
services:
- docker<ESC>
jVGdo
script:
- docker build -t logger .<ESC>
:wq
```
Добавление **Docker** в репозиторий.
```sh
$ git add Dockerfile
$ git add .travis.yml
$ git commit -m"adding Dockerfile"
$ git push origin master
```
Активация непрерывной интеграции с **Travis CI**.
```sh
# Флаг --com  для работы с travis-ci.com
$ travis login --auto --com
Successfully logged in as nishaque!
$ travis enable --com
nishaque/lab08: enabled :)
```

## Report

```sh
Создание отчета по ЛР № 8
$ popd
$ export LAB_NUMBER=08
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ atom REPORT.md
$ gist REPORT.md
```

## Links

- [Book](https://www.dockerbook.com)
- [Instructions](https://docs.docker.com/engine/reference/builder/)

```
Copyright (c) 2015-2019 The ISC Authors
```
