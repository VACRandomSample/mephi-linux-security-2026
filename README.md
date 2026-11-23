# Защита в операционных системах. ДЗ № 1

Ubuntu 24.04.5 LTS, отдельный чистый учебный дистрибутив `Ubuntu-24.04` в WSL 2 на Windows 11, ядро `6.18.58.1-microsoft-standard-WSL2`. Классические `sudo 1.9.15p5` и GNU coreutils 9.4 — синтаксис правила и формат установки времени работали без замен пакетов.

## Пользователь и пароль

У `user1` UID `1234`, домашний каталог `/home/user1`, отдельная основная группа `user1` и дополнительная `students`. Пароль установлен интерактивным `passwd`, затем `chage -M 90 user1`; по `chage -l` пароль, заданный 23.11.2026, истекает 21.02.2027. В публичной копии строки из `/etc/shadow` хеш заменён на `!` awk-фильтром при извлечении, поля старения сохранены. [passwd](passwd) · [group](group) · [shadow](shadow) · [chage.out](chage.out)

## Мониторинг

`find / -xdev -type f -perm -4000 -print` нашёл 14 файлов до создания учебных копий. В WSL `/`, `/home` и `/var/lib` — один том ext4 без `nosuid`; диски Windows в `/mnt` исключены ключом `-xdev`. [suid-files.out](suid-files.out)

`ps -e -o pid=,ruid=,euid=,comm=` снята, пока `passwd` от `user1` держал запрос нового пароля: реальный UID `1234`, эффективный `0` (сверка UID по `getent passwd 1234`, границы из `UID_MIN`/`UID_MAX`). [euid-root-processes.out](euid-root-processes.out)

## set-UID и capabilities

Копия `stat-suid` принадлежит `root:user1`, режим `4750`. До установки бита запуск от `user1` возвращал `Permission denied`, после — прочитал метаданные закрытого `/var/lib/mephi-zos/proof.txt`. Копия `ping-cap` — `0750` с `cap_net_raw=ep`: до назначения — отказ создания RAW-сокета (код 2), после — один ответ от `127.0.0.1` (код 0). Коды завершения и вывод — в [checks.out](checks.out), атрибуты — в [stat.out](stat.out) и [getcap.out](getcap.out).

## sudo и системное время

Правило в `/etc/sudoers.d/99-mephi-zos` (0440) разрешает `user1` запуск `/usr/bin/date` от root с аргументом вида `MMDDhhmmYYYY.ss`. `visudo -c` — parsed OK; проверка числового аргумента вернула код `0`, `--file=/dev/null` — код `1`. Из интерактивного сеанса `user1` время установлено на `16:03:31 МСК`, код `0`; после перезапуска правило сохранилось. [sudoers](sudoers)

## После перезапуска

После `wsl --terminate Ubuntu-24.04` и повторного входа `stat-suid` и `ping-cap` от `user1` завершились с кодом `0`, правило sudo действует — весь вывод в [checks.out](checks.out). История: команды сеанса администратора и сеанса `user1` — две реальные интерактивные сессии с отдельными HISTFILE, объединённые в [history.out](history.out). На скриншоте — номер студента: [mephi-screenshot.png](mephi-screenshot.png)
