
# Представления

Все представления были реализованы как в функциональном стиле, так и с использованием классов. Они обеспечивают полный функционал для работы с пользователями, бронированиями, отелями и отзывами.

## Общие контроллеры

### Главная страница
Отображает приветственное сообщение с именем пользователя или "анонимный", если пользователь не авторизован.
```python
def IndexView(request):
    return render(request, 'index.html', context={'username': request.user.username or 'anonymous'})
```

### Выход из аккаунта
Пользователь выходит из аккаунта, после чего перенаправляется на главную страницу.
```python
def logout_view(request):
    logout(request)
    return redirect('/')
```

### Тестовый вид
Тестовый рендеринг базового шаблона.
```python
def test_view(request):
    return render(request, 'base.html')
```


## Пользователи

### Регистрация
Регистрация пользователя через форму. После успешной регистрации пользователь перенаправляется на страницу входа.
```python
class RegisterView(View):
    def get(self, request):
        form = CustomUserCreationForm()
        return render(request, 'register.html', {'form': form})

    def post(self, request):
        form = CustomUserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('/login/')
        return render(request, 'register.html', {'form': form})
```


## Отели и комнаты

### Список отелей
Выводит список отелей с пагинацией (10 отелей на страницу). Включает средний рейтинг отеля.
```python
class HotelListView(ListView):
    model = Hotel
    template_name = 'hotels/list.html'
    context_object_name = 'hotels_list'
    paginate_by = 10

    def get_context_data(self, *args, **kwargs):
        context = super().get_context_data(*args, **kwargs)
        context.update({
            "hotel_ratings": {
                h: Review.objects.filter(reservation__room__hotel=h).aggregate(Avg('rating'))['rating__avg'] or 'No data'
                for h in context['hotels_list']
            }
        })
        return context
```

### Детализация отеля
Отображает информацию об отеле, включая список комнат и отзывы.
```python
class HotelDetailView(DetailView):
    model = Hotel
    template_name = 'hotels/details.html'

    def get_context_data(self, *args, **kwargs):
        context = super().get_context_data(**kwargs)
        context.update({
            "hotel_rooms": Room.objects.filter(hotel=context['hotel']),
            "review_list": Review.objects.filter(reservation__room__hotel=context['hotel']),
        })
        return context
```

### Детализация комнаты
Отображает информацию о комнате, доступные для бронирования даты и форму для бронирования.
```python
class RoomDetailView(DetailView):
    model = Room
    template_name = 'room/details.html'

    def get_context_data(self, *args, **kwargs):
        context = super().get_context_data(**kwargs)
        dt_start = self.request.GET.get('dt_start') or datetime.today().date()
        dt_end = self.request.GET.get('dt_end') or datetime.today().date() + timedelta(days=30)
        dt_start = parse_or_return(dt_start)
        dt_end = parse_or_return(dt_end)
        reservations = Reservation.objects.filter(Q(room=context['room']) & (Q(dt_end__gt=dt_start) | Q(dt_end__lte=dt_end)))
        possible_dates = [dt_start + timedelta(days=i) for i in range((dt_end - dt_start).days + 1)]

        for res in reservations:
            for dt_possible in range((res.dt_end - res.dt_start).days + 1):
                try:
                    possible_dates.remove(res.dt_start + timedelta(days=dt_possible))
                except ValueError:
                    pass

        context.update({
            "possible_dates": [t.strftime('%d/%m/%Y') for t in possible_dates],
            "form": ReservationForm
        })
        return context
```


## Бронирования

### Создание бронирования
Создает бронирование, проверяя доступность выбранных дат.
```python
@require_POST
@login_required
def create_reservation(request, pk):
    room = get_object_or_404(Room, pk=pk)
    dt_start = parse_or_return(request.POST.get('dt_start'))
    dt_end = parse_or_return(request.POST.get('dt_end'))
    current_date = datetime.today()

    if dt_start > dt_end:
        messages.error(request, 'Error: Start date > End Date')
        return redirect(f'/room/{pk}/')
    if current_date > dt_start:
        messages.error(request, "Error: You can't reserve room in the past")
        return redirect(f'/room/{pk}/')

    reservations = Reservation.objects.filter(Q(room=room) & (Q(dt_end__gt=dt_start) | Q(dt_end__lte=dt_end)))
    possible_dates = [dt_start + timedelta(days=i) for i in range((dt_end - dt_start).days + 1)]
    form = ReservationForm(request.POST)

    for res in reservations:
        for dt_possible in range((res.dt_end - res.dt_start).days + 1):
            try:
                possible_dates.remove(res.dt_start + timedelta(days=dt_possible))
            except ValueError:
                pass

    if dt_start in possible_dates and dt_end in possible_dates:
        try:
            if form.is_valid():
                reservation = form.save(commit=False)
                reservation.room = room
                reservation.user = request.user
                reservation.save()
                messages.success(request, 'Reservation created successfully!')
                return redirect(f'/room/{pk}/', pk=pk)
            else:
                messages.error(request, 'Please correct the errors below.')
        except ValueError:
            return HttpResponseForbidden("Bad data")
    return messages.error(request, "Reservation not created.")
```

### Список бронирований
Выводит список бронирований пользователя.
```python
class BookingsListView(LoginRequiredMixin, ListView):
    model = Reservation
    template_name = 'bookings/list.html'
    context_object_name = 'reservations'

    def get_queryset(self):
        if self.request.user.is_authenticated:
            return Reservation.objects.filter(user=self.request.user)
        return Reservation.objects.none()
```

### Удаление бронирования
Удаляет бронирование, если пользователь имеет на это право.
```python
class BookingDeleteView(LoginRequiredMixin, DeleteView):
    model = Reservation
    template_name = 'bookings/delete.html'
    success_url = reverse_lazy('bookings')

    def post(self, request, *args, **kwargs):
        instance = self.get_object()
        if instance.user != self.request.user:
            return HttpResponseForbidden("You do not have permission to delete this object.")
        return super().post(request, *args, **kwargs)
```


## Отзывы

### Создание отзыва
Пользователь может оставить отзыв только для своего бронирования.
```python
class ReviewCreateView(LoginRequiredMixin, CreateView):
    model = Review
    template_name = 'bookings/create.html'
    fields = ['text', 'rating']
    success_url = reverse_lazy('bookings')

    def form_valid(self, form):
        form.instance.author = self.request.user
        reservation = get_object_or_404(Reservation, pk=self.kwargs['pk'])
        if reservation.user != self.request.user:
            return HttpResponseForbidden("You do not have permission to review this reservation.")
        form.instance.reservation = reservation
        return super().form_valid(form)
```
