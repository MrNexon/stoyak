# Деплой контейнеров на @stoyak

## Нейминг стеков
Стеки должны быть названы в формате: `<subdomain>_<host>_<domain>`

> (Напримем, для проекта по адресу `registry.sostoya.com` стек должен быть назван `registry_sostoya_com`)

## Подключение к хосту
1) В контейнере (сервисе) в docker compose должен быть опубликован принимающий порт
> Для `registry` это выглядит так:<br>
> ```yaml
> expose:
>   - 5000
> ```

2) Для контейнера (сервиса) должна быть указана `proxy` сеть, которая подключена к traefik
```yaml
networks:
   - proxy
```

Также внизу всего docker compose должно быть прописано:
```yaml
networks:
  proxy:
    external: true
```

3) Для публикуемого контейнера нужно прописать `labels`
```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.<ROUTER_NAME>.rule=Host(`<HOST>`)"
  - "traefik.http.routers.<ROUTER_NAME>.entrypoints=<ENTRY_POINT>"
```

| Key           | Values          | Description                                             |
|---------------|-----------------|---------------------------------------------------------|
| `ROUTER_NAME` | `[a-zA-Z_-]`    | Имя роутера уникальное на всем сервере                  |
| `HOST`        | `[a-zA-Z_-.\d]` | Хост (например: `registry.sostoya.com`)                 |
| `ENTRY_POINT` | `web`, `secure` | Входная точка traefik (`web` - 80 порт, `secure` - 443) |

> У контейнера (сервиса) может быть несколько роутеров (если например требуется опубликовать несколько портов). 
> Подробную документацию по правилам роутинга чекай [тут](https://doc.traefik.io/traefik/routing/routers/)


> **Важно!**
> 
> Если требуется автоматически сгенерировать сертификат, то нужно добавить еще один label:<br>
> `- "traefik.http.routers.<ROUTER_NAME>.tls.certresolver=ssl"`

> Также можно закрыть сервис под vpn
> 
> Для этого надо добавить middleware на роутер:<br>
> `- "traefik.http.routers.<ROUTER_NAME>.middlewares=vpn@docker"`

# Хуяк хуяк и в продакшен
Для быстрого копирования

```yaml
    expose:
      - PORT
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.ROUTER_NAME.rule=Host(`HOST`)"
      - "traefik.http.routers.ROUTER_NAME.entrypoints=secure"
      - "traefik.http.routers.ROUTER_NAME.tls.certresolver=ssl"
    networks:
      - proxy

networks:
  proxy:
    external: true
```

## Пример
```yaml
version: '3'

services:
  registry:
    image: registry:2
    environment:
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
      
    # Публикуем порт (он не будет опубликован для сети, это только его объявление)
    expose:
      - 5000 
  
    volumes:
      - "storage:/var/lib/registry"
    restart: always
    
    # Подключаем сервис к сети traefik
    networks:
      - proxy
    
    # Проставляем метки для traefik
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.registry.rule=Host(`registry.sostoya.com`)"
      - "traefik.http.routers.registry.entrypoints=secure"
      - "traefik.http.routers.registry.tls.certresolver=ssl"

volumes:
  storage:

# Говорим что proxy сеть не нужно создавать, она уже создана traefik
networks:
  proxy:
    external: true
```
