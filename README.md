# Parsing-calories
Collect information from the website on the caloric content of products and their composition
# Парсинг на Python - сайт с калориями продуктов

![Screenshot from 2023-09-06 06-10-00.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/7e4285c8-c355-457c-86d7-9b6905c7adce/2bce39a2-8943-4314-93d0-a9115afb72e0/Screenshot_from_2023-09-06_06-10-00.png)

### Предисловие:

В данном проекте я осуществляю парсинг сайта с информацией по калориям продуктов https://health-diet.ru/table_calorie/ .

П**арсинг** – **это** автоматический сбор данных по конкретным параметрам или под какие-то задачи. Соответственно, **парсеры** – специальные сервисы для автоматического сбора данных. Собирать информацию можно практически из любых источников. Там где вы можете вычленить данные вручную, там можно использовать и **парсинг**, главное подобрать правильный инструмент для этого

Решение данной задачи было в качестве учебного проекта, чтобы еще больше отточить свои навыки в парсинге сайтов. 

*На каждый пунктик и скрин на каждое действие не делал, поскольку данный проект представлен как презентация, а не учебное пошаговое пособие.*

### В данном проекте мы используем:

Язык программирования - Python;

Библиотеки ЯП -  requests, BeautifulSoup

Модули ЯП - random, time

Сохраняем файлы в формат - Json, CSV, html


### Алгоритм действий:

- Сначала забираем страницу с сайта и после оперируем ей
    
    ```python
    import random
    from time import sleep
    import requests
    from bs4 import BeautifulSoup
    import json
    import csv
    
    # **Сохраняем копию страницы сайта на которой расположены все ссылки на категории и после работаем с ней** #
     url = "https://health-diet.ru/table_calorie/"
     #
     headers = {
         "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.",
         "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36"
     }
    # # #
     req = requests.get(url, headers=headers)
     src = req.text
     print(src)
    
     with open("index.html", "w", encoding="utf-8") as file:
         file.write(src)
     
    ```
    
- Проверяем ссылки сохраненной страницы, собираем их по общему классу, добавляем доменное имя
    
    ```python
    with open("index.html", encoding="utf-8") as file:
        src = file.read()
    
    soup = BeautifulSoup(src, "lxml")
    all_products_hrefs = soup.find_all(class_="mzr-tc-group-item-href")
    
    for item in all_products_hrefs:
        # print(item)
        item_text = item.text
        item_href = "https://health-diet.ru" + item.get("href")
        # print(f"{item_text}: {item_href}")
     
    ```
    
- Сохраняем в словарь
    
    сохраняем данные в словарь, создаем переменную `all_categories_dict = {}` и на каждой итерации цикла будем наполнять наш словарик  `all_categories_dict[item_text] = item_href` в котором ключи, это имена категорий, а значения это ссылки `[item_text] = item_href`
    
    ```python
    
     all_categories_dict = {}
    for item in all_products_hrefs:
        # print(item)
        item_text = item.text
        item_href = "https://health-diet.ru" + item.get("href")
        # print(f"{item_text}: {item_href}")
    
        all_categories_dict[item_text] = item_href
    ```
    
- Сохраним наш словарь в json файл
    
    <aside>
    💡 очень часто бывает необходимо сохранить данный в json файл и сохранение таких файлов съкономит вам поиск информации в интернете
    
    </aside>
    
    ```python
    
    with open("index.html", encoding="utf-8") as file:
        src = file.read()
    
    soup = BeautifulSoup(src, "lxml")
    all_products_hrefs = soup.find_all(class_="mzr-tc-group-item-href")
    
    all_categories_dict = {}
    for item in all_products_hrefs:
        # print(item)
        item_text = item.text
        item_href = "https://health-diet.ru" + item.get("href")
        # print(f"{item_text}: {item_href}")
    
        all_categories_dict[item_text] = item_href
    
    with open("all_categories_dict.json", "w", encoding="utf-8") as file:
        json.dump(all_categories_dict, file, indent=4, ensure_ascii=False)
    ```
    
- Закомментируем код выше и загрузим наш файл в переменную `all_categories`
    
    ```python
    with open("all_categories_dict.json") as file:
        all_categories = json.load(file)
    ```
    
    Убедились, что все ок 
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2a9407a4-684f-435f-b1c2-dbdf004db203/Untitled.png)
    
- Создаем цикл на каждой итерации которого мы будем заходить на новую страницу категории, собирать с нее данные о продуктах и хим. составе и записывать все в файл. Также пробелы из знаки препинания заменим на нижний слэш, для более лаконичного составления имен
    
    ```python
    iteration_count = int(len(all_categories)) - 1
    count = 0
    print(f"Всего итераций: {iteration_count}")
    
    for category_name, category_href in all_categories.items():
    ```
    
    ```python
    # метод replace для замены нескольких символов сразу к нескольким элементам
    rep = [",", " ", "-", "'"]
         for item in rep:
             if item in category_name:
                 category_name = category_name.replace(item, "_")
    # если символ есть в имени, то меняем его на нижний слэш
    ```
    
- Переходим к запросам на странице, расскоментируем заголовки (headers), чтобы мы могли снова их использовать. Создадим папку data, чтобы сохранять все страницы туда и сделаем счетчик для категорий (смотри выше в п.6 где count = 0)
    
    ```python
    req = requests.get(url=category_href, headers=headers)
        src = req.text
    # сохраним в переменную src 
    with open(f"data/{count}_{category_name}.html", "w") as file:
            file.write(src)
    ```
    
    Но прежде чем бомбить сайт запросами, протестируем наш код на одной странице с помощью if  
    
    ```python
    count = 0
    for category_name, category_href in all_categories.items():
        if count == 0:
            rep = [",", " ", "-", "'"]
            for item in rep:
                if item in category_name:
                    category_name = category_name.replace(item, "_")
        # print(category_name)
    #
        req = requests.get(url=category_href, headers=headers)
        src = req.text
    #
        with open(f"data/{count}_{category_name}.html", "w") as file:
            file.write(src)
      count += 1
    # count += 1 после всех операций увеличиваем имя файла на 1
    ```
    
    ```python
    #  откроем и сохраним код страницы в переменную src
    with open(f"data/{count}_{category_name}.html") as file:
            src = file.read()
    # Создаем объект супа
    soup = BeautifulSoup(src, "lxml")
    # смотрим на сайте заголовки для таблицы калорий
      
    ```
    
    ```python
     # собираем заголовки таблицы
            # поскольку мы получили список, мы можем обращаться к нему по индексам
    table_head = soup.find(class_="mzr-tc-group-table").find("tr").find_all("th")
        product = table_head[0].text
        calories = table_head[1].text
        proteins = table_head[2].text
        fats = table_head[3].text
        carbohydrates = table_head[4].text
    # Запишем в файлы csv категории продукты, с помощью метода writer.writerow передаем 5 объектов
        with open(f"data/{count}_{category_name}.csv", "w", encoding="utf-8") as file:
            writer = csv.writer(file)
            writer.writerow(
                (
                    product,
                    calories,
                    proteins,
                    fats,
                    carbohydrates
                )
            )
    
        count += 1
                )
            )
    ```
    
- Соберем данные продуктов в теге tbody из тегов tr
    
    и далее в цикле собререм все td теги 
    
    ```python
     products_data = soup.find(class_="mzr-tc-group-table").find("tbody").find_all("tr")
    
        product_info = []
        for item in products_data:
            product_tds = item.find_all("td")
    
            title = product_tds[0].find("a").text
            calories = product_tds[1].text
            proteins = product_tds[2].text
            fats = product_tds[3].text
            carbohydrates = product_tds[4].text
    ```
    
- Запишем снова в наши csv скопируем код с одним отличием, вместо “w” write (записать), поставим “a” , что значит добавить append
    
    ```python
     with open(f"data/{count}_{category_name}.csv", "a", encoding="utf-8") as file:
            writer = csv.writer(file)
            writer.writerow(
                (
                    product,
                    calories,
                    proteins,
                    fats,
                    carbohydrates
                )
            )
    ```
    
- При запуске кода, нам выпадет ошибка, поскольку по одной из ссылок, код не найдет таблицы, поэтому мы допишем  условие
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/31fa5538-0a0a-403a-ac50-d53eb0009dea/Untitled.png)
    
    ```python
     # проверка страницы на наличие таблицы с продуктами
        alert_block = soup.find(class_="uk-alert-danger")
        if alert_block is not None:
            continue
    ```
    
- Cоздадим переменную, в которую положим количество страниц категории. Так как наш count начинается с нуля , отнимем единицу от функции len предварительно обернув в строку, поскольку len возвращает объект строки
    
    ```python
    iteration_count = int(len(all_categories)) - 1
    count = 0
    print(f"Всего итераций: {iteration_count}")
    ```
    
- Для наглядности выведем операции на принт
    
    ```python
    print(f"# Итерация {count}. {category_name} записан...")
    # Отнимаем единицу от количества итераций
    iteration_count = iteration_count - 1
    # также допишем условие, когда итераций остается 0, печатаем Работа завершена и выходим из цикла
    if iteration_count == 0:
            print("Работа завершена")
            break
    print(f"Осталось итераций: {iteration_count}")
    # Поскольку сбор данных идет быстро, для наглядности добавим рандомную задержку
    sleep(random.randrange(2, 4))
    ```
    
- Также создадим json файл и в него мы будем собирать все данные из в виде словаря
    
    ```python
    #добавляем csv и json 
    product_info.append(
                {
                    "Title": title,
                    "Calories": calories,
                    "Proteins": proteins,
                    "Fats": fats,
                    "Carbohydrates": carbohydrates
                }
            )
    
            with open(f"data/{count}_{category_name}.csv", "a", encoding="utf-8") as file:
                writer = csv.writer(file)
                writer.writerow(
                    (
                        title,
                        calories,
                        proteins,
                        fats,
                        carbohydrates
                    )
                )
        with open(f"data/{count}_{category_name}.json", "a", encoding="utf-8") as file:
            json.dump(product_info, file, indent=4, ensure_ascii=False)
    ```
    
- Итоговые файлы
- Так выглядит директория проекта:

![Screenshot from 2023-09-06 06-03-06.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/7e4285c8-c355-457c-86d7-9b6905c7adce/647718e5-d573-4e19-abf5-5de0ae22a420/Screenshot_from_2023-09-06_06-03-06.png)

Так выглядит выгрузка в json файле:

![Screenshot from 2023-09-06 06-02-52.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/7e4285c8-c355-457c-86d7-9b6905c7adce/8ee9c08b-e064-46a2-ad8b-7ad0dab44dbc/Screenshot_from_2023-09-06_06-02-52.png)

Так выглядит выгрузка CSV файла:

![Screenshot from 2023-09-06 06-20-55.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/7e4285c8-c355-457c-86d7-9b6905c7adce/803649d2-14b5-47e8-8078-3ade5cd41025/Screenshot_from_2023-09-06_06-20-55.png)
