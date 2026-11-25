# Урок 10. Конвеєр «|»

⏱ ~30 хв · Рівень: початковий · Потрібно: [урок 9](09-use-ye-obyektom.md)

## Що ви вмітимете після уроку

- передавати результат однієї команди в іншу через `|`;
- сортувати результати за будь-якою властивістю (`Sort-Object`);
- залишати лише потрібні стовпці й перші або останні рядки (`Select-Object`);
- безпечно передавати знайдені об’єкти командам, які щось змінюють.

## Ситуація з життя

На диску закінчується місце. Хочеться побачити **10 найбільших файлів** у папці «Завантаження» і подумати, що з них можна видалити. Провідник уміє сортувати за розміром, але лише в одній папці. PowerShell зробить це одним рядком і для всіх вкладених папок.

## Пояснення (~15 хв)

### 1. Як працює конвеєр

Символ `|` (вертикальна риска, на клавіатурі зазвичай **Shift + \\**) з’єднує команди в ланцюжок. Кожна команда отримує об’єкти від попередньої, щось із ними робить і передає далі.

```text
Get-ChildItem   →   Sort-Object   →   Select-Object
 (знайти файли)     (відсортувати)     (взяти перші 10)
```

Це як конвеєр на заводі: деталь проходить кілька станцій, і на кожній з нею щось роблять.

> [!TIP]
> Довгий конвеєр можна розбити на кілька рядків: поставте `|` у **кінці** рядка і натисніть **Enter**. PowerShell зрозуміє, що команда не закінчена, і покаже `>>` для продовження. Коли допишете, натисніть **Enter** ще раз на порожньому рядку.

### 2. Сортуємо: Sort-Object

```powershell
Get-ChildItem $HOME\Downloads -File | Sort-Object -Property Length
```

Файли відсортуються за розміром від найменшого до найбільшого. Щоб навпаки, додайте `-Descending`:

```powershell
Get-ChildItem $HOME\Downloads -File | Sort-Object -Property Length -Descending
```

Сортувати можна за будь-якою властивістю, яку ви знайшли через `Get-Member`: за назвою (`Name`), датою зміни (`LastWriteTime`), розширенням (`Extension`).

### 3. Обираємо: Select-Object

`Select-Object` уміє дві речі.

**Взяти лише кілька перших або останніх об’єктів:**

```powershell
Get-ChildItem $HOME\Downloads -File | Sort-Object Length -Descending | Select-Object -First 10
```

Ось і 10 найбільших файлів. `-Last 5` дасть п’ять останніх.

**Залишити лише потрібні властивості (стовпці):**

```powershell
Get-ChildItem $HOME\Downloads -File |
    Sort-Object Length -Descending |
    Select-Object -First 10 -Property Name, Length, LastWriteTime
```

```text
Name                         Length LastWriteTime
----                         ------ -------------
Win11_24H2_Ukrainian_x64.iso 6123847680 12.10.2026 18:22:15
відео-з-весілля.mp4          1843200512 03.09.2026 21:05:44
Setup_Office.exe              412876544 15.06.2026 10:11:02
...
```

> [!NOTE]
> Назви параметрів `-Property` у `Sort-Object` і `Select-Object` можна не писати: `Sort-Object Length` означає те саме, що `Sort-Object -Property Length`. Так коротше, і ви часто бачитимете такий запис у прикладах.

### 4. Конвеєр до команд, що змінюють

Найсильніше конвеєр проявляє себе, коли наприкінці стоїть команда, яка **щось робить**: зупиняє, видаляє, копіює.

> [!WARNING]
> Такий конвеєр діє на **все**, що до нього дійде. Якщо фільтр перед ним помилився, зміни зачеплять зайве. Тому спершу завжди запускайте ланцюжок з `-WhatIf` в останній команді і уважно читайте, що буде зроблено.

**Спробуйте самі:**

1. Відкрийте два Блокноти: двічі виконайте `notepad`.
2. Перевірте, що буде закрито:

```powershell
Get-Process -Name notepad | Stop-Process -WhatIf
```

```text
What if: Performing the operation "Stop-Process" on target "Notepad (8124)".
What if: Performing the operation "Stop-Process" on target "Notepad (9310)".
```

3. Закрийте Блокноти вручну (хрестиком), бо в них може бути незбережений текст.

Так само можна, наприклад, скопіювати в архівну папку всі PDF з «Завантажень»:

```powershell
Get-ChildItem $HOME\Downloads -Filter *.pdf | Copy-Item -Destination $HOME\PS-Kurs -WhatIf
```

> [!NOTE]
> **PowerShell 7:** з’явилися оператори `&&` і `||`, знайомі з cmd. `команда1 && команда2` запускає другу команду, лише якщо перша завершилася успішно, а `||` — лише якщо з помилкою. У Windows PowerShell 5.1 їх немає, і такий рядок дасть помилку `The token '&&' is not a valid statement separator in this version.`

## Типові помилки

### ❌ Спочатку взяли перші, потім відсортували

```powershell
Get-ChildItem $HOME\Downloads -File | Select-Object -First 10 | Sort-Object Length -Descending
```

**Що сталося:** ви отримали не 10 найбільших файлів, а 10 **перших за абеткою**, відсортованих за розміром.

**Чому:** конвеєр виконується зліва направо. `Select-Object -First 10` узяв перші 10 файлів, а `Sort-Object` побачив лише їх.

### ✅ Правильно

Спершу сортуємо всі, потім беремо перші:

```powershell
Get-ChildItem $HOME\Downloads -File | Sort-Object Length -Descending | Select-Object -First 10
```

---

### ❌ Помилка в назві стовпця

```powershell
Get-ChildItem $HOME\Downloads -File | Select-Object Name, Size
```

```text
Name                    Size
----                    ----
договір.pdf
фото.jpg
```

**Що сталося:** стовпець `Size` порожній, жодної помилки.

**Чому:** такої властивості в файлів немає (розмір — це `Length`). `Select-Object` просто створює порожній стовпець.

### ✅ Правильно

```powershell
Get-ChildItem $HOME\Downloads -File | Select-Object Name, Length
```

Сумніваєтесь у назві — перевірте через `Get-Member` (урок 9).

---

### ❌ Конвеєр зі змінами без перевірки

```powershell
Get-ChildItem $HOME\Downloads | Remove-Item
```

**Що сталося:** PowerShell почав видаляти **все** в «Завантаженнях», а для кожної вкладеної папки ще й перепитує. Видалене не потрапляє до Кошика.

**Чому:** автор хотів видалити лише старі інсталятори, але забув фільтр і не перевірив команду.

### ✅ Правильно

Звужуйте вибірку і перевіряйте з `-WhatIf`:

```powershell
Get-ChildItem $HOME\Downloads -Filter *.exe | Remove-Item -WhatIf
```

Як відібрати файли ще точніше (наприклад, старші за місяць), ви дізнаєтеся в наступному уроці.

## Підсумок

| Команда | Що робить |
|---|---|
| `команда1 \| команда2` | Передає результат першої команди другій |
| `Sort-Object Властивість` | Сортує за зростанням |
| `Sort-Object Властивість -Descending` | Сортує за спаданням |
| `Select-Object -First N` / `-Last N` | Перші / останні N об’єктів |
| `Select-Object Влас1, Влас2` | Лише вибрані стовпці |
| `... \| Stop-Process -WhatIf` | Перевірка, що зробить конвеєр |

## Перевір себе (~5 хв)

1. У якому порядку виконуються команди в конвеєрі?
2. Як відсортувати процеси за назвою у зворотному порядку (від Я до А)?
3. Чим `Select-Object -First 3` відрізняється від `Select-Object Name, Id`?
4. Чому важливо ставити `Sort-Object` **перед** `Select-Object -First`?
5. Навіщо `-WhatIf` у кінці конвеєра, який видаляє файли?

<details>
<summary>Відповіді</summary>

1. Зліва направо: кожна команда отримує результат попередньої.
2. `Get-Process | Sort-Object Name -Descending`.
3. `-First 3` залишає три перші **рядки** (об’єкти), а `Name, Id` залишає всі рядки, але лише два **стовпці**.
4. Бо інакше ви відсортуєте вже обрізаний список, а не всі об’єкти.
5. Щоб побачити, які саме файли буде видалено, до того як вони зникнуть назавжди.

</details>

## Практика (~10 хв)

**Завдання 1** (легке). Виведіть 5 процесів, які займають найбільше пам’яті. Пам’ять процесу зберігається у властивості `WorkingSet`.

<details>
<summary>Розв’язок</summary>

```powershell
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 5
```

</details>

**Завдання 2.** Виведіть 3 файли з папки «Документи» (разом із вкладеними папками), які ви змінювали **найостанніше**. Показуйте лише назву і дату зміни.

<details>
<summary>Розв’язок</summary>

```powershell
Get-ChildItem $HOME\Documents -File -Recurse |
    Sort-Object LastWriteTime -Descending |
    Select-Object -First 3 -Property Name, LastWriteTime
```

Якщо «Документи» у вас в OneDrive, замініть шлях на `$HOME\OneDrive\Documents`.

</details>

**Завдання 3** ⭐. Знайдіть **найбільший** файл на Робочому столі та скопіюйте його в папку `PS-Kurs`. Спершу перевірте команду з `-WhatIf`.

<details>
<summary>Розв’язок</summary>

```powershell
Get-ChildItem $HOME\Desktop -File |
    Sort-Object Length -Descending |
    Select-Object -First 1 |
    Copy-Item -Destination $HOME\PS-Kurs -WhatIf
```

Якщо все правильно, запустіть те саме без `-WhatIf`. Якщо Робочий стіл в OneDrive, візьміть шлях з `[Environment]::GetFolderPath('Desktop')` (урок 5).

</details>

## Джерела

- [about_Pipelines](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_pipelines?view=powershell-5.1)
- [One-liners and the pipeline (PowerShell 101)](https://learn.microsoft.com/powershell/scripting/learn/ps101/04-pipelines)
- [Sort-Object](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/sort-object?view=powershell-5.1)
- [Select-Object](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/select-object?view=powershell-5.1)
- [about_Pipeline_Chain_Operators (PowerShell 7)](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_pipeline_chain_operators?view=powershell-7.5)

---
[← Урок 9](09-use-ye-obyektom.md) · [Зміст курсу](../../README.md) · [Урок 11. Фільтруємо дані →](11-filtruyemo-dani.md)
