# Tutkimuksen tavoite

Tutkimuksen tavoitteena oli selvittää, millaisia turvallisuuteen liittyviä mekanismeja Tapo C200 V3 -kameran firmware sisältää.

Tutkimus tehtiin ensisijaisesti staattisena firmware-analyysinä.

# Tutkimusympäristö ja työkalut

- Debian Linux
  * Analyysiympäristönä käytettiin Debiania. Komentoriviä käytettiin muun muassa firmware-tiedoston tarkastamiseen ja analyysityökalujen suorittamiseen. (Esim. file Tapo_C200v3_en_1.4.2.bin.dec)
- Ghidra
  * Ghidra oli hyvin hyödyllinen työkälu reverse engineeringissä. Käytin sitä esimerkiksi: avasin firmware/binaari, käytin decompileria, etsin merkkijonoja, etsin stringien cross-referencejä ja seurasin funktijokutsuja.
  Nämä asiat esimerkiksi löysin Ghidran avulla
```
/user_management/root
/user_management/admin
/user_management/guest
/user_management/hub_auth
```

# Firmwaresta kiinnostavien merkkijonojen etsiminen

Ensimmäinen hyödyllinen havainto oli käyttäjähallintaan liittyvien stringien löytäminen.

Esimerkiksi:
```
/user_management/root
/user_management/admin
/user_management/guest
```
sekä
```
/user_management/hub_auth
```
Tämän perusteella tutkimus rajattiin käyttäjähallintaan liittyvään koodiin.

# Cross-reference-analyysi
Kun string löydettiin, Ghidrassa katsoin sen XREFit, eli missä kohdissa ohjelmakoodia kyseistä stringiä käytetään.
<img width="410" height="290" alt="Screenshot_2026-09-15_21-12-20" src="https://github.com/user-attachments/assets/5a911ff9-1440-4243-aa43-0b1ca411ed1b" />

<img width="410" height="403" alt="Screenshot_2026-09-15_21-14-35" src="https://github.com/user-attachments/assets/4d9220f3-a951-4dea-a5fe-7a90114fb3a9" />


