# Registračný flow Azet.sk — natívna registrácia

> Tento dokument popisuje natívny proces registrácie nového používateľa na Azet.sk (registrácia emailom + heslom). Sociálne prihlásenia (Google, Apple) a firemné kontá sú mimo rozsah tohto dokumentu a budú dokumentované samostatne.

---

## 1. Cieľ a rozsah

**Cieľ:** Umožniť novému používateľovi vytvoriť si osobný účet na Azet.sk prostredníctvom natívnej registrácie (email + heslo) s overením identity cez email / SMS.

**V rozsahu:**

* Natívna registrácia emailom a heslom
* Overenie emailu a telefónneho čísla
* Zadanie základných profilových údajov (meno, dátum narodenia, pohlavie, prezývka)
* Akceptácia podmienok používania a spracovania osobných údajov
* Flow pre „zabudnuté heslo" / obnovu prístupu

**Mimo rozsah (rieši sa samostatne):**

* Prihlásenie cez Google
* Prihlásenie cez Apple
* Firemné kontá

---

## 2. Vstupné body (Entry points)

Používateľ sa dostane do registračného flow z týchto miest:

* CTA „Registrovať sa" na úvodnej obrazovke / login screene
* Redirect z gated obsahu (akcia, ktorá vyžaduje prihlásenie)
* Deeplink z marketingovej kampane alebo emailu

---

## 3. Prehľad flow (High-level)

```
[Entry] → [Výber typu účtu] → [Zadanie emailu] → [Zadanie hesla]
   → [Telefónne číslo] → [SMS overenie] → [Meno + Prezývka]
   → [Dátum narodenia] → [Pohlavie] → [Súhlasy s podmienkami]
   → [Overenie emailu] → [Úspešná registrácia → Profil]
```

Každý krok má svoju chybovú vetvu (validácia, už existujúci účet, nesprávny kód a pod.) a možnosť vrátiť sa naspäť.

---

## 4. Kroky registrácie (detailný popis)

### 4.1 Výber typu účtu

* Obrazovka, kde si používateľ vyberie, či si chce vytvoriť **osobný účet**.
* Firemný účet je v tomto momente skrytý / preskočený.
* CTA: „Pokračovať" → Zadanie emailu.

### ~~4.2 Zadanie emailu~~

* ~~Input: Email~~
* ~~Validácia:~~
    * ~~Formát emailu (RFC 5322, zjednodušená kontrola)~~
    * ~~Email ešte nie je registrovaný (kontrola voči DB)~~
    * ~~Povinné pole~~
* ~~Error stavy:~~
    * ~~„Zadaj platný email"~~
    * ~~„Na tento email už existuje účet" + CTA „Prihlásiť sa" / „Zabudol som heslo"~~

### 4.3 Zadanie hesla

* Input: Heslo (s toggle „zobraziť / skryť")
* Požiadavky na heslo:
    * Min. 8 znakov
    * Aspoň 1 veľké písmeno
    * Aspoň 1 číslica
    * Aspoň 1 špeciálny znak (odporúčané, nie povinné)
* Vizuálny indikátor sily hesla (slabé / stredné / silné)
* Validácia v reálnom čase pri písaní

### 4.4 Telefónne číslo

* Input: Predvoľba krajiny (default +421) + číslo
* Validácia formátu
* Kontrola, či číslo nie je už priradené k inému účtu (v závislosti od politiky)
* CTA: „Poslať overovací kód"

### 4.5 SMS overenie

* Zadanie 4/6-miestneho kódu doručeného SMS
* Automatické prečítanie SMS kódu (Android SMS Retriever API / iOS autofill)
* CTA „Poslať kód znova" — aktívne po 30–60 s (cooldown)
* Limit počtu pokusov (napr. 5) → dočasný lockout

### 4.6 Meno a prezývka

* Inputy:
    * Krstné meno (povinné)
    * Priezvisko (povinné)
    * Prezývka / nickname (voliteľné alebo povinné podľa produktového rozhodnutia)
* Validácia prezývky:
    * Min. dĺžka (napr. 3 znaky)
    * Povolené znaky (alfanumerické + `_`, `-`)
    * Unikátnosť (kontrola voči DB)
    * Blacklist (vulgarizmy, rezervované slová, admin názvy)

### 4.7 Dátum narodenia

* UI: Kalendár / date picker
* Validácia:
    * Minimálny vek (napr. 16 alebo 18 rokov podľa GDPR a produktových pravidiel)
    * Maximálny vek (sanity check, napr. 120 rokov)
* Hint text: „Potrebujeme na overenie veku, nebude verejne zobrazený."

### 4.8 Pohlavie

* Výber: Muž / Žena / Iné / Nechcem uviesť
* Voliteľné pole (odporúčanie — nenútiť používateľa)

### 4.9 Súhlasy a podmienky

* Checkboxy:
    * [ ] Súhlasím s **Podmienkami používania** (povinné, link na plné znenie)
    * [ ] Súhlasím so **spracovaním osobných údajov** (povinné, link na privacy policy)
    * [ ] Chcem dostávať novinky a marketingové emaily (voliteľné, opt-in)
* CTA „Registrovať sa" je disabled, kým nie sú odfajknuté povinné súhlasy.

### 4.10 Overenie emailu

* Po dokončení registrácie systém pošle email s overovacím linkom.
* Obrazovka: „Poslali sme ti email na adresu `xxx@xxx.sk`. Klikni na link a aktivuj si účet."
* CTA „Poslať email znova" (cooldown 60 s)
* Pri rozkliknutí linku v emaile → aktivácia účtu → redirect na success screen.

### 4.11 Úspešná registrácia

* Potvrdzovacia obrazovka: „Účet úspešne vytvorený."
* CTA „Prejsť na profil" / „Začať používať Azet.sk"
* Používateľ je automaticky prihlásený (session token).

---

## 5. Validácie — súhrn

| Pole | Pravidlo | Chybová hláška |
| --- | --- | --- |
| Email | Platný formát, neexistuje v DB | „Zadaj platný email" / „Účet už existuje" |
| Heslo | Min. 8 znakov, veľké písmeno, číslica | „Heslo musí mať aspoň 8 znakov…" |
| Telefón | Platný formát (E.164) | „Zadaj platné telefónne číslo" |
| SMS kód | 4–6 číslic, nie expirovaný | „Nesprávny kód" / „Kód expiroval" |
| Dátum narodenia | Min. vek 16 rokov | „Na registráciu musíš mať aspoň 16 rokov" |
| Prezývka | Unikátna, povolené znaky | „Prezývka je obsadená" |

---

## 6. Error stavy a edge cases

**Sieťové problémy**

* Výpadok siete počas SMS overenia → zachovať zadaný kód, retry
* Timeout requestu → hláška „Niečo sa pokazilo, skús znova"

**Bezpečnostné limity**

* Max. 5 pokusov o zadanie SMS kódu → lockout na 15 min
* Rate-limiting na odoslanie SMS / emailu (napr. max 3x za hodinu na rovnaké číslo/email)
* Captcha / bot detection pri podozrivej aktivite

**Duplicita účtu**

* Email už existuje → ponúknuť prihlásenie alebo reset hesla
* Telefón priradený k inému účtu → buď blokovať, alebo ponúknuť merge (produktové rozhodnutie)

**Prerušenie registrácie**

* Používateľ opustí flow v polovici → uchovať progress (session / draft) aspoň 24 h, umožniť návrat
* Nedokončený účet (napr. neoverený email) → po X dňoch vymazať, resp. notifikovať

---

## 7. Zabudnuté heslo (Forgot password flow)

1. Používateľ klikne „Zabudol som heslo" na login screene.
2. Zadá email.
3. Systém pošle email s reset linkom (platnosť 60 min).
4. Používateľ klikne na link → zadá nové heslo (2x pre kontrolu).
5. Validácia hesla rovnaká ako pri registrácii.
6. Potvrdenie zmeny → automatické prihlásenie → redirect na profil.

---

## 8. Otvorené otázky (Open questions)

* Minimálny vek — 16 alebo 18 rokov?
* Je telefónne číslo povinné alebo voliteľné?
* Chceme overovať email **aj** telefón, alebo stačí jedno z toho?
* Ako sa správame pri neoverenom účte — má používateľ prístup k obmedzenej funkčnosti, alebo je úplne zablokovaný?
* Chceme zachovať možnosť zmeny prezývky neskôr v profile?
* Lockout politika — koľko pokusov, ako dlho?
* Captcha — od začiatku, alebo až pri podozrivej aktivite?

---

## 9. Success metriky (návrh)

* **Conversion rate** — % používateľov, ktorí dokončili registráciu z tých, čo ju začali
* **Drop-off per step** — na ktorom kroku najviac ľudí odchádza (identifikácia úzkych miest)
* **Time to complete** — medián času na dokončenie registrácie
* **SMS delivery rate / email delivery rate** — úspešnosť doručenia overovacích správ
* ~~**Email verification rate**~~ ~~— % aktivovaných účtov z registrovaných~~
* **Forgot password rate** — % používateľov, čo do 7 dní od registrácie požiadajú o reset hesla (indikátor problému s onboardingom)

---

## 10. Prílohy

* Flow diagram (zdroj: Figma / priložený obrázok)
* Wireframy jednotlivých obrazoviek (TBD)
* Copy deck — texty hlášok, CTA, error messages (TBD)
* Prepojenie na existujúce komponenty v design systéme (TBD)

---

## Majka poznámky — Porovnanie Azet účet vs. Google účet

| **Obrazovka / účel** | **Azet účet (email + heslo)** | **Google účet** |
| --- | --- | --- |
| **1** Výber spôsobu registrácie | **Nadpis:** Ešte nemáte svoj Azet účet? **Akcie:** Vytvoriť Azet účet / Pokračovať s Google | Identická obrazovka |
| **2** Vytvorenie identity | **Nadpis:** Vytvorte si Azet účet **Polia:** Emailová adresa / meno účtu (`@azet.sk`) + Heslo **Helper:** Táto adresa bude Vašou oficiálnou emailovou adresou na Azete. V nastaveniach Vášho účtu si môžete zvoliť aj prezývku, ktorá sa bude zobrazovať pri používaní ostatných služieb Azet.sk. **CTA:** Pokračovať | Google autentifikácia (systémový Google screen, bez UI Azetu) |
| **3** Emailová identita | — *(email je už zadaný)* | **Nadpis:** Vyberte si svoju emailovú adresu na Azete **Pole:** `__________@azet.sk` **Helper:** Táto adresa bude Vašou oficiálnou emailovou adresou na Azete. **CTA:** Pokračovať |
| **4** Typ účtu | **Nadpis:** Na čo budete účet používať? **Voľby:** Súkromný účet / Firemný účet **CTA:** Pokračovať | Identická obrazovka |
| **5** Osobné údaje | **Zobrazí sa len pri súkromnom účte** **Nadpis:** O Vás **Polia:** Oslovenie (Pán / Pani) + Dátum narodenia + Bydlisko + Prezývka (nepovinné) **CTA:** Pokračovať | Identická obrazovka a logika |
| **6** Zabezpečenie účtu | **Nadpis:** Zabezpečte si účet **Možnosti:** Zabezpečenie tel. číslom / Zabezpečenie e-mailom **Povinné CTA:** Pokračovať | **Nadpis:** Odporúčame zabezpečiť Váš účet **Možnosti:** Zabezpečenie tel. číslom / Zabezpečenie e-mailom **Nepovinné CTA:** Zabezpečiť účet / Preskočiť |
| **7** Súhlasy | ☑ Podmienky používania (povinné) + Spracovanie osobných údajov + 16 (povinné) ☐ Marketing (nepovinné) **CTA:** Vytvoriť účet | Identická obrazovka |
| **8** Dokončenie | ✔️ Váš účet bol úspešne vytvorený! **CTA:** Pokračovať na Azet | ✔️ Váš účet bol úspešne vytvorený! **CTA:** Pokračovať na Azet |

**Dôležité:**

* Google slúži výhradne ako spôsob prihlásenia, nie ako definícia emailovej adresy
* Emailová adresa na Azete sa vždy volí vedome používateľom
* Typ účtu rozhoduje o rozsahu zbieraných údajov
* Osobné údaje sa zbierajú iba pre súkromné účty
* Zabezpečenie účtu je povinné pri Azet účte a odporúčané pri Google logine
* Marketingový súhlas je vždy nepovinný

**Otázky:**

* Je potrebná aktivácia služby Pokec?
