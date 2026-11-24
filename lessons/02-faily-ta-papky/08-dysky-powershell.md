# Урок 8. Диски PowerShell

⏱ ~30 хв · Рівень: початковий · Потрібно: [урок 7](07-tekstovi-faily.md)

## Що ви вмітимете після уроку

- дізнаватися, скільки вільного місця на диску;
- розуміти, що в PowerShell «диском» може бути не лише `C:`;
- читати змінні середовища: ім’я користувача, ім’я комп’ютера, папку тимчасових файлів;
- безпечно переглядати, які програми запускаються разом з Windows;
- створювати власний «диск»-ярлик до потрібної папки.

## Ситуація з життя

Комп’ютер пише «Недостатньо місця на диску», а ще він став довго завантажуватися. Хочеться швидко подивитися, скільки місця залишилось і які програми стартують разом із Windows. Усе це можна дізнатися з PowerShell, нічого не змінюючи в системі.

## Пояснення (~15 хв)

### 1. Які диски бачить PowerShell: Get-PSDrive

```powershell
Get-PSDrive
```

```text
Name           Used (GB)     Free (GB) Provider      Root
----           ---------     --------- --------      ----
Alias                                  Alias
C                 152,37         85,21 FileSystem    C:\
Cert                                   Certificate   \
D                  12,04        453,70 FileSystem    D:\
Env                                    Environment
Function                               Function
HKCU                                   Registry      HKEY_CURRENT_USER
HKLM                                   Registry      HKEY_LOCAL_MACHINE
Variable                               Variable
WSMan                                  WSMan
```

Крім звичайних дисків `C` і `D`, ви бачите й інші. Для PowerShell «диск» — це будь-яке сховище, по якому можна ходити як по папках: змінні середовища (`Env`), реєстр Windows (`HKCU`, `HKLM`), сертифікати (`Cert`) тощо.

Скільки вільного місця на диску `C`:

```powershell
Get-PSDrive C
```

Колонка `Free (GB)` показує вільне місце в гігабайтах.

> [!NOTE]
> **PowerShell 7:** з’явився ще один зручний диск `Temp:`, який веде до папки тимчасових файлів. У 5.1 його немає, там використовуйте `$env:TEMP` (див. далі).

### 2. Змінні середовища: диск Env:

Windows зберігає корисні відомості у **змінних середовища**: ім’я користувача, ім’я комп’ютера, шляхи до системних папок.

> [!IMPORTANT]
> Назва диска завжди пишеться **з двокрапкою**: `Env:`, `HKCU:`, `C:`. Без двокрапки PowerShell шукатиме звичайну папку з такою назвою в поточній папці.

Переглянути всі змінні:

```powershell
Get-ChildItem Env:
```

Отримати одне значення найзручніше так — зі знаком `$` на початку:

```powershell
$env:USERNAME        # ім’я користувача
$env:COMPUTERNAME    # ім’я комп’ютера
$env:TEMP            # папка тимчасових файлів
```

```text
Andrii
DESKTOP-7K2QF1M
C:\Users\Andrii\AppData\Local\Temp
```

Відкрити папку тимчасових файлів у Провіднику:

```powershell
ii $env:TEMP
```

> [!NOTE]
> Змінну можна змінити (`$env:MYVAR = "123"`), але зміна діє лише в цьому вікні PowerShell. Після закриття вікна все повертається як було.

### 3. Реєстр: лише дивимось

**Реєстр** — це велика база налаштувань Windows і програм. PowerShell показує його як два диски: `HKCU:` (налаштування поточного користувача) і `HKLM:` (налаштування всього комп’ютера).

> [!WARNING]
> Неправильна зміна реєстру може зламати програму або навіть Windows. У цьому курсі ми реєстр **лише читаємо**. Не запускайте з дисками `HKCU:` і `HKLM:` команди `Set-…`, `New-…`, `Remove-…`, `Rename-…`, доки точно не знаєте, що робите.

Подивимось, які програми запускаються разом з Windows для вашого користувача:

```powershell
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
```

```text
OneDrive     : "C:\Program Files\Microsoft OneDrive\OneDrive.exe" /background
Telegram     : "C:\Users\Andrii\AppData\Roaming\Telegram Desktop\Telegram.exe" -autostart
PSPath       : Microsoft.PowerShell.Core\Registry::HKEY_CURRENT_USER\Software\Microsoft\...
PSParentPath : ...
PSChildName  : Run
PSDrive      : HKCU
PSProvider   : Microsoft.PowerShell.Core\Registry
```

Перші рядки — назви програм і шляхи до них. Рядки, що починаються з `PS`, PowerShell додає від себе, їх можна не читати.

> [!TIP]
> Вимкнути зайву програму з автозапуску безпечніше не через реєстр, а вручну: **Ctrl + Shift + Esc** (Диспетчер завдань) → вкладка **Автозавантаження** (у Windows 11 — **Програми автозавантаження**) → клацніть програму правою кнопкою → **Вимкнути**.

### 4. Власний диск-ярлик: New-PSDrive

Якщо ви часто працюєте з глибоко вкладеною папкою, можна створити для неї коротку назву:

```powershell
New-PSDrive -Name Kurs -PSProvider FileSystem -Root "$HOME\PS-Kurs"
Set-Location Kurs:
Get-ChildItem
```

Запрошення тепер виглядає як `PS Kurs:\>`.

> [!NOTE]
> Такий диск існує лише у поточному вікні PowerShell і видно його тільки в PowerShell (у Провіднику він не з’являється). Після закриття вікна він зникає, а файли в папці, звісно, залишаються.

## Типові помилки

### ❌ Забули двокрапку після назви диска

```powershell
Set-Location HKCU
```

```text
Set-Location : Cannot find path 'C:\Users\Andrii\HKCU' because it does not exist.
```

**Чому:** без двокрапки `HKCU` — це просто назва папки, і PowerShell шукає її в поточній папці.

### ✅ Правильно

```powershell
Set-Location HKCU:
Set-Location C:      # повернутися на диск C
```

---

### ❌ Звертання до змінної без знака `$`

```powershell
env:USERNAME
```

```text
env:USERNAME : The term 'env:USERNAME' is not recognized as the name of a cmdlet, function,
script file, or operable program.
```

**Чому:** без `$` PowerShell вважає написане назвою команди і шукає таку команду.

### ✅ Правильно

```powershell
$env:USERNAME
```

---

### ❌ Змінили змінну і чекаєте, що зміна збережеться

```powershell
$env:MYNAME = "Андрій"
```

Ви закрили PowerShell, відкрили знову, а `$env:MYNAME` порожня.

**Чому:** зміни через `$env:` діють лише в поточному вікні. Це навіть зручно: так неможливо випадково зіпсувати налаштування Windows.

### ✅ Правильно

Для разових експериментів так і має бути. Постійні змінні середовища налаштовують вручну: **Пуск** → введіть «змінні середовища» → **Редагувати змінні середовища для свого облікового запису**.

---

### ❌ Шукаємо створений диск у Провіднику

Після `New-PSDrive -Name Kurs ...` ви відкрили «Цей ПК», а диска `Kurs` там немає.

**Чому:** диски, створені `New-PSDrive`, існують лише всередині PowerShell.

### ✅ Правильно

Для швидкого доступу у Провіднику просто закріпіть папку: клацніть її правою кнопкою → **Закріпити на панелі швидкого доступу**.

## Підсумок

| Команда | Що робить |
|---|---|
| `Get-PSDrive` | Показує всі диски PowerShell |
| `Get-PSDrive C` | Зайняте й вільне місце на диску C |
| `Get-ChildItem Env:` | Усі змінні середовища |
| `$env:USERNAME`, `$env:TEMP` | Значення однієї змінної |
| `Get-ItemProperty -Path "HKCU:\..."` | Читає значення з реєстру |
| `New-PSDrive -Name N -PSProvider FileSystem -Root шлях` | Тимчасовий диск-ярлик до папки |
| `Set-Location Назва:` | Перехід на диск (з двокрапкою!) |

## Перевір себе (~5 хв)

1. Яка команда покаже вільне місце на диску `D`?
2. Чому `Set-Location Env` дає помилку, а `Set-Location Env:` ні?
3. Як вивести ім’я комп’ютера?
4. Що станеться з диском, створеним `New-PSDrive`, після закриття PowerShell?
5. Чому в курсі ми лише читаємо реєстр, а не змінюємо його?

<details>
<summary>Відповіді</summary>

1. `Get-PSDrive D`.
2. Без двокрапки `Env` сприймається як звичайна папка в поточній папці, а такої немає.
3. `$env:COMPUTERNAME`.
4. Він зникне, а сама папка й файли залишаться.
5. Неправильна зміна реєстру може зламати програми або Windows, а скасувати її не завжди просто.

</details>

## Практика (~10 хв)

**Завдання 1** (легке). Дізнайтеся, скільки вільного місця на диску `C`.

<details>
<summary>Розв’язок</summary>

```powershell
Get-PSDrive C
```

Дивіться колонку `Free (GB)`.

</details>

**Завдання 2.** Виведіть ім’я користувача та ім’я комп’ютера, а потім відкрийте у Провіднику папку тимчасових файлів і подивіться, скільки місця вона займає (клацніть правою кнопкою на будь-якому порожньому місці → **Властивості**).

<details>
<summary>Розв’язок</summary>

```powershell
$env:USERNAME
$env:COMPUTERNAME
ii $env:TEMP
```

</details>

**Завдання 3** ⭐. Створіть диск `Kurs:` для папки `PS-Kurs`, перейдіть на нього і виведіть лише текстові файли з уроку 7. Потім, не змінюючи нічого, подивіться список програм в автозапуску вашого користувача.

<details>
<summary>Розв’язок</summary>

```powershell
New-PSDrive -Name Kurs -PSProvider FileSystem -Root "$HOME\PS-Kurs"
Set-Location Kurs:
Get-ChildItem *.txt
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
```

Якщо `New-PSDrive` повідомляє, що диск `Kurs` вже існує, ви створили його раніше в цьому вікні: просто переходьте до другої команди.

</details>

## Джерела

- [about_Providers](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_providers?view=powershell-5.1)
- [Managing PowerShell drives](https://learn.microsoft.com/powershell/scripting/samples/managing-windows-powershell-drives)
- [Get-PSDrive](https://learn.microsoft.com/powershell/module/microsoft.powershell.management/get-psdrive?view=powershell-5.1)
- [New-PSDrive](https://learn.microsoft.com/powershell/module/microsoft.powershell.management/new-psdrive?view=powershell-5.1)
- [about_Environment_Variables](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_environment_variables?view=powershell-5.1)
- [Working with registry entries](https://learn.microsoft.com/powershell/scripting/samples/working-with-registry-entries)
- [about_FileSystem_Provider — диск Temp: у PowerShell 7](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_filesystem_provider?view=powershell-7.5)

---
[← Урок 7](07-tekstovi-faily.md) · [Зміст курсу](../../README.md) · [Тест модуля 2 →](README.md#підсумковий-тест-модуля-2)
