# Eco Zubr Bot

Презентация: https://disk.360.yandex.ru/i/MgaQwiw2G-aTsg

Telegram бот на MAX Long Poll API с Java Spring Boot и PostgreSQL.

## Инструкция по запуску

### 1. Клонируйте репозиторий

git clone https://github.com/ZhenShenITIS/eco-zubr-bot.git

cd eco-zubr-bot

### 2. Создайте .env файл в корневой папке проекта

cp .env.example .env

### Отредактируйте \`.env\` и добавьте:

- \`MAX_TOKEN\` — ваш токен MAX API
- \`YANDEX_GEOCODER_TOKEN\` — ваш токен Яндекс Геокодера
  Следующие аттрибуты можно оставить как есть(по умолчанию докер создаст базу данных eco_zybr20, пользователя userp, с паролем 12345. см. Dockerfile, init-db.sql. Если оставить следующие поля по умолчанию, то отдельно создавать бд не потребуется)
- \DB_USER=userp (Пользователь бд)
- DB_PASSWORD=12345 (Пароль пользователя)
- DB_NAME=eco_zybr20 (имя бд)
- DB_HOST=postgres (хост бд)
- DB_PORT=5432 (порт бд)

### Как получить токен YANDEX_GEOCODER_TOKEN:

1) перейти по ссылке: https://developer.tech.yandex.ru/
2) Нажать подключить api и выбрать Javascript Api и HTTP Геокодер
3) Заполнить форму и нажать Отправить
4) Скопировать предоставленный api токен и добавить в файл .env

### 3. Собрать и запустить контейнеры

docker-compose up -d --build

### 4. Смотреть логи

docker-compose logs -f app

Бот запустится и подключится к PostgreSQL.

### 5.1 Остановить

docker-compose down

### 5.2 Остановить и удалить бд:

docker-compose down -v

# Описание:

# Eco Zubr Bot - Docker образ

## Multi-stage сборка (две стадии)

### Стадия 1: Builder (Компиляция)

- Базовый образ: eclipse-temurin:21-jdk-jammy
- Назначение: Компилировать Java приложение

Этапы:
1. Установить Git и Maven
2. Клонировать и собрать библиотеку MAX Bot API Client
3. Клонировать и собрать библиотеку MAX Bot SDK
4. Скачать pom.xml и исходный код
5. Скомпилировать: mvn clean package

Результат: target/*.jar (скомпилированное приложение)

### Стадия 2: Runtime (Финальный образ)

Назначение: Запустить Java приложение

Этапы:
1. Скопировать скомпилированный JAR из стадии Builder
2. Скопировать статические файлы (PNG изображения)
3. Открыть порт 8080
4. Установить переменные окружения
5. Запустить: java -jar app.jar

## Переменные окружения

SPRING_DATASOURCE_URL=""
SPRING_DATASOURCE_USERNAME=""
SPRING_DATASOURCE_PASSWORD=""
SPRING_JPA_HIBERNATE_DDL_AUTO=update

MAX_TOKEN=""
YANDEX_GEOCODER_TOKEN=""

(Переопределяются docker-compose.yml во время запуска)

## Возможности

- Java 21 runtime (среда выполнения)
- Spring Boot 3.5.7 приложение
- Драйвер PostgreSQL включен
- Long Poll бот для MAX API
- Hibernate ORM с PostgreSQL
- Lombok для генерации кода

# Eco Zubr Bot - Docker Compose конфигурация

## Версия: 3.8

## Сервис 1: PostgreSQL (База данных)

### Основное

Образ: postgres:11-alpine
Имя контейнера: eco-zubr-postgres

### Переменные окружения

POSTGRES_USER: postgres - Администратор БД

POSTGRES_PASSWORD: postgres - Пароль администратора

POSTGRES_DB: postgres - Служебная БД (создаётся при запуске)

### Volumes

postgres_data:/var/lib/postgresql/data - Сохраняет данные БД между перезагрузками контейнера

./init-db.sql:/docker-entrypoint-initdb.d/init.sql -Копирует скрипт инициализации в контейнер. PostgreSQL автоматически выполнит его при первом запуске. Создаст пользователя userp и БД eco_zybr20

### Проверка здоровья (Healthcheck)

test: ["CMD-SHELL", "pg_isready -U postgres"] - проверяет: готова ли БД к подключениям?

### Сеть

networks: app-network - подключен к сети app-network

## Сервис 2: Spring Boot приложение (Бот)

### Основное

build: . - Собрать образ из Dockerfile в текущей папке

container_name: eco-zubr-app - Имя контейнера

### Переменные окружения (из .env)

SPRING_DATASOURCE_URL: jdbc:postgresql://${DB_HOST}:${DB_PORT}/${DB_NAME}
SPRING_DATASOURCE_USERNAME: ${DB_USER}
SPRING_DATASOURCE_PASSWORD: ${DB_PASSWORD}

SPRING_JPA_HIBERNATE_DDL_AUTO: update
Hibernate автоматически создаёт/обновляет таблицы

MAX_TOKEN: ${MAX_TOKEN}
Токен MAX API (из .env)

YANDEX_GEOCODER_TOKEN: ${YANDEX_GEOCODER_TOKEN}
Токен Яндекс Геокодера (из .env)

### Зависимости (depends_on)

depends_on:
postgres:
condition: service_healthy

## Volumes

postgres_data:
Именованный том для хранения данных БД
Создаётся автоматически при первом запуске
Сохраняет данные между перезагрузками

Где хранятся?
Linux/Mac: ~/.docker/volumes/eco-zubr-bot_postgres_data/_data/
Windows: %LOCALAPPDATA%\Docker\volumes\eco-zubr-bot_postgres_data\_data\

## Сеть (Networks)

app-network:
driver: bridge

Сеть типа bridge (виртуальная сеть между контейнерами)

## Порядок запуска

1. docker-compose up -d --build

2. Создаёт и запускает postgres контейнер
   Выполняет init-db.sql
   Создаёт userp и БД eco_zybr20
   Ждёт пока healthcheck пройдёт

3. Проверяет: postgres healthy? ДА
   Можно запускать app

4. Собирает Docker образ (из Dockerfile)
   Компилирует Java код

5. Запускает app контейнер
   Читает переменные окружения из .env
   Подключается к postgres
   Стартует Spring Boot
   Бот готов!

