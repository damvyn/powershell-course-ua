# Урок 3. Як знайти потрібну команду

⏱ ~30 хв · Рівень: початковий · Потрібно: [урок 2](02-pershi-komandy.md)

## Що ви вмітимете після уроку

- знаходити команду за ключовим словом за допомогою `Get-Command`;
- читати довідку до команди і знаходити в ній готові приклади;
- завантажувати й оновлювати довідку командою `Update-Help`;
- відкривати довідку в браузері та в окремому вікні.

## Ситуація з життя

Ви хочете перезапустити комп’ютер через PowerShell, але не знаєте, як називається команда. Можна шукати в інтернеті й натрапити на застарілу пораду. А можна запитати сам PowerShell: він має вбудований пошук команд і довідку з прикладами.

Навіть досвідчені фахівці не пам’ятають тисячі команд. Їхня перевага в тому, що вони вміють швидко знайти потрібну. Саме цьому навчимося сьогодні.

## Пояснення (~15 хв)

### 1. Шукаємо команду: Get-Command

`Get-Command` показує список команд. Найкорисніше — шукати за частиною назви. Для цього використовують **зірочку** `*`, яка означає «будь-які символи».

**Спробуйте самі:** знайдемо всі команди, у назві яких є слово `Computer`:

```powershell
Get-Command *Computer*
```

Серед результатів будуть такі рядки:

```text
CommandType     Name                          Version    Source
-----------     ----                          -------    ------
Cmdlet          Restart-Computer              3.1.0.0    Microsoft.PowerShell.Management
Cmdlet          Stop-Computer                 3.1.0.0    Microsoft.PowerShell.Management
Cmdlet          Rename-Computer               3.1.0.0    Microsoft.PowerShell.Management
...
```

`Restart-Computer` — ось і команда для перезапуску.

> [!WARNING]
> Не запускайте зараз `Restart-Computer` чи `Stop-Computer`: комп’ютер одразу перезавантажиться або вимкнеться, і незбережені документи пропадуть.

Пам’ятаєте правило «Дієслово-Іменник» з уроку 2? `Get-Command` вміє шукати окремо за дієсловом або іменником:

```powershell
Get-Command -Noun Process     # усі команди для роботи з процесами
Get-Command -Verb Stop        # усі команди, що щось зупиняють
```

```text
CommandType     Name              Version    Source
-----------     ----              -------    ------
Cmdlet          Debug-Process     3.1.0.0    Microsoft.PowerShell.Management
Cmdlet          Get-Process       3.1.0.0    Microsoft.PowerShell.Management
Cmdlet          Start-Process     3.1.0.0    Microsoft.PowerShell.Management
Cmdlet          Stop-Process      3.1.0.0    Microsoft.PowerShell.Management
Cmdlet          Wait-Process      3.1.0.0    Microsoft.PowerShell.Management
```

> [!TIP]
> Шукайте ширше, а не вужче. `*Computer*` знайде більше, ніж `*Computers*`, бо назви команд завжди в однині.

### 2. Читаємо довідку: Get-Help

Знайшли команду? Тепер дізнаймося, як нею користуватися:

```powershell
Get-Help Restart-Computer
```

Довідка складається з розділів:

| Розділ | Що в ньому |
|---|---|
| **NAME** | Назва команди |
| **SYNOPSIS** | Одне речення про те, що робить команда |
| **SYNTAX** | Як записувати команду і які в неї є параметри |
| **DESCRIPTION** | Докладний опис |
| **RELATED LINKS** | Пов’язані команди і посилання на сайт |
| **REMARKS** | Підказки, як отримати ще більше інформації |

Найкорисніше для новачка — **готові приклади**:

```powershell
Get-Help Restart-Computer -Examples
```

Інші корисні варіанти:

| Команда | Що покаже |
|---|---|
| `Get-Help Get-Process -Examples` | Лише приклади |
| `Get-Help Get-Process -Full` | Усю довідку, включно з описом кожного параметра |
| `Get-Help Get-Process -Online` | Найсвіжішу версію довідки на сайті Microsoft Learn у браузері |
| `Get-Help Get-Process -ShowWindow` | Довідку в окремому вікні з пошуком |

Замість `Get-Help` можна писати коротше `help`. Різниця в тому, що `help` показує довідку **посторінково**: **Пробіл** гортає далі, **Q** закриває перегляд.

```powershell
help Get-Process -Full
```

> [!NOTE]
> Довідка PowerShell написана англійською, перекладу українською немає. Якщо англійська складна, відкрийте її з параметром `-Online` і скористайтеся вбудованим перекладачем браузера.

### 3. Завантажуємо довідку: Update-Help

Спочатку на комп’ютері немає повної довідки, щоб не займати місце. Без неї `Get-Help` покаже лише скорочений варіант і таке повідомлення:

```text
REMARKS
    Get-Help cannot find the Help files for this cmdlet on this computer. It is displaying
    only partial help.
        -- To download and install Help files for the module that includes this cmdlet,
           use Update-Help.
```

Під час першого запуску `Get-Help` PowerShell може сам запропонувати завантажити довідку:

```text
Do you want to run Update-Help?
...
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):
```

Але у Windows PowerShell 5.1 завантаження спрацює лише у вікні адміністратора. Тому надійніше зробити так:

1. Закрийте всі вікна PowerShell.
2. Відкрийте PowerShell **від імені адміністратора** (урок 1).
3. Введіть команду:

```powershell
Update-Help -UICulture en-US
```

4. Зачекайте 1–3 хвилини, доки зникне смужка прогресу.
5. Закрийте вікно адміністратора командою `exit`.

Параметр `-UICulture en-US` просить завантажити саме англійську довідку. Без нього на українській Windows PowerShell шукатиме українську, якої не існує (див. «Типові помилки»).

> [!TIP]
> Під час оновлення можуть з’явитися червоні повідомлення про окремі модулі (наприклад, `Failed to update Help for the module(s) 'BitsTransfer'`). Це нормально: автори деяких модулів не опублікували довідку. Основні команди все одно оновляться.

> [!NOTE]
> **PowerShell 7:** `Update-Help` працює **без** прав адміністратора і зберігає довідку у вашу особисту папку. Довідки для 5.1 і 7 зберігаються окремо, тому оновлювати їх треба в кожній версії.

### 4. Довідкові статті про поняття: about_

Окрім довідки до команд, є статті про загальні поняття. Їхні назви починаються з `about_`:

```powershell
Get-Help about_*                  # список усіх статей
help about_Command_Syntax         # як читати розділ SYNTAX
help about_Execution_Policies     # що таке політика виконання скриптів (знадобиться в модулі 6)
```

Назви цих статей завжди вводяться англійською.

## Типові помилки

### ❌ Update-Help у звичайному вікні Windows PowerShell 5.1

```powershell
Update-Help
```

```text
Update-Help : Failed to update Help for the module(s) 'Microsoft.PowerShell.Management, ...'
with UI culture(s) {en-US} : Access is denied. The command could not update Help topics for the
Windows PowerShell core modules, or for any modules in the $pshome\Modules directory. To update
these Help topics, start Windows PowerShell by using the "Run as Administrator" command, and try
the command again.
```

**Чому:** у 5.1 довідка основних команд зберігається в системній папці Windows, а писати туди може лише адміністратор. Повідомлення саме про це й каже: «Access is denied» (доступ заборонено).

### ✅ Правильно

Відкрийте PowerShell **від імені адміністратора** і повторіть команду. Або, якщо прав адміністратора немає, читайте довідку онлайн: `Get-Help Get-Process -Online`.

---

### ❌ Update-Help на українській Windows без вибору мови

```text
Update-Help : Failed to update Help for the module(s) '...' with UI culture(s) {uk-UA} :
Unable to retrieve the HelpInfo XML file for UI culture uk-UA. Make sure the HelpInfoUri
property in the module manifest is valid or check your network connection and then try the
command again.
```

**Чому:** PowerShell шукає довідку мовою вашої Windows (`uk-UA`), а Microsoft публікує її англійською.

### ✅ Правильно

```powershell
Update-Help -UICulture en-US
```

---

### ❌ Шукаємо команду українською

```powershell
Get-Command *перезавантаж*
```

Результат порожній, PowerShell нічого не знайшов.

**Чому:** назви всіх команд PowerShell англійські.

### ✅ Правильно

Шукайте англійським словом: перезавантажити — `Restart`, комп’ютер — `Computer`, файл — `File` або `Item`, служба — `Service`.

```powershell
Get-Command *Restart*
```

---

### ❌ Пошук з дефісом на початку

```powershell
help -Process
```

```text
help : A parameter cannot be found that matches parameter name 'Process'.
```

**Чому:** усе, що починається з дефіса, PowerShell сприймає як назву **параметра** (про параметри — в уроці 4), а в `Get-Help` немає параметра `-Process`.

### ✅ Правильно

```powershell
help *-Process
```

## Підсумок

| Команда | Що робить |
|---|---|
| `Get-Command *слово*` | Шукає команди, в назві яких є слово |
| `Get-Command -Noun Process` | Шукає команди з потрібним іменником |
| `Get-Command -Verb Stop` | Шукає команди з потрібним дієсловом |
| `Get-Help команда` | Коротка довідка |
| `Get-Help команда -Examples` | Лише приклади |
| `Get-Help команда -Full` | Повна довідка |
| `Get-Help команда -Online` | Довідка в браузері |
| `help команда` | Довідка посторінково (Пробіл, Q) |
| `Update-Help -UICulture en-US` | Завантажити довідку (у 5.1 — від імені адміністратора) |
| `Get-Help about_*` | Статті про поняття |

## Перевір себе (~5 хв)

1. Що означає символ `*` у команді `Get-Command *Service*`?
2. Як побачити лише приклади використання команди `Get-Date`?
3. Чому `Update-Help` у звичайному вікні Windows PowerShell 5.1 повідомляє «Access is denied»?
4. Чим відрізняються `Get-Help` і `help`?
5. Ви не маєте прав адміністратора. Як прочитати найсвіжішу довідку до `Get-Process`?

<details>
<summary>Відповіді</summary>

1. «Будь-які символи». Буде знайдено всі команди, в назві яких є слово `Service` у будь-якому місці.
2. `Get-Help Get-Date -Examples`.
3. У 5.1 довідка основних команд зберігається в системній папці, куди може писати лише адміністратор.
4. Показують ту саму довідку, але `help` виводить її посторінково (Пробіл — далі, Q — вихід).
5. `Get-Help Get-Process -Online`: довідка відкриється в браузері на сайті Microsoft Learn.

</details>

## Практика (~10 хв)

**Завдання 1** (легке). Знайдіть усі команди, що працюють зі службами Windows (іменник `Service`). Скільки серед них команд з дієсловом `Get`?

<details>
<summary>Розв’язок</summary>

```powershell
Get-Command -Noun Service
```

З дієсловом `Get` лише одна: `Get-Service`.

</details>

**Завдання 2.** Завантажте довідку на свій комп’ютер, а потім подивіться приклади до команди `Get-Process`.

<details>
<summary>Підказка</summary>

Пам’ятайте: у Windows PowerShell 5.1 для `Update-Help` потрібне вікно адміністратора.

</details>

<details>
<summary>Розв’язок</summary>

У вікні **адміністратора**:

```powershell
Update-Help -UICulture en-US
exit
```

У звичайному вікні:

```powershell
Get-Help Get-Process -Examples
```

</details>

**Завдання 3** ⭐. Вам потрібно стиснути папку в ZIP-архів. Знайдіть потрібну команду (підказка: «архів» англійською — `Archive`) і відкрийте довідку до неї в браузері.

<details>
<summary>Розв’язок</summary>

```powershell
Get-Command *Archive*
```

Серед результатів будуть `Compress-Archive` (стиснути) і `Expand-Archive` (розпакувати). Довідка в браузері:

```powershell
Get-Help Compress-Archive -Online
```

</details>

## Джерела

- [Chapter 2 — The Help system](https://learn.microsoft.com/powershell/scripting/learn/ps101/02-help-system)
- [Get-Command](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/get-command?view=powershell-5.1)
- [Get-Help](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/get-help?view=powershell-5.1)
- [Update-Help](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/update-help?view=powershell-5.1)
- [about_Updatable_Help](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_updatable_help)

---
[← Урок 2](02-pershi-komandy.md) · [Зміст курсу](../../README.md) · [Урок 4. Параметри та псевдоніми →](04-parametry-ta-psevdonimy.md)
