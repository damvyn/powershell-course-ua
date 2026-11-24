# Урок 5. Переходимо папками

⏱ ~30 хв · Рівень: початковий · Потрібно: [модуль 1](../01-znayomstvo/README.md)

## Що ви вмітимете після уроку

- розуміти, що таке шлях до файлу, і відрізняти повний шлях від відносного;
- переходити між папками командою `Set-Location`;
- переглядати лише файли, лише папки або приховані елементи;
- перевіряти, чи існує файл або папка;
- відкривати файл чи папку з PowerShell у звичайній програмі.

## Ситуація з життя

У Провіднику ви шукаєте потрібний документ, клацаючи папку за папкою: «Документи» → «Робота» → «2026» → «Звіти». У PowerShell той самий шлях можна пройти одним рядком, а потім одразу побачити всі файли Word у цій папці, навіть приховані.

## Пояснення (~15 хв)

### 1. Що таке шлях

Кожен файл і кожна папка мають **адресу** — шлях. Він схожий на поштову адресу: від найбільшого до найменшого, через зворотну косу риску `\`.

```text
C:\Users\Andrii\Documents\Звіт.docx
│  │     │      │         └── файл
│  │     │      └── папка «Документи»
│  │     └── папка користувача
│  └── папка всіх користувачів
└── диск C:
```

Шляхи бувають двох видів:

| Вид | Приклад | Як читати |
|---|---|---|
| **Повний** (абсолютний) | `C:\Users\Andrii\Documents` | Повна адреса від диска. Працює звідусіль |
| **Відносний** | `Documents` або `..\Downloads` | Адреса відносно папки, де ви зараз. Як «другі двері ліворуч» |

У відносних шляхах є два спеціальні позначення:

- `.` (одна крапка) — **поточна** папка;
- `..` (дві крапки) — папка **на рівень вище** (батьківська).

> [!IMPORTANT]
> У Провіднику Windows показує назви системних папок українською («Документи», «Завантаження», «Робочий стіл»), але **справжні** назви цих папок англійські: `Documents`, `Downloads`, `Desktop`. У PowerShell завжди використовуйте англійські назви, інакше отримаєте помилку «Cannot find path».

| У Провіднику | Справжня назва |
|---|---|
| Документи | `Documents` |
| Завантаження | `Downloads` |
| Робочий стіл | `Desktop` |
| Зображення | `Pictures` |
| Музика | `Music` |
| Відео | `Videos` |

### 2. Де я зараз і як перейти в іншу папку

Поточну папку показує запрошення `PS C:\Users\Andrii>` і команда:

```powershell
Get-Location
```

Перейти в іншу папку можна командою `Set-Location` (псевдонім `cd`):

> [!TIP]
> Не набирайте довгі шляхи вручну. Наберіть перші літери назви папки і натисніть **Tab**: PowerShell допише назву й сам візьме її в лапки, якщо в ній є пробіл.

**Спробуйте самі:**

1. Перейдіть у папку «Завантаження» за повним шляхом (замініть `Andrii` на ім’я своєї папки користувача або скористайтеся `$HOME`):

```powershell
Set-Location $HOME\Downloads
```

Запрошення зміниться:

```text
PS C:\Users\Andrii\Downloads>
```

2. Підніміться на рівень вище:

```powershell
Set-Location ..
```

```text
PS C:\Users\Andrii>
```

3. Перейдіть у «Документи» за відносним шляхом (ви вже в `C:\Users\Andrii`, тож достатньо назви):

```powershell
Set-Location Documents
```

4. Поверніться додому одним рухом:

```powershell
Set-Location ~
```

Знак `~` (тильда) означає вашу домашню папку, як і `$HOME`.

> [!NOTE]
> PowerShell розуміє обидві риски, `\` і `/`. Команди `cd C:\Windows` і `cd C:/Windows` працюють однаково. У курсі ми пишемо `\`, як заведено у Windows.

> [!NOTE]
> **PowerShell 7:** команда `cd -` повертає в попередню папку, а `cd` без шляху переносить у домашню папку. У Windows PowerShell 5.1 цього немає.

### 3. Що лежить у папці: Get-ChildItem

З уроку 4 ви вже знаєте `Get-ChildItem` (псевдоніми `dir`, `ls`). Результат виглядає так:

```text
    Directory: C:\Users\Andrii\Documents

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        14.10.2026     09:12                Робота
-a----        02.11.2026     18:40          24576 Звіт.docx
-a----        28.10.2026     11:05         102400 Фото.jpg
```

| Стовпчик | Що означає |
|---|---|
| **Mode** | `d` — папка (directory), `a` — файл; `h` — прихований, `r` — лише для читання |
| **LastWriteTime** | Коли востаннє змінювали |
| **Length** | Розмір у байтах (у папок порожньо) |
| **Name** | Назва |

Корисні перемикачі:

```powershell
Get-ChildItem -File          # лише файли
Get-ChildItem -Directory     # лише папки
Get-ChildItem -Force         # разом із прихованими
```

Знаки підстановки (wildcards) допомагають відібрати файли за назвою:

| Знак | Значення | Приклад | Знайде |
|---|---|---|---|
| `*` | будь-яка кількість будь-яких символів | `*.pdf` | усі PDF |
| `?` | рівно один будь-який символ | `Звіт-?.docx` | `Звіт-1.docx`, `Звіт-2.docx`, але не `Звіт-10.docx` |

```powershell
Get-ChildItem $HOME\Downloads\*.pdf
Get-ChildItem $HOME\Pictures\IMG_2026*
```

### 4. Чи існує файл: Test-Path

Перш ніж працювати з файлом, корисно перевірити, що він є:

```powershell
Test-Path $HOME\Documents
Test-Path $HOME\Documents\Нема-такого-файлу.txt
```

```text
True
False
```

`True` означає «так, існує», `False` — «ні».

### 5. Відкрити файл чи папку: Invoke-Item

`Invoke-Item` (псевдонім `ii`) робить те саме, що подвійне клацання в Провіднику: відкриває файл у програмі за замовчуванням, а папку — у Провіднику.

```powershell
Invoke-Item $HOME\Downloads        # відкриє «Завантаження» у Провіднику
Invoke-Item .                      # відкриє поточну папку
```

## Типові помилки

### ❌ Українська назва системної папки

```powershell
Set-Location $HOME\Документи
```

```text
Set-Location : Cannot find path 'C:\Users\Andrii\Документи' because it does not exist.
```

**Чому:** «Документи» — це лише підпис у Провіднику. Справжня назва папки — `Documents`.

### ✅ Правильно

```powershell
Set-Location $HOME\Documents
```

---

### ❌ Робочий стіл або Документи «не там»

```powershell
Get-ChildItem $HOME\Desktop
```

Команда працює, але показує порожню папку або не ті файли, що ви бачите на екрані.

**Чому:** якщо увімкнено **OneDrive**, Windows часто переносить «Робочий стіл», «Документи» і «Зображення» в папку `C:\Users\Andrii\OneDrive\...` (у деяких мовах — `OneDrive - Personal`).

### ✅ Правильно

Запитайте у Windows, де насправді лежить потрібна папка:

```powershell
[Environment]::GetFolderPath('Desktop')      # робочий стіл
[Environment]::GetFolderPath('MyDocuments')  # документи
```

```text
C:\Users\Andrii\OneDrive\Desktop
```

Скопіюйте показаний шлях і використовуйте його в командах. Що означають квадратні дужки й двокрапки в цій команді, ви дізнаєтеся пізніше; поки що користуйтеся нею як готовим рецептом.

---

### ❌ Шлях із пробілом без лапок

```powershell
Set-Location C:\Program Files
```

```text
Set-Location : A positional parameter cannot be found that accepts argument 'Files'.
```

**Чому:** ви це вже бачили в уроці 4: пробіл розділяє шлях на два значення.

### ✅ Правильно

```powershell
Set-Location "C:\Program Files"
```

## Підсумок

| Команда | Що робить |
|---|---|
| `Get-Location` | Показує поточну папку |
| `Set-Location шлях` (`cd`) | Переходить у папку |
| `Set-Location ..` | На рівень вище |
| `Set-Location ~` | У домашню папку |
| `Get-ChildItem -File` / `-Directory` / `-Force` | Лише файли / лише папки / разом із прихованими |
| `Get-ChildItem *.pdf` | Файли за маскою |
| `Test-Path шлях` | Чи існує (True / False) |
| `Invoke-Item шлях` (`ii`) | Відкрити, як подвійним клацанням |
| `[Environment]::GetFolderPath('Desktop')` | Справжній шлях до робочого столу |

## Перевір себе (~5 хв)

1. Ви в папці `C:\Users\Olena\Documents`. Куди приведе `Set-Location ..`?
2. Чим повний шлях відрізняється від відносного? Наведіть приклад кожного.
3. Чому `cd $HOME\Завантаження` видає помилку?
4. Яка команда покаже в поточній папці лише папки, без файлів?
5. Які файли знайде маска `Фото-?.jpg`: `Фото-1.jpg`, `Фото-12.jpg`, `Фото-.jpg`?

<details>
<summary>Відповіді</summary>

1. У `C:\Users\Olena`.
2. Повний шлях починається з диска і працює з будь-якого місця (`C:\Windows\System32`). Відносний рахується від поточної папки (`Documents`, `..\Downloads`).
3. Справжня назва папки — `Downloads`. «Завантаження» — лише підпис у Провіднику.
4. `Get-ChildItem -Directory`.
5. Лише `Фото-1.jpg`: `?` — рівно один символ.

</details>

## Практика (~10 хв)

Спочатку створимо навчальну папку вручну, щоб побачити зв’язок Провідника і PowerShell.

**Підготовка (вручну):**

1. Відкрийте Провідник і перейдіть у свою домашню папку (`C:\Users\<ваше ім’я>`). Найпростіше: наберіть в адресному рядку `%USERPROFILE%` і натисніть Enter.
2. Клацніть правою кнопкою на порожньому місці → **Створити** → **Папку**.
3. Назвіть папку `PS-Kurs` (латиницею, без пробілів) і натисніть Enter.

**Завдання 1** (легке). Перейдіть у папку `PS-Kurs` з PowerShell і перевірте, що ви там.

<details>
<summary>Розв’язок</summary>

```powershell
Set-Location $HOME\PS-Kurs
Get-Location
```

</details>

**Завдання 2.** Не виходячи з `PS-Kurs`, покажіть усі PDF-файли з папки «Завантаження», використовуючи **відносний** шлях.

<details>
<summary>Підказка</summary>

`PS-Kurs` і `Downloads` лежать в одній папці. Спершу підніміться на рівень вище (`..`), а потім зайдіть у `Downloads`.

</details>

<details>
<summary>Розв’язок</summary>

```powershell
Get-ChildItem ..\Downloads\*.pdf
```

</details>

**Завдання 3** ⭐. Перевірте, чи є на вашому комп’ютері папка `C:\Program Files\Microsoft Office`. Якщо є, відкрийте її у Провіднику командою PowerShell.

<details>
<summary>Розв’язок</summary>

```powershell
Test-Path "C:\Program Files\Microsoft Office"
Invoke-Item "C:\Program Files\Microsoft Office"
```

Якщо `Test-Path` повернув `False`, Office встановлено в інше місце або його немає. Тоді спробуйте `"C:\Program Files (x86)\Microsoft Office"`.

</details>

## Джерела

- [about_Paths](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_paths?view=powershell-5.1)
- [about_Wildcards](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_wildcards?view=powershell-5.1)
- [Set-Location](https://learn.microsoft.com/powershell/module/microsoft.powershell.management/set-location?view=powershell-5.1)
- [Get-ChildItem](https://learn.microsoft.com/powershell/module/microsoft.powershell.management/get-childitem?view=powershell-5.1)
- [Test-Path](https://learn.microsoft.com/powershell/module/microsoft.powershell.management/test-path?view=powershell-5.1)
- [Invoke-Item](https://learn.microsoft.com/powershell/module/microsoft.powershell.management/invoke-item?view=powershell-5.1)
- [Environment.GetFolderPath](https://learn.microsoft.com/dotnet/api/system.environment.getfolderpath)
- [What's new in PowerShell 7 — `cd -`](https://learn.microsoft.com/powershell/scripting/whats-new/differences-from-windows-powershell)

---
[← Модуль 1](../01-znayomstvo/README.md) · [Зміст курсу](../../README.md) · [Урок 6. Створюємо, копіюємо, видаляємо →](06-stvoryuyemo-kopiyuyemo-vydalyayemo.md)
