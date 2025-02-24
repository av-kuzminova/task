# Django-проект: TaskManagers

## Введение

Цель проекта — разработать приложение **TaskManager** для управления задачами. Основной функционал включает:

- Управление задачами (создание, редактирование, удаление)
- Управление статусами задач (создание, редактирование, удаление)
- Организация задач по проектам (создание, редактирование, удаление)
- Разделение задач по спринтам внутри проектов (создание, редактирование, удаление)
- Просмотр истории изменений задач
- Уведомление исполнителя о смене статуса задачи

## Реализация

### Используемые технологии

- **Python 3.10** — язык программирования.
- **Django 4.1** — веб-фреймворк.
- **PostgreSQL** — база данных.
- **Django REST framework** — для создания API.
- Дополнительные зависимости перечислены в [requirements.txt](requirements.txt).

### Описание проекта

#### Общая структура

Проект состоит из трёх основных сущностей: **проект**, **спринт** и **задача**. Спринт всегда принадлежит проекту, а задача может принадлежать как проекту, так и спринту.

Все пользователи могут просматривать данные через интерфейс или API. Авторизованные пользователи с соответствующими правами могут создавать новые записи. Редактировать и удалять записи может только автор. Исполнитель задачи может изменять дату её завершения.

При изменении проекта, спринта или задачи, информация об изменениях записывается в историю зависимых задач, а также отправляются уведомления по email авторам и исполнителям.

По умолчанию, в интерфейсе отображается 5 записей на страницу, а в API — 10.

На главной странице отображается список незавершённых задач.

#### Проекты

В разделе **"Проекты"** отображается список незавершённых проектов. Фильтрация доступна по статусам: *"Весь список"*, *"Выполнено"*, *"В работе"*. Авторизованные пользователи могут дополнительно фильтровать проекты по автору или отсутствию отчёта. В зависимости от прав пользователя, отображаются кнопки *"Добавить"*, *"Редактировать"* и *"Удалить"*.

При выборе проекта отображается детальная информация, включая списки спринтов и задач. Проект нельзя закрыть, пока есть незавершённые спринты или задачи. Минимальная дата закрытия рассчитывается на основе дат создания и закрытия спринтов и задач.

Удаление проекта приводит к удалению всех связанных спринтов и задач.

#### Спринты

В разделе **"Спринты"** отображается список незавершённых спринтов. При выборе спринта открывается страница с детальной информацией. Спринт нельзя закрыть, пока есть незавершённые задачи. Минимальная дата закрытия рассчитывается на основе дат создания и закрытия задач.

Планируемая дата спринта определяется как минимальная из планируемых дат проекта и спринта.

Удаление спринта приводит к удалению всех связанных задач.

При редактировании спринта проект выбирается из списка незакрытых проектов. При создании спринта со страницы проекта, проект автоматически подставляется.

#### Задачи

В разделе **"Задачи"** отображается список незавершённых задач. Фильтрация доступна по статусу и исполнителю. На странице детальной информации отображается история изменений задачи, а также история изменений в родительской задаче, спринте и проекте.

Задачи могут быть зависимыми, но зависимость одноуровневая: задача с зависимыми задачами не может быть зависимой сама.

Планируемая дата задачи определяется как минимальная из планируемых дат проекта, спринта и задачи.

При редактировании задачи проект выбирается из списка незакрытых проектов, а спринт — из списка незакрытых спринтов проекта. Родительская задача выбирается из списка задач спринта или проекта.

Исполнитель может изменять только дату завершения задачи.

#### Поиск

В разделе **"Поиск"** отображаются списки незавершённых проектов, спринтов и задач. Доступна фильтрация по статусу и автору. Поиск не зависит от регистра.

#### API

В разделе **"API"** доступен веб-интерфейс Django REST framework. Фильтрация возможна по точному совпадению значений. Например:  
`/api/tasks/?date_end=none&date_beg=2023-06-10`  
Создание и редактирование записей через API недоступно.

#### Авторизация

Неавторизованные пользователи видят кнопку *"Вход"*. После успешной авторизации пользователь перенаправляется на главную страницу. Авторизованные пользователи видят своё имя и кнопки *"Смена"* и *"Выход"*.

### Особенности

#### Тестирование

Тестирование включает два блока:

1. Генерация случайных данных и проверка корректности их записи в базу.
2. Имитация открытия страниц и проверка корректности отображения данных.

Результаты тестирования:

```text
Name  Stmts   Miss  Cover  
TOTAL 1318    364    72%
```

#### Генерация данных

Для генерации данных можно использовать команду:  
`python manage.py shell -c="from app_task.functions import gen_data; gen_data(cnt=50, close=60, clear=True, parent=True, clear_user=True)"`

Параметры функции `gen_data`:

- `cnt=0` — количество проектов
- `close=0` — процент закрытия проектов и спринтов
- `clear=False` — очистка базы перед генерацией
- `parent=False` — создание зависимых задач
- `clear_user=False` — удаление тестовых пользователей

Даты выбираются из диапазона ±3 месяца от текущей даты. Тестовые пользователи создаются с шаблоном имени из настроек `MY_TEST_USER` и паролем из `MY_TEST_PASS`.

Этапы генерации данных:

1. Очистка базы (если указано).
2. Удаление тестовых пользователей (если указано).
3. Создание тестовых пользователей (если необходимо).
4. Создание проектов, спринтов и задач.
5. Создание зависимых задач (если указано).
6. Закрытие спринтов и проектов (если указано).

Данные сохраняются с проверкой корректности: даты, зависимости, возможность закрытия и т.д.