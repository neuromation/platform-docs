# Устранение ошибок

## Утерянный/неизвестный ID задания

Прежде чем расследовать любую проблему, нужно знать ID задания. Если вы запускали задание через `neuro run`, его ID было показано в выводе этой команды. 

Если вы не можете найти изначальный вывод, используйте одну из этих команд, чтобы найти конкретное задание:

{% tabs %}
{% tab title="задания neuro run" %}
`neuro ps` выводит список запущенных заданий.   
`neuro ps -a` выводит список всех заданий.  
`neuro ps -s failed` выводит список заданий со статусом Failed.
{% endtab %}

{% tab title="задания neuro-flow" %}
`neuro-flow ps` выводит список всех заданий.
{% endtab %}
{% endtabs %}

## Провалена сборка образа

Когда вы запускаете `neuro-flow build ИМЯ_ОБРАЗА`, neuro-flow загружает контекст сборки на платформу и создаёт задание, использующее [Kaniko](https://github.com/GoogleContainerTools/kaniko) для сборки docker-образа и выгрузки его на реестр платформы.

Если сборка не удаётся, вы можете проверить статус задания и его логи для получения более полной информации. 

#### Получения статуса задания

Чтобы проверить статус задания, запустите: 

```text
$ neuro status <ID-задания> 
```

Секция **Status transitions** в выводе этой команды может показать вам, на каком шагу провалилась сборка. 

#### Получение логов сборщика

Чтобы проверить логи сборщика, запустите:

```text
$ neuro logs <ID-задания> 
```

## Провален neuro run / neuro-flow run

Есть несколько основных причин, по которым может не удасться запуск задания. Вот самые частые из них:

#### Неверное имя образа

Такое может случиться, если в названии образа есть опечатка или указанный образ не был предварительно собран. Список всех образов можно получить, запуская `neuro image ls`. Вы также можете посмотреть список тегов конкретного образа с помощью команды `neuro image tags <URI_ОБРАЗА>`.

#### Примонтирован неверный том

К заданию может быть примонтирован неверный том. Например, Например, вы примонтировали том к папке `/my-project`, но ваш код ожидает папку `/my_project`. Вы можете проверить это в логах.

#### Не удалось масштабировать кластер

Если вы видите ошибку **Cluster Scale Up Failed** в статусе, скорее всего вы запросили ресурсы, недоступные в кластере на данный момент. Например, все GPU заняты и ваше задание не модет быть запланировано.

#### Проблемы с кодом

В вашем python-скрипте могут быть ошибки, не дающие заданию запуститься должным образом.

## Невозможно доступиться до задания по HTTP

Есть несколько шагов, помогающих в таких случаях.

#### Проверка наличия открытого HTTP-порта

Первое, что стоит проверить - наличие открытого HTTP-порта у вашего задания. Чтобы сделать это: 

{% tabs %}
{% tab title="Задания neuro run" %}
Используйте параметр `--http_port`.
{% endtab %}

{% tab title="Задания neuro-flow" %}
Используйте опцию `http_port:`.
{% endtab %}
{% endtabs %}

#### Проверка IP прослушивания

Следующим шагом нужно удостовериться, что ваше веб-приложение слушает адрес 0.0.0.0, а не 127.0.0.1 или `localhost` — в противном случае оно не сможет получать входящие запросы извне контейнера.

#### Отключение HTTP-аутентификации

Если вы можете доступиться к заданию через браузер, но `curl` и подобные инструменты не работают, скорее всего вы не отключили HTTP-аутентификацию. Платформа по умолчанию создаёт слой HTTP-аутентификации перед вашим веб-приложением из соображений безопасности. 

Вы можете вручную отключать это поведение при запуске заданий:

{% tabs %}
{% tab title="Задания neuro run" %}
Используйте параметр `--no-http-auth`.
{% endtab %}

{% tab title="Задания neuro-flow" %}
Используйте опцию `http_auth: False`.
{% endtab %}
{% endtabs %}

## Исправление неполадок в запущенном задании

Также как и в Docker, вы можете подключиться через shell к запущенному заданию для проверки его статуса. Для этого запустите:

```text
$ neuro exec JOB_ID /bin/sh
```

В Docker к командам обычно добавляются параметры `-it`, но они необязательны для `neuro exec`.
