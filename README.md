# Lab Writeup: Reflected XSS protected by CSP, with CSP bypass

- Tento dokument slouží jako studijní materiál a záznam z PortSwigger Web Security Academy
- **Lab Name:** Reflected XSS protected by CSP, with CSP bypass

# Popis útoku
Aplikace byla chráněna pomocí Content Security Policy (CSP), která běžně blokuje spouštění skriptů (např. klasických `<script>alert(1)</script>`). Cílem bylo najít slabinu v konfiguraci CSP nebo v parametrech logování/chybových hlášení a CSP obejít.

# Proč to fungovalo?
Využitím funkce Burp Suite (Intercept) jsem zachytil HTTP požadavek, podstrčil vlastní XSS payload (`<script>alert(1)</script>`) do vyhledávacího pole a odeslal ho serveru přes funkci **Forward**. Aplikace následně v chybové hlášce nebo direktivě CSP (obsahující parametr `token` a `unsafe-inline`) reflektovala direktivu zpět. Úpravou URL adresy a přidáním správného tokenu se mi podařilo CSP ochranu zneškodnit a spustit skript v kontextu prohlížeče.

## Použitý payload:
> **Pozor:** Místo `YOUR-LAB-ID` si v URL adrese dosaď své vlastní ID aktivního labu z PortSwiggeru.

```text
https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cscript%3Ealert%281%29%3C%2Fscript%3E&token=;script-src-elem%20%27unsafe-inline%27


# Můj postup řesení:

Zachycení požadavku (Intercept): Do hlavního vyhledávacího pole jsem napsal payload <script>alert(1)</script> Než jsem stiskl Enter, zapnul jsem v Burp Suite zachytávání provozu pomocí Interceptu (pozastavení požadavku)

Odeslání do Repeateru (Ctrl + R): V Burp Suite jsem zachycený požadavek označil a pomocí zkratky Ctrl + R jsem ho odeslal do Repeateru k další analýze.

Nalezení tokenu v Response: Na druhé straně (v odpovědi serveru / Response) jsem se podíval do třetího řádku a v chybové hlášce jsem uviděl hodnotu začínající na ?token=, která prozrazovala nastavení CSP.

Úprava URL a bypass: V  URL adrese jsem promazal přebytečnou část, nechal jen základ a dopsal jsem tam získaný token s direktivou pro povolení skriptů (&token=;script-src-elem %27unsafe-inline%27)

Provedení skriptu a výhra: Po obnovení stránky (nebo odeslání) se na mě ukázalo vyskakovací okno s alertem, stránka mě hodila zpět a lab byl úspěšně vyřešen.
