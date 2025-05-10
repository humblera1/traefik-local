
# Traefik для локальной разработки (только HTTP)

Этот проект позволяет использовать [Traefik](https://traefik.io/) как локальный reverse-proxy для разработки на вашей машине. Traefik будет слушать порт 80 и проксировать запросы к вашим приложениям по доменам вида `mysite.local`.

## Шаг 1. Создание docker-сети

Traefik и ваши приложения должны находиться в одной docker-сети. Сначала создайте внешнюю сеть:

```sh
docker network create traefik_net
```

## Шаг 2. Запуск Traefik

Убедитесь, что файл `docker-compose.yml` находится в корне проекта. Запустите Traefik:

```sh
docker compose up -d
```

Traefik теперь слушает порт 80 и готов проксировать запросы.

## Шаг 3. Добавление записи в hosts

Чтобы обращаться к своим приложениям по домену вида `mysite.local`, добавьте запись в файл hosts.

### Windows 10

1. Откройте Блокнот от имени администратора.
2. Откройте файл:  
   `C:\Windows\System32\drivers\etc\hosts`
3. Добавьте строку:
   ```
   127.0.0.1   mysite.local
   ```
4. Сохраните файл.

### Linux (Ubuntu)

Откройте терминал и выполните:

```sh
sudo nano /etc/hosts
```

Добавьте строку:

```
127.0.0.1   mysite.local
```

Сохраните файл (Ctrl+O, Enter, Ctrl+X).

## Шаг 4. Пример docker-compose для приложения

Пример сервиса приложения, который будет доступен по адресу http://mysite.local:

```yaml
services:
  app:
    image: your-app-image
    container_name: app
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.app.rule=Host(`mysite.local`)"
      - "traefik.http.routers.app.entrypoints=web"
    networks:
      - traefik_net
```

- `traefik.enable=true` — включает Traefik для этого контейнера.
- `traefik.http.routers.app.rule=Host(`mysite.local`)` — указывает домен, по которому будет доступно приложение.
- `traefik.http.routers.app.entrypoints=web` — использует HTTP (80 порт).

## Дашборд Traefik

Traefik предоставляет удобный дашборд для мониторинга и отладки маршрутов.

### Как включить дашборд

1. В `docker-compose.yml` должен быть проброшен порт 8080 и добавлены параметры:

    ```yaml
    command:
      - "--api.dashboard=true"
      - "--api.insecure=true"
      - "--entrypoints.traefik.address=:8080"
    ports:
      - "8080:8080"
    ```

2. После запуска Traefik дашборд будет доступен по адресу:  
   [http://localhost:8080](http://localhost:8080)

> ⚠️ **Внимание:**  
> Дашборд открыт без авторизации (`--api.insecure=true`). Не делайте так в продакшене!

## Примечания

- Для каждого нового приложения используйте уникальный домен (например, `another.local`) и добавляйте его в hosts.
- Если используете динамические правила, поместите их в папку `dynamic`.

---

**Теперь вы можете открывать http://mysite.local в браузере и видеть ваше приложение!**

---

## Troubleshooting

### 80 порт должен быть свободен

Traefik не сможет стартовать, если порт 80 уже занят другим процессом (например, IIS, Apache, Nginx и др.).

#### Как проверить, занят ли порт 80

##### Windows

Откройте командную строку и выполните:

```sh
netstat -ano | findstr :80
```

В колонке PID будет идентификатор процесса, который занимает порт.

##### Linux (Ubuntu)

```sh
sudo lsof -i :80
```
или
```sh
sudo netstat -tulpn | grep :80
```

#### Как освободить порт 80

- Найдите процесс по PID и завершите его через диспетчер задач (Windows) или командой `kill` (Linux).
- На Windows часто порт 80 занимает служба "World Wide Web Publishing Service" (IIS). Остановите её:

    ```sh
    net stop w3svc
    iisreset /stop
    ```

- На Linux остановите соответствующий сервис, например:

    ```sh
    sudo systemctl stop apache2
    sudo systemctl stop nginx
    ```

После освобождения порта 80 перезапустите Traefik:

```sh
docker compose up -d
```
