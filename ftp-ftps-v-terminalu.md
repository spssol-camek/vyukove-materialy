# FTP a bezpečné nasazení webu z terminálu

Autor: Mgr. Libor Čamek
Aktualizováno: 3. 9. 2026

**Téma:** Přenos souborů mezi počítačem a webovým serverem pomocí FTP/FTPS a nástroje `lftp`   
**Výchozí situace:** nový web je připravený v místní složce `www/`, na hostingu zatím běží starý WordPress

> Bezpečnostní zásada celé hodiny: žáci nepracují s produkčním heslem ani s ostrým serverem. Praktická část probíhá na testovacím účtu, školním serveru nebo pouze jako simulace.

---

## 1. Smysl hodiny

Žák běžně vidí web jako stránku v prohlížeči. Tato hodina ukazuje „druhou stranu“ webu: soubory musí být fyzicky uloženy na serveru, ve správném adresáři, pod správnými názvy a s odpovídajícím zabezpečením.

Hodina propojuje:

- počítačové sítě,
- klient–server komunikaci,
- souborový systém,
- šifrování a certifikáty,
- práci v terminálu,
- správu webhostingu,
- zálohování a řízení rizika.

Nejde jen o naučení jednoho příkazu. Cílem je porozumět tomu, **co se při nasazování webu skutečně děje a jak zabránit ztrátě dat**.

---

## 2. Výukové cíle

Na konci hodiny žák:

1. vysvětlí rozdíl mezi lokálním a vzdáleným souborovým systémem;
2. popíše účel protokolů FTP, FTPS a SFTP;
3. vysvětlí, proč obyčejné FTP není vhodné pro přenos hesel;
4. rozliší řídicí a datové spojení FTP;
5. vysvětlí rozdíl mezi aktivním a pasivním režimem;
6. bezpečně přečte a rozebere příkaz nástroje `lftp`;
7. rozliší stahování, nahrávání, kopírování a přejmenování;
8. vysvětlí význam dokumentového kořene webu;
9. navrhne bezpečný postup nasazení nové verze webu;
10. pozná nebezpečné přepínače a operace, zejména mazání a `--delete`.

### Očekávané výstupy žáka

Žák dokáže vlastními slovy vysvětlit například:

> „Příkaz se připojí šifrovaně k serveru, přihlásí uživatele, nahraje místní složku do připravené vzdálené složky a původní web zatím nesmaže.“

---

## 3. Potřebné předchozí znalosti

Žák by měl znát:

- pojem soubor a adresář;
- absolutní a relativní cestu alespoň intuitivně;
- základní představu o internetu, IP adrese a doméně;
- rozdíl mezi webovým prohlížečem a webovým serverem;
- základní práci s příkazovým řádkem je výhodou, nikoli podmínkou.

---

## 4. Pomůcky a příprava učitele

- počítač s projekcí;
- terminál;
- nainstalovaný nástroj `lftp`;
- testovací FTP/FTPS účet bez cenných dat;
- dvě místní testovací složky, například `stary_web/` a `novy_web/`;
- připravený jednoduchý `index.html`;
- volitelně FileZilla pro porovnání grafického a terminálového klienta;
- tabule pro nákres klient–server.

### Doporučené testovací soubory

```text
novy_web/
├── index.html
├── css/
│   └── style.css
└── images/
    └── logo.png
```

Učitel před hodinou ověří, že testovací účet nemá přístup k produkčním webům a že případné smazání testovacích dat není škodlivé.

---

## 5. Časový scénář hodiny

### Varianta 45 minut

| Čas | Fáze | Činnost učitele | Činnost žáků |
|---:|---|---|---|
| 0–5 min | Motivace | Ukáže hotový web a položí otázku: „Jak se tyto soubory dostaly na server?“ | Formulují hypotézy. |
| 5–15 min | Teorie | Vysvětlí klient–server, FTP, FTPS, SFTP a dvě FTP spojení. | Doplňují schéma, ptají se. |
| 15–22 min | Souborový systém | Vysvětlí místní/vzdálenou cestu a dokumentový kořen. | Určují zdroj a cíl v příkladech. |
| 22–32 min | Demonstrace | Připojení, výpis adresářů, bezpečné nahrání do mezisložky. | Sledují výstup, komentují příkazy. |
| 32–40 min | Aktivita | Ve dvojicích analyzují příkaz a seřazují kroky nasazení. | Vypracují pracovní úkol. |
| 40–45 min | Ověření | Krátký kvíz a exit ticket. | Odevzdají odpověď jednou větou. |

### Rozšíření na 60 minut

V minutách 45–60 přidejte:

- praktické přejmenování testovacích složek;
- návrh rollbacku;
- analýzu chybného příkazu s `--delete`;
- porovnání FTP nasazení s Git/CI/CD.

---

## 6. Motivační situace

Učitel promítne dvě místa:

```text
Počítač vývojáře                     Webhosting
----------------                     ----------
projekt/www/index.php      ?         /domains/moje-domena.cz/index.php
projekt/www/css/style.css  ?         /domains/moje-domena.cz/css/style.css
```

Otázky pro třídu:

1. Která strana je lokální a která vzdálená?
2. Co musí být splněno, aby server soubory přijal?
3. Co se stane, když nový web nahrajeme přes starý bez zálohy?
4. Proč nestačí soubor poslat e-mailem správci hostingu?

Pointa:

> Nasazení webu je řízený přenos souborů do adresáře, ze kterého webový server obsluhuje požadavky návštěvníků.

---

## 7. Teoretický základ

### 7.1 Model klient–server

**Klient** zahajuje komunikaci a žádá o službu. V této hodině je klientem program `lftp` spuštěný na našem počítači.

**Server** čeká na spojení, ověřuje uživatele a podle jeho oprávnění umožňuje číst nebo měnit vzdálené soubory.

```text
┌─────────────────────────┐              ┌─────────────────────────┐
│ Lokální počítač         │              │ Webhosting              │
│                         │              │                         │
│ terminál → lftp klient  │ ───────────► │ FTP/FTPS server         │
│ místní soubory          │ ◄─────────── │ vzdálené soubory        │
└─────────────────────────┘              └─────────────────────────┘
```

Pojmy „upload“ a „download“ vždy vztahujeme k pohledu klienta:

- **upload**: z klienta na server;
- **download**: ze serveru ke klientovi.

### 7.2 Co je FTP

FTP znamená **File Transfer Protocol**. Je to aplikační síťový protokol určený k přenosu souborů a práci se vzdálenými adresáři. Základní specifikaci popisuje [RFC 959](https://www.rfc-editor.org/info/rfc959/).

FTP umí například:

- ověřit uživatele;
- vypsat vzdálený adresář;
- změnit pracovní adresář;
- stáhnout nebo nahrát soubor;
- vytvořit, přejmenovat či odstranit soubor nebo adresář;
- pokračovat v některých přerušených přenosech.

Klasické FTP samo o sobě **nešifruje přihlašovací údaje ani přenášená data**. Na nedůvěryhodné síti je tedy lze odposlechnout.

### 7.3 Proč FTP používá dvě spojení

Na rozdíl od mnoha jednodušších protokolů FTP odděluje:

1. **řídicí spojení** – příkazy a odpovědi;
2. **datové spojení** – výpis adresáře nebo obsah souboru.

```text
Klient                                        Server
  │                                              │
  ├──── řídicí spojení: USER, PASS, LIST ───────►│
  │◄──────── odpovědi: 220, 230, 226 ───────────┤
  │                                              │
  ├════ datové spojení: obsah souboru ══════════►│
  │                                              │
```

Řídicí spojení běžně používá TCP port 21. Datové spojení se vytváří podle aktivního nebo pasivního režimu.

### 7.4 Aktivní a pasivní režim

#### Aktivní režim

Klient sdělí serveru, na kterém portu čeká, a server zahájí datové spojení směrem ke klientovi. To bývá problém za firewallem nebo NATem.

#### Pasivní režim

Server sdělí klientovi port, na kterém čeká na datové spojení, a obě spojení zahájí klient. Tento režim lépe funguje v dnešních sítích a za firewally.

V příkazu `curl` jej můžeme výslovně požadovat:

```bash
--ftp-pasv
```

### 7.5 FTP, FTPS a SFTP nejsou totéž

| Technologie | Princip | Typický port | Šifrování |
|---|---|---:|---|
| FTP | původní protokol pro přenos souborů | 21 | ne |
| FTPS / FTPES | FTP zabezpečené pomocí TLS | 21 u explicitního FTPS | ano |
| implicitní FTPS | TLS se očekává ihned po spojení | často 990 | ano |
| SFTP | přenos souborů jako součást SSH | obvykle 22 | ano |

**FTPS není SFTP.** Podobný název neznamená stejný protokol.

Explicitní FTPS začne spojením na portu 21 a následně přejde na TLS. Zabezpečení FTP pomocí TLS popisuje [RFC 4217](https://www.rfc-editor.org/info/rfc4217/).

### 7.6 Co poskytuje TLS

TLS poskytuje zejména:

- **důvěrnost** – data nejsou čitelná pro odposlouchávajícího;
- **integritu** – změnu dat během přenosu lze odhalit;
- **ověření serveru** – certifikát pomáhá ověřit, že komunikujeme se správným serverem.

Proto používáme:

```text
set ftp:ssl-force true
set ftp:ssl-protect-data true
set ssl:verify-certificate yes
```

Význam:

- bez TLS se klient odmítne připojit;
- šifrované je i datové spojení;
- certifikát serveru musí být důvěryhodný.

Vypnutí kontroly certifikátu sice někdy „vyřeší“ chybové hlášení, ale současně odstraní důležitou ochranu proti podvrženému serveru.

---

## 8. Lokální a vzdálený souborový systém

Při práci s FTP současně existují dva pracovní adresáře:

```text
lokální pracovní adresář     vzdálený pracovní adresář
na našem počítači            na serveru
```

Příklad:

```text
lokálně:    /projekty/autobazar/www/
vzdáleně:  /domains/autobazar.example/
```

Stejný název souboru může existovat na obou stranách, ale jde o dvě různé kopie.

### 8.1 Dokumentový kořen

**Dokumentový kořen** je adresář, ze kterého webový server zpřístupňuje web návštěvníkům.

Například:

```text
https://www.example.cz/              → /domains/example.cz/index.php
https://www.example.cz/css/style.css → /domains/example.cz/css/style.css
```

Když soubory nahrajeme o úroveň vedle nebo vytvoříme omylem `www/www/`, server je na očekávané adrese nenajde.

### 8.2 Příklad struktury WEDOS

Vedlejší FTP účet může být nasměrovaný přímo do `/www/`. V něm lze vidět například:

```text
/www/
├── .htaccess                 ← směrování hostingu; nepřepisovat bez rozmyslu
└── domains/
    ├── example.cz/           ← aktivní web
    └── example.cz-new/       ← připravená nová verze
```

WEDOS popisuje adresářovou strukturu a správu webových souborů ve své [znalostní bázi](https://kb.vedos.cz/webhosting-soubory/).

---

## 9. Terminál a anatomie příkazu

Ukázka promptu:

```text
student@notebook projekt %
```

- `student@notebook` označuje uživatele a počítač;
- `projekt` je aktuální adresář;
- `%` nebo `$` je výzva shellu;
- píšeme až za tuto výzvu;
- text `Password:` je výzva programu, nikoli příkaz.

Při bezpečném zadávání hesla se na obrazovce obvykle neobjeví ani hvězdičky. Klávesnice přesto funguje.

### 9.1 Proč heslo nepíšeme do příkazu

Nevhodně:

```bash
# Nedělat: heslo by mohlo zůstat v historii shellu nebo být viditelné procesům.
lftp -u uzivatel,tajne-heslo ftp://ftp.example.cz
```

Lépe:

```bash
lftp -u uzivatel ftp://ftp.example.cz
```

Program se na heslo zeptá skrytě.

---

## 10. Nástroj `curl` pro rychlou kontrolu FTP

`curl` se hodí pro jednoduchý jednorázový požadavek. Následující příkaz pouze vypíše obsah vzdáleného kořene:

```bash
curl --ssl-reqd \
  --ftp-pasv \
  --user uzivatel_ftp \
  --list-only \
  ftp://ftp.example.cz/
```

### Význam částí

| Část | Význam |
|---|---|
| `curl` | spustí klienta |
| `--ssl-reqd` | vyžaduje TLS |
| `--ftp-pasv` | použije pasivní režim |
| `--user uzivatel_ftp` | určí uživatelské jméno; heslo se doplní skrytě |
| `--list-only` | vypíše jen názvy položek |
| `ftp://ftp.example.cz/` | server a vzdálená cesta |

Příklad výstupu:

```text
.htaccess
domains
```

To je read-only kontrola. Sama o sobě nic nenahrává ani nemaže.

---

## 11. Nástroj `lftp`

`lftp` je interaktivní i skriptovatelný klient. Vedle FTP/FTPS podporuje také další protokoly. Pro nasazení webu je užitečný především příkaz `mirror`.

### 11.1 Základní příkazy

| Příkaz | Operace | Strana |
|---|---|---|
| `pwd` | zobrazí vzdálený pracovní adresář | server |
| `lpwd` | zobrazí lokální pracovní adresář | počítač |
| `cls -la` | podrobný výpis vzdáleného adresáře včetně skrytých položek | server |
| `cd cesta` | změní vzdálený adresář | server |
| `lcd cesta` | změní lokální adresář | počítač |
| `get soubor` | stáhne soubor | server → počítač |
| `put soubor` | nahraje soubor | počítač → server |
| `mkdir nazev` | vytvoří vzdálený adresář | server |
| `mv zdroj cil` | přejmenuje nebo přesune vzdálenou položku | server |
| `mirror zdroj cil` | zrcadlí vzdálený adresář lokálně | server → počítač |
| `mirror --reverse zdroj cil` | zrcadlí lokální adresář na server | počítač → server |
| `bye` | ukončí spojení | — |

### 11.2 Směr `mirror`

Pomůcka:

```text
mirror             vzdáleně → lokálně
mirror --reverse   lokálně   → vzdáleně
```

`--reverse` lze zkrátit na `-R`.

### 11.3 Nebezpečný přepínač `--delete`

Příkaz:

```bash
mirror --reverse --delete www/ domains/example.cz/
```

nejen nahraje nové soubory, ale může z cíle odstranit soubory, které ve zdroji nejsou. To může být při špatně zadané cestě katastrofální.

Pro první nasazení je bezpečnější:

```bash
mirror --reverse --verbose --parallel=4 --no-perms \
  www/ domains/example.cz-new/
```

Bez `--delete`.

### 11.4 Důležité pojistky

```text
set cmd:fail-exit yes
```

Při chybě ukončí dávku příkazů. Bez této pojistky by mohl klient pokračovat k dalšímu kroku, i když předchozí krok nevyšel.

```text
--verbose
```

Vypisuje prováděné operace.

```text
--parallel=4
```

Přenáší několik souborů souběžně.

```text
--no-perms
```

Nepokouší se přenášet místní unixová oprávnění, která nemusejí odpovídat hostingu.

Podrobnou syntaxi popisuje [manuál `lftp`](https://manpages.debian.org/trixie/lftp/lftp.1.en.html).

---

## 12. Bezpečný model nasazení webu

### 12.1 Rizikový postup

```text
nový web → rovnou přes starý web
```

Rizika:

- část starých a část nových souborů může vytvořit nefunkční směs;
- při přerušení přenosu zůstane web neúplný;
- odstraněné soubory nelze snadno obnovit;
- rollback je pomalý nebo nemožný.

### 12.2 Doporučený postup přes mezisložku

```text
1. Aktivní web:      example.cz/
2. Nahrajeme nový:   example.cz-new/
3. Ověříme přenos.
4. Starý přejmenujeme na zálohu.
5. Nový přejmenujeme na aktivní název.
```

Stav před přepnutím:

```text
domains/
├── example.cz/       ← původní aktivní web
└── example.cz-new/   ← připravený nový web
```

Stav po přepnutí:

```text
domains/
├── example.cz/                       ← nový aktivní web
└── example.cz-wordpress-backup/      ← původní web
```

Přejmenování adresáře na stejném serveru je obvykle výrazně rychlejší než opětovný přenos všech souborů.

### 12.3 Nahrání nové verze

Obecný příklad:

```bash
lftp -u uzivatel_ftp -e '
  set cmd:fail-exit yes;
  set ftp:ssl-force true;
  set ftp:ssl-protect-data true;
  set ssl:verify-certificate yes;
  mkdir -p domains/example.cz-new;
  mirror --reverse --verbose --parallel=4 --no-perms \
    www/ domains/example.cz-new/;
  cls -la domains/example.cz-new;
  bye
' ftp://ftp.example.cz
```

Pro jednorázové použití může být příkaz na jednom řádku. Víceřádková podoba je však pro výuku čitelnější.

### 12.4 Vratné přepnutí

```bash
lftp -u uzivatel_ftp -e '
  set cmd:fail-exit yes;
  set ftp:ssl-force true;
  set ftp:ssl-protect-data true;
  set ssl:verify-certificate yes;
  mv domains/example.cz domains/example.cz-wordpress-backup;
  mv domains/example.cz-new domains/example.cz;
  cls -la domains;
  bye
' ftp://ftp.example.cz
```

Příkaz:

1. naváže šifrované spojení;
2. přejmenuje původní web na zálohu;
3. přejmenuje připravený web na aktivní název;
4. vypíše výsledek;
5. ukončí spojení.

Mezi oběma `mv` může být web velmi krátce nedostupný.

### 12.5 Rollback

Pokud nový web nefunguje, postup obrátíme. Nejdříve však nový web neodstraňujeme, ale opět jen přejmenujeme:

```text
example.cz                    → example.cz-nefunkcni
example.cz-wordpress-backup   → example.cz
```

Ukázkový příkaz:

```bash
lftp -u uzivatel_ftp -e '
  set cmd:fail-exit yes;
  set ftp:ssl-force true;
  set ftp:ssl-protect-data true;
  set ssl:verify-certificate yes;
  mv domains/example.cz domains/example.cz-nefunkcni;
  mv domains/example.cz-wordpress-backup domains/example.cz;
  cls -la domains;
  bye
' ftp://ftp.example.cz
```

---

## 13. Záloha není totéž co přejmenování

Přejmenovaná složka na témže serveru umožňuje rychlý rollback, ale není plnohodnotnou zálohou.

### Skutečná záloha by měla být

- na jiném úložišti;
- časově označená;
- ověřená;
- obnovitelná;
- chráněná před stejnou havárií jako originál.

### Dynamický web má zpravidla dvě části

```text
1. soubory na FTP
2. databáze
```

U WordPressu samotné stažení souborů nestačí. Články, uživatelé a velká část nastavení jsou v databázi. Pro úplnou zálohu je potřeba také databázový export.

```text
FTP záloha bez databáze ≠ úplná záloha WordPressu
```

WEDOS popisuje ruční zálohu souborů i databáze v návodu [Webhosting – Zálohování](https://kb.vedos.cz/webhosting-zalohovani/).

---

## 14. Skryté a konfigurační soubory

Soubor začínající tečkou je v unixových systémech skrytý:

```text
.htaccess
```

Výpis:

```bash
cls -la
```

zahrnuje i skryté položky.

`.htaccess` může řídit například:

- přesměrování HTTP na HTTPS;
- canonical variantu s `www`;
- zákaz výpisu adresářů;
- bezpečnostní hlavičky;
- blokování přístupu k interním souborům;
- cache a kompresi.

Chybný nebo hostingem nepovolený příkaz v `.htaccess` může způsobit chybu HTTP 500. Proto systémový `.htaccess` hostingu nepřepisujeme, dokud přesně nevíme, jakou má úlohu.

---

## 15. HTTP kontrola po nasazení

FTP potvrzuje přenos souborů. Neříká však, zda webový server stránku správně obslouží. Po nasazení proto následuje HTTP kontrola.

### 15.1 Pouze hlavičky odpovědi

```bash
curl -I https://www.example.cz/
```

### 15.2 Přesměrování

```bash
curl -I http://example.cz/
```

Očekáváme například:

```text
HTTP/1.1 301 Moved Permanently
Location: https://www.example.cz/
```

### 15.3 Typické stavové kódy

| Kód | Význam |
|---:|---|
| 200 | požadavek uspěl |
| 301 | trvalé přesměrování |
| 302 | dočasné přesměrování |
| 403 | přístup zakázán |
| 404 | soubor nebo stránka nenalezena |
| 500 | vnitřní chyba serveru, často konfigurace nebo aplikace |

### 15.4 Kontrolní seznam

- [ ] hlavní stránka vrací 200;
- [ ] HTTP se přesměruje na HTTPS;
- [ ] doména bez `www` se přesměruje na zvolenou variantu;
- [ ] CSS, JavaScript a obrázky se načtou;
- [ ] administrace funguje a není indexována;
- [ ] neveřejné soubory nejsou dostupné;
- [ ] formuláře nebo změnové operace fungují;
- [ ] `robots.txt` a `sitemap.xml` vracejí 200;
- [ ] v konzoli prohlížeče nejsou chyby;
- [ ] existuje funkční cesta zpět.

---

## 16. Řízená demonstrace učitele

### Krok 1: Ověření místního umístění

```bash
pwd
ls -la
```

Učitel zdůrazní, že příkaz FTP spouštíme ze složky, ve které se skutečně nachází místní `www/`.

### Krok 2: Read-only připojení

```bash
curl --ssl-reqd --ftp-pasv \
  --user uzivatel_ftp \
  --list-only \
  ftp://ftp.example.cz/
```

Žáci určují, která část je server, uživatel, bezpečnostní požadavek a vzdálená cesta.

### Krok 3: Nahrání do mezisložky

Učitel spustí `mirror --reverse` bez `--delete` a nechá žáky předem odhadnout výsledek.

### Krok 4: Kontrola

```text
cls -la domains/example.cz-new
```

### Krok 5: Přepnutí

Učitel nejdřív popíše obě přejmenování, teprve potom příkaz spustí.

### Krok 6: HTTP test

```bash
curl -I https://www.example.cz/
```

---

## 17. Pracovní úkol pro žáky

### Úloha A: Rozbor příkazu

Vysvětlete vlastními slovy každou část:

```bash
lftp -u skolni_ucet -e '
  set cmd:fail-exit yes;
  set ftp:ssl-force true;
  mirror --reverse --verbose web/ domains/test.example-new/;
  bye
' ftp://ftp.example.cz
```

1. Kde je uživatelské jméno?
2. Kde je heslo?
3. Která cesta je místní?
4. Která cesta je vzdálená?
5. Jakým směrem se přenášejí data?
6. Co se stane při chybě?
7. Smaže příkaz starý web?

### Úloha B: Seřazení postupu

Seřaďte kroky:

- ověřit nový web přes HTTP;
- připojit se šifrovaně;
- nahrát nový web do vedlejší složky;
- přejmenovat původní web na zálohu;
- zjistit správný dokumentový kořen;
- přejmenovat nový web na aktivní název;
- připravit rollback.

### Úloha C: Najdi chyby

```bash
lftp -u admin,123456 ftp://ftp.example.cz \
  -e 'mirror --reverse --delete / domains/example.cz; bye'
```

Najděte alespoň čtyři rizika.

### Úloha D: Navrhni rollback

Na serveru je:

```text
example.cz/              ← nový, ale nefunkční web
example.cz-backup/       ← starý funkční web
```

Navrhněte dvě bezpečná přejmenování, která obnoví starý web a zachovají nový k pozdější analýze.

---

## 18. Řešení pracovních úloh

### Řešení A

1. Uživatelské jméno je `skolni_ucet` za `-u`.
2. Heslo v příkazu není; program se na něj zeptá.
3. Místní cesta je `web/`.
4. Vzdálená cesta je `domains/test.example-new/`.
5. `--reverse` znamená počítač → server.
6. `cmd:fail-exit yes` dávku při chybě ukončí.
7. Nesmaže; cílem je vedlejší složka a není použit `--delete`.

### Řešení B

1. zjistit správný dokumentový kořen;
2. připravit rollback;
3. připojit se šifrovaně;
4. nahrát nový web do vedlejší složky;
5. přejmenovat původní web na zálohu;
6. přejmenovat nový web na aktivní název;
7. ověřit nový web přes HTTP.

### Řešení C

- slabé heslo;
- heslo je přímo v příkazu a může zůstat v historii;
- není vynuceno TLS;
- zdrojem je kořen `/`, tedy mimořádně široká cesta;
- `--delete` může odstranit vzdálené soubory;
- cíl je přímo aktivní web;
- chybí mezisložka;
- chybí `cmd:fail-exit`;
- chybí ověření certifikátu;
- chybí kontrola výsledku.

### Řešení D

```text
example.cz          → example.cz-nefunkcni
example.cz-backup   → example.cz
```

Pořadí je důležité, protože nelze mít dvě položky se stejným názvem.

---

## 19. Formativní kvíz

1. **Co znamená FTPS?**  
   FTP zabezpečené pomocí TLS.

2. **Je SFTP totéž co FTPS?**  
   Ne. SFTP je jiný protokol nad SSH.

3. **Co dělá `mirror --reverse`?**  
   Zrcadlí místní adresář na vzdálený server.

4. **Proč nepíšeme heslo do příkazu?**  
   Mohlo by zůstat v historii nebo být viditelné dalším procesům či uživatelům.

5. **Co je dokumentový kořen?**  
   Adresář, ze kterého webový server poskytuje soubory webu.

6. **Proč nasazujeme nejdříve do `example.cz-new`?**  
   Abychom nepoškodili běžící web neúplným přenosem.

7. **Co znamená HTTP 403?**  
   Server požadavek pochopil, ale přístup zakázal.

8. **Je přejmenovaná WordPress složka úplná záloha?**  
   Ne. Je na stejném serveru a WordPress navíc používá databázi.

---

## 20. Exit ticket

Každý žák dokončí dvě věty:

1. „Největší rozdíl mezi FTP a FTPS je…“
2. „Před přepnutím nového webu musím vždy…“

Možná správná odpověď:

> „FTPS šifruje spojení pomocí TLS. Před přepnutím musím ověřit správné cesty, mít plán návratu a nejdříve nahrát web do vedlejší složky.“

---

## 21. Hodnocení praktické práce

| Kritérium | 0 bodů | 1 bod | 2 body |
|---|---|---|---|
| Rozlišení místní/vzdálené cesty | nerozliší | rozliší s pomocí | rozliší samostatně |
| Volba protokolu | volí nešifrované FTP | ví, že je třeba šifrování | správně vysvětlí FTPS/SFTP |
| Bezpečnost hesla | vloží heslo do příkazu | heslo skryje | vysvětlí riziko historie a procesů |
| Postup nasazení | přepisuje ostrý web | použije mezisložku | navrhne i rollback |
| Kontrola výsledku | nekontroluje | zkontroluje soubory | zkontroluje FTP i HTTP stav |

Maximum: 10 bodů.

---

## 22. Nejčastější miskoncepce

### „FTP je program.“

FTP je protokol. `lftp`, `curl`, FileZilla nebo WebFTP jsou klientské programy, které jej používají.

### „FTPS a SFTP jsou dvě jména pro totéž.“

Nejsou. FTPS rozšiřuje FTP o TLS; SFTP patří do rodiny SSH.

### „Když se soubor přenesl, web funguje.“

Nemusí. Soubor může být ve špatné složce, mít nevhodná oprávnění nebo obsahovat chybnou konfiguraci.

### „Přejmenovaná složka je záloha.“

Je to rychlá možnost návratu, nikoli nezávislá záloha.

### „`--delete` uklidí jen nepotřebné soubory.“

Odstraní vše v cíli, co podle zvoleného zdroje nemá existovat. Při chybné cestě může smazat důležitá data.

### „Když při psaní hesla nic nevidím, klávesnice nefunguje.“

Terminál znaky z bezpečnostních důvodů nezobrazuje.

---

## 23. Diferenciace výuky

### Podpora pro začátečníky

- barevně označit lokální a vzdálené cesty;
- nejprve pracovat jen s `pwd`, `lpwd`, `cls` a `put`;
- dát žákům rozstříhané části příkazu k seřazení;
- použít testovací soubory bez podsložek.

### Rozšíření pro pokročilé

- porovnat FTPS se SFTP a SCP;
- vysvětlit DNS, TLS certifikát a HTTP redirect;
- doplnit kontrolní součty souborů;
- navrhnout automatizované nasazení přes CI/CD;
- diskutovat nulový výpadek, verzování release adresářů a symbolické odkazy;
- analyzovat rozdíl mezi soubory aplikace a stavovými daty v databázi.

---

## 24. Domácí úkol

Navrhněte bezpečný postup aktualizace školního webu. Výstup musí obsahovat:

1. přípravu místních souborů;
2. volbu šifrovaného protokolu;
3. ověření cílové cesty;
4. zálohu;
5. nahrání do mezisložky;
6. přepnutí;
7. HTTP kontrolu;
8. rollback.

Rozsah: jedna strana nebo jednoduché blokové schéma.

---

## 25. Tahák pro žáka

```text
FTP       protokol pro přenos souborů; bez TLS není bezpečný
FTPS      FTP zabezpečené pomocí TLS
SFTP      jiný protokol, běží nad SSH
upload    počítač → server
download  server → počítač
pwd       vzdálený adresář
lpwd      místní adresář
cls -la   výpis vzdálených položek včetně skrytých
get       stáhnout soubor
put       nahrát soubor
mirror    server → počítač
mirror -R počítač → server
mv        vzdáleně přesunout nebo přejmenovat
403       přístup zakázán
404       nenalezeno
500       chyba serveru nebo konfigurace
```

### Bezpečné minimum

```text
1. Ověř zdroj a cíl.
2. Vyžaduj TLS.
3. Heslo nedávej do příkazu.
4. Nepoužívej --delete bez přesného porozumění.
5. Nahraj nejdřív do vedlejší složky.
6. Zachovej cestu zpět.
7. Po FTP vždy otestuj web přes HTTP/HTTPS.
```

---

## 26. Poznámky pro učitele

- Skutečné přihlašovací údaje nepromítejte a nevkládejte do pracovního listu.
- Produkční heslo nikdy nenechávejte v historii terminálu.
- Vysvětlete, že zpětné lomítko před znakem v kopírovaném textu může pocházet z formátování. Běžnou adresu píšeme jako `ftp://server/`, nikoli jako `ftp\://server/`.
- Před každou změnovou operací nechte žáky nahlas určit zdroj, cíl a směr.
- Rozlišujte „přenos proběhl“ a „aplikace funguje“.
- Mazání na serveru demonstrujte pouze na testovacích datech.
- Pokud hodina probíhá bez serveru, lze všechny kroky simulovat dvěma místními složkami označenými „počítač“ a „server“.

---

## 27. Odborné zdroje

1. [RFC 959 – File Transfer Protocol](https://www.rfc-editor.org/info/rfc959/) – základní specifikace FTP.
2. [RFC 4217 – Securing FTP with TLS](https://www.rfc-editor.org/info/rfc4217/) – zabezpečení FTP pomocí TLS.
3. [`lftp` manual](https://manpages.debian.org/trixie/lftp/lftp.1.en.html) – syntaxe klienta a příkazu `mirror`.
4. [WEDOS: Webhosting – Správa souborů](https://kb.vedos.cz/webhosting-soubory/) – adresářová struktura a práce se soubory.
5. [WEDOS: Webhosting – Zálohování](https://kb.vedos.cz/webhosting-zalohovani/) – záloha souborů a databáze.

---

## 28. Jednověté shrnutí hodiny

> Bezpečné nasazení webu znamená šifrovaně přenést správné místní soubory do ověřené vzdálené mezisložky, zachovat možnost návratu, přepnout web přejmenováním a výsledek zkontrolovat přes HTTPS.
