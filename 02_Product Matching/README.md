# Matching. Поиск наиболее похожих товаров

## Задача проекта:

Разработать алгоритм, который для всех товаров из validation.csv предложит 5 вариантов наиболее похожих товаров из base.csv.   
  
Оценка качества алгоритма производится по метрике **accuracy@5**.   


## Описание данных:

* **base.csv** - анонимизированный набор товаров. Каждый товар представлен как уникальный id (0-base, 1-base, 2-base) и вектор признаков размерностью 72.

* **train.csv** - обучающий датасет. Каждая строчка - один товар, для которого известен уникальный id (0-query, …, 100000-query), вектор признаков и id товара из base.csv, который максимально похож на него по мнению экспертов.

* **validation.csv** - датасет с товарами (уникальный id и вектор признаков), для которых надо найти наиболее близкие товары из base.csv.

* **validation_answer.csv** - правильные ответы к датасету с товарами для поиска (даётся для оценки работы простроенной модели).

При решении использовались библиотеки и модели ML:  

* **Kmeans++**
* **KNN**
* **Faiss**

## План исследования:

1. Загрузка библиотек и датасетов
2. Изучение данных в датасетах
3. Предобработка данных (проверка типов данных, пропущенных значений, явных дублей)
4. Исследовательский анализ данных (изучить основную информацию о данных, построить графики, проверить корреляцию признаков)
5. Решение задачи с помощью Kmeans++ и KNN
6. Решение задачи с помощью Faiss:
* Baseline Faiss FlatL2  и Faiss IVFFlat c метрикой Euclidean (без регулязации числовых признаков);
* Регулязация признаков с помощью RobustScaler():
  1. Faiss FlatL2 (Euclidean)
  2. Faiss IVFFlat FlatL2 (Euclidean)
  3. Faiss IVFFlat FlatL2 + nprobe (Euclidean)
  4. Faiss FlatIP (Cosine)
  5. Faiss Flat (Manhattan)
  6. Faiss IVFFlat FlatIP (Cosine)

* Регулязация признаков с помощьюStandardScaler():
  1. Faiss FlatL2 (Euclidean)
  2. Faiss IVFFlat FlatL2 (Euclidean)
  3. Faiss IVFFlat FlatL2 + nprobe (Euclidean)
  4. Faiss FlatIP (Cosine)
  5. Faiss IVFFlat FlatIP (Cosine)
  6. Faiss Flat (Manhattan)

7. Анализ результатов, выбор лучшей модели с лучшим показателем accuracy@5
8. Тестирование лучшей модели на данных validation.csv, замер accuracy@5
9. Вывод
10. Что можно было бы улучить в проекте

## Решение

В процессе исследования увидели, что плоские модели без кластеризации дают наилучший результат (Accuracy@5):

Faiss Flat (Manhattan) с регулязацией RobustScaler: 70.488
Faiss Flat (Manhattan) с регулязацией StandardScaler: 70.351
FlatL2 (Euclidean) с регулязацией RobustScaler: 70.068
Однако эти модели требуют больше времени для поиска похожих товаров.

Если очень важен критерий времени, можно использовать модели Faiss IVFFlat, основанные на кластеризации, лучшие результаты были получены при добавлении параметра nprobe:

IVFFlat + nprobe (Euclidean) с регулязацией RobustScaler: 69.915
IVFFlat + nprobe (Euclidean) с регулязациейStandardScaler: 69.395
  
Выбрали лучшую модель: Faiss Flat (Manhattan) с регулязацией RobustScaler, обучили ее на тестовой выборке valid_data и нашли топ-5 наиболее похожих товаров из базового набора base_data.
  
Получили довольно высокий accuracy@5 - 70.335, что говорит о хорошем качестве модели, ее точности в предсказании похожих товаров (примерно 3-4 товара из 5 предсказываются верно).

## Используемые библиотеки

Pandas, NumPy, Matplotlib, Seaborn, Sklearn, Faiss