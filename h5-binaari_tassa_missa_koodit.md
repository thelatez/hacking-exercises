# h5 - Binääri tässä, missä koodit? 

### Tehtävänanto
* Lab0: Etsi virhe, korjaa ja kerro miten.
* Lab1: Etsi miksi ohjelma kaatuu, voiko sen korjata?
* Lab2: Ohjelma ilman lähdekoodia. Etsi kysymä uusi salasana ja ohjelman tulostama lippu. Selitä miten sai selville ja mitä uutta oppi GNU Debuggerista.
* Lab3: Nora Crackme -haasteita. Valitse yksi tiedosto ja yritä ratkaista binäärin salasana. Miten salasanan sai selville?
* Lab4 (Vapaaehtoinen): Ratkaise binäärin salasana ja kerro miten sai selville.

# Vastaus

## Lab0:

Lab0:n ongelma on ylivuotava for-loop. Ongelma muodostuu siitä, että parametrinä annettu listan koko "size" kertoo taulukon (array) elementtien määrän, kun taas for-loop, joka aloittaa arvosta 0, laskee elementtien sijaintia taulukossa (5-kokoisen taulukon arvot ovat 0-4, eikä 1-5). 

Alla kuvassa näkyy todisteet: "Hardware watchpoint 4: i" tarkoittaa for-loopin i:n arvoa, joka alkaa 0:sta. Esimerkiksi i:n arvolla 4, on meillä taulukossa arvo 5. Arvo 5 on siis taulukon neljäs elementti. Sen vois myös huomata kohdasta: "Element 4: 5". Kuvasta voidaan myös huomata se, että kun mennään taulukon yli, i:n arvoon 5, elementin arvo on "0", jota ei löydy annetusta taulukosta.  

<img width="915" height="515" alt="image" src="https://github.com/user-attachments/assets/4e6270c4-7d0d-442c-a2a3-8ebfa3c7c5c7" />

Koodin voi korjata muutamalla tavalla, mutta tässä miten sen tein. "for(int i = 0; i <= size; i++)" voidaan vaihtaa:
* "for (int i = 0; **i < size**; i++)"

Muutoksen jälkeen ohjelman suorittamisen tulos:

<img width="823" height="509" alt="image" src="https://github.com/user-attachments/assets/4bb3c5ed-f934-4114-88fa-f62d745b1919" />

Ohjelma toimii nyt halutulla tavalla. 


## Lab1:

<img width="502" height="535" alt="image" src="https://github.com/user-attachments/assets/a6fb6766-1723-400c-b8fc-cbc565f6092a" />

Alkuun katsoin ohjelman lähdekoodia komennolla: "more gdb_example1.c". Sieltä koodin toiminallisuus ja virhe eivät tulleet heti selviksi, joten suoritin myös ohjelman komennolla "./gdb.example1", jonka tuloksena oli: 
    
    Khoor/#zruog1
    Segmentation fault

Nopealla silmäyksellä koodista ja tulostuksesta ilmenee selvitettävää: "register" -avainsana, bad_messagen arvo NULL ja "Segmentation fault" tuloksessa. Segmentation fault ei ole koodin toiminnallisuus, jolloin tämä kertoo siis virheestä, joka koodissa tapahtuu.
* Register-avainsana tarkoittaa, että se muuttuja tallennetaan mieluummin esimerkiksi prosessorin rekisteriin, eikä RAM-muistiin. Käyttötarkoituksena nopeuttaa arvon hakua muistista, ja ei pitäisi olla merkitystä tehtävään.
* NULL-arvo tarkoittaa tyhjää arvoa. Koska tehtävässä NULL ei ole "NULL", se on siis myös avainsana, eikä tekstiä kuten good_messagen arvo. Tämä aiheuttaa lähes varmasti segmentation faultin.
* Segmentation fault tarkoittaa käytännössä sitä, että ohjelma yrittää esim. päästä sisään muistiin paikkaan, johon sillä ei ole oikeutta. Koska segmentation fault ilmenee NULL -arvon yhteydessä, ohjelmalla ei siis ole pääsyä sinne, missä NULL arvo sijaitsee muistissa. 

Selvittääkseni mistä "Segmentation fault" johtui, kokeilin myös suorittaa ohjelman ilman "bad_message", sekä eri "register int i":n arvolla. Esimerkiski i = 123:n tulos oli: "����꧛����ߩ", ja muutaman muun oli esimerkiksi: "Jgnnq."yqtnf0" ja "Rovvy6*�y|vn8". Sain tästä selville sen, että i:n arvolla on väliä tulostuksen sisällölle. 



### Lähteet
Lab1: https://en.wikipedia.org/wiki/Register_(keyword) (mikä on register-avainsana)
Lab1: https://www.scaler.com/topics/segmentation-fault-in-c-cpp/ (mikä on segmentation fault)
