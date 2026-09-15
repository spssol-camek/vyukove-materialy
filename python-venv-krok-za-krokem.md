# Virtuální prostředí v Pythonu — krok za krokem

Praktický průvodce vytvořením a používáním virtuálního prostředí, instalací knihoven a uložením jejich verzí.

**Autor:** Mgr. Libor Čamek  
**Určeno pro:** žáky se základní znalostí Pythonu, kteří dosud nepracovali s virtuálním prostředím  
**Aktualizováno:** 15. 9. 2026

## Proč používat `venv`?

Virtuální prostředí odděluje knihovny jednotlivých projektů. Každý projekt tak může používat vlastní knihovny a jejich verze, aniž by ovlivňoval ostatní projekty.

- **`.venv`** je složka s virtuálním prostředím projektu.
- **`pip`** slouží k instalaci knihoven.
- **`requirements.txt`** obsahuje seznam knihoven a jejich verzí pro opětovnou instalaci.

> **Důležité:** Virtuální prostředí odděluje knihovny. Není to bezpečnostní izolace — spuštěný program má stále přístup k souborům a síti podle oprávnění uživatele.

## 1. Otevřete složku projektu

Ve VS Code otevřete složku projektu přes **Soubor → Otevřít složku**.

Potom otevřete **Terminál → Nový terminál**.

Pokud složka ještě neexistuje, můžete ji vytvořit v terminálu:

```bash
mkdir muj_projekt
cd muj_projekt
```

- `mkdir` vytvoří složku.
- `cd` přejde do složky.

> **Pozor:** Následující příkazy spouštějte ze složky projektu. Podle aktuální složky se určuje, kde vznikne `.venv` nebo `requirements.txt`.

## 2. Vytvořte virtuální prostředí

### Windows

```powershell
py -m venv .venv
```

Pokud používáte příkaz `python` místo `py`:

```powershell
python -m venv .venv
```

### macOS / Linux

```bash
python3 -m venv .venv
```

### Co příkaz znamená?

| Část příkazu | Význam |
|---|---|
| `py` / `python` / `python3` | Spustí Python. |
| `-m venv` | Spustí modul pro vytvoření virtuálního prostředí. |
| `.venv` | Určuje název složky prostředí. |

Ve složce projektu vznikne složka `.venv`.

**Proč:** Do tohoto prostředí budete instalovat knihovny potřebné pro projekt.

> **Pozor:** Prostředí vytváříte obvykle jen jednou. Vytvoření prostředí ho ještě neaktivuje. Vlastní kód ukládejte vedle složky `.venv`, nikoli do ní.

## 3. Aktivujte prostředí

Použijte příkaz podle systému a terminálu.

### Windows — PowerShell

```powershell
.\.venv\Scripts\Activate.ps1
```

### Windows — CMD

```cmd
.venv\Scripts\activate.bat
```

### macOS / Linux — bash nebo zsh

```bash
source .venv/bin/activate
```

Na začátku řádku terminálu se obvykle objeví označení:

```text
(.venv)
```

**Proč:** Aktivace upraví nastavení aktuálního terminálu tak, aby příkaz `python` používal Python z prostředí `.venv`.

Po aktivaci používají následující kroky na všech systémech příkaz `python`.

Aktivace není nezbytná pro fungování prostředí. Python z `.venv` lze spustit také přímo jeho cestou. V tomto návodu používáme aktivaci, abychom mohli zadávat kratší příkazy.

> **Pozor:** Aktivace platí pro aktuální terminál. Po otevření nového terminálu může být potřeba prostředí znovu aktivovat.

### Pokud PowerShell blokuje aktivaci

Pokud se zobrazí chyba, že je zakázané spouštění skriptů, můžete ve VS Code otevřít terminál **Command Prompt / CMD** a použít:

```cmd
.venv\Scripts\activate.bat
```

Nemusíte kvůli tomu měnit systémová pravidla pro spouštění skriptů.

## 4. Ověřte používaný Python

```bash
python -c "import sys; print(sys.executable)"
```

Příkaz vypíše cestu ke spuštěnému Pythonu. Cesta musí směřovat do `.venv`.

Například ve Windows:

```text
C:\projekty\muj_projekt\.venv\Scripts\python.exe
```

Na macOS nebo Linuxu:

```text
/.../muj_projekt/.venv/bin/python
```

**Proč:** Ověříte, že instalace knihoven i spouštění programu budou probíhat ve správném prostředí.

## 5. Vyberte prostředí ve VS Code

1. Otevřete paletu příkazů:
   - Windows / Linux: **Ctrl + Shift + P**
   - macOS: **Cmd + Shift + P**
2. Vyhledejte **Python: Select Interpreter**.
3. Vyberte Python ze složky **`.venv`**.

Tento příkaz poskytuje rozšíření **Python** od Microsoftu.

**Proč:** VS Code musí vědět, který Python má používat pro spouštění programu, ladění a kontrolu importů.

> **Pozor:** Výběr interpretu ve VS Code a aktivace v již otevřeném terminálu nemusí být totéž. Při problémech ověřte cestu příkazem z předchozího kroku.

## 6. Nainstalujte potřebnou knihovnu

V aktivovaném prostředí spusťte:

```bash
python -m pip install requests
```

Knihovna `requests` slouží k odesílání HTTP požadavků, například při práci s webovým API.

**Proč používáme `python -m pip`:** Instalaci provede `pip` patřící právě používanému Pythonu. Snižuje se tím riziko instalace do jiného prostředí.

Seznam nainstalovaných knihoven zobrazíte:

```bash
python -m pip list
```

> **Pozor:** Před instalací ověřte aktivní prostředí. Pro běžnou instalaci do `.venv` nepoužívejte `sudo`.

## 7. Vytvořte a spusťte program

Ve složce projektu vytvořte soubor `main.py`:

```python
import requests

print("Knihovna requests je dostupná.")
print(f"Verze knihovny: {requests.__version__}")
```

Spusťte ho:

```bash
python main.py
```

**Proč:** Tím ověříte, že Python dokáže najít a načíst nainstalovanou knihovnu.

Pokud se objeví:

```text
ModuleNotFoundError: No module named 'requests'
```

zkontrolujte:

1. Je aktivované správné prostředí?
2. Je v něm knihovna nainstalovaná?
3. Používá VS Code interpret ze složky `.venv`?

## 8. Uložte knihovny a jejich verze

```bash
python -m pip freeze > requirements.txt
```

Ve složce projektu vznikne soubor `requirements.txt`.

Ukázka jednoho řádku:

```text
requests==2.32.3
```

Číslo verze je pouze příklad. Ve vašem souboru může být jiné.

### Co příkaz znamená?

| Část | Význam |
|---|---|
| `python -m pip freeze` | Vypíše nainstalované knihovny a jejich verze ve formátu pro opětovnou instalaci. |
| `>` | Přesměruje výstup do souboru. Existující obsah přepíše. |
| `requirements.txt` | Soubor se seznamem závislostí projektu. |
| `==` | Požaduje přesnou verzi knihovny. |

**Proč:** Na jiném počítači nebo po smazání prostředí můžete znovu nainstalovat stejné verze knihoven.

### Na co si dát pozor

- Soubor obsahuje **seznam knihoven**, nikoli samotné knihovny.
- Objeví se v něm i další závislosti, které si knihovny nainstalovaly automaticky.
- `freeze` nezkoumá váš kód. Vypisuje knihovny z prostředí, i když je projekt nepoužívá.
- Soubor se **neaktualizuje automaticky**. Po změně knihoven spusťte příkaz znovu.
- Soubor **neukládá verzi Pythonu** ani systémové závislosti.
- `pip freeze` standardně vynechává některé instalační nástroje, například samotný `pip`. Nejde o úplnou zálohu prostředí.

Verzi Pythonu zjistíte:

```bash
python --version
```

Poznamenejte ji například do `README.md` spolu s návodem ke spuštění.

## 9. Deaktivujte prostředí

```bash
deactivate
```

Příkaz je stejný na Windows, macOS i Linuxu.

**Co se stane:** Terminál přestane upřednostňovat Python z aktivovaného prostředí.

**Co zůstane uložené:** Složka `.venv`, nainstalované knihovny i zdrojový kód.

> **Důležité:** Deaktivace prostředí nic nemaže. Zavřením terminálu jeho aktivace také končí.

## 10. Pokračujte v práci příště

Pokud `.venv` stále existuje:

1. Otevřete složku projektu a terminál.
2. Aktivujte prostředí příkazem pro svůj systém.
3. Spusťte program:

```bash
python main.py
```

**Proč:** Prostředí i knihovny zůstaly uložené. Nemusíte je při každém spuštění vytvářet a instalovat znovu.

## 11. Obnovte prostředí na jiném počítači

Přeneste zdrojové soubory a `requirements.txt`. Složku `.venv` nekopírujte.

Na cílovém počítači musí být nainstalovaná vhodná verze Pythonu.

### Windows — PowerShell

```powershell
# Vytvoření nového prostředí
py -m venv .venv

# Aktivace
.\.venv\Scripts\Activate.ps1

# Instalace knihoven podle souboru
python -m pip install -r requirements.txt

# Spuštění programu
python main.py
```

### macOS / Linux

```bash
# Vytvoření nového prostředí
python3 -m venv .venv

# Aktivace
source .venv/bin/activate

# Instalace knihoven podle souboru
python -m pip install -r requirements.txt

# Spuštění programu
python main.py
```

Parametr **`-r`** říká, že se mají požadavky načíst ze souboru.

**Proč vytváříme nové prostředí:** `.venv` obsahuje soubory a cesty navázané na konkrétní instalaci Pythonu a operační systém. Není určené k přenášení mezi počítači.

> **Pozor:** Stejné verze knihoven pomáhají obnovit prostředí, ale nezaručují funkčnost na každém systému a s každou verzí Pythonu.

## 12. Nastavte, co patří do GitHubu

Typická struktura projektu:

```text
muj_projekt/
├── .venv/
├── main.py
├── requirements.txt
├── README.md
└── .gitignore
```

Do souboru `.gitignore` vložte:

```gitignore
.venv/
__pycache__/
```

| Položka | Do GitHubu? | Důvod |
|---|---|---|
| `main.py` | Ano | Zdrojový kód programu. |
| `requirements.txt` | Ano | Seznam knihoven potřebný k obnově prostředí. |
| `README.md` | Ano | Popis projektu, verze Pythonu a postup spuštění. |
| `.gitignore` | Ano | Pravidla pro soubory, které Git nemá sledovat. |
| `.venv/` | Ne | Prostředí si každý vytvoří na svém počítači. |
| `__pycache__/` | Ne | Automaticky vytvářená mezipaměť Pythonu. |

> **Pozor:** `.gitignore` nezruší sledování souborů, které už byly přidané do Gitu. Proto ho vytvořte před prvním přidáním souborů.

## Přehled příkazů

Příkazy s `python` používejte po aktivaci prostředí.

| Činnost | Příkaz |
|---|---|
| Vytvoření prostředí — Windows | `py -m venv .venv` |
| Vytvoření prostředí — macOS / Linux | `python3 -m venv .venv` |
| Aktivace — Windows PowerShell | `.\.venv\Scripts\Activate.ps1` |
| Aktivace — Windows CMD | `.venv\Scripts\activate.bat` |
| Aktivace — macOS / Linux | `source .venv/bin/activate` |
| Ověření cesty k Pythonu | `python -c "import sys; print(sys.executable)"` |
| Zjištění verze Pythonu | `python --version` |
| Instalace knihovny | `python -m pip install requests` |
| Přehled knihoven | `python -m pip list` |
| Uložení knihoven a verzí | `python -m pip freeze > requirements.txt` |
| Instalace podle seznamu | `python -m pip install -r requirements.txt` |
| Spuštění programu | `python main.py` |
| Deaktivace prostředí | `deactivate` |
