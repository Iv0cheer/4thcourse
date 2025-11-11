# **Лабораторная работа 4.1. **

**Цель:** Сравнить подходы к хранению и извлечению бинарных данных. Проанализировать производительность и удобство использования bytea и GridFS.

**Вариант:** 19

## Установка необходимых библиотек:
```python
!pip install psycopg2-binary
!pip install pymongo
```

Скриншот:
<img width="787" height="358" alt="image" src="https://github.com/user-attachments/assets/0964ba44-b61e-4b1c-8243-67a1a0c0d5c4" />


## Выполнение задания №1.
### Задание:
> Бинарные данные (файлы). Создать таблицу files с полем data типа bytea. Вставить и затем извлечь 100 файлов по 1 МБ. Измерить среднее время.

Код:
```python
import psycopg2
import random
import os

def generate_files_data():
    try:
        pg_conn = psycopg2.connect(
            dbname="studpg",
            user="student",
            password="Stud2024!!!",
            host="postgresql",
            port="5432"
        )
        pg_cursor = pg_conn.cursor()
        
        pg_cursor.execute("""
            CREATE TABLE IF NOT EXISTS files (
                id SERIAL PRIMARY KEY,
                data BYTEA,
                file_size INTEGER,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """)
        pg_conn.commit()
        print("Table files created (or exists)")
        

        print("Generating...")
        
        for i in range(100):
            file_size = 1024 * 1024
            binary_data = os.urandom(file_size)
            
            pg_cursor.execute(
                "INSERT INTO files (data, file_size) VALUES (%s, %s)",
                (psycopg2.Binary(binary_data), file_size)
            )
            
        
        pg_conn.commit()
        print("Insert ok")



        print("\nExporting data from table...")
        pg_cursor.execute("SELECT id, file_size, LENGTH(data) as data_length FROM files")
        files = pg_cursor.fetchall()

        pg_cursor.close()
        pg_conn.close()
        print("\nok")
        
    except Exception as e:
        print(f"Ошибка: {e}")



if __name__ == "__main__":
    print("=== Start generating [POSTGRESQL] ===\n")
    
    generate_files_data()
    
    print("\n" + "="*50 + "\n")
```

Скриншот выполнения:
<img width="810" height="428" alt="image" src="https://github.com/user-attachments/assets/9f6034e1-c614-483e-acb4-5a22bf91e8e2" />



## Выполнение задания №2.
### Задание:
> Бинарные данные (файлы). Использовать GridFS для хранения 100 файлов по 1 МБ. Измерить среднее время вставки и извлечения одного файла.

Код:
```python
import os
from pymongo import MongoClient
import gridfs
import datetime

def generate_files_mongo():
    host = 'localhost'
    port = 27017
    username = 'mongouser'
    password = 'mongopass'
    database_name = 'studmongo'
    
    client = MongoClient('mongodb://mongouser:mongopass@mongodb:27017/')

    try:
        client.admin.command('ping')
        print("Connect success")
    except Exception as e:
        print(f"Connect refused: {e}")
        return
    
    db = client['studmongo']
    
    fs = gridfs.GridFS(db)

    try:
        for i in range(100):
            file_size = 1024 * 1024
            binary_data = os.urandom(file_size)
            
            metadata = {
                "upload_date": datetime.datetime.now(),
                "file_size": file_size,
                "file_number": i + 1,
                "description": f"Тестовый файл {i+1} размером 1 МБ"
            }
            
            file_id = fs.put(
                binary_data, 
                filename=f"file_{i+1}.bin",
                metadata=metadata
            )

        print(f"success, created {i+1} files.bin")
        
    except Exception as e:
        print(f"Error with MongoDB: {e}")
    
    finally:
        client.close()
        print("=== Connection closed ===")

if __name__ == "__main__":
    
    print("\n=== Start generating [MONGODB] ===\n")
    
    generate_files_mongo()

    print("\n" + "="*50 + "\n")
```

Скриншот выполнения:
<img width="778" height="281" alt="image" src="https://github.com/user-attachments/assets/13a90d32-28af-4fdb-9259-3f8d1a1c3d8a" />


## Сравнение производительности:
### 1. Измерение времени выполнения запросов:

Код:
```python
if __name__ == "__main__":
    print("=== Start generating [POSTGRESQL] ===\n")

    start_time = time.time()
    generate_files_data() #eto postgresql
    end_time = time.time()
    postgresql_time = end_time - start_time
    
    print("\n" + "="*50 + "\n")
    
    print("\n=== Start generating [MONGODB] ===\n")

    start_time = time.time()
    generate_files_mongo() #eto mongo
    end_time = time.time()
    mongodb_time = end_time - start_time
    
    print("\n" + "="*50 + "\n")

    print(f"postgresql time = {postgresql_time} sec\nmongodb time = {mongodb_time} sec")

```

Результат выполнения:
<img width="741" height="587" alt="image" src="https://github.com/user-attachments/assets/66259ae5-8918-4a32-814d-4d72731f2d1e" />


### 2. Визуализация результатов

Код:
```python
import matplotlib.pyplot as plt

databases = ['PostgreSQL', 'MongoDB']
times = [postgresql_time, mongodb_time]

plt.figure(figsize=(10,6))
plt.bar(databases, times, color=['blue', 'red'])
plt.title('Proizvoditelnost baz dannih postgresql i mongodb')
plt.xlabel('database')
plt.ylabel('time (sec)')
plt.show()
```

Результат выполнения:
<img width="751" height="649" alt="image" src="https://github.com/user-attachments/assets/2e4ef898-443a-41aa-a3e9-3b6f2a2fe365" />

