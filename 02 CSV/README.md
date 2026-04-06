# Harjutus: IT-seadmete inventuuri analüüs CSV-ga

**Kursus:** KIT-24  
**Õpetaja:** Toivo Pärnpuu  
**Moodul:** `csv`, `datetime` (tulevad Pythoniga kaasa, pole vaja installida)

---

## Eesmärk

Kujuta ette, et oled IT-administraator ja pead üle vaatama organisatsiooni seadmete inventuuri. Sul on CSV-fail 15 seadmega. Sinu ülesanne on Pythoniga leida seadmed, mis vajavad tähelepanu — vana tarkvara, täis ketas, lõppenud garantii.

---

## Ettevalmistus

1. Klooni repositoorium oma arvutisse:

```bash
git clone git@github.com:Tallinna-Polutehnikum/skriptimisvahendid.git
cd skriptimisvahendid
```

2. Ava kaust `csv-harjutus` — seal on fail `seadmed.csv`.

3. Loo samasse kausta uus fail nimega `seadmete_analyys.py`.

---

## Andmestik

Fail `seadmed.csv` sisaldab 15 rida järgmiste veergudega:

| Veerg | Näide | Tähendus |
|---|---|---|
| `seadme_id` | PC-001 | unikaalne ID |
| `nimi` | RIKU-LAPTOP | seadme nimi |
| `tüüp` | laptop / desktop / server | seadme liik |
| `osakond` | IT, Müük, ... | kasutaja osakond |
| `os` | Windows 11 | operatsioonisüsteem |
| `viimane_uuendus` | 2024-11-03 | OS viimane uuendus |
| `kettaruum_gb` | 512 | ketta kogumaht GB |
| `kettaruum_vaba_gb` | 48 | vaba ruum GB |
| `garantii_lõpp` | 2026-03-15 | garantii lõppkuupäev |

---

## Kood samm-sammult

### 1. osa — loe CSV fail sisse

```python
import csv
from datetime import date

seadmed = []  # tühi nimekiri, kuhu salvestame kõik read

with open("seadmed.csv", newline="", encoding="utf-8") as f:
    lugeja = csv.DictReader(f)  # DictReader loeb iga rea sõnastikuna
    for rida in lugeja:
        seadmed.append(rida)    # lisa iga seade nimekirja

print(f"Kokku seadmeid andmebaasis: {len(seadmed)}")
print()

# Vaata, milline näeb välja üks rida
print("Näidis — esimene seade:")
print("-" * 40)
for väli, väärtus in seadmed[0].items():
    print(f"  {väli}: {väärtus}")
```

Käivita skript. Peaksid nägema `Kokku seadmeid andmebaasis: 15` ja kõiki esimese seadme välju.

---

### 2. osa — filtreerimine: leia probleemid

```python
tana = date.today()

vanad_uuendused  = []  # seadmed, mida pole üle aasta uuendatud
vähe_ruumi       = []  # seadmed, kus vaba kettaruum alla 10%
aegunud_garantii = []  # seadmed, mille garantii on lõppenud

for seade in seadmed:

    # Kontrolli viimast uuendust
    uuendus = date.fromisoformat(seade["viimane_uuendus"])
    paevi_tagasi = (tana - uuendus).days
    if paevi_tagasi > 365:
        vanad_uuendused.append((seade["nimi"], paevi_tagasi))

    # Kontrolli kettaruumi
    kokku = int(seade["kettaruum_gb"])
    vaba  = int(seade["kettaruum_vaba_gb"])
    protsent = vaba / kokku * 100
    if protsent < 10:
        vähe_ruumi.append((seade["nimi"], round(protsent, 1)))

    # Kontrolli garantiid
    garantii = date.fromisoformat(seade["garantii_lõpp"])
    if garantii < tana:
        aegunud_garantii.append((seade["nimi"], seade["garantii_lõpp"]))

print("Seadmed, mida pole üle aasta uuendatud:")
for nimi, paevi in vanad_uuendused:
    print(f"  {nimi} — {paevi} päeva tagasi")

print("\nSeadmed, kus vaba kettaruum alla 10%:")
for nimi, protsent in vähe_ruumi:
    print(f"  {nimi} — {protsent}% vaba")

print("\nSeadmed, mille garantii on lõppenud:")
for nimi, kuupäev in aegunud_garantii:
    print(f"  {nimi} — lõppes {kuupäev}")
```

---

### 3. osa — arvutused osakondade kaupa

```python
osakonnad = {}  # sõnastik: osakonna nimi → seadmete arv

for seade in seadmed:
    osakond = seade["osakond"]
    if osakond not in osakonnad:
        osakonnad[osakond] = 0
    osakonnad[osakond] += 1

print("\nSeadmete arv osakondade kaupa:")
for osakond, arv in sorted(osakonnad.items()):
    print(f"  {osakond}: {arv} seadet")

# Keskmine vaba kettaruum
vaba_ruumid = [int(s["kettaruum_vaba_gb"]) for s in seadmed]
keskmine_vaba = sum(vaba_ruumid) / len(vaba_ruumid)
print(f"\nKeskmine vaba kettaruum: {round(keskmine_vaba, 1)} GB")
```

---

### 4. osa — kirjuta aruanne uude CSV-faili

```python
with open("probleemseadmed.csv", "w", newline="", encoding="utf-8") as f:
    väljad = ["nimi", "probleem", "detail"]
    kirjutaja = csv.DictWriter(f, fieldnames=väljad)
    kirjutaja.writeheader()  # kirjuta päiserida

    for nimi, paevi in vanad_uuendused:
        kirjutaja.writerow({
            "nimi": nimi,
            "probleem": "vana uuendus",
            "detail": f"{paevi} päeva tagasi"
        })

    for nimi, protsent in vähe_ruumi:
        kirjutaja.writerow({
            "nimi": nimi,
            "probleem": "vähe kettaruumi",
            "detail": f"{protsent}% vaba"
        })

    for nimi, kuupäev in aegunud_garantii:
        kirjutaja.writerow({
            "nimi": nimi,
            "probleem": "garantii lõppenud",
            "detail": kuupäev
        })

print("Aruanne salvestatud faili: probleemseadmed.csv")
```

---

## Mida sa õppisid

| Käsk | Tähendus |
|---|---|
| `csv.DictReader(f)` | loe CSV ridu sõnastikena |
| `csv.DictWriter(f, fieldnames)` | kirjuta CSV sõnastikega |
| `kirjutaja.writeheader()` | kirjuta päiserida |
| `date.fromisoformat(str)` | teisenda tekst kuupäevaks |
| `(kuupäev1 - kuupäev2).days` | päevade vahe kahe kuupäeva vahel |
| `int(string)` | teisenda tekst täisarvuks |

---

## Lisaküsimused

1. Lisa filter: kuva ainult Windows 10 seadmed — neid tuleks uuendada Windows 11-le
2. Sorteeri `vähe_ruumi` nimekiri protsendi järgi kasvavalt — kasuta `sorted()` koos `key=` argumendiga
3. Arvuta, mitu seadet garantii lõpeb järgmise 6 kuu jooksul

---

## Tulemuse esitamine — Pull Request

### 1. Fork ja klooni repo

Mine [github.com/Tallinna-Polutehnikum/skriptimisvahendid](https://github.com/Tallinna-Polutehnikum/skriptimisvahendid), tee **Fork** ja klooni oma arvutisse.

### 2. Loo oma kaust

```bash
mkdir -p tulemused/Eesnimi_P
```

### 3. Kopeeri oma failid sinna

```bash
cp seadmete_analyys.py tulemused/Eesnimi_P/
cp probleemseadmed.csv tulemused/Eesnimi_P/
```

### 4. Commit ja push

```bash
git checkout -b harjutus-csv-Eesnimi
git add tulemused/
git commit -m "Lisa CSV harjutus — Eesnimi P"
git push -u origin harjutus-csv-Eesnimi
```

### 5. Ava Pull Request

Mine GitHubis oma fork'i lehele, klõpsa **Compare & pull request** ja loo PR pealkirjaga:

```
CSV harjutus — Eesnimi P
```

---

*Kui jooksed probleemi otsa, ava repositooriumis **Issue** ja kirjelda, mis juhtus.*