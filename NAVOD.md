# BadCars – hosting a záloha (OneDrive, Google Drive, soubor)

Aplikace je statický web (žádný server). Data jsou v zařízení a jen se zálohují:
- **OneDrive** – soubor `Apps/BadCars/badcars-backup.json` (aplikace vidí jen svou složku),
- **Google Drive** – skrytá data aplikace na Disku Google (v Disku je neuvidíš, aplikace nevidí nic jiného),
- **ruční záloha do souboru** – bez registrace, přes Sdílet/Stáhnout (iCloud, Soubory, Disk, e-mail…).

Stačí nastavit jedno úložiště (kroky 2 nebo 2b), nebo žádné a zálohovat jen do souboru.
Zálohuje se vždy do jednoho připojeného úložiště, přepnout jde v Nastavení.

## 1. Hosting na GitHub Pages (zdarma)
1. Na github.com vytvoř repozitář (např. `badcars`, stačí public – v kódu nejsou žádná data).
2. Nahraj do něj všechny soubory z této složky (index.html, config.js, sw.js, manifest.webmanifest, 3× icon-*.png).
3. Settings → Pages → Source: *Deploy from a branch*, větev `main`, složka `/ (root)` → Save.
4. Za minutu bude web na `https://TVOJE-JMENO.github.io/badcars/` (včetně lomítka na konci).

## 2. Registrace aplikace v Microsoft Entra (jednorázově)
1. Otevři https://entra.microsoft.com (nebo portal.azure.com) a přihlas se účtem Microsoft.
   (Když to chce adresář/tenant, vytvoř zdarma, stačí výchozí.)
2. Identity → Applications → **App registrations** → **New registration**.
3. Name: `BadCars`.
   Supported account types: **Accounts in any organizational directory and personal Microsoft accounts**.
   Redirect URI: platforma **Single-page application (SPA)**, hodnota přesně adresa z kroku 1,
   např. `https://TVOJE-JMENO.github.io/badcars/` → **Register**.
4. Na stránce aplikace zkopíruj **Application (client) ID**.
5. API permissions → Add a permission → Microsoft Graph → **Delegated permissions** →
   zaškrtni `Files.ReadWrite.AppFolder` a `offline_access` (User.Read tam už je) → Add.
   (Souhlas udělíš při prvním přihlášení v aplikaci, správce nepotřebuješ.)

## 2b. Registrace pro Google Drive (jednorázově, volitelné)
1. Otevři https://console.cloud.google.com a vytvoř nový projekt `BadCars`.
2. **APIs & Services → Library** → vyhledej **Google Drive API** → **Enable**.
3. **Google Auth Platform** (dříve OAuth consent screen) → Get started:
   název aplikace `BadCars`, e-mail podpory, Audience **External**, kontaktní e-mail → Create.
4. **Data Access** → Add or remove scopes → přidej `.../auth/drive.appdata` → Update → Save.
5. **Audience → Test users** → přidej svůj účet Google (a účty lidí, kterým aplikaci pošleš; v testovacím režimu až 100).
6. **Clients → Create client** → typ **Web application**:
   - Authorized JavaScript origins: `https://TVOJE-JMENO.github.io`
   - Authorized redirect URIs: přesně adresa aplikace, např. `https://TVOJE-JMENO.github.io/badcars/`
   → Create a zkopíruj **Client ID** (končí na `.apps.googleusercontent.com`).

## 3. Vložení client ID
V repozitáři otevři `config.js` a vlož zkopírované ID: OneDrive do `clientId: ''`, Google do `googleClientId: ''`. Ulož (commit).
Po změně počkej minutu, než se web přenasadí.

## 4. První spuštění
1. Otevři adresu na telefonu a **přidej na plochu** (Safari: Sdílet → Přidat na plochu; Chrome: Instalovat aplikaci).
   Na iPhonu má aplikace z plochy **vlastní úložiště**, odděleně od Safari. Přihlašuj se a importuj data až v ní.
2. Nastavení (⚙) → **Připojit OneDrive** nebo **Připojit Google Drive** → přihlas se a potvrď oprávnění.
3. Data z dosavadní aplikace přenes: ve staré aplikaci Nastavení → Export JSON → Zkopírovat,
   v nové Nastavení → Import JSON → vložit → Sloučit (nebo **Obnovit ze souboru**). Hned se zazálohují do cloudu.

## Jak to funguje
- Hlavní jsou lokální data. Záloha po každé změně doplňuje soubor na OneDrive (nikdy ho nepřepíše starším stavem).
- Na novém zařízení po přihlášení nabídne „Obnovit ze zálohy“.
- Přihlášení nevydrží napořád: Microsoft u webových aplikací zhruba 24 hodin, Google asi 1 hodinu.
  Aplikace pak ukáže nahoře „Přihlášení vypršelo“ a jedno klepnutí na Přihlásit to obnoví
  (obvykle bez hesla). Změny do té doby zůstávají v zařízení a zazálohují se po přihlášení.
- Ruční záloha: Nastavení → **Uložit zálohu do souboru**. Na telefonu se otevře Sdílet (Uložit do Souborů,
  iCloud, Disk Google, e-mail), na PC se soubor stáhne. Obnovení: **Obnovit ze souboru** – data se sloučí.
- Offline: po prvním načtení běží aplikace i bez internetu (service worker). Při nahrání nové verze
  zvyš v `sw.js` číslo `CACHE = 'badcars-v2'`, ať si zařízení stáhnou nové soubory.

## Když něco nejde
- `AADSTS50011` (redirect URI): adresa v registraci se musí přesně shodovat s adresou aplikace (včetně `/` na konci).
- `AADSTS700016` / neplatný client ID: zkontroluj `config.js`.
- Google `redirect_uri_mismatch`: adresa v Authorized redirect URIs se musí přesně shodovat (včetně `/`).
- Google `access_denied` / „aplikace není ověřená“: přidej účet mezi Test users (krok 2b.5).
- Přihlášení se zacyklí nebo je „neplatný stav“: smaž data webu pro tuto adresu (jen přihlašovací stav) a zkus znovu.
