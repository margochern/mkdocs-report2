# Список отелей: поиск и пагинация

### Реализация
На странице со списком отелей были реализованы функции поиска и пагинации.


## Поиск
Поиск реализован через GET-запрос. Пользователь может ввести название отеля или имя владельца в поле поиска, после чего список отелей фильтруется в соответствии с запросом.  
Поиск осуществляется по следующим полям:
- Название отеля (`name`).
- Имя владельца (`owner__username`).

Пример кода для обработки поиска:
```python
query = request.GET.get('q')
hotels = Hotel.objects.all()
if query:
    hotels = hotels.filter(
        Q(name__icontains=query) | Q(owner__username__icontains=query)
    )
```


## Пагинация
На одной странице отображается ограниченное количество отелей (например, 3). Пользователь может перемещаться между страницами с помощью навигационных ссылок: "first", "previous", "next", "last".

Пример кода для обработки пагинации:
```python
paginator = Paginator(hotels, 3)  # Отображать по 3 отеля на странице
page_number = request.GET.get('page')
page_obj = paginator.get_page(page_number)
context = {
    'page_obj': page_obj,
    'hotels': page_obj,
    'query': query,
}
return render(request, 'hotel/hotel_list.html', context)
```


## Шаблон `hotel_list.html`

Шаблон страницы отображает список отелей и управляет пагинацией. Каждый элемент списка содержит:
- Название отеля.
- Описание.
- Имя владельца.
- Адрес.
- Средний рейтинг.

Код шаблона:
```html
{% extends 'base.html' %}

{% block title %}Hotel List{% endblock %}

{% block main_contents %}

<h1>Hotels available</h1>
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
<div class="pagination">
    <span class="step-links">
        {% if page_obj.has_previous %}
            <a href="?page=1">&laquo; first</a>
            <a href="?page={{ page_obj.previous_page_number }}">previous</a>
        {% endif %}

        <span class="current">
            Page {{ page_obj.number }} of {{ page_obj.paginator.num_pages }}.
        </span>

        {% if page_obj.has_next %}
            <a href="?page={{ page_obj.next_page_number }}">next</a>
            <a href="?page={{ page_obj.paginator.num_pages }}">last &raquo;</a>
        {% endif %}
    </span>
</div>

{% endblock %}
```


## Результат
Реализованные функции позволяют пользователю:
1. Найти отели по ключевым словам, введенным в строку поиска.
2. Удобно перемещаться по списку отелей с использованием пагинации.