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
   
a)




## Lähteet
* OWASP: OWASP Top 10. Luettavissa: https://top10.owasp.org/2021/A01_2021-Broken_Access_Control/
* Karvinen 2023: Find Hidden Web Directories - Fuzz URLs with ffuf. Luettavissa: https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/
* Portswigger: Access control vulnerabilities and privilege escalation. Luettavissa: https://portswigger.net/web-security/access-control
* Karvinen 2006: Raportin kirjoittaminen. Luettavissa: https://terokarvinen.com/2006/raportin-kirjoittaminen-4/
