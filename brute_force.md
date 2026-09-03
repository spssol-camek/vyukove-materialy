# Brute-force a bezpečnost hesel v Pythonu

Autor: Mgr. Libor Čamek
Aktualizováno: 3. 9. 2026

## Cíl hodiny

Na konci hodiny dokážete:

- vysvětlit, co znamená **brute-force**,
- spočítat počet možných kombinací hesla,
- vytvořit jednoduchou simulaci brute-force v Pythonu,
- měřit čas běhu programu,
- vypočítat přibližnou rychlost zkoušení kombinací,
- vysvětlit, proč délka hesla a počet použitých znaků zásadně ovlivňují bezpečnost,
- porovnat teoretický počet kombinací s reálným časem výpočtu.

> Tato úloha je určena pouze k výuce principu brute-force na známém testovacím řetězci. Nepracuje s žádnými skutečnými účty, přihlašováním ani cizími hesly.

---

# 1. Co je brute-force

**Brute-force** je metoda, při které program systematicky zkouší všechny možné kombinace, dokud nenajde správnou hodnotu.

Příklad:

Máme povolené znaky:

```text
abc
```

a hledáme heslo délky 2.

Možné kombinace jsou:

```text
aa
ab
ac
ba
bb
bc
ca
cb
cc
```

Celkem tedy:

```text
3² = 9 kombinací
```

Obecně platí:

```text
počet kombinací = počet možných znaků ^ délka hesla
```

Značení:

```text
N = počet možných znaků
L = délka hesla

počet kombinací = N^L
```

---

# 2. Proč počet kombinací roste tak rychle

Každý další znak hesla násobí počet možností počtem všech povolených znaků.

Pokud máme 36 znaků:

```text
abcdefghijklmnopqrstuvwxyz0123456789
```

pak každý další znak znamená:

```text
36× více kombinací
```

Příklad:

```text
36^4 = 1 679 616
36^5 = 60 466 176
36^6 = 2 176 782 336
36^7 = 78 364 164 096
36^8 = 2 821 109 907 456
```

---

# 3. První jednoduchý program

Použijeme knihovnu `itertools`.

```python
from itertools import product
import string


# Cílové heslo známé pouze pro tuto simulaci
target_password = "abc"

# Povolené znaky: malá písmena a-z
characters = string.ascii_lowercase

# Délka hesla
password_length = len(target_password)


for combination in product(characters, repeat=password_length):

    # combination je například:
    # ('a', 'b', 'c')

    # Tuple převedeme na textový řetězec:
    guess = "".join(combination)

    # Porovnáme aktuální pokus s cílovým heslem
    if guess == target_password:

        print("Heslo nalezeno:", guess)

        break
```

---

# 4. Jak funguje `product()`

Příkaz:

```python
product("abc", repeat=2)
```

vytvoří všechny kombinace délky 2:

```text
aa
ab
ac
ba
bb
bc
ca
cb
cc
```

Python jednotlivé kombinace vrací například jako:

```python
('a', 'b')
```

Proto používáme:

```python
guess = "".join(combination)
```

Výsledek:

```text
ab
```

---

# 5. Počítání pokusů

Přidáme proměnnou:

```python
attempts = 0
```

a při každém pokusu ji zvýšíme:

```python
attempts += 1
```

Kompletní příklad:

```python
from itertools import product
import string


target_password = "code"

characters = string.ascii_lowercase

password_length = len(target_password)

attempts = 0


for combination in product(characters, repeat=password_length):

    guess = "".join(combination)

    attempts += 1

    if guess == target_password:

        print("Heslo nalezeno:", guess)
        print("Počet pokusů:", attempts)

        break
```

---

# 6. Kolik kombinací vůbec existuje

Počet kombinací můžeme spočítat ještě před spuštěním hledání.

```python
total_combinations = len(characters) ** password_length
```

Například:

```python
characters = string.ascii_lowercase
password_length = 4
```

Počet možností:

```text
26^4 = 456 976
```

---

# 7. Měření času

Použijeme knihovnu:

```python
import time
```

Před spuštěním smyčky si uložíme čas:

```python
start_time = time.time()
```

Po skončení:

```python
elapsed_time = time.time() - start_time
```

Celý příklad:

```python
from itertools import product
import string
import time


target_password = "code"

characters = string.ascii_lowercase

password_length = len(target_password)

attempts = 0

start_time = time.time()


for combination in product(characters, repeat=password_length):

    guess = "".join(combination)

    attempts += 1

    if guess == target_password:

        break


elapsed_time = time.time() - start_time


print("Heslo nalezeno:", guess)
print("Počet pokusů:", attempts)
print("Čas:", elapsed_time, "s")
```

---

# 8. Výpočet rychlosti programu

Rychlost vypočítáme:

```text
počet pokusů / čas
```

V Pythonu:

```python
speed = attempts / elapsed_time
```

Výpis:

```python
print("Rychlost:", round(speed), "pokusů/s")
```

---

# 9. Přehledná finální verze

```python
from itertools import product
import string
import time


# --------------------------------------------------
# NASTAVENÍ
# --------------------------------------------------

target_password = "code"

characters = string.ascii_lowercase

password_length = len(target_password)


# --------------------------------------------------
# VÝPOČET POČTU KOMBINACÍ
# --------------------------------------------------

total_combinations = len(characters) ** password_length


print("BRUTE-FORCE SIMULACE")
print("--------------------")

print("Cílové heslo:", target_password)
print("Počet povolených znaků:", len(characters))
print("Délka hesla:", password_length)
print("Celkový počet kombinací:", f"{total_combinations:,}")

print()


# --------------------------------------------------
# HLEDÁNÍ
# --------------------------------------------------

attempts = 0

start_time = time.time()


for combination in product(characters, repeat=password_length):

    guess = "".join(combination)

    attempts += 1

    if guess == target_password:

        break


# --------------------------------------------------
# VÝSLEDKY
# --------------------------------------------------

elapsed_time = time.time() - start_time

speed = attempts / elapsed_time


print("HESLO NALEZENO")
print("--------------------")

print("Heslo:", guess)
print("Počet pokusů:", f"{attempts:,}")
print("Čas:", round(elapsed_time, 6), "s")
print("Rychlost:", f"{round(speed):,}", "pokusů/s")
```

---

# 10. Zobrazení průběhu

U delšího hledání není vhodné vypisovat každý pokus.

Můžeme však například každých 100 000 pokusů zobrazit stav.

```python
if attempts % 100_000 == 0:

    elapsed_time = time.time() - start_time

    speed = attempts / elapsed_time

    print(
        f"Pokusů: {attempts:,} | "
        f"Aktuální kombinace: {guess} | "
        f"Rychlost: {speed:,.0f} pokusů/s"
    )
```

Kompletní varianta:

```python
from itertools import product
import string
import time


target_password = "code"

characters = string.ascii_lowercase

password_length = len(target_password)

total_combinations = len(characters) ** password_length

attempts = 0

start_time = time.time()


print("BRUTE-FORCE SIMULACE")
print("--------------------")
print("Počet znaků:", len(characters))
print("Délka hesla:", password_length)
print("Kombinací:", f"{total_combinations:,}")
print()


for combination in product(characters, repeat=password_length):

    guess = "".join(combination)

    attempts += 1


    if attempts % 100_000 == 0:

        elapsed_time = time.time() - start_time

        speed = attempts / elapsed_time

        print(
            f"Pokusů: {attempts:,} | "
            f"Aktuální kombinace: {guess} | "
            f"Rychlost: {speed:,.0f} pokusů/s"
        )


    if guess == target_password:

        break


elapsed_time = time.time() - start_time

speed = attempts / elapsed_time


print()
print("HESLO NALEZENO")
print("--------------------")
print("Heslo:", guess)
print("Pokusů:", f"{attempts:,}")
print("Čas:", round(elapsed_time, 6), "s")
print("Rychlost:", f"{speed:,.0f}", "pokusů/s")
```

---

# 11. Proč nevypisovat každý pokus

Do smyčky bychom mohli přidat:

```python
print(guess)
```

Program by potom zobrazoval:

```text
aaaa
aaab
aaac
aaad
...
```

To je názorné, ale velmi pomalé.

Výpis do terminálu je mnohem pomalejší než samotné porovnání řetězců.

Proto může být program:

```python
print(guess)
```

výrazně pomalejší než stejný program bez tohoto řádku.

---

# 12. Experiment 1 — vliv `print()`

Spusťte program ve dvou variantách.

## Varianta A

Uvnitř smyčky:

```python
print(guess)
```

## Varianta B

Bez:

```python
print(guess)
```

Zapište výsledky:

| Varianta | Čas | Počet pokusů | Rychlost |
|---|---:|---:|---:|
| s `print()` |  |  |  |
| bez `print()` |  |  |  |

### Otázka

Proč je varianta s `print()` pomalejší?

---

# 13. Experiment 2 — délka hesla

Použijte pouze malá písmena:

```python
characters = string.ascii_lowercase
```

Vyzkoušejte hesla:

```text
cat
code
hello
```

Zapište:

| Heslo | Délka | Počet kombinací | Počet pokusů | Čas |
|---|---:|---:|---:|---:|
| cat | 3 |  |  |  |
| code | 4 |  |  |  |
| hello | 5 |  |  |  |

### Otázky

1. Jak se mění počet kombinací?
2. Jak se mění doba výpočtu?
3. Je růst lineární?

---

# 14. Experiment 3 — různé množiny znaků

## Pouze číslice

```python
characters = string.digits
```

Počet znaků:

```text
10
```

---

## Malá písmena

```python
characters = string.ascii_lowercase
```

Počet znaků:

```text
26
```

---

## Malá písmena + číslice

```python
characters = (
    string.ascii_lowercase
    + string.digits
)
```

Počet znaků:

```text
36
```

---

## Malá + velká písmena + číslice

```python
characters = (
    string.ascii_lowercase
    + string.ascii_uppercase
    + string.digits
)
```

Počet znaků:

```text
62
```

---

# 15. Porovnání počtu kombinací

Pro heslo délky 8:

| Množina znaků | Počet znaků | Kombinací |
|---|---:|---:|
| číslice | 10 | 100 000 000 |
| malá písmena | 26 | 208 827 064 576 |
| malá písmena + číslice | 36 | 2 821 109 907 456 |
| malá + velká + číslice | 62 | 218 340 105 584 896 |

Je tedy vidět, že bezpečnost výrazně ovlivňuje:

- délka hesla,
- počet možných znaků.

---

# 16. Odhad doby hledání

Předpokládejme, že program zvládne:

```text
100 000 pokusů za sekundu
```

Počet sekund:

```python
seconds = combinations / attempts_per_second
```

Převod:

```python
minutes = seconds / 60
hours = minutes / 60
days = hours / 24
years = days / 365
```

Příklad:

```python
characters_count = 36
password_length = 8

attempts_per_second = 100_000


combinations = characters_count ** password_length

seconds = combinations / attempts_per_second

days = seconds / 60 / 60 / 24

years = days / 365


print("Kombinací:", f"{combinations:,}")
print("Dní:", round(days, 2))
print("Let:", round(years, 2))
```

---

# 17. Nejhorší a průměrný případ

Pokud je hledané heslo poslední možná kombinace, program musí projít:

```text
100 % kombinací
```

To je **nejhorší případ**.

V průměru lze očekávat nalezení přibližně v polovině prostoru:

```text
50 % kombinací
```

Proto můžeme orientačně počítat:

```python
average_attempts = total_combinations / 2
```

---

# 18. Úkol — vytvořte kalkulačku kombinací

Napište program, který si nechá zadat:

```text
počet znaků
délku hesla
rychlost pokusů za sekundu
```

a vypíše:

```text
počet kombinací
nejhorší odhad času
průměrný odhad času
```

Možná kostra:

```python
characters_count = int(input("Počet možných znaků: "))
password_length = int(input("Délka hesla: "))
attempts_per_second = int(input("Pokusů za sekundu: "))


combinations = characters_count ** password_length

worst_seconds = combinations / attempts_per_second

average_seconds = worst_seconds / 2


print("Počet kombinací:", combinations)
print("Nejhorší případ:", worst_seconds, "s")
print("Průměrný případ:", average_seconds, "s")
```

---

# 19. Rozšiřující úkol

Upravte program tak, aby se cílové heslo zadávalo pomocí:

```python
target_password = input("Zadejte testovací heslo: ")
```

Program před spuštěním zobrazí:

```text
Délka hesla:
Počet dostupných znaků:
Počet kombinací:
```

Potom provede simulaci a zobrazí:

```text
Počet pokusů:
Čas:
Rychlost:
```

---

# 20. Bonus — odhad před samotným spuštěním

Je dobré nejprve spočítat počet kombinací a teprve potom rozhodnout, zda má smysl simulaci skutečně spouštět.

Příklad:

```python
if total_combinations > 100_000_000:

    print("Simulace by byla příliš dlouhá.")

else:

    # brute-force smyčka
    pass
```

Tím lze zabránit tomu, aby program zbytečně běžel velmi dlouho.

---

# 21. Co z experimentu plyne

Počet kombinací neroste lineárně.

Roste exponenciálně:

```text
N^L
```

kde:

```text
N = počet možných znaků
L = délka hesla
```

Například pro 36 znaků:

```text
4 znaky:
36^4
= 1 679 616

8 znaků:
36^8
= 2 821 109 907 456
```

Heslo délky 8 tedy nemá pouze dvakrát více možností než heslo délky 4.

Má přibližně:

```text
1 679 616× více možností
```

---

# 22. Důležitá poznámka k reálnému světu

Tato simulace je zjednodušená.

Skutečná bezpečnost hesel závisí například na:

- způsobu uložení hesla,
- použité hashovací funkci,
- použití soli,
- rychlostních omezeních,
- blokování po chybných pokusech,
- vícefaktorovém ověřování,
- délce a náhodnosti hesla.

Proto nelze rychlost našeho jednoduchého Python programu přímo zaměňovat za rychlost reálných systémů.

Princip exponenciálního růstu počtu kombinací však zůstává stejný.

---

# 23. Shrnutí

Brute-force:

```text
systematicky zkouší možné kombinace
```

Počet kombinací:

```text
N^L
```

Bezpečnost roste zejména s:

```text
délkou hesla
počtem možných znaků
```

Každý další znak hesla násobí počet kombinací velikostí znakové sady.

Například při 36 možných znacích:

```text
každý další znak = 36× více možností
```

---

# 24. Otázky na závěr

1. Co znamená brute-force?
2. Jak vypočítáme počet možných kombinací?
3. Co představuje `N` ve vzorci `N^L`?
4. Co představuje `L`?
5. Proč každý další znak výrazně zvyšuje počet kombinací?
6. Co dělá funkce `product()`?
7. Proč používáme `"".join(combination)`?
8. K čemu slouží proměnná `attempts`?
9. Jak změříme čas běhu programu?
10. Jak vypočítáme počet pokusů za sekundu?
11. Proč program zpomaluje `print()` uvnitř smyčky?
12. Jaký je rozdíl mezi nejhorším a průměrným případem?
13. Proč nelze rychlost této simulace přímo převést na skutečný útok proti reálné službě?

---

# 25. Mini výzva

Bez spuštění programu odhadněte:

```text
Znaky:
a-z + 0-9

Počet znaků:
36

Délka hesla:
10
```

Kolik existuje kombinací?

```text
36^10 = ?
```

Potom výsledek ověřte v Pythonu:

```python
print(36 ** 10)
```

---

# Vtip na konec 😄

Programátor si nastaví heslo:

```text
123456
```

Admin se ptá:

> „To myslíš vážně?“

Programátor:

> „Jasně. Je to šestifaktorové heslo. Každá číslice je jeden faktor.“
