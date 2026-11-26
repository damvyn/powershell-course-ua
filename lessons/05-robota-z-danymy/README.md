# Модуль 5. Робота з даними

У цьому модулі ви навчитеся читати й зберігати таблиці CSV для Excel, отримувати дані з сайтів у форматі JSON і розбиратися з XML. Усе, що ви вивчили про об’єкти, конвеєр і змінні, тут працює з реальними даними: витратами, курсами валют, налаштуваннями.

| Урок | Тема | Час |
|---|---|---|
| [17](17-csv-ta-excel.md) | CSV та Excel | ~30 хв |
| [18](18-json-i-dani-z-internetu.md) | JSON і дані з інтернету | ~30 хв |
| [19](19-xml-korotko.md) | XML коротко | ~20 хв |
| [Тест](#підсумковий-тест-модуля-5) | 5 питань і 5 практичних завдань | ~20 хв |

## Підсумковий тест модуля 5

Спершу виконайте все самостійно і лише потім розгортайте відповіді. Якщо помилилися, поверніться до уроку, вказаного в дужках.

### Питання

**1.** Ви прочитали таблицю `Import-Csv` і відсортували за стовпцем `Amount`, але сума 90 опинилася вище за 1200. Чому і як виправити? *(урок 17)*

**2.** Ви надіслали колезі CSV-файл, створений `Export-Csv -NoTypeInformation -Encoding UTF8`, а в Excel уся таблиця в одному стовпці. Що додати до команди? *(урок 17)*

**3.** Чому ця команда виводить усі валюти, а не лише євро? *(урок 18)*

```powershell
Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?json" | Where-Object { $_.cc -eq "EUR" }
```

**4.** Навіщо `-Raw` у команді `Get-Content курс.json -Raw -Encoding UTF8 | ConvertFrom-Json`? *(урок 18)*

**5.** Що робить `[xml]` перед змінною і чому курс з XML-файлу перед множенням треба записати як `[double]$usd.rate`? *(урок 19)*

<details>
<summary>Відповіді на питання</summary>

1. Значення з CSV — текст, а текст сортується за абеткою: `"9"` більше за `"1"`. Треба перетворити на число під час сортування: `Sort-Object { [int]$_.Amount }`.
2. `-UseCulture` (або `-Delimiter ';'`): Excel з українськими налаштуваннями чекає розділювач `;`.
3. `Invoke-RestMethod` передає конвеєром увесь список одним об’єктом, і умова виконується для списку цілком. Збережіть результат у змінну або візьміть команду в круглі дужки.
4. `-Raw` читає файл одним текстом, а `ConvertFrom-Json` потрібен увесь JSON разом.
5. `[xml]` перетворює текст на XML-документ, по якому можна пройти через крапки. Усі значення в XML — текст, тому для обчислень їх перетворюють на число.

</details>

### Практичні завдання

**Завдання 1.** Збережіть для Excel десять процесів, що займають найбільше пам’яті: назва і пам’ять у МБ, округлена до цілого. Відкрийте файл в Excel. *(урок 17)*

<details>
<summary>Розв’язок</summary>

```powershell
Get-Process |
    Sort-Object WorkingSet -Descending |
    Select-Object -First 10 Name, @{ Name = "MemoryMB"; Expression = { [math]::Round($_.WorkingSet / 1MB) } } |
    Export-Csv "$HOME\PS-Kurs\процеси.csv" -NoTypeInformation -Encoding UTF8 -UseCulture
Invoke-Item "$HOME\PS-Kurs\процеси.csv"
```

`WorkingSet` — це пам’ять, яку процес зараз займає, у байтах.

</details>

**Завдання 2.** Прочитайте `витрати.csv` з уроку 17 і виведіть середню витрату, округлену до цілого, та опис найдорожчої покупки. *(урок 17)*

<details>
<summary>Розв’язок</summary>

```powershell
$expenses = Import-Csv "$HOME\PS-Kurs\витрати.csv"
[math]::Round(($expenses | Measure-Object Amount -Average).Average)
($expenses | Sort-Object { [int]$_.Amount } -Descending | Select-Object -First 1).Description
```

</details>

**Завдання 3.** Збережіть у файл `мої-валюти.json` курси долара, євро і злотого (лише `cc` і `rate`), а потім прочитайте файл і виведіть курс злотого. *(урок 18)*

<details>
<summary>Розв’язок</summary>

```powershell
$rates = Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?json"
$rates |
    Where-Object { $_.cc -in "USD", "EUR", "PLN" } |
    Select-Object cc, rate |
    ConvertTo-Json |
    Set-Content "$HOME\PS-Kurs\мої-валюти.json" -Encoding UTF8

$saved = Get-Content "$HOME\PS-Kurs\мої-валюти.json" -Raw -Encoding UTF8 | ConvertFrom-Json
($saved | Where-Object { $_.cc -eq "PLN" }).rate
```

</details>

**Завдання 4.** Отримайте курси всіх валют НБУ і збережіть їх для Excel у файл `курси.csv`: код, назва і курс, від найдорожчої валюти до найдешевшої. *(уроки 17 і 18)*

<details>
<summary>Розв’язок</summary>

```powershell
$rates = Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?json"
$rates |
    Sort-Object rate -Descending |
    Select-Object cc, txt, rate |
    Export-Csv "$HOME\PS-Kurs\курси.csv" -NoTypeInformation -Encoding UTF8 -UseCulture
Invoke-Item "$HOME\PS-Kurs\курси.csv"
```

Тут `[double]` для сортування не потрібен: дані з JSON уже числа.

</details>

**Завдання 5.** Прочитайте файл `курси.xml` з уроку 19 і порахуйте, скільки валют мають курс понад 10 грн. *(урок 19)*

<details>
<summary>Розв’язок</summary>

```powershell
[xml]$saved = Get-Content "$HOME\PS-Kurs\курси.xml" -Raw -Encoding UTF8
($saved.exchange.currency | Where-Object { [double]$_.rate -gt 10 }).Count
```

</details>

---
[← Урок 19](19-xml-korotko.md) · [Зміст курсу](../../README.md)
