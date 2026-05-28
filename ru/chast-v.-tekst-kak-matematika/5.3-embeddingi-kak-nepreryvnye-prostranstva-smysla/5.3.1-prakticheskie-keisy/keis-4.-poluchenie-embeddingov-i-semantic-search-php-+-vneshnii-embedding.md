---
hidden: true
---

# Кейс 4. Получение embeddings и semantic search (PHP + внешний embedding)

Идея\
Показать полный pipeline, но без перегруза.

Сценарий\
– небольшой корпус текстов\
– внешний embedding (через API или локальную модель, если доступно)

Что делаем\
– получаем embeddings\
– сохраняем их в массив / файл\
– выполняем semantic search

Важно\
Фокус не на API, а на том, что код вокруг не меняется, даже если модель другая.\
\
TIP: используй [_Transformers PHP_.](https://www.google.com/search?sca_esv=cc34000cb1b7ea38\&biw=1920\&bih=968\&sxsrf=ANbL-n7KYRb7od1KfWoCq0vP2-vu4hLxMQ:1779828485405\&q=%D0%BA%D0%B0%D0%BA+%D0%B3%D0%B5%D0%BD%D0%B5%D1%80%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D1%82%D1%8C+%D1%8D%D0%BC%D0%B1%D0%B5%D0%B4%D0%B4%D0%B8%D0%BD%D0%B3%D0%B8+%D0%B2+php+Transformers+PHP.\&spell=1\&sa=X\&ved=2ahUKEwiM0Ijl6deUAxWv3gIHHSWsJ8YQkeECKAB6BAgPEAE) !!!!!!!!

