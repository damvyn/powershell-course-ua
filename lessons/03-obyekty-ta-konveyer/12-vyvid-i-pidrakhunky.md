# Урок 12. Гарний вивід і підрахунки

⏱ ~30 хв · Рівень: початковий · Потрібно: [урок 11](11-filtruyemo-dani.md)

## Що ви вмітимете після уроку

- показувати результат таблицею або списком (`Format-Table`, `Format-List`);
- переглядати й фільтрувати дані у вікні з таблицею (`Out-GridView`);
- рахувати кількість, суму, середнє, найбільше й найменше (`Measure-Object`);
- групувати однакові об’єкти (`Group-Object`);
- показувати розмір у мегабайтах замість байтів.

## Ситуація з життя

Ви хочете знати, скільки місця займають фото в «Зображеннях», яких файлів у «Завантаженнях» найбільше (PDF? відео? інсталятори?) і бачити розміри в зрозумілих мегабайтах, а не в мільйонах байтів. Провідник такої зведеної інформації не дає.

## Пояснення (~15 хв)

### 1. Таблиця чи список: Format-Table, Format-List

PowerShell сам вирішує, як показати результат: таблицею, якщо властивостей мало, або списком, якщо багато. Але ви можете вибрати самі.

**Таблиця з потрібними стовпцями.** П’ять найбільших файлів у «Завантаженнях»: назва, розмір і повний шлях:

```powershell
Get-ChildItem $HOME\Downloads -File | Sort-Object Length -Descending | Select-Object -First 5 | Format-Table Name, Length, FullName -AutoSize
```

`-AutoSize` підганяє ширину стовпців під вміст, щоб текст не обрізався трикрапкою `...`.

**Список: кожна властивість з нового рядка.** Зручно, коли об’єкт один, а властивостей багато. Усе про найбільший файл у «Завантаженнях»:

```powershell
Get-ChildItem $HOME\Downloads -File | Sort-Object Length -Descending | Select-Object -First 1 | Format-List *
```

Зірочка `*` означає «усі властивості». Так видно навіть ті, що не показуються за замовчуванням.

> [!IMPORTANT]
> Команди `Format-*` перетворюють об’єкти на «картинку» для екрана. Після них об’єктів уже немає, тому **`Format-Table` і `Format-List` ставте лише в самому кінці конвеєра**. Після них не можна сортувати, фільтрувати чи зберігати в CSV (див. «Типові помилки»).

### 2. Вікно з таблицею: Out-GridView

```powershell
Get-ChildItem $HOME\Downloads -File | Out-GridView
```

Відкриється окреме вікно з таблицею. У ньому можна:

- клацнути заголовок стовпця, щоб відсортувати;
- ввести текст у поле **Filter** угорі, щоб швидко знайти рядок;
- натиснути **Add criteria**, щоб додати умову фільтра мишкою.

Закрийте вікно хрестиком, коли закінчите.

> [!NOTE]
> **PowerShell 7:** `Out-GridView` працює лише у Windows. Його прибрали в PowerShell 6 і повернули у версії 7.0, тож у старих статтях можна прочитати, що його немає.

### 3. Підрахунки: Measure-Object

Скільки файлів у «Завантаженнях»:

```powershell
Get-ChildItem $HOME\Downloads -File | Measure-Object
```

```text
Count    : 214
Average  :
Sum      :
Maximum  :
Minimum  :
Property :
```

Скільки вони займають разом, а також найбільший і середній розмір:

```powershell
Get-ChildItem $HOME\Downloads -File | Measure-Object -Property Length -Sum -Average -Maximum
```

```text
Count    : 214
Average  : 38772149,6
Sum      : 8297240014
Maximum  : 6123847680
Minimum  :
Property : Length
```

> [!TIP]
> Щоб отримати суму одразу в мегабайтах або гігабайтах, візьміть результат у дужки і поділіть на `1MB` чи `1GB`:
> ```powershell
> (Get-ChildItem $HOME\Downloads -File | Measure-Object -Property Length -Sum).Sum / 1GB
> ```
> ```text
> 7,72744178771973
> ```
> Тобто майже 7,7 ГБ.

### 4. Групування: Group-Object

Яких файлів у «Завантаженнях» найбільше:

```powershell
Get-ChildItem $HOME\Downloads -File | Group-Object Extension | Sort-Object Count -Descending
```

```text
Count Name                      Group
----- ----                      -----
   87 .pdf                      {рахунок-газ.pdf, договір.pdf, ...}
   41 .jpg                      {IMG_2031.jpg, IMG_2032.jpg, ...}
   23 .exe                      {ChromeSetup.exe, Setup_Office.exe, ...}
...
```

`Count` — скільки файлів у групі, `Name` — за чим згруповано (тут розширення), `Group` — самі файли.

> [!NOTE]
> **PowerShell 7:** `Group-Object` одразу сортує групи за назвою (`Name`). У 5.1 групи йдуть у тому порядку, у якому вперше трапилися. Якщо потрібен певний порядок, у будь-якій версії додавайте `Sort-Object`, як у прикладі вище.

### 5. Розмір у мегабайтах: обчислюваний стовпець

`Length` у байтах читати незручно. `Select-Object` уміє створити **новий стовпець**, значення якого обчислюється для кожного об’єкта:

```powershell
Get-ChildItem $HOME\Downloads -File |
    Sort-Object Length -Descending |
    Select-Object -First 5 Name, @{ Name = "Розмір (МБ)"; Expression = { [math]::Round($_.Length / 1MB, 1) } }
```

```text
Name                         Розмір (МБ)
----                         -----------
Win11_24H2_Ukrainian_x64.iso      5840,1
відео-з-весілля.mp4               1757,8
Setup_Office.exe                   393,7
...
```

Як читати запис `@{ ... }`:

- `Name = "Розмір (МБ)"` — заголовок нового стовпця;
- `Expression = { ... }` — як обчислити значення. `$_` — поточний файл, як у `Where-Object`;
- `[math]::Round(число, 1)` — округлити до одного знака після коми.

> [!TIP]
> Цей запис довгий, але його не треба запам’ятовувати. Скопіюйте його собі в нотатки й підставляйте потрібну властивість і дільник (`1KB`, `1MB`, `1GB`).

## Типові помилки

### ❌ Format-Table посеред конвеєра

```powershell
Get-ChildItem $HOME\Downloads -File | Format-Table Name, Length | Export-Csv $HOME\PS-Kurs\завантаження.csv
```

Відкрийте файл (`notepad $HOME\PS-Kurs\завантаження.csv`): замість назв файлів там рядки на кшталт рядки на кшталт `"ClassId2e4f51ef21dd47e99d3c952918aff9cd","pageHeaderEntry",...`.

**Чому:** `Format-Table` повертає не файли, а службові об’єкти для малювання таблиці на екрані. Наступні команди отримують саме їх.

### ✅ Правильно

Для вибору стовпців посеред конвеєра використовуйте `Select-Object`, а `Format-Table` ставте лише в кінець:

```powershell
Get-ChildItem $HOME\Downloads -File | Select-Object Name, Length | Export-Csv $HOME\PS-Kurs\завантаження.csv
Get-ChildItem $HOME\Downloads -File | Format-Table Name, Length -AutoSize
```

---

### ❌ Сума без `-Property`

```powershell
Get-ChildItem $HOME\Downloads -File | Measure-Object -Sum
```

```text
Measure-Object : Input object "договір.pdf" is not numeric.
```

**Чому:** без `-Property` команда намагається додати самі файли, а не їх розміри. Файл — не число.

### ✅ Правильно

```powershell
Get-ChildItem $HOME\Downloads -File | Measure-Object -Property Length -Sum
```

---

### ❌ Обрізані стовпці

```powershell
Get-ChildItem $HOME\Downloads -File | Select-Object FullName, Length, LastWriteTime
```

Довгі шляхи в стовпці `FullName` закінчуються трикрапкою `...`, або PowerShell показує результат списком замість таблиці.

**Чому:** вікну бракує ширини, щоб показати всі стовпці повністю.

### ✅ Правильно

Залиште менше стовпців, додайте `-AutoSize` або подивіться результат у вікні:

```powershell
Get-ChildItem $HOME\Downloads -File | Format-Table FullName, Length -AutoSize
Get-ChildItem $HOME\Downloads -File | Select-Object FullName, Length, LastWriteTime | Out-GridView
```

## Підсумок

| Команда | Що робить |
|---|---|
| `Format-Table Влас1, Влас2 -AutoSize` | Таблиця (лише в кінці конвеєра) |
| `Format-List *` | Усі властивості списком |
| `Out-GridView` | Вікно з таблицею, сортуванням і фільтром |
| `Measure-Object` | Кількість об’єктів |
| `Measure-Object -Property Length -Sum -Average -Maximum` | Сума, середнє, максимум |
| `Group-Object Властивість` | Групує однакові значення |
| `@{ Name = "..."; Expression = { ... } }` | Обчислюваний стовпець у `Select-Object` |

## Перевір себе (~5 хв)

1. Чому `Format-Table` не можна ставити перед `Export-Csv`?
2. Яка команда покаже **всі** властивості одного файлу?
3. Як дізнатися, скільки процесів зараз запущено?
4. Що покаже стовпець `Count` після `Group-Object Extension`?
5. Навіщо ділити `Length` на `1MB`?

<details>
<summary>Відповіді</summary>

1. `Format-Table` перетворює об’єкти на службові об’єкти для екрана, і в CSV потраплять вони, а не ваші дані.
2. `Get-Item шлях | Format-List *`.
3. `Get-Process | Measure-Object` (стовпець `Count`).
4. Скільки файлів з кожним розширенням.
5. Щоб отримати розмір у мегабайтах замість байтів.

</details>

## Практика (~10 хв)

**Завдання 1** (легке). Порахуйте, скільки служб Windows зараз запущено.

<details>
<summary>Розв’язок</summary>

```powershell
Get-Service | Where-Object Status -eq Running | Measure-Object
```

</details>

**Завдання 2.** Дізнайтеся, скільки гігабайтів займають усі фото `.jpg` у папці «Зображення» разом із вкладеними папками.

<details>
<summary>Розв’язок</summary>

```powershell
(Get-ChildItem $HOME\Pictures -Filter *.jpg -Recurse | Measure-Object -Property Length -Sum).Sum / 1GB
```

</details>

**Завдання 3** ⭐. Згрупуйте файли з «Завантажень» за розширенням і для кожної групи покажіть розширення, кількість файлів і загальний розмір у МБ. Відкрийте результат в `Out-GridView`.

<details>
<summary>Підказка</summary>

У кожної групи є властивість `Group` — це файли групи. Їх можна передати в `Measure-Object` всередині обчислюваного стовпця: `($_.Group | Measure-Object Length -Sum).Sum`.

</details>

<details>
<summary>Розв’язок</summary>

```powershell
Get-ChildItem $HOME\Downloads -File |
    Group-Object Extension |
    Select-Object Name, Count, @{ Name = "Разом (МБ)"; Expression = { [math]::Round(($_.Group | Measure-Object Length -Sum).Sum / 1MB, 1) } } |
    Out-GridView
```

У вікні клацніть заголовок «Разом (МБ)», щоб побачити, які файли займають найбільше місця.

</details>

## Джерела

- [Using Format commands to change output view](https://learn.microsoft.com/powershell/scripting/samples/using-format-commands-to-change-output-view)
- [Format-Table](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/format-table?view=powershell-5.1)
- [Format-List](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/format-list?view=powershell-5.1)
- [Out-GridView](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/out-gridview?view=powershell-5.1)
- [Measure-Object](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/measure-object?view=powershell-5.1)
- [Group-Object](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/group-object?view=powershell-5.1)
- [Select-Object — обчислювані властивості](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_calculated_properties?view=powershell-5.1)
- [Export-Csv — приклад із Format-Table у конвеєрі](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/export-csv?view=powershell-5.1)
- [Differences between Windows PowerShell 5.1 and PowerShell 7.x](https://learn.microsoft.com/powershell/scripting/whats-new/differences-from-windows-powershell)

---
[← Урок 11](11-filtruyemo-dani.md) · [Зміст курсу](../../README.md) · [Тест модуля 3 →](README.md#підсумковий-тест-модуля-3)
