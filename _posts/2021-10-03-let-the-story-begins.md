---
title:  "Выходные с Let's encrypt"
date:   2021-10-03 13:35:00 +3
categories: jekyll update
---

30 сентября многих "обрадовал" Let's Encrypt своим устаревшим сертификатом.
Причём проблемы проявились даже на сравнительно недавно купленной читалке.
Причём не только с собственным инстансом NextCloud, но и, например, 
сервисом Pocket который использовал сертификаты от Amazon

Делать нечего, пришлось обновляться, искать обходные пути и набивать очередные шишки.
А потом я решил что пора бы и обновить всё и вся и завести бложик вместо заглушки на
главной странице.

# Мой зоопарк и как укращал

Есть у меня небольшой личный сервер на котором крутится NextCloud
Не имея желания строить и перестраивать всё с нуля, крутится эта кухня в Docker
обмазанном docker-compose. А ещё точнее образ `jwilder/nginx-proxy` к которому
был прикручен `jrcs/letsencrypt-nginx-proxy-companion`. Из них получается хитромудрый
реверс-прокси, который динамически строит свой конфиг для запуска зоопарка сайтов и 
приложений, да ещё и сам на всё это дело стягивает сертификаты с Let's Encrypt.
В процессе обновления пришлось `jrcs/letsencrypt-nginx-proxy-companion` заменить на
`nginxproxy/acme-companion`, т.к. старый перестал развиваться. Отличаются они тем,
что первый использовал `certbot`, а второй использует `acme.sh` и хорошо бы под него и
volume отдельный сразу сделать, чтобы хранить промежуточные данные и не словить недельный 
бан за превышение limits rate.

Помимо обновления для поддержки старых железок нужно было указать, что сертификаты должны
использовать `preferred_chain=ISRG Root X1`. А для совсем уж легаси (как например старые
электронные книги) может понадобиться и поддержка старых протоколов защиты, вплоть до 
tls 1.0. Что автоматически понижает рейтинг сайта на всяких ssllabs.com с A+ до B. Мол
нечего всякое решето использовать. Вроде как следом и всякие поисковики тоже могут 
понижать рейтинг сайта в поисковой выдаче, но т.к. это моя личная песочница и ничего
критичного тут нет - пофиг.

`docker-compose.yml` выглядит теперь так
```docker-compose
version: '3'

networks:
    default:
        external:
            name: network

services:
    nginx-proxy:
        container_name: nginx-proxy
        image: jwilder/nginx-proxy
        ports:
            - 80:80
            - 443:443
        volumes:
            - vhost.d:/etc/nginx/vhost.d
            - ./certs:/etc/nginx/certs:ro
            - html:/usr/share/nginx/html
            - /var/run/docker.sock:/tmp/docker.sock:ro
            - ./conf.d/client_max_body_size.conf:/etc/nginx/conf.d/client_max_body_size.conf:ro
        environment:
            # - SSL_POLICY=AWS-TLS-1-1-2017-01
            - SSL_POLICY=AWS-2016-08
        restart: always

    nginx-proxy-letsencrypt:
        container_name: nginx-proxy-letsencrypt
        # image: jrcs/letsencrypt-nginx-proxy-companion
        image: nginxproxy/acme-companion
        volumes:
            - acme:/etc/acme.sh
            - vhost.d:/etc/nginx/vhost.d
            - ./certs:/etc/nginx/certs:rw
            - html:/usr/share/nginx/html
            - /var/run/docker.sock:/var/run/docker.sock:ro
        environment:
            - NGINX_PROXY_CONTAINER=nginx-proxy
            - acme_preferred_chain=ISRG Root X1
        restart: always

volumes:
    vhost.d:
    html:
    acme:                                                                                                                                                                                                                                                                                                                                            acme:
```

После всех плясок с сертификатами сам NextCloud обновлён до текущей 
стабильной версии 21.0.5.

Процесс тоже не без приключений. Если сам образ можно обновить 
банальной заменой версии в docker-compose.yml, то базу и конфиги
после каждого шага повышения версии (до той, которую сам инстанс 
советует в настройках) приходится фиксить поочерёдно выполняя

```bash
$ docker exec -it --user www-data cloudyakauleupro_nextcloud_1 bash

$ php occ upgrade
$ php occ db:add-missing-indices
$ php occ db:add-missing-columns
$ php occ db:add-missing-primary-keys
$ php occ db:convert-filecache-bigint
$ php occ maintenance:repair --include-expensive
$ php occ maintenance:mode --off 
```

и добавляя параметры к MariaDB чтобы php не отваливался.

Актуальный `docker-compose.yml` для NextCloud выглядит теперь так

```docker-compose
version: '3.0'

services:
    nextcloud:
        image: nextcloud:21.0.5
        environment:
            - VIRTUAL_HOST=%your.domain%
            - LETSENCRYPT_HOST=%your.domain%
            - LETSENCRYPT_EMAIL=admin@%your.domain%
            - LC_ALL=C.UTF-8
            - LANG=C.UTF-8
        volumes:
            - ./nextcloud:/var/www/html
            - ./data:/var/www/html/data
        networks:
            - default
            - backend
        links:
            - db
        restart: always

    db:
        image: mariadb
        command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW --innodb-file-per-table=1 --skip-innodb-read-only-compressed --innodb_read_only_compressed=OFF
        volumes:
            - ./db:/var/lib/mysql
        environment:
            - MYSQL_ROOT_PASSWORD=%всякое_личное%
            - MYSQL_PASSWORD=%всякое_личное%
            - MYSQL_DATABASE=nextcloud
            - MYSQL_USER=nextcloud
        networks:
            - backend
        restart: always

networks:
  default:
    external:
      name: network
  backend:
```

Ну а базовый сайт-заглушку я в итоге решил наполнить из Jekyll.
До этого смотрел всякие GitBooks (умер, как открытый проект), Pandoc,
MkDocs... не тот формат. Хотя для документации MkDocs выглядит круто.

С Jekyll можно отделаться минимальной болью через тот же Docker

```bash
$ docker run --rm --volume="%cd%:/srv/jekyll" -it jekyll/jekyll:3.8 jekyll new "Your New Blog"
$ cd "Your New Blog"
... change what you want
$ docker run --rm --volume="%cd%:/srv/jekyll" -it jekyll/jekyll:3.8 jekyll build
```

А потом уже думать над CI/CD на свой хостинг.

Актуальный `docker-compose.yml` для простого сайта

```docker-compose
version: '3'

networks:
  default:
    external:
      name: network

services:
    site:
        image: nginx:latest
        volumes:
            - ./html:/usr/share/nginx/html
        environment:
            - VIRTUAL_HOST=%your.domain%
            - LETSENCRYPT_HOST=%your.domain%
            - LETSENCRYPT_EMAIL=admin@%your.domain%
        restart: always
```

Разумеется доменные имена у разных сайтов разные.
