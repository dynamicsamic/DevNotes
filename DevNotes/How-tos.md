---
Created: 2024-07-17T08:26
---
- If Django models lack `id` field when using MongoDB, you need to:
    1. Delete migration files and `/__pycahe__` directory from `/<your_app>/migrations` dir.
    2. Delete corrupted collections from MongoDB database.
    3. Apply `makemigrations` and `migrate` commands.