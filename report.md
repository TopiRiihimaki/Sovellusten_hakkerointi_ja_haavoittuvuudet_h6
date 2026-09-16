# Tutkimuksen tavoite

Tutkimuksen tavoitteena oli selvittää, millaisia turvallisuuteen liittyviä mekanismeja Tapo C200 V3 -kameran firmware sisältää ja olisiko niissä joitakin heikkouksia.

Tutkimus tehtiin ensisijaisesti staattisena firmware-analyysinä.

# Tutkimusympäristö ja työkalut

- Debian Linux
  * Analyysiympäristönä käytettiin Debiania. Komentoriviä käytettiin muun muassa firmware-tiedoston tarkastamiseen ja analyysityökalujen suorittamiseen. (Esim. file Tapo_C200v3_en_1.4.2.bin.dec)

- tp-link-decrypt
    
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

```
00420230
00425f98
0044f23c
0044f3f0
0046650c
00494310
00494340
004b9204
004b93c4
004b945c
...
```
Tämä osoitti, että root-käyttäjätietoa käytetään useissa eri ohjelmakohdissa.

Tämän jälkeen aloin käymään kiinnostavimpia XREFejä tarkemmin läpi.

# /user_management/root-rakenteen tutkiminen

Firmwaresta löytyi `/user_management/root`-niminen tietue. Funktio `FUN_004b91dc` lukee sen `ds_read()`-funktiolla 440 tavun kokoiseksi rakenteeksi ja käsittelee siitä muun muassa kenttiä `username`, `ciphertext` ja `comment`.

Tämä viittaa siihen, että kyseessä on root-käyttäjän tietue. Sen perusteella ei kuitenkaan voida todeta, että root-salasana olisi tallennettu sinne selväkielisenä.

# ciphertext-kentän löytäminen

FUN_004b91dc:ssä nähtiin:
```
jso_add_string(..., "username", ...)
jso_add_string(..., "ciphertext", ...)
jso_add_string(..., "comment", ...)
```
Tämä on kiinnostava havainto, koska firmware ei näytä yksinkertaisesti palauttavan esimerkiksi:
```
password = plaintext
```
vaan käyttäjätietueessa on ainakin kenttä nimeltä ciphertext.

Toki kentän nimi ei tarkoita, että tässä käytettäisiin mitään salausalgoritmia.

# general_password

Toinen erittäin kiinnostava haara oli:

general_password

Sen käsittely johti funktioon:

FUN_004267e4

Funktio tekee seuraavaa:

Ensimmäiseksi tarkistetaan parametrin pituus:
```
sVar1 = strlen(param_1);

if (sVar1 != 0x40)
    return error;
```
Eli parametrin täytyy olla 0x40 = 64 merkkiä

Sen jälkeen luetaan /user_management/hub_auth ja parametrin sisältö kopioidaan authentication-rakenteeseen.

Lopuksi tehdään:
```
ds_advanced_write("/user_management/hub_auth", ...)
```

# Salasanan ja käyttäjähallinnan yhdistäminen

Seuraavaksi tutkin hiukan:
```
root_passwd
general_password
get_cam_passwd
gen_root_passwd
ciphertext
```
Ilmeisesti firmwarella vaikuttaisi olevan useita mekanismeja salasanaan liittyen. En kuitenkaan pystynyt todistamaan, että noita pystyttäsiin suoraan hyödyntämään salasanan löytämisessä. 

# FUN_004b93a4 ja käyttäjien etsiminen

Löysin:
```
FUN_004b93a4
```
joka käyttää kolmea käyttäjähallinnan polkua:
```
/user_management/root
/user_management/admin
/user_management/guest
```
joka myös kutsuu:
```
FUN_004b9260
```
tämä funktio muodostaa nimen %s_%d ja käyttää ds_read() sekä strcmp().

Tämän perusteella koodi näyttää etsivän tiettyä käyttäjätietuetta persistentistä datasta. 

# storage_info-haara

tarkistin FUN_0042a33c, josta löytyi jälleen general_password ja kutsu FUN_004267e4(iVar2,2);

Eli samaa salasanaan liittyvää funktiota käytetään useammassa ohjelmakohdassa.

# Mikä oli tutkimuksen rajoitus
Tutkimus jäi pääasiassa staattiseksi reverse engineeringiksi.

# Teköälyn käyttö
Teköälyä on käytetty seuraavasti:
- Auttanut käyttämään Ghidraa
- Ghidrasta saatujen tietojen ymmärtämistä
- Raportin ideoinnissa
- Oikeiden komentojen laittamista
