## Домашнее задание к занятию "10.02. Системы мониторинга"

### Обязательные задания

>1. Опишите основные плюсы и минусы pull и push систем мониторинга.

`Ответ: `

Push-модель – когда сервер мониторинга ожидает подключений от агентов для получения метрик;

Pull-модель – когда сервер мониторинга сам подключается к агентам мониторинга и забирает данные;


**PUSH** 

*Плюсы:*
* Упрощение репликации данных в разные системы мониторинга или их резервные копии
* Более гибкая настройка отправки пакетов данных с метриками, удобна для использования в динамически создаваемых машинах
* UDP является менее затратным способом передачи данных, следовательно может вырости производительность сбора метрик

*Минусы:*
* Безопасность: передача данных в открытом виде по сети, есть риск утечки данных.
* Отсутствие гарантии доставки пакетов. Есть риск потери данных при недоступности системы мониторинга

**PULL** 

*Плюсы:*
* Легче контролировать подлинность данных (точно знаем откуда и что берем)
* Безопасность (единый proxy-server до всех агентов с TLS)
* Упрощенная отладка получения данных с агентов 

*Минусы:*
* неудобство для динамических машин (докер-контейнеры) нужно постоянно иметь актуальную информацию о инфрастуктуре.
   
---


>2. Какие из ниже перечисленных систем относятся к push модели, а какие к pull? А может есть >гибридные?
>
>* Prometheus
>* TICK
>* Zabbix
>* VictoriaMetrics
>* Nagios

`Ответ: `
| Система  | Тип  | 
|---|---|
| Prometheus  | PULL, но также похоже на PUSH, т.к. есть `node_exporter` — агент, который может быть установлен на другой сервер  | 
|  TICK |  PUSH : telegraph передает информацию в хранилище, так же данные получает Kapasitor по Pull модели |
| Zabbix  | PUSH : нужно настраивать агенты для получения метрик  |
| VictoriaMetrics  |  похоже на PUSH. Используется в инфраструктуре для долгого хранения метрик . Как я понял агенты, передают свои метрики в хранилище |
| Nagios  | PULL : Так же использует опрос SNMP-агентов, которые собирают информацию  |

------

3. Склонируйте себе репозиторий и запустите TICK-стэк, используя технологии docker и docker-compose.

В виде решения на это упражнение приведите выводы команд с вашего компьютера (виртуальной машины):

- curl http://localhost:8086/ping
- curl http://localhost:8888
- curl http://localhost:9092/kapacitor/v1/ping

А также скриншот веб-интерфейса ПО chronograf (http://localhost:8888).

P.S.: если при запуске некоторые контейнеры будут падать с ошибкой - проставьте им режим Z, например `./data:/var/lib:Z`

`Ответ: `
```
romrsch@ubuntu:~/netology/10.2$ git clone https://github.com/influxdata/sandbox.git
Cloning into 'sandbox'...
remote: Enumerating objects: 1718, done.
remote: Counting objects: 100% (32/32), done.
remote: Compressing objects: 100% (32/32), done.
remote: Total 1718 (delta 13), reused 1 (delta 0), pack-reused 1686
Receiving objects: 100% (1718/1718), 7.17 MiB | 1.18 MiB/s, done.
Resolving deltas: 100% (946/946), done.
romrsch@ubuntu:~/netology/10.2$ 
```
Запуск
![alt](https://i.ibb.co/W2YJj21/Screenshot-2.jpg)

Выводы команд:
![alt](https://i.ibb.co/Lx5d9vh/Screenshot-3.jpg)

![alt](https://i.ibb.co/z4rgx1J/Screenshot-4.jpg)

![alt](https://i.ibb.co/KbqFvtJ/Screenshot-5.jpg)

Скриншот интрефейса:
![alt](https://i.ibb.co/4ZgG7BR/Screenshot-6.jpg)

-----


4. Перейдите в веб-интерфейс Chronograf (http://localhost:8888) и откройте вкладку `Data explorer`.

* Нажмите на кнопку `Add a query`
* Изучите вывод интерфейса и выберите БД `telegraf.autogen`
* В measurments выберите mem->host->telegraf_container_id , а в `fields` выберите `used_percent`. Внизу появится график утилизации оперативной памяти в контейнере telegraf.
* Вверху вы можете увидеть запрос, аналогичный SQL-синтаксису. Поэкспериментируйте с запросом, попробуйте изменить группировку и интервал наблюдений.

Для выполнения задания приведите скриншот с отображением метрик утилизации места на диске (disk->host->telegraf_container_id) из веб-интерфейса.

`Ответ: `

![alt](https://i.ibb.co/r4b8GMD/Screenshot-7.jpg)

----

5. Изучите список `telegraf inputs`. Добавьте в конфигурацию telegraf следующий плагин - docker:

```
[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
```
Дополнительно вам может потребоваться донастройка контейнера telegraf в docker-compose.yml дополнительного volume и режима privileged:
```
  telegraf:
    image: telegraf:1.4.0
    privileged: true
    volumes:
      - ./etc/telegraf.conf:/etc/telegraf/telegraf.conf:Z
      - /var/run/docker.sock:/var/run/docker.sock:Z
    links:
      - influxdb
    ports:
      - "8092:8092/udp"
      - "8094:8094"
      - "8125:8125/udp"

```
`Ответ: `

Добавление плагина Docker

![alt](https://i.ibb.co/0MqX3j3/Screenshot-8.jpg)

![alt](https://i.ibb.co/rmyBcV3/Screenshot-9.jpg)



После настройки перезапустите `telegraf`, обновите веб интерфейс и приведите скриншотом список `measurments` в веб-интерфейсе базы `telegraf.autogen` . Там должны появиться метрики, связанные с docker.

Факультативно можете изучить какие метрики собирает telegraf после выполнения данного задания.