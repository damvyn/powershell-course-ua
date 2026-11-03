# Урок 6. Створюємо, копіюємо, видаляємо

⏱ ~30 хв · Рівень: початковий · Потрібно: [урок 5](05-perekhodymo-papkamy.md), папка `PS-Kurs` у домашній папці

## Що ви вмітимете після уроку

- створювати папки й файли;
- копіювати, переміщувати й перейменовувати файли та папки;
- безпечно видаляти, спершу перевіряючи команду з `-WhatIf`;
- робити резервну копію папки у ZIP-архів.

## Ситуація з життя

Перед поїздкою ви хочете зробити резервну копію папки з документами на флешку, а заодно стиснути її в архів, щоб надіслати поштою. Мишкою — це кілька вікон і діалогів. У PowerShell — дві команди, які можна повторювати хоч щодня.

## Пояснення (~15 хв)

> [!WARNING]
> Команди цього уроку змінюють і видаляють файли. Виконуйте всі вправи **лише** в папці `PS-Kurs`, яку ви створили в уроці 5. Перед початком перейдіть у неї:
> ```powershell
> Set-Location $HOME\PS-Kurs
> ```
> Переконайтеся, що запрошення показує `PS C:\Users\<ім’я>\PS-Kurs>`.

### 1. Створюємо: New-Item

Одна команда створює і папки, і файли. Що саме створити, вказує параметр `-ItemType`:

```powershell
New-Item -ItemType Directory -Name "Звіти"        # папка
New-Item -ItemType File -Name "нотатки.txt"       # порожній файл
```

```text
    Directory: C:\Users\Andrii\PS-Kurs

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        03.11.2026     19:50                Звіти
-a----        03.11.2026     19:50              0 нотатки.txt
```

Можна одразу створити кілька вкладених папок, вказавши повний шлях:

```powershell
New-Item -ItemType Directory -Path "$HOME\PS-Kurs\Архів\2026\Листопад"
```

PowerShell сам створить усі папки, яких бракує.

> [!TIP]
> Для папок є коротша команда `mkdir`, знайома з cmd: `mkdir Фото`.

**Підготовка до вправ.** Створіть кілька файлів, з якими працюватимемо далі (кожен рядок — окрема команда):

```powershell
New-Item -ItemType File -Name "договір.docx"
New-Item -ItemType File -Name "фото-1.jpg"
New-Item -ItemType File -Name "фото-2.jpg"
New-Item -ItemType File -Name "рахунок.pdf"
```

Відкрийте папку у Провіднику (`ii .`) і переконайтеся, що файли з’явилися.

### 2. Копіюємо: Copy-Item

```powershell
Copy-Item -Path "договір.docx" -Destination "Звіти"
```

Файл скопіюється в папку `Звіти`. Якщо в `-Destination` вказати нову назву, копія отримає її:

```powershell
Copy-Item -Path "договір.docx" -Destination "Звіти\договір-копія.docx"
```

Копіювати можна одразу кілька файлів за маскою:

```powershell
Copy-Item -Path "*.jpg" -Destination "Звіти"
```

> [!IMPORTANT]
> Щоб скопіювати **папку разом із вмістом**, обов’язково додайте `-Recurse`. Без нього PowerShell скопіює лише порожню папку, і жодної помилки не покаже.

```powershell
Copy-Item -Path "Звіти" -Destination "Звіти-копія" -Recurse
```

### 3. Переміщуємо і перейменовуємо: Move-Item, Rename-Item

`Move-Item` — це «вирізати і вставити»:

```powershell
Move-Item -Path "рахунок.pdf" -Destination "Звіти"
```

`Rename-Item` змінює лише назву. У `-NewName` вказуйте **тільки нову назву**, без шляху:

```powershell
Rename-Item -Path "нотатки.txt" -NewName "план-поїздки.txt"
```

### 4. Видаляємо: Remove-Item

> [!WARNING]
> `Remove-Item` видаляє **назавжди, повз Кошик**. Відновити файл стандартними засобами Windows не вийде. Тому завжди:
> 1. спершу запускайте команду з `-WhatIf` і читайте, що саме буде видалено;
> 2. і лише потім запускайте її без `-WhatIf`.

**Спробуйте самі:**

1. Перевірте, що буде видалено:

```powershell
Remove-Item -Path "фото-*.jpg" -WhatIf
```

```text
What if: Performing the operation "Remove File" on target "C:\Users\Andrii\PS-Kurs\фото-1.jpg".
What if: Performing the operation "Remove File" on target "C:\Users\Andrii\PS-Kurs\фото-2.jpg".
```

2. Видаляються саме ті файли, які ви очікували? Тоді запустіть команду без `-WhatIf`:

```powershell
Remove-Item -Path "фото-*.jpg"
```

Папку з усім вмістом видаляють з `-Recurse`:

```powershell
Remove-Item -Path "Звіти-копія" -Recurse -WhatIf
```

### 5. Резервна копія в архів: Compress-Archive

```powershell
Compress-Archive -Path "$HOME\PS-Kurs\Звіти" -DestinationPath "$HOME\PS-Kurs\Звіти.zip"
```

Розпакувати архів можна так:

```powershell
Expand-Archive -Path "$HOME\PS-Kurs\Звіти.zip" -DestinationPath "$HOME\PS-Kurs\Розпаковано"
```

> [!NOTE]
> Якщо архів із такою назвою вже існує, `Compress-Archive` повідомить про помилку. Щоб перезаписати його, додайте `-Force`, а щоб доповнити — `-Update`.

## Типові помилки

### ❌ Скопіювали папку без вмісту

```powershell
Copy-Item -Path "Звіти" -Destination "Звіти-2"
```

**Що сталося:** з’явилася папка `Звіти-2`, але вона порожня. Помилки немає.

**Чому:** без `-Recurse` команда `Copy-Item` копіює лише саму папку.

### ✅ Правильно

Спочатку видаліть порожню копію, а потім скопіюйте ще раз з `-Recurse`:

```powershell
Remove-Item -Path "Звіти-2"
Copy-Item -Path "Звіти" -Destination "Звіти-2" -Recurse
```

> [!NOTE]
> Якщо папка `Звіти-2` вже існує, `Copy-Item -Recurse` покладе всередину неї ще одну папку: `Звіти-2\Звіти`. Тому перед повторним копіюванням порожню копію краще видалити.

---

### ❌ Копіюємо файл у папку, якої ще немає

```powershell
Copy-Item -Path "договір.docx" -Destination "Резерв"
```

**Що сталося:** замість папки `Резерв` з’явився **файл** `Резерв` без розширення. Відкрити його двічі клацнувши не вийде.

**Чому:** папки `Резерв` не існувало, тому PowerShell зрозумів `Резерв` як нову назву для копії файлу.

### ✅ Правильно

Спочатку створіть папку, потім копіюйте:

```powershell
Remove-Item -Path "Резерв"                       # видаляємо помилковий файл
New-Item -ItemType Directory -Name "Резерв"
Copy-Item -Path "договір.docx" -Destination "Резерв"
```

---

### ❌ Видалення папки без -Recurse

```powershell
Remove-Item -Path "Звіти"
```

```text
Confirm
The item at C:\Users\Andrii\PS-Kurs\Звіти has children and the Recurse parameter was not
specified. If you continue, all children will be removed with the item. Are you sure you
want to continue?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"):
```

**Чому:** у папці є файли, і PowerShell перепитує, чи ви справді хочете видалити все разом.

### ✅ Правильно

Натисніть **N**, якщо не впевнені. Перевірте вміст (`Get-ChildItem Звіти`) і, якщо його справді треба видалити, запустіть `Remove-Item -Path "Звіти" -Recurse -WhatIf`, а потім без `-WhatIf`.

---

### ❌ Шлях у новій назві

```powershell
Rename-Item -Path "план-поїздки.txt" -NewName "Звіти\план.txt"
```

```text
Rename-Item : Cannot rename the specified target, because it represents a path or device name.
```

**Чому:** `Rename-Item` уміє лише перейменовувати на місці, а не переносити в іншу папку.

### ✅ Правильно

Щоб перенести й одночасно перейменувати, використовуйте `Move-Item`:

```powershell
Move-Item -Path "план-поїздки.txt" -Destination "Звіти\план.txt"
```

---

### ❌ Квадратні дужки в назві файлу

Створіть файл `Звіт [2026].docx` вручну у Провіднику, а потім спробуйте його скопіювати:

```powershell
Copy-Item -Path "Звіт [2026].docx" -Destination "Звіти"
```

**Що сталося:** нічого. Файл не скопіювався, помилки немає.

**Чому:** квадратні дужки `[ ]` у PowerShell — це теж знак підстановки (як `*` і `?`), і `[2026]` означає «одна з цифр 2, 0, 2 або 6». Файлу з такою назвою немає, тож копіювати нічого.

### ✅ Правильно

Використовуйте параметр `-LiteralPath`: він сприймає назву буквально, без підстановок.

```powershell
Copy-Item -LiteralPath "Звіт [2026].docx" -Destination "Звіти"
```

## Підсумок

| Команда | Що робить |
|---|---|
| `New-Item -ItemType Directory -Name назва` | Створює папку |
| `New-Item -ItemType File -Name назва` | Створює порожній файл |
| `Copy-Item -Path що -Destination куди` | Копіює (для папок — з `-Recurse`) |
| `Move-Item -Path що -Destination куди` | Переміщує (можна з новою назвою) |
| `Rename-Item -Path що -NewName нова-назва` | Перейменовує на місці |
| `Remove-Item -Path що -WhatIf` | Показує, що буде видалено |
| `Remove-Item -Path що` | Видаляє назавжди, повз Кошик |
| `Compress-Archive` / `Expand-Archive` | Стискає в ZIP / розпаковує |
| `-LiteralPath` | Шлях буквально, без підстановок `* ? [ ]` |

## Перевір себе (~5 хв)

1. Що станеться після `Copy-Item -Path Фото -Destination Фото-копія`, якщо `Фото` — це папка з файлами?
2. Куди потрапляють файли, видалені командою `Remove-Item`?
3. Чим `Move-Item` відрізняється від `Rename-Item`?
4. Навіщо запускати `Remove-Item` спочатку з `-WhatIf`?
5. Коли потрібен параметр `-LiteralPath`?

<details>
<summary>Відповіді</summary>

1. Створиться **порожня** папка `Фото-копія`: без `-Recurse` вміст не копіюється.
2. Нікуди: вони видаляються назавжди, повз Кошик.
3. `Move-Item` переносить в інше місце (і може водночас перейменувати), `Rename-Item` лише змінює назву на місці.
4. Щоб побачити, які саме файли буде видалено, і переконатися, що маска не зачепила зайвого.
5. Коли в назві файлу є символи, що PowerShell сприймає як підстановку, найчастіше квадратні дужки `[ ]`.

</details>

## Практика (~10 хв)

Усі завдання виконуйте в папці `PS-Kurs`.

**Завдання 1** (легке). Створіть структуру папок `Подорожі\2026\Карпати` однією командою і перевірте результат у Провіднику.

<details>
<summary>Розв’язок</summary>

```powershell
New-Item -ItemType Directory -Path "$HOME\PS-Kurs\Подорожі\2026\Карпати"
ii "$HOME\PS-Kurs\Подорожі"
```

</details>

**Завдання 2.** Створіть файли `маршрут.txt` і `квитки.pdf` у папці `PS-Kurs`, а потім перенесіть **обидва** однією командою в папку `Подорожі\2026\Карпати`.

<details>
<summary>Підказка</summary>

`-Path` приймає кілька значень через кому.

</details>

<details>
<summary>Розв’язок</summary>

```powershell
New-Item -ItemType File -Name "маршрут.txt"
New-Item -ItemType File -Name "квитки.pdf"
Move-Item -Path "маршрут.txt", "квитки.pdf" -Destination "Подорожі\2026\Карпати"
```

</details>

**Завдання 3** ⭐. Зробіть резервну копію папки `Подорожі` у ZIP-архів `Подорожі-резерв.zip`, а потім видаліть саму папку `Подорожі`. Перед видаленням перевірте команду з `-WhatIf` і переконайтеся, що архів існує.

<details>
<summary>Розв’язок</summary>

```powershell
Compress-Archive -Path "Подорожі" -DestinationPath "Подорожі-резерв.zip"
Test-Path "Подорожі-резерв.zip"                    # має бути True
Remove-Item -Path "Подорожі" -Recurse -WhatIf
Remove-Item -Path "Подорожі" -Recurse
```

</details>

## Джерела

- [Working with files and folders](https://learn.microsoft.com/powershell/scripting/samples/working-with-files-and-folders)
- [Manipulating items directly](https://learn.microsoft.com/powershell/scripting/samples/manipulating-items-directly)
- [New-Item](https://learn.microsoft.com/powershell/module/microsoft.powershell.management/new-item?view=powershell-5.1)
- [Copy-Item](https://learn.microsoft.com/powershell/module/microsoft.powershell.management/copy-item?view=powershell-5.1)
- [Move-Item](https://learn.microsoft.com/powershell/module/microsoft.powershell.management/move-item?view=powershell-5.1)
- [Rename-Item](https://learn.microsoft.com/powershell/module/microsoft.powershell.management/rename-item?view=powershell-5.1)
- [Remove-Item](https://learn.microsoft.com/powershell/module/microsoft.powershell.management/remove-item?view=powershell-5.1)
- [Compress-Archive](https://learn.microsoft.com/powershell/module/microsoft.powershell.archive/compress-archive?view=powershell-5.1)
- [about_Wildcards — квадратні дужки](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_wildcards?view=powershell-5.1)

---
[← Урок 5](05-perekhodymo-papkamy.md) · [Зміст курсу](../../README.md) · [Урок 7. Текстові файли →](07-tekstovi-faily.md)
