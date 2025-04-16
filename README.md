## Part 1
Підготовка до проекту: короткий опис та інструкція по налаштуванню середовища
## Intel Image Classification
link on Kaggle -> https://www.kaggle.com/datasets/puneet6060/intel-image-classification?resource=download
## Content
6 класів, які містять зображення наступних типів місцевостей:
  - buildings
  - forest
  - glacier
  - mountain
  - sea
  - street
## Overall
Будемо мати справу з глибоким навчанням для класифікації зображень на наші 6 класів. Модель буде використовувати згорткові нейронні мережі (CNN) для автоматичної класифікації зображень. Протягом реалізації метою буде покращення точності класифікації на основі різних методів попередньої обробки зображень, аугментації даних та оптимізації гіперпараметрів.
## Налаштування середовища
Виконуємо наступну команду для запуску цього проекту - встановлення необхідних бібліотек:

`pip install -r requirements.txt`

## Part 2
Побудова базової моделі нейронної мережі.
Вхідні зображення мають розмір 150x150, а кількість каналів — 3 (RGB). 
Після трьох згорткових шарів з пулінгом отримуємо вектор для подальшої класифікації.
На цьому етапі досягнута Train Accuracy ≈ 91%, Validation Accuracy ≈ 83%, що є хорошим результатом для простої моделі.

