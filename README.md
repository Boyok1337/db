# PostgreSQL в Django на прикладі моделі Book 📚

Модель **Book** в Django представляє книгу з такими полями, як назва (title), автор (author), тип обкладинки (cover), кількість одиниць на складі (inventory), та щоденний збір за оренду (daily_fee). Ця модель буде зберігатися у базі даних PostgreSQL, і через Django ORM ми можемо працювати з нею без написання складних SQL запитів.

## Структура таблиці у PostgreSQL 🗃️

Коли ви запускаєте міграції, Django автоматично генерує таблицю для цієї моделі у базі даних. SQL-вираз для створення таблиці в PostgreSQL може виглядати так:

```sql
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(255) DEFAULT 'Unknown Author',
    cover VARCHAR(4) CHECK (cover IN ('HARD', 'SOFT')),
    inventory INTEGER NOT NULL,
    daily_fee NUMERIC(6, 2) NOT NULL
);
```

Таблиця books міститиме:

- `id`: Автоматично згенероване значення для кожної нової книги (це поле є первинним ключем). 🔑
- `title`: Назва книги. 📖
- `author`: Автор книги, з дефолтним значенням "Unknown Author". ✍️
- `cover`: Тип обкладинки (може бути або "HARD", або "SOFT"). 💼📚
- `inventory`: Кількість одиниць цієї книги на складі. 📦
- `daily_fee`: Щоденний збір за оренду книги (десяткове значення). 💵

## CRUD операції 🛠️

### 1. Create (Створення нового запису) ✨

Для додавання нової книги до бази даних ми можемо використовувати Django ORM, щоб створити новий запис. Наприклад:

```python
from myapp.models import Book

new_book = Book.objects.create(
    title="The Great Gatsby",
    author="F. Scott Fitzgerald",
    cover="HARD",
    inventory=10,
    daily_fee=1.99
)
```

Це створить SQL-запит типу:

```sql
INSERT INTO books (title, author, cover, inventory, daily_fee)
VALUES ('The Great Gatsby', 'F. Scott Fitzgerald', 'HARD', 10, 1.99);
```

### 2. Read (Читання записів) 📖

Щоб отримати інформацію про книги, можна використовувати різні методи для вибору даних:

Вибір усіх книг:
```python
books = Book.objects.all()
```

Це згенерує SQL-запит:

```sql
SELECT * FROM books;
```

Вибір книги за її ID:
```python
book = Book.objects.get(id=1)
```

SQL-запит:

```sql
SELECT * FROM books WHERE id=1;
```

### 3. Update (Оновлення запису) 🔄

Для оновлення записів використовуємо метод `save()` після зміни значення одного з полів:

```python
book = Book.objects.get(id=1)
book.inventory = 12  # Оновлюємо кількість книг на складі
book.save()
```

Це згенерує SQL-запит:

```sql
UPDATE books SET inventory=12 WHERE id=1;
```

### 4. Delete (Видалення запису) ❌

Для видалення записів ми використовуємо метод `delete()`:

```python
book = Book.objects.get(id=1)
book.delete()  # Видалити книгу з id=1
```

SQL-запит буде таким:

```sql
DELETE FROM books WHERE id=1;
```

Або для видалення кількох записів:

```python
Book.objects.filter(author="F. Scott Fitzgerald").delete()
```

Запит:

```sql
DELETE FROM books WHERE author='F. Scott Fitzgerald';
```

## Підсумки 🎯

- **Створення таблиці**: Django генерує SQL для створення таблиці PostgreSQL на основі визначених моделей.
- **CRUD операції**: Django ORM надає прості методи для виконання стандартних операцій з базою даних (Create, Read, Update, Delete).
- **SQL-запити**: Всі запити автоматично генеруються Django ORM, так що вам не потрібно писати складний SQL вручну. 🔄

## Використання Elasticsearch для оптимізації пошуку 🔍
Під час роботи з базою даних книг ми зіткнулися з необхідністю прискорити пошукові операції. Незважаючи на надійність PostgreSQL, для більш ефективного повнотекстового пошуку ми впровадили Elasticsearch.
Міграція даних 🔄
Процес міграції включав наступні кроки:

- Часткове дублювання даних з PostgreSQL в Elasticsearch
- Індексація книг за ключовими полями: title, author
- Налаштування analyzer для більш точного пошуку

# Приклад індексації в Elasticsearch
```python
def index_book(book):
    es_client.index(
        index='books',
        body={
            'title': book.title,
            'author': book.author,
            'inventory': book.inventory
        }
    )
```
### Переваги впровадження 🚀

- Час пошуку зменшився на 70-80%
- Можливість нечіткого пошуку
- Підтримка складних пошукових запитів

### Порівняння SQL та NoSQL баз даних 📊
#### SQL (PostgreSQL):
Плюси:

- Строга структура даних
- ACID-транзакції
- Цілісність даних

Мінуси:

- Складність горизонтального масштабування
- Фіксована схема
- Повільніший повнотекстовий пошук

#### NoSQL (Elasticsearch):
Плюси:

- Висока швидкість пошуку
- Гнучка схема даних
- Легке горизонтальне масштабування

Мінуси:

- Eventual consistency
- Складніша підтримка складних транзакцій
- Вища вартість інфраструктури
