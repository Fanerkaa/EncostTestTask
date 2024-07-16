# EncostTestTask
**Тестовое задание на позицию - IT Специалист технической поддержки SQL, Python**

### Задание №1

**Клиент не получил ежедневный почтовый отчет (excel-файл, который формируется на нашем сервере с помощью python, содержит в себе данные о простоях и работе).**

- Какой по вашему мнению алгоритм решения проблемы?

*Проверить конфигурацию почтового сервера, чтобы убедиться, что он правильно настроен для отправки отчетов и корректно работает. Глянуть код Python, который отвечает за формирование и отправку отчета, что код не содержит ошибок, которые могли бы привести к не отправке писем, посмотреть журналы ошибок на сервере и в коде Python*

- Что Вы ответите клиенту?
  
*Здравствуйте, работаем над устранением ошибки, отправили вам в ручную excel-файл.*

- Что делать если эта ситуация повторяется несколько раз?
  
*Провести более глубокий анализ возможных причин.*

### Задание №2

**К тестовому заданию приложена база данных SQLite - client, в ней есть 3 таблицы:**

- endpoints – представляет станки на предприятии
- endpoints_reasons – представляет причины простоя
- endpoints_groups – представляет участки на предприятии

**Требуется выполнить следующие задания:**

- Написать запрос, который выводит причины простоя только активных станков.
```SQL
select endpoints.name, endpoint_reasons.reason_hierarchy 
from endpoints
join endpoint_reasons on endpoints.id = endpoint_reasons.endpoint_id
where endpoints.active = 'true'
```

- Написать запрос, который выводит количество причин простоев для каждой неактивной точки
```SQL
select endpoints.id, count(endpoint_reasons.reason_hierarchy) as reasons_count
from endpoints
join endpoint_reasons on endpoints.id = endpoint_reasons.endpoint_id
where endpoints.active = 'false'
group by endpoints.id
```

- Написать запрос, который выведет для каждого активного оборудования, количество причины простоя “ Перебои напряжения” (нужно учесть что это надо сделать для каждой группы(reason_hierarchy))
```SQL
select endpoints.id, count(endpoint_reasons.reason_name) as reasons_count
from endpoints
join endpoint_reasons on endpoints.id = endpoint_reasons.endpoint_id
where endpoints.active = 'true' and endpoint_reasons.reason_name = 'Перебои напряжения'
group by endpoints.id, endpoint_reasons.reason_name
```

### Задание №3

**Клиент просит сделать интеграцию с 1С.Предприятие(в 95% случаев клиентские запросы сначала попадают к Вам):**

- Что Вы ему ответите?

*Да мы можем сделать интеграцию с 1С.Предприятие, что именно вас интересует?*

- Ваш план действий после того как Вы ему ответили?

*Ознакомиться подробно с тем, что хочет клиент и помочь ему*

### Задание №4

**Произошло серьезное падение сервера, которое продлилось несколько часов, у множества клиентов не было данных за этот период:**

- Что Вы ответите клиенту?

*Здравствуйте, да упали сервера, приносим свои извинения, уже работаем над проблемой.  
Могло произойти такое, что какой-то клиент сам положил сервер или же это может не зависеть от вас (например DDoS атака)*

### Задание №5

**Клиент не может попасть в личный кабинет (клиенту предоставляется логин/пароль), ваши действия?**

*Спросить какую выдаёт ошибку, посмотреть логи, указать на то, как сбросить/поменять пароль(если такая возможность присутствует) и помочь, не передавая персональные данные (логин/пароль)* 
**152-ФЗ(может плохо кончится это)**

### Задание №6

**Есть огромный текстовый файл (более 100 ГБ - он точно не поместится в оперативной памяти), состоящий из строк, как его оптимально обрабатывать? - Напишите код на python.**
```python
import pandas as pd

file_path = 'encost_test.txt'

# Функция для обработки каждой строки
def process_line(line):
    processed_line = line
    return processed_line

# Чтение файла в DataFrame
df = pd.read_csv(file_path, header=None, names=['text'])

# Обработка каждой строки
df['processed_text'] = df['text'].apply(process_line)

# Запись обработанных строк в другой файл
df['processed_text'].to_csv('encost.txt', index=False, header=False, mode='a')
```

### Задание №7

**Что бы вы ответили клиенту? С чего бы начали проверку?**

![Screenshot](https://github.com/Fanerkaa/EncostTestTask/blob/main/encost.png)

*Мы постараемся разобраться в ситуации, я буду держать вас в курсе новостей*

### Задание №8

**Напишите Python скрипт который будет писать в базу данных client следующую информацию:**
- Добавить 3 станка: “Сварочный аппарат №1”, “Пильный аппарат №2”, “Фрезер №3”, сделать их активными.
- Скопировать со станков: “Фрезерный станок”, “Старый, ЧПУ”, “Сварка”, причины простоя и перенести их на новые станки.
- Определить группу “Цех №2” для новых станков.
- Добавить станки “Пильный станок” и “Старый ЧПУ” к новой группе.

```python
import sqlite3

# Подключаемся к базе данных
conn = sqlite3.connect('client.sqlite')
cursor = conn.cursor()

#1 Добавляем новые станки и делаем их активными
new_machines = [
    (7, "Сварочный аппарат №1", "True"),
    (8, "Пильный аппарат №2", "True"),
    (9, "Фрезер №3", "True")
]

cursor.executemany('INSERT INTO endpoints (id, name, active) VALUES (?, ?, ?)', new_machines,)

#2 Получаем информацию о причинах простоя существующих станков и переносим их на новые станки
existing_machines = ["Фрезерный станок", "Старый ЧПУ", "Сварка"]
num = 0

for machine in existing_machines:
    cursor.execute('SELECT endpoint_reasons.reason_hierarchy FROM endpoint_reasons \
    JOIN endpoints ON endpoints.id = endpoint_reasons.id \
    WHERE endpoints.name = ?',
                   (machine,))

    downtime_reason = cursor.fetchone()[0]  # Извлечение значения из кортежа

    cursor.execute('INSERT INTO endpoint_reasons (endpoint_id, reason_name, reason_hierarchy) \
    VALUES (?, ?, ?)',
    (new_machines[num][0], new_machines[num][1], downtime_reason))

    num += 1

#3 Определить группу “Цех №2” для новых станков.
num_groups = 0
name_group = "Цех №2"
for i in range(len(new_machines)):
    cursor.execute('INSERT INTO endpoint_groups (endpoint_id, name) \
                   VALUES(?, ?)',
                   (new_machines[num_groups][0], name_group))
    num_groups += 1


#4 Добавляем станки "Пильный станок" и "Старый ЧПУ" к группе "Цех №2"
add_machines = [("Пильный станок"),
                ("Старый ЧПУ")
                ]
for machines_4 in add_machines:
    cursor.execute('SELECT endpoint_groups.id FROM endpoints \
    JOIN endpoint_groups ON endpoints.id = endpoint_groups.id \
    WHERE endpoints.name = ?', (machines_4,))

    endpoint_id = cursor.fetchone()[0]
    cursor.execute('UPDATE endpoint_groups SET name = "Цех №2" WHERE endpoint_id = ?', (endpoint_id,))

# Сохраняем изменения и закрываем соединение с базой данных
conn.commit()
conn.close()
```




