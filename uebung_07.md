# Tutorium - Grundlagen Datenbanken - Blatt 7

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

Analyse den untenstehenden anonymen PL/SQL-Codeblock. Was macht er?
Passe den Codeblock so an, dass nicht nur die ID des Benutzers ausgegeben wird, sondern auch der Vor- und Nachname, als auch die Anzahl seiner Fahrzeuge.

```sql
DECLARE
  v_account_id account.account_id%TYPE;

BEGIN
  SELECT MAX(a.account_id) INTO v_account_id
  FROM account a
  WHERE a.surname LIKE 'P%';

  DBMS_OUTPUT.PUT_LINE('Der neuste Benutzer mit dem Anfangsbuchstaben P im Nachnamen hat die ID ' || v_account_id);

EXCEPTION
  WHEN NO_DATA_FOUND
    THEN RAISE_APPLICATION_ERROR(-20001, 'Es wurde kein Benutzer gefunden');
  WHEN OTHERS
    THEN DBMS_OUTPUT.PUT_LINE ('Folgender unerwarteter Fehler ist aufgetreten: ');
  RAISE;
END;
/
```

#### Lösung
```sql
DECLARE
  v_account_id account.account_id%TYPE;
  v_surname account.surname%TYPE;
  v_forename account.forename%TYPE;
  v_countCars acc_vehic.vehicle_id%TYPE;

BEGIN
  SELECT a.surname, a.forename,max(a.account_id), count(accv.vehicle_id)
  INTO v_surname, v_forename, v_account_id, v_countCars
  FROM account a
  inner join acc_vehic accv on a.account_id = accv.account_id
  where a.surname LIKE 'P%' 
  group by a.surname, a.forename;

  DBMS_OUTPUT.PUT_LINE('Der neuste Benutzer mit dem Anfangsbuchstaben P im Nachnamen hat die ID ' || v_account_id);
  DBMS_OUTPUT.PUT_LINE('Der Nachname ist: ' || v_surname);
  DBMS_OUTPUT.PUT_LINE('Der Vorname ist: ' || v_forename);
  DBMS_OUTPUT.PUT_LINE('Anzahl Autos: ' || v_countCars);

EXCEPTION
  WHEN NO_DATA_FOUND
    THEN RAISE_APPLICATION_ERROR(-20001, 'Es wurde kein Benutzer gefunden');
  WHEN OTHERS
    THEN DBMS_OUTPUT.PUT_LINE ('Folgender unerwarteter Fehler ist aufgetreten: ');
  RAISE;
END;
/
```

### Aufgabe 2
Schreibe einen anonymen PL/SQL-Codeblock, der die Tankstelle mit der kleinsten ID auflistet mit Informationen über den Anbieter und der Addresse. Implementiere ein `IF-ELSE` Konstrukt, dass wenn eine Tankstelle mehr Kundenbesuch erziehlt hat, als alle anderen im Durchschnitt, die Tankstelle als gut Besucht gekennzeichnet wird in der Ausgabe. Andernfalls wird die Tankstelle als schlecht Besucht gekennzeichnet.

#### Lösung
```sql
DECLARE
  v_min_tankstellen_id gas_station.gas_station_id%TYPE;
  v_provider_name provider.provider_name%TYPE;
  v_plz address.plz%TYPE;
  v_city address.city%TYPE;
  v_country_name country.country_name%TYPE;
  v_average_visit NUMBER;
  v_count_visits NUMBER;
  v_rating VARCHAR2(100);

BEGIN

	BEGIN
		select min(gas_station_id) into v_min_tankstellen_id
		from gas_station;
	EXCEPTION
		WHEN NO_DATA_FOUND
                THEN RAISE_APPLICATION_ERROR(-20001, 'Es wurde keine Tankstelle gefunden');
	END;

	BEGIN
		select p.provider_name, a.plz, a.city, c.country_name
		into v_provider_name, v_plz, v_city, v_country_name
		from gas_station gs
		inner join provider p on gs.provider_id = p.provider_id
		inner join address a on gs.address_id = a.address_id
		inner join country c on gs.country_id = c.country_id
		where gs.gas_station_id = v_min_tankstellen_id;
	EXCEPTION
		WHEN NO_DATA_FOUND
                THEN RAISE_APPLICATION_ERROR(-20001, 'Es wurde keine Tankstelle gefunden');
	END;

	BEGIN
		select avg(count(gas_station_id))
		into v_average_visit
		from receipt
		where gas_station_id != v_min_tankstellen_id
		group by gas_station_id;
	END;

	BEGIN
		select count(gas_station_id)
		into v_count_visits
		from receipt
		where gas_station_id = 1;
	EXCEPTION
		WHEN NO_DATA_FOUND
                THEN RAISE_APPLICATION_ERROR(-20001, 'Es wurde keine Tankstelle gefunden');
	END;

	IF v_count_visits > v_average_visit THEN
		v_rating := 'Gute Tankstelle!';
	ELSE
		v_rating := 'Schlechte Tankstelle!';
	END IF;

	
  
  DBMS_OUTPUT.PUT_LINE('TankstellenID ' || v_min_tankstellen_id);
  DBMS_OUTPUT.PUT_LINE('Name: ' || v_provider_name);
  DBMS_OUTPUT.PUT_LINE('PLZ: ' || v_plz);
  DBMS_OUTPUT.PUT_LINE('Stadt: ' || v_city);
  DBMS_OUTPUT.PUT_LINE('Land: ' || v_country_name);
  DBMS_OUTPUT.PUT_LINE('Bewertung: ' || v_rating);
  

EXCEPTION
  WHEN OTHERS
    THEN DBMS_OUTPUT.PUT_LINE ('Folgender unerwarteter Fehler ist aufgetreten: ');
  RAISE;
END;
/

```

### Aufgabe 3
Analysiere den untenstehenden anonymen PL/SQL-Code. Was macht er?
Passe den Codeblock so an, dass für jede Tankstelle alle Kunden die dort einmal tanken, waren ausgegeben werden.

```sql
DECLARE
BEGIN
  DBMS_OUTPUT.PUT_LINE('Liste alle Tankstellen aus Deutschland');
  DBMS_OUTPUT.PUT_LINE('____________________________________________');
  FOR rec_gs IN (  SELECT p.provider_name, gs.street, a.plz, a.city, c.country_name
                    FROM gas_station gs
                      INNER JOIN address a ON (a.address_id = gs.address_id)
                      INNER JOIN provider p ON (gs.provider_id = p.provider_id)
                      INNER JOIN country c ON (gs.country_id = c.country_id)
                    WHERE c.country_name LIKE 'Deutschland') LOOP
    DBMS_OUTPUT.PUT_LINE('++ ' || rec_gs.provider_name || ' ++ ' || rec_gs.street || ' ++ ' || rec_gs.plz || ' ++ ' || rec_gs.city || ' ++ ' || rec_gs.country_name);
  END LOOP;
END;
/
```

#### Lösung
```sql
DECLARE
BEGIN
  DBMS_OUTPUT.PUT_LINE('Liste alle Tankstellen aus Deutschland');
  DBMS_OUTPUT.PUT_LINE('____________________________________________');
  FOR rec_gs IN (  SELECT gs.gas_station_id,p.provider_name, gs.street, a.plz, a.city, c.country_name
                    FROM gas_station gs
                      INNER JOIN address a ON (a.address_id = gs.address_id)
                      INNER JOIN provider p ON (gs.provider_id = p.provider_id)
                      INNER JOIN country c ON (gs.country_id = c.country_id)
                    WHERE c.country_name LIKE 'Deutschland') LOOP
    DBMS_OUTPUT.PUT_LINE('++ ' || rec_gs.provider_name || ' ++ ' || rec_gs.street || ' ++ ' || rec_gs.plz || ' ++ ' || rec_gs.city || ' ++ ' || rec_gs.country_name);
  
	  FOR rec_acc IN (SELECT distinct a.surname, a.forename
			  FROM receipt r
			  INNER JOIN account a on r.account_id = a.account_id
			  WHERE r.gas_station_id = rec_gs.gas_station_id) LOOP
	  DBMS_OUTPUT.PUT_LINE('-> Getankt: ' || rec_acc.surname|| '  ' || rec_acc.forename);
   	  END LOOP;
  END LOOP;
END;
/
```

### Aufgabe 4
Schreibe einen anonymen PL/SQL-Codeblock, der alle deine Fahrzeuge auflistet und die dazugehörigen Belege inkl. Betrag, der ausgegeben wurde für jeden Tankvorgang.

#### Lösung
```sql
DECLARE
BEGIN
  DBMS_OUTPUT.PUT_LINE('Liste aller eigener Fahrzeuge');
  DBMS_OUTPUT.PUT_LINE('____________________________________________');
  FOR rec_vehic IN (  SELECT v.vehicle_id, vt.vehicle_type_name, p.producer_name, v.version, v.ps
                      FROM acc_vehic acv
		      INNER JOIN vehicle v on acv.vehicle_id = v.vehicle_id
		      INNER JOIN producer p on v.producer_id = p.producer_id
		      INNER JOIN vehicle_type vt on v.vehicle_type_id = vt.vehicle_type_id
		      WHERE acv.account_id = 10  ) LOOP
    DBMS_OUTPUT.PUT_LINE('++ ' || rec_vehic.vehicle_id || ' ++ ' || rec_vehic.vehicle_type_name || ' ++ ' || rec_vehic.producer_name || ' ++ ' || rec_vehic.version || ' ++ ' || rec_vehic.ps);

	FOR rec_receipt IN (Select price_l, liter, price_l*liter+(1*duty_amount) as Betrag, c_date
                            from receipt 
			    where acc_vehic_id in (Select acc_vehic_id from acc_vehic where vehicle_id = rec_vehic.vehicle_id )) LOOP
	DBMS_OUTPUT.PUT_LINE('-> Datum: ' || rec_receipt.c_date || ' || Preis/Liter: ' || rec_receipt.price_l || ' || Liter: ' || 			rec_receipt.liter || ' || Betrag: ' || rec_receipt.Betrag);
	DBMS_OUTPUT.PUT_LINE('-----------------------------------------------------');
        END LOOP;
  END LOOP;
END;
/
```
