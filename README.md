# CTF Write-Up: Oh no, not again!
https://tryhackme.com/room/committed
## Challenge
Einer unserer Entwickler hat versehentlich sensible Daten in ein GitHub-Repository committed.  
Unsere Aufgabe: herausfinden, **was** und **wo** das war.  
Die Dateien lagen im Verzeichnis `/home/ubuntu/commited` auf der VM.

---

## 1. Erste Analyse
Zunächst lag dort nur ein Archiv:

```bash
ls -la /home/ubuntu/commited
# -> commited.zip
```

Nach dem Entpacken tauchte ein Projekt mit `.git/`-Ordner auf:

```bash
unzip commited.zip
cd commited
```

---

## 2. Überblick über das Repository
Wir prüfen den Zustand und die Branches:

```bash
git status
git branch -v
```

Ausgabe:
```
  dbint  4e16af9 Reminder Added.
* master 28c3621 Finished
```

Es existieren also zwei Branches: `master` und `dbint`.

---

## 3. Commit-History durchsuchen
Mit einer Log-Übersicht konnten wir verdächtige Commits identifizieren:

```bash
git log --all --decorate --oneline --graph --stat --date=short
```

Auffällig:
- `3a8cc16 DB check`
- `c56c470 Oops`

---

## 4. Suche nach Passwörtern in der History
Gezielt nach hartcodierten Passwörtern gesucht:

```bash
git log --all -p -G 'password\s*=' -- main.py
```

Treffer im Commit `3a8cc16 (DB check)`:

```diff
 user="root", # Username Goes Here
-password="flag{********************************}" # Password Goes Here
+password="" # Password Goes Here
```

Im nächsten Commit (`c56c470 Oops`) wurde das Passwort entfernt – bleibt aber in der Git-History sichtbar.

---

## 5. Ergebnis
- **Commit mit Leak:** `3a8cc16`
- **Datei:** `main.py`
- **Secret:**  
  ```
  flag{********************************}
  ```

---

## 6. Lessons Learned
- **Secrets nie hardcoden** → Umgebungsvariablen oder `.env` verwenden.  
- **History bereinigen**, nicht nur den letzten Commit.  
- **BFG Repo-Cleaner / git filter-repo** nutzen, um versehentlich geleakte Secrets endgültig zu entfernen.  
- **Secrets rotieren**, sobald sie veröffentlicht wurden.

---

## Flag
```
flag{********************************}
```
