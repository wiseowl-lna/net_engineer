##  **Развертывание коммутируемой сети с резервными каналами**

  ###  Схема подключения:

Рис.1

![](Topology.png)

  ### Таблица адресации:
Табица 1
|  Device  |  Interface  |   IP Address   |   Subnet Mask   |
|----------|-------------|----------------|-----------------|
| S1       | VLAN 1      | 192.168.1.1    | 255.255.255.0   |
| S2       | VLAN 1      | 192.168.1.2    | 255.255.255.0   |
| S3       | VLAN 1      | 192.168.1.2    | 255.255.255.0   |

### Задание:
  Необходимо настроить оборудование и организовать сетевую связанность согласно условию лабораторной работы.
  1. Создать сеть и настроить основные параметры устройства.
  2. Необходимо выбрать корневой мост.
  3. Пронаблюдать за процессом выбора протоколом STP порта, исходя из стоимости портов.
  4. Пронаблюдать за процессом выбора протоколом STP порта, исходя из приоритета портов.

 ### Ход выполнения:
  Для выполнения лабораторной работы использовался эмулятор Сisco Packet Tracer.
 
 #### **_I. Создание сети и настройка основных параметров устройства._**
 
 #### Сбор схемы:
  1. Подключила устройства, как показано на рисунке 1.

#### Инициализация коммутаторов:
1. Подключилась консолью к коммутатору, вошла в привилигированный режим.
2. С помощью команды **_show flash_** проверила есть ли на коммутаторе ранее созданные сати VLAN. Файла **vlan.dat** не оказалось. В противном случае, если этот файл был бы обнаружен во флеш-памяти, его нужно было бы удалить с помощью команды **_delete vlan.dat_**.
3. Следующим этапом необходимо удалить файл загрузочной конфигурации из NVRAM. Команда: **_erase startup-config_**.
4. Перезагрузила коммутатор - команда **_reload_**.
5. Данную процедуру провела на всех используемых коммутаторах.

#### Настройка базовых параметров коммутатора:
Базовые настройки коммутаторов находятся в в папке [configs](configs/) в файлах **OP_S1.txt**, **OP_S2.txt**, **OP_S3.txt** соответственно.

#### Проверка связности между коммутаторами:
1. Выполнила команду ping с коммутатора S1 на коммутатор S2. Результат положительный.
2. Выполнила команду ping с коммутатора S1 на коммутатор S3. Результат положительный.

Рис.2

![](Ping_from_S1_to_S2_and_S3.png)

3. Выполнила команду ping с коммутатора S2 на коммутатор S3. Результат положительный.

Рис.3

![](Ping_from_S2_to_S3.png)

    Данное кол-во тестов достаточно, так как команда ping проверяет доступност в обоих направлениях.


#### **_II. Определение корневого моста._**

1. Отключила все порты на коммутаторе S1.

       conf t
       !
       interface range f0/1-24, g0/1-2
        shutdown
        exit
       exit
       !
Данные действия проделала и на коммутаторах S2 и S3.

2. Настроила подключенные порты (см.Рис.1) как транковые.
3. Включила порты F0/2 и F0/4 на всех коммутаторах.

       conf t
       !
       interface range f0/2, F0/4
        switchport mode trunk
        no shutdown
        exit
       exit
       !

4. Ввела команду **_show spanning-tree_** на всех трех коммутаторах.

Рис.4

![](PIM_S1.png)

Рис.5

![](PIM_S2.png)

Рис.6

![](PIM_S3.png)

    Проанализировав выходные данные с коммутаторов получила следующую картину:


Рис.7

![](Rol_switch.png) 

     А так же получила ответы на ряд вопросов:

*Вопрос_1:*
Какой коммутатор является корневым мостом?

*Ответ:*
Коммутатор S3.

*Вопрос_2:*
Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?

*Ответ:*
Так как выбор корневого моста складывается из значений приоритета (по умолчанию 32768), расширенного идентификатора системы - VLAN (в моем случае – это vlan 1 на всех коммутаторах) и MAC-адреса коммутатора (уникальные для каждого коммутатора), а коммутатор с самым низким значением MAC-адреса, при равных остальных условиях, становится корневым мостом, то следовательно, в моем случае – это коммутатор S3 (см. Рис.7)

*Вопрос_3:*
Какие порты на коммутаторе являются корневыми портами?

*Ответ:*
Порт коммутатора, который имеет кратчайший путь к корневому коммутатору называется корневым портом. У любого не корневого коммутатора может быть только один корневой порт. В моем случае – это порт F0/4 на коммутаторе S1 и порт F0/4 на коммутаторе S2.

*Вопрос_4:*
Какие порты на коммутаторе являются назначенными портами?

*Ответ:*
Коммутатор в сегменте сети, имеющий наименьшее расстояние до корневого коммутатора называется назначенным коммутатором (мостом). Порт этого коммутатора, который подключен к рассматриваемому сегменту сети, называется назначенным портом. В моем случае – это порт F0/2 на коммутаторе S2.

*Вопрос_5:*
Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?

*Ответ:*
В моем случае – это порт F0/2 на коммутаторе S1 (см.Рис.8).

Рис.8

![](Trafic.png)

*Вопрос_6:*
Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?

*Ответ:*
Порт коммутатора, имеющего самое наибольшее расстояние до корневого коммутатора,  и, к тому же, который подключен к назначенному коммутатору называется альтернативным и блокируется до тех пор, пока основной путь не будет разорван.

В результате работы протокола spanning-tree получается древовидная структура (математический граф) с вершиной в виде корневого коммутатора.

Рис.9

![](Graf_1.png)


#### **_III. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов._**
Итак, на этом этапе лабораторной работы необходимо проследить как протокол spanning-tree поведет себя при изменении приоритетов портов.

1. По ходу лабораторной работы, с помощью команды **_show spanning-tree_**, определила коммутатор на котором оказался заблокированный протоколом порт - это коммутатор S1 (рис.4) порт F0/2. Хоть порт F0/4 и имеет более высокий идентификатор, но он смотрит в сторону корневого моста, по этому протокол заблокировал порт F0/2.

2. Изменила стоимость, оставшегося не заблокированным, порта на коммутаторе S1, понизив его на единицу.

       conf t
        interface f0/4
        spanning-tree vlan 1 cost 18
        exit
       exit
       !
3. В течении 30 секунд увидела изменение картины:

Рис.10

![](Trafic_s_izmeneniem_cost_18.png)

4. Повторно запустила команду show **_spanning-tree_** на коммутаторах S1 и S2.

Рис.11

![](PIM_S1_18.png)

Рис.12

![](PIM_S2_18.png)

*Вопрос:* Почему протокол spanning-tree заменяет ранее заблокированный порт на назначенный порт и блокирует порт, который был назначенным портом на другом коммутаторе?

*Ответ:* Алгоритм протокола spanning-tree (STA) использует корневой мост как точку привязки, после чего определяет, какие порты будут заблокированы, исходя из стоимости пути. Порт с более низкой стоимостью пути является предпочтительным, По-этому протокол заблокировал порт со стоимостью 19 на коммутаторе S2.

5. Вернула стоимость порта F0/4 в первоначальное состояние.

       conf t
        interface f0/4
        no spanning-tree vlan 1 cost 18
        exit
       exit
       !

#### **_IV.	Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов._**

1. С помощью команды **_no shutdown_** включила порты F0/1 и F0/3.
2. Через 30 секунд, после того как протокол STPзавершит процесс перевода порта, мы видим, что порт корневого моста переместился на порт с меньшим номером, связанный с коммутатором корневого моста, и заблокировал предыдущий порт корневого моста.

Рис.13

![](Trafic_no_sh_1_3.png)

Так же это можно увидеть с помощью команды **_show spanning-tree_**, запущенной на некорневых коммутаторах.

Рис.13

![](PIM_S1_no_sh_1_3.png)

Рис.13

![](PIM_S2_no_sh_1_3.png)

*Вопрос:* Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста?

*Ответ:* На коммутаторе S1 - это порт F0/3, на коммутаторе S2 - порт F0/3.

*Вопрос:* Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?

*Ответ:* Потому, что это порты с меньшим номером, что является более приоритетным.

#### **_Вопросы для повторения._**

*Вопрос_1:* Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта?

*Ответ:* Этим значением является стоимость порта (cost).

*Вопрос_2:* Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта?

*Ответ:* Далее протокол проверяет приоритет порта (по умолчанию - 128).

*Вопрос_3:* Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта?

*Ответ:* Номар порта.

**_В итоге получаем:_**

Выбор корневого моста складывается из:

             BID = Pm + Vlan + MAC, где

       BID - идентификатора моста,
       Pm - приоритет моста (значение приоритета может находиться в диапазоне от 0 до 65535 с шагом 4096. По умолчанию используется значение 32768),
       Vlan - номер сети VLAN,
       MAC - mac-адрес коммутатора.

Коммутатор с наименьшим BID становится корневым.


Выбор корневого порта складывается из:

              BID = Cost + Prio + Np, где

       Cost - стоимость порта,
       Prio - приоритет порта (по умолчанию - 128),
       Np - номар порта.

Порт с наименьшим BID становится корневым.
