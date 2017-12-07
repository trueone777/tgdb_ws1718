# Tutorium - Grundlagen Datenbanken - Blatt 9

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
Wo liegen die Vor- und Nachteile eines Trigger in Vergleich zu einer Prozedur?

#### Lösung
Trigger werden immer wieder automatisch ausgeführt, falls ein Ereignis in der Datenbank passiert.
Eine Prozedur muss man explizit ausführen.
Der Nachteil des Triggers ist, dass es zu einer Trigger Verkettung kommen kann. Trigger müssen dann
auf Ausführungen anderer Trigger warten um die Operation abzuschließen. Das führt zur langsamer Performance.

### Aufgabe 2
Wo drin unterscheidet sich der `Row Level Trigger` von einem `Statement Trigger`?

#### Lösung
Row Level Trigger - Der Trigger wird bei jeder Zeile in der Tabelle ausgeführt
Statement Trigger - Der Trigger wird nur einmal ausgeführt, egal ob Zeilen in der Tabelle vorhanden sind oder nicht.

### Aufgabe 3
Schaue dir den folgenden PL/SQL-Code an. Was macht er?

```sql
CREATE SEQUENCE seq_account_id
START WITH 1000
INCREMENT BY 1
MAXVALUE 99999999
CYCLE
CACHE 20;

CREATE OR REPLACE TRIGGER BIU_ACCOUNT
BEFORE INSERT OR UPDATE OF account_id ON account
FOR EACH ROW
DECLARE

BEGIN
  IF UPDATING('account_id') THEN
    RAISE_APPLICATION_ERROR(-20001, 'Die Account-ID darf nicht verändert oder frei gewählt werden!');
  END IF;

  IF INSERTING THEN
    :NEW.account_id := seq_account_id.NEXTVAL;
  END IF;
END;
/
```

#### Lösung
Zuerst wird eine Sequence mit einem Zahlenbereich angelegt, wobei es immer um 1 inkrementiert wird.
Falls man die account_id in der Tabelle account einfügen oder verändern möchte, wird davor ein Trigger ausgelöst.
Bei einem Update des Datensatzes stellt er sicher, dass man die AccountID nicht verändern kann.
Fügt man einen neuen Eintrag hinzu so wird als account_id eine neu generierte Zahl aus der Sequence übernommen. (Auto Increment)

### Aufgabe 4
Verbessere den Trigger aus Aufgabe 2 so, dass
+ wenn versucht wird einen Datensatz mit `NULL` Werten zu füllen, die alten Wert für alle Spalten, die als `NOT NULL` gekennzeichnet sind, behalten bleiben.
+ es nicht möglich ist, das die Werte für `C_DATE` und `U_DATE` in der Zukunkt liegen
+ `U_DATE` >= `C_DATE` sein muss
+ der erste Buchstabe jedes Wortes im Vor- und Nachnamen groß geschrieben wird
+ die Account-ID aus einer `SEQUENCE` entnommen wird

Nutze die Lösung der Aufgabe 2, Aufgabenblatt 8 um die Aufgabe zu lösen. Dort solltest du einige Hilfestellungen finden.

#### Lösung
```sql
CREATE OR REPLACE TRIGGER receipt_trigger
	BEFORE 
			INSERT
		OR	UPDATE receipt_id, account_id, acc_vehic_id, duty_amount,gas_id,
			gas_station_id,price_l,kilometer,liter,receipt_date,c_date,u_date
	ON receipt
	FOR EACH ROW
BEGIN
	CASE
		WHEN INSERTING THEN
			DBMS_OUTPUT.PUT_LINE('Inserting ' || :NEW.Spalte1)
		WHEN UPDATING (receipt_id) THEN
			IF (NEW.receipt_id IS NULL) THEN
				receipt_id = receipt_id;
			END IF;
			
			
			
		WHEN UPDATING (account_id) THEN
			IF (NEW.account_id IS NULL) THEN
				account_id = account_id;
			END IF;

									
		WHEN UPDATING (acc_vehic_id) THEN
			IF (NEW.acc_vehic_id IS NULL) THEN
				acc_vehic_id = acc_vehic_id;
			END IF;
		WHEN UPDATING (duty_amount) THEN
			IF (NEW.duty_amount IS NULL) THEN
				NEW.duty_amount = duty_amount;
			END IF;
		WHEN UPDATING (gas_id) THEN
			DBMS_OUTPUT.PUT_LINE('Updating Spalte1')
		WHEN UPDATING (gas_station_id) THEN
			DBMS_OUTPUT.PUT_LINE('Updating Spalte1')
		WHEN UPDATING (price_l) THEN
			DBMS_OUTPUT.PUT_LINE('Updating Spalte1')
		WHEN UPDATING (kilometer) THEN
			DBMS_OUTPUT.PUT_LINE('Updating Spalte1')
		WHEN UPDATING (liter) THEN
			DBMS_OUTPUT.PUT_LINE('Updating Spalte1')
		WHEN UPDATING (receipt_date) THEN
			DBMS_OUTPUT.PUT_LINE('Updating Spalte1')
		WHEN UPDATING (c_date) THEN
			IF (c_date > sysdate) THEN
				raise_application_error(-20000,'c_date darf nicht in der Zukunft liegen');
			END IF;
		WHEN UPDATING (u_date) THEN
			IF (u_date < c_date) THEN
				raise_application_error(-20000,'u_date muss groesser als c_date sein');
			END IF;
		WHEN DELETING THEN
			DBMS_OUTPUT.PUT_LINE('Deleting ' || :OLD.Spalte1)
			RAISE_APPLICATION_ERROR (-20501, 'Verboten!');
	END CASE;
END;
```

### Aufgabe 5
Angenommen der Steuersatz in Deutschland sinkt von 19% auf 17%.
+ Aktualisiere den Steuersatz von Deutschland und
+ alle Quittungen die nach dem `01.10.2017` gespeichert wurden.

#### Lösung
```sql

```

### Aufgabe 6
Liste alle Hersteller auf, die LKW's produzieren und verknüpfe diese ggfl. mit den Eigentümern.

#### Lösung
```sql
Deine Lösung
```


























