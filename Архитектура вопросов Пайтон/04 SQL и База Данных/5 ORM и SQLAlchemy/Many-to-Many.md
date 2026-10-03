# Many-to-Many 🔗

## 🎤 Короткий ответ

**Many-to-Many (многие-ко-многим)** — это связь, при которой одна запись может быть связана со многими записями другой таблицы, и наоборот. В реляционной БД обычно реализуется через **промежуточную таблицу (association/junction table)** с двумя Foreign Key.

**Django:** `ManyToManyField`
**SQLAlchemy:** `relationship(..., secondary=...)`

---

## 🎯 Формула для собеседования

```text id="x7m2q4"
Many-to-Many
      ↓
промежуточная таблица
      ↓
FK → Table A
FK → Table B
```

Пример:

```text id="p4n8c2"
Student ←── student_course ──→ Course
   │             │                │
   └────── N ────┴──── N ─────────┘
```

---

# 1. Что такое Many-to-Many

Простой пример — **студенты и курсы**.

Один студент может изучать несколько курсов:

```text id="m3q8v1"
Ilya
 ├── Python
 ├── SQL
 └── Django
```

Но один курс могут изучать несколько студентов:

```text id="c6x2n9"
Python
 ├── Ilya
 ├── Alex
 └── Maria
```

Поэтому:

```text id="k8r4m5"
Student N ↔ N Course
```

---

# 2. Почему нельзя просто поставить Foreign Key

Если сделать:

```text id="v5n2q7"
Student
-------
id
course_id
```

то у студента можно хранить только **один** `course_id`.

А нам нужно:

```text id="z3m7c9"
Student 1
 ├── Course 1
 ├── Course 2
 └── Course 3
```

Если попытаться хранить список ID в одном поле:

```text id="q6x4n8"
course_ids = [1, 2, 3]
```

это нарушает нормальную реляционную модель и создаёт проблемы с:

* индексами;
* FK;
* JOIN;
* целостностью данных;
* фильтрацией.

Поэтому используется отдельная таблица.

---

# 3. Промежуточная таблица

Получаем три таблицы:

```text id="r8m2v5"
STUDENT
-------
id
name

COURSE
------
id
name

STUDENT_COURSE
--------------
student_id
course_id
```

Пример:

```text id="n4c7x1"
student_id | course_id
-----------+----------
1          | 10
1          | 20
1          | 30
2          | 10
2          | 20
```

Это означает:

```text id="w6q3m8"
Student 1 → Course 10
Student 1 → Course 20
Student 1 → Course 30

Student 2 → Course 10
Student 2 → Course 20
```

---

# 4. Foreign Key

Промежуточная таблица содержит два FK:

```text id="b9m4q2"
student_id → student.id
course_id  → course.id
```

Схема:

```text id="t5x8n3"
┌──────────────┐
│   STUDENT    │
├──────────────┤
│ id PK        │
│ name         │
└──────┬───────┘
       │
       │ 1
       ↓
┌────────────────────┐
│   STUDENT_COURSE   │
├────────────────────┤
│ student_id FK      │
│ course_id FK       │
└─────────┬──────────┘
          │
          │ N
          ↓
┌──────────────┐
│    COURSE    │
├──────────────┤
│ id PK        │
│ name         │
└──────────────┘
```

Фактически связь:

```text id="p7v2m6"
Student 1
    ↓
N records in junction table
    ↓
Course
```

---

# 5. Django ORM

В Django Many-to-Many создаётся через:

```python id="f3k8q1"
class Student(models.Model):
    name = models.CharField(max_length=100)


class Course(models.Model):
    name = models.CharField(max_length=100)
    students = models.ManyToManyField(Student)
```

Django автоматически создаёт промежуточную таблицу.

Упрощённо:

```text id="y8m4c2"
student
course
course_students
```

---

# 6. Добавление связи

Создадим объекты:

```python id="q5n7x3"
student = Student.objects.create(
    name="Ilya",
)

course = Course.objects.create(
    name="Python",
)
```

Добавляем связь:

```python id="m2v8k4"
course.students.add(student)
```

Можно добавить несколько:

```python id="c7r3n9"
course.students.add(
    student1,
    student2,
    student3,
)
```

---

# 7. Получение связанных объектов

Все курсы студента:

```python id="x4q8m1"
student.course_set.all()
```

Если указать `related_name`:

```python id="v9n3k6"
courses = models.ManyToManyField(
    Course,
    related_name="students",
)
```

можно получить:

```python id="p6m2r8"
student.courses.all()
```

И наоборот:

```python id="z3x7c5"
course.students.all()
```

---

# 8. Удаление связи

Удалить связь:

```python id="n8q4m2"
course.students.remove(student)
```

Удалить все связи:

```python id="k5v9x3"
course.students.clear()
```

Важно:

`remove()` удаляет **связь**, а не сам объект `Student`.

---

# 9. Проверка связи

Можно проверить:

```python id="r7m3c8"
course.students.filter(
    id=student.id,
).exists()
```

То есть:

```text id="x2n6q4"
существует запись в промежуточной таблице?
```

---

# 10. Фильтрация через Many-to-Many

Например, найти курсы, которые изучает Ilya:

```python id="c9m4v7"
Course.objects.filter(
    students__name="Ilya",
)
```

Django построит необходимые JOIN через промежуточную таблицу.

Концептуально:

```text id="q8x2n5"
Course
   ↓
Student_Course
   ↓
Student
```

---

# 11. SQL JOIN

Для Many-to-Many обычно необходимо пройти через промежуточную таблицу:

```sql id="v4m7c2"
SELECT course.*
FROM course
JOIN student_course
    ON course.id = student_course.course_id
JOIN student
    ON student.id = student_course.student_id
WHERE student.name = 'Ilya';
```

То есть:

```text id="m8q3x6"
Course
  ↓
junction table
  ↓
Student
```

---

# 12. SQLAlchemy

В SQLAlchemy связь обычно описывается через промежуточную таблицу.

Например:

```python id="x7n4p2"
from sqlalchemy import Table, Column, ForeignKey


student_course = Table(
    "student_course",
    Base.metadata,
    Column(
        "student_id",
        ForeignKey("students.id"),
        primary_key=True,
    ),
    Column(
        "course_id",
        ForeignKey("courses.id"),
        primary_key=True,
    ),
)
```

Модели:

```python id="m5q8v3"
class Student(Base):
    __tablename__ = "students"

    id: Mapped[int] = mapped_column(
        primary_key=True,
    )

    courses: Mapped[list["Course"]] = relationship(
        secondary=student_course,
        back_populates="students",
    )


class Course(Base):
    __tablename__ = "courses"

    id: Mapped[int] = mapped_column(
        primary_key=True,
    )

    students: Mapped[list["Student"]] = relationship(
        secondary=student_course,
        back_populates="courses",
    )
```

Здесь:

```python id="r3c9x6"
secondary=student_course
```

указывает SQLAlchemy на промежуточную таблицу.

---

# 13. SQLAlchemy: работа со связью

Добавить курс студенту:

```python id="q8m4n2"
student.courses.append(course)
```

Получить курсы:

```python id="v6x3k9"
student.courses
```

Получить студентов курса:

```python id="p2r7m5"
course.students
```

---

# 14. Eager Loading Many-to-Many

Для Many-to-Many часто используется:

```python id="c4n8q2"
selectinload()
```

Например:

```python id="x7m3v9"
stmt = select(Student).options(
    selectinload(Student.courses)
)

students = session.scalars(stmt).all()
```

SQLAlchemy:

```text id="k5q2r8"
SELECT students
        ↓
SELECT courses
через student_course
        ↓
связывает результаты
```

---

# 15. Django: prefetch_related

В Django аналогичная задача:

```python id="n9v4c6"
students = Student.objects.prefetch_related(
    "courses",
)
```

Получаем:

```text id="m3x8q1"
SELECT students ...

SELECT courses ...
через промежуточную таблицу
```

После этого:

```python id="r7k2n5"
student.courses.all()
```

использует предварительно загруженные данные.

---

# 16. Почему не select_related()

Для Many-to-Many:

```python id="w5m8q3"
select_related()
```

не используется.

В Django:

```text id="c2n7x4"
Many-to-Many
     ↓
prefetch_related()
```

В SQLAlchemy:

```text id="v8q3m6"
Many-to-Many
     ↓
selectinload()
```

Потому что связь проходит через промежуточную таблицу и возвращает коллекцию объектов.

---

# 17. Уникальность связи

Обычно одна пара:

```text id="j4n8c2"
student_id = 1
course_id = 10
```

не должна встречаться дважды.

Поэтому на промежуточной таблице часто используют составной Primary Key:

```sql id="x6m3q9"
PRIMARY KEY (student_id, course_id)
```

Это запрещает:

```text id="z8r2v5"
1 | 10
1 | 10  ← duplicate
```

---

# 18. Дополнительные данные связи

Иногда самой связи недостаточно.

Например, нужно хранить:

* дату записи на курс;
* статус;
* оценку;
* дату окончания;
* роль.

Тогда простой `ManyToManyField` может быть недостаточен.

Используют **явную промежуточную модель**.

Django:

```python id="q7m4x2"
class Enrollment(models.Model):
    student = models.ForeignKey(
        Student,
        on_delete=models.CASCADE,
    )

    course = models.ForeignKey(
        Course,
        on_delete=models.CASCADE,
    )

    enrolled_at = models.DateTimeField(
        auto_now_add=True,
    )

    grade = models.IntegerField(
        null=True,
    )
```

И:

```python id="m9c3v8"
class Student(models.Model):
    courses = models.ManyToManyField(
        Course,
        through="Enrollment",
    )
```

Теперь промежуточная таблица — полноценная модель.

---

# 19. Association Object в SQLAlchemy

В SQLAlchemy аналогичный подход называется **association object pattern**.

Вместо простой:

```python id="x3n7q5"
secondary=student_course
```

создаётся отдельная ORM-модель:

```python id="c8m2v6"
class Enrollment(Base):
    __tablename__ = "enrollment"

    id: Mapped[int] = mapped_column(
        primary_key=True,
    )

    student_id: Mapped[int] = mapped_column(
        ForeignKey("student.id"),
    )

    course_id: Mapped[int] = mapped_column(
        ForeignKey("course.id"),
    )

    grade: Mapped[int | None]
```

Теперь объект связи может содержать собственные данные.

---

# 20. Many-to-Many vs One-to-Many

### One-to-Many

```text id="v4q8m2"
Author 1
   ↓
   N
Books
```

Один FK:

```text id="n7c3x5"
Book.author_id
```

### Many-to-Many

```text id="m2r9k6"
Students N
     ↕
Courses N
```

Нужна промежуточная таблица:

```text id="x5v8q3"
Student
   ↕
student_course
   ↕
Course
```

---

# 21. Many-to-Many и N+1

Например:

```python id="q3m7x9"
students = session.scalars(
    select(Student)
).all()

for student in students:
    print(student.courses)
```

Без eager loading может появиться:

```text id="k8n4v2"
1 SELECT → students
N SELECT → courses
```

То есть N+1.

В SQLAlchemy:

```python id="r6c2m8"
select(Student).options(
    selectinload(Student.courses)
)
```

В Django:

```python id="p9x3n5"
Student.objects.prefetch_related(
    "courses",
)
```

---

# 22. Row Multiplication

Many-to-Many особенно важно рассматривать с точки зрения JOIN.

Например:

```text id="w4m8q2"
Student 1
 ├── Course A
 ├── Course B
 └── Course C
```

JOIN создаёт:

```text id="n7c3x9"
Student 1 | Course A
Student 1 | Course B
Student 1 | Course C
```

При большом количестве связей результат может значительно увеличиться.

Поэтому для eager loading коллекций часто выбирают:

```text id="m5q8r2"
selectinload()
```

или:

```text id="x3v7n4"
prefetch_related()
```

---

# 23. Реальный пример

### Пользователь → роли

```text id="c8m2q6"
User
  ↕
user_role
  ↕
Role
```

Один пользователь:

```text id="p7x4n9"
Ilya
 ├── admin
 ├── editor
 └── manager
```

И одна роль:

```text id="v3m8k2"
editor
 ├── Ilya
 ├── Alex
 └── Maria
```

Это классический Many-to-Many.

---

# 24. Архитектурная схема

```text id="q6n3v8"
              MANY
   ┌────────────────────┐
   │      STUDENT       │
   └─────────┬──────────┘
             │
             │ 1:N
             ↓
   ┌────────────────────┐
   │  STUDENT_COURSE    │
   └─────────┬──────────┘
             │
             │ N:1
             ↓
   ┌────────────────────┐
   │       COURSE       │
   └────────────────────┘
```

И с точки зрения всей связи:

```text id="m8r4x2"
Student N ↔ N Course
```

---

# 🔥 Главное для собеседования

### Что такое Many-to-Many?

> Связь, при которой одна запись может быть связана с множеством записей другой таблицы, и наоборот.

### Как реализуется в БД?

**Через промежуточную таблицу с двумя Foreign Key.**

```text id="z5c8m3"
Student
   ↕
student_course
   ↕
Course
```

### Django

```python id="n4x7q2"
courses = models.ManyToManyField(
    Course,
)
```

### SQLAlchemy

```python id="v8m3k6"
courses = relationship(
    secondary=student_course,
)
```

### Eager loading

```text id="c2q7n9"
Django
Many-to-Many
     ↓
prefetch_related()
```

```text id="m5x8r3"
SQLAlchemy
Many-to-Many
     ↓
selectinload()
```

### Если у связи есть свои данные

Используем **явную промежуточную модель**:

```text id="p7n4v2"
Student
   ↓
Enrollment
   ↓
Course
```

В `Enrollment` можно хранить:

* `enrolled_at`;
* `grade`;
* `status`;
* другие атрибуты связи.

---

## 🧠 Финальная формула

```text id="x8m3q7"
             Many-to-Many
                   ↓
          промежуточная таблица
             ↙           ↘
          FK               FK
           ↓                ↓
       Student            Course
           ↕                ↕
          MANY             MANY
```

```text id="r4n8c2"
Django
ManyToManyField
      ↓
prefetch_related()
```

```text id="v7m2x5"
SQLAlchemy
relationship(secondary=...)
      ↓
selectinload()
```

**Самая короткая версия для собеседования:**

> Many-to-Many — это связь «многие-ко-многим». В реляционной БД она реализуется через промежуточную таблицу, содержащую Foreign Key на обе связанные таблицы. В Django используется `ManyToManyField`, а в SQLAlchemy — `relationship()` с `secondary`. Для eager loading коллекции обычно применяют `prefetch_related()` в Django и `selectinload()` в SQLAlchemy.
