# h3 - No Strings Attached

## Tehtävänanto:
* a) Lataa "ezbin-challenges.zip" (https://terokarvinen.com/loota/yctjx7/ezbin-challenges.zip). Aja "passtr" ja löydä oikea salasana ja lippu "strings" komennolla
* b) Tee uusi versio passtr.c ohjelmasta, jossa salasana ei ole selvästi saatavilla strings-komennolla (obfuskointi riittää)
* c) Aja "packd" paketista ezbin-challenges. Mikä salasana ja flag? Kirjoita ylös yritykset ja hypoteesit.

## Vastaus:
### Prep) 
Aloitetaan ensin tekemällä uusi hakemisto h3 ja siirtymällä sen sisään.

	mkdir h3 && cd h3

Ladataan ezbin-challenges:

	wget https://terokarvinen.com/loota/yctjx7/ezbin-challenges.zip

Puretaan ezbin:

	unzip ezbin-challenges.zip

Siirrytään puretun kansion challenges sisään:

	cd challenges

### a)
Siirrytään passtr:

	cd passtr

Ajetaan ensin ohjelma, että tiedämme mitä voisimme etsiä:

	./passtr

<img width="438" height="95" alt="Passtr application functionality" src="https://github.com/user-attachments/assets/1fa2a0a2-51b6-493d-ad02-5ac1c4ed1e63" />

Sovellus kysyy salasanaa, ja sitten kertoo onko se oikein. Voisimme siis löytää strings-komennolla salasanan, jos sitä ei ole sekoitettu yms millään tavalla.

Ajetaan strings komento passtr-tiedostoon:

	strings passtr

<img width="703" height="540" alt="strings passtr output" src="https://github.com/user-attachments/assets/8d53161f-7306-49c5-b512-cdd1dbfe8fd2" />

<sub>Ei sisällä kaikkea strings komennon ulostulosta</sub>

Sattumoisin listan alkupäästä löytyy muun muassa sovelluksen kysely `What's the password?`, syötteen määrittävä yksikkö `%19s` (korkeintaan 19 merkkiä, kertoo luultavasti syötteen maksimipituuden), `sala-hakkeri-321` (epäilyttävä, kenties salasana?) ja muun muassa `Yes! That's the password. FLAG{Tero-d75ee66af0a68663f15539ec0f46e3b1}` (luultavasti ulostulo eli print oikealla salasanalla). 

Tästä voisi siis päätellä, että kyselyn jälkeen annetaan korkeintaan 19 merkkiä pitkä salasana, joka liittyy tavalla tai toisella tekstiin "sala-hakkeri-321", ja sitten saadaan syötteeksi "Yes! ..." tai "Sorry, no bonus". Kokeillaan syöttää sovelluksen salasanaksi "sala-hakkeri-321":

<img width="702" height="95" alt="Input sala-hakkeri-321 as password to passtr" src="https://github.com/user-attachments/assets/e9b81521-dd76-43c7-81bf-b601e66d8297" />

`sala-hakkeri-321` on nähtävästi oikea salasana ja lippu: `FLAG{Tero-d75ee66af0a68663f15539ec0f46e3b1}`, ei sen monimutkaisempi asia.

### b)
Avataan `passtr.c` koodieditoriin:

	micro passtr.c

<img width="937" height="346" alt="passtr.c original contents" src="https://github.com/user-attachments/assets/5ef738cb-bff7-4d67-af8f-7bffd0bb94c4" />

Koodista löytyy suoraan strcmp (stringien vertailu) ja siinä vertaus oikeaan salasanaan. Meidän pitää vähintäänkin sotkea salasanaa niin, että sitä ei saa niin helposti selville.

Tehdään sotkeminen, joka tekee seuraavaa koodissa:
* Vertaa syötteen kokoa oikeaan salasanaan. Jos koko on eri, salasana on väärin. Tämä ehto on pakko olla, koska muuten seuraavia ehtoja voisi huijata.
* Lisätään oikeaan salasanaan joka toiseen ylimääräinen kirjain.
* Käydään oikea salasana kirjain kerralta läpi niin, että joka toinen ohitetaan täysin. Jos kirjainta ei ohiteta (eli oikea kirjain), verrataan sitä käyttäjän salasanaan. Jos kirjaimet eriävät, salasana ei ole sama ja lopetetaan loop siihen. Jos kaikki kirjaimet ovat samoja, loppuu loop vasta päätösmerkkiin `\0`, joka tarkoittaa oikeaa salasanaa.

Muutettu koodi:

<img width="938" height="647" alt="Revised code for passtr.c" src="https://github.com/user-attachments/assets/38f22ae3-4a39-4485-b526-22e83de3c8b3" />

<sub>Osaan koodista käytetty ilmaista ChatGPT-tekoälymallia</sub>

Selitykset:
* Lisätään valmiiksi sotkettu "oikea" salasana pysyväksi muuttujaksi `const char *correctpw = "sla0l6a2-zhka0kbk2etrki--238251"`. Tämä löytyy siis strings-komennolla, mutta se ei ole sellaisenaan oikea tulos: <img width="439" height="94" alt="Obfuscated code does not pass as password" src="https://github.com/user-attachments/assets/ccdc36e8-5a6c-40ba-afb9-a3d027463c73" />
* `int correct = 1` toimii kirjanpitona. Jos salasana ei täsmää jossain ohjelman kohdassa, vaihdetaan arvoksi 0. Jos arvo on lopussa 1, salasana on oikein, muuten se on väärin.
* for-loop alkaa kohdasta `i = 0` ja jatkuu niin kauan, kun ehto `correctpw[i] != '\0'`, eli niin kauan kuin `correctpw`n merkki kohdassa `i` ei ole päätösmerkki `\0`. `if (input[i/2] != correctpw[i]` katsoo, onko syötteen arvo kohdassa `i` jaettuna kahdella sama, kuin "oikean" salasanan arvo kohdassa `i`. Syötteen arvo jaetaan kahdella, koska oikeassa salasanassa on puolet virheellisiä merkkejä, joka toinen. Jos ehto täyttyy (eli merkit eivät ole samat), asetetaan `correct` arvoksi 0, eli väärä salasana, ja lopetetaan for-loop heti `break` avainsanalla. Jos ehto ei täyty (eli merkit ovat samat), jatketaan suoritusta.
* `if (strlen(input) != 1+strlen(correctpw)/2` katsoo, onko syötetyn salasanan ja "oikean salasanan" arvo jaettuna kahdella samat. Oikeaan salasanaan lisätään 1, koska sotkumerkkejä ei ole jokaisen merkin edestä, vaan yksi vähemmän (vikan oikean merkin jälkeen ei tule sotkumerkkiä).
* Lopun if-rakenteessa vain katsotaan, onko correct arvo yhä 1. Jos on, annetaan oikean salasanan vastaus.

Vielä todiste, että ohjelma toimii yhä samalla salasanalla:

<img width="981" height="762" alt="Proof that password still works" src="https://github.com/user-attachments/assets/1cce3844-8570-42ac-8ad9-6ee8b6bac73d" />

Ja strings-komennon sisältö:

<img width="703" height="419" alt="Proof that password is scrambled with strings" src="https://github.com/user-attachments/assets/72d52598-813e-490e-a8f9-fe753670e4f0" />

Muut "ongelmat", miksi tämä riittää palautukseen, mutta ei realistisesti estä salasanan löytämistä millään tavalla:
* Lippu (flag) löytyy yhä strings-komennolla, koska sitä ei edes sotkettu millään tavalla. Jos pitäisi löytää vain lippu tai jos se mainitsisi syystä tai toisesta oikean salasanan, olisi sotku turhaa.
* Tämä ei tietenkään lisää turvallisuutta, vain karkoittaa helpoimmat tavat murtautua pois.
* Sotku ei ole monimutkainen. Varsinkin, kun salasana on yhä selkokieltä (sama `sala-hakkeri-321`), on se helpohko huomata.
* Vaikka sotku olisi monimutkaisempi (esim. h5 tehtävän binäärit), voi niitä yhä ratkoa seuraamalla esim. konekoodia ja rekistereitä.

### c)
Aloitetaan siirtymällä packd-kansioon:

	cd ~/h3/challenges/packd

Suoritetaan packd ohjelma:

	./packd

<img width="419" height="80" alt="packd program functionality" src="https://github.com/user-attachments/assets/16da5808-0458-48b6-985a-e594c95f6c1b" />

Toiminnallisuus vaikuttaa olevan samanlainen kuin edellinen tehtävä: kysyy salasanan ja kertoo onko se väärin/oikein.

Katsotaan seuraavaksi strings-komennolla packd-tiedostoa:

	strings packd

<img width="786" height="762" alt="strings packd results" src="https://github.com/user-attachments/assets/f3f66a14-9158-4129-b9ed-4ae839d04b16" />

Tulos on mielenkiintoinen. Sovellusta on selvästi sotkettu nyt vahvemmin. Jopa alussa olevat selkeämmät asiat kuten kirjastot tai paketit ovat sotkuisia. Strings-komennolla tulee ilmi jotakin salasanaan viittaavaa, kuten `piilos-An`, mutta se ei riitä. Syötettä on jotenkin leikattu, nähtävästi joskus alusta, joskus lopusta. Kiinnitänkin enemmän huomiota siis riveihin `$Info: This file is packed with the UPX executable packer http://upx.sf.net $
$Id: UPX 4.21 Copyright (C) 1996-2023 the UPX Team. All Rights Reserved. $`. Tämä viittaisi siihen, että tiedosto todellakin on kompressoitu, käyttäen "UPX executable packer"ia. Etsitään netistä, voiko UPX kompressoiduille tiedostoille tehdä jotain (https://github.com/upx/upx). 

Tiedostoja pitäisi pystyä myös dekompressoimaan joten ladataan UPX:

	sudo apt install upx

Kokeillaan sen jälkeen dekompressoida:

	upx -d packd

<img width="859" height="192" alt="decompressing UPX compressed file packd" src="https://github.com/user-attachments/assets/42ee8f37-7a7a-4ce3-bdd1-54355f3966ad" />

Nähtävästi dekompressointi toimi. Katsotaan, muuttiko dekompressointi mitään:

	strings packd

<img width="728" height="419" alt="Strings packd after decompression" src="https://github.com/user-attachments/assets/16800ec5-b24e-4dfb-b66b-4cb232b90145" />

Nyt rakenne näyttää tutulta. Näyttäisi siltä, että salasana on kenties `piilos-AnAnAs`. Kokeillaan sitä:

	./packd

<img width="701" height="102" alt="piilos-AnAnAs as password for packd" src="https://github.com/user-attachments/assets/bac87427-72ce-457d-8102-cd325c603560" />

Nähtävästi tehtävä oli siinä. Salasana `piilos-AnAnAs`, lippu: `FLAG{Tero-0e3bed0a89d8851da933c64fefad4ff2}`. 

Fiilis kohdasta c: oletin tehtäväkuvauksen perusteella vaikeampaa tehtävää ("This task is slightly more challenging. Write down the approaches you tried and hypotheses you came up with. Hopefully you'll reach the goal yourself, but if not, the walkthrough will be revealed in class..."). En tiedä onko hyvä vai huono juttu, että tässä ei tarvinnut jauhaa pidempään.

## Lähteet:
* Karvinen 2026. Tehtävänanto. Luettavissa: https://terokarvinen.com/application-hacking/#homework
* Karvinen, ezbin-challenges.zip. Ladattavissa: https://terokarvinen.com/loota/yctjx7/ezbin-challenges.zip
* OpenAI, ilmainen ChatGPT-laaja kielimalli. Käytetty luomaan osa kohdan b koodista. Käytetty 26.9.2026. Saatavilla: https://chatgpt.com/
* UPX, tiedostojen kompressoija. Luettavissa: https://github.com/upx/upx
