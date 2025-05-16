---
title: Проброс SSH agent из Windows в WSL
description: Сквозная авторизация по SSH ключам
date: 2023-08-29 13:13:18 +03:00
categories: blog
layout: post
tags: ["KeePassXC", "ssh", "socat"]
---

Я давно использую для хранения паролей, 2FA (TOTP) и SSH-ключей KeePassXC, у него прям [из коробки доступен экспорт ключей в SSH агента](https://github.com/keepassxreboot/keepassxc/blob/develop/docs/topics/SSHAgent.adoc). При логине KeePassXC может сам экспортировать сохранённые в шифрованном контейнере приватные ключи и применять к ним пароли. Но как-то не было задачи ходить в SSH прямо из WSL, так как WSL доступен локально, а SSH - удалённые хосты, на которые можно ходить через OpenSSH прямо из основной системы. Но вот спросили и в быстропоиске нашлось такое решение

[https://spin.atomicobject.com/2022/10/05/ssh-keys-linux/](https://spin.atomicobject.com/2022/10/05/ssh-keys-linux/)

Немного актуализировал и прикрутил в автозапуск (да, я ленивый!)

### Windows
```shell
winget install GoLang.Go
go install github.com/jstarks/npiperelay@latest
```

### WSL
```shell
sudo apt install socat
```

Добавляем в `~/.bashrc` следующий код
```
SSH_AUTH_SOCK=$HOME/.ssh/wsl-ssh-agent.sock
if [ ! -S "$SSH_AUTH_SOCK" ]
then
  export SSH_AUTH_SOCK
  /usr/bin/socat UNIX-LISTEN:"$SSH_AUTH_SOCK",fork EXEC:"$(which npiperelay.exe) -ei -s //./pipe/openssh-ssh-agent",nofork 2>&1 &
fi
```

Выходим из WSL, заходим назад, радуемся выдаче `ssh-add -L`

Если не прокатило, разбираемся почему ~/.bashrc не выполняется. Например тщательно читаем `~/.profile` и избавляемся от пустых `~/.bash_profile` и `~/.bash_login`

Ну а `~/.ssh/config` и прочие `known_hosts` можно подкинуть с основной системы в WSL просто симлинками.

Иногда файл $HOME/.ssh/wsl-ssh-agent.sock ошибочно сохраняется между сессиями и чтобы агент заработал его нужно руками удалить.
