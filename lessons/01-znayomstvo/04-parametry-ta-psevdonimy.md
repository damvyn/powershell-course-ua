# Урок 4. Параметри та псевдоніми

⏱ ~30 хв · Рівень: початковий · Потрібно: [урок 3](03-poshuk-komand.md)

## Що ви вмітимете після уроку

- уточнювати роботу команди за допомогою параметрів;
- правильно вказувати шляхи з пробілами;
- безпечно перевіряти небезпечні команди з `-WhatIf` і `-Confirm`;
- користуватися короткими назвами команд (псевдонімами) і знаходити, що за ними стоїть.

## Ситуація з життя

Папка «Завантаження» переповнена, і вам потрібно знайти в ній усі PDF-файли, разом із вкладеними папками. Команда `Get-ChildItem` покаже весь вміст папки. Але якщо додати до неї кілька **параметрів**, вона покаже саме PDF-файли, і саме там, де треба.

## Пояснення (~15 хв)

### 1. Що таке параметр

Параметр — це **уточнення** до команди. Повернімося до помічника з уроку 1: «принеси книжку» — це команда, а «принеси книжку **з верхньої полиці**, **червону**» — команда з параметрами.

Параметр завжди починається з дефіса `-`, а після нього через пробіл іде **значення**:

```text
Команда  -Параметр Значення  -Параметр Значення
```

**Спробуйте самі:**

1. Відкрийте Блокнот.
2. У PowerShell введіть:

```powershell
Get-Process -Name notepad
```

Тут `-Name` — параметр, а `notepad` — його значення. Команда покаже лише процеси з назвою `notepad`.

Параметр може приймати кілька значень через кому:

```powershell
Get-Process -Name notepad, explorer
```

### 2. Шлях до папки як параметр

Команда `Get-ChildItem` показує вміст папки. Параметр `-Path` вказує, яку саме папку:

```powershell
Get-ChildItem -Path C:\Windows
```

Додамо параметр `-Filter`, щоб показати лише файли певного типу:

```powershell
Get-ChildItem -Path C:\Windows -Filter *.exe
```

Тут `*.exe` означає «будь-яка назва, що закінчується на `.exe`».

### 3. Позиційні параметри: можна не писати назву

Деякі параметри настільки часто використовуються, що їхню назву можна пропустити. Їх називають **позиційними**: PowerShell розуміє, що мається на увазі, за місцем (позицією) значення в команді.

Ці дві команди роблять те саме:

```powershell
Get-ChildItem -Path C:\Windows -Filter *.exe
Get-ChildItem C:\Windows *.exe
```

Перше значення PowerShell вважає шляхом (`-Path`), друге — фільтром (`-Filter`). Позиції описані в повній довідці (`Get-Help Get-ChildItem -Full`, рядок `Position?`).

> [!TIP]
> Поки ви вчитеся, пишіть назви параметрів повністю. Так команду легше прочитати і важче помилитися. Скорочений запис зручний, коли ви вже добре знаєте команду.

### 4. Перемикачі: параметри без значення

Деякі параметри працюють як вимикач світла: або увімкнені, або ні. Їх називають **перемикачами** (switch). Значення після них не потрібне.

```powershell
Get-ChildItem -Path $HOME\Downloads -Filter *.pdf -Recurse
```

`-Recurse` означає «зазирни також у всі вкладені папки». Без нього команда перегляне тільки саму папку «Завантаження».

Ще один корисний перемикач `-Force` показує також приховані файли:

```powershell
Get-ChildItem -Path $HOME -Force
```

> [!NOTE]
> `$HOME` — це ваша домашня папка, наприклад `C:\Users\Andrii`. Тому `$HOME\Downloads` означає папку «Завантаження». Так команда працюватиме на будь-якому комп’ютері, хоч би яке там було ім’я користувача.

### 5. Шляхи з пробілами беремо в лапки

Якщо в шляху є пробіл, візьміть увесь шлях у лапки:

```powershell
Get-ChildItem -Path "C:\Program Files"
```

Без лапок PowerShell розріже шлях на два окремі значення (див. «Типові помилки»).

### 6. Безпечна перевірка: -WhatIf і -Confirm

Команди, що щось змінюють або видаляють, зазвичай мають два «запобіжники»:

- `-WhatIf` — «що буде, якщо?». Команда **нічого не робить**, лише описує, що зробила б.
- `-Confirm` — команда перед кожною дією **питає дозволу**.

**Спробуйте самі:**

1. Відкрийте Блокнот і нічого в ньому не пишіть.
2. Введіть у PowerShell:

```powershell
Stop-Process -Name notepad -WhatIf
```

```text
What if: Performing the operation "Stop-Process" on target "notepad (12345)".
```

Блокнот залишився відкритим: це була лише перевірка. Тепер із запитом підтвердження:

```powershell
Stop-Process -Name notepad -Confirm
```

```text
Confirm
Are you sure you want to perform this action?
Performing the operation "Stop-Process" on target "notepad (12345)".
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"):
```

3. Натисніть **N** і Enter: Блокнот залишиться відкритим. Якщо натиснути **Y**, він закриється.

> [!WARNING]
> `Stop-Process` закриває програму **без** запитання «Зберегти зміни?». Усе незбережене в ній пропаде. Тому перед справжнім запуском таких команд завжди спочатку перевіряйте їх з `-WhatIf`.

### 7. Псевдоніми: короткі назви команд

Довгі назви зручні для читання, але не для набору. Тому в багатьох команд є **псевдоніми** (alias), тобто короткі імена.

| Псевдонім | Повна команда | Звідки знайомий |
|---|---|---|
| `dir`, `ls`, `gci` | `Get-ChildItem` | `dir` з cmd, `ls` з Linux |
| `cd`, `sl` | `Set-Location` | cmd, Linux |
| `cls`, `clear` | `Clear-Host` | cmd, Linux |
| `copy`, `cp` | `Copy-Item` | cmd, Linux |
| `del`, `rm` | `Remove-Item` | cmd, Linux |
| `gps`, `ps` | `Get-Process` | Linux |

Що стоїть за псевдонімом:

```powershell
Get-Alias dir
```

```text
CommandType     Name                      Version    Source
-----------     ----                      -------    ------
Alias           dir -> Get-ChildItem
```

Які псевдоніми є в команди:

```powershell
Get-Alias -Definition Get-ChildItem
```

```text
CommandType     Name
-----------     ----
Alias           dir -> Get-ChildItem
Alias           gci -> Get-ChildItem
Alias           ls -> Get-ChildItem
```

Псевдонім працює так само, як повна команда, з тими самими параметрами:

```powershell
dir $HOME\Downloads -Filter *.pdf -Recurse
```

> [!TIP]
> Псевдоніми зручні, коли ви вводите команди вручну. У скриптах, які читатимуть інші люди (або ви самі через пів року), пишіть повні назви команд.

> [!NOTE]
> **PowerShell 7** на Windows має ті самі псевдоніми. А на Linux і macOS у PowerShell 7 немає псевдонімів `ls`, `cp`, `rm`, `mv`, `cat` та кількох інших, бо там ці імена належать вбудованим програмам системи. Тому `ls` у Linux запустить звичайну програму Linux, а не `Get-ChildItem`.

## Типові помилки

### ❌ Шлях із пробілом без лапок

```powershell
cd C:\Program Files
```

```text
Set-Location : A positional parameter cannot be found that accepts argument 'Files'.
```

**Чому:** пробіл розділяє значення. PowerShell отримав два значення: `C:\Program` і `Files`. Перше він вважає шляхом, а для другого немає місця. Помилка «A positional parameter cannot be found» майже завжди означає зайвий пробіл або забуті лапки.

### ✅ Правильно

```powershell
cd "C:\Program Files"
```

---

### ❌ Звичка з командного рядка cmd: dir /s

```powershell
dir /s
```

```text
Get-ChildItem : Cannot find path 'C:\s' because it does not exist.
```

**Чому:** `dir` у PowerShell — лише псевдонім `Get-ChildItem`, а не команда cmd. Параметри PowerShell починаються з `-`, а не з `/`. Тому `/s` PowerShell вважає шляхом до папки `C:\s`, якої немає.

### ✅ Правильно

```powershell
dir -Recurse
```

---

### ❌ Забули значення параметра

```powershell
Get-Process -Name
```

```text
Get-Process : Missing an argument for parameter 'Name'. Specify a parameter of type
'System.String[]' and try again.
```

**Чому:** параметр `-Name` вимагає значення (назву процесу), а його немає. `System.String[]` означає «текст, можна кілька значень».

### ✅ Правильно

```powershell
Get-Process -Name notepad
```

---

### ❌ Одруківка в назві параметра

```powershell
Get-Process -Nmae notepad
```

```text
Get-Process : A parameter cannot be found that matches parameter name 'Nmae'.
```

**Чому:** у команди немає параметра `-Nmae`. Повідомлення «A parameter cannot be found» означає неіснуючий параметр.

### ✅ Правильно

Наберіть `-Na` і натисніть **Tab**: PowerShell доповнює не лише команди, а й назви параметрів.

---

### ❌ Одразу зупиняємо програму без перевірки

```powershell
Stop-Process -Name chrome
```

**Що сталося:** закрилися **всі** вікна Chrome разом з усіма вкладками, без жодного запитання.

**Чому:** `Stop-Process` зупиняє кожен процес з такою назвою, а в браузера їх зазвичай десятки.

### ✅ Правильно

Спочатку подивіться, що буде зроблено:

```powershell
Stop-Process -Name chrome -WhatIf
```

і лише потім вирішуйте, чи запускати команду без `-WhatIf`.

## Підсумок

| Що | Приклад |
|---|---|
| Параметр зі значенням | `Get-Process -Name notepad` |
| Кілька значень | `Get-Process -Name notepad, explorer` |
| Позиційний параметр | `Get-ChildItem C:\Windows` |
| Перемикач | `Get-ChildItem -Recurse` |
| Шлях із пробілом | `Get-ChildItem "C:\Program Files"` |
| Перевірка без виконання | `Stop-Process -Name notepad -WhatIf` |
| Запит підтвердження | `Stop-Process -Name notepad -Confirm` |
| Що стоїть за псевдонімом | `Get-Alias dir` |
| Псевдоніми команди | `Get-Alias -Definition Get-ChildItem` |

## Перевір себе (~5 хв)

1. Що в команді `Get-ChildItem -Path C:\Windows -Filter *.exe` є параметрами, а що — їхніми значеннями?
2. Чим перемикач відрізняється від звичайного параметра? Наведіть приклад перемикача.
3. Чому команда `cd C:\Program Files` не працює і як її виправити?
4. Що станеться після виконання `Stop-Process -Name notepad -WhatIf`?
   - а) Блокнот закриється
   - б) Блокнот закриється після підтвердження
   - в) нічого не закриється, PowerShell лише опише, що зробив би
5. Яка повна команда стоїть за псевдонімом `ls` і як це перевірити?

<details>
<summary>Відповіді</summary>

1. Параметри: `-Path` і `-Filter`. Значення: `C:\Windows` і `*.exe`.
2. Перемикач не потребує значення: він або вказаний (увімкнений), або ні. Приклади: `-Recurse`, `-Force`, `-WhatIf`.
3. Через пробіл PowerShell отримує два значення, `C:\Program` і `Files`. Шлях треба взяти в лапки: `cd "C:\Program Files"`.
4. **в)**.
5. `Get-ChildItem`. Перевірити можна командою `Get-Alias ls`.

</details>

## Практика (~10 хв)

**Завдання 1** (легке). Покажіть усі PDF-файли в папці «Завантаження», включно з вкладеними папками.

<details>
<summary>Розв’язок</summary>

```powershell
Get-ChildItem -Path $HOME\Downloads -Filter *.pdf -Recurse
```

Якщо результату немає, у «Завантаженнях» просто немає PDF-файлів. Спробуйте інший тип, наприклад `*.jpg` або `*.docx`.

</details>

**Завдання 2.** Відкрийте два вікна Блокнота. Спочатку перевірте з `-WhatIf`, що зробить команда їх зупинки. Потім закрийте **лише одне** з них за допомогою `-Confirm`.

<details>
<summary>Підказка</summary>

`-Confirm` питає окремо про кожен процес. На одне запитання відповідайте **Y**, на інше — **N**.

</details>

<details>
<summary>Розв’язок</summary>

```powershell
Stop-Process -Name notepad -WhatIf    # покаже два рядки What if: ... для двох процесів
Stop-Process -Name notepad -Confirm   # на перше запитання Y, на друге N
```

> У Windows 11 нові вікна Блокнота можуть відкриватися як вкладки одного процесу. Тоді `-WhatIf` покаже лише один рядок. Щоб отримати два процеси, увімкніть у налаштуваннях Блокнота відкриття файлів у новому вікні.

</details>

**Завдання 3** ⭐. Знайдіть усі псевдоніми команди `Remove-Item`. Потім перевірте, яка команда стоїть за псевдонімом `gps`.

<details>
<summary>Розв’язок</summary>

```powershell
Get-Alias -Definition Remove-Item
Get-Alias gps
```

Псевдоніми `Remove-Item`: `del`, `erase`, `rd`, `ri`, `rm`, `rmdir`. За `gps` стоїть `Get-Process`.

</details>

## Джерела

- [about_Parameters](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_parameters?view=powershell-5.1)
- [about_Aliases](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_aliases?view=powershell-5.1)
- [about_CommonParameters — WhatIf і Confirm](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_commonparameters?view=powershell-5.1)
- [Get-ChildItem](https://learn.microsoft.com/powershell/module/microsoft.powershell.management/get-childitem?view=powershell-5.1)
- [Stop-Process](https://learn.microsoft.com/powershell/module/microsoft.powershell.management/stop-process?view=powershell-5.1)
- [Differences between Windows PowerShell 5.1 and PowerShell 7.x](https://learn.microsoft.com/powershell/scripting/whats-new/differences-from-windows-powershell)

---
[← Урок 3](03-poshuk-komand.md) · [Зміст курсу](../../README.md) · [Підсумковий тест модуля 1 →](README.md)
