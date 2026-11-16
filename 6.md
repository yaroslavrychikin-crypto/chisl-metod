# Сравнение кода: время, память и визуализация с Matplotlib

Сравнить два куска кода — значит ответить на два вопроса: **сколько времени** они работают и **сколько памяти** тратят. Ниже — всё, что нужно, без лишней теории. Каждый пример можно скопировать и запустить.

---

## Часть 1: Измерение времени

### 1.1 `time.perf_counter()` — замер одного блока

Простой и точный таймер. Используйте его вместо `time.time()`.

```python
import time

# ============================================================
# Пример 1: Ручной замер с perf_counter
# ============================================================

def sum_with_loop(n):
    """Суммирование через цикл"""
    total = 0
    for i in range(n):
        total += i
    return total

def sum_with_builtin(n):
    """Суммирование через встроенную функцию"""
    return sum(range(n))

# --- Замер первого варианта ---
start = time.perf_counter()
result1 = sum_with_loop(1_000_000)
end = time.perf_counter()
time_loop = end - start
print(f"Цикл:       {time_loop:.6f} сек, результат={result1}")

# --- Замер второго варианта ---
start = time.perf_counter()
result2 = sum_with_builtin(1_000_000)
end = time.perf_counter()
time_builtin = end - start
print(f"Встроенная: {time_builtin:.6f} сек, результат={result2}")

print(f"\nВстроенная функция быстрее в {time_loop / time_builtin:.1f} раз")
```

**Что важно:** один замер ненадежен — CPU может быть занят. Для сравнения делайте несколько замеров.

### 1.2 `timeit` — для сравнения коротких фрагментов

Автоматически повторяет замеры и берет лучший результат.

```python
import timeit

# ============================================================
# Пример 2: Сравнение двух способов конкатенации строк
# ============================================================

# Вариант 1: конкатенация через +=
stmt1 = """
s = ""
for i in range(100):
    s += str(i)
"""

# Вариант 2: join
stmt2 = """
s = "".join(str(i) for i in range(100))
"""

# number=10000 — выполнить каждый фрагмент 10000 раз
# repeat=5 — провести 5 серий и взять лучшую
times1 = timeit.repeat(stmt1, number=10000, repeat=5)
times2 = timeit.repeat(stmt2, number=10000, repeat=5)

best1 = min(times1) / 10000 * 1_000_000  # микросекунды
best2 = min(times2) / 10000 * 1_000_000

print(f"+= конкатенация: {best1:.2f} мкс на операцию")
print(f"join:            {best2:.2f} мкс на операцию")
print(f"join быстрее в {best1 / best2:.1f} раз")

print(f"\nВсе замеры для +=:   {[f'{t/10000*1e6:.2f}' for t in times1]} мкс")
print(f"Все замеры для join: {[f'{t/10000*1e6:.2f}' for t in times2]} мкс")
```

---

## Часть 2: Измерение памяти с `tracemalloc`

Стандартный модуль для отслеживания памяти в Python.

```python
import tracemalloc

# ============================================================
# Пример 3: Память для list vs generator
# ============================================================

N = 1_000_000

# --- Вариант 1: список (хранит все элементы) ---
tracemalloc.start()
list_data = [i for i in range(N)]
current, peak = tracemalloc.get_traced_memory()
tracemalloc.stop()

print("=== Список ===")
print(f"Текущая память: {current / 1024:.1f} КБ")
print(f"Пиковая память: {peak / 1024:.1f} КБ")

# --- Вариант 2: генератор (вычисляет по одному) ---
tracemalloc.start()
gen_data = (i for i in range(N))
current_gen, peak_gen = tracemalloc.get_traced_memory()
tracemalloc.stop()

print("\n=== Генератор (до вычислений) ===")
print(f"Текущая память: {current_gen / 1024:.1f} КБ")
print(f"Пиковая память: {peak_gen / 1024:.1f} КБ")

# --- Потребление при фактической обработке ---
tracemalloc.start()
total = sum(gen_data)
current_gen2, peak_gen2 = tracemalloc.get_traced_memory()
tracemalloc.stop()

print("\n=== Генератор (во время sum) ===")
print(f"Пиковая память: {peak_gen2 / 1024:.1f} КБ")
print(f"Результат суммы: {total}")
```

**Вывод:** список на миллион чисел занимает ~8 МБ, генератор — несколько сотен байт.

### Память для list vs set vs dict

```python
import tracemalloc

# ============================================================
# Пример 4: Память для разных структур данных
# ============================================================

N = 500_000

def measure_memory(data_type):
    """Измеряет пиковую память при создании структуры."""
    tracemalloc.start()
    if data_type == "list":
        data = list(range(N))
    elif data_type == "set":
        data = set(range(N))
    elif data_type == "dict":
        data = {i: i for i in range(N)}
    current, peak = tracemalloc.get_traced_memory()
    tracemalloc.stop()
    return peak / 1024 / 1024  # МБ

print("=== Пиковая память для 500,000 элементов ===")
print(f"list: {measure_memory('list'):.1f} МБ")
print(f"set:  {measure_memory('set'):.1f} МБ")
print(f"dict: {measure_memory('dict'):.1f} МБ")
```

**Вывод:** set и dict едят больше памяти, чем list — это плата за быстрый поиск.

---

## Часть 3: Полный пример — сравнение алгоритмов с визуализацией

Объединяем всё: замеряем время и память, строим графики.

```python
import time
import tracemalloc
import matplotlib.pyplot as plt

# ============================================================
# ПОЛНЫЙ ПРИМЕР: Поиск дубликатов тремя способами
# ============================================================

def has_duplicates_quadratic(arr):
    """Вложенные циклы — медленно"""
    n = len(arr)
    for i in range(n):
        for j in range(i + 1, n):
            if arr[i] == arr[j]:
                return True
    return False

def has_duplicates_sorting(arr):
    """Сортировка + проверка соседей"""
    sorted_arr = sorted(arr)
    for i in range(len(sorted_arr) - 1):
        if sorted_arr[i] == sorted_arr[i + 1]:
            return True
    return False

def has_duplicates_set(arr):
    """Через set — быстро"""
    seen = set()
    for x in arr:
        if x in seen:
            return True
        seen.add(x)
    return False

# --- Функция для замера времени и памяти ---
def benchmark(func, arr, repeat=3):
    """Возвращает (лучшее_время_сек, пиковая_память_МБ)."""
    times = []
    peaks = []
    for _ in range(repeat):
        tracemalloc.start()
        start = time.perf_counter()
        func(arr)
        end = time.perf_counter()
        current, peak = tracemalloc.get_traced_memory()
        tracemalloc.stop()
        times.append(end - start)
        peaks.append(peak / 1024 / 1024)
    return min(times), max(peaks)

# --- Генерируем данные разного размера ---
# Массивы без дубликатов, чтобы каждый алгоритм прошел весь путь
sizes = [100, 200, 400, 800, 1600]
algorithms = {
    "Вложенные циклы": has_duplicates_quadratic,
    "Сортировка":      has_duplicates_sorting,
    "Множество (set)": has_duplicates_set,
}

results_time = {name: [] for name in algorithms}
results_mem = {name: [] for name in algorithms}

print("Запуск бенчмарков...\n")
for n in sizes:
    arr = list(range(n))
    print(f"Размер n={n}:")
    for name, func in algorithms.items():
        t, m = benchmark(func, arr)
        results_time[name].append(t * 1000)  # мс
        results_mem[name].append(m)
        print(f"  {name:20s}  время={t*1000:8.3f} мс  память={m:6.2f} МБ")
    print()

# ============================================================
# ВИЗУАЛИЗАЦИЯ С MATPLOTLIB
# ============================================================

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# --- График времени ---
ax = axes[0]
for name in algorithms:
    ax.plot(sizes, results_time[name], marker='o', label=name, linewidth=2)
ax.set_xlabel("Размер входных данных (n)", fontsize=12)
ax.set_ylabel("Время выполнения (мс)", fontsize=12)
ax.set_title("Сравнение времени выполнения", fontsize=14, fontweight='bold')
ax.set_xscale('log')
ax.set_yscale('log')
ax.legend(fontsize=10)
ax.grid(True, alpha=0.3)

# --- График памяти ---
ax = axes[1]
for name in algorithms:
    ax.plot(sizes, results_mem[name], marker='s', label=name, linewidth=2)
ax.set_xlabel("Размер входных данных (n)", fontsize=12)
ax.set_ylabel("Пиковая дополнительная память (МБ)", fontsize=12)
ax.set_title("Сравнение потребления памяти", fontsize=14, fontweight='bold')
ax.set_xscale('log')
ax.legend(fontsize=10)
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()

# ============================================================
# Интерпретация
# ============================================================
print("\n" + "="*60)
print("ИНТЕРПРЕТАЦИЯ:")
print("="*60)
print("""
1. Время: вложенные циклы растут катастрофически быстрее
   остальных. Сортировка и set близки, но set стабильно
   быстрее, потому что не тратит время на сортировку.

2. Память: set и сортировка требуют O(n) дополнительной
   памяти (сам set или копия списка). Вложенные циклы
   используют почти O(1) памяти — это их единственный
   плюс, но он не компенсирует медленность.

3. Вывод: set — лучший выбор в большинстве случаев.
   Сортировка оправдана, если данные уже частично
   отсортированы или нужна дополнительная обработка.
""")
```

---

Справедливо, в прошлый раз `matplotlib` получил слишком мало внимания. Ниже — расширенный раздел, посвящённый именно визуализации результатов сравнения кода: от базовых настроек до продвинутых приёмов, которые делают графики читаемыми и информативными.

---

## Часть 3 (расширенная): Matplotlib для сравнения кода

### 3.1 Базовая структура: `Figure` и `Axes`

В `matplotlib` есть два подхода: старый `plt.plot()` (работает с «текущим» графиком) и объектный `fig, ax = plt.subplots()`. Для сравнения кода всегда используйте объектный — он даёт полный контроль над каждым элементом .

```python
import matplotlib.pyplot as plt
import numpy as np

# ============================================================
# Пример 5: Figure и Axes — объектный подход
# ============================================================
# fig — всё окно (холст), ax — конкретная область с осями.

x = np.linspace(0, 10, 100)

# Создаём Figure с одной Axes
fig, ax = plt.subplots(figsize=(8, 5))

# Рисуем две линии с разными стилями
ax.plot(x, x**2, label="O(n²)", color="red", linestyle="--", linewidth=2)
ax.plot(x, x, label="O(n)", color="blue", linestyle="-", linewidth=2)

# Подписи осей и заголовок
ax.set_xlabel("Размер данных (n)", fontsize=12)
ax.set_ylabel("Время (условные единицы)", fontsize=12)
ax.set_title("Сравнение роста сложности", fontsize=14, fontweight="bold")

# Легенда — какая линия что означает
ax.legend(fontsize=10, loc="upper left")

# Сетка для удобства чтения
ax.grid(True, alpha=0.3)

plt.tight_layout()  # автоматически подгоняет отступы
plt.show()
```

**Ключевые объекты:**
- `fig` — всё изображение. Его размер задаётся `figsize=(ширина, высота)` в дюймах.
- `ax` — система координат. У неё есть `xaxis`, `yaxis`, `title`, `legend` и методы `plot()`, `bar()`, `scatter()`.
- `plt.tight_layout()` — убирает обрезание подписей по краям.

---

### 3.2 Несколько графиков на одном рисунке: `subplots`

Для сравнения времени и памяти рядом нужны две `Axes` в одной `Figure` .

```python
import matplotlib.pyplot as plt
import numpy as np

# ============================================================
# Пример 6: Два графика рядом (1 строка, 2 столбца)
# ============================================================

sizes = [100, 200, 400, 800]
time_quad = [0.1, 0.4, 1.6, 6.4]      # O(n²)
time_lin  = [0.01, 0.02, 0.04, 0.08]  # O(n)
mem_quad  = [0.01, 0.01, 0.01, 0.01]  # O(1) память
mem_lin   = [0.1, 0.2, 0.4, 0.8]      # O(n) память

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 4))

# --- Левый график: время ---
ax1.plot(sizes, time_quad, "ro-", label="Вложенные циклы (O(n²))")
ax1.plot(sizes, time_lin,  "bs-", label="Множество (O(n))")
ax1.set_xlabel("Размер данных (n)")
ax1.set_ylabel("Время (мс)")
ax1.set_title("Время выполнения")
ax1.legend()
ax1.grid(True, alpha=0.3)

# --- Правый график: память ---
ax2.plot(sizes, mem_quad, "ro-", label="Вложенные циклы (O(1) память)")
ax2.plot(sizes, mem_lin,  "bs-", label="Множество (O(n) память)")
ax2.set_xlabel("Размер данных (n)")
ax2.set_ylabel("Память (МБ)")
ax2.set_title("Потребление памяти")
ax2.legend()
ax2.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

**Распаковка `axs`:** при `subplots(1, 2)` возвращается кортеж `(ax1, ax2)`. При `subplots(2, 2)` — двумерный массив, по которому удобно итерироваться через `for ax in axs.flat:` .

---

### 3.3 Логарифмические оси — когда разница в порядках

Если один алгоритм работает за миллисекунды, а другой за секунды, линейная шкала сделает быстрый график плоской линией у нуля. Логарифмическая шкала решает эту проблему.

```python
import matplotlib.pyplot as plt

# ============================================================
# Пример 7: Логарифмические оси
# ============================================================

sizes = [10, 100, 1000, 10000, 100000]
time_lin   = [0.001, 0.01, 0.1, 1.0, 10.0]
time_quad  = [0.0001, 0.01, 1.0, 100.0, 10000.0]

fig, ax = plt.subplots(figsize=(8, 5))

ax.plot(sizes, time_lin,  "o-", label="O(n)")
ax.plot(sizes, time_quad, "s-", label="O(n²)")

# Логарифм по обеим осям — видно, что O(n²) растёт гораздо круче
ax.set_xscale("log")
ax.set_yscale("log")

ax.set_xlabel("Размер данных (n, лог. шкала)", fontsize=12)
ax.set_ylabel("Время (мс, лог. шкала)", fontsize=12)
ax.set_title("Логарифмическая шкала выявляет разницу сложностей", fontsize=13)
ax.legend(fontsize=11)
ax.grid(True, which="both", alpha=0.2)  # which="both" — сетка и для мелких делений

plt.tight_layout()
plt.show()
```

**Почему это работает:** на лог-лог графике степенная зависимость `y = x^k` превращается в прямую с наклоном `k`. O(n²) наклонена вдвое круче, чем O(n). Если алгоритм на графике «выпрямляется» в лог-лог — его сложность степенная.

---

### 3.4 Две оси Y на одном графике: `twinx`

Иногда нужно показать время и память на одном графике, но у них разные единицы (мс и МБ). `twinx()` создаёт вторую ось Y справа, разделяющую ту же ось X .

```python
import matplotlib.pyplot as plt

# ============================================================
# Пример 8: twinx — время слева, память справа
# ============================================================

sizes = [100, 200, 400, 800, 1600]
time_ms = [0.1, 0.4, 1.6, 6.4, 25.6]
mem_mb  = [0.5, 1.0, 2.0, 4.0, 8.0]

fig, ax1 = plt.subplots(figsize=(9, 5))

# --- Левая ось: время ---
color_time = "tab:red"
ax1.set_xlabel("Размер данных (n)", fontsize=12)
ax1.set_ylabel("Время (мс)", color=color_time, fontsize=12)
line1 = ax1.plot(sizes, time_ms, "o-", color=color_time, label="Время")
ax1.tick_params(axis="y", labelcolor=color_time)

# --- Правая ось: память ---
ax2 = ax1.twinx()  # вторая ось Y, делит ту же X
color_mem = "tab:blue"
ax2.set_ylabel("Память (МБ)", color=color_mem, fontsize=12)
line2 = ax2.plot(sizes, mem_mb, "s--", color=color_mem, label="Память")
ax2.tick_params(axis="y", labelcolor=color_mem)

# Общая легенда для двух осей
lines = line1 + line2
labels = [l.get_label() for l in lines]
ax1.legend(lines, labels, loc="upper left", fontsize=10)

ax1.set_title("Время и память на одном графике", fontsize=14, fontweight="bold")
ax1.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

**Важно:** `twinx()` создаёт независимую ось Y, но общую X. Цвета осей и подписей помогают читателю понять, какая линия к какой оси относится .

---

### 3.5 Столбчатые диаграммы для категориальных сравнений

Когда нужно сравнить несколько алгоритмов на фиксированных размерах данных, столбцы читаются легче, чем линии.

```python
import matplotlib.pyplot as plt
import numpy as np

# ============================================================
# Пример 9: Группированные столбцы — время трёх алгоритмов
# ============================================================

algorithms = ["Вложенные циклы", "Сортировка", "Множество"]
sizes = ["n=100", "n=400", "n=1600"]

# Данные: строки — алгоритмы, столбцы — размеры
times = np.array([
    [0.1, 1.6, 25.6],   # вложенные циклы
    [0.05, 0.3, 1.5],   # сортировка
    [0.02, 0.1, 0.5],   # множество
])

x = np.arange(len(sizes))  # позиции групп по X
width = 0.25               # ширина одного столбца

fig, ax = plt.subplots(figsize=(10, 5))

# Три группы столбцов, сдвинутые друг относительно друга
ax.bar(x - width, times[0], width, label=algorithms[0], color="red", alpha=0.8)
ax.bar(x,          times[1], width, label=algorithms[1], color="orange", alpha=0.8)
ax.bar(x + width,  times[2], width, label=algorithms[2], color="green", alpha=0.8)

ax.set_xlabel("Размер входных данных", fontsize=12)
ax.set_ylabel("Время (мс)", fontsize=12)
ax.set_title("Сравнение трёх алгоритмов поиска дубликатов", fontsize=14)
ax.set_xticks(x)
ax.set_xticklabels(sizes)
ax.legend(fontsize=10)
ax.grid(True, axis="y", alpha=0.3)  # только горизонтальная сетка

plt.tight_layout()
plt.show()
```

**Ключевой приём:** `x - width`, `x`, `x + width` сдвигают столбцы внутри группы, чтобы они стояли рядом, а не друг на друге. Для стековых диаграмм используется параметр `bottom` .

---

### 3.6 Сохранение графиков в файл

Для отчётов и презентаций графики нужно сохранять. `savefig()` делает это с контролем разрешения и обрезки .

```python
import matplotlib.pyplot as plt
import numpy as np

# ============================================================
# Пример 10: Сохранение с настройками
# ============================================================

x = np.linspace(0, 10, 50)
fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(x, np.sin(x), label="sin(x)")
ax.plot(x, np.cos(x), label="cos(x)")
ax.legend()
ax.set_title("Демонстрация savefig")

# Базовое сохранение — формат по расширению
fig.savefig("plot_basic.png")

# С контролем качества:
# dpi=150 — выше разрешение (для печати/презентаций)
# bbox_inches="tight" — обрезает лишние поля вокруг графика
# transparent=True — прозрачный фон (для вставки в слайды)
fig.savefig("plot_hd.png", dpi=150, bbox_inches="tight", transparent=True)

# Векторный формат для масштабирования без потерь
fig.savefig("plot_vector.svg", bbox_inches="tight")

print("Сохранено: plot_basic.png, plot_hd.png, plot_vector.svg")
plt.show()
```

**Параметры, которые стоит запомнить:**
- `dpi` — точек на дюйм. 72 (по умолчанию) для экрана, 150–300 для печати.
- `bbox_inches="tight"` — убирает пустые поля, график занимает весь файл.
- `transparent=True` — фон прозрачный, удобно для наложения.
- `format="pdf"` или `.svg` — векторный формат, не размывается при увеличении .

---

### 3.7 Настройка внешнего вида: цвета, маркеры, линии

`matplotlib` позволяет управлять каждой деталью. Вот основные параметры, которые чаще всего нужны при сравнении кода .

```python
import matplotlib.pyplot as plt
import numpy as np

# ============================================================
# Пример 11: Полный контроль над стилем
# ============================================================

x = np.linspace(0, 10, 20)

fig, ax = plt.subplots(figsize=(9, 5))

# Линия 1: сплошная, красная, с кружками
ax.plot(x, x**1.5,
        color="crimson",          # цвет
        linestyle="-",            # сплошная
        linewidth=2,              # толщина
        marker="o",               # маркер-кружок
        markersize=6,             # размер маркера
        markerfacecolor="white",  # заливка маркера
        markeredgecolor="crimson",# обводка маркера
        label="O(n^1.5)")

# Линия 2: штриховая, синяя, с квадратами
ax.plot(x, x**2,
        color="steelblue",
        linestyle="--",           # штриховая
        linewidth=2,
        marker="s",               # квадрат
        markersize=6,
        label="O(n²)")

ax.set_xlabel("n", fontsize=12)
ax.set_ylabel("Время", fontsize=12)
ax.set_title("Стилизация линий и маркеров", fontsize=14)
ax.legend(fontsize=11)
ax.grid(True, alpha=0.25)

plt.tight_layout()
plt.show()
```

**Шпаргалка по стилям линий:** `"-"` сплошная, `"--"` штриховая, `"-."` штрих-пунктир, `":"` точечная. **Маркеры:** `"o"` круг, `"s"` квадрат, `"^"` треугольник, `"D"` ромб, `"*"` звезда .

---

### 3.8 Полный пример: все приёмы вместе

Соберём финальный скрипт, который сравнивает алгоритмы и рисует информативный дашборд.

```python
import time
import tracemalloc
import matplotlib.pyplot as plt
import numpy as np

# ============================================================
# ФИНАЛЬНЫЙ ПРИМЕР: Сравнение алгоритмов с полной визуализацией
# ============================================================

def has_duplicates_quadratic(arr):
    n = len(arr)
    for i in range(n):
        for j in range(i + 1, n):
            if arr[i] == arr[j]:
                return True
    return False

def has_duplicates_sorting(arr):
    sorted_arr = sorted(arr)
    for i in range(len(sorted_arr) - 1):
        if sorted_arr[i] == sorted_arr[i + 1]:
            return True
    return False

def has_duplicates_set(arr):
    seen = set()
    for x in arr:
        if x in seen:
            return True
        seen.add(x)
    return False

def benchmark(func, arr, repeat=3):
    times, peaks = [], []
    for _ in range(repeat):
        tracemalloc.start()
        start = time.perf_counter()
        func(arr)
        end = time.perf_counter()
        _, peak = tracemalloc.get_traced_memory()
        tracemalloc.stop()
        times.append(end - start)
        peaks.append(peak / 1024 / 1024)
    return min(times), max(peaks)

# --- Данные ---
sizes = [100, 200, 400, 800, 1600]
algorithms = {
    "Вложенные циклы": has_duplicates_quadratic,
    "Сортировка":      has_duplicates_sorting,
    "Множество":       has_duplicates_set,
}

res_time = {k: [] for k in algorithms}
res_mem  = {k: [] for k in algorithms}

print("Запуск...")
for n in sizes:
    arr = list(range(n))
    for name, func in algorithms.items():
        t, m = benchmark(func, arr)
        res_time[name].append(t * 1000)  # мс
        res_mem[name].append(m)          # МБ
    print(f"  n={n}: готово")

# ============================================================
# ВИЗУАЛИЗАЦИЯ: дашборд 2×2
# ============================================================

fig, axes = plt.subplots(2, 2, figsize=(14, 10))
fig.suptitle("Сравнение алгоритмов поиска дубликатов", fontsize=16, fontweight="bold")

# --- (0,0) Линейный график времени ---
ax = axes[0, 0]
for name, data in res_time.items():
    ax.plot(sizes, data, marker="o", label=name, linewidth=2)
ax.set_xlabel("Размер данных (n)")
ax.set_ylabel("Время (мс)")
ax.set_title("Время выполнения (линейная шкала)")
ax.legend(fontsize=9)
ax.grid(True, alpha=0.3)

# --- (0,1) Лог-лог график времени ---
ax = axes[0, 1]
for name, data in res_time.items():
    ax.plot(sizes, data, marker="s", label=name, linewidth=2)
ax.set_xscale("log")
ax.set_yscale("log")
ax.set_xlabel("Размер данных (n, лог)")
ax.set_ylabel("Время (мс, лог)")
ax.set_title("Время выполнения (лог-лог: наклон = сложность)")
ax.legend(fontsize=9)
ax.grid(True, which="both", alpha=0.2)

# --- (1,0) Столбцы: память ---
ax = axes[1, 0]
x = np.arange(len(sizes))
width = 0.25
for i, (name, data) in enumerate(res_mem.items()):
    ax.bar(x + (i - 1) * width, data, width, label=name, alpha=0.85)
ax.set_xticks(x)
ax.set_xticklabels([f"n={n}" for n in sizes])
ax.set_ylabel("Пиковая память (МБ)")
ax.set_title("Потребление памяти")
ax.legend(fontsize=9)
ax.grid(True, axis="y", alpha=0.3)

# --- (1,1) twinx: время + память для «Множества» ---
ax1 = axes[1, 1]
name = "Множество"
ax1.plot(sizes, res_time[name], "o-", color="tab:red", label="Время (левая ось)")
ax1.set_xlabel("Размер данных (n)")
ax1.set_ylabel("Время (мс)", color="tab:red")
ax1.tick_params(axis="y", labelcolor="tab:red")

ax2 = ax1.twinx()
ax2.plot(sizes, res_mem[name], "s--", color="tab:blue", label="Память (правая ось)")
ax2.set_ylabel("Память (МБ)", color="tab:blue")
ax2.tick_params(axis="y", labelcolor="tab:blue")

ax1.set_title(f"«{name}»: время и память вместе")
ax1.grid(True, alpha=0.3)

# Объединённая легенда
lines1, labels1 = ax1.get_legend_handles_labels()
lines2, labels2 = ax2.get_legend_handles_labels()
ax1.legend(lines1 + lines2, labels1 + labels2, loc="upper left", fontsize=9)

plt.tight_layout(rect=[0, 0, 1, 0.96])  # место для suptitle
fig.savefig("algorithm_comparison.png", dpi=150, bbox_inches="tight")
plt.show()

print("\nГрафик сохранён: algorithm_comparison.png")
```

**Что показывает этот дашборд:**
- Верхний левый: наглядно видно, что O(n²) уходит вверх.
- Верхний правый: в лог-лог масштабе наклон линий показывает сложность — вложенные циклы круче всех.
- Нижний левый: столбцы показывают, что set и сортировка тратят память, а вложенные циклы — почти нет.
- Нижний правый: `twinx` позволяет увидеть компромисс «время vs память» для одного алгоритма на одном графике.

---

### Сводка по Matplotlib

| Приём | Метод | Когда использовать |
|-------|-------|-------------------|
| Один график | `fig, ax = plt.subplots()` | База, всегда |
| Несколько графиков | `plt.subplots(rows, cols)` | Время и память рядом |
| Лог-шкала | `ax.set_xscale("log")` | Разница в порядках величин |
| Две оси Y | `ax2 = ax1.twinx()` | Разные единицы на одном X |
| Столбцы | `ax.bar()` | Категориальные сравнения |
| Сохранение | `fig.savefig(..., dpi=150, bbox_inches="tight")` | Отчёты, презентации |
| Стили | `color`, `linestyle`, `marker`, `linewidth` | Читаемость линий |

**Главное правило:** график должен отвечать на вопрос «какой алгоритм лучше и при каких условиях». Лог-шкала, `twinx` и группированные столбцы — три инструмента, которые чаще всего нужны для сравнения кода.

---

## Часть 4: Сводка — что когда использовать

| Инструмент | Что измеряет | Когда использовать |
|------------|-------------|---------------------|
| **`time.perf_counter()`** | Время одного блока | Быстрый замер в скрипте |
| **`timeit`** | Время с повторениями | Сравнение коротких фрагментов |
| **`tracemalloc`** | Пиковую и текущую память | Поиск утечек, сравнение структур |
| **`matplotlib`** | Визуализация | Презентация результатов |

### Типичные ошибки

| Ошибка | Как исправить |
|--------|---------------|
| `time.time()` для коротких замеров | Используйте `perf_counter()` |
| Один замер без повторений | `timeit.repeat(repeat=5)` |
| Замер с включенным отладчиком | Запускайте без отладчика |
| Сравнение кода на разных машинах | Сравнивайте только на одной машине |
| Игнорирование памяти | Всегда проверяйте `tracemalloc` для больших данных |

---

## Заключение

Чтобы сравнить код, нужно три вещи: **`perf_counter` / `timeit`** для времени, **`tracemalloc`** для памяти и **`matplotlib`** для наглядности. Замеряйте несколько раз, смотрите на разброс и не забывайте, что быстрый код не всегда экономный по памяти.


## Часть 5 задачи на отработку

### Задача 1: Сравнить два способа проверки вхождения в коллекцию

**Условие.** Даны два способа проверить, есть ли число в коллекции: `x in list` и `x in set`. Напишите бенчмарк, который для `n = 100, 1000, 10000` замеряет время 1000 проверок в `list` и в `set`. Постройте график с лог-шкалой по Y.

**Требования:**
- Используйте `timeit` (не ручной замер).
- Сделайте `repeat=5`, берите минимум.
- Лог-шкала по оси Y.
- Выведите, во сколько раз `set` быстрее на каждом `n`.

**Текстовые подсказки:**

1. **Подготовка данных.** Для каждого `n` создайте `data_list = list(range(n))` и `data_set = set(range(n))`. Ищите число, которого в коллекции **нет** (например, `-1`) — это худший случай для `list`: он пройдёт весь список до конца, и разница с `set` будет максимально наглядной.

2. **Как передать переменные в `timeit`.** Функция `timeit.repeat(stmt, globals=..., number=..., repeat=...)` принимает словарь `globals`, в котором лежат переменные, используемые в строке `stmt`. Например, `globals={"target": target, "data_list": data_list}`. Без этого `timeit` не увидит ваши данные.

3. **Нормировка на одну проверку.** `timeit` возвращает **суммарное** время за `number` выполнений. Чтобы получить время одной проверки, разделите результат на `number`. Затем умножьте на `1_000_000`, чтобы перевести секунды в микросекунды — так числа читаемее.

4. **Выбор лучшего замера.** `timeit.repeat` возвращает список из `repeat` значений. Берите `min(...)` — это защищает от случайных выбросов CPU.

5. **Лог-шкала.** После `ax.plot(...)` вызовите `ax.set_yscale("log")`. Для сетки, которая видна и на мелких делениях, используйте `ax.grid(True, which="both", alpha=0.2)`.

6. **Ожидаемый результат.** На лог-шкале линия `set` должна быть почти горизонтальной (O(1)), а линия `list` — расти с наклоном (O(n)). Если `list` не растёт — вы, скорее всего, ищете существующий элемент, и `list` находит его в начале.

7. **Вывод в консоль.** В цикле по `n` печатайте время обоих вариантов и отношение `t_list / t_set` — это и есть «во сколько раз быстрее».

---

### Задача 2: Память для трёх способов создать список квадратов

**Условие.** Создайте список квадратов чисел `0..N-1` тремя способами: цикл с `append`, списочное включение (list comprehension) и `list(map(lambda ...))`. Для `N = 100_000` и `N = 1_000_000` измерьте время (`timeit`) и пиковую память (`tracemalloc`). Постройте два графика рядом: время и память.

**Требования:**
- Два `subplots` в одной строке.
- На графике памяти — столбцы (bar), на графике времени — линии с маркерами.
- Сделайте вывод, какой способ лучший и почему.

**Текстовые подсказки:**

1. **Три фрагмента кода.** Для `timeit` подготовьте три строки: `"out = []\nfor i in range(N):\n    out.append(i*i)"`, `"out = [i*i for i in range(N)]"` и `"out = list(map(lambda i: i*i, range(N)))"`. Обратите внимание на `\n` — многострочный код в `timeit` допустим, но отступы должны быть корректными.

2. **Замер времени.** Для каждого `N` и каждого фрагмента вызовите `timeit.repeat(stmt, globals={"N": N}, number=1, repeat=5)` и возьмите `min(...)`. `number=1` здесь достаточно: создание списка на миллион элементов — не микрооперация, повторять её 1000 раз внутри одного замера не нужно. Умножьте результат на `1000`, чтобы получить миллисекунды.

3. **Замер памяти.** `tracemalloc` работает так: `tracemalloc.start()` → выполнить код → `tracemalloc.get_traced_memory()` (возвращает кортеж `(current, peak)`) → `tracemalloc.stop()`. Вам нужен `peak` — пиковая память. Переведите байты в МБ делением на `1024 * 1024`.

4. **Как выполнить тот же код для замера памяти.** `timeit` выполняет строку, а `tracemalloc` — нет. Используйте `exec(stmt, {"N": N})` — это выполнит ту же строку в отдельном пространстве имён. Важно: `exec` не возвращает значение, но нам оно и не нужно — нам нужен факт создания списка.

5. **Графики рядом.** `fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(13, 5))`. На `ax1` — линии времени (`ax1.plot(...)` с маркерами `"o-"`), на `ax2` — столбцы памяти. Для столбцов по трём алгоритмам на каждом `N` используйте группировку: `x = range(len(N_values))`, `width = 0.25`, и сдвигайте столбцы на `(i - 1) * width`.

6. **Подписи на оси X для столбцов.** После `ax2.bar(...)` вызовите `ax2.set_xticks(list(x))` и `ax2.set_xticklabels([f"N={N}" for N in N_values])`. Без этого на оси X будут числа 0, 1, а не размеры.

7. **Ожидаемый результат по памяти.** Все три способа дадут **примерно одинаковую** пиковую память — ведь результат один и тот же список на миллион `int`. Промежуточные объекты (`append`-вызовы, lambda-вызовы) краткоживущие и почти не влияют на пик. Это важный вывод: память определяется **результатом**, а не способом его получения.

8. **Ожидаемый результат по времени.** List comprehension обычно быстрее всех: цикл выполняется на уровне C без вызовов `append`. `map + lambda` проигрывает из-за накладных расходов на вызов lambda для каждого элемента. Ручной цикл с `append` — где-то посередине или медленнее всех.

9. **Формулировка вывода.** В конце скрипта напишите `print(...)` с тремя пунктами: (а) по памяти все способы равны, потому что результат одинаковый; (б) по времени list comprehension быстрее из-за отсутствия вызовов `append`; (в) `map + lambda` медленнее из-за накладных расходов на lambda.

