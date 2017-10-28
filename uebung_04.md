# Tutorium - Grundlagen Datenbanken - Blatt 4

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
Um genauere Informationen und Prognosen mit Data Mining Werkzeugen zu schöpfen, ist es notwendig mehr Informationen über die registrierten Benutzer zu sammeln und zu speichern. Die in Zukunft gesammelten Informationen sollen in neuen Tabellen des bestehenden Datenbankmodells gespeichert werden. Dazu soll jedem Benutzer einen Erst- und Zweitwohnsitz zugeordnet werden. Jeder Wohnsitz besitzt eine eigene Adresse. Integriere in das bestehende Datenbankmodell Tabellen die den genauen Erst- und Zweitwohnsitz abbilden können. Beachte dazu die Normalisierungsformen bis 3NF - [Dokumentation](https://de.wikipedia.org/wiki/Normalisierung_(Datenbank)). Wie lautet deine SQL-Syntax um deine Erweiterung des Datenbankmodells zu implementieren?

#### Lösung
```sql
create table residence ( 
res_id number(5) not null,
street varchar(40) not null,
plz number(5) not null,
city varchar(32) not null);

create table res_acc (
account_id number(38) not null,
first_res_id number(38) not null,
second_res_id number(38));
```

### Aufgabe 2
Als App Entwickler/in für Android und iOS möchtest du dich nicht darauf verlassen, dass die Adresse exakt richtig ist und überlegst in dem Datenbankmodell noch zwei zusätzliche Attribute (X und Y Koordinate) zur genauen GPS Lokalisierung einer Tankstelle aufzunehmen. Wie lautet deine SQL-Syntax um das Datenbankmodell auf die zwei Attribute zu erweitern?

#### Lösung
```sql
alter table gas_station
add (x_coord number(10,5),
     y_coord number(10,5));
```

### Aufgabe 3
Welche Kunden haben im Jahr 2017 mehr als den Durchschnitt getank?

#### Lösung
```sql
select distinct account_id
from receipt
where extract(YEAR from c_date) = 2017
and price_l*liter*(1+duty_amount) > (Select avg(price_l*liter*(1+duty_amount)) from receipt);
```

```sql
Für Liter:
select distinct account_id
from receipt
where extract(YEAR from c_date) = 2017
and receipt.liter > (Select avg(liter) from receipt);
```

### Aufgabe 4
Ermittle, warum du INSERT-Rechte auf die Tabelle `SCOTT.EMP` und UPDATE-Rechte auf die Tabelle `SCOTT.DEPT` besitzt. Beantworte dazu schrittweise die Aufgaben von 4.1 bis 4.4.

#### Aufgabe 4.1
Wurden die Tabellen-Rechte direkt an dich bzw. an `PUBLIC` vergeben?

##### Lösung
```sql
select * from all_tab_privs
where table_schema='SCOTT';

update Rechte von DEPT an public vergeben
```

#### Aufgabe 4.2
Welche Rollen besitzt du direkt?

##### Lösung
```sql
select * from user_role_privs;
Role: FH-TRIER
```

#### Aufgabe 4.3
Welche Rollen haben die Rollen?

##### Lösung
```sql
select * from role_role_privs;
ROLE FH-Trier besitzt Role WI_STUDENT
```

#### Aufgabe 4.4
Haben die Rollen Rechte an `SCOTT.EMP` oder `SCOTT.DEPT`?

##### Lösung
```sql
select * from role_role_privs;
dept gibt keine rechte an rollen, sonder direkt an public
```

### Aufgabe 5
Es soll für jede Tankstelle der Umsatz einzelner Jahre aufgelistet werden auf Basis der Daten, die Benutzer durch ihre Quittungen eingegeben haben. Sortiere erst nach Jahr und anschließend nach der Tankstelle. Beispiel:

| Jahr  | Anbieter  | Straße            | PLZ   | Stadt | Land          | Umsatz    |
| ----- | --------- | ----------------- | ----- | ----- | --------------| --------- |
| 2017  | Esso      | Triererstraße 15  | 54292 | Trier | Deutschland   | 54784.14  |
| 2017  | Shell     | Zurmainerstraße 1 | 54292 | Trier | Deutschland   | 67874.78  |
| 2016  | Esso      | Triererstraße 15  | 54292 | Trier | Deutschland   | 57412.66  |
| 2016  | Shell     | Zurmainerstraße 1 | 54292 | Trier | Deutschland   | 72478.42  |

#### Lösung
```sql
select extract(year from c_date) jahr, p.provider_name Anbieter,
street Straße, a.plz, a.city Stadt, c.country_name Land, sum(price_l*liter*(1+r.duty_amount)) Umsatz
from receipt r
inner join gas_station gs on r.gas_station_id = gs.gas_station_id
inner join provider p on gs.provider_id = p.provider_id
inner join address a on gs.address_id = a.address_id
inner join country c on gs.country_id = c.country_id
group by c_date, p.provider_name, street, a.plz, a.city,c.country_name
order by extract(year from c_date), p.provider_name,
street, a.plz, a.city, c.country_name; 
```


