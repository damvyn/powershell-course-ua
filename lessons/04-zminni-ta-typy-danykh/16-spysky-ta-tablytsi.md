# Урок 16. Списки та таблиці

⏱ ~30 хв · Рівень: початковий · Потрібно: [урок 15](15-ryadky.md)

## Що ви вмітимете після уроку

- зберігати кілька значень в одній змінній (масив) і звертатися до них за номером;
- додавати елементи до списку й перевіряти, чи є в ньому значення;
- зберігати пари «ключ — значення» (хеш-таблиця);
- створювати власні об’єкти з потрібними властивостями (`[PSCustomObject]`).

## Ситуація з життя

Ви переходите на новий ноутбук і треба перенести важливі папки: «Документи», «Робочий стіл», «Зображення». Хочеться мати їх **списком**, щоб одна команда перевірила, чи всі існують, і порахувала, скільки файлів і гігабайтів доведеться переносити. А наприкінці — короткий **підсумок**: скільки папок, файлів і ГБ, щоб зберегти його поруч з копією.

## Пояснення (~15 хв)

### 1. Масив: кілька значень в одній змінній

**Масив** — це список значень через кому:

```powershell
$folders = "$HOME\Documents", "$HOME\Desktop", "$HOME\Pictures"
$folders
```

```text
C:\Users\Andrii\Documents
C:\Users\Andrii\Desktop
C:\Users\Andrii\Pictures
```

Скільки елементів у списку:

```powershell
$folders.Count
```

```text
3
```

> [!IMPORTANT]
> Елементи масиву нумеруються **з нуля**: перший — `[0]`, другий — `[1]`. Так само нумеруються частини після `-split` (урок 15). Останній елемент зручно брати як `[-1]`.

```powershell
$folders[0]     # перший: Documents
$folders[2]     # третій: Pictures
$folders[-1]    # останній: Pictures
```

Додати елемент у кінець — оператор `+=`:

```powershell
$folders += "$HOME\Videos"
$folders.Count
```

```text
4
```

> [!TIP]
> Порожній масив створюють так: `$folders = @()`. Запис `@( )` ще знадобиться, коли список треба розбити на кілька рядків: усередині `@( )` кожен елемент можна писати з нового рядка, без коми.

> [!NOTE]
> Якщо «Робочий стіл» чи «Документи» у вас в OneDrive, візьміть справжні шляхи з `[Environment]::GetFolderPath('Desktop')` і `[Environment]::GetFolderPath('MyDocuments')` (урок 5).

### 2. Список працює з командами одразу

Багато команд приймають цілий список. Перевіримо, чи всі папки існують:

```powershell
Test-Path $folders
```

```text
True
True
True
False
```

Відповідь для кожного елемента окремо, у тому самому порядку. Папки `Videos` у цьому прикладі немає.

Чи є певна папка у списку — оператори `-contains` і `-in`:

```powershell
$folders -contains "$HOME\Music"        # чи містить список...?
"$HOME\Desktop" -in $folders            # чи є ... у списку?
```

```text
False
True
```

Скільки файлів і ГБ доведеться переносити? `Get-ChildItem` теж приймає список папок:

```powershell
$files = Get-ChildItem $folders -File -Recurse -ErrorAction SilentlyContinue
$files.Count
($files | Measure-Object Length -Sum).Sum / 1GB
```

`-ErrorAction SilentlyContinue` прибирає червоні повідомлення про папки, яких немає або до яких немає доступу. Докладно про помилки — у модулі 6.

> [!TIP]
> Послідовність чисел задає оператор `..`: `1..5` — це масив `1, 2, 3, 4, 5`. Знадобиться, наприклад, щоб згенерувати назви `Папка1`…`Папка5` у модулі 6.

### 3. Хеш-таблиця: пари «ключ — значення»

Справжні назви папок англійські, а в звіті хочеться бачити українські. Для таких відповідностей є **хеш-таблиця**: набір пар «ключ = значення» у `@{ }`, кожна пара з нового рядка або через `;`:

```powershell
$names = @{
    Documents = "Документи"
    Desktop   = "Робочий стіл"
    Pictures  = "Зображення"
}
```

Значення беруть за ключем — у квадратних дужках або через крапку:

```powershell
$names["Desktop"]
$names.Pictures
```

```text
Робочий стіл
Зображення
```

Додати або змінити пару:

```powershell
$names["Videos"] = "Відео"
```

Усі ключі — `$names.Keys`, усі значення — `$names.Values`.

> [!NOTE]
> Звичайна хеш-таблиця не запам’ятовує порядок, у якому ви додавали пари, і показує їх у довільному порядку. Якщо порядок важливий, пишіть `[ordered]` перед `@{`: `$names = [ordered]@{ ... }`.

Ви вже бачили хеш-таблицю в уроці 12: обчислюваний стовпець `@{ Name = "..."; Expression = { ... } }` — це теж хеш-таблиця з двома ключами.

### 4. Власний об’єкт: [PSCustomObject]

Хеш-таблиця зручна для пошуку за ключем. А щоб отримати **рядок таблиці**, як у результатах команд, з властивостями, які можна сортувати, фільтрувати й експортувати, перетворіть її на об’єкт: допишіть `[PSCustomObject]` перед `@{`.

Складемо підсумок переносу:

```powershell
$summary = [PSCustomObject]@{
    Date    = Get-Date -Format "yyyy-MM-dd"
    Folders = $folders.Count
    Files   = $files.Count
    GB      = [math]::Round(($files | Measure-Object Length -Sum).Sum / 1GB, 1)
}
$summary
```

```text
Date       Folders Files   GB
----       ------- -----   --
2026-11-25       4  3871 18,6
```

Це такий самий об’єкт, як файл чи процес (урок 9): у нього є властивості, `$summary.GB` повертає `18,6`, а `$summary | Get-Member` покаже всі чотири властивості. Такий об’єкт можна зберегти у файл CSV, про це — в уроці 17.

> [!TIP]
> Називайте властивості **англійською**, так само як змінні: так називаються властивості в усіх командах PowerShell (`Name`, `Length`, `LastWriteTime`), і з такими назвами об’єкт без проблем піде далі конвеєром, у CSV чи інший скрипт.

#### Таблиця з українськими заголовками

Якщо підсумок потрібен саме з українськими заголовками, наприклад щоб показати комусь, не змінюйте властивості об’єкта, а перейменуйте стовпці лише **під час виведення**. Це робить `Format-Table` з обчислюваними стовпцями, як `Select-Object` в уроці 12.

> [!WARNING]
> `Format-Table` лише малює таблицю на екрані, тому вона має бути **останньою** командою конвеєра (урок 12). Якщо підсумок треба зберегти в CSV чи обробити далі, передавайте туди сам `$summary`, а не результат `Format-Table`.

```powershell
$summary | Format-Table @{ Name = "Дата";   Expression = { $_.Date } },
                        @{ Name = "Папок";  Expression = { $_.Folders } },
                        @{ Name = "Файлів"; Expression = { $_.Files } },
                        @{ Name = "ГБ";     Expression = { $_.GB } }
```

```text
Дата       Папок Файлів   ГБ
----       ----- ------   --
2026-11-25     4   3871 18,6
```

Кома в кінці рядка означає, що список стовпців продовжується на наступному рядку.

## Типові помилки

### ❌ Нумерація з одиниці

```powershell
$folders = "$HOME\Documents", "$HOME\Desktop", "$HOME\Pictures"
$folders[1]
```

```text
C:\Users\Andrii\Desktop
```

**Що сталося:** очікували першу папку («Документи»), а отримали другу.

**Чому:** нумерація починається з нуля.

### ✅ Правильно

```powershell
$folders[0]
```

---

### ❌ Номер за межами списку

```powershell
$folders[3]
```

**Що сталося:** порожньо, жодної помилки.

**Чому:** у списку три елементи з номерами 0, 1, 2. Елемента `[3]` не існує, і PowerShell мовчки повертає `$null`, як для змінної з помилкою в назві (урок 13).

### ✅ Правильно

Перевірте кількість: `$folders.Count`. Останній елемент — `$folders[-1]` або `$folders[$folders.Count - 1]`.

---

### ❌ Забули кому між елементами

```powershell
$folders = "$HOME\Documents" "$HOME\Desktop"
```

```text
Unexpected token '"$HOME\Desktop"' in expression or statement.
```

**Чому:** без коми PowerShell не розуміє, що це два елементи одного списку.

### ✅ Правильно

```powershell
$folders = "$HOME\Documents", "$HOME\Desktop"
```

Або в `@( )` з кожним елементом з нового рядка:

```powershell
$folders = @(
    "$HOME\Documents"
    "$HOME\Desktop"
)
```

---

### ❌ Номер телефону без лапок

```powershell
$phones = @{ Mama = 0501112233 }
$phones.Mama
```

```text
501112233
```

**Що сталося:** нуль на початку номера зник.

**Чому:** без лапок PowerShell сприйняв номер як **число**, а в числа нулі на початку не мають значення. Так само зникне знак `+` у `+380...`.

### ✅ Правильно

Номери телефонів, поштові індекси, номери рахунків — це текст, хоч і складається з цифр. Беріть їх у лапки:

```powershell
$phones = @{ Mama = "0501112233" }
```

## Підсумок

| Що | Як |
|---|---|
| Масив | `$a = "x", "y", "z"` або `@( ... )` |
| Порожній масив | `@()` |
| Елемент за номером | `$a[0]` (перший), `$a[-1]` (останній) |
| Кількість | `$a.Count` |
| Додати | `$a += "новий"` |
| Чи є в списку | `$a -contains "x"` або `"x" -in $a` |
| Послідовність чисел | `1..10` |
| Хеш-таблиця | `@{ Desktop = "Робочий стіл"; Pictures = "Зображення" }` |
| Значення за ключем | `$h["Desktop"]` або `$h.Desktop` |
| Зберегти порядок пар | `[ordered]@{ ... }` |
| Власний об’єкт | `[PSCustomObject]@{ Name = значення }` (назви властивостей англійською) |
| Українські заголовки на екрані | `... \| Format-Table @{ Name = "Заголовок"; Expression = { $_.Name } }` |

## Перевір себе (~5 хв)

1. Що поверне `$a[0]`, якщо `$a = "хліб", "молоко", "яйця"`?
2. Що станеться, якщо звернутися до `$a[5]` у списку з трьох елементів?
3. Чим розділяють пари в однорядковій хеш-таблиці?
4. Чим `[PSCustomObject]@{...}` відрізняється від `@{...}`?
5. Що поверне `1..4`?

<details>
<summary>Відповіді</summary>

1. `хліб` — перший елемент має номер 0.
2. Нічого (`$null`), без помилки.
3. Крапкою з комою `;`. Кома створила б список, а не нову пару.
4. `@{...}` — це хеш-таблиця для пошуку значення за ключем. `[PSCustomObject]@{...}` — об’єкт, як рядок таблиці: його можна показати таблицею, сортувати й експортувати.
5. Масив чисел `1, 2, 3, 4`.

</details>

## Практика (~10 хв)

**Завдання 1** (легке). Створіть масив `$shopping` з трьох покупок, виведіть другу з них, додайте четверту і виведіть, скільки тепер покупок у списку.

<details>
<summary>Розв’язок</summary>

```powershell
$shopping = "хліб", "молоко", "яйця"
$shopping[1]
$shopping += "сир"
$shopping.Count
```

</details>

**Завдання 2.** Створіть хеш-таблицю `$phones` з телефонами трьох людей (ключ — ім’я латиницею, значення — номер у лапках). Виведіть номер однієї людини, додайте четверту і виведіть усі імена.

<details>
<summary>Розв’язок</summary>

```powershell
$phones = @{
    Mama   = "+380501112233"
    Tato   = "+380671234567"
    Robota = "+380445556677"
}
$phones["Mama"]
$phones["Likar"] = "+380931112244"
$phones.Keys
```

Номери телефонів беріть у лапки: це текст, а не число. Інакше `+380...` перетвориться на число, і знак `+` зникне.

</details>

**Завдання 3** ⭐. Складіть список з двох папок (наприклад, «Документи» і «Зображення») і створіть `[PSCustomObject]` з властивостями `Folders`, `Files`, `GB` (з одним знаком після коми) і `AnyMissing` (`True`, якщо хоча б однієї папки немає).

<details>
<summary>Підказка</summary>

`Test-Path $folders` повертає список `True`/`False`. Перевірте, чи містить він `False`, оператором `-contains`.

</details>

<details>
<summary>Розв’язок</summary>

```powershell
$folders = "$HOME\Documents", "$HOME\Pictures"
$files = Get-ChildItem $folders -File -Recurse -ErrorAction SilentlyContinue
[PSCustomObject]@{
    Folders    = $folders.Count
    Files      = $files.Count
    GB         = [math]::Round(($files | Measure-Object Length -Sum).Sum / 1GB, 1)
    AnyMissing = (Test-Path $folders) -contains $false
}
```

</details>

## Джерела

- [about_Arrays](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_arrays?view=powershell-5.1)
- [about_Hash_Tables](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_hash_tables?view=powershell-5.1)
- [about_PSCustomObject](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_pscustomobject?view=powershell-5.1)
- [about_Comparison_Operators — `-contains`, `-in`](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_comparison_operators?view=powershell-5.1)
- [about_Operators — оператор діапазону `..`](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_operators?view=powershell-5.1)
- [Format-Table — обчислювані стовпці](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/format-table?view=powershell-5.1)
- [Test-Path](https://learn.microsoft.com/powershell/module/microsoft.powershell.management/test-path?view=powershell-5.1)

---
[← Урок 15](15-ryadky.md) · [Зміст курсу](../../README.md) · [Тест модуля 4 →](README.md#підсумковий-тест-модуля-4)
