# Tutorium - Grundlagen Datenbanken - Blatt 6

## Vorbereitungen
* Für dieses Aufgabenblatt wird die SQL-Dump-Datei `schema_default.sql` benötigt, die sich im Verzeichnis `sql` befindet.
* Die SQL-Dump-Datei wird in SQL-Plus mittels `start <Dateipfad/zur/sql-dump-datei.sql>` in die Datenbank importiert.
* Beispiele
  * Linux `start ~/Tutorium.sql`
  * Windows `start C:\Users\max.mustermann\Desktop\Tutorium.sql`

## Datenbankmodell
![Datenbankmodell](./img/schema_default.png)

## Data-Dictionary-Views
![Data-Dictionary-Views](./img/constraint_schema.png)

## Aufgaben

### Aufgabe 1
Wie heißt der Primary Key Contraint der Tabelle `VEHICLE` und für welche Spalten wurde er angelegt?

#### Lösung
```sql
select constraint_name,column_name
from user_cons_columns
where constraint_name = (select constraint_name
from user_constraints uc
where uc.table_name='VEHICLE' and
constraint_type='P');
```

### Aufgabe 2
Für welche Spalte**n** der Tabelle `ACC_VEHIC` wurde ein Foreign Key angelegt und auf welche Spalte/n in welcher Tabelle wird er referenziert?

#### Lösung
```sql
select constraint_name, column_name, table_name
from user_cons_columns
where constraint_name in (Select constraint_name
from user_constraints uc
where uc.table_name='ACC_VEHIC' and
constraint_type='R');
```

### Aufgabe 3
Erstelle einen Check Constraint für die Tabelle `ACCOUNT`, dass der Wert der Spalte `U_DATE` nicht älter sein kann als `C_DATE`.

#### Lösung
```sql
alter table account
add constraint c_datum
check (u_date <= c_date);

Bei Bedarf: enable novalidate, hinter den Ausruck
```

### Aufgabe 4
Erstelle einen Check Constraint der überprüft, ob der erste Buchstabe der Spalte `GAS_NAME` der Tabelle `GAS` groß geschrieben ist.

#### Lösung
```sql
alter table gas
add constraint c_gas check
(GAS_NAME = initcap(GAS_NAME));
```

### Aufgabe 5
Erstelle einen Check Contraint der überprüft, ob der Wert der Spalte `IDENTICATOR` der Tabelle `ACC_VEHIC` eins von diesen möglichen Fahrzeugkennzeichenmustern entspricht. Nutze Reguläre Ausdrücke.

+ B:AB:5000
+ TR:MP:1
+ Y:123456
+ THW:98765
+ MZG:XZ:96

#### Lösung
```sql
alter table acc_vehic
add constraint c_identicator
check
(regexp_like(identicator,'^[A-Z]{1,3}:([0-9]{1,6}|[A-Z]{2}:[0-9]{1,4})$','c'));
```

### Aufgabe 6 - Wiederholung
Liste für alle Personen den Verbrauch an Kraftstoff auf (Missachte hier die unterschiedlichen Kraftstoffe). Dabei ist interessant, wie viel Liter die einzelne Person getankt hat und wie viel Euro sie für Kraftstoffe ausgegeben hat.

#### Lösung
```sql
select surname, sum(liter) as liter, sum(price_l*liter*(1+r.duty_amount)) as preis
from receipt r
inner join account a on (r.account_id = a.account_id)
group by surname;
```

### Aufgabe 7 - Wiederholung
Liste die Tankstellen absteigend sortiert nach der Kundenanzahl über alle Jahre.

#### Lösung
```sql
select provider_name as Tankstelle, sum(account_id) as "Anzahl Kunden"
from gas_station gs
inner join receipt r on (gs.gas_station_id = r.gas_station_id)
inner join provider p on (gs.provider_id = p.provider_id)
group by provider_name
order by sum(account_id) desc;
```

### Aufgabe 8 - Wiederholung
Erweitere das Datenbankmodell um ein Fahrtenbuch, sowie es Unternehmen für ihren Fuhrpark führen. Dabei ist relevant, welche Person an welchem Tag ab wie viel Uhr ein Fahrzeug für die Reise belegt, wie viele Kilometer zurück gelegt wurden und wann die Person das Fahrzeug wieder abgibt.

Berücksichtige bitte jegliche Constraints!

#### Lösung
```sql
CREATE TABLE logbook
( log_id number(10) NOT NULL,
  account_id number(38) NOT NULL,
  acc_vehic_id number(38) NOT NULL,
  kilometer number(7,2) NOT NULL,
  c_date date NOT NULL,
  t_date date NOT NULL,
  CONSTRAINT pk_log_id PRIMARY KEY (log_id),
  CONSTRAINT fk_account_id FOREIGN KEY (account_id) references account(account_id),
  CONSTRAINT fk_account_vehic_id FOREIGN KEY (acc_vehic_id) references acc_vehic(acc_vehic_id)
);
```






