# Tutorium - Grundlagen Datenbanken - Blatt 5

## Vorbereitungen
* Für dieses Aufgabenblatt wird die SQL-Dump-Datei `schema_default.sql` benötigt, die sich im Verzeichnis `sql` befindet.
* Die SQL-Dump-Datei wird in SQL-Plus mittels `start <Dateipfad/zur/sql-dump-datei.sql>` in die Datenbank importiert.
* Beispiele
  * Linux `start ~/Tutorium.sql`
  * Windows `start C:\Users\max.mustermann\Desktop\Tutorium.sql`

## Datenbankmodell
![Datenbankmodell](./img/schema_default.png)

## Aufgaben

### Aufgabe 1
Erstelle mit Dia oder einem anderen Werkzeug eine Abbilung der Mengen, die durch `INNER JOIN`, `RIGHT JOIN`, `LEFT JOIN` und `OUTER JOIN` gemeint sind.

#### Lösung
> Mit Dia abgebildet

### Aufgabe 2
Welche Personen haben kein Fahrzeug? Löse dies einmal mit `LEFT JOIN` und `RIGHT JOIN`.

#### Lösung
```sql
select surname, forename
from account a
left join acc_vehic ac
on (a.account_id = ac.account_id)
where vehicle_id is null;


select surname, forename, vehicle_id
from acc_vehic ac
right join account a
on (ac.account_id = a.account_id)
where vehicle_id is null;
```
