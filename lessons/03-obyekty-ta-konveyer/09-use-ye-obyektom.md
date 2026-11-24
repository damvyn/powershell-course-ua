# Урок 9. Усе є об’єктом

⏱ ~30 хв · Рівень: початковий · Потрібно: [модуль 2](../02-faily-ta-papky/README.md), папка `PS-Kurs` з файлами з уроку 7

## Що ви вмітимете після уроку

- розуміти, чим PowerShell відрізняється від командного рядка cmd;
- дізнаватися, які відомості «всередині» файлу, дати чи процесу, командою `Get-Member`;
- діставати одне потрібне значення через крапку: розмір файлу, день тижня, дату через 30 днів;
- викликати прості дії (методи) з дужками.

## Ситуація з життя

Вам треба дізнатися точний розмір одного файлу, дату його останньої зміни або який день тижня буде через 30 днів, коли закінчується підписка. Мишкою це «Властивості» файлу й календар. У PowerShell — один короткий рядок, бо кожна команда повертає не просто текст, а **об’єкт** з усіма відомостями.

## Пояснення (~15 хв)

### 1. Що таке об’єкт

Уявіть картку товару в інтернет-магазині. На ній є:

- **властивості** — назва, ціна, вага, колір;
- **дії** — «додати в кошик», «порівняти».

Об’єкт у PowerShell влаштований так само. Файл — це об’єкт з властивостями `Name` (назва), `Length` (розмір у байтах), `LastWriteTime` (дата зміни) і діями (вони називаються **методами**), наприклад «скопіювати».

Таблиця, яку ви бачите на екрані після `Get-ChildItem`, — це лише **кілька** властивостей, які PowerShell вирішив показати. Насправді їх набагато більше.

> [!NOTE]
> Саме тут головна відмінність від cmd: команда `dir` у cmd повертає звичайний текст, і щоб дістати з нього розмір файлу, його треба «вирізати» з рядка. PowerShell одразу повертає об’єкти, і потрібне значення можна просто взяти за назвою.

### 2. Що всередині: Get-Member

Перед початком перейдіть у робочу папку:

```powershell
Set-Location $HOME\PS-Kurs
```

Отримаємо один файл командою `Get-Item` і подивимось, що в ньому є:

```powershell
Get-Item покупки.txt | Get-Member
```

```text
   TypeName: System.IO.FileInfo

Name              MemberType     Definition
----              ----------     ----------
CopyTo            Method         System.IO.FileInfo CopyTo(string destFileName), ...
Delete            Method         void Delete()
...
CreationTime      Property       datetime CreationTime {get;set;}
Directory         Property       System.IO.DirectoryInfo Directory {get;}
Extension         Property       string Extension {get;}
LastWriteTime     Property       datetime LastWriteTime {get;set;}
Length            Property       long Length {get;}
Name              Property       string Name {get;}
...
```

Як читати:

- `TypeName` — **тип** об’єкта, тобто «що це таке». `FileInfo` — відомості про файл;
- `Property` — властивість (відомості);
- `Method` — метод (дія);
- у `Definition` видно тип значення: `string` — текст, `long` — ціле число, `datetime` — дата й час.

> [!TIP]
> Список довгий. Щоб побачити лише властивості, додайте `-MemberType Property`:
> ```powershell
> Get-Item покупки.txt | Get-Member -MemberType Property
> ```

Символ `|` (конвеєр) ви вже бачили в уроці 7. Тут він передає файл команді `Get-Member`. Докладно про конвеєр — у наступному уроці.

### 3. Беремо одне значення: крапка

Щоб дістати властивість, візьміть команду в **круглі дужки** і через крапку допишіть назву властивості:

```powershell
(Get-Item покупки.txt).Length          # розмір у байтах
(Get-Item покупки.txt).LastWriteTime   # коли змінювали
(Get-Item покупки.txt).Extension       # розширення
```

```text
52
24 листопада 2026 р. 12:41:07
.txt
```

Дужки потрібні, щоб PowerShell **спочатку виконав команду**, а вже потім узяв властивість з її результату. Як у математиці: `(2 + 3) * 4`.

Так само з датою:

```powershell
(Get-Date).Year         # поточний рік
(Get-Date).DayOfWeek    # день тижня
```

```text
2026
Tuesday
```

> [!NOTE]
> Назви днів тижня й місяців у властивостях завжди англійські (`Monday`, `Tuesday`…), незалежно від мови Windows. Це нормально: так PowerShell однаково працює на будь-якому комп’ютері.

### 4. Виконуємо дію: методи

Метод — це дія, яку вміє виконувати об’єкт. Метод викликають через крапку, і **після назви завжди ставлять дужки** `()`. У дужках іноді передають значення.

Яка дата буде через 30 днів:

```powershell
(Get-Date).AddDays(30)
```

```text
24 грудня 2026 р. 12:45:10
```

А який це буде день тижня? Можна поєднати метод і властивість:

```powershell
(Get-Date).AddDays(30).DayOfWeek
```

```text
Thursday
```

Текст теж є об’єктом:

```powershell
"Привіт".Length          # кількість літер: 6
"Привіт".ToUpper()       # ПРИВІТ
```

> [!TIP]
> Як дізнатися, які методи є в дати? Так само, через `Get-Member`:
> ```powershell
> Get-Date | Get-Member -MemberType Method
> ```
> Корисні: `AddDays`, `AddMonths`, `AddYears`, `AddHours`. Від’ємне число рахує назад: `AddDays(-7)` — тиждень тому.

## Типові помилки

### ❌ Метод без дужок

```powershell
(Get-Date).AddDays
```

```text
OverloadDefinitions
-------------------
datetime AddDays(double value)
```

**Що сталося:** замість дати PowerShell показав «опис» методу.

**Чому:** без дужок PowerShell не виконує метод, а лише розповідає, як ним користуватися: `AddDays` очікує одне число (`double value`).

### ✅ Правильно

```powershell
(Get-Date).AddDays(30)
```

---

### ❌ Забули круглі дужки навколо команди

```powershell
Get-Date.DayOfWeek
```

```text
Get-Date.DayOfWeek : The term 'Get-Date.DayOfWeek' is not recognized as the name of a cmdlet,
function, script file, or operable program.
```

**Чому:** без дужок PowerShell сприймає `Get-Date.DayOfWeek` як назву якоїсь іншої команди, а такої немає.

### ✅ Правильно

```powershell
(Get-Date).DayOfWeek
```

---

### ❌ Помилка в назві властивості

```powershell
(Get-Item покупки.txt).Size
```

**Що сталося:** нічого. Порожній рядок, жодної помилки.

**Чому:** у файлу немає властивості `Size` (розмір називається `Length`). Якщо властивості не існує, PowerShell мовчки повертає «нічого». Це підступно: здається, що команда спрацювала.

### ✅ Правильно

Якщо результат порожній, перевірте назву через `Get-Member`:

```powershell
Get-Item покупки.txt | Get-Member -MemberType Property
(Get-Item покупки.txt).Length
```

## Підсумок

| Що | Як | Приклад |
|---|---|---|
| Подивитися, що є в об’єкті | `команда \| Get-Member` | `Get-Date \| Get-Member` |
| Взяти властивість | `(команда).Властивість` | `(Get-Item f.txt).Length` |
| Виконати метод | `(команда).Метод(значення)` | `(Get-Date).AddDays(30)` |
| Метод без значення | дужки все одно потрібні | `"текст".ToUpper()` |

## Перевір себе (~5 хв)

1. Чим відрізняється властивість від методу? Наведіть по прикладу.
2. Що покаже `Get-Member` у рядку `TypeName`?
3. Чому `(Get-Date).AddDays` не повертає дату?
4. Навіщо круглі дужки в `(Get-Item покупки.txt).Length`?
5. Ви написали `(Get-Item звіт.docx).Lenght` і отримали порожній результат. Що сталося?

<details>
<summary>Відповіді</summary>

1. Властивість — це відомості (`Length`, `Name`, `DayOfWeek`), метод — дія (`AddDays()`, `ToUpper()`). Метод викликають з дужками.
2. Тип об’єкта, тобто що це таке: наприклад, `System.IO.FileInfo` для файлу або `System.DateTime` для дати.
3. Бракує дужок зі значенням: `AddDays(30)`. Без них PowerShell лише показує опис методу.
4. Щоб спочатку виконати команду `Get-Item`, а потім взяти властивість з її результату.
5. Помилка в назві: `Lenght` замість `Length`. Неіснуюча властивість повертає порожнечу без помилки.

</details>

## Практика (~10 хв)

**Завдання 1** (легке). Виведіть лише поточну годину (властивість `Hour`) і поточний місяць (`Month`).

<details>
<summary>Розв’язок</summary>

```powershell
(Get-Date).Hour
(Get-Date).Month
```

</details>

**Завдання 2.** Гарантія на пральну машину — 2 роки від сьогодні. Виведіть дату закінчення гарантії і день тижня, на який вона припаде.

<details>
<summary>Підказка</summary>

Знайдіть потрібний метод: `Get-Date | Get-Member -Name Add*`.

</details>

<details>
<summary>Розв’язок</summary>

```powershell
(Get-Date).AddYears(2)
(Get-Date).AddYears(2).DayOfWeek
```

</details>

**Завдання 3** ⭐. Для файлу `покупки.txt` з папки `PS-Kurs` виведіть:

1. дату його **створення** (не зміни);
2. повну назву папки, у якій він лежить.

Назви властивостей знайдіть самостійно через `Get-Member`.

<details>
<summary>Розв’язок</summary>

```powershell
Get-Item покупки.txt | Get-Member -MemberType Property
(Get-Item покупки.txt).CreationTime
(Get-Item покупки.txt).DirectoryName
```

`DirectoryName` повертає шлях до папки текстом. Є ще властивість `Directory`: вона повертає саму папку як об’єкт.

</details>

## Джерела

- [Discover objects, properties, and methods (PowerShell 101)](https://learn.microsoft.com/powershell/scripting/learn/ps101/03-discovering-objects)
- [about_Objects](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_objects?view=powershell-5.1)
- [about_Properties](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_properties?view=powershell-5.1)
- [about_Methods](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_methods?view=powershell-5.1)
- [Get-Member](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/get-member?view=powershell-5.1)
- [Get-Date](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/get-date?view=powershell-5.1)

---
[← Модуль 2](../02-faily-ta-papky/README.md) · [Зміст курсу](../../README.md) · [Урок 10. Конвеєр «|» →](10-konveyer.md)
