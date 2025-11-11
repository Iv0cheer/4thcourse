```python
import os
from pymongo import MongoClient
import gridfs
import datetime

def generate_and_store_files():
    # Данные для подключения
    host = 'localhost'
    port = 27017
    username = 'mongouser'
    password = 'mongopass'
    database_name = 'admin'  # или ваша основная база данных
    
    # Подключение к MongoDB с аутентификацией
    connection_string = f'mongodb://{username}:{password}@{host}:{port}/{database_name}'
    client = MongoClient(connection_string)
    
    # Проверка подключения
    try:
        client.admin.command('ping')
        print("✓ Успешное подключение к MongoDB")
    except Exception as e:
        print(f"✗ Ошибка подключения к MongoDB: {e}")
        return
    
    # Используем основную базу данных
    db = client[database_name]
    
    # Инициализация GridFS
    fs = gridfs.GridFS(db)
    
    successful_uploads = 0
    
    try:
        for i in range(100):
            file_size = 1024 * 1024  # 1 МБ (в Байт)
            binary_data = os.urandom(file_size)
            
            # Метаданные для файла
            metadata = {
                "upload_date": datetime.datetime.now(),
                "file_size": file_size,
                "file_number": i + 1,
                "description": f"Тестовый файл {i+1} размером 1 МБ"
            }
            
            # Вставка данных в GridFS с метаданными
            file_id = fs.put(
                binary_data, 
                filename=f"file_{i+1}.bin",
                metadata=metadata
            )
            
            # Проверка, что файл действительно сохранен
            if fs.exists(file_id):
                successful_uploads += 1
                if (i + 1) % 10 == 0:
                    print(f"✓ Сгенерировано и вставлено {i + 1} файлов (ID: {file_id})")
                else:
                    print(f"✓ Сгенерировано и загружено {i + 1} файлов")
            else:
                print(f"✗ Ошибка при сохранении файла {i + 1}")
        
        print(f"\nРезультат: успешно загружено {successful_uploads} из 100 файлов")
        
        # Статистика
        total_files = len(list(fs.find()))
        print(f"Всего файлов в GridFS: {total_files}")
        
    except Exception as e:
        print(f"Ошибка при работе с MongoDB: {e}")
    
    finally:
        client.close()
        print("Подключение к MongoDB закрыто")
```
