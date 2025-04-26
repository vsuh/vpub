## Задача 

Разрабатывается статичный сайт на сервере в локальной сети без выхода в интернет. 
- Сайт доступен по адресу  http://192.168.Х.Х/index.html (http://hl)
- Содержимое сайта лежит в /var/www/html сервера 192.168.Х.Х. 
- Вебсервер - `nginx`
 
Изменения проекта помещаются в git репозиторий https://github.com/USER/www 
 
Необходимо, чтобы при пуше в репозиторий, содержимое каталога /var/www/html обновлялось

## Описание решения
Для автоматического обновления содержимого каталога /var/www/html при пуше в репозиторий, нужно настроить post-receive hook в bare-репозитории на сервере. 

## Цель
Обеспечить автоматическое обновление /var/www/html при каждом пуше в https://github.com/USER/www


## 🔧 Шаги по настройке:

> предполагается, что репозиторий https://github.com/USER/www уже создан (доступен)

1. 📥 Склонируй репозиторий в /opt/www-repo:
```bash
sudo mkdir -p /opt/www-repo
sudo chown -R $USER:$USER /opt/www-repo
git clone https://github.com/USER/www.git /opt/www-repo
```

2. 📤 Настрой bare-репозиторий для приёма пушей:
```bash
sudo mkdir -p /opt/www-bare.git
cd /opt/www-bare.git
git init --bare
```

3. 🧩 Создай post-receive hook:
```bash
sudo nano /opt/www-bare.git/hooks/post-receive
```

Вставь следующее содержимое:

```bash
#!/bin/bash
GIT_WORK_TREE=/var/www/html git checkout -f
```

Сделай файл исполняемым:

```bash
sudo chmod +x /opt/www-bare.git/hooks/post-receive
```

4. ⛓️ Свяжи локальный каталог с этим репозиторием
Теперь нужно, чтобы GitHub пушил изменения не в обычный репозиторий, а в наш bare.
```bash
cd /opt/www-repo
git remote add live /opt/www-bare.git
```

Теперь, чтобы выложить изменения (после коммита):
```bash
git push live master
```
Если основная ветка — `main`, замени `master` на `main`.

## 🧪 Проверка
Измени какой-нибудь файл в /opt/www-repo, сделай коммит, и затем:
```bash
git push live main
```

Проверь, что `/var/www/html` обновилось.

## Перенос разработки на другой компьютер

1. 📦 Клонируй GitHub-репозиторий
```bash
git clone https://github.com/USER/www.git
cd www
```
Это твоя рабочая копия проекта.

2. ➕ Добавь live-удалённый репозиторий (тот самый bare-репозиторий на сервере)

Предположим, у тебя есть SSH-доступ к серверу, и он называется hl.local (или IP-адрес, например 192.168.2.2). Тогда добавляй так:
```bash
git remote add live ssh://USER@hl.local/opt/www-bare.git
```

Замени `USER` на имя пользователя на сервере, и hl.local на IP или hostname сервера.

Пример:
```bash
git remote add live ssh://valera@192.168.1.100/opt/www-bare.git
```

3. 🧪 Проверь подключение
```bash
git remote -v
```

Ты должен увидеть:
```
origin  https://github.com/USER/www.git (fetch)
origin  https://github.com/USER/www.git (push)
live    ssh://user@server/opt/www-bare.git (push)
```

4. 🚀 Публикация сайта
После любых изменений:
```bash

git add .
git commit -m "Изменения"
git push origin main       # сохраняем на GitHub
git push live main         # деплой на сервер
```


### 🔐 Упрощение: доступ по SSH без пароля
Чтобы не вводить пароль при каждом пуше:

На локальной машине:

ssh-keygen -t ed25519 -C "your_email@example.com"
Скопируй публичный ключ на сервер:

ssh-copy-id user@server
Теперь git push live будет работать без пароля.

✅ Готово!
==Теперь ты можешь разработать сайт на любом компьютере, просто добавив live-репозиторий как второй remote. Всё работает централизованно и автоматически — GitHub для хранения истории, локальный сервер для мгновенного выкладывания.==

> Хочешь, чтобы git push origin сам триггерил деплой на сервер (без второго push live)? Тогда можем перейти к GitHub Actions + self-hosted runner.