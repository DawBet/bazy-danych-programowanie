# Kursory

## Przygotowanie

### Baza danych

Utwórz bazę danych i użytkownika z wszystkimi uprawnieniami do tej bazy danych. Wykorzystaj do tego MySQL Workbench.

```mysql
CREATE USER 'uzytkownik'@'localhost' IDENTIFIED BY 'haslo';
CREATE DATABASE 'sda';
USE sda;
GRANT ALL PRIVILEGES ON 'sda' TO 'uzytkownik'@'localhost';
```

### Projekt

Utwórz projekt w PyCharm i zainstaluj paczkę connectora MySQL:

```bash
pip install 'mysql-connector-python<=8.0.12'
```

## Ćwiczenia

1. Stwórz nowy plik `.py`. Utwórz połączenie z bazą danych:

    ```python
    import mysql.connector

    connection = mysql.connector.connect(
        user='uzytkownik',
        password='haslo',
        database='sda',
        host='localhost',
    )

    connection.close()
    ```

    Zawsze należy pamiętać o wywołaniu `connection.close()`!

2. Utwórz **kursor**, który będziemy mogli wykorzystać do poruszania się po bazie danych w Pythonie:

    ```python
    ...
    cursor = connection.cursor()

    cursor.close()
    ...
    ```

    Podobnie jak w przypadku połączenia, należy również pamiętać, aby zamykać kursor po zakończeniu wykonywania operacji na bazie danych.

3. Utwórz nową tabelę `users`, korzystając z utworzonego kursora:

    ```python
    cursor.execute("""
        CREATE TABLE users (
            id INTEGER PRIMARY KEY AUTO_INCREMENT,
            first_name VARCHAR(50) NOT NULL,
            last_name VARCHAR(50) NOT NULL,
            birthday DATE,
            post_count INTEGER DEFAULT 0,
            rating FLOAT,
            joined_date DATETIME DEFAULT NOW()
        );
    """)
    ```

    Tabela powinna pojawić się w bazie danych. Możemy to sprawdzić w MySQL Workbench.

4. Stwórz nowy plik `.py` i skopiuj do niego kod odpowiadający za nawiązywanie połączenia i utworzenie kursora. Dodaj nowy rekord do tabeli:

    ```python
    cursor.execute("""
        INSERT INTO users (first_name, last_name, birthday)
            VALUES (%s, %s, %s);
    """, ('Imie', 'Nazwisko', '1970-01-01'))
    ```

    Teraz sprawdźmy, czy rekord pojawił się w bazie danych, korzystając z MySQL Workbench.

5. Powtórzmy powyższą operację z dodatkowym wywołaniem metody `commit()`:

    ```python
    cursor.execute("""
        INSERT INTO users (first_name, last_name, birthday)
            VALUES (%s, %s, %s);
    """, ('Imie', 'Nazwisko', '1970-01-01'))

    connection.commit()
    ```

    Ponownie sprawdźmy, czy rekord pojawił się w bazie danych, za pomocą MySQL Workbench.

    Używanie metody `commit()` za każdym razem możemy pominąć, dodając do metody `connect()` argument `autocommit=True`. Nie jest to jednak polecane, ponieważ zwiększa podatność bazy danych na błędy, zwłaszcza w przypadku zmian w wielu tabelach jednocześnie.

6. Ponownie stwórz nowy plik `.py` zawierający kod nawiązania połączenia i utworzenia kursora. Pobierzmy listę użytkowników za pomocą kursora:

    ```python
    cursor.execute("""
        SELECT * FROM users;
    """)

    for row in cursor:
        print(row)
    ```

7. Wykonaj wszystkie zapytania SQL z pliku `dane/users.sql`:

    ```python
    with open('dane/users.sql') as users_sql:
        for line in users_sql:
            cursor.execute(line)
        connection.commit()
    ```

8. Ponownie pobierzmy listę wszystkich użytkowników:

    ```python
    cursor.execute("""
        SELECT * FROM users;
    """)

    for row in cursor:
        print(row)
    ```

## Zadania

Wszystkie poniższe zadania mogą być wykonane w jednym pliku `.py`. W takim przypadku dobrym pomysłem będzie oznaczenie

1. Wybierz imiona (`first_name`) i nazwiska (`last_name`) wszystkich użytkowników posortowanych alfabetycznie wg. imienia. Ogranicz wyniki do pierwszych dziesięciu użytkowników.

2. Wybierz imiona (`first_name`) i nazwiska (`last_name`) wszystkich użytkowników posortowanych w porządku odwrotnym alfabetycznie wg. nazwiska. Ogranicz wyniki do pierwszych dziesięciu użytkowników.

3. Wybierz imiona (`first_name`), nazwiska (`last_name`) i liczbę postów (`post_count`) wszystkich użytkowników, którzy mają przynajmniej `3400` postów, posortowanych malejąco wg. liczby postów.

4. Wybierz imiona (`first_name`), nazwiska (`last_name`), liczbę postów (`post_count`) i ocenę (`rating`) wszystkich użytkowników, którzy mają  przynajmniej `3400` postów **lub** ocenę większą niż `4.0`, posortowanych malejąco wg. oceny.

5. Wybierz imiona (`first_name`), nazwiska (`last_name`), liczbę postów (`post_count`), ocenę (`rating`) i datę dołączenia (`joined_date`) wszystkich użytkowników, którzy mają przynajmniej `3400` postów **lub** ocenę większą niż `4.0`, a ponadto dołączyli między 2000, a 2010 rokiem, posortowanych rosnąco wg. daty dołączenia.

6. Wybierz imiona (`first_name`) i nazwiska (`last_name`) wszystkich użytkowników, którzy mają na imię `Carolynn`, `Christie` lub `West`.

7. Wybierz imiona (`first_name`) i nazwiska (`last_name`) wszystkich użytkowników, których imię zaczyna się na literę `S`.

8. Wybierz imiona (`first_name`) i nazwiska (`last_name`) wszystkich użytkowników, których imię zaczyna się na literę `S` lub `D` i kończy się literą `a`.

9. Wybierz imiona (`first_name`) i nazwiska (`last_name`) wszystkich użytkowników, których nazwisko zawiera literę `G` (lub małe `g`).

10. Policz ile użytkowników znajduje się w tabeli `users`.

11. Policz ile postów (`post_count`) mają sumarycznie wszyscy użytkownicy.

12. Sprawdź, jakie są wartości minimalne i maksymalne ocen (`rating`) użytkowników.

13. Policz jaka jest średnia ocena (`rating`) wszystkich użytkowników.

14. Wyświetl liczbę powtórzeń dla każdego imienia (`first_name`) w tabeli `users` i posortuj wyniki malejąco wg. liczby powtórzeń. Ogranicz wyniki do 10 najczęściej występujących imion.

15. Znajdź wszystkich użytkowników, którzy mają przynajmniej jeden post (`post_count`), ale nie mają oceny (`rating`). Posortuj wyniki malejąco wg. liczby postów.
