# Урок 18. JSON і дані з інтернету

⏱ ~30 хв · Рівень: початковий · Потрібно: [урок 17](17-csv-ta-excel.md), доступ до інтернету

## Що ви вмітимете після уроку

- розуміти, як виглядають дані у форматі JSON;
- отримувати дані з сайтів однією командою `Invoke-RestMethod`;
- рахувати з отриманими даними, як зі звичайними об’єктами;
- зберігати дані у JSON-файл і читати їх назад.

## Ситуація з життя

Ви хочете замовити на закордонному сайті річ за 120 доларів і зрозуміти, скільки це в гривнях. Можна щоразу шукати курс у браузері, але Національний банк України публікує офіційні курси валют у вигляді даних, які PowerShell отримує однією командою. А якщо зберегти сьогоднішній курс у файл, завтра можна буде порівняти, подешевшав долар чи подорожчав.

## Пояснення (~15 хв)

### 1. Як виглядають дані для програм

Відкрийте в браузері адресу:

```text
https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?valcode=USD&json
```

Замість звичайної сторінки ви побачите рядок тексту на кшталт:

```text
[{"r030":840,"txt":"Долар США","rate":41.8523,"cc":"USD","exchangedate":"27.11.2026"}]
```

Це **JSON** — найпоширеніший формат, у якому сайти й програми передають дані одне одному. Читається він так:

| Запис | Що означає |
|---|---|
| `{ ... }` | Один об’єкт, як `[PSCustomObject]` в уроці 16 |
| `"rate":41.8523` | Властивість `rate` зі значенням `41.8523` |
| `"txt":"Долар США"` | Текстові значення беруть у лапки, числа — ні |
| `[ ... ]` | Список (масив) об’єктів через кому |

Тобто тут список з одного об’єкта з п’ятьма властивостями: `r030` — код валюти числом, `txt` — назва, `rate` — курс у гривнях, `cc` — буквений код, `exchangedate` — дата, на яку встановлено курс.

> [!NOTE]
> Курси в прикладах уроку умовні. У вас будуть інші числа й дата.

### 2. Отримуємо дані: Invoke-RestMethod

`Invoke-RestMethod` завантажує дані за адресою і сам перетворює JSON на об’єкти PowerShell. Візьмемо курси всіх валют одразу, без `valcode=USD`:

```powershell
$rates = Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?json"
$rates.Count
```

```text
62
```

У `$rates` тепер список об’єктів, по одному на валюту. З ним працює все, що ви знаєте з модулів 3 і 4:

```powershell
$usd = $rates | Where-Object { $_.cc -eq "USD" }
$usd
```

```text
r030         : 840
txt          : Долар США
rate         : 41.8523
cc           : USD
exchangedate : 27.11.2026
```

> [!IMPORTANT]
> Результат `Invoke-RestMethod` спершу збережіть у змінну, а вже потім передавайте конвеєром далі. Якщо одразу написати `Invoke-RestMethod ... | Where-Object`, фільтр не спрацює (див. «Типові помилки»).

### 3. Рахуємо: скільки це в гривнях

`rate` — це вже **число**, а не текст, як було з CSV: JSON зберігає числа без лапок, і PowerShell це розуміє. Тому одразу множимо:

```powershell
$price = 120
[math]::Round($price * $usd.rate, 2)
```

```text
5022,28
```

Кілька валют одним поглядом:

```powershell
$rates | Where-Object { $_.cc -in "USD", "EUR", "PLN" } | Select-Object cc, txt, rate
```

```text
cc  txt          rate
--  ---          ----
USD Долар США    41.8523
EUR Євро         48.3017
PLN Злотий       11.3512
```

### 4. Курс на певну дату

Адреса може містити **параметри** після знака `?`, розділені `&`. Ви вже бачили `valcode=USD` (яка валюта) і `json` (у якому форматі відповісти). Ще один параметр, `date`, задає дату. Цей сайт чекає дату у форматі `yyyyMMdd`, без рисочок:

```powershell
$date = Get-Date -Date "2026-01-01" -Format "yyyyMMdd"
$date
```

```text
20260101
```

```powershell
$old = Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?valcode=USD&date=$date&json"
$old.rate
```

Змінна `$date` підставилася просто в адресу, бо адреса записана в подвійних лапках (урок 15). Дату ми, як завжди, задали у форматі `yyyy-MM-dd`, а потрібний сайту вигляд отримали через `-Format`.

### 5. Зберігаємо у файл і читаємо назад

Збережемо сьогоднішній курс долара, щоб завтра порівняти. `ConvertTo-Json` перетворює об’єкт на JSON-текст, а `Set-Content` записує його у файл (урок 7):

```powershell
$usd | ConvertTo-Json | Set-Content "$HOME\PS-Kurs\курс.json" -Encoding UTF8
Get-Content "$HOME\PS-Kurs\курс.json" -Encoding UTF8
```

```text
{
    "r030":  840,
    "txt":  "Долар США",
    "rate":  41.8523,
    "cc":  "USD",
    "exchangedate":  "27.11.2026"
}
```

Наступного дня прочитайте файл і перетворіть текст назад на об’єкт командою `ConvertFrom-Json`:

```powershell
$saved = Get-Content "$HOME\PS-Kurs\курс.json" -Raw -Encoding UTF8 | ConvertFrom-Json
$current = (Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?valcode=USD&json").rate
$current - $saved.rate
```

Додатне число означає, що долар подорожчав, від’ємне — що подешевшав.

`-Raw` читає файл **одним рядком тексту**, а не списком окремих рядків. `ConvertFrom-Json` потрібен увесь JSON разом, тому з JSON-файлами завжди пишіть `-Raw`.

> [!NOTE]
> Якщо об’єкт складний, з об’єктами всередині об’єктів, `ConvertTo-Json` за замовчуванням зберігає лише два рівні вкладеності, а глибші перетворює на текст на кшталт `System.Collections.Hashtable`. Для таких даних додайте `-Depth 5`. Windows PowerShell 5.1 робить це мовчки, PowerShell 7 попереджає.

## Типові помилки

### ❌ Фільтр пропускає всі валюти

```powershell
Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?json" | Where-Object { $_.cc -eq "USD" }
```

**Що сталося:** замість одного долара на екрані всі 62 валюти.

**Чому:** `Invoke-RestMethod` передає конвеєром увесь список **одним шматком**, а не по одній валюті. У `$_` потрапляє весь список, `$_.cc` — це коди всіх валют, і серед них є `USD`, тож умова виконується, і `Where-Object` пропускає весь список.

### ✅ Правильно

Збережіть результат у змінну або візьміть команду в круглі дужки. Тоді список піде конвеєром по одному об’єкту:

```powershell
$rates = Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?json"
$rates | Where-Object { $_.cc -eq "USD" }

(Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?json") | Where-Object { $_.cc -eq "USD" }
```

---

### ❌ Помилка в адресі або немає інтернету

```powershell
Invoke-RestMethod "https://bank.gov.u/NBUStatService/v1/statdirectory/exchange?json"
```

```text
Invoke-RestMethod : The remote name could not be resolved: 'bank.gov.u'
```

**Чому:** комп’ютер не знайшов сайт з такою назвою: в адресі помилка (`gov.u` замість `gov.ua`) або немає з’єднання з інтернетом.

### ✅ Правильно

Перевірте адресу, відкривши її в браузері. Якщо в браузері вона відкривається, а в PowerShell ні, перевірте, чи не скопіювалися зайві пробіли чи лапки.

---

### ❌ Invoke-WebRequest просить Internet Explorer

Для завантаження звичайних сторінок і файлів є схожа команда `Invoke-WebRequest`. У Windows PowerShell 5.1 вона може завершитися так:

```powershell
Invoke-WebRequest "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?json"
```

```text
Invoke-WebRequest : The response content cannot be parsed because the Internet Explorer engine is not available, or
Internet Explorer's first-launch configuration is not complete. Specify the UseBasicParsing parameter and try again.
```

**Чому:** у 5.1 `Invoke-WebRequest` за замовчуванням розбирає сторінку за допомогою Internet Explorer, якого в сучасній Windows немає або його жодного разу не запускали.

### ✅ Правильно

Для даних (JSON) користуйтеся `Invoke-RestMethod`, йому Internet Explorer не потрібен. Якщо потрібен саме `Invoke-WebRequest`, додайте `-UseBasicParsing`, як і радить повідомлення:

```powershell
Invoke-WebRequest "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?json" -UseBasicParsing
```

> [!NOTE]
> **PowerShell 7** не використовує Internet Explorer, і `-UseBasicParsing` там нічого не змінює.

---

### ❌ Дата з сайту не перетворюється

```powershell
[datetime]$usd.exchangedate
```

```text
Cannot convert value "27.11.2026" to type "System.DateTime". Error: "String was not recognized as a valid DateTime."
```

**Чому:** як ви пам’ятаєте з уроку 14, `[datetime]"..."` читає дату в порядку місяць.день.рік, а 27-го місяця не буває. Сайт віддає дату як текст у форматі `dd.MM.yyyy`, а не в `yyyy-MM-dd`.

### ✅ Правильно

Підкажіть PowerShell, у якому форматі записано дату, методом `ParseExact`:

```powershell
[datetime]::ParseExact($usd.exchangedate, "dd.MM.yyyy", $null)
```

Перше значення — текст з датою, друге — його формат (як у `-Format`), третє `$null` означає «решту налаштувань візьми звичайні».

## Підсумок

| Що | Як |
|---|---|
| Отримати дані з сайту | `$data = Invoke-RestMethod "адреса"` |
| Фільтрувати отримане | спершу у змінну, потім `$data \| Where-Object ...` |
| Параметри в адресі | `...?valcode=USD&date=$date&json` |
| Об’єкт → JSON-файл | `$obj \| ConvertTo-Json \| Set-Content файл -Encoding UTF8` |
| JSON-файл → об’єкт | `Get-Content файл -Raw -Encoding UTF8 \| ConvertFrom-Json` |
| Глибокі об’єкти | `ConvertTo-Json -Depth 5` |
| Текст у вигляді `27.11.2026` → дата | `[datetime]::ParseExact(текст, "dd.MM.yyyy", $null)` |

## Перевір себе (~5 хв)

1. Що в JSON означають фігурні дужки `{ }`, а що — квадратні `[ ]`?
2. Чому з курсом `rate` можна одразу рахувати, а з `Amount` з CSV-файлу — ні?
3. Чому `Invoke-RestMethod ... | Where-Object ...` пропускає всі валюти?
4. Навіщо `-Raw`, коли читаєте JSON-файл?
5. Яка команда в Windows PowerShell 5.1 може вимагати `-UseBasicParsing`?

<details>
<summary>Відповіді</summary>

1. `{ }` — один об’єкт з властивостями, `[ ]` — список.
2. JSON зберігає числа як числа, а CSV — усе як текст.
3. `Invoke-RestMethod` передає конвеєром увесь список одним об’єктом. Треба спершу зберегти результат у змінну або взяти команду в дужки.
4. Щоб отримати весь файл одним текстом, бо `ConvertFrom-Json` потрібен JSON цілком.
5. `Invoke-WebRequest`.

</details>

## Практика (~10 хв)

**Завдання 1** (легке). Виведіть сьогоднішній офіційний курс євро: лише назву валюти і курс.

<details>
<summary>Розв’язок</summary>

```powershell
$rates = Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?json"
$rates | Where-Object { $_.cc -eq "EUR" } | Select-Object txt, rate
```

</details>

**Завдання 2.** Квиток на концерт за кордоном коштує 85 євро. Порахуйте, скільки це в гривнях, округливши до цілих, і виведіть рядком на кшталт `85 EUR = 4106 грн`.

<details>
<summary>Розв’язок</summary>

```powershell
$eur = Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?valcode=EUR&json"
$price = 85
$uah = [math]::Round($price * $eur.rate)
"$price EUR = $uah грн"
```

</details>

**Завдання 3** ⭐. Дізнайтеся, на скільки гривень змінився курс долара з 2026-01-01 до сьогодні. Виведіть обидва курси і різницю, округлену до двох знаків після коми.

<details>
<summary>Підказка</summary>

Потрібні два запити: з параметром `date` і без нього. Дату для адреси отримайте через `Get-Date -Date "2026-01-01" -Format "yyyyMMdd"`.

</details>

<details>
<summary>Розв’язок</summary>

```powershell
$date = Get-Date -Date "2026-01-01" -Format "yyyyMMdd"
$then = (Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?valcode=USD&date=$date&json").rate
$now  = (Invoke-RestMethod "https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?valcode=USD&json").rate
$then
$now
[math]::Round($now - $then, 2)
```

Круглі дужки навколо `Invoke-RestMethod` дають змогу одразу взяти властивість `.rate` результату.

</details>

## Джерела

- [Invoke-RestMethod](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/invoke-restmethod?view=powershell-5.1)
- [Invoke-WebRequest](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-5.1)
- [ConvertTo-Json](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/convertto-json?view=powershell-5.1)
- [ConvertFrom-Json](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/convertfrom-json?view=powershell-5.1)
- [DateTime.ParseExact](https://learn.microsoft.com/dotnet/api/system.datetime.parseexact)
- [Відкриті дані НБУ: API для розробників](https://bank.gov.ua/ua/open-data/api-dev)

---
[← Урок 17](17-csv-ta-excel.md) · [Зміст курсу](../../README.md) · [Урок 19. XML коротко →](19-xml-korotko.md)
