
# Отчет по модели системы бронирования отелей

## Описание системы
Система бронирования отелей предоставляет пользователям возможность резервировать номера, оставлять отзывы и управлять процессами бронирования. Она ориентирована как на пользователей (гостей), так и на владельцев отелей, предлагая удобный интерфейс для управления данными об отелях, номерах и бронированиях.

### Основные возможности системы:
- Регистрация и управление отелями владельцами.
- Создание и управление номерами, включая их описание и оснащение.
- Бронирование номеров пользователями, с поддержкой статусов бронирования (создано, одобрено, отменено и др.).
- Возможность оставления отзывов на основе завершенных бронирований.
- Гибкая структура номеров с учетом их типа и набора удобств.


## Описание моделей

### RoomType
**Назначение:** Представляет тип номера (например, одноместный, люкс и т. д.).  
**Атрибуты:**
- `room_type` (строка) — тип номера.

```python
class RoomType(models.Model):
    room_type = models.CharField(max_length=120)

    def __str__(self):
        return self.room_type
```


### Convenience
**Назначение:** Представляет удобства, доступные в номерах (например, Wi-Fi, кондиционер).  
**Атрибуты:**
- `convenience` (строка) — описание удобства.

```python
class Convenience(models.Model):
    convenience = models.CharField(max_length=120)

    def __str__(self):
        return self.convenience
```


### Hotel
**Назначение:** Хранит информацию об отелях, добавленных владельцами.  
**Атрибуты:**
- `name` (строка) — название отеля.
- `description` (текст) — описание отеля.
- `address` (строка) — адрес отеля.
- `owner` (внешний ключ) — ссылка на пользователя (владельца отеля).

```python
class Hotel(models.Model):
    name = models.CharField(max_length=120)
    description = models.TextField()
    address = models.CharField(max_length=120)
    owner = models.ForeignKey(User, on_delete=models.CASCADE)

    def __str__(self):
        return self.name
```


### Room
**Назначение:** Представляет номера в отелях.  
**Атрибуты:**
- `hotel` (внешний ключ) — отель, к которому принадлежит номер.
- `n_people` (число) — вместимость номера.
- `cost` (положительное число) — стоимость номера за день.
- `room_type` (внешний ключ) — тип номера.
- `conveniences` (связь многие-ко-многим) — список удобств номера.

```python
class Room(models.Model):
    hotel = models.ForeignKey(Hotel, on_delete=models.CASCADE)
    n_people = models.IntegerField()
    cost = models.PositiveIntegerField()
    room_type = models.ForeignKey(RoomType, on_delete=models.CASCADE)
    conveniences = models.ManyToManyField(Convenience)

    def __str__(self):
        return f'room {self.id} in {self.hotel.name}'
```


### Reservation
**Назначение:** Описывает бронирования номеров.  
**Атрибуты:**
- `user` (внешний ключ) — пользователь, который сделал бронирование.
- `room` (внешний ключ) — номер, который бронируется.
- `dt_start` (дата) — начало бронирования.
- `dt_end` (дата) — конец бронирования.
- `status` (строка) — текущий статус бронирования (создано, одобрено, отменено и т. д.).

```python
class Reservation(models.Model):
    RESERVATION_TYPES = (
        ('cr', 'Created'),
        ('ap', 'Approved'),
        ('ca', 'Canceled'),
        ('ci', 'Checked in'),
        ('co', 'Checked out'),
    )
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    room = models.ForeignKey(Room, on_delete=models.CASCADE)
    dt_start = models.DateField()
    dt_end = models.DateField()
    status = models.CharField(max_length=15, choices=RESERVATION_TYPES, default='cr')
```


### Review
**Назначение:** Позволяет пользователям оставлять отзывы после завершенного бронирования.  
**Атрибуты:**
- `reservation` (связь один-к-одному) — бронирование, для которого оставлен отзыв.
- `author` (внешний ключ) — автор отзыва.
- `text` (текст) — содержание отзыва.
- `rating` (положительное число) — оценка от 1 до 10.

```python
class Review(models.Model):
    reservation = models.OneToOneField(Reservation, on_delete=models.CASCADE)
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    text = models.TextField()
    rating = models.PositiveIntegerField(
        validators=[
            MinValueValidator(1),
            MaxValueValidator(10)
        ]
    )

    def __str__(self):
        return f"{self.student} - {self.homework}"
```


## Вывод
Система бронирования отелей предоставляет все необходимые инструменты для управления номерами, бронированиями и отзывами. Структура моделей поддерживает гибкое добавление новых данных и логически соответствует основным задачам системы.
