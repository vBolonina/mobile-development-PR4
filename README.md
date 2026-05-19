## Практическая работа №4. Работа с встроенной базой данных SQLite.
Выполнила: Болонина Валерия Романовна ИНС-б-о-24-2
### Цель работы
Изучить основы работы с СУБД SQLite в Android-приложениях. Научиться создавать базу данных, таблицы, выполнять основные операции CRUD (Create, Read, Update, Delete) с использованием класса SQLiteOpenHelper и отображать данные на экране.

### Ход работы
#### Задание 1: Создание класса-помощника SQLHelper
  1. Был открыт Android Studio и создан новый проект с шаблоном **Empty Views Activity**. Проекту дано имя `SQLiteLab`.
  2. В папке `java/com.example.sqlitelab` создан новый Java-класс с именем `SQLHelper`.
  3. Сделано так, чтобы `SQLHelper` наследовался от `SQLiteOpenHelper`.
  4. Добавлен конструктор класса. Определены константы для названия базы данных, версии и таблицы.
  ##### Код для `SQLHelper`
  ```java
    public class SQLHelper extends SQLiteOpenHelper {
        // Константы для базы данных и таблицы
        public static final String DATABASE_NAME = "university.db";
        public static final int DATABASE_VERSION = 1;
        public static final String TABLE_NAME = "students";

        // Константы для названий столбцов
        public static final String COLUMN_ID = "_id";
        public static final String COLUMN_NAME = "name";
        public static final String COLUMN_AGE = "age";

        // SQL запрос для создания таблицы
        private static final String CREATE_TABLE = "CREATE TABLE " + TABLE_NAME + " ("
                + COLUMN_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, "
                + COLUMN_NAME + " TEXT NOT NULL, "
                + COLUMN_AGE + " INTEGER);";

        public SQLHelper(Context context) {
            super(context, DATABASE_NAME, null, DATABASE_VERSION);
        }

        @Override
        public void onCreate(SQLiteDatabase db) {
            // Выполняем создание таблицы при первом создании БД
            db.execSQL(CREATE_TABLE);
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            // При обновлении версии удаляем старую таблицу и создаём новую
            db.execSQL("DROP TABLE IF EXISTS " + TABLE_NAME);
            onCreate(db);
        }
    }
  ```

#### Задание 2: Создание модели данных Person
  1. Создан новый Java-класс с именем `Person`
  2. Добавлены поля, конструктор, геттеры и сеттеры.
  ##### Код для `Person`
  ```java
    public class Person {
        private int id;
        private String name;
        private int age;

        public Person(int id, String name, int age) {
            this.id = id;
            this.name = name;
            this.age = age;
        }

        // Геттеры и сеттеры
        public int getId() { return id; }
        public void setId(int id) { this.id = id; }

        public String getName() { return name; }
        public void setName(String name) { this.name = name; }

        public int getAge() { return age; }
        public void setAge(int age) { this.age = age; }
    }
  ```

#### Задание 3: Реализация методов для работы с данными в SQLHelper
  Были добавлены методы для выполнения CRUD операций в класс `SQLHelper`. В методе `getAllStudents()` для предотвращения ошибки: `Value must be >= 0 but 'getColumnIndex' can be -1`, вместо `cursor.getColumnIndex()` был использован более безопасный метод `cursor.getColumnIndexOrThrow()`, позволяющий обнаружить ошибки в именах столбцов на этапе разработки, а не во время выполнения.
  ##### Новые методы
  ```java
    // Добавление записи (Create)
    public long addStudent(String name, int age) {
        SQLiteDatabase db = this.getWritableDatabase();
        ContentValues values = new ContentValues();
        values.put(COLUMN_NAME, name);
        values.put(COLUMN_AGE, age);
        long id = db.insert(TABLE_NAME, null, values);
        db.close();
        return id;
    }

    // Получение всех записей (Read)
    public ArrayList<Person> getAllStudents() {
        ArrayList<Person> studentList = new ArrayList<>();
        String selectQuery = "SELECT * FROM " + TABLE_NAME;
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor cursor = db.rawQuery(selectQuery, null);

        if (cursor.moveToFirst()) {
            do {
                int id = cursor.getInt(cursor.getColumnIndexOrThrow(COLUMN_ID));
                String name = cursor.getString(cursor.getColumnIndexOrThrow(COLUMN_NAME));
                int age = cursor.getInt(cursor.getColumnIndexOrThrow(COLUMN_AGE));
                studentList.add(new Person(id, name, age));
            } while (cursor.moveToNext());
        }
        cursor.close();
        db.close();
        return studentList;
    }

    // Обновление записи (Update)
    public int updateStudent(Person person) {
        SQLiteDatabase db = this.getWritableDatabase();
        ContentValues values = new ContentValues();
        values.put(COLUMN_NAME, person.getName());
        values.put(COLUMN_AGE, person.getAge());
        return db.update(TABLE_NAME, values, COLUMN_ID + " = ?",
                new String[]{String.valueOf(person.getId())});
    }

    // Удаление записи (Delete)
    public void deleteStudent(int id) {
        SQLiteDatabase db = this.getWritableDatabase();
        db.delete(TABLE_NAME, COLUMN_ID + " = ?",
                new String[]{String.valueOf(id)});
        db.close();
    }
  ```

#### Задание 4: Работа с базой данных в MainActivity
  В файле `activity_main.xml` создан простой интерфейс с кнопками для добавления и отображения данных, а также `LinearLayout` для динамического вывода списка студентов.
  ##### Код `activity_main.xml`
  ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="16dp">

        <Button
            android:id="@+id/btnAdd"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Добавить студента" />

        <Button
            android:id="@+id/btnShow"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Показать всех"
            android:layout_marginTop="8dp"/>

        <LinearLayout
            android:id="@+id/container"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical"
            android:layout_marginTop="16dp"/>

    </LinearLayout>
  ```
  ##### Код `MainActivity.java`
  ```java
    import android.os.Bundle;
    import android.view.View;
    import android.widget.Button;
    import android.widget.LinearLayout;
    import android.widget.TextView;
    import android.widget.Toast;
    import androidx.appcompat.app.AppCompatActivity;
    import java.util.ArrayList;

    public class MainActivity extends AppCompatActivity {

        private SQLHelper dbHelper;
        private LinearLayout container;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);

            dbHelper = new SQLHelper(this);
            container = findViewById(R.id.container);

            Button btnAdd = findViewById(R.id.btnAdd);
            Button btnShow = findViewById(R.id.btnShow);

            btnAdd.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    // Для примера добавляем тестовые данные
                    long id = dbHelper.addStudent("Иванов Иван", 20);
                    if (id != -1) {
                        Toast.makeText(MainActivity.this, "Студент добавлен с ID: " + id, Toast.LENGTH_SHORT).show();
                    } else {
                        Toast.makeText(MainActivity.this, "Ошибка добавления", Toast.LENGTH_SHORT).show();
                    }
                }
            });

            btnShow.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    displayAllStudents();
                }
            });
        }

        private void displayAllStudents() {
            ArrayList<Person> students = dbHelper.getAllStudents();
            container.removeAllViews(); // Очищаем контейнер перед обновлением

            if (students.isEmpty()) {
                TextView emptyView = new TextView(this);
                emptyView.setText("Список студентов пуст");
                container.addView(emptyView);
                return;
            }

            for (Person student : students) {
                TextView textView = new TextView(this);
                textView.setText(student.getId() + ": " + student.getName() + ", возраст " + student.getAge());
                textView.setTextSize(16);
                textView.setPadding(8, 8, 8, 8);
                container.addView(textView);
            }
        }
    }
  ```
  Приложение было запущено. При нажатии на кнопку "Добавить студента" добавляются тестовые записи, а кнопка "Показать всех" отображает их на экране.<br>
  <img width="393" height="865" alt="image" src="https://github.com/user-attachments/assets/0d56c579-e79f-4571-b695-22f7c8a47bf0" />

#### Задания для самостоятельного выполнения
**Вариант 11:** Швейная компания: Заказ (номер заказа, ФИО клиента, дата приёма, стоимость, статус).<br>
1. Спроектирована структура таблицы.
2. Реализован класс `SQLHelper2` для создания БД и таблицы.
3. Создана модель данных.
4. Реализованы методы `addOrder()`, `getAllOrders()`, `updateOrder()`, `deleteOrder()`.
5. Реализован простой интерфейс с возможностью:
- Добавлять новую запись (через диалог или отдельную активность).
- Просматривать список всех записей.
- Удалять запись по нажатию (например, долгое нажатие на элемент списка).
- Обновлять запись.<br>
![Рисунок 2 - Результат задания для самостоятельного выполнения](images/image_2.png)

### Вывод
В результате выполнения практической работы были изучены основы работы с СУБД SQLite в Android-приложениях. Получены навыки создавать базу данных, таблицы, выполнять основные операции CRUD (Create, Read, Update, Delete) с использованием класса SQLiteOpenHelper и отображать данные на экране.

### Ответы на контрольные вопросы
**1. Какие типы данных поддерживает SQLite? Как в SQLite можно хранить логические значения и даты?**

SQLite поддерживает: `NULL`, `INTEGER` (целые числа), `REAL` (числа с плавающей точкой), `TEXT` (строки UTF-8/UTF-16), `BLOB` (бинарные данные), `NUMERIC` (универсальный тип).

**Логические значения** хранятся как `INTEGER`, где `1` = true, `0` = false.

**Даты** хранятся в трёх форматах:
- `TEXT` в формате ISO8601 ("YYYY-MM-DD HH:MM:SS.SSS")
- `INTEGER` — Unix timestamp (секунды с 1970-01-01)
- `REAL` — юлианские дни

---

**2. Для чего нужен класс `SQLiteOpenHelper`? Опишите назначение методов `onCreate()` и `onUpgrade()`.**

`SQLiteOpenHelper` — абстрактный класс для управления созданием и обновлением БД.

**`onCreate(SQLiteDatabase db)`** — вызывается при первом создании базы данных. Здесь выполняются SQL-запросы для создания таблиц.

**`onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion)`** — вызывается при увеличении версии БД. Используется для миграции данных при изменении структуры таблиц.

---

**3. В чем разница между методами `getWritableDatabase()` и `getReadableDatabase()`? В каких ситуациях может возникнуть ошибка при вызове `getWritableDatabase()`?**

**`getWritableDatabase()`** — возвращает БД для чтения и записи.

**`getReadableDatabase()`** — возвращает БД только для чтения (может работать, когда диск переполнен).

**Ошибка при вызове `getWritableDatabase()`** может возникнуть, если:
- На диске недостаточно места
- БД заблокирована другим процессом
- Отсутствуют права на запись

---

**4. Что такое `Cursor`? Как правильно перемещаться по его элементам и почему важно закрывать его после использования?**

`Cursor` — интерфейс для доступа к результирующему набору запроса.

**Перемещение:**
- `moveToFirst()` — перейти к первой строке
- `moveToNext()` — перейти к следующей строке
- `isAfterLast()` — проверка на конец

**Пример:**
```java
if (cursor.moveToFirst()) {
    do {
        String name = cursor.getString(cursor.getColumnIndexOrThrow("name"));
    } while (cursor.moveToNext());
}
```

**Важно закрывать** вызовом `cursor.close()`, чтобы освободить ресурсы и избежать утечек памяти.

---

**5. Что такое `ContentValues` и для каких операций он применяется?**

`ContentValues` — контейнер для пар "ключ-значение", где ключ — имя столбца, значение — данные.

**Применяется для:**
- **Вставки** (`insert()`)
- **Обновления** (`update()`)

**Пример:**
```java
ContentValues values = new ContentValues();
values.put("name", "Иванов");
values.put("age", 20);
```

---

**6. В чем отличие методов `query()` и `rawQuery()`? Приведите пример использования `rawQuery()` с параметром-плейсхолдером (?).**

**`query()`** — конструктор запроса с параметрами (таблица, столбцы, условие и т.д.).

**`rawQuery()`** — выполнение сырого SQL-запроса (полный контроль над SQL).

**Пример `rawQuery()` с плейсхолдером:**
```java
Cursor cursor = db.rawQuery("SELECT * FROM students WHERE age > ?", 
                           new String[]{"18"});
```

---

**7. Как обработать ситуацию, когда таблица уже существует, но её структура была изменена (например, добавлено новое поле)?**

**В методе `onUpgrade()`:**
1. Увеличить `DATABASE_VERSION`
2. Удалить старую таблицу: `db.execSQL("DROP TABLE IF EXISTS " + TABLE_NAME)`
3. Создать новую: `onCreate(db)`

**Пример:**
```java
@Override
public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
    db.execSQL("DROP TABLE IF EXISTS " + TABLE_NAME);
    onCreate(db);
}
```

**Альтернатива** (сохранение данных): выполнить `ALTER TABLE` для добавления нового поля.
