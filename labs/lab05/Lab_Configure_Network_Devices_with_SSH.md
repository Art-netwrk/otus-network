# Лабораторная работа. Доступ к сетевым устройствам по протоколу SSH
<img width="731" height="339" alt="image" src="https://github.com/user-attachments/assets/88ac4b52-54de-4de7-b36c-b03337f637ce" />

## Задачи

#### Часть 1. Настройка основных параметров устройства
#### Часть 2. Настройка маршрутизатора для доступа по протоколу SSH
#### Часть 3. Настройка коммутатора для доступа по протоколу SSH
#### Часть 4. SSH через интерфейс командной строки (CLI) коммутатора

## Инструкции

### Часть 1. Настройка основных параметров устройств
В части 1 потребуется настроить топологию сети и основные параметры, такие как IP-адреса интерфейсов, доступ к устройствам и пароли на маршрутизаторе.

#### Шаг 1. Создайте сеть согласно топологии.
<img width="542" height="284" alt="image" src="https://github.com/user-attachments/assets/e167e4fd-1c91-4697-b8b2-e21733cfc6bc" />

#### Шаг 2. Выполните инициализацию и перезагрузку маршрутизатора и коммутатора.

```
Switch:
S1# delete flash:vlan.dat
S1# erase startup-config
S1# reload

Router:
R1# erase startup-config
R1# reload
```
#### Шаг 3. Настройте маршрутизатор.

Откройте окно конфигурации

a.	Подключитесь к маршрутизатору с помощью консоли и активируйте привилегированный режим EXEC.

b.	Войдите в режим конфигурации.

c.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

d.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.

e.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.

f.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.

g.	Зашифруйте открытые пароли.

h.	Создайте баннер, который предупреждает о запрете несанкционированного доступа.

i.	Настройте и активируйте на маршрутизаторе интерфейс G0/0/1, используя информацию, приведенную в таблице адресации.

j.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.

```
Router>enable
Router#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#no ip domain-lookup 
Router(config)#enable secret class
Router(config)#line console 0
Router(config-line)#password cisco
Router(config-line)#login
Router(config-line)#exit
Router(config)#line vty 0 4
Router(config-line)#password cisco
Router(config-line)#login
Router(config-line)#exit
Router(config)#service password-encryption 
Router(config)#banner motd "ADMIN ONLY"
Router(config)#interface g0/0/1
Router(config-if)#ip address 192.168.1.1 255.255.255.0
Router(config-if)#no shutdown 
Router(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up
Router(config-if)#exit
Router(config)#end
Router#write memory 
Building configuration...
[OK]
```
#### Шаг 4. Настройте компьютер PC-A.

a.	Настройте для PC-A IP-адрес и маску подсети.

b.	Настройте для PC-A шлюз по умолчанию.

<img width="696" height="292" alt="image" src="https://github.com/user-attachments/assets/fcaf12c8-86f9-4b20-8711-1f9c3cc99925" />

#### Шаг 5. Проверьте подключение к сети.

Пошлите с PC-A команду Ping на маршрутизатор R1. Если эхо-запрос с помощью команды ping не проходит, найдите и устраните неполадки подключения.

<img width="420" height="343" alt="image" src="https://github.com/user-attachments/assets/b6938356-8c41-4ace-bec5-019bdde3a2cd" />

### Часть 2. Настройка маршрутизатора для доступа по протоколу SSH

Подключение к сетевым устройствам по протоколу Telnet сопряжено с риском для безопасности, поскольку вся информация передается в виде открытого текста. Протокол SSH шифрует данные сеанса и обеспечивает аутентификацию устройств, поэтому для удаленных подключений рекомендуется использовать именно этот протокол. В части 2 вам нужно настроить маршрутизатор для приема соединений SSH по линиям VTY.

#### Шаг 1. Настройте аутентификацию устройств.

При генерации ключа шифрования в качестве его части используются имя устройства и домен. Поэтому эти имена необходимо указать перед вводом команды crypto key.
Откройте окно конфигурации

a.	Задайте имя устройства.
```
Router(config)#hostname R1
R1(config)#
```
b.	Задайте домен для устройства.
```
R1(config)#ip domain-name otus.local
```
#### Шаг 2. Создайте ключ шифрования с указанием его длины.
```
R1(config)#crypto key generate rsa
The name for the keys will be: R1.otus.local
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 2048
% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]
```
#### Шаг 3. Создайте имя пользователя в локальной базе учетных записей.
Настройте имя пользователя, используя admin в качестве имени пользователя и Adm1nP @55 в качестве пароля.
```
R1(config)#username admin secret Adm1nP@55
```
#### Шаг 4. Активируйте протокол SSH на линиях VTY.

a.	Активируйте протоколы Telnet и SSH на входящих линиях VTY с помощью команды transport input.
```
R1(config)#line vty 0 4
R1(config-line)#transport input ssh
```
b.	Измените способ входа в систему таким образом, чтобы использовалась проверка пользователей по локальной базе учетных записей.
```
R1(config-line)#login local
R1(config-line)#end
R1#
%SYS-5-CONFIG_I: Configured from console by console
```
#### Шаг 5. Сохраните текущую конфигурацию в файл загрузочной конфигурации.
```
R1#write memory
Building configuration...
[OK]
```
#### Шаг 6. Установите соединение с маршрутизатором по протоколу SSH.

a.	Запустите Tera Term с PC-A.

b.	Установите SSH-подключение к R1. Use the username admin and password Adm1nP@55. У вас должно получиться установить SSH-подключение к R1.

<img width="396" height="257" alt="image" src="https://github.com/user-attachments/assets/00906950-f6ae-42c8-92cc-e8a178395d62" />

### Часть 3. Настройка коммутатора для доступа по протоколу SSH

В части 3 вам предстоит настроить коммутатор для приема подключений по протоколу SSH, а затем установить SSH-подключение с помощью программы Tera Term.

#### Шаг 1. Настройте основные параметры коммутатора.

a.	Подключитесь к коммутатору с помощью консольного подключения и активируйте привилегированный режим EXEC.

b.	Войдите в режим конфигурации.

c.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

d.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.

e.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.

f.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.

g.	Зашифруйте открытые пароли.

h.	Создайте баннер, который предупреждает о запрете несанкционированного доступа.

i.	Настройте и активируйте на коммутаторе интерфейс VLAN 1, используя информацию, приведенную в таблице адресации.

j.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.

```
Switch>enable 
Switch#configure terminal 
Switch(config)#hostname S1
S1(config)#no ip domain-lookup 
S1(config)#enable secret class
S1(config)#service password-encryption 
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#interface vlan 1
S1(config-if)#ip address 192.168.1.11 255.255.255.0
S1(config-if)#no shutdown
S1(config-if)#exit
S1(config)#ip default-gateway 192.168.1.1
S1(config)#write memory 
Building configuration...
[OK]
S1(config)#end
```
#### Шаг 2. Настройте коммутатор для соединения по протоколу SSH.

Для настройки протокола SSH на коммутаторе используйте те же команды, которые применялись для аналогичной настройки маршрутизатора в части 2.

a.	Настройте имя устройства, как указано в таблице адресации.

b.	Задайте домен для устройства.

c.	Создайте ключ шифрования с указанием его длины.

d.	Создайте имя пользователя в локальной базе учетных записей.

e.	Активируйте протоколы Telnet и SSH на линиях VTY.

f.	Измените способ входа в систему таким образом, чтобы использовалась проверка пользователей по локальной базе учетных записей.

```
S1>enable
S1#configure terminal
S1(config)#ip domain-name otus.local
S1(config)#crypto key generate rsa
The name for the keys will be: S1.otus.local
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 2048
% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]
S1(config)#username admin secret Adm1nP@55
S1(config)#line vty 0 4
S1(config-line)#transport input ssh
S1(config-line)#login local
S1(config-line)#exit
```

#### Шаг 3. Установите соединение с коммутатором по протоколу SSH.

Запустите программу Tera Term на PC-A, затем установите подключение по протоколу SSH к интерфейсу SVI коммутатора S1.

Вопрос:

Удалось ли вам установить SSH-соединение с коммутатором?

Да, удалось.

<img width="382" height="241" alt="image" src="https://github.com/user-attachments/assets/41d099e5-59ac-4935-b8af-f8ed9ddc9823" />

### Часть 4. Настройка протокола SSH с использованием интерфейса командной строки (CLI) коммутатора

Клиент SSH встроен в операционную систему Cisco IOS и может запускаться из интерфейса командной строки. В части 4 вам предстоит установить соединение с маршрутизатором по протоколу SSH, используя интерфейс командной строки коммутатора.

#### Шаг 1. Посмотрите доступные параметры для клиента SSH в Cisco IOS.

Откройте окно конфигурации

Используйте вопросительный знак (?), чтобы отобразить варианты параметров для команды ssh.
```
S1#ssh ?
  -l  Log in using this user name
  -v  Specify SSH Protocol Version
```
  
Шаг 2. Установите с коммутатора S1 соединение с маршрутизатором R1 по протоколу SSH.

a.	Чтобы подключиться к маршрутизатору R1 по протоколу SSH, введите команду –l admin. Это позволит вам войти в систему под именем admin. При появлении приглашения введите в качестве пароля Adm1nP@55
```
S1#ssh -l admin 192.168.1.1
Password: 
ADMIN ONLY
R1>
```
b.	Чтобы вернуться к коммутатору S1, не закрывая сеанс SSH с маршрутизатором R1, нажмите комбинацию клавиш Ctrl+Shift+6. Отпустите клавиши Ctrl+Shift+6 и нажмите x. Отображается приглашение привилегированного режима EXEC коммутатора.
```
R1>
S1#
```
c.	Чтобы вернуться к сеансу SSH на R1, нажмите клавишу Enter в пустой строке интерфейса командной строки. Чтобы увидеть окно командной строки маршрутизатора, нажмите клавишу Enter еще раз.
```
S1#
[Resuming connection 1 to 192.168.1.1 ... ]
R1>
```
d.	Чтобы завершить сеанс SSH на маршрутизаторе R1, введите в командной строке маршрутизатора команду exit.
```
R1# exit
[Connection to 192.168.1.1 closed by foreign host]
S1#
```
Вопрос:
Какие версии протокола SSH поддерживаются при использовании интерфейса командной строки?
```
S1#ssh -v ?
  1  Protocol Version 1
  2  Protocol Version 2
```

#### Вопрос для повторения

Как предоставить доступ к сетевому устройству нескольким пользователям, у каждого из которых есть собственное имя пользователя?
```
Нужно создать несколько локальных пользователей и включить аутентификацию login local на VTY-линиях. Тогда каждый пользователь сможет входить под своим именем и паролем.
```
