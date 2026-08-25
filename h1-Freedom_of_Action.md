# h1 - Freedom of Action, Control, and Risk Mitigation

### Tehtäväkuvaus
a1) Mitä kuuluu ISMS laajuuteen (kodin verkkoinfrastruktuuri, laitteet, data)

a2) Mitä ei sisällytetä ISMS laajuuteen

a3) Avainliitännät ja rajat (esim. pilvipalvelut, etäyhteydet, rajat kotiverkon ja internetin välillä, palveluntarjoajat)

b) Tunnista vähintään kaksi (2) kodin verkosta kiinnostunutta osapuolta.

## Vastaus

### ISMS laajuus

Sovellusten hakkerointi ja haavoittuvuudet - ICI012AS3A-3004, Lauri Rantala, 25.8.2026.

a1) ISMS laajuuteen kuuluu kotiympäristössäni: 
* Kodin verkkoinfrastruktuuri:
    * Wi-Fi reititin, jotta opiskeluun voidaan käyttää verkkoyhteyttä
* Kurssilaitteet:
    * Kannettava tietokone, jolla pyöritetään Linux virtuaalikonetta
    * Linux virtuaalikone, jolla tehdään tehtävät
    * henkilökohtainen älypuhelin, jolla hoidetaan MFA ja autentikaatio
* Informaatio ja data:
    * Kurssimateriaalit
    * Repositoriot, jotka sisältävät esimerkiksi kurssin tehtäviä ja vastauksia
    * Käyttäjätiedot, joita käytetään kurssin tehtävien suorittamiseen

a2) ISMS laajuuteen ei sisällytetä:
* Perheenjäsenten henkilökohtaiset laitteet:
    * Puhelimet, tabletit tai tietokoneet, jotka jokin toinen perheenjäsen omistaa
    * Syy: ei hallittavissa, ei merkityksellinen kurssille
* Pääasiassa viihdettä suorittavat laitteet:
    * Älytelevisiot, pelikonsolit
    * Syy: ei hyödynnetä kurssin työskentelyssä, ylläpidetään erikseen opiskeluun liittyvistä turvallisuusasioista
* Töihin liittyvät laitteet:
    * Työpaikalta saatu tietokone ja puhelin
    * Syy: hallinta ja vastuu kuuluu työpaikan omalle turvallisuuspolitiikalle
* Internet-palveluntarjoajan verkkoinfrastruktuuri:
    * Julkinen verkko kodin verkon ulkopuolella
    * Syy: hallinta kuuluu palveluntarjoajalle (Telia)
  
a3)

### Mitä todisteita voisi olla?
* Verkkoinfrastruktuurista voisi ottaa kuvaa esimerkiksi reitittimen kotisivusta, jossa näkyy liikenteen kulkua ja yleiskatsaus esimerkiksi laitemäärästä. Tällä voisi myös todistaa, että laitteita on myös scopen ulkopuolelta.
* Kurssilaitteista voisi ottaa esimerkiksi yksilölliset laitetunnukset ja MAC-osoitteet. MAC-osoitteet pystyy yhdistämään reitittimen laitesivulla myös yhdistettyihin laitteisiin. Linux virtuaalikoneesta pystyisi ottamaan kuvaan esimerkiksi VirtualBoxin käyttöliittymästä, missä näkyy kaikki laitteen virtuaalikoneet.
* Informaatiosta ja datasta pystyisi ottamaan esimerkiksi tämän GitHub repositorion linkin, käyttäjänimiä tileistä tai jonkinlaista käyttöliittymäkuvaa, missä näkyy useampi käytetty sähköpostiosoite.

b)
| Sidosryhmä                     | Tarve/Vaatimus                                                                                                                   | ISO 27001 vaatimusalue            | Todisteet |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- | --------- |
| Minä                           | Saatavuus & Datan pysyvyys: Kurssin suorittaminen ympäristössä on mahdollista, tiedostot pysyvät kunnossa ja saatavilla          | Suunnittelu, Toiminta             | Pilvitallennusten logit, commit-historia |
| Perheenjäsenet                 | Yksityisyys & Saatavuus: Perheenjäsenten data ei vaaraannu työskentelyssä, kurssin suorittaminen ei häiritse normaalia toimintaa | Toiminta, Suorituskyvyn arviointi | Tiedonsiirtorajojen asetukset |
| Haaga-Helia ammattikorkeakoulu | Vaatimustenmukaisuus & Turvallisuus: Koulun ja kurssin sääntöjen mukainen työskentely                                            | Konteksti, Johtajuus              | Palautetut tehtävät, hyväksytty sopimus kurssin alussa, hyväksytty sopimus säännöistä koulun kanssa |
| Yleinen virkavalta             | Lainmukaisuus: Työskentely noudattaa lakia, esim. hakkeroinnin yhteydessä                                                        | Konteksti, Edistys                | Linux-ympäristön komentorivilogit |

### Lähteet
* https://www.digiturvamalli.fi/blogi/mika-on-isms (mikä on isms)
* https://terokarvinen.com/application-hacking/#homework (tehtävänanto, esimerkit)
* https://hightable.io/iso-27001-clause-4-2-understanding-the-needs-and-expectations-of-interested-parties/ (b-osioon apua)
* 25.8.2026 käytetty Googlen Gemini -laajaa kielimallia. Syötteenä: "I need help with an ISO 27001 compliant 'interested parties' table, explain 'ISO 27001 reference (requirement area)' and how I could prove them for 'you', 'family members', 'school', and 'general authorities' in the scope of a home network". 
