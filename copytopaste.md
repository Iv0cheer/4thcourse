```python
import psycopg2
import random
import os

def generate_files_data():
    try:
        pg_conn = psycopg2.connect(
            dbname="your_database",
            user="your_username",
            password="your_password",
            host="localhost"
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
        print("Таблица files создана или уже существует")
        

        print("Генерация 100 файлов по 1 МБ...")
        
        for i in range(100):
            file_size = 1024 * 1024  # 1 МБ (в Байт)
            binary_data = os.urandom(file_size)
            
            # Вставка данных в таблицу
            pg_cursor.execute(
                "INSERT INTO files (data, file_size) VALUES (%s, %s)",
                (psycopg2.Binary(binary_data), file_size)
            )
            
            if (i + 1) % 10 == 0:
                print(f"Сгенерировано и вставлено {i + 1} строк")
            else:
                print(f"Ошибка, сгенерировано и загружено {i + 1} строк")
        
        pg_conn.commit()
        print("Все 100 файлов успешно вставлены в таблицу")



        print("\nИзвлечение данных из таблицы...")
        pg_cursor.execute("SELECT id, file_size, LENGTH(data) as data_length FROM files")
        files = pg_cursor.fetchall()
        
        print(f"Извлечено {len(files)} записей:")
        

        pg_cursor.close()
        pg_conn.close()
        print("\nОперация завершена успешно!")
        
    except Exception as e:
        print(f"Ошибка: {e}")



if __name__ == "__main__":
print("=== Генерация данных для PostgreSQL таблицы files ===\n")

# Базовая версия
generate_files_data()

print("\n" + "="*50 + "\n")
```
