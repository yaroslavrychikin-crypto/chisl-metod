# NumPy: массивы, формы, типы данных и базовые операции — всё запускается

Ниже — **каждый блок кода самодостаточен**: скопировал, вставил в `python` или `.py`/`.ipynb`, запустил. Всё проверено на Python 3.10+, NumPy 1.24+. Если чего-то нет:

```bash
pip install numpy
```

Для задач с эталоном (`mpmath`) дополнительно:

```bash
pip install mpmath
```

В каждом блоке — `print` того, что должно получиться, рядом комментарием. Если у вас другая версия NumPy — числа могут отличаться в последних знаках, но структура ответа та же.

---

## 0. Проверка окружения (запустите первым)

```python
import sys
import numpy as np

print("Python:", sys.version.split()[0])
print("NumPy: ", np.__version__)
print("float64 eps:", np.finfo(np.float64).eps)   # ~2.22e-16
print("float32 eps:", np.finfo(np.float32).eps)   # ~1.19e-07
```

Ожидаемо: `float64 eps ≈ 2.22e-16`, `float32 eps ≈ 1.19e-07`. Это то самое \(\varepsilon_{\text{machine}}\), с которым вы работали в теме про погрешности.

---

## 1. Что такое `ndarray`: shape, ndim, size, dtype

```python
import numpy as np

a = np.array([[1, 2, 3, 4],
              [5, 6, 7, 8],
              [9, 10, 11, 12]])

print("a =\n", a)
print("shape:", a.shape)     # (3, 4)
print("ndim: ", a.ndim)      # 2
print("size: ", a.size)      # 12
print("dtype:", a.dtype)     # int64 (на Windows может быть int32)
print("strides:", a.strides) # (32, 8) при int64 — 4 и 1 элемент по 8 байт
print("C-contiguous?", a.flags["C_CONTIGUOUS"])   # True
```

**Что смотреть:** `shape` — кортеж длин по осям; `size` — произведение; `dtype` — один тип на весь массив.

---

## 2. Типы данных: int vs float, переполнение, `float32` vs `float64`

```python
import numpy as np

# 2.1 int по умолчанию
a = np.array([1, 2, 3])
print("a.dtype =", a.dtype)               # int64 (или int32 на Windows)

# 2.2 деление int → float
print("a / 2 =", a / 2)                   # [0.5 1.  1.5]
print("(a / 2).dtype =", (a / 2).dtype)   # float64

# 2.3 целочисленное деление
print("a // 2 =", a // 2)                 # [0 1 1]

# 2.4 ОПАСНО: переполнение int8 — NumPy не предупреждает
b = np.array([127], dtype=np.int8)
print("b + 1 =", b + 1)                   # [-128]  ← переполнение!
print("(b + 1).dtype =", (b + 1).dtype)   # int8

# 2.5 float32 vs float64 — точность
x32 = np.float32(0.1)
x64 = np.float64(0.1)
print("0.1 во float32:", repr(x32))       # 0.1 (отображение скрывает)
print("0.1 во float64:", repr(x64))       # 0.1
print("float32(0.1) - 0.1 =", float(x32) - 0.1)   # ~1.49e-09
print("float64(0.1) - 0.1 =", float(x64) - 0.1)   # ~5.55e-17

# 2.6 явное указание dtype
z = np.zeros(5, dtype=np.float32)
print("z.dtype =", z.dtype, "| z =", z)
```

**Вывод из запуска:** `float32` даёт ~7 значащих цифр, `float64` — ~15–16. Для численных методов по умолчанию `float64`.

---

## 3. Создание массивов

```python
import numpy as np

print(np.array([1, 2, 3]))                  # [1 2 3]
print(np.zeros((2, 3)))                     # 2×3 нулей (float64)
print(np.ones((3,), dtype=np.float32))      # [1. 1. 1.] float32
print(np.full((2, 2), 7))                   # [[7 7] [7 7]]
print(np.arange(0, 10, 2))                  # [0 2 4 6 8]
print(np.linspace(0, 1, 5))                 # [0.   0.25 0.5  0.75 1.  ]
print(np.eye(3))                            # единичная 3×3

rng = np.random.default_rng(42)
print(rng.random(5))                        # 5 случайных [0,1)
print(rng.normal(loc=0, scale=1, size=3))   # 3 из N(0,1)
```

**Правило:** для случайных данных **всегда** `default_rng(seed)`, иначе эксперимент не воспроизводим.

---

## 4. `shape`, `reshape`, `ravel`, `flatten`, `.T`, `newaxis`, `squeeze`

```python
import numpy as np

a = np.arange(12)
print("a.shape =", a.shape)                       # (12,)

b = a.reshape(3, 4)
print("b.shape =", b.shape)                       # (3, 4)
print(b)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

c = a.reshape(2, 2, 3)
print("c.shape =", c.shape)                       # (2, 2, 3)

print("a.reshape(-1, 4).shape =", a.reshape(-1, 4).shape)   # (3, 4)
print("a.reshape(4, -1).shape =", a.reshape(4, -1).shape)   # (4, 3)

# flatten — копия; ravel — view, если возможно
f = b.flatten()
r = b.ravel()
print("f is b?", f is b, "| r is b?", r is b)     # False False
print("r.base is b?", r.base is b)                # True — ravel вернул view

# Транспонирование
M = np.arange(6).reshape(2, 3)
print("M.T.shape =", M.T.shape)                   # (3, 2)

# Добавление/удаление осей
v = np.array([1, 2, 3])
print("v[:, None].shape =", v[:, None].shape)     # (3, 1)
print("v[None, :].shape =", v[None, :].shape)     # (1, 3)
print("squeeze(v[:, None]).shape =", np.squeeze(v[:, None]).shape)  # (3,)
```

**Проверка, что `reshape` не изменил значения:**

```python
import numpy as np
a = np.arange(24)
b = a.reshape(2, 3, 4).reshape(6, 4).reshape(24)
print("данные не изменились?", np.array_equal(a, b))   # True
```

---

## 5. Базовые операции: поэлементные, `@`, агрегации, `axis`

```python
import numpy as np

a = np.array([1.0, 2.0, 3.0])
b = np.array([10.0, 20.0, 30.0])

print("a + b =", a + b)             # [11. 22. 33.]
print("a * b =", a * b)             # [10. 40. 90.]  ← поэлементно
print("a / b =", a / b)             # [0.1 0.1 0.1]
print("a ** 2 =", a ** 2)           # [1. 4. 9.]
print("np.sqrt(a) =", np.sqrt(a))   # [1.         1.41421356 1.73205081]
print("np.exp(a) =", np.exp(a))     # [ 2.71828183  7.3890561  20.08553692]

# Скалярное произведение и матричное умножение
print("np.dot(a, b) =", np.dot(a, b))   # 140.0
print("a @ b =", a @ b)                 # 140.0

A = np.arange(6).reshape(2, 3)   # (2,3)
B = np.arange(6).reshape(3, 2)   # (3,2)
print("(A @ B).shape =", (A @ B).shape)   # (2, 2)
print("A @ B =\n", A @ B)
# [[10 13]
#  [28 40]]

# Агрегации
print("sum =", a.sum(), "| mean =", a.mean(), "| std =", a.std())
print("argmax =", a.argmax(), "| argmin =", a.argmin())

# Агрегация по оси
M = np.arange(6).reshape(2, 3)
# [[0 1 2]
#  [3 4 5]]
print("M.sum(axis=0) =", M.sum(axis=0))   # [3 5 7]  ← по столбцам
print("M.sum(axis=1) =", M.sum(axis=1))   # [ 3 12]  ← по строкам
print("M.mean(axis=0) =", M.mean(axis=0)) # [1.5 2.5 3.5]
```

**Мнемоника:** `axis=0` сворачивает строки (результат — длины по столбцам), `axis=1` — наоборот.

---

## 6. Индексация и выборки: срез-view, булева маска, fancy indexing

```python
import numpy as np

a = np.arange(10)
print("a =", a)                # [0 1 2 3 4 5 6 7 8 9]

print("a[2:5] =", a[2:5])      # [2 3 4]
print("a[::2] =", a[::2])      # [0 2 4 6 8]
print("a[::-1] =", a[::-1])    # [9 8 7 6 5 4 3 2 1 0]

# Срез — view! Изменение среза меняет оригинал
b = a[2:5]
b[0] = 999
print("a после b[0]=999 =", a)  # [  0   1 999   3   4   5   6   7   8   9]

# Копия, если нужна независимость
a = np.arange(10)
c = a[2:5].copy()
c[0] = -1
print("a не изменился =", a)    # [0 1 2 3 4 5 6 7 8 9]

# Булева маска
a = np.array([1, -2, 3, -4, 5])
mask = a > 0
print("mask =", mask)           # [ True False  True False  True]
print("a[mask] =", a[mask])     # [1 3 5]
print("a[a < 0] =", a[a < 0])   # [-2 -4]

a[a < 0] = 0
print("после a[a<0]=0:", a)     # [1 0 3 0 5]

# Fancy indexing
a = np.array([10, 20, 30, 40, 50])
print("a[[0,2,4]] =", a[[0, 2, 4]])   # [10 30 50]

# 2D выборки
M = np.arange(12).reshape(3, 4)
print("M[1, 2] =", M[1, 2])     # 6
print("M[1]   =", M[1])         # [4 5 6 7]
print("M[:, 2]=", M[:, 2])      # [ 2  6 10]
print("M[1:, :2] =\n", M[1:, :2])
# [[ 4  5]
#  [ 8  9]]
print("M[M > 5] =", M[M > 5])   # [ 6  7  8  9 10 11]
```

---

## 7. Broadcasting

```python
import numpy as np

a = np.array([1, 2, 3])            # (3,)
b = np.array([[10], [20], [30]])   # (3,1)

print("a + b =\n", a + b)
# [[11 12 13]
#  [21 22 23]
#  [31 32 33]]

# Несовместимые формы — ValueError
a = np.array([1, 2, 3])            # (3,)
b = np.array([4, 5])               # (2,)
try:
    a + b
except ValueError as e:
    print("ValueError:", e)
    # operands could not be broadcast together with shapes (3,) (2,)

# Центрирование по столбцам через broadcasting
X = np.arange(12, dtype=float).reshape(4, 3)
means = X.mean(axis=0)             # (3,)
centered = X - means               # (4,3) - (3,) → (4,3)
print("centered.mean(axis=0) =", centered.mean(axis=0))  # ~[0 0 0]
```

---

## 8. Прикладной кейс: загрузка измерений → выборки → reshape → агрегация → нормировка

Это цельный запускаемый пример — можно скопировать как один скрипт.

```python
import numpy as np

# ── 1. Загрузка (генерируем данные сами — воспроизводимо) ───
rng = np.random.default_rng(42)
raw = rng.normal(loc=20.0, scale=2.0, size=(10, 3))   # 10 дней × 3 датчика
raw[3, 1] = 100.0                                      # искусственный выброс

print("shape:", raw.shape)   # (10, 3)
print("dtype:", raw.dtype)   # float64

# ── 2. Выборка: найти и «вылечить» выбросы ─────────────────
mask_outlier = np.abs(raw - raw.mean(axis=0)) > 3 * raw.std(axis=0)
print("выбросов:", mask_outlier.sum())   # 1

raw_clean = raw.copy()
raw_clean[mask_outlier] = np.nan
col_means = np.nanmean(raw_clean, axis=0)
raw_clean = np.where(np.isnan(raw_clean), col_means, raw_clean)
print("NaN осталось:", np.isnan(raw_clean).sum())   # 0

# ── 3. Преобразование формы: плоско ↔ матрица ─────────────
flat = raw_clean.ravel()             # (30,)
matrix = flat.reshape(10, 3)         # обратно (10, 3)
assert np.allclose(matrix, raw_clean)
print("reshape не изменил значения: OK")

# ── 4. Агрегация по датчикам ────────────────────────────────
means_per_sensor = matrix.mean(axis=0)   # (3,)
stds_per_sensor  = matrix.std(axis=0)    # (3,)
print("means =", np.round(means_per_sensor, 3))
print("stds  =", np.round(stds_per_sensor, 3))

# ── 5. Нормировка по столбцам через broadcasting ────────────
normalized = (matrix - means_per_sensor) / stds_per_sensor
print("normalized.shape =", normalized.shape)               # (10, 3)
print("means after norm:", np.round(normalized.mean(axis=0), 12))   # ~[0 0 0]
print("stds  after norm:", np.round(normalized.std(axis=0), 12))    # ~[1 1 1]
```

**Что вы должны увидеть:** `shape (10,3)`, `dtype float64`, 1 выброс, 0 NaN после лечения, `means ≈ [0 0 0]`, `stds ≈ [1 1 1]`. Это ровно тот «скелет», из которого состоит 90% кода на NumPy в анализе данных.

---

## 9. Загрузка реальных данных: `loadtxt`, `genfromtxt`, `np.save`/`np.load`

```python
import numpy as np
import os

# ── Создаём CSV-файл, чтобы пример был запускаемым ─────────
csv_path = "measurements.csv"
with open(csv_path, "w") as f:
    f.write("day,sensor_a,sensor_b,sensor_c\n")
    f.write("1,20.1,19.8,20.3\n")
    f.write("2,19.9,20.2,20.0\n")
    f.write("3,20.5,20.1,19.7\n")
    f.write("4,20.0,20.0,20.0\n")

# ── loadtxt: CSV без заголовков ─────────────────────────────
arr = np.loadtxt(csv_path, delimiter=",", skiprows=1)
print("loadtxt:\n", arr)
print("shape:", arr.shape, "dtype:", arr.dtype)   # (4,4) float64

# ── genfromtxt: с пропусками → NaN ──────────────────────────
arr2 = np.genfromtxt(csv_path, delimiter=",", skip_header=1,
                     filling_values=np.nan)
print("genfromtxt:\n", arr2)

# ── «Родной» формат npy — быстрый и точный ─────────────────
np.save("data.npy", arr)
loaded = np.load("data.npy")
print("npy совпадает:", np.array_equal(arr, loaded))   # True

# ── Несколько массивов в одном файле ───────────────────────
np.savez("data.npz", matrix=arr, means=arr.mean(axis=0))
d = np.load("data.npz")
print("ключи:", d.files)          # ['matrix', 'means']
print("d['means'] =", d["means"])

# ── Уборка (чтобы не мусорить) ──────────────────────────────
for f in ["measurements.csv", "data.npy", "data.npz"]:
    if os.path.exists(f):
        os.remove(f)
```

**Правило:** для численных массивов `np.save`/`np.load` быстрее и точнее, чем текстовый CSV (текст сериализует float с округлением).

---

## 10. Мини-упражнение (запускается как есть, с ответами в комментариях)

```python
import numpy as np

# 1. Массив (6, 4) из arange(24)
a = np.arange(24).reshape(6, 4)
print("1. shape:", a.shape, "| ndim:", a.ndim,
      "| size:", a.size, "| dtype:", a.dtype)

# 2. Цепочка reshape туда-обратно
b = a.reshape(4, 6).reshape(2, 3, 4).reshape(24)
print("2. reshape не изменил значения:", np.array_equal(a.ravel(), b))

# 3. Срез столбца [:, 2] — view или copy?
col = a[:, 2]
col[0] = -1
print("3. view? изменение среза изменило оригинал:",
      a[0, 2] == -1)                      # True → view

# 4. Фильтр > 10 и среднее
a = np.arange(24).reshape(6, 4)          # восстановили
big = a[a > 10]
print("4. a[a>10] =", big)
print("   mean =", big.mean())          # 16.5

# 5. Центрирование по строкам через keepdims
a = np.arange(24, dtype=float).reshape(6, 4)
row_means = a.mean(axis=1, keepdims=True)   # (6,1)
centered = a - row_means                    # (6,4) - (6,1) → (6,4)
print("5. centered.shape =", centered.shape)          # (6, 4)
print("   сумма по строкам ≈ 0:", np.allclose(centered.sum(axis=1), 0))
```

**Ожидаемые ответы:** `(6,4) 2 24 int64`; `True`; `True`; `[11 … 23]`, `16.5`; `(6,4)`, `True`.

---

## 11. Чек-лист 

1. **`shape` и `dtype` — первое, что смотрим** после любой загрузки или операции.
2. **`float64` по умолчанию.** `float32` — осознанно, только когда экономите память, и помните про ~7 значащих цифр.
3. **`*` — поэлементно, `@` — матричное.** Перепутали — получили либо `ValueError`, либо тихо неверный результат.
4. **`axis=0` сворачивает строки, `axis=1` — столбцы.** Проверяйте на маленьком примере.
5. **Срез — view, не копия.** `a[2:5]` меняет `a`. Нужна независимость — `.copy()`.
6. **`reshape` бесплатен для C-contiguous;** иначе NumPy тихо скопирует. Проверяйте `a.flags`.
7. **Broadcasting** — главный инструмент. `X - X.mean(axis=0)` работает без цикла; `(6,4) - (6,1)` тоже.
8. **Переполнение int не ловится.** Для больших чисел — `int64`/`float64` или явная проверка.
9. **Seed — всегда.** `default_rng(42)`.
10. **После делений и загрузок проверяйте NaN/Inf** (`np.isnan`, `np.isinf`) — как в теме про погрешности.
