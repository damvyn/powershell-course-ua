# Модуль 1. Знайомство

У цьому модулі ви відкриєте PowerShell, виконаєте перші команди і навчитеся самостійно знаходити потрібну команду та довідку до неї. Це фундамент для всього курсу.

| Урок | Тема | Час |
|---|---|---|
| [1](01-shcho-take-powershell.md) | Що таке PowerShell і навіщо він мені | ~30 хв |
| [2](02-pershi-komandy.md) | Перші команди | ~30 хв |
| [3](03-poshuk-komand.md) | Як знайти потрібну команду | ~30 хв |
| [4](04-parametry-ta-psevdonimy.md) | Параметри та псевдоніми | ~30 хв |
| [Тест](#підсумковий-тест-модуля-1) | 5 питань і 5 практичних завдань | ~20 хв |

## Підсумковий тест модуля 1

Спершу виконайте все самостійно і лише потім розгортайте відповіді. Якщо помилилися, поверніться до уроку, вказаного в дужках.

### Питання

**1.** Ви відкрили вікно, і в ньому написано `C:\Users\Olena>`. Це PowerShell? Як це перевірити? *(урок 1)*

**2.** Колега надіслав вам команду `Restart-Computer`. Не запускаючи її, поясніть з назви, що вона зробить. *(урок 2)*

**3.** Яка з цих команд покаже **лише приклади** використання `Get-ChildItem`? *(урок 3)*
- а) `Get-Command Get-ChildItem`
- б) `Get-Help Get-ChildItem -Examples`
- в) `Get-ChildItem -Examples`

**4.** Знайдіть помилку в команді й виправте її: *(урок 4)*

```powershell
Get-ChildItem C:\Users\Olena\My Photos -Recurse
```

**5.** Чим відрізняється запуск `Stop-Process -Name notepad -WhatIf` від `Stop-Process -Name notepad`? *(урок 4)*

<details>
<summary>Відповіді на питання</summary>

1. Ні, це командний рядок cmd: у запрошенні PowerShell на початку є `PS`, наприклад `PS C:\Users\Olena>`. Перевірити можна командою `$PSVersionTable`: у cmd вона видасть помилку «is not recognized as an internal or external command».
2. Дієслово `Restart` — перезапустити, іменник `Computer` — комп’ютер. Команда перезавантажить комп’ютер.
3. **б)**.
4. У шляху є пробіл, тому його треба взяти в лапки:
   ```powershell
   Get-ChildItem "C:\Users\Olena\My Photos" -Recurse
   ```
5. З `-WhatIf` нічого не відбувається, PowerShell лише описує, що зробив би. Без `-WhatIf` усі Блокноти одразу закриються без запитання про збереження.

</details>

### Практичні завдання

**Завдання 1.** Дізнайтеся, яку версію PowerShell ви використовуєте. Виведіть **лише номер версії**, без решти таблиці. *(урок 1)*

<details>
<summary>Розв’язок</summary>

```powershell
$PSVersionTable.PSVersion
```

</details>

**Завдання 2.** Запустіть Paint з PowerShell, а потім знайдіть його процес командою `Get-Process`. *(урок 2)*

<details>
<summary>Розв’язок</summary>

```powershell
mspaint
Get-Process -Name mspaint
```

</details>

**Завдання 3.** Знайдіть команду, яка показує вміст буфера обміну (підказка: «буфер обміну» англійською — *Clipboard*). Скопіюйте мишкою будь-який текст у браузері та виведіть його в PowerShell. *(урок 3)*

<details>
<summary>Розв’язок</summary>

```powershell
Get-Command *Clipboard*
```

Знайдуться `Get-Clipboard` (отримати вміст) і `Set-Clipboard` (записати). Скопіюйте текст і виконайте:

```powershell
Get-Clipboard
```

</details>

**Завдання 4.** Покажіть усі файли `.docx` у папці «Документи» разом із вкладеними папками. Потім напишіть ту саму команду через псевдонім `dir`. *(урок 4)*

> [!NOTE]
> Якщо у вас увімкнено OneDrive, «Документи» можуть бути в `$HOME\OneDrive\Documents`. Тоді вкажіть цей шлях.

<details>
<summary>Розв’язок</summary>

```powershell
Get-ChildItem -Path $HOME\Documents -Filter *.docx -Recurse
dir $HOME\Documents -Filter *.docx -Recurse
```

</details>

**Завдання 5.** Відкрийте Калькулятор. Перевірте з `-WhatIf`, що станеться, якщо його зупинити, а потім закрийте його командою з запитом підтвердження. Назву процесу Калькулятора ви знаєте з уроку 2. *(уроки 2 і 4)*

<details>
<summary>Розв’язок</summary>

Windows 11:

```powershell
calc
Stop-Process -Name CalculatorApp -WhatIf
Stop-Process -Name CalculatorApp -Confirm   # натисніть Y
```

У Windows 10 замість `CalculatorApp` вкажіть `Calculator`.

</details>

---
[← Урок 4](04-parametry-ta-psevdonimy.md) · [Зміст курсу](../../README.md)
