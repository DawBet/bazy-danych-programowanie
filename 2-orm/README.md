# ORM

## Przygotowanie

### Projekt Django

Utwórz w PyCharm projekt Django o nazwie `orm` z aplikacją `sda`. Po utworzeniu projektu zainstaluj pakiet connectora MySQL:

```bash
pip install 'mysql-connector-python<=8.0.12'
```

#### Konfiguracja

W pliku `orm/settings.py` zmień poniższy fragment:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

na:

```python
DATABASES = {
    'default': {
        'ENGINE': 'mysql.connector.django',
        'NAME': 'sda',
        'USER': 'uzytkownik',
        'PASSWORD': 'haslo',
        'HOST': 'localhost',
        'PORT': '3306',
    }
}
```

Dodatkowo, na końcu pliku `orm/settings.py`, dodajmy konfigurację logowania, aby widzieć jakie zapytania SQL Django wykonuje pod "przykrywką" ORM:

```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'filters': {},
    'handlers': {
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': True,
        },
    },
}
```

## Ćwiczenia

### Tworzenie modelu

W pliku `sda/models.py`:

```python
from django.db import models


class User(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    birthday = models.DateField(null=True, blank=True)
    post_count = models.IntegerField(default=0)
    rating = models.FloatField(null=True, blank=True)
    joined_date = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.first_name} {self.last_name}"

```

Zwróć uwagę, że nie definiujemy dla modelu klucza głównego (primary key). Nie jest to konieczne, ponieważ w przypadku kiedy nie stworzymy pola będącego kluczem głównym, Django ORM sam utworzy pole pełniące rolę klucza głównego.

### Migracje

1. Tworzymy plik migracji:

    ```bash
    ./manage.py makemigrations
    ```

    Wykonanie powyższej komendy utworzy nam plik migracji w katalogu `sda/migrations/`. Plików tych (zwykle) nie powinniśmy zmieniać ręcznie, chyba że dokładnie wiemy co robimy.

2. Aplikujemy migracje do bazy danych:

    ```bash
    ./manage.py migrate
    ```

    W wyniku wykonania migracji możemy zobaczyć, że poza migracją z naszej  aplikacji `sda`, zostały zaaplikowane również inne migracje, takie jak `auth`, `admin`, `contenttypes`, czy `sessions`.

    Są to standardowe migracje Django, używane do podstawowych, wbudowanych funkcjonalności tego frameworka. Nie będziemy się nimi zajmować.

    W wyniku migracji, nasz model `User` jest tłumaczony na polecenia w języku SQL silnika bazy danych, którego używamy (w naszym przypadku MySQL), które są wykonywane na wskazanej przez nas bazie danych.

    Tabelę `sda_user` utworzoną w wyniku migracji możemy zobaczyć w MySQL Workbench. Znajdziemy też inne tabele automatycznie utworzone z domyślnych migracji Django.

## Django Shell

### Wprowadzenie

Django Shell jest powłoką Pythonową, czyli interaktywnym interfejsem terminalowym, w którym możemy wykonywać polecenia w języku Python, podobnie jak ma to miejsce w powłoce Bash.

Możemy go uruchomić wykonując polecenie:

```bash
./manage.py shell
```

Powinniśmy zobaczyć konsolę podobną do poniższej:

```python
Python 3.7.4 (default, Sep  7 2019, 18:27:02)
[Clang 10.0.1 (clang-1001.0.46.4)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>>
```

Znak `>>>` jest miejscem, w którym wprowadzamy polecenia Pythonowe. Jeśli polecenie będzie składało się z wielu linii, każda kolejna będzie zaczynała się znakami `...`.

Wyniki wywołania poleceń nie są poprzedzone żadnymi znakami.

Przykład:

```python
Python 3.7.4 (default, Sep  7 2019, 18:27:02)
[Clang 10.0.1 (clang-1001.0.46.4)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> import random
>>> for i in range(10):
...     print(random.randint(1, 100))
...
99
83
54
64
87
9
75
13
80
2
>>>
```

### Praca z Django ORM

Zacznijmy od zaimportowania naszego modelu `User`:

```python
>>> from sda.models import User
```

Model ten jest naszym punktem wejściowym do zarządzania danymi w naszej bazie danych. Korzystając z niego możemy tworzyć nowe rekordy w bazie danych, usuwać je, wybierać i aktualizować.

1. Wybierz wszystkich użytkowników:

    ```python
    >>> User.objects.all()
    <QuerySet []>
    ```

    Czym jest `QuerySet`? W jaki sposób działa?

2. Utwórz nowego użytkownika:

    ```python
    >>> user = User(first_name='Imie', last_name='Nazwisko', birthday='1985-01-01')
    >>> user
    <User: Imie Nazwisko>
    ```

    Sprawdźmy teraz, korzystając z MySQL Workbench, czy użytkownik pojawił się w bazie danych.

3. Ponownie stwórz nowego użytkownika i sprawdź, czy pojawił się w bazie danych:

    ```python
    >>> user = User(first_name='Imie', last_name='Nazwisko', birthday='1985-01-01')
    >>> user.save()
    >>> user
    <User: Imie Nazwisko>
    ```

4. To, czy użytkownik został dodany do bazy danych, możemy poznać po jego kluczu głównym (primary key):

    ```python
    >>> user = User(first_name='Inny', last_name='Uzytkownik')
    >>> print(user.pk)
    None
    >>> user.save()
    >>> print(user.pk)
    2
    ```

5. Ponownie wybierzmy wszystkich użytkowników z bazy danych:

    ```python
    >>> User.objects.all()
    <QuerySet [<User: Imie Nazwisko>, <User: Inny Uzytkownik>]>
    ```

6. Możemy również pobrać jednego, konkretnego użytkownika, na podstawie dowolnych pól:

    ```python
    >>> User.objects.get(first_name='Inny')
    <User: Inny Uzytkownik>
    >>> User.objects.get(first_name='Imie', last_name='Nazwisko')
    <User: Imie Nazwisko>
    ```

    Należy jednak uważać, ponieważ jeśli podany przez nas zestaw pól i ich wartości występuje u więcej niż jednego użytkownika, rzucony zostanie wyjątek:

    ```python
    >>> User.objects.get(post_count=0)
    Traceback (most recent call last):
      File "<console>", line 1, in <module>
      File "./django/db/models/manager.py", line 82, in manager_method
        return getattr(self.get_queryset(), name)(*args, **kwargs)
      File "./django/db/models/query.py", line 412, in get
        (self.model._meta.object_name, num)
    sda.models.User.MultipleObjectsReturned: get() returned more than one User -- it returned 2!
    ```

7. Możemy też wybrać pierwszego z brzegu użytkownika:

    ```python
    >>> User.objects.first()
    <User: Imie Nazwisko>
    ```

    Zwykle będzie to pierwszy rekord, jaki został dodany do bazy danych, chyba że wcześniej zastosujemy filtrowanie (klaula `WHERE`) lub posortujemy rekordy (klauzula `ORDER BY`).

8. Przy tworzeniu nowych rekordów, możemy uniknąć używania metody `save()`, tworząc i dodając nowe rekordy jednym poleceniem:

    ```python
    >>> User.objects.create(first_name='Uzytkownik', last_name='Trzeci')
    <User: Uzytkownik Trzeci>
    ```

9. Jeśli nie jesteśmy pewni, czy rekord który chcemy dodać do bazy danych już istnieje, a jednocześnie chcemy uniknąć duplikacji danych, możemy skorzystać z poniższej metody:

    ```python
    >>> User.objects.get_or_create(first_name='Uzytkownik', last_name='Trzeci')
    (<User: Uzytkownik Trzeci>, False)
    ```

    W wyniku otrzymamy krotkę (`tuple`) z dwoma elementami: obiektem użytkownika oraz wartością logiczną.

    Jeśli rekord już istniał w bazie danych (tak jak w powyższym przypadku), zostanie on pobrany i zwrócony, a wartość logiczna przyjmie wartość `False`.

    Jeśli rekord nie istniał w bazie, zostanie on stworzony, zapisany i dopiero zwrócony (tak jak w metodzie `create()`), a wartość logiczna przyjmie wartość `True`.

10. W Django możemy korzystać również z bezpośredniego połączenia z bazą danych, a co za tym idzie - z kursorów:

    ```python
    >>> from django.db import connection
    >>> with connection.cursor() as cursor:
    ...     cursor.execute('SELECT * FROM sda_user')
    ...     cursor.fetchone()
    ...
    ```

    Zwróć uwagę, że w Django możemy korzystać z polecenia `with`, więc nie musimy się martwić o zamykanie połączenia, czy kursora.

11. Wykorzystajmy powyższy mechanizm do zaimportowania użytkowników:

    ```python
    >>> with open('dane/users_orm.sql') as users_sql:
    ...    with connection.cursor() as cursor:
    ...        for line in users_sql:
    ...            cursor.execute(line)
    ...        connection.commit()
    ...
    ```

    Możemy teraz użyć MySQL Workbench do sprawdzenia, czy użytkownicy zostali dodani.

    Możemy też wykorzystać Django ORM:

    ```python
    >>> users = User.objects.all()
    >>> users
    <QuerySet [<User: Imie Nazwisko>, <User: Inny Uzytkownik>, <User: Uzytkownik Trzeci>, <User: Margarete Tewkesberry>, <User: Joann Loghan>, <User: Devan McAnellye>, <User: Tanny Tokell>, <User: Kristin Fulker>, <User: Loren Westnage>, <User: Ferdinanda Stollenhof>, <User: Rahel Wulfinger>, <User: Charissa Rabb>, <User: Collie Chanter>, <User: Fritz Mowday>, <User: Margaretha Plumbley>, <User: Bathsheba Dabell>, <User: Marika Lingwood>, <User: Bary Heisman>, <User: Etienne Davidson>, <User: Ira Hovel>, '...(remaining elements truncated)...']>
    ```

    Zwróć uwagę, że standardowo próba wyświetlenia wszystkich użytkowników jest ograniczona do pierwszych 21 rekordów. Limit ten oczywiście można zmienić, jednak służy to temu, aby nie obciążać niepotrzebnie bazy danych dużymi zapytaniami.

    Aby sprawdzić ile rzeczywiście rekordów zostało wybranych, możemy użyć metodę `count()`:

    ```python
    >>> users.count()
    1003
    ```

    Nie powinniśmy używać funkcji `len()`, ponieważ w jej przypadku konieczne jest pobranie danych wszystkich użytkowników, aby zliczyć liczbę ich obiektów. W przypadku metody `count()`, zliczanie odbywa się po stronie bazy danych, co znacznie przyspiesza operację.

#### Filtrowanie rekordów

Wykonywanie zapytań: [Making queries | Django 2.2](https://docs.djangoproject.com/pl/2.2/topics/db/queries/)

Field lookups: [Field lookups | Django 2.2](https://docs.djangoproject.com/pl/2.2/topics/db/queries/#field-lookups)

1. Wybierz wielu użytkowników z polem o określonej wartości.

    ```python
    >>> users = User.objects.filter(first_name='West)
    >>> users
    <QuerySet [<User: West Magson>, <User: West Tawse>]>
    ```

    Aby zobaczyć wartości innych pól użytkownika poza imieniem i nazwiskiem, możemy wykorzystać wybranych użytkowników jako iterator:

    ```python
    >>> for user in users:
    ...     print(user.first_name, user.last_name, user.post_count, user.rating)
    ...
    West Magson 1232 4.66
    West Tawse 1523 4.25
    ```

2. Wybierz użytkowników posortowanych alfabetycznie wg. imienia.

    ```python
    >>> User.objects.order_by('first_name')
    ```

3. Wybierz użytkowników posortowanych w porządku odwrotnym alfabetycznie (od Z do A) wg. nazwiska. Ogranicz wyniki do pierwszych dziesięciu użytkowników

    ```python
    >>> User.objects.order_by('-last_name')[:10]
    ```

4. Wybierz użytkowników, którzy mają przynajmniej 3400 postów (`post_count`). Posortuj wyniki malejąco wg. liczby postów.

    ```python
    >>> User.objects.filter(post_count__gte=3400).order_by('-post_count')
    ```

5. Wybierz użytkowników, którzy mają mniej niż 100 postów i ocenę (`rating`) przynajmniej 3.0. Posortuj wyniki rosnąco wg. oceny.

    ```python
    >>> User.objects.filter(post_count__lt=100).filter(rating__gte=3.0).order_by('rating')
    ```

6. Wybierz użytkowników, którzy dołączyli (`joined_date`) między 2000, a 2010 rokiem. Posortuj wyniki rosnąco wg. daty dołączenia.

    ```python
    >>> User.objects.filter(joined_date__between=('2000-01-01', '2010-01-01')).order_by('joined_date')
    >>> # LUB
    >>> User.objects.filter(joined_date__gte='2000-01-01').filter(joined_date__lt='2010-01-01').order_by('joined_date')
    ```

7. Wybierz użytkowników, którzy mają na imię Carolynn, Christie lub West.

    ```python
    >>> User.objects.filter(first_name__in=('Carolynn', 'Christie', 'West'))
    ```

8. Wybierz użytkowników, których imiona zaczynają się na literę "S".

    ```python
    >>> User.objects.filter(first_name__startswith='S')
    ```

9. Wybierz użytkowników, których imiona zaczynają się na literę "D" i kończą na literę "a".

    ```python
    >>> User.objects.filter(first_name__startswith='D').filter(first_name__endswith='a')
    ```

10. Wybierz użytkowników, których imiona zawierają literę "g" lub "G".

    ```python
    >>> User.objects.filter(first_name__icontains="g")
    ```

11. Pobierz sumę postów wszystkich użytkowników.

    ```python
    >>> from django.db.models import Sum
    >>> User.objects.aggregate(Sum('post_count'))
    ```

12. Pobierz wartości minimalne i maksymalne ocen użytkowników.

    ```python
    >>> from django.db.models import Min, Max
    >>> User.objects.aggregate(Min('rating'), Max('rating'))
    ```

13. Oblicz średnią ocenę wszystkich użytkowników.

    ```python
    >>> from django.db.models import Avg
    >>> User.objects.aggregate(Avg('rating'))
    ```

14. Wyświetl liczbę powtórzeń każdego imienia. Posortuj wyniki malejąco wg. liczby powtórzeń.

    ```python
    >>> from django.db.models import Count
    >>> User.objects.values('first_name').annotate(Count('first_name')).order_by('-first_name__count)
    >>> # LUB
    >>> User.objects.values_list('first_name').annotate(Count('first_name')).order_by('-first_name__count)
    ```

    Jaka jest różnica między wariantem z metodą `values()` i `values_list()`?

15. Znajdź wszystkich użytkowników, którzy mają przynajmniej jeden post, ale nie mają oceny. Posortuj wyniki malejąco wg. liczby postów.

    ```python
    >>> User.objects.filter(post_count__gt=0).filter(rating=None).order_by('-post_count')
    ```

16. Wybierz wszystkich użytkowników, którzy nie spełniają warunków z poprzedniego ćwiczenia.

    ```python
    >>> User.objects.exclude(post_count__gt=0).exclude(rating=None)
    ```

17. Wybierz wszystkich użytkowników, których imiona zaczynają się na literę "S" lub literę "D". Posortuj wyniki rosnąco wg. nazwiska.

    ```python
    >>> from django.db.models import Q
    >>> User.objects.filter(Q(first_name__startswith='S') | Q(first_name__startswith='D')).order_by('last_name')
    ```

18. Wybierz wszystkich użytkowników, których imiona zaczynają się na literę "S" lub "D" i kończą się literą "a".

    ```python
    >>> User.objects.filter((Q(first_name__startswith='S') | Q(first_name__startswith='D')) & Q(first_name__endswith="a"))
    >>> # LUB
    >>> User.objects.filter(Q(first_name__startswith='S') | Q(first_name__startswith='D')).filter(first_name__endswith='a')
    ```

19. Wybierz wszystkich użytkowników, których nazwisko zawiera literę "x" lub ich imię nie zawiera litery "a".

    ```python
    >>> User.objects.filter(Q(last_name__icontains='x') | ~Q(first_name__icontains='a'))
    ```

20. Wybierz wszystkich użytkowników, których imię zawiera literę "s" (lub "S"), mają mniej niż 1000 lub więcej niż 2500 postów, a ich ocena jest pomiędzy 2.0, a 4.0. Pomiń osoby urodzone przed 1990 rokiem. Wykini posortuj malejąco wg. daty dołączenia.

    ```python
    >>> User.objects.filter(first_name__icontains='s').filter(Q(post_count__lt=1000) | Q(post_count__gt=2500)).filter(rating__gt=2.0).filter(rating__lt=4.0).exclude(birthday__lt='1990-01-01').order_by('-joined_date')
    ```

## Zadania

Zadania komunikacji z bazą danych w Pythonie powinny być wykonane w Django Shell. Pamiętaj, że po zaaplikowaniu każdej migracji powinieneś zrestartować Shell.

1. Stwórz model `Post`, bazując na poniższym zapytaniu SQL.

    ```sql
    CREATE TABLE sda_post (
        id INTEGER PRIMARY KEY AUTO_INCREMENT,
        author_id INTEGER NOT NULL FOREIGN KEY REFERENCES sda_user(id),
        content TEXT NOT NULL,
        added_date DATETIME DEFAULT NOW(),
        changed_date DATETIME DEFAULT NOW(),
    );
    ```

2. Upewnij się, że do stworzenia relacji między użytkownikiem, a postami, użyłeś mechanizmów Django do tworzenia relacji jeden-do-wielu:

    ```python
    class Post(models.Model):
        ...
        author = models.ForeignKey(User, related_name='posts')
        ...
    ```

3. Stwórz nowy plik migracji i zaaplikuj go do bazy danych.

4. Wykonaj wszystkie polecenia SQL z pliku `dane/posts.sql` na swojej bazie danych. Możesz to zrobić zarówno korzystając z Pythona, jak i przez MySQL Workbench.

5. Policz ile postów znajduje się w bazie danych.

6. Znajdź 20 najnowszych postów.

7. Znajdź posty, które mają inną datę dodania (`added_date`) i datę zmiany (`changed_date`).

8. Znajdź posty wykonane przez autorów, którzy w swoim imieniu mają literę "x".

9. Wybierz imiona i nazwiska wszystkich użytkowników, wraz z liczbą postów, jakie są do nich przypisane. Wyniki posortuj malejąco wg. liczby postów.

10. Znajdź użytkownika, który jest autorem najstarszego (`added_date`) posta.

11. Znajdź wszystkich użytkowników, którzy napisali posty w latach 2005 - 2010. Wyświetl ile postów napisali w tym okresie. Wyniki posortuj malejąco wg. liczby postów napisanych w tym okresie.
