# Tutorium - Grundlagen Datenbanken - Blatt 3

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
Erstelle eine `INNER JOIN` (optional `WHERE`) Abfrage um die Beziehungen zwischen den Tabellen `GAS_STATION`, `Provider`, `COUNTRY` und `ADDRESS` aufzulösen, sodass eine ähnliche Ausgabe erstellt wird wie abgebildet.

| Anbieter  | Straße            | PLZ   | Ort         | Land        | Steuer  |
| --------- | ----------------- | ----- | ----------- | ----------- | ------- |
| Shell     | Zurlaubener 1     | 54292 | Trier       | Deutschland | 0.19    |
| Esso      | Triererstraße 10  | 53937 | Hellenthal  | Deutschland | 0,19    |
| ...       | ...               | ...   | ...         | ...         | ...     |

#### Lösung
```sql
Select p.provider_name as Anbieter, gs.street as Straße, a.plz , a.city as Ort, c.country_name as Land, c.duty_amount as Steuer from gas_station gs
inner join provider p
on (gs.provider_id = p.provider_id)
inner join country c
on (gs.country_id = c.country_id)
inner join address a
on (gs.address_id = a.address_id);
```

### Aufgabe 2
Suche alle Tankstellen raus, deren Straßenname an zweiter Stelle ein `U` haben (case-insensetive). Verändere dazu die Abfrage aus Aufgabe 1. Optional für Enthusiasten, suche mittels Regulärem Ausdruck.

#### Lösung
```sql
Select p.provider_name as Anbieter, gs.street as Straße, a.plz , a.city as Ort, c.country_name as Land, c.duty_amount as Steuer from gas_station gs
inner join provider p
on (gs.provider_id = p.provider_id)
inner join country c
on (gs.country_id = c.country_id)
inner join address a
on (gs.address_id = a.address_id)
 where regexp_like(gs.street, '^.[n,N]');
```

### Aufgabe 3
Suche alle Tankstellen raus, die sich in Trier befinden.

#### Lösung
```sql
Select p.provider_name as Anbieter, gs.street as Straße, a.plz , a.city as Ort, c.country_name as Land, c.duty_amount as Steuer from gas_station gs
inner join provider p
on (gs.provider_id = p.provider_id)
inner join country c
on (gs.country_id = c.country_id)
inner join address a
on (gs.address_id = a.address_id)
 where a.city = 'Trier';
```

#### Aufgabe 4
Füge eine fiktive Tankstelle hinzu. Sie darf auf keine bestehenden Informationen basieren. Nutze möglichst wenige SQL-Befehle. Rufe fehlende Informationen möglichst direkt ab.

#### Lösung
```sql
Deine Lösung
```

### Aufgabe 5
Erstelle eine INNER JOIN (optional `WHERE`) Abfrage um die Beziehung zwischen den Tabellen `ACCOUNT`, `VEHICLE`, `VEHICLE_TYPE`, `GAS` und `PRODUCER` aufzulösen und zeige die Spalten `FORNAME`, `SURNAME`, `VEHICLE_TYPE_NAME`, `VERSION`, `BUILD_YEAR`, `PRODUCER_NAME` und `GAS_NAME` an. Richte SQL-Plus so ein, dass möglicht jeder Datensatz nur eine Zeile belegt.

* COLUMN <SPALTENNAME> FORMAT a<Zeichenlänge>
* COLUMN <SPALTENNAME> FORMAT <Zahlenlänge, pro Länge eine 9>
* Beispiel für eine Spalte des Datentyps `VARCHAR2`: `COLUMN SURNAME FORMAT a16`
* Beispiel für eine Spalte des Datentyps `NUMERIC`: `COLUMN SOLD_KILOMETER 9999`

#### Lösung
```sql
Select acc.forename, acc.surname, veht.vehicle_type_name, vehc.version, vehc.build_year, prod.producer_name, g.gas_name
from acc_vehic accve
inner join account acc
on (accve.account_id = acc.account_id)
inner join vehicle vehc
on (accve.vehicle_id = vehc.vehicle_id)
inner join vehicle_type veht
on (vehc.vehicle_type_id = veht.vehicle_type_id)
inner join producer prod
on (vehc.producer_id = prod.producer_id)
inner join gas g
on (vehc.default_gas_id = g.gas_id);

column forename format a16 usw.
```

### Aufgabe 6
Welche Fahrzeuge wurden noch keinem Benutzer zugewiesen? Gebe über das Fahrzeug Informationen über den Typ, den Hersteller, das Modell, Baujahr und den Kraftstoff aus.

#### Lösung
```sql
Select  veht.vehicle_type_name, vehc.version, vehc.build_year, prod.producer_name, g.gas_name
from vehicle veh
inner join vehicle vehc
on (veh.vehicle_id = vehc.vehicle_id)
inner join vehicle_type veht
on (vehc.vehicle_type_id = veht.vehicle_type_id)
inner join producer prod
on (vehc.producer_id = prod.producer_id)
inner join gas g
on (vehc.default_gas_id = g.gas_id)
where veh.vehicle_id not in (Select vehicle_id from acc_vehic);
```

### Aufgabe 7
Verknüpfe eines der Autos aus Aufgabe 6 mit deinem Benutzernamen. Verwende dazu möglichst wenige SQL-Statements.

#### Lösung
```sql
insert into acc_vehic
values (5661, 10, 1, 'g:334' , 'Autowrack', 555.55,210000, 477.77, 245000, sysdate, sysdate, null,sysdate,sysdate);
```

### Aufgabe 8
An welcher Tankstelle wurde noch nie getankt? Gebe zu den Tankstellen die Informationen über den Standort der Tankstellen aus.

#### Lösung
```sql
select gas_station_id, street,a.plz, a.city,c.country_name from gas_station
inner join country c
on (gas_station.country_id = c.country_id)
inner join address a
on (gas_station.address_id = a.address_id)
where  gas_station_id  in (select gas_station_id from gas_station minus select default_gas_station from acc_vehic);
```

### Aufgabe 9
Liste alle Benutzer (Vorname und Nachname) mit Fahrzeug (Hersteller, Modell, Alias) auf, die noch nie einen Beleg hinzugefügt haben.

#### Lösung
```sql
select a.forename as Vorname, a.surname as Nachname, prod.producer_name as Hersteller, vehc.version as Modell, alias from acc_vehic av
inner join account a
on (av.account_id = a.account_id)
inner join vehicle vehc
on (av.vehicle_id = vehc.vehicle_id)
inner join producer prod
on (vehc.producer_id = prod.producer_id)
where a.account_id not in (select account_id from receipt);
```

### Aufgabe 10
Liste alle Benutzer auf, die mit einem Fahrzeug schonmal im Außland tanken waren.

#### Lösung
```sql
Select default_gas_station,a.forename as Vorname, a.surname as Nachname
from acc_vehic av
inner join account a
on (av.account_id = a.account_id)
 where default_gas_station != 1 and default_gas_station is not null;

```

### Aufgabe 11
Wie viele Benutzer haben einen LKW registriert?

#### Lösung
```sql
select count(account_id) as Anzahl_LKW_Nutzer from acc_vehic av
inner join vehicle vehc
on (av.vehicle_id = vehc.vehicle_id)
inner join vehicle_type t
on (vehc.vehicle_type_id = t.vehicle_type_id)
where t.vehicle_type_name = 'LKW';
```

### Aufgabe 12
Wie viele Benutzer haben einen PKW und einen LKW registriert?

#### Lösung
```sql
Deine Lösung
```

### Aufgabe 13
Führe den Patch `02_patch.sql`, der sich im Verzeichnis `sql` befindet, in deiner Datenbank aus. Wie lautet der Befehlt zum import?

#### Lösung
```sql
Deine Lösung
```

### Aufgabe 14
Aktualisiere den Steuersatz aller Belege auf den Steuersatz des Landes, indem die Kunden getankt haben.

#### Lösung
```sql
select count(distinct account_id) as Anzahl_PKW_und_LKW_Nutzer from acc_vehic av
inner join vehicle vehc
on (av.vehicle_id = vehc.vehicle_id)
inner join vehicle_type t
on (vehc.vehicle_type_id = t.vehicle_type_id)
where t.vehicle_type_name in ('LKW', 'PKW')
group by account_id
having count(distinct t.vehicle_type_name) = 2;
```


