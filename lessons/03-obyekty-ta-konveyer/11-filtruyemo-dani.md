# Урок 11. Фільтруємо дані

⏱ ~30 хв · Рівень: початковий · Потрібно: [урок 10](10-konveyer.md)

## Що ви вмітимете після уроку

- залишати в конвеєрі лише ті об’єкти, що відповідають умові (`Where-Object`);
- порівнювати числа, текст і дати операторами `-eq`, `-gt`, `-lt`, `-like`, `-match`;
- поєднувати кілька умов через `-and`, `-or`, `-not`;
- знаходити великі й давні файли, щоб звільнити місце.

## Ситуація з життя

У «Завантаженнях» роками збираються інсталятори й архіви. Хочеться знайти файли, **більші за 100 МБ**, яких ви **не відкривали понад пів року**. Сортування з уроку 10 тут не допоможе: потрібен фільтр за двома умовами одночасно.

## Пояснення (~15 хв)

### 1. Where-Object і `$_`

Почнемо з першої умови нашого завдання: файли, більші за 100 МБ.

> [!IMPORTANT]
> У PowerShell для порівняння **не використовують** звичні знаки `=`, `>` і `<`. Замість них пишуть оператори з дефісом: `-eq`, `-gt`, `-lt`. Знак `=` означає «присвоїти значення», а `>` — «записати у файл» (урок 7). Що буде, якщо їх переплутати, дивіться в «Типових помилках».

> [!TIP]
> PowerShell розуміє скорочення розмірів: `1KB`, `100MB`, `2GB`. Пишіть їх **без пробілу**. `100MB` — це 104 857 600 байтів, рахувати самому не треба.

```powershell
Get-ChildItem $HOME\Downloads -File | Where-Object { $_.Length -gt 100MB }
```

Що тут відбувається:

1. `Get-ChildItem` знаходить усі файли в «Завантаженнях». Кожен файл — це окремий об’єкт з властивостями `Name`, `Length`, `LastWriteTime` тощо (урок 9).
2. Конвеєр передає ці об’єкти в `Where-Object` **не всі разом, а по одному**, як деталі на конвеєрі.
3. Кожен файл, що надійшов, `Where-Object` кладе в спеціальну змінну `$_`. Тому `$_` щоразу означає **той файл, який перевіряється саме зараз**.
4. Для цього файлу `Where-Object` виконує умову у фігурних дужках. Якщо вона правдива (`True`), файл іде далі конвеєром. Якщо хибна (`False`), файл відкидається.
5. Потім надходить наступний файл, `$_` тепер означає вже його, і все повторюється, доки файли не закінчаться.

Наприклад, у «Завантаженнях» три файли:

| Файл, що надійшов у `$_` | `$_.Length` | `$_.Length -gt 100MB` | Результат |
|---|---|---|---|
| `договір.pdf` | 245 760 (240 КБ) | `False` | відкинуто |
| `Setup_Office.exe` | 412 876 544 (394 МБ) | `True` | пропущено далі |
| `фото.jpg` | 3 145 728 (3 МБ) | `False` | відкинуто |

На екран потрапить лише `Setup_Office.exe`.

Як читати запис:

- `{ ... }` — фігурні дужки, у яких записана умова;
- `$_` — поточний об’єкт (тут — файл), який `Where-Object` отримав від попередньої команди. `$_.Length` — його розмір;
- `-gt` — оператор «більше» (**g**reater **t**han);
- `100MB` — 100 мегабайтів.

Разом: «пропусти файл, якщо його розмір більший за 100 МБ».

### 2. Оператори порівняння

| Оператор | Означає | Англійською | Приклад |
|---|---|---|---|
| `-eq` | дорівнює | **eq**ual | `$_.Extension -eq ".pdf"` |
| `-ne` | не дорівнює | **n**ot **e**qual | `$_.Extension -ne ".tmp"` |
| `-gt` | більше | **g**reater **t**han | `$_.Length -gt 100MB` |
| `-ge` | більше або дорівнює | **g**reater or **e**qual | `$_.Length -ge 1GB` |
| `-lt` | менше | **l**ess **t**han | `$_.LastWriteTime -lt (Get-Date).AddMonths(-6)` |
| `-le` | менше або дорівнює | **l**ess or **e**qual | `$_.Length -le 1KB` |
| `-like` | схоже на маску з `*` і `?` | like | `$_.Name -like "*звіт*"` |
| `-match` | відповідає шаблону | match | `$_.Name -match "2026"` |

Порівняння тексту за замовчуванням **не розрізняє** великі й малі літери: `"PDF" -eq "pdf"` дає `True`.

Тепер друга умова нашого завдання: файли, які не змінювали понад пів року. `(Get-Date).AddMonths(-6)` — це дата пів року тому (урок 9), а `-lt` («менше») для дат означає «раніше»:

```powershell
Get-ChildItem $HOME\Downloads -File | Where-Object { $_.LastWriteTime -lt (Get-Date).AddMonths(-6) }
```

> [!NOTE]
> Дата останнього **відкриття** файлу (`LastAccessTime`) у Windows часто не оновлюється, щоб не сповільнювати диск, тому ми перевіряємо дату останньої **зміни** (`LastWriteTime`). Для інсталяторів і архівів у «Завантаженнях» це майже завжди дата, коли файл завантажили.

**Спробуйте самі:** знайдіть у «Завантаженнях» файли, у назві яких є слово «рахунок»:

```powershell
Get-ChildItem $HOME\Downloads -File | Where-Object { $_.Name -like "*рахунок*" }
```

### 3. Кілька умов: -and, -or, -not

| Оператор | Означає |
|---|---|
| `-and` | обидві умови правдиві |
| `-or` | хоча б одна умова правдива |
| `-not` | навпаки: правдиве стає хибним |

Тепер з’єднаємо обидві умови із завдання на початку уроку — файли великі **і** давні:

```powershell
Get-ChildItem $HOME\Downloads -File |
    Where-Object { $_.Length -gt 100MB -and $_.LastWriteTime -lt (Get-Date).AddMonths(-6) } |
    Select-Object Name, Length, LastWriteTime
```

А щоб побачити серед «Завантажень» лише інсталятори **або** архіви:

```powershell
Get-ChildItem $HOME\Downloads -File | Where-Object { $_.Extension -eq ".exe" -or $_.Extension -eq ".zip" }
```

### 4. Коротший запис

Для простої умови з однією властивістю можна обійтися без `{ }` і `$_`:

```powershell
Get-ChildItem $HOME\Downloads -File | Where-Object Length -gt 100MB
```

Це те саме, що `Where-Object { $_.Length -gt 100MB }`. Але для кількох умов (`-and`, `-or`) або для обчислень на кшталт `(Get-Date).AddMonths(-6)` потрібен повний запис з фігурними дужками.

> [!TIP]
> Якщо команда має власний фільтр, використовуйте спершу його: він працює швидше. Наприклад, `Get-ChildItem -Filter *.pdf` швидше, ніж `Get-ChildItem | Where-Object { $_.Extension -eq ".pdf" }`, особливо у великих папках.

> [!NOTE]
> **PowerShell 7:** з’явилися короткі оператори з інших мов програмування: тернарний `умова ? так : ні` і `??` («якщо порожньо, то…»). У 5.1 їх немає. Для фільтрації вони не потрібні, у курсі ми обходимося без них.

## Типові помилки

### ❌ `>` замість `-gt`

```powershell
Get-ChildItem $HOME\Downloads -File | Where-Object { $_.Length > 100MB }
```

**Що сталося:** команда нічого не вивела, а в поточній папці з’явився файл з назвою `100MB`.

**Чому:** `>` — це не «більше», а перенаправлення у файл (урок 7). PowerShell записав розмір кожного файлу у файл `100MB`, а умова завжди виявлялася «порожньою», тому жоден файл фільтр не пройшов.

### ✅ Правильно

```powershell
Remove-Item .\100MB                       # прибираємо випадковий файл
Get-ChildItem $HOME\Downloads -File | Where-Object { $_.Length -gt 100MB }
```

---

### ❌ `=` замість `-eq`

```powershell
Get-ChildItem $HOME\Downloads -File | Where-Object { $_.Extension = ".zip" }
```

```text
'Extension' is a ReadOnly property.
```

**Чому:** `=` означає «присвоїти». PowerShell спробував **змінити** розширення кожного файлу на `.zip`, але змінювати цю властивість не можна. З іншими об’єктами таке присвоєння могло б спрацювати й непомітно зіпсувати дані.

### ✅ Правильно

```powershell
Get-ChildItem $HOME\Downloads -File | Where-Object { $_.Extension -eq ".zip" }
```

---

### ❌ `-like` без зірочок

```powershell
Get-ChildItem $HOME\Downloads -File | Where-Object { $_.Name -like "рахунок" }
```

**Що сталося:** порожній результат, хоча файл `рахунок-газ.pdf` є.

**Чому:** без `*` оператор `-like` працює як `-eq`: шукає назву, яка **точно** дорівнює «рахунок».

### ✅ Правильно

```powershell
Get-ChildItem $HOME\Downloads -File | Where-Object { $_.Name -like "*рахунок*" }
```

---

### ❌ Пробіл у розмірі

```powershell
Get-ChildItem $HOME\Downloads -File | Where-Object { $_.Length -gt 100 MB }
```

```text
Unexpected token 'MB' in expression or statement.
```

**Чому:** `100 MB` з пробілом — це число `100` і незрозуміле слово `MB`.

### ✅ Правильно

Пишіть разом: `100MB`.

## Підсумок

| Що | Як |
|---|---|
| Фільтр | `... \| Where-Object { умова }` |
| Поточний об’єкт | `$_`, його властивість — `$_.Назва` |
| Порівняння | `-eq -ne -gt -ge -lt -le` |
| Пошук за маскою | `-like "*текст*"` |
| Кілька умов | `-and`, `-or`, `-not` |
| Розміри | `1KB`, `100MB`, `2GB` (без пробілу) |
| Дата в минулому | `(Get-Date).AddMonths(-6)` |
| Короткий запис | `Where-Object Length -gt 100MB` |

## Перевір себе (~5 хв)

1. Що означає `$_` у `Where-Object { $_.Length -gt 1MB }`?
2. Чому в умові не можна писати `>` замість `-gt`?
3. Яка умова знайде файли з розширенням `.jpg` **або** `.png`?
4. Чим `-like "звіт"` відрізняється від `-like "звіт*"`?
5. Як записати умову «файл змінено протягом останніх 7 днів»?

<details>
<summary>Відповіді</summary>

1. Поточний об’єкт у конвеєрі, тобто файл, який зараз перевіряється.
2. `>` записує результат у файл. Умова не спрацює, а в папці з’явиться зайвий файл.
3. `{ $_.Extension -eq ".jpg" -or $_.Extension -eq ".png" }`.
4. `"звіт"` знайде лише назву, що точно дорівнює «звіт», а `"звіт*"` — усі назви, що **починаються** зі «звіт».
5. `{ $_.LastWriteTime -gt (Get-Date).AddDays(-7) }`.

</details>

## Практика (~10 хв)

**Завдання 1** (легке). Знайдіть у «Завантаженнях» усі ZIP-архіви за допомогою `Where-Object`. Спробуйте і повний, і короткий запис.

<details>
<summary>Розв’язок</summary>

```powershell
Get-ChildItem $HOME\Downloads -File | Where-Object { $_.Extension -eq ".zip" }
Get-ChildItem $HOME\Downloads -File | Where-Object Extension -eq .zip
```

</details>

**Завдання 2.** Знайдіть у папці «Зображення» (разом із вкладеними папками) усі фото `.jpg`, які більші за 5 МБ. Виведіть лише назву й розмір, від найбільшого до найменшого.

<details>
<summary>Розв’язок</summary>

```powershell
Get-ChildItem $HOME\Pictures -Filter *.jpg -Recurse |
    Where-Object { $_.Length -gt 5MB } |
    Sort-Object Length -Descending |
    Select-Object Name, Length
```

Фільтр за розширенням тут зроблено параметром `-Filter`: так швидше, ніж через `Where-Object`.

</details>

**Завдання 3** ⭐. Знайдіть у «Завантаженнях» інсталятори (`.exe` і `.msi`), старші за 3 місяці, і перевірте з `-WhatIf`, що їх можна видалити. Самі файли не видаляйте, якщо не впевнені.

<details>
<summary>Розв’язок</summary>

```powershell
Get-ChildItem $HOME\Downloads -File |
    Where-Object { ($_.Extension -eq ".exe" -or $_.Extension -eq ".msi") -and $_.LastWriteTime -lt (Get-Date).AddMonths(-3) } |
    Remove-Item -WhatIf
```

Круглі дужки навколо `-or` потрібні, щоб спершу перевірити розширення, а вже потім дату. Без них умова читалася б як «`.exe` будь-якої дати **або** старий `.msi`».

</details>

## Джерела

- [Where-Object](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/where-object?view=powershell-5.1)
- [about_Comparison_Operators](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_comparison_operators?view=powershell-5.1)
- [about_Logical_Operators](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_logical_operators?view=powershell-5.1)
- [about_Automatic_Variables — `$_`](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_automatic_variables?view=powershell-5.1)
- [about_Numeric_Literals — суфікси KB, MB, GB](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_numeric_literals?view=powershell-5.1)
- [What’s new in PowerShell 7.0 — тернарний оператор і `??`](https://learn.microsoft.com/powershell/scripting/whats-new/what-s-new-in-powershell-70)

---
[← Урок 10](10-konveyer.md) · [Зміст курсу](../../README.md) · [Урок 12. Гарний вивід і підрахунки →](12-vyvid-i-pidrakhunky.md)
