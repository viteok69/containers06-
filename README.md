# Lucrarea 7: Crearea unei aplicații multi-container

> Realizat de studentul: Badia Victor \
> Grupa: I2301
> \
> Verificat de Mihail Croitor

## Scopul Lucrării

Famialiarizarea cu gestiunea aplicației multi-container creat cu docker-compose.

## Sarcina

Creați o aplicație php pe baza a trei containere: nginx, php-fpm, mariadb, folosind docker-compose.

## Pregatire

Pentru a efectua această lucrare, trebuie să aveți instalat pe computer Docker.
Lucrarea se efectuează pe baza laboratorului nr. 5.

## Executare

Creați un repozitoriu containers06 și copiați-l pe computerul dvs.
![image](https://github.com/user-attachments/assets/3cf60e57-25cb-473a-8ce0-71111419c098)

## Site-ul pe php

În directorul containers06 creați directorul mounts/site. În acest director, rescrieți site-ul pe php, creat în cadrul disciplinei php.
![image](https://github.com/user-attachments/assets/e0d22a28-7a4b-4501-80fb-e14a9c10ca96)

## Fișiere de configurare

Creați fișierul .gitignore în rădăcina proiectului și adăugați următoarele linii:

```bash
# Ignore files and directories
mounts/site/*
```
![image](https://github.com/user-attachments/assets/4988ad50-732b-462d-b096-b3b77c336590)

Creați în directorul containers06 fișierul nginx/default.conf cu următorul conținut:

```bash
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.php;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    location ~ \.php$ {
        fastcgi_pass backend:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```
![image](https://github.com/user-attachments/assets/008b527f-afe8-4b5f-923b-a3b2209d6973)
![image](https://github.com/user-attachments/assets/adac759d-76a5-452f-abd6-63b22e1fdc8a)

Creați în directorul containers06 fișierul docker-compose.yml cu următorul conținut:

```bash
version: '3.9'

services:
  frontend:
    image: nginx:1.19
    volumes:
      - ./mounts/site:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "80:80"
    networks:
      - internal
  backend:
    image: php:7.4-fpm
    volumes:
      - ./mounts/site:/var/www/html
    networks:
      - internal
    env_file:
      - mysql.env
  database:
    image: mysql:8.0
    env_file:
      - mysql.env
    networks:
      - internal
    volumes:
      - db_data:/var/lib/mysql

networks:
  internal: {}

volumes:
  db_data: {}
```

![image](https://github.com/user-attachments/assets/ef9eb7c6-385f-44cd-a13d-7dbf0b0c651a)
![image](https://github.com/user-attachments/assets/90981c87-82b4-4861-983f-3586e34914d7)

Creați fisierul mysql.env în rădăcina proiectului și adăugați în el următoarele linii:

```bash
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=app
MYSQL_USER=user
MYSQL_PASSWORD=secret
```

![image](https://github.com/user-attachments/assets/8bc4f041-132a-4b81-8e7f-3cc314a94c0b)
![image](https://github.com/user-attachments/assets/58b3ea99-bb61-4788-92de-b8424e2f865a)

## Pornire și testare

Porniți containerele cu comanda:
```bash
docker-compose up -d
```
![image](https://github.com/user-attachments/assets/539cf00b-eb18-4736-8d8d-f9e7708b2801)

Verificați funcționarea site-ului în browser, trecând la adresa http://localhost. Dacă este afișată pagina de bază nginx, atunci reîncărcați pagina.

## Concluzie

În cadrul acestei lucrări de laborator am realizat o aplicație web PHP modularizată, utilizând o arhitectură multi-container Docker orchestrată cu docker-compose. Aplicația este compusă din trei servicii principale:
- nginx – serverul web care servește conținutul static și redirecționează cererile PHP către backend;
- php-fpm – procesatorul PHP care interpretează fișierele .php;
- MariaDB (MySQL) – baza de date relațională utilizată pentru stocarea datelor aplicației.
Această abordare reflectă o bună practică în dezvoltarea modernă a aplicațiilor, oferind separare clară a responsabilităților, ușurință în testare, mentenanță și scalabilitate. De asemenea, utilizarea docker-compose permite pornirea și gestionarea facilă a întregului mediu de dezvoltare cu o singură comandă, reducând considerabil timpul de configurare manuală.

## Intrebari

1. În ce ordine sunt pornite containerele?

Containerele sunt pornite conform ordinii specificate în fișierul docker-compose.yml, iar aceasta este influențată de dependențele dintre servicii și rețelele definite. 
- database – este primul container pornit, deoarece acesta trebuie să aibă baza de date disponibilă înainte ca celelalte servicii să poată interacționa cu ea.
- backend – se lansează după ce database este activ, deoarece serviciul PHP (backend) va interacționa cu baza de date.
- frontend – este lansat ultima instanță, pentru că depinde de backend, care trebuie să răspundă la cererile PHP.

2. Unde sunt stocate datele bazei de date?

Datele bazei de date sunt stocate într-un volum Docker specificat în fișierul docker-compose.yml:
```bash
volumes:
  db_data: {}
```

3. Cum se numesc containerele proiectului?

Denumirele containerelor proiectelor sunt: containers06_frontend_1, containers06_backend_1, containers06_database_1.


