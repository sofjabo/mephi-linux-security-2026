## Раздел 1. Создание пользователя

| Необходимо | Команда | Результат |
|---|---|---|
| Создать user1 с UID 1234 | `sudo useradd -u 1234 -m user1` | `uid=1234(user1)` |
| Группа students | `sudo groupadd students`<br>`sudo usermod -aG students user1` | `groups=1234(user1),1235(students)` |
| Смена пароля каждые 3 месяца | `sudo chage -M 90 user1` | `Maximum number of days between password change : 90` |

**Проверка:**
```
$ id user1
uid=1234(user1) gid=1234(user1) groups=1234(user1),1235(students)

$ sudo chage -l user1
Last password change                                    : Sep 12, 2026
Password expires                                        : Dec 11, 2026
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 90
Number of days of warning before password expires       : 7
```

## Раздел 2. Мониторинг файлов и процессов

### 2.1. Поиск файлов с битом set-UID

| Команда | Артефакт |
|---|---|
| `sudo find / -perm -4000 -type f 2>/dev/null` | `task21.out` |

### 2.2. Поиск процессов с eUID=0, rUID≠0

| Команда | Артефакт |
|---|---|
| `ps -eo pid,ruid,euid,comm --sort=pid \| awk 'NR==1 \|\| ($3==0 && $2!=0)'` | `task22.out` |

## Раздел 3. Изучение механизма set-UID 

### 3.1. Выбор утилиты

Выбрана утилита `cat`. Привилегированная операция - чтение файла `/etc/shadow`, который недоступен обычному пользователю.

### 3.2. Выполнение привилегированной операции

| Шаг | Команда |
|---|---|
| Скопировать утилиту | `cp /usr/bin/cat ./mycat` |
| Сменить владельца на root | `sudo chown root:root ./mycat` |
| Установить бит set-UID | `sudo chmod u+s ./mycat` |

**Проверка:**
```
ls -l ./mycat

-rwsr-xr-x. 1 root root 36568 Sep 12 22:24 ./mycat

./mycat /etc/shadow | head -3

root:$y$j9T$IEsCEbbLkv/RUp.4jsrpQe8j$oiCHloeYMxTyhJEJlbM0P5untTsSPqc.DlBybHuRPU0::0:99999:7:::
bin:*:20047:0:99999:7:::
daemon:*:20047:0:99999:7:::
```

## Раздел 4. Изучение механизма привилегий 

### 4.1. Выбор утилиты

Выбрана утилита `chown`. Привилегированная операция — изменение владельца файла на произвольного пользователя.

### 4.2. Выполнение привилегированной операции

| Шаг | Команда |
|---|---|
| Скопировать утилиту | `cp /usr/bin/chown ./mychown` |
| Выдать capability CAP_CHOWN | `sudo setcap cap_chown+ep ./mychown` |

**Проверка:**
```
touch testfile 
ls -l ./testfile
-rw-r--r--. 1 sonya sonya 0 Sep 12 22:35 ./testfile

./mychown user1 ./testfile 
ls -l ./testfile
-rw-r--r--. 1 user1 sonya 0 Sep 12 22:35 ./testfile
```

## Раздел 5. Изучение механизма sudo 

Необходимо разрешить `user1` устанавливать системное время.

| Шаг | Команда |
|---|---|
| Редактирование sudoers | `sudo EDITOR=nano visudo` |
| Добавленная строка | `user1 ALL=(root) /usr/bin/timedatectl` |

**Проверка:**
```
su−user1 
sudo timedatectl set-time "2026-09-12 12:00:00"
```