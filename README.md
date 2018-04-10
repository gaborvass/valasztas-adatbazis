Live SQL interfész
==================

http://valasztas.sztupy.hu/

A fenti link kis terhelésű szerveren fut, így esélyes, hogy lassú, vagy nem működik,
így ha lehet inkább töltsd le az adatokat innen: https://github.com/sztupy/valasztas-adatbazis/tree/master/sql
valamint a sémát innen: https://github.com/sztupy/valasztas-adatbazis/blob/master/schema.sql
és egy helyi adatbázisban használd

Az SQLite adatbázis elkészítve letölthető innen is: https://github.com/sztupy/valasztas-adatbazis/raw/master/deploy_tools/valasztas.sqlite3

Letöltés után, ha van docker-ed az alábbi paranccsal tudod a web interfészt előhozni:

    docker run -it --rm -p 8080:8080 -v /path/to/valasztas.sqlite3:/data/valasztas.sqlite3:ro -e SQLITE_DATABASE=valasztas.sqlite3 coleifer/sqlite-web

2018-as választási eredmények SQL-ben
=====================================

Pár példa keresés:

1. Adott szavazókör listát szavazatai

```sql
SELECT tlista.tnev, szavt.szav FROM szavf
  JOIN szavt ON szavf.jfid = szavt.jfid
  JOIN tlista ON szavt.jlid = tlista.tlid
  WHERE szavf.maz = '03' AND szavf.taz = '004' AND szavf.sorsz = '009'
  AND szavf.valtip = 'L'
```

2. Adott szavazókör egyéni szavazatai

```sql
SELECT ejelolt.nev, ejelolt.unev1, jlcs.nevt, szavt.szav FROM szavf
  JOIN szavt ON szavf.jfid = szavt.jfid
  JOIN ejelolt ON szavt.jlid = ejelolt.eid
  LEFT JOIN jlcs ON jlcs.jlcs = ejelolt.jlcs
  WHERE szavf.maz = '03' AND szavf.taz = '004' AND szavf.sorsz = '009'
  AND szavf.valtip = 'J'
```

3. Adott szavazókörben a listára és egyénire adott szavazatok különbsége

```sql
SELECT lista.maz, lista.taz, lista.sorsz, lista.tnev, lista.szav as listaszav, egyen.szav as egyenszav FROM
  (SELECT szavf.maz, szavf.taz, szavf.sorsz, tlista.tnev, szavt.szav FROM szavf
   JOIN szavt ON szavf.jfid = szavt.jfid
   JOIN tlista ON szavt.jlid = tlista.tlid
   WHERE szavf.maz = '03' AND szavf.taz = '004' AND szavf.sorsz = '009'
   AND szavf.valtip = 'L') as lista
 JOIN
  (SELECT szavf.maz, szavf.taz, szavf.sorsz, ejelolt.nev, ejelolt.unev1, jlcs.nevt, szavt.szav FROM szavf
   JOIN szavt ON szavf.jfid = szavt.jfid
   JOIN ejelolt ON szavt.jlid = ejelolt.eid
   LEFT JOIN jlcs ON jlcs.jlcs = ejelolt.jlcs
   WHERE szavf.maz = '03' AND szavf.taz = '004' AND szavf.sorsz = '009'
   AND szavf.valtip = 'J') as egyen
 ON lista.tnev = egyen.nevt AND lista.maz = egyen.maz and lista.taz = egyen.taz and lista.sorsz = egyen.sorsz
```

4. Szavazókörök, ahol a listán lényegesen kevesebb szavazat érkezett egy pártra, mint egyéniben

```sql
SELECT lista.maz, lista.taz, lista.sorsz, lista.tnev, lista.szav as listaszav, egyen.szav as egyenszav FROM
  (SELECT szavf.maz, szavf.taz, szavf.sorsz, tlista.tnev, szavt.szav FROM szavf
   JOIN szavt ON szavf.jfid = szavt.jfid
   JOIN tlista ON szavt.jlid = tlista.tlid
   AND szavf.valtip = 'L') as lista
 JOIN
  (SELECT szavf.maz, szavf.taz, szavf.sorsz, ejelolt.nev, ejelolt.unev1, jlcs.nevt, szavt.szav FROM szavf
   JOIN szavt ON szavf.jfid = szavt.jfid
   JOIN ejelolt ON szavt.jlid = ejelolt.eid
   LEFT JOIN jlcs ON jlcs.jlcs = ejelolt.jlcs
   AND szavf.valtip = 'J') as egyen
 ON lista.tnev = egyen.nevt AND lista.maz = egyen.maz and lista.taz = egyen.taz and lista.sorsz = egyen.sorsz
 WHERE listaszav*10 < egyenszav AND egyenszav>20
```

5. Előző ellenkezője: több a listát szavazat, pedig egyéniben is indult a párt

```sql
SELECT lista.maz, lista.taz, lista.sorsz, lista.tnev, lista.szav as listaszav, egyen.szav as egyenszav FROM
  (SELECT szavf.maz, szavf.taz, szavf.sorsz, tlista.tnev, szavt.szav FROM szavf
   JOIN szavt ON szavf.jfid = szavt.jfid
   JOIN tlista ON szavt.jlid = tlista.tlid
   AND szavf.valtip = 'L') as lista
 JOIN
  (SELECT szavf.maz, szavf.taz, szavf.sorsz, ejelolt.nev, ejelolt.unev1, jlcs.nevt, szavt.szav FROM szavf
   JOIN szavt ON szavf.jfid = szavt.jfid
   JOIN ejelolt ON szavt.jlid = ejelolt.eid
   LEFT JOIN jlcs ON jlcs.jlcs = ejelolt.jlcs
   AND szavf.valtip = 'J') as egyen
 ON lista.tnev = egyen.nevt AND lista.maz = egyen.maz and lista.taz = egyen.taz and lista.sorsz = egyen.sorsz
 WHERE listaszav > egyenszav*5 AND egyenszav > 0
 ```

6. Szavazókörök, ahol jelentős mennyiségű érvénytelen szavazat érkezett

```sql
SELECT maz,taz,sorsz,valtip,m
FROM "szavf" WHERE m > 20
```

7. Szavazókörök 85%-os részvétel felett

```sql
SELECT *
FROM "szavf" WHERE cast(f as float)/cast(a as float)>0.85
```
