# Модуль 3. Об’єкти та конвеєр

У цьому модулі ви дізнаєтеся, чому PowerShell повертає не текст, а об’єкти, і навчитеся з’єднувати команди в конвеєр: сортувати, відбирати, фільтрувати, рахувати й гарно показувати результат. Це головна навичка, на якій тримається решта курсу.

| Урок | Тема | Час |
|---|---|---|
| [9](09-use-ye-obyektom.md) | Усе є об’єктом | ~30 хв |
| [10](10-konveyer.md) | Конвеєр «\|» | ~30 хв |
| [11](11-filtruyemo-dani.md) | Фільтруємо дані | ~30 хв |
| [12](12-vyvid-i-pidrakhunky.md) | Гарний вивід і підрахунки | ~30 хв |
| [Тест](#підсумковий-тест-модуля-3) | 5 питань і 5 практичних завдань | ~20 хв |

## Підсумковий тест модуля 3

Спершу виконайте все самостійно і лише потім розгортайте відповіді. Якщо помилилися, поверніться до уроку, вказаного в дужках.

### Питання

**1.** Яка команда покаже лише **методи** об’єкта дати? *(урок 9)*

**2.** Що виведе команда `(Get-Date).AddMonths` і чому? *(урок 9)*

**3.** Колега хоче отримати 5 найстаріших файлів у папці й пише так. Що не так? *(урок 10)*

```powershell
Get-ChildItem | Select-Object -First 5 | Sort-Object LastWriteTime
```

**4.** Після цієї команди в папці з’явився файл `1GB`, а на екрані порожньо. Що сталося? *(урок 11)*

```powershell
Get-ChildItem -Recurse | Where-Object { $_.Length > 1GB }
```

**5.** Чому в CSV-файлі після такої команди немає назв процесів? *(урок 12)*

```powershell
Get-Process | Format-Table Name, Id | Export-Csv процеси.csv
```

<details>
<summary>Відповіді на питання</summary>

1. `Get-Date | Get-Member -MemberType Method`.
2. Опис методу (`OverloadDefinitions`), а не дату: метод викликано без дужок і без значення. Правильно, наприклад, `(Get-Date).AddMonths(1)`.
3. Порядок команд: спочатку взято 5 перших файлів за абеткою, а потім відсортовано лише їх. Правильно: `Get-ChildItem | Sort-Object LastWriteTime | Select-Object -First 5`.
4. `>` — це запис у файл, а не «більше». Розміри записалися у файл `1GB`, а фільтр нічого не пропустив. Правильно: `-gt 1GB`.
5. `Format-Table` перетворив процеси на службові об’єкти для екрана. Для вибору стовпців перед `Export-Csv` потрібен `Select-Object Name, Id`.

</details>

### Практичні завдання

**Завдання 1.** Виведіть, яким днем тижня був ваш день народження цього року. Підставте свою дату у форматі `рррр-мм-дд`. *(урок 9)*

<details>
<summary>Підказка</summary>

`Get-Date` уміє не лише показувати поточну дату, а й створювати будь-яку: `Get-Date -Date "2026-05-17"`.

</details>

<details>
<summary>Розв’язок</summary>

```powershell
(Get-Date -Date "2026-05-17").DayOfWeek
```

</details>

**Завдання 2.** Виведіть 10 найменших за розміром файлів у папці `C:\Windows` (без вкладених папок): лише назву й розмір, від найменшого. *(урок 10)*

<details>
<summary>Розв’язок</summary>

```powershell
Get-ChildItem C:\Windows -File | Sort-Object Length | Select-Object -First 10 -Property Name, Length
```

</details>

**Завдання 3.** Знайдіть у «Документах» (разом із вкладеними папками) усі файли Word (`.docx`), змінені за останні 30 днів. *(урок 11)*

<details>
<summary>Розв’язок</summary>

```powershell
Get-ChildItem $HOME\Documents -Filter *.docx -Recurse |
    Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-30) }
```

Якщо «Документи» у вас в OneDrive, замініть шлях на `$HOME\OneDrive\Documents`.

</details>

**Завдання 4.** Порахуйте, скільки процесів Chrome (або Edge — `msedge`) зараз запущено і скільки пам’яті вони займають разом у мегабайтах. *(уроки 11 і 12)*

<details>
<summary>Розв’язок</summary>

```powershell
Get-Process -Name chrome | Measure-Object -Property WorkingSet -Sum
(Get-Process -Name chrome | Measure-Object -Property WorkingSet -Sum).Sum / 1MB
```

`Count` у першому результаті — кількість процесів. Браузери запускають багато процесів: окремо для вкладок і розширень.

</details>

**Завдання 5.** Покажіть у вікні `Out-GridView` 20 найбільших файлів з «Завантажень»: назва, розмір у МБ (округлений до цілого) і дата зміни. *(уроки 10 і 12)*

<details>
<summary>Розв’язок</summary>

```powershell
Get-ChildItem $HOME\Downloads -File |
    Sort-Object Length -Descending |
    Select-Object -First 20 Name, @{ Name = "МБ"; Expression = { [math]::Round($_.Length / 1MB) } }, LastWriteTime |
    Out-GridView
```

</details>

---
[← Урок 12](12-vyvid-i-pidrakhunky.md) · [Зміст курсу](../../README.md)
