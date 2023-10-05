# LDE = Local Docker Environment (application)

Это шаблонный проект для создания типовой LDE.

## Требования к локальной станции на примере OS Linux

- git (>= 2.33)
- make (>= 4.3)
- docker (>= 20.10.6)
- docker compose (>= 2.0)

## Дополнительные требования к локальной станции MacOS

- добавить в настройках docker папку пользователя где находятся ключи в разрешения работы с файлами (Preferences > Resources > File Sharing)

## Предварительные шаги

## 1. Перед установкой

Если вы только что поставили docker, не забудьте выполнить post-install-инструкции [здесь она есть](https://docs.docker.com/engine/install/linux-postinstall/).

```shell
sudo groupadd docker
sudo usermod -aG docker $USER 
```

Если вы не выполните данные действия, то увидите ошибку
`Got permission denied while trying to connect ti the Docker daemon socket ...`

> 🚨🚨🚨 Все действия консольных команд предполагают что вы находитесь в директории проекта lde.

## 2. env-файл окружения проекта LDE

Перед началом работы необходимо создать и проверить файл .env,

Если его нет, то можно скопировать из шаблона

```shell
cp .env.example .env
```

Если у Вас уже есть файл .env с внесенными изменениями, то будьте внимательны, не перезатрите его копированием шаблона.

> 🚨🚨🚨  важно внимательно привести его в соответствие с требованиями и локальной станцией - пути до директории кода, ssh пользователя и т.д., сформировать его можно на основе {.env.example}, файл в корне проекта. Переменные USER и UID обязательные они должны соответствовать данным на хост системе. UID по умолчанию 1000 если ваш UID отличается, пожалуйста внесите правки в .env

## 3. Остановить балансир от других окружений

### 3.1 Если у вас запущен traefik от другого проекта (например, LDE другого проекта)

Выполните его остановку так как traefik для приложения содержит алиасы необходимые для работы микросервисов. 
Команды, чтобы найти traefik от другого проекта примерно следующие(названия могут быть _ingress_, _traefik_).

```shell
docker ps | grep traefik 
```

```shell
docker stop container_id
```

### 3.2 Остановка локального web-сервера

Так же порты на вашей локальной машине могут занимать другие приложения, например, другие веб-серверы. Их необходимо остановить.
Например, для Ubuntuостановка nginx выполняется командой

```shell
sudo service nginx stop
```

А apache2

```shell
sudo systemctl stop apache2
```

## 4. Залить сидирующие данные в БД

Для этого нужно добавить SQL-файлы с данными в папку ./init/mysql.
В качестве примера в папке лежит init.sql с закомментированными данными.
Все SQL-файлы, которые вы разместите, будут залиты при первой установке LDE.
Чтобы залить их снова с нуля, нужно удалить соответствующий docker volume.
 
## Запуск проекта (short way)

```shell
make lde
```

> 🚨🚨🚨 после ряда процедур должны быть запущены все службы и контейнеры проекта - проект должен быть доступен по ожидаемым адресам в вашем браузере.

### Дополнительные требования к локальной станции Windows

Необходимо руками дописать домены для локальной разработки в файл:

```shell
C:\Windows\System32\drivers\etc\hosts
```

Актуальный список хостов выдаётся в процессе установки LDE, либо можно получить командой:

```shell
make hosts
```

Выглядит примерно так:

```shell
127.0.0.1  local.develop
127.0.0.1  local.pma.develop
127.0.0.1  local.traefik.develop
```

## Запуск проекта (long way). Если что-то пошло не так или вам хочется понять как это устроено

> ℹ️ Make команда - оболочка над вызовом более низкоуровневых команд (в контексте данного проекта - стандартными командами docker-compose).
Открыв Makefile в корне проекта, вы можете найти детальную информацию о выполняемой команде.

## 1. Работа проекта подразумевает комплект сопоставленных DNS имен с локальным хостом - для добавления DNS в /etc/hosts необходимо выполнить команду

```shell
make hosts
```

## 2. Рассматривая чистую установку - тянем файлы кода и gitlab

```shell
make git-clone
```

## 3. Проект LDE содержит шаблоны переменных для всех микросервисов, копирование в директории проектов можно выполнить одной командой

> 🚨🚨🚨  важно эта команда перезапишет имеющиеся .env файлы в ваших локальных директориях с кодом

```shell
make env-copy
```

## 4. Build образов для локального окружения (если не изменяются Dockerfile выполнить достаточно один раз, если вносятся коррективы, то необходимо делать "re" build)

```shell
make build
```

## 5. Запуск локального docker окружения

```shell
make up
```

## 6. Установка npm пакетов и зависимостей

```shell
make packages-install
```

## 7. Выполняем миграции баз

```shell
make migration
```

## Остановка докер окружения

```shell
make down
```

## Очистка системы по завершению работы

```shell
make rm-code
```

## Как работать с LDE после установки

По завершении команд с префиксом **--rm** контейнер выполнявший работу автоматически удаляется освобождая ресурсы локальной станции.

Пример запуска команд запуска миграций БД для PHP с yii2:

```shell
docker compose exec -it app-back bash -c "cd yii2 && php yii migrate"
```

Все вышеприведенные примеры команды **docker compose exec** **app**= имя сервиса можно выполнить последовательно находясь прямо в оболочке контейнера, например:

Переход в оболочку контейнера рабочей среды:

```shell
docker compose exec app-back bash
```

и далее... (внутри оболочки сервиса/контейнера)

```shell
cd yii2 && php yii migrate
```

и так далее аналогично тому если бы вы выполняли это на хост среде. При этом стоит отметить что права на созданных файлах не будут "страдать" и автором будет ваш пользователь - а не root.

## Доступы по умолчанию

| Url                                 | Login              | Password         | Description                                     |
|-------------------------------------|--------------------|------------------|-------------------------------------------------|
| http://local.develop/backend/       |                    |                  | Админка                                         |
| http://local.develop/               |                    |                  | Фронт                                           |
| http://local.traefik.develop:8080/  |                    |                  | Морда traefik`а                                 |
| http://local.pma.develop/           | root               | root             | Phpmyadmin                                      |
