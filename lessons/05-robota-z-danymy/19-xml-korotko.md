# Урок 19. XML коротко

⏱ ~20 хв · Рівень: початковий · Потрібно: [урок 18](18-json-i-dani-z-internetu.md), доступ до інтернету

Це короткий оглядовий урок. Самі ви навряд чи часто створюватимете XML-файли, але читати їх доведеться: у XML зберігають налаштування багато програм, у ньому публікують стрічки новин (RSS), і в ньому ж віддають дані деякі сайти.

## Що ви вмітимете після уроку

- впізнавати формат XML і розуміти його будову;
- отримувати XML-дані з сайту і з файлу;
- діставати потрібні значення через крапку.

## Ситуація з життя

В уроці 18 ви отримали курси валют НБУ у форматі JSON. Той самий сайт уміє віддавати їх і в XML, а деякі старіші сервіси й програми знають лише XML. Розберемося, як прочитати ті самі курси, якщо вони прийшли в такому вигляді, і як зберегти їх у файл.

## Пояснення (~10 хв)

### 1. Як виглядає XML

Відкрийте в браузері ту саму адресу, що й в уроці 18, але **без** `&json` у кінці:

```text
https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?valcode=USD
```

```xml
<exchange>
  <currency>
    <r030>840</r030>
    <txt>Долар США</txt>
    <rate>41.8523</rate>
    <cc>USD</cc>
    <exchangedate>27.11.2026</exchangedate>
  </currency>
</exchange>
```

Дані ті самі, тільки записані інакше:

- кожне значення стоїть між **тегом**, що відкривається, `<rate>` і тегом, що закривається, `</rate>`;
- теги вкладаються один в одний, як папки: `exchange` містить `currency`, а `currency` містить `rate`, `cc` тощо;
- усе разом — одне «дерево» з одним головним тегом, тут `exchange`.

### 2. Отримуємо XML з сайту

`Invoke-RestMethod` розпізнає XML так само, як JSON:

```powershell
$xml = Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange"
```

До значень звертайтеся через вкладені теги, від головного, через крапку. Це схоже на шлях до файлу, тільки замість `\` крапки:

```powershell
$xml.exchange.currency.Count
$usd = $xml.exchange.currency | Where-Object { $_.cc -eq "USD" }
$usd.rate
```

```text
62
41.8523
```

> [!IMPORTANT]
> У XML **усі значення — текст**, як у CSV (урок 17). Перед обчисленнями перетворіть значення на число: `[double]$usd.rate`.

```powershell
[math]::Round(120 * [double]$usd.rate, 2)
```

```text
5022,28
```

`[double]` — тип для дробових чисел, як `[int]` для цілих (урок 14). Число з крапкою `41.8523` він розуміє за будь-яких регіональних налаштувань.

### 3. Зберігаємо у файл і читаємо з файлу

Збережіть отримані курси у файл:

```powershell
$xml.Save("$HOME\PS-Kurs\курси.xml")
```

`Save()` — метод самого XML-документа (урок 9). Шлях йому давайте **повний**, як тут.

Щоб прочитати XML-файл, прочитайте його як текст і перетворіть на XML, написавши `[xml]` перед змінною:

```powershell
[xml]$saved = Get-Content "$HOME\PS-Kurs\курси.xml" -Raw -Encoding UTF8
$saved.exchange.currency | Where-Object { $_.cc -eq "EUR" } | Select-Object txt, rate
```

```text
txt  rate
---  ----
Євро 48.3017
```

Так само читаються налаштування програм у XML: відкрийте файл у Блокноті, подивіться, як вкладені теги, і пройдіть цим шляхом через крапки.

### 4. Стрічки новин (RSS)

Стрічки новин на сайтах — теж XML. Для них `Invoke-RestMethod` одразу повертає список новин, без шляху через крапки. Ось останні записи блогу команди PowerShell:

```powershell
Invoke-RestMethod "https://devblogs.microsoft.com/powershell/feed/" | Select-Object -First 5 title, pubDate
```

Адресу стрічки на іншому сайті зазвичай можна знайти внизу сторінки за значком RSS або словом «RSS».

## Типові помилки

### ❌ Забули [xml]

```powershell
$saved = Get-Content "$HOME\PS-Kurs\курси.xml" -Raw -Encoding UTF8
$saved.exchange.currency
```

**Що сталося:** нічого не виведено, жодної помилки.

**Чому:** без `[xml]` у змінній лежить просто текст. У тексту немає властивості `exchange`, а відсутня властивість мовчки повертає «нічого» (урок 9).

### ✅ Правильно

```powershell
[xml]$saved = Get-Content "$HOME\PS-Kurs\курси.xml" -Raw -Encoding UTF8
$saved.exchange.currency
```

---

### ❌ Зіпсований XML-файл

Ви відредагували файл у Блокноті й випадково стерли тег `</cc>`. Під час читання:

```text
Cannot convert value "..." to type "System.Xml.XmlDocument".
Error: "The 'cc' start tag on line ... does not match the end tag of 'currency'. ..."
```

**Чому:** у XML кожен тег, що відкрився, має закритися, і в правильному порядку. Якщо хоча б одного бракує, файл не читається зовсім.

### ✅ Правильно

У повідомленні вказано, який тег не закрито (`'cc'`) і номер рядка. Відкрийте файл у Блокноті, знайдіть цей рядок і допишіть тег, що закривається. Перед редагуванням XML-файлів програм завжди робіть копію (урок 6).

## Підсумок

| Що | Як |
|---|---|
| XML з сайту | `$xml = Invoke-RestMethod "адреса"` |
| XML з файлу | `[xml]$x = Get-Content файл -Raw -Encoding UTF8` |
| Дістати значення | `$x.головний.вкладений.тег` |
| Перетворити значення на число | `[double]$x....rate` |
| Зберегти у файл | `$xml.Save("повний шлях")` |
| Стрічка новин | `Invoke-RestMethod "адреса RSS" \| Select-Object title, pubDate` |

## Перевір себе (~3 хв)

1. Чим XML відрізняється від JSON на вигляд?
2. Чому перед обчисленнями значення з XML треба перетворити на число?
3. Що буде, якщо прочитати XML-файл без `[xml]`?

<details>
<summary>Відповіді</summary>

1. У XML кожне значення стоїть між тегами `<назва>` і `</назва>`, а в JSON записується як `"назва": значення` у фігурних дужках.
2. Усі значення в XML — текст.
3. У змінній буде звичайний текст, і шлях через крапки нічого не поверне.

</details>

## Практика (~5 хв)

**Завдання 1.** Отримайте курси НБУ в XML і виведіть назви й курси долара, євро та злотого.

<details>
<summary>Розв’язок</summary>

```powershell
$xml = Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange"
$xml.exchange.currency | Where-Object { $_.cc -in "USD", "EUR", "PLN" } | Select-Object txt, rate
```

</details>

**Завдання 2.** Прочитайте збережений файл `курси.xml` і виведіть п’ять валют з найвищим курсом.

<details>
<summary>Розв’язок</summary>

```powershell
[xml]$saved = Get-Content "$HOME\PS-Kurs\курси.xml" -Raw -Encoding UTF8
$saved.exchange.currency |
    Sort-Object { [double]$_.rate } -Descending |
    Select-Object -First 5 txt, rate
```

Без `[double]` курси сортувалися б як текст, як `Amount` в уроці 17.

</details>

## Джерела

- [about_Type_Accelerators — `[xml]`](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_type_accelerators?view=powershell-5.1)
- [Invoke-RestMethod — RSS і XML](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/invoke-restmethod?view=powershell-5.1)
- [XmlDocument.Save](https://learn.microsoft.com/dotnet/api/system.xml.xmldocument.save)
- [Відкриті дані НБУ: API для розробників](https://bank.gov.ua/ua/open-data/api-dev)

---
[← Урок 18](18-json-i-dani-z-internetu.md) · [Зміст курсу](../../README.md) · [Тест модуля 5 →](README.md#підсумковий-тест-модуля-5)
