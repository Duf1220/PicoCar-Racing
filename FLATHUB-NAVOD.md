# PicoCar Racing → Flathub: návod krok za krokem

Stav k 13. 10. 2026. Hra na SD kartě je Linux x86_64 build z **Unity 2022.3.3f1**, ~780 MB.

## 0. Co je potřeba rozhodnout / dodat (bez toho to nejde odeslat)

| Položka | Kde doplnit | Poznámka |
|---|---|---|
| GitHub uživatelské jméno (místo `Duf1220`) | názvy souborů + obsah všech souborů | Flathub ID `io.github.<user>.PicoCarRacing` musí odpovídat reálnému účtu. Mělo by být bez pomlček a teček. |
| Název repozitáře (místo `PicoCar-Racing`) | manifest, metainfo | Repo, kde bude Release s tar.gz a screenshoty. |
| Ikona 256×256 PNG (lépe SVG) | `io.github.Duf1220.PicoCar-Racing.png` | Ve hře je jen výchozí Unity ikona (modré šipky), tu Flathub nepřijme. |
| Screenshoty ze hry | metainfo `<screenshots>` | Aspoň 1, lépe 3, PNG 16:9, bez rámu okna. |
| Licence | metainfo `<project_license>` | Na GitHubu je GPL-3.0 -> `GPL-3.0-only`. |
| Kontaktní e-mail | metainfo `<update_contact>` | |
| OARS hodnocení obsahu | metainfo `<content_rating>` | Vygenerovat na https://hughsie.github.io/oars/generate.html |
| Síť ano/ne | manifest `--share=network` | Viz bod 2. |

## 1. Bezpečnost: CVE-2025-59489 (rebuild je nutný)

Build 2022.3.3f1 je touto chybou zasažen. **Unity Application Patcher Linux buildy neumí** (patchuje jen Windows, macOS a Android; na Linuxu jen běží). Unity pro Linux doporučuje jediné: **rebuild v opraveném editoru**.

1. V Unity Hubu nainstalujte nejnovější **2022.3.x LTS** (oprava je od 2022.3.67f1, ověřte v seznamu na https://unity.com/security/sept-2025-01).
2. Otevřete projekt, při té příležitosti opravte v *Project Settings → Player*:
   - **Company Name** a **Product Name** (teď je tam `DefaultCompany` / `My project`, podle toho se jmenuje složka s uloženými daty `~/.config/unity3d/DefaultCompany/My project`).
   - **Default Icon** (vlastní ikona).
   - Zvažte odebrání balíčků **Unity Analytics, IAP (Purchasing) a Advertisements**, pokud je hra na desktopu nepoužívá. Zmenší to build a vyhnete se dotazům recenzentů na sběr dat.
3. Build: *File → Build Settings → Linux, x86_64*, **vypněte** „Development Build“ a nekopírujte soubory `*_s.debug` (ladicí symboly, do distribuce nepatří).
4. Nový build zabalte (bod 3) – hash v manifestu se musí přepočítat.

## 2. Testování: funguje hra bez sítě?

Hra obsahuje knihovny Unity Services (Analytics 4.4.0, Purchasing 4.9.3, Advertisements). Flathub chce co nejméně oprávnění, proto je v manifestu `--share=network` zakomentované.

- Spusťte Flatpak bez sítě. Pokud hra normálně běží (Unity Services jen zaloguje chybu), síť nedávejte.
- Pokud hra bez sítě padá nebo nejde hrát, odkomentujte `--share=network` a do popisu v metainfo přidejte větu, k čemu se síť používá.

## 3. Zabalení hry do tar.gz

Pro build z SD karty je hotovo: `release/PicoCarRacing-1.0.0-linux-x86_64.tar.gz`, hash v `release/SHA256SUMS`.

Po rebuildu zabalte znovu. Na Linuxu (zachová příznak spustitelnosti):

```bash
tar --transform 's,^,PicoCarRacing-1.0.0/,' -czf PicoCarRacing-1.0.0-linux-x86_64.tar.gz "PicoCar Racing.x86_64" "PicoCar Racing_Data" UnityPlayer.so
```

Na Windows v Git Bash (příznak se ztratí, manifest to řeší příkazem `chmod`):

```bash
tar --force-local --transform 's,^,PicoCarRacing-1.0.0/,' -czf PicoCarRacing-1.0.0-linux-x86_64.tar.gz "PicoCar Racing.x86_64" "PicoCar Racing_Data" UnityPlayer.so
```

Hash:

```bash
sha256sum PicoCarRacing-1.0.0-linux-x86_64.tar.gz
```

## 4. Hosting souborů na GitHubu

Flathub si zdroje stahuje sám z pevné URL, SD kartu ani lokální soubory použít nelze.

1. Založte repozitář `https://github.com/Duf1220/PicoCar-Racing` (může být soukromý jen pokud je Release veřejný, jednodušší je veřejný).
2. Nahrajte screenshoty do `screenshots/1.png` atd. (odkazy v metainfo jsou `raw.githubusercontent.com/.../main/screenshots/1.png`).
3. Vytvořte **Release** s tagem `v1.0.0` a jako asset nahrajte `PicoCarRacing-1.0.0-linux-x86_64.tar.gz` (limit GitHubu je 2 GB na soubor).
4. Do manifestu dejte URL assetu a `sha256`.
5. Volitelně nahrajte do repozitáře složku `io.github.Duf1220.PicoCar-Racing/` a `.github/workflows/flatpak.yml` – GitHub Actions pak build a lint udělá za vás (nemáte doma Linux ani WSL).

## 5. Lokální build a test (potřebuje Linux)

Na tomto PC není Linux, WSL ani flatpak-builder. Možnosti: Linux počítač / live USB, nebo WSL2 (`wsl --install -d Ubuntu`), nebo CI workflow z bodu 4.

```bash
flatpak install -y flathub org.flatpak.Builder org.freedesktop.Platform//26.08 org.freedesktop.Sdk//26.08
cd io.github.Duf1220.PicoCar-Racing
flatpak run org.flatpak.Builder --force-clean --sandbox --user --install --install-deps-from=flathub --ccache --mirror-screenshots-url=https://dl.flathub.org/media/ --repo=repo builddir io.github.Duf1220.PicoCar-Racing.yml
flatpak run io.github.Duf1220.PicoCar-Racing
```

Lint (musí projít bez chyb):

```bash
flatpak run --command=flatpak-builder-lint org.flatpak.Builder manifest io.github.Duf1220.PicoCar-Racing.yml
flatpak run --command=flatpak-builder-lint org.flatpak.Builder repo repo
```

Co zkontrolovat ve hře: zvuk, celá obrazovka, gamepad, ukládání (data budou v `~/.var/app/io.github.Duf1220.PicoCar-Racing/config/unity3d/`), chování bez sítě. Log Unity: `~/.var/app/io.github.Duf1220.PicoCar-Racing/config/unity3d/<Company>/<Product>/Player.log`.

## 6. Odeslání na Flathub

1. GitHub účet se zapnutým **2FA** (Flathub to vyžaduje pro udělení přístupu k repu).
2. Forkněte `https://github.com/flathub/flathub` (odškrtněte „Copy the master branch only“).
3. Klonujte větev `new-pr` a vytvořte vlastní větev:

```bash
git clone --branch=new-pr https://github.com/Duf1220/flathub.git && cd flathub && git checkout -b picocar-racing new-pr
```

4. Zkopírujte do kořene forku **obsah** složky `io.github.Duf1220.PicoCar-Racing/` (manifest, desktop, metainfo, PNG, `picocar-racing`, `flathub.json`). Šablonu z větve `new-pr` nemažte.
5. Commit, push, a na GitHubu otevřete PR proti větvi **`new-pr`** (ne `master`) s názvem `Add io.github.Duf1220.PicoCar-Racing`.
6. V PR napište, že jste autor hry (proto není třeba `extra-data`), že je uzavřená, a zda a proč potřebuje síť. Až recenzent odsouhlasí, napište do PR komentář `bot, build`.
7. Po schválení dostanete pozvánku do repa `flathub/io.github.Duf1220.PicoCar-Racing` – přijměte do týdne. Další verze se vydávají úpravou manifestu v tomto repu (nová URL + sha256 + nový `<release>` v metainfo).

## 7. Co je v této složce

- `io.github.Duf1220.PicoCar-Racing/io.github.Duf1220.PicoCar-Racing.yml` – manifest (runtime freedesktop 26.08, X11, PulseAudio, dri, input).
- `io.github.Duf1220.PicoCar-Racing/picocar-racing` – spouštěcí skript.
- `io.github.Duf1220.PicoCar-Racing/io.github.Duf1220.PicoCar-Racing.desktop` – upravený desktop soubor (klíčová slova, `StartupWMClass`).
- `io.github.Duf1220.PicoCar-Racing/io.github.Duf1220.PicoCar-Racing.metainfo.xml` – doplněné metainfo (licence, kontakt, ovládání, branding, CZ překlady).
- `io.github.Duf1220.PicoCar-Racing/flathub.json` – jen x86_64 (Unity build nemá aarch64).
- `.github/workflows/flatpak.yml` – CI build + lint na GitHub Actions.
- `../release/PicoCarRacing-1.0.0-linux-x86_64.tar.gz` + `SHA256SUMS` – zabalený build z SD karty (bez ladicích symbolů).
