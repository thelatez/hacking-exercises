# h2 - Break & Unbreak

## Tehtävänanto
x) Lue/katso/kuuntele ja tiivistä lähteitä:
  * OWASP: OWASP Top 10: https://top10.owasp.org/2021/A01_2021-Broken_Access_Control/ (Tehtävänannossa rikkinäinen linkki, tämä toimii)
  * Karvinen 2023: Find Hidden Web Directories - Fuzz URLs with ffuf. https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/
  * Portswigger: Access control vulnerabilities and privilege escalation. https://portswigger.net/web-security/access-control
  * Karvinen 2006: Raportin kirjoittaminen. https://terokarvinen.com/2006/raportin-kirjoittaminen-4/

a) Murtaudu 010-staff-only.
  * Karvinen 2024: Hack'n Fix. https://terokarvinen.com/hack-n-fix/

b) Korjaa 010-staff-only:n vika koodista. Todista.

c) Ratkaise dirfuzt-1 artikkelista.
  * Karvinen 2023: Find Hidden Web Directories - Fuzz URLs with ffuf. https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/

d) Murtaudu 020-your-eyes-only. 
  * Karvinen 2024: Hack'n Fix. https://terokarvinen.com/hack-n-fix/

e) Korjaa 020-your-eyes-only haavoittuvuus. Todista.

## Ratkaisu
x)
  
  OWASP Top 10: A01:2021 - Broken Access Control
  * Access Control tarkoittaa ideaa, jonka mukaan käyttäjä ei saa toimia oikeuksiensa ulkopuolella.
  * Johtaa yleensä vakaviin ongelmiin, kuten henkilökohtaisen datan leviäminen, muokkaus tai poisto.
  * Parhaita tapoja estää käyttäjän pääsy oikeuksiensa ulkopuolelle: kiellä pyynnöt vakiona poislukien julkiset resurssit, rajapintojen rate limit, lyhytaikainen access token tai heti uloskirjautuessa päättyvä access token.

  Karvinen 2023: Find Hidden Web Directories - Fuzz URLs with ffuf
  * Fuff on työkalu, jolla voi "fuzzata" tien sisään piilotettuihin kansioihin, headereihin, POST parametreihin jne.
  * Penetraatiotestauksen tekniikat vaativat laki- ja etiikkapohjaista huomiointia. Ffuf:ia ei voi käyttää kohteisiin ilman kirjallista lupaa.
  * Hyvä vaihtoehto josta saa yleisiä kohteita ja sanoja joilla käyttää ffuf:ia, on "SecLists", kokoelma erilaisia sanalistoja, kuten "common".
  * Kaikki sanat johtavat hakemistossa Status 200, mutta niistä 99% ovat silti samaa kuin ilman mitään tekstiä hakemiston etsimiseksi. Pitää siis löytää muu tapa erotella kuin Status, kuten size.

  Portswigger: Access control vulnerabilities and privilege escalation
  * Verkkosovelluksissa, autentikaatio, session hallinta ja access control ovat eri asioita. Autentikaatio tarkoittaa käyttäjän todentamista siihen, kuka käyttäjä väittää olevansa. Session hallinta tarkoittaa HTTP-pyyntöjen identifioimista. Access control määrittelee onko käyttäjällä oikeus suorittaa toiminto, jota yrittää.
  * Access controllia voi harjoittaa muun muassa vertikaalisella, horisontaalilla ja kontekstiriippuvaisella tavalla.
    * Vertikaalinen: eri käyttäjätyypit pääsevät käsiin erilaisiin sovelluksen osiin, esim. admin ja peruskäyttäjä.
    * Horisontaali: rajaa pääsyä materiaaliin tietyiltä käyttäjiltä. Eri käyttäjillä on pääsy eri alaosioon resursseja.
    * Kontekstiriippuvainen: Rajaa pääsyn toimintoihin ja resursseihin sovelluksen tilan pohjalta. Tähtää siihen, että toimintoja ei voi suorittaa väärässä järjestyksessä.
  * Security by obscurity (turvallisuuden lisääminen sotkemisella) voi kuulostaa tehokkaalta arvailun estämiseksi, mutta omat viat voivat silti paljastaa sijainnin, esimerkiksi koodissa.
  * Turvallisuutta ei voi tehdä vain käyttöliittymän puolella, koska mm. jotkut ohjelmointikehykset sallivat HTTP pyyntöjen muutoksen, vaikka itse kieltäisit sen pyynnössä.

  Karvinen 2006: Raportin kirjoittaminen
  * Raportissa tulisi kertoa tarkasti, mitä teki ja mitä siitä tapahtui.
  * Raportin tulee olla: toistettava:
    * Tulos pitäisi olla sama, mukaanlukien harhapolut. Myös ympäristö tulisi raportoida, päivämäärästä laitteeseen ja versioihin
  * ... täsmällinen:
    * Mitä komentoja tai toimintoja käytti, milloin. Jos epäonnistuu, miten todentaa?
    * Kirjoita imperfektissä (mennyt muoto): "valitsin", "kaatui", "itkin"
  * ... helppolukuinen:
    * Väliotsikot, selvä ja huolellinen kieli.
  * Lähdeviittaukset:
    * Todistaa, että olet oikeasti perehtynyt asiaan
  * Kiellettyä:
    * Sepittäminen (valehtelu/keksiminen), plagiointi
   
a, b) 010-staff-only
Tekoympäristö:
* Debian 13 -virtuaalinen käyttöjärjestelmä VirtualBoxin avulla. Käyttöjärjestelmässä käytössä 7 GB RAM, 5 prosessoria.
* Firefox -selain
* Wi-Fi yhteydellä kotiverkko

Murtautuminen:
* Epäonnistuneita yrityksiä:
  * Kokeiltu ensin kaikkea yksinkertaista, kuten ', admin, --, -123. Nämä epäonnistuvat, koska jo ennen pyynnön lähetystä, sovellus tarkistaa onko syöte numero.
  * Kokeiltu seuraavaksi manuaalisesti lähetettyjä POST pyyntöjä, jossa pin oli edellä mainittuja arvoja. Pyyntöjä voi lähettää mm. selaimen kautta: CTRL + SHIFT + I, "Network", "+" -nappi (New Request). Osoite localhost:5000, POST pyyntö, headerit peritty oikeasta pyynnöstä, body: pin=admin <- jossa admin on kokeiltu arvo. Välilehdeltä "Response" saa näkymän näkyviin, joka kertoo tuloksen. Esimerkiksi request bodyllä " pin=' OR "1"="1" -- " tuloksena on "your password is foo" <img width="1916" height="398" alt="proof of failure" src="https://github.com/user-attachments/assets/43ae560d-f42a-48c6-81e1-354a1d21139f" />
* Onnistunut yritys: Piti noin tunnin hinkkaamisen jälkeen katsoa vinkkiä, ja oma suoritus olikin jo aika lähellä oikeaa. Piti vain lisätä oikeanlainen LIMIT -avainsana, jota en ollut aikaisemmin käyttänyt. Halutun salasanan saa pinkoodilla " pin=' OR 1=1 LIMIT 2,1; -- ". <img width="1480" height="642" alt="proof of success" src="https://github.com/user-attachments/assets/c6cffd63-0a17-4866-8baf-81485eebfcc2" />. Tulos siis: SUPERADMIN%%rootALL-FLAG{Tero-e45f8764675e4463db969473b6d0fcdd}. 

* Vian selitys: Vika tapahtuu, koska käyttäjän syöte "pin" vain lisätään suoraan SQL-käskyyn tekstinä, esim. **"SELECT password...' " + pin + " ';"**, joka ei ole turvallinen tapa. Lisäksi, koska syötteen validointi tapahtuu vain käyttöliittymän puolella, sen pystyy helposti kiertämään. Hyvä koodi tarkistaa syötteen myös järjestelmän puolella ennen käskyn lähettämistä tietokantaan, sekä käsittelee sen niin, että se ei voi sisältää haitallista koodia.

Korjaus:
* Koodissa vika löytyy funktion hello ensimmäiseltä muutamalta riviltä (18, 22). <img width="618" height="216" alt="image" src="https://github.com/user-attachments/assets/a6a0f37a-7879-48a5-ba73-226ff748b3b0" />
 
* Miten virhe on saattanut aiheutua:
* Miten korjaus toimii:
* Johtuuko korjauksesta muita ongelmia:

Reflektointi:
* Minkälaisissa kohteissa voisi olla sama haavoittuvuus:
* Onko yleinen ja realistinen:
* Miten välttää vastaavanlaista haavoittuvuutta:
* Muuta opittua?




## Lähteet
* Karvinen 2026: Kotitehtävän tehtävänanto. Luettavissa: https://terokarvinen.com/application-hacking/#homework
* OWASP: OWASP Top 10. Luettavissa: https://top10.owasp.org/2021/A01_2021-Broken_Access_Control/
* Karvinen 2023: Find Hidden Web Directories - Fuzz URLs with ffuf. Luettavissa: https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/
* Portswigger: Access control vulnerabilities and privilege escalation. Luettavissa: https://portswigger.net/web-security/access-control
* Karvinen 2006: Raportin kirjoittaminen. Luettavissa: https://terokarvinen.com/2006/raportin-kirjoittaminen-4/
