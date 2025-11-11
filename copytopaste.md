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

sudo lsof -i tcp:5432
COMMAND      PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
docker-pr 158171 root    7u  IPv4 531005      0t0  TCP *:postgresql (LISTEN)
docker-pr 158178 root    7u  IPv6 531006      0t0  TCP *:postgresql (LISTEN)

Unable to connect to server:

connection failed: connection to server at "127.0.0.1", port 5432 failed: Connection refused
Is the server running on that host and accepting TCP/IP connections?
Multiple connection attempts failed. All failures were:
- host: 'localhost', port: '5432', hostaddr: '::1': connection failed: connection to server at "::1", port 5432 failed: Connection refused
Is the server running on that host and accepting TCP/IP connections?
- host: 'localhost', port: '5432', hostaddr: '127.0.0.1': connection failed: connection to server at "127.0.0.1", port 5432 failed: Connection refused
Is the server running on that host and accepting TCP/IP connections



Error: Database is uninitialized and superuser password is not specified.
       You must specify POSTGRES_PASSWORD to a non-empty value for the
       superuser. For example, "-e POSTGRES_PASSWORD=password" on "docker run".

       You may also use "POSTGRES_HOST_AUTH_METHOD=trust" to allow all
       connections without a password. This is *not* recommended.

       See PostgreSQL documentation about "trust":
       https://www.postgresql.org/docs/current/auth-trust.html


```
services:
  # MongoDB Service (версия 4.4 - стабильная)
  mongodb:
    container_name: mongodb
    image: mongo:4.4
    environment:
      MONGO_INITDB_ROOT_USERNAME: mongouser
      MONGO_INITDB_ROOT_PASSWORD: mongopass
    ports:
      - "27017:27017"
    volumes:
      - mongodb_volume:/data/db
    networks:
      - db_network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Mongo Express Service (Web UI for MongoDB)
  mongo-express:
    container_name: mongo-express
    image: mongo-express:latest
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=mongouser
      - ME_CONFIG_MONGODB_ADMINPASSWORD=mongopass
      - ME_CONFIG_BASICAUTH_USERNAME=adminuser
      - ME_CONFIG_BASICAUTH_PASSWORD=adminpasswd
      - ME_CONFIG_MONGODB_SERVER=mongodb
    ports:
      - "8081:8081"
    depends_on:
      mongodb:
        condition: service_healthy
    networks:
      - db_network
    restart: unless-stopped

  # PostgreSQL Service (версия 16)
  postgresql:
    container_name: postgresql
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: changeme
      POSTGRES_DB: studpg
    ports:
      - "5432:5432"
    volumes:
      - postgres_volume:/var/lib/postgresql/data
    networks:
      - db_network
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d studpg"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 30s

  # pgAdmin Service
  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: pgadmin4@pgadmin.org
      PGADMIN_DEFAULT_PASSWORD: admin
      PGADMIN_CONFIG_SERVER_MODE: 'False'
    ports:
      - "5050:80"
    depends_on:
      postgresql:
        condition: service_healthy
    networks:
      - db_network
    restart: unless-stopped

  # Jupyter Notebook Service
  jupyter:
    container_name: jupyter
    image: jupyter/scipy-notebook:latest
    environment:
      JUPYTER_ENABLE_LAB: yes
      JUPYTER_TOKEN: jupyter123
    ports:
      - "8888:8888"
    volumes:
      - ./:/home/jovyan/work
    networks:
      - db_network
    restart: unless-stopped
      
volumes:
  mongodb_volume:
  postgres_volume:

networks:
  db_network:
    driver: bridge
```
