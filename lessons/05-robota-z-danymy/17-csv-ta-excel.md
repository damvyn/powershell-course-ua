# Урок 17. CSV та Excel

⏱ ~30 хв · Рівень: початковий · Потрібно: [модуль 4](../04-zminni-ta-typy-danykh/README.md)

## Що ви вмітимете після уроку

- читати таблицю з CSV-файлу і працювати з нею як з об’єктами;
- рахувати суми й групувати рядки таблиці;
- зберігати результат у CSV-файл, який правильно відкриється в Excel;
- дописувати нові рядки в наявний файл.

## Ситуація з життя

Ви записуєте витрати в таблицю: дата, категорія, на що витратили, сума. Наприкінці місяця хочеться знати, скільки пішло на продукти, транспорт і розваги, і зберегти короткий підсумок, щоб відкрити його в Excel або надіслати комусь із родини. Рахувати все вручну калькулятором довго, а PowerShell зробить це кількома командами.

## Підготовка

Створимо файл з витратами за листопад. Скопіюйте всі рядки разом і вставте у вікно PowerShell:

```powershell
New-Item -ItemType Directory -Path "$HOME\PS-Kurs" -Force | Out-Null
$data = @"
Date,Category,Description,Amount
2026-11-02,Продукти,Супермаркет,1250
2026-11-03,Транспорт,Таксі,180
2026-11-05,Комунальні,Електроенергія,812
2026-11-08,Продукти,Супермаркет,640
2026-11-10,Розваги,Кіно,350
2026-11-15,Продукти,Ринок,95
2026-11-18,Транспорт,Пальне,1500
2026-11-22,Розваги,Кафе,720
"@
Set-Content -Path "$HOME\PS-Kurs\витрати.csv" -Value $data -Encoding UTF8
```

`-Force` у `New-Item` означає «створи папку, якщо її ще немає, і не сварися, якщо вже є». `| Out-Null` прибирає зайвий вивід.

## Пояснення (~15 хв)

### 1. Що таке CSV

Відкрийте файл у Блокноті:

```powershell
notepad "$HOME\PS-Kurs\витрати.csv"
```

Це звичайний текстовий файл (урок 7). Перший рядок — **заголовки** стовпців, кожен наступний — один рядок таблиці. Значення розділені комою, звідси й назва: CSV — *comma-separated values*, «значення через кому». Такі файли вміють відкривати Excel, Google Таблиці, банківські застосунки й майже всі програми, що працюють з таблицями.

Закрийте Блокнот, нічого не змінюючи.

### 2. Читаємо таблицю: Import-Csv

```powershell
$expenses = Import-Csv "$HOME\PS-Kurs\витрати.csv"
$expenses
```

```text
Date       Category   Description    Amount
----       --------   -----------    ------
2026-11-02 Продукти   Супермаркет    1250
2026-11-03 Транспорт  Таксі          180
2026-11-05 Комунальні Електроенергія 812
...
```

`Import-Csv` перетворює кожен рядок файлу на **об’єкт**, а заголовки — на назви властивостей. Тобто ви отримали такі самі об’єкти, як `[PSCustomObject]` в уроці 16, і з ними працює все, що ви вже знаєте:

```powershell
$expenses.Count                                       # скільки записів
$expenses[0]                                          # перший запис
$expenses | Where-Object { $_.Category -eq "Продукти" }  # лише продукти
```

> [!TIP]
> Заголовки у файлі стають назвами властивостей, тому для своїх таблиць пишіть їх англійською: `Date`, `Category`, `Amount`. Так їх зручно набирати в командах.

### 3. Рахуємо: скільки витратили

Загальна сума за місяць:

```powershell
($expenses | Measure-Object Amount -Sum).Sum
```

```text
5547
```

А тепер по категоріях. `Group-Object` (урок 12) збирає записи з однаковою категорією в групи, а обчислюваний стовпець рахує суму кожної групи:

```powershell
$byCategory = $expenses |
    Group-Object Category |
    Select-Object @{ Name = "Category"; Expression = { $_.Name } },
                  @{ Name = "Total";    Expression = { ($_.Group | Measure-Object Amount -Sum).Sum } } |
    Sort-Object Total -Descending
$byCategory
```

```text
Category   Total
--------   -----
Продукти    1985
Транспорт   1680
Розваги     1070
Комунальні   812
```

Розберемо по кроках:

1. `Group-Object Category` складає записи в чотири групи. У кожної групи `Name` — назва категорії, а `Group` — записи цієї категорії.
2. Перший обчислюваний стовпець просто перейменовує `Name` на зрозуміліший `Category`.
3. Другий для кожної групи передає її записи в `Measure-Object` і бере суму.
4. `Sort-Object Total -Descending` ставить найбільші витрати вгору.

> [!IMPORTANT]
> Усі значення, прочитані з CSV, — це **текст**, навіть якщо виглядають як числа. `Measure-Object` сам перетворює текст на числа, тому сума правильна. А от сортування за `Amount` порівнюватиме текст, а не числа (див. «Типові помилки»).

### 4. Зберігаємо підсумок для Excel: Export-Csv

> [!WARNING]
> У Windows PowerShell 5.1 `Export-Csv` без `-Encoding` записує файл у кодуванні ASCII, у якому немає українських літер: замість «Продукти» у файлі буде `????????`. Тому **завжди** додавайте `-Encoding UTF8`.

```powershell
$byCategory | Export-Csv "$HOME\PS-Kurs\підсумок.csv" -NoTypeInformation -Encoding UTF8 -UseCulture
```

Що означає кожен параметр:

| Параметр | Навіщо |
|---|---|
| `-NoTypeInformation` | Не писати в перший рядок службовий рядок `#TYPE ...`. У PowerShell 7 цей рядок і так не пишеться |
| `-Encoding UTF8` | Українські літери збережуться правильно, і Excel їх розпізнає |
| `-UseCulture` | Розділювач значень береться з регіональних налаштувань Windows |

Навіщо `-UseCulture`? Excel, відкриваючи CSV подвійним клацанням, очікує той розділювач, який указано в регіональних налаштуваннях Windows. В українських налаштуваннях це **крапка з комою** `;`, бо кома вже зайнята під дробові числа. Подивитися, який розділювач у вас:

```powershell
(Get-Culture).TextInfo.ListSeparator
```

```text
;
```

Ті самі налаштування, що й для формату дати в уроці 14: **Win + R** → `intl.cpl` → **Додаткові параметри** → поле **Роздільник елементів списку**. Змінювати там нічого не треба.

Відкрийте результат в Excel, наче подвійним клацанням (урок 5):

```powershell
Invoke-Item "$HOME\PS-Kurs\підсумок.csv"
```

Ви побачите акуратну таблицю з двома стовпцями. Якщо Excel не встановлено, файл відкриється в іншій програмі для таблиць або в Блокноті.

> [!NOTE]
> Файл `витрати.csv` ми лишили з комами: його читає PowerShell, а `Import-Csv` за замовчуванням чекає саме кому. `-UseCulture` потрібен тоді, коли файл відкриватиме Excel. Щоб прочитати файл з крапкою з комою, додайте той самий параметр: `Import-Csv "$HOME\PS-Kurs\підсумок.csv" -UseCulture`.

### 5. Дописуємо новий запис

Сьогодні ви купили квитки до театру. Новий рядок — це об’єкт з тими самими властивостями, що й у файлі. `-Append` дописує його в кінець, не стираючи решту:

```powershell
[PSCustomObject]@{
    Date        = Get-Date -Format "yyyy-MM-dd"
    Category    = "Розваги"
    Description = "Театр"
    Amount      = 600
} | Export-Csv "$HOME\PS-Kurs\витрати.csv" -Append -NoTypeInformation -Encoding UTF8
```

Перевірте: `Import-Csv "$HOME\PS-Kurs\витрати.csv" | Select-Object -Last 2`.

> [!CAUTION]
> Без `-Append` команда `Export-Csv` мовчки **замінить** весь файл одним новим рядком. Дописуючи, вказуйте те саме кодування й той самий розділювач, що й у файлі.

### 6. А як щодо файлів Excel (.xlsx)?

Файл `.xlsx` — це не текст, а запакований набір даних (урок 7), і стандартні команди PowerShell його не читають. Найпростіший шлях — зберегти таблицю з Excel як CSV:

1. Відкрийте таблицю в Excel.
2. **Файл** → **Зберегти як** → виберіть папку.
3. У списку типів файлу виберіть **CSV UTF-8** (у списку він підписаний як «з розділювачами-комами», але з українськими налаштуваннями Excel все одно поставить `;`).
4. Натисніть **Зберегти** і погодьтеся, що збережеться лише поточний аркуш.

Такий файл читайте з `-UseCulture`:

```powershell
Import-Csv "$HOME\Documents\таблиця.csv" -UseCulture
```

> [!NOTE]
> Якщо замість **CSV UTF-8** вибрати просто **CSV**, Excel збереже файл у старому кодуванні Windows. Тоді додайте `-Encoding Default`: `Import-Csv шлях -UseCulture -Encoding Default`.

Існує й окремий модуль, який читає і створює `.xlsx` напряму, без Excel. Що таке модулі і як їх встановлювати — в уроці 25.

## Типові помилки

### ❌ Знаки питання замість українських літер

```powershell
$byCategory | Export-Csv "$HOME\PS-Kurs\підсумок.csv" -NoTypeInformation -UseCulture
Get-Content "$HOME\PS-Kurs\підсумок.csv"
```

```text
"Category";"Total"
"????????";"1985"
"?????????";"1680"
```

**Чому:** у Windows PowerShell 5.1 `Export-Csv` за замовчуванням записує в кодуванні ASCII, де немає кирилиці. Кожна українська літера перетворилася на `?`, і повернути її вже не можна.

### ✅ Правильно

Запишіть файл ще раз з `-Encoding UTF8`:

```powershell
$byCategory | Export-Csv "$HOME\PS-Kurs\підсумок.csv" -NoTypeInformation -Encoding UTF8 -UseCulture
```

> [!NOTE]
> **PowerShell 7** записує UTF-8 за замовчуванням, але без позначки на початку файлу (BOM). Без неї Excel, відкриваючи файл подвійним клацанням, може показати «кракозябри», тому для Excel там пишіть `-Encoding utf8BOM`.

---

### ❌ Усе в одному стовпці Excel

Ви зберегли файл без `-UseCulture` і відкрили в Excel, а вся таблиця опинилася в стовпці A: `"Category","Total"`, `"Продукти","1985"`.

**Чому:** `Export-Csv` за замовчуванням розділяє значення комою, а Excel з українськими налаштуваннями чекає крапку з комою і не розуміє, де закінчується стовпець.

### ✅ Правильно

Додайте `-UseCulture` (або `-Delimiter ';'`):

```powershell
$byCategory | Export-Csv "$HOME\PS-Kurs\підсумок.csv" -NoTypeInformation -Encoding UTF8 -UseCulture
```

---

### ❌ Зайвий рядок #TYPE

```powershell
$byCategory | Export-Csv "$HOME\PS-Kurs\підсумок.csv" -Encoding UTF8 -UseCulture
Get-Content "$HOME\PS-Kurs\підсумок.csv" -Encoding UTF8 -TotalCount 2
```

```text
#TYPE Selected.Microsoft.PowerShell.Commands.GroupInfo
"Category";"Total"
```

**Що сталося:** в Excel перша клітинка містить незрозумілий текст `#TYPE ...`, а заголовки з’їхали на другий рядок.

**Чому:** Windows PowerShell 5.1 за замовчуванням записує в перший рядок тип об’єктів. PowerShell його розуміє, а для людей і Excel це сміття.

### ✅ Правильно

Додавайте `-NoTypeInformation`. У PowerShell 7 цей параметр уже не потрібен: рядок `#TYPE` там не пишеться.

---

### ❌ Сортування «ламає» числа

Яка покупка найдорожча?

```powershell
$expenses | Sort-Object Amount -Descending | Select-Object -First 3
```

```text
Date       Category   Description    Amount
----       --------   -----------    ------
2026-11-15 Продукти   Ринок          95
2026-11-05 Комунальні Електроенергія 812
2026-11-22 Розваги    Кафе           720
```

**Що сталося:** «найдорожчою» стала покупка за 95 грн, а пальне за 1 500 грн навіть не потрапило в трійку.

**Чому:** значення з CSV — текст. Текст сортується за абеткою, символ за символом: `"9"` більше за `"8"` і за `"1"`, тому `"95"` опинилося вище за `"1500"`.

### ✅ Правильно

Перетворіть значення на число просто під час сортування. Замість назви властивості дайте `Sort-Object` блок `{ }` з обчисленням, як у `Where-Object`:

```powershell
$expenses | Sort-Object { [int]$_.Amount } -Descending | Select-Object -First 3
```

```text
Date       Category   Description    Amount
----       --------   -----------    ------
2026-11-18 Транспорт  Пальне         1500
2026-11-02 Продукти   Супермаркет    1250
2026-11-05 Комунальні Електроенергія 812
```

Так само для фільтра: `Where-Object { [int]$_.Amount -gt 500 }`.

## Підсумок

| Що | Як |
|---|---|
| Прочитати CSV | `$t = Import-Csv шлях` |
| Прочитати CSV з `;` (від Excel) | `Import-Csv шлях -UseCulture` |
| Сума стовпця | `($t \| Measure-Object Amount -Sum).Sum` |
| Сортувати числа з CSV | `Sort-Object { [int]$_.Amount }` |
| Зберегти для PowerShell | `Export-Csv шлях -NoTypeInformation -Encoding UTF8` |
| Зберегти для Excel | `Export-Csv шлях -NoTypeInformation -Encoding UTF8 -UseCulture` |
| Дописати рядок | `... \| Export-Csv шлях -Append -NoTypeInformation -Encoding UTF8` |
| Розділювач у вашій Windows | `(Get-Culture).TextInfo.ListSeparator` |

## Перевір себе (~5 хв)

1. На що перетворюється перший рядок CSV-файлу після `Import-Csv`?
2. Чому без `-Encoding UTF8` у файлі з’являються знаки питання?
3. Навіщо `-UseCulture`, якщо файл відкриватиме Excel?
4. Чому `Sort-Object Amount` після `Import-Csv` ставить 95 вище за 1500?
5. Що станеться, якщо дописати рядок командою `Export-Csv` без `-Append`?

<details>
<summary>Відповіді</summary>

1. На назви властивостей об’єктів.
2. У 5.1 `Export-Csv` за замовчуванням пише в ASCII, де немає кирилиці.
3. Excel з українськими налаштуваннями чекає розділювач `;`, а не кому. `-UseCulture` бере розділювач з налаштувань Windows.
4. Значення з CSV — текст, і сортуються за абеткою. Треба `Sort-Object { [int]$_.Amount }`.
5. Файл буде замінено, у ньому залишиться лише новий рядок.

</details>

## Практика (~10 хв)

**Завдання 1** (легке). Прочитайте `витрати.csv` і виведіть лише записи категорії «Транспорт», а потім суму витрат на транспорт.

<details>
<summary>Розв’язок</summary>

```powershell
$expenses = Import-Csv "$HOME\PS-Kurs\витрати.csv"
$transport = $expenses | Where-Object { $_.Category -eq "Транспорт" }
$transport
($transport | Measure-Object Amount -Sum).Sum
```

</details>

**Завдання 2.** Збережіть у файл `великі-витрати.csv` для Excel лише покупки, дорожчі за 500 грн, відсортовані від найдорожчої. Відкрийте файл в Excel.

<details>
<summary>Розв’язок</summary>

```powershell
$expenses = Import-Csv "$HOME\PS-Kurs\витрати.csv"
$expenses |
    Where-Object { [int]$_.Amount -gt 500 } |
    Sort-Object { [int]$_.Amount } -Descending |
    Export-Csv "$HOME\PS-Kurs\великі-витрати.csv" -NoTypeInformation -Encoding UTF8 -UseCulture
Invoke-Item "$HOME\PS-Kurs\великі-витрати.csv"
```

</details>

**Завдання 3** ⭐. Збережіть у файл `файли.csv` для Excel список файлів з «Завантажень»: назва, розмір у МБ з одним знаком після коми і дата зміни у форматі `yyyy-MM-dd`. Відкрийте його в Excel.

<details>
<summary>Підказка</summary>

Стовпці виберіть `Select-Object` з обчислюваними стовпцями (урок 12). Дату перетворіть на текст через `ToString("yyyy-MM-dd")` або `-f` (урок 15).

</details>

<details>
<summary>Розв’язок</summary>

```powershell
Get-ChildItem "$HOME\Downloads" -File |
    Select-Object Name,
                  @{ Name = "SizeMB";   Expression = { [math]::Round($_.Length / 1MB, 1) } },
                  @{ Name = "Modified"; Expression = { $_.LastWriteTime.ToString("yyyy-MM-dd") } } |
    Export-Csv "$HOME\PS-Kurs\файли.csv" -NoTypeInformation -Encoding UTF8 -UseCulture
Invoke-Item "$HOME\PS-Kurs\файли.csv"
```

</details>

## Джерела

- [Import-Csv](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/import-csv?view=powershell-5.1)
- [Export-Csv](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/export-csv?view=powershell-5.1)
- [Group-Object](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/group-object?view=powershell-5.1)
- [Sort-Object](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/sort-object?view=powershell-5.1)
- [about_Character_Encoding](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_character_encoding?view=powershell-5.1)

---
[← Модуль 4](../04-zminni-ta-typy-danykh/README.md) · [Зміст курсу](../../README.md) · [Урок 18. JSON і дані з інтернету →](18-json-i-dani-z-internetu.md)
