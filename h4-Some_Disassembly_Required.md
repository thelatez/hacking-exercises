# h4 - Some Disassembly Required

## Tehtävänanto
* x) Katso video (Hammond 2022, GHIDRA for Reverse Engineering (PicoCTF 2022 #42 'bbbloat')), tiivistä.
* a) Lataa Ghidra
* b) Rever-C: käänteismallinna `packd` binääri C-kieleksi Ghidralla. Löydä pääohjelma. Anna muuttujille kuvaavat nimet, selitä ohjelman toiminnallisuus. Ratkaise tehtävä binääristä, ilman lähdekoodia.
* c) If takaperin: muokkaa `passtr` ohjelman binääri niin, että se hyväksyy kaikki salasanat paitsi oikean. Todista.
* d) Nora CrackMe: binäärit Tindall 2023: NoraCodes / crackmes. https://github.com/NoraCodes/crackmes. Käännä binääreiksi. 
* e) Nora crackme01, crackme01e. Ratkaise binäärit.
* f) Nora crackme02. Nimeä ohjelman muuttujat käänteismallinnetusta binääristä, ja selitä sen toiminnallisuus. Ratkaise binääri.

## Vastaus

### x) 
Hammond 2022, GHIDRA for Reverse Engineering (PicoCTF 2022 #42 'bbbloat'):
* Ghidran ylävalikossa "Window" on mielenkiintoinen työkalu "Defined Strings". Tällä saattaa olla helpompi löytää esimerkiksi printattujen kysymysten sijanti binäärissä.
* Jos printatun tekstin perässä on `XREF... : FUN` se tarkoittaa viittausta funktioon. Sitä tuplaklikkaamalla pääsee käskyriville (mitä esim. gdb:llä katsottiin h5).

Video oli omasta mielestä aika huono. Tästä syystä vain 2 aika niche kohtaa.

### a) Lataa Ghidra
Lataa openjdk, esim. openjdk 25:

	sudo apt install openjdk-25-jdk

Lataa Ghidra sen GitHub-repositoriosta: `https://github.com/NationalSecurityAgency/ghidra/releases`. Valitse viimeisin release zip, esim. `ghidra_12.1.4_PUBLIC_20260921.zip`. Ladattua, tee esim. `apps` tai `tools` kansio, ja siirrä tiedosto sinne. Pura.

Purettuasi, kansiossa pitäisi olla esim. `ghidra_12.1.4_PUBLIC` kansio, jonka sisältä löytyy `ghidraRun` skripti. Sen käynnistämällä käynnistät Ghidran. Nimeän itse kansion vielä nimellä "ghidra", käytön helpottamiseksi.

### b) rever-C
Tehdään näitä tehtäviä varten kansio.

	cd ~
	mkdir h4 && cd h4

Ladataan ezbin-challenges.zip. (Tämä sama on ladattu jo useamman kerran eri tehtävissä, ladataan silti uusiksi). 

	wget https://terokarvinen.com/loota/yctjx7/ezbin-challenges.zip
	unzip ezbin-challenges.zip

Siirrytään `packd`-kansioon:

	cd challenges/packd

Katsotaan file-komennolla, onko packd pakattu:

	file packd

Packd on sotkuinen, ja sisältää rivejä `$Info: This file is packed with the UPX executable packer http://upx.sf.net $
$Id: UPX 4.21 Copyright (C) 1996-2023 the UPX Team. All Rights Reserved. $`. (Huom. jos tästä haluaa nähdä hieman tarkemmin, katso tehtävä h3c) Puretaan pakkaus upx:llä:

	upx -d packd

Jos purku onnistui, upx ilmoittaa niin. Sen voi vielä tarkistaa file-komennolla.

Käynnistetään Ghidra:

	bash ~/apps/ghidra/ghidraRun

Tehdään uusi projekti ja lisätään packd-tiedosto kansioon:
* Vasemman yläkulman `File` -> `New Project` -> `Non-Shared Project` -> Valitse sijainti. Itse teen uuden kansion `g_packd` kansioon h4, kansion `challenges` viereen. Projektin nimeksi esim. sama kuin kansion nimi, g_packd. -> `Finish`.
* Klikkaa kansiota
* Vasemman yläkulman `File` -> `Import File...` -> Navigoi packd -tiedostoon. Löytyy h4->challenges->packd sisältä. -> `Select File To Import`. Jos tiedosto on käyttökelpoinen, Ghidran pitäisi huomata se heti. Esim. siis aukeavassa ikkunassa tulisi lukea `Format: Executable and Linking Format (ELF)`. 

Kaksoisklikataan tiedostoa packd, Ghidran valikossa. Ohjelman pitäisi avata uusi ikkuna, jossa se kysyy haluatko analysoida tiedoston. Hyväksy se.

Valitse seuraavaksi vasemalla olevasta valikosta `Symbol Tree` valinta `Functions`, ja valitse sieltä `main`. Ruudun oikealle puolelle pitäisi avautua `Decompile: main - (packd)` ikkuna. 

<img width="1919" height="744" alt="Ghidra window" src="https://github.com/user-attachments/assets/e4e2efc0-6a4d-462b-97bf-026d4b0dde6b" />

Entuudestaan tiedetään jo, että C-kielessä main-funktio on aina int-tyyppinen. Voidaan siis vaihtaa `undefined8` -> `int`. Tyypin voi vaihtaa oikealla klikkauksella ja valitsemalla `Retype Return`. Kirjoita kenttään `int` ja valitse `int - packd/int`.

<img width="605" height="342" alt="Changed main type to int" src="https://github.com/user-attachments/assets/d8501581-e875-4cdc-b6c5-32070b6cd866" />

Seuraavaksi voi kiinnittää huomiota määritettyihin muuttujiin:
* `int iVar1` tallennettu yksi int-tyyppinen muuttuja, arvoa ei vielä määritelty. Hieman myöhäisemmässä vaiheessa koodia huomataankin, että siihen laitetaan muuttuja strcmp -funktiosta (string comparison, tekstien vertailu). Tulosta verrataan nollaan, ja annetaan oikean salasanan vastaus ja lippu, jos iVar1 on 0. Muuttuja on siis yksinkertaisesti salasanan vertauksen tuloksen tallennus. Muutetaan sen nimeksi vaikkapa `cmpResult`.
* `char local_28 [32]` tarkoittaa char tyyppistä tekstitaulukkoa, eli merkkijonoa. Maksimipituus on 32. Myöhemmin `scanf` funktio näyttää ottavan käyttäjältä syötteen, ja asettavan sen local_28:aan. local_28 on siis syötteemme, vaihdetaan sen nimeksi `input`.

<img width="598" height="347" alt="Changed variable names" src="https://github.com/user-attachments/assets/4cab7d27-abb9-4ae8-9a98-0a2c959f5cc7" />.

Nyt ohjelman toiminnallisuus on selvä.
* Rivillä 10, ohjelma käyttää `puts` funktiota salasanan kysymistä varten. `puts` on käytännössä printf, vähäisemmällä muokkauksella ja automaattisella uuden rivin merkillä `\n` (Lähde: geeksforgeeks: puts vs printf).
* Rivi 11 ottaa vastaan käyttäjän syötteen, asettaa sen muuttujaan `input`.
* Rivi 12 vertaa syötettämme tekstiin `piilos-AnAnAs`, joka on ohjelman salasana. Tämä tulos asetetaan muuttujaan `cmpResult`.
* Rivi 13 katsoo, onko `cmpResult` yhtäsuuri kuin 0. Tuloksena voi olla negatiivinen, nolla tai positiivinen luku. Se perustuu `strcmp`n tulokseen. Lähteestä voi lukea lisää. (Lähde: geeksforgeeks: strcmp). 
* Rivit 14-18 sisältävät if-else logiikan loput osat, tulostetaan onnistunut tulos jos vertailun tulos on 0, muuten `"Sorry, no bonus."`.

Salasanan tulisi siis olla `piilos-AnAnAs`. Varmistetaan se suorittamalla ohjelma:

	./packd

<img width="707" height="101" alt="Proof of running the application" src="https://github.com/user-attachments/assets/51007f12-67d4-43b6-a0cb-df4fe8f545b2" />

Salasana toimi. Salasana siis `piilos-AnAnAs`, lippu: `FLAG{Tero-0e3bed0a89d8851da933c64fefad4ff2}`.


### c) If takaperin.

Aloitetaan siirtymällä `passtr` kansioon.

	cd ~/h4/challenges/passtr/

Suoritetaan ohjelma

	./passtr

<img width="447" height="103" alt="ran passtr program" src="https://github.com/user-attachments/assets/460a4c4e-91cf-48d3-8603-7122e33c895a" />

Nyt tiedetään, että ainakaan `salaisina` ei käy salasanaksi. Käännetään seuraavaksi ohjelma taas Ghidraan.

Tehdään uusi projekti ja lisätään passtr-tiedosto kansioon:
* Vasemman yläkulman `File` -> `New Project` -> `Non-Shared Project` -> Valitse sijainti. Itse teen uuden kansion `g_passtr` kansioon h4, kansion `challenges` viereen. Projektin nimeksi esim. sama kuin kansion nimi, g_passtr. -> `Finish`.
* Klikkaa kansiota
* Vasemman yläkulman `File` -> `Import File...` -> Navigoi passtr -tiedostoon. Löytyy h4->challenges->passtr sisältä. -> `Select File To Import`. Jos tiedosto on käyttökelpoinen, Ghidran pitäisi huomata se heti. Esim. siis aukeavassa ikkunassa tulisi lukea `Format: Executable and Linking Format (ELF)`. 

Kaksoisklikataan tiedostoa passtr, Ghidran valikossa. Ohjelman pitäisi avata uusi ikkuna, jossa se kysyy haluatko analysoida tiedoston. Hyväksy se.

Valitse seuraavaksi vasemalla olevasta valikosta `Symbol Tree` valinta `Functions`, ja valitse sieltä `main`. Ruudun oikealle puolelle pitäisi avautua `Decompile: main - (passtr)` ikkuna.

<img width="1919" height="737" alt="Passtr open with ghidra" src="https://github.com/user-attachments/assets/34eb3312-001c-46e0-8724-08bf6d324ea4" />

Funktio on käytännössä sama, verrattuna edelliseen. Tehdään siis selkeyden vuoksi samat muutokset. Main-tyypiksi `int`, muuttuja `iVar1` -> `cmpResult`, `local_28` -> `input`. 

<img width="605" height="345" alt="Changed basic things in decompiled passtr" src="https://github.com/user-attachments/assets/69b8ecad-f4ce-48cb-8e12-e259a60bb771" />

Nyt etsitään kohta, missä if-ehto tapahtuu. Klikataan disassembly-ikkunassa riville, jossa if-ehto tapahtuu (rivi 13). Keskimäisessä ikkunassa `Listing: passtr` tulisi olla listaus eri käskyjä. if-ehtoa klikatessa, kutakuinkin rivi `        001011a3 75 11           JNZ        LAB_001011b6` tulisi olla korostettu. 

<img width="1641" height="601" alt="image" src="https://github.com/user-attachments/assets/50299a61-4882-4c34-896a-b54d965c6a41" />

`JNZ` tarkoittaa ohjeessa "jump not zero", eli siirry tiettyyn kohtaan, jos tulos ei ole nolla. Tietty kohta on tässä tapauksessa `LAB_001011b6`, joka löytyy kuvassakin alempana. Tällä siis ohitettaisiin kohta, jossa annetaan ulos `puts`-funktiolla haluttu tulos. Voidaan siis päätellä, että tämän haluamme kääntää päälaelleen. Vaihdetaan ohje `JNZ` (jump not zero) sen sijaan ohjeeksi `JZ` (jump zero).

Klikataan kohtaa `JNZ` ja valitaan `Patch instruction`, ja poistetaan JNZ:stä N-kirjain. Painetaan enter. Tämän jälkeen, tulos muuttui myös `Decompile`-ikkunassa:

<img width="1639" height="615" alt="Patched JNZ to JZ" src="https://github.com/user-attachments/assets/b5f68129-fabb-41c4-b78a-e88f5ba51232" />

Oikea ja väärä tulos ovat vaihtaneet paikkaa. Työ on tehty, nyt pitää tallentaa sovellus ja kokeilla! 

Valitaan `File` valikosta `Export Program`. Valitaan tyypiksi `Original File`. Jos tallennuspolkua ei muuttanut, se on nyt juuressa. Kokeillaan eri salasanoja!

	~/passtr

<img width="706" height="251" alt="Proof that wrong passwords now work, correct doesn't" src="https://github.com/user-attachments/assets/4146ed4a-d8cd-4093-ba74-84f89e830dec" />

Kuvasta voidaan nähdä, kaksi ensimmäistä selkeästi väärää salasanaa antoivat halutun tuloksen, kun taas koodista löydetty `sala-hakkeri-321` ei toiminut. If-ehdon kääntö siis onnistui. 

Oikea salasana alussa `sala-hakkeri-321`, muokatussa binäärissä kaikki muut salasanat paitsi `sala-hakkeri-321`. Lippu: `FLAG{Tero-d75ee66af0a68663f15539ec0f46e3b1}`. 

### d) Käännä crackme binäärit

Ladataan ensin tiedostot.

	cd ~/h4
	git clone https://github.com/NoraCodes/crackmes.git
	cd crackmes

<img width="684" height="101" alt="downloaded git repository files" src="https://github.com/user-attachments/assets/079a75dc-c8dc-4aa0-8f15-15c2adcbdbd9" />


Nyt meillä on crackme01 - crackme09 tehtävät. Ladatut tiedostot sisältävät myös Makefilen, jolla voi kääntää lähdekoodit binääreiksi.

	make

Nyt kaikista lähdekoodeista pitäisi olla käännetyt versiot.

<img width="739" height="179" alt="Compiled versions of Nora crackmes" src="https://github.com/user-attachments/assets/587ebc90-dd5b-4826-9ac2-0754ba96d187" />

### e) Nora crackme01 & crackme01e

Ohjelmat `crackme01` ja `crackme01e` voisi ratkaista esim. Ghidralla, mutta siinä ei ole mitään järkeä. Tulokset ovat helposti saatavilla pelkällä `strings` komennolla. Ghidrassakin salasanat olisivat selkeästi esillä. Kuvissa strings-esimerkit:

<img width="463" height="389" alt="strings crackme01" src="https://github.com/user-attachments/assets/1cde45e2-127d-46d0-89e3-4af085f7e7a2" />
<img width="469" height="370" alt="strings crackme01e" src="https://github.com/user-attachments/assets/d68597ea-ede5-4a4a-976c-f22e50c4fbf9" />

Esimerkki myös Ghidrasta, binääri `crackme01e`:

<img width="347" height="464" alt="crackme01e ghidra decompile code" src="https://github.com/user-attachments/assets/bff79789-47dc-4054-b0ee-88f63cb4719e" />

Kuvassa rivillä 11 löytyy myös tyypillinen strncmp, ja sen sisältä kovakoodattu salasana.

Salasanat siis:
* `crackme01` = `password1`
* `crackme01e` = `slm!paas.k` 

<img width="542" height="101" alt="crackme1 and crackme01e proof" src="https://github.com/user-attachments/assets/7e7a6c77-0200-4db4-8367-3130a12e57fc" />



## Lähteet
* Karvinen, tehtävänanto. Luettavissa: https://terokarvinen.com/application-hacking/#homework
* Hammond 2022, GHIDRA video kohtaan x. Katsottavissa: https://www.youtube.com/watch?v=oTD_ki86c9I
* Karvinen, ezbin-challenges.zip. Ladattavissa: https://terokarvinen.com/loota/yctjx7/ezbin-challenges.zip
* Tindall 2023, NoraCodes / crackmes. Ladattavissa: github.com/NoraCodes/crackmes
* geeksforgeeks: puts vs printf. Luettavissa: https://www.geeksforgeeks.org/c/puts-vs-printf-for-printing-a-string/
* geeksforgeeks: strcmp. Luettavissa: https://www.geeksforgeeks.org/c/strcmp-in-c/
