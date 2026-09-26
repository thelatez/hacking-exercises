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

<img width="937" height="346" alt="image" src="https://github.com/user-attachments/assets/5ef738cb-bff7-4d67-af8f-7bffd0bb94c4" />

Koodista löytyy suoraan strcmp (stringien vertailu) ja siinä vertaus oikeaan salasanaan. Meidän pitää vähintäänkin sotkea salasanaa niin, että sitä ei saa niin helposti selville.

Tehdään sotkeminen, joka tekee seuraavaa koodissa:
* Vertaa syötteen kokoa oikeaan salasanaan. Jos koko on eri, salasana on heti väärin. Tämä ehto on pakko olla, koska muuten seuraavia ehtoja voisi huijata.
* Lisätään oikeaan salasanaan joka toiseen ylimääräinen kirjain.
* Käydään oikea salasana kirjain kerralta läpi niin, että joka toinen ohitetaan täysin. Jos kirjainta ei ohiteta (eli oikea kirjain), verrataan sitä käyttäjän salasanaan. Jos kirjaimet eriävät, salasana ei ole sama. Jos kaikki kirjaimet ovat samoja, lopetamme loopin päätösmerkkiin `\0` tuloksella sama salasana.

Muutettu koodi:




## Lähteet:
* Karvinen 2026. Tehtävänanto. Luettavissa: https://terokarvinen.com/application-hacking/#homework
* Karvinen, ezbin-challenges.zip. Ladattavissa: https://terokarvinen.com/loota/yctjx7/ezbin-challenges.zip
