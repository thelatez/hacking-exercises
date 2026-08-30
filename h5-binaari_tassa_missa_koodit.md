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


