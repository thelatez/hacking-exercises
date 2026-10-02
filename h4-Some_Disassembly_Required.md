# h4 - Some Disassembly Required

## Tehtävänanto
* x) Katso video (Hammond 2022, GHIDRA for Reverse Engineering (PicoCTF 2022 #42 'bbbloat')), tiivistä.
* a) Lataa Ghidra
* b) Rever-C: käänteismallinna `packd` binääri C-kieleksi Ghidralla. Löydä pääohjelma. Anna muuttujille kuvaavat nimet, selitä ohjelman toiminnallisuus. Ratkaise tehtävä binääristä, ilman lähdekoodia.
* c) If takaperin: muokkaa `passtr` ohjelman binääri niin, että se hyväksyy kaikki salasanat paitsi oikean. Todista.
* d) Nora CrackMe: binäärit Tindall 2023: NoraCodes / crackmes. https://github.com/NoraCodes/crackmes. 
* e) Nora crackme01. Ratkaise binääri.
* f) Nora crackme01e. Ratkaise binääri.
* g) Nora crackme02. Nimeä ohjelman muuttujat käänteismallinnetusta binääristä, ja selitä sen toiminnallisuus. Ratkaise binääri.

HUOM. alkuperäisessä tehtävänannossa (https://terokarvinen.com/application-hacking/#homework) kohtani e+f ovat molemmat e, varmaankin virhe. Ratkaisuni ei siis sisällä vapaaehtoisia tehtäviä, vain eroteltu kaksi e kohtaa toisistaan.

## Vastaus

### x) 
Hammond 2022, GHIDRA for Reverse Engineering (PicoCTF 2022 #42 'bbbloat'):
* Ghidran ylävalikossa "Window" on mielenkiintoinen työkalu "Defined Strings". Tällä saattaa olla helpompi löytää esimerkiksi printattujen kysymysten sijanti binäärissä.
* Jos printatun tekstin perässä on `XREF... : FUN` se tarkoittaa viittausta funktioon. Sitä tuplaklikkaamalla pääsee käskyriville (mitä esim. gdb:llä katsottiin h5).

### a)
Lataa openjdk, esim. openjdk 25:

	sudo apt install openjdk-25-jdk

Lataa Ghidra sen GitHub-repositoriosta: `https://github.com/NationalSecurityAgency/ghidra/releases`. Valitse viimeisin release zip, esim. `ghidra_12.1.4_PUBLIC_20260921.zip`. Ladattua, tee esim. `apps` tai `tools` kansio, ja siirrä tiedosto sinne. Pura.

Purettuasi, kansiossa pitäisi olla esim. `ghidra_12.1.4_PUBLIC` kansio, jonka sisältä löytyy `ghidraRun` skripti. Sen käynnistämällä käynnistät Ghidran. Nimeän itse kansion vielä nimellä "ghidra", käytön helpottamiseksi.

### b)
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
$Id: UPX 4.21 Copyright (C) 1996-2023 the UPX Team. All Rights Reserved. $`. Puretaan pakkaus upx:llä:

	upx -d packd

Jos purku onnistui, upx ilmoittaa niin. Sen voi vielä tarkistaa file-komennolla.

Käynnistetään Ghidra:

	bash ~/apps/ghidra/ghidraRun

Tehdään uusi projekti ja lisätään packd-tiedosto kansioon:
* Vasemman yläkulman `File` -> `New Project` -> `Non-Shared Project` -> Valitse sijainti. Itse teen uuden kansion `g_packd` kansioon h4, kansion `challenges` viereen. Projektin nimeksi esim. sama kuin kansion nimi, g_packd. -> `Finish`.
* Klikkaa kansiota
* Vasemman yläkulman `File` -> `Import File...` -> Navigoi packd -tiedostoon. Löytyy h4->challenges->packd sisältä. -> `Select File To Import`. Jos tiedosto on käyttökelpoinen, Ghidran pitäisi huomata se heti. Esim. siis aukeavassa ikkunassa tulisi lukea `Format: Executable and Linking Format (ELF)`. 

## Lähteet
* Karvinen, tehtävänanto. Luettavissa: https://terokarvinen.com/application-hacking/#homework
* Hammond 2022, GHIDRA video kohtaan x. Katsottavissa: https://www.youtube.com/watch?v=oTD_ki86c9I
* Karvinen, ezbin-challenges.zip. Ladattavissa: https://terokarvinen.com/loota/yctjx7/ezbin-challenges.zip
* Tindall 2023, NoraCodes / crackmes. Ladattavissa: github.com/NoraCodes/crackmes
