# Шаблоны
### Были созданы следующие шаблоны, каждый из которых соответствует представлению:

## Общие шаблоны:
- Базовая страница с хедером и подключением стилей
- Главная страница со списком отелей


## Реализация

## Базовый шаблон
Шаблон `base.html` используется как основа для всех страниц проекта. Он включает:
- Подключение стилей и библиотек (Bootstrap 5.3, Flatpickr).
- Хедер с логотипом, навигацией и пользовательскими кнопками (логин, регистрация, аккаунт, выход).
- Основной контейнер для вставки содержимого страниц через блоки `{% block title %}` и `{% block main_contents %}`.

### Код:
```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}Base page{% endblock %}</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet"
          integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
    {% load static %}
    <style>
        a:link, a:visited {
            text-decoration: inherit;
            color: inherit;
            cursor: auto;
        }
    </style>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/flatpickr/dist/flatpickr.min.css">
</head>
<body>
<div class="container">
    <header class="d-flex flex-wrap align-items-center justify-content-center justify-content-md-between py-3 mb-4 border-bottom">
        <div class="col-md-3 mb-2 mb-md-0">
            <a href="/" class="d-inline-flex link-body-emphasis text-decoration-none">
                <!-- SVG Логотип -->
            </a>
        </div>
        <ul class="nav col-12 col-md-auto mb-2 justify-content-center mb-md-0">
            <li><a href="/" class="nav-link px-2 link-secondary">Home</a></li>
            <li><a href="/hotels/" class="nav-link px-2">Hotels</a></li>
            <li><a href="/bookings/" class="nav-link px-2">Your bookings</a></li>
            <li><a href="/admin/" class="nav-link px-2">Admin</a></li>
        </ul>
        <div class="col-md-3 text-end">
            {% if not user.is_authenticated %}
                <button type="button" class="btn btn-outline-primary me-2"><a href="/login/" class="nav-link px-2">Login</a></button>
                <button type="button" class="btn btn-primary"><a href="/register/" class="nav-link px-2" style="color: white">Sign-up</a></button>
            {% else %}
                <button type="button" class="btn btn-outline-primary me-2"><a href="/bookings/" class="nav-link px-2">Your account</a></button>
                <button type="button" class="btn btn-primary"><a href="/logout/" class="nav-link px-2" style="color: white">Logout</a></button>
            {% endif %}
        </div>
    </header>
</div>
<div class="container">
    {% block main_contents %}
        <h1>Hello, world!</h1>
        <p>You should never see that</p>
    {% endblock %}
</div>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"
        integrity="sha384-YvpcrYf0tY3lHB60NNkmXc5s9fDVZLESaAA55NDzOxhy9GkcIdslK1eN7N6jIeHz"
        crossorigin="anonymous"></script>
<script>
    document.addEventListener('DOMContentLoaded', function () {
        flatpickr(".flatpickr", {
            dateFormat: "Y-m-d",
            minDate: "today",
        });
    });
</script>
</body>
</html>
```


## Навигационное меню
Навигация в шаблоне выполнена с помощью Bootstrap. Она содержит ссылки на главную страницу, список отелей, ваши бронирования и страницу администратора. В зависимости от статуса пользователя (`is_authenticated`), в хедере отображаются кнопки:
- Для незарегистрированных пользователей: "Login" и "Sign-up".
- Для зарегистрированных пользователей: "Your account" и "Logout".


## Список отелей
Шаблон отображает список отелей в виде интерактивных карточек. Каждый элемент содержит:
- Название отеля.
- Описание.
- Владелец.
- Адрес.
- Среднюю оценку.

### Пример:
```html
<ul>
    {% for hotel, rating in hotel_ratings.items %}
        <li>
            <a href="/hotels/{{ hotel.id }}/" style="color: black">
                <div class="container">
                    <div class="row">
                        <div class="col">
                            <div class="h4">{{ hotel.name }}</div>
                            <p class="h5">Description</p>
                            <p>{{ hotel.description }}</p>
                        </div>
                        <div class="col">
                            <div class="h5">Owner</div>
                            <p>{{ hotel.owner.username }}</p>
                            <p>Address: {{ hotel.address }}</p>
                            <p>Average rating: {{ rating }}</p>
                        </div>
                    </div>
                </div>
            </a>
            <hr>
        </li>
    {% endfor %}
</ul>
```
