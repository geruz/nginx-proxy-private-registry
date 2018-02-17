# Приватный docker registry через проксируюший nginx

Nginx должен быть установлен, если получать сертивкаты через certbot
то он тоже должен быть установелен 


Папки необходимые для **registry**
* **/opt/registry/auth** файлы для http авторизации
* **/opt/registry/crts** ключи для SSL
* **/opt/registry/data** папка для хранения оброзов

Переменные 
* **CRTS_DIR** - папка где храняться pem сертификаты
* **REGISTRY_USER** - пользователь под которым будете авторизовыватся (```docker login registry.example.com```) 
* **REGISTRY_PASS** - пароль для авторизации (```docker login registry.example.com```) 

## получение сертификата через certbot
```
certbot --authenticator webroot --installer nginx
```

## Генерируем пользователя и пароль для http авторизации

```
docker run --rm --entrypoint htpasswd registry:2 -Bbn REGISTRY_USER REGISTRY_PASS > /opt/registry/auth/htpasswd
```

## Копирование ключей для registry
```
cp CRTS_DIR/privkey.pem /opt/registry/certs/registry.key
cat CRTS_DIR/cert.pem CRTS_DIR/chain.pem> /opt/registry/certs/registry.crt
```

## Запуск контейнера 
```
docker run -d \
  -p 127.0.0.1:5000:5000 \
  --restart=always \
  --name registry \
  -e REGISTRY_AUTH=htpasswd \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  -e "REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.crt" \
  -e "REGISTRY_HTTP_TLS_KEY=/certs/registry.key" \
  -v /opt/registry/auth:/auth \
  -v /opt/registry/certs:/certs \
  -v /opt/registry/data:/var/lib/registry \
  registry:2
  ```
