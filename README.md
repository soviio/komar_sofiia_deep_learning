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
<img src="https://github.com/user-attachments/assets/e6f682de-636d-4b83-b082-236ff103ccbb" alt="image" width="550" />

## Архітектура моделі
`Conv2D(3 → 32) + ReLU → MaxPool(2x2)   # 150×150 → 75×75`

`Conv2D(32 → 64) + ReLU → MaxPool(2x2)  # 75×75 → 37×37`

`Conv2D(64 → 128) + ReLU → MaxPool(2x2) # 37×37 → 18×18`

`Flatten → FC(128*18*18 → 256) → Dropout(0.5) → FC(256 → 128) → FC(128 → 6)`

На цьому етапі маємо:
- 3 згорткові шари з поступовим збільшенням кількості фільтрів: 32 → 64 → 128
- Поступове зменшення розміру зображення: 150 → 75 → 37 → 18
- Повнозв’язані шари з дропаутом для регуляризації
- Оптимальну глибину для початкового експерименту з класифікацією

## Part 3
Тут оптимізуємо навчання базової CNN-моделі, оцінюємо вплив різних параметрів. Досліджуємо вплив кількості згорткових шарів та кількості нейронів у повнозв’язаних шарах.

## Експерименти з Learning Rate
Через нестачу обчислювальних ресурсів перевірити комбінації золотих параметрів (batch size, epochs, lr) повністю не вдалося. Проведено лише серію тестів по learning rate.
Після запуску циклу з трьома значеннями learning rate 0.0001 дало найвищу val_acc.

## Архітектурна оптимізація CNNModel1
Додано 4-й згортковий шар + MaxPooling → розмір зменшено до 8×8. Повнозв’язані шари стали глибшими, Dropout залишено на рівні 0.5.

`self.conv4 = nn.Conv2d(128, 256, kernel_size=3, padding=1)  # новий шар`

`self.fc1 = nn.Linear(256 * 8 * 8, 512)  # більша кількість нейронів`

`self.fc2 = nn.Linear(512, 256)`

Модель з 3-ма Conv шарами:
- Поступове поліпшення точності і F1-score
- Найкраща val accuracy ≈ 83.1% досягається на ~18–20 епосі
- Плавна збіжність loss та accuracy

Модель з 4-ма Conv шарами:
- Різке покращення вже на 5-й епосі
- Досягнення 92.1% точності всього за 8 епох
- Loss зменшується майже вдвічі за кожну другу епоху

## Optuna (для пошуку гіперпараметрів)
Модель базується на 4-х згорткових шарах:

`conv1 → conv2 → conv3 → conv4 → Flatten → FC(512) → FC(256) → Output(6)`

Найкраща конфігурація:

`{
    'batch_size': 32,
    'lr': 0.000473,
    'dropout_rate': 0.273,
    'epochs': 4
}`

- Optuna успішно підібрала параметри, які дозволили досягти validation accuracy ≈ 80.6%
- Найкраща точність на тренуванні за 4 епохи — 84.04%, при значно зниженому loss
- За 5 спроб Optuna знайшла набір гіперпараметрів, близький до оптимального

## Confusion Matrix
<img src="https://github.com/user-attachments/assets/15211984-0e83-415a-936a-a56b79edcedc" alt="image" width="350" />

Cлабкі місця:
- "glacier" ↔ "mountain" плутаються дуже часто → можна покращити розпізнавання текстур
- "street" ↔ "buildings" — ще одна зона з високою плутаниною

## Part 4
Тут досліджуємо Transfer Learning на основі переднавченої моделі ResNet50, що дозволяє повторно використовувати вже вивчені ознаки, зменшуючи обчислювальні витрати і час тренування. Модель адаптована до задачі класифікації 6 типів природних середовищ.

Ми адаптуємо fc-шар під наші 6 категорій (buildings, forest, glacier, mountain, sea, street):

`model.fc = nn.Sequential(`

    `nn.Linear(model.fc.in_features, 512),`
    
    `nn.ReLU(),`
    
    `nn.Dropout(0.5),`
    
   ` nn.Linear(512, 6)  # 6 класів`
   
`)`

## Confusion Matrix
<img src="https://github.com/user-attachments/assets/1aab2fa6-c6e8-41d1-a154-fb75ecaa42c4" alt="image" width="350" />

- "glacier" ↔ "mountain" — найчастіше плутаються (схожі візуально)
- "street" ↔ "buildings" — також мають схожі патерни
- найвища точність для "forest" та "sea"

## Висновок
Transfer Learning з ResNet50 суттєво скоротив час навчання без втрати точності.
Модель досягла високої класифікаційної здатності, тренуючи лише останні шари.
Цей підхід — ідеальний компроміс між якістю і ресурсами, особливо при обмеженому доступі до GPU (з яким я зіштовхнулась).
