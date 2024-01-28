## Маршрутизация. Пример.


<img src="../misc/images/network_route.png" alt="network_route" width="500"/>  


Давайте рассмотрим (на картинке) пример инфраструктуры с несколькими подсетями.

Пример таблицы маршрутизации для хоста eggplant:
```
[root@eggplant ~]# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
128.17.75.0      128.17.75.20   255.255.255.0   UN        1500 0          0 eth0
default          128.17.75.98   0.0.0.0         UGN       1500 0          0 eth0
127.0.0.1        127.0.0.1      255.0.0.0       UH        3584 0          0 lo
128.17.75.20     127.0.0.1      255.255.255.0   UH        3584 0          0 lo
```
Как видно на примере вывода команды `netstat -rn`:  

Первая запись (строка) предназначена для сети *128.17.75.0/24*, все пакеты для этой сети будут отправляться на шлюз *128.17.75.20*, который является IP-адресом самого хоста.  

Вторая запись представляет собой маршрут по умолчанию, который применяется ко всем пакетам, отправляемым в сети, не указанные в этой маршрутной таблице.  
Здесь маршрут проходит через хост papaya (IP *128.17.75.98*), который является выходом к внешнему миру.  
Этот маршрут должен быть записан во всех машинах сети *128.17.75.0/24*, которые должны иметь доступ к другим сетям.   

Третья запись создана для интерфейса обратной связи (lo). Этот адрес используется, если машина должна подключиться к самой себе через протокол **TCP/IP**.

Последняя запись в таблице маршрутизации предназначена для IP *128.17.75.20* и маршрутизируется на интерфейс lo, таким образом, если машина подключается к самой себе на *128.17.75.20*, все пакеты будут отправлены на интерфейс *127.0.0.1*.


Если хост eggplant хочет отправить пакет хосту zucchini, (пакет будет содержать отправителя -*128.17.75.20* и получателя - *128.17.75.37*), протокол IP определит на основании таблицы маршрутизации, что оба хоста принадлежат одной сети и отправит пакет напрямую в сеть, где получит его zucchini.
Для более точного описания, сетевой интерфейс контроллера будет передавать **ARP** запрос - "Кто IP *128.17.75.37*? Это *128.17.75.20* кричит".

Все машины, получившие это сообщение, игнорируют его, и хост с адресом *128.17.75.37* отвечает "Это я и мой MAC-адрес такой-то...", затем они соединяются и обмениваются данными на основе таблиц **ARP**, сопоставляющих IP-MAC-адреса. "Крики" означают, что этот пакет отправляется всем хостам в сети. Это происходит потому, что MAC-адрес получателя указан как широковещательный адрес (*FF:FF:FF:FF:FF:FF:FF*).
Такие пакеты принимаются всеми хостами в сети.

Давайте рассмотрим ситуацию, где хост eggplant хочет отправить пакет хосту pear, например, или даже еще дальше?
В этом случае адрес назначения пакета будет *128.17.112.21*, протокол IP попытается найти маршрут для *128.17.112* в таблице маршрутизации, но такого маршрута там нет, поэтому выберет маршрут по умолчанию, шлюзом которого является papaya (*128.17.75.98*).

Получив пакет, papaya найдет адрес назначения в своей таблице маршрутизации:
```
[root@papaya ~]# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
128.17.75.0      128.17.75.98   255.255.255.0   UN        1500 0          0 eth0
128.17.112.0     128.17.112.3   255.255.255.0   UN        1500 0          0 eth1
default          128.17.112.40  0.0.0.0         UGN       1500 0          0 eth1
127.0.0.1        127.0.0.1      255.0.0.0       UH        3584 0          0 lo
128.17.75.98     127.0.0.1      255.255.255.0   UH        3584 0          0 lo
128.17.112.3     127.0.0.1      255.255.255.0   UH        3584 0          0 lo
```


В этом примере видно, что papaya подключена к двум сетям: *128.17.75.0/24* через устройство eth0 и *128.17.112* через устройство eth1.

Маршрут по умолчанию проходит через хост pineapple, который в свою очередь является шлюзом к внешней сети.

Поэтому, получив пакет для pear, маршрутизатор papaya увидит, что адрес назначения принадлежит *128.17.112* и перенаправит пакет в соответствии со второй записью в таблице маршрутизации.

Таким образом пакеты передаются от маршрутизатора к маршрутизатору, пока они не достигнут своего адреса назначения.

## **Протокол ICMP**

Протокол **ICMP** - это протокол уведомления об ошибках.

В стеке TCP/IP есть специальный механизм сообщений, который позволяет маршрутизаторам уведомлять узлы сети об ошибках или аномальных ситуациях, называемый Протокол управления интернетом (**ICMP**).