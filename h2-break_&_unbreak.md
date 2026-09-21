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
  * Ffuf on työkalu, jolla voi "fuzzata" tien sisään piilotettuihin kansioihin, headereihin, POST parametreihin jne.
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
* Onnistunut yritys: Piti noin tunnin hinkkaamisen jälkeen katsoa vinkkiä, ja oma suoritus olikin jo aika lähellä oikeaa. Piti vain lisätä oikeanlainen LIMIT -avainsana, jota en ollut aikaisemmin käyttänyt. Halutun salasanan saa pinkoodilla " pin=' OR 1=1 LIMIT 2,1; -- ". <img width="1480" height="642" alt="proof of success" src="https://github.com/user-attachments/assets/c6cffd63-0a17-4866-8baf-81485eebfcc2" />
Tulos siis: SUPERADMIN%%rootALL-FLAG{Tero-e45f8764675e4463db969473b6d0fcdd}. 

* Vian selitys: Vika tapahtuu, koska käyttäjän syöte "pin" vain lisätään suoraan SQL-käskyyn tekstinä, esim. **"SELECT password...' " + pin + " ';"**, joka ei ole turvallinen tapa. Lisäksi, koska syötteen validointi tapahtuu vain käyttöliittymän puolella, sen pystyy helposti kiertämään. Hyvä koodi tarkistaa syötteen myös järjestelmän puolella ennen käskyn lähettämistä tietokantaan, sekä käsittelee sen niin, että se ei voi sisältää haitallista koodia.

Korjaus:
* Koodissa isoimmat viat löytyy riveiltä 18 ja 22.

  <img width="618" height="216" alt="image" src="https://github.com/user-attachments/assets/a6a0f37a-7879-48a5-ba73-226ff748b3b0" />

* Rivi 18 asettaa muuttujaan "pin" käyttäjän syötteen string-muodossa. Tästä puuttuu kokonaan validointi, onko syöte edes numero. Tarkistaessa voisi esimerkiksi kokeilla muuttaa syötteen ensin numeroksi. Muutoksesta aiheutuu ohjelman kaatuminen, jos muutos ei ole mahdollinen, jonka taas voi estää "except" lohkolla. Tässä miten itse muuttaisin koodin:

  <img width="438" height="117" alt="Added type check of pin" src="https://github.com/user-attachments/assets/1385a9e9-fa7d-4164-a761-b720447d5c00" />

  Testattaessa nyt kaikki muut syötteet paitsi numerot johtavat siihen, että pin on 0. Mukaanlukien injektio, jolla admin salasana saatiin. 

* Rivi 22 asettaa muuttujaan "sql" tietokantaan lähtevän käskyn, joka vain lisää tekstinä "pin" muuttujan arvon. Oikeaoppisessa järjestelmässä syötettä ei lisätä suoraan käskyyn, sille vain varataan paikka parametrina. Tähän löytyy verkosta tietoa, valinta riippuu tietokannan perusteella: MySQL käyttää "?", SQL Server käyttää "@", PostgreSQL käyttää "$", meidän SQLAlchemy käyttää ":". (https://www.w3schools.com/sql/sql_parameterized_queries.asp). Tehdään käsky siis parametrisoituna, käyttäen ":". Tätä kohtaa tehdessä peruutan myös aikaisemman korjauksen muutokset, sillä ne jo "korjaavat ohjelman". Muutettu koodi: <img width="584" height="119" alt="Parametrized query" src="https://github.com/user-attachments/assets/816b6a6f-49be-4e99-aab9-7399d0cec688" />

  Tällä estetään SQL-injektiot. Testattaessa kaikenlaista, en löytänyt tapaa jolla oltaisiin saatu tulos, joka on väärin. Ratkaisu siis toimii.

* Miten virhe on saattanut aiheutua: Tämänlaiset virheet tulevat lähes varmasti kokemattomalta kehittäjältä. Ainakin luulen ja toivon, että kokenut kehittäjä tietää, että syötettä ei voi suoraan lisätä SQL-käskyyn. Tyyppitarkistuksen puuttuminen voisi olla vielä yksinkertaista laiskuutta tai huolettomuutta.
* Miten korjaus toimii:
  * Korjaus 1 (rivi 18) yrittää muuttaa tuloksen numeroksi. Jos numeron muuttaa numeroksi, se onnistuu hyvin. Ei kuitenkaan ole mahdollista muuttaa jotakin muuta numeroksi, jos se ei ole numero. Jos tulos ei ole numero, se aiheuttaisi ohjelman kaatumisen, jonka "except"-lohko pelastaa, ja sen sijaan asettaa pin-koodiksi 0. Tällöin esimerkiksi injektio (joka ei ole pelkästään numeroita) muuttuu numeroksi 0.
  * Korjaus 2 (rivi 22) korjaa suoraan SQL-injektion. Parametrisointi syöttää käyttäjän syötteen palasissa, jolloin ne käsitellään erikseen. Tällöin esimerkiksi SQL-injektio ei pysty aiheuttamaan haittaa.
  
* Korjauksista johtuvat ongelmat:
  * Korjaustapa 1 toimii nyt tässä tilanteessa vain numeroille. Jos pin-koodissa voisikin jostain syystä olla vaikka kirjain, ei koodi toimisi.
  * Korjaustapa 2 muuttaa samalla sivun alareunassa olevan syötteen "SELECT password FROM pins WHERE pin= käyttäjänsyöte" -> "SELECT password FROM pins WHERE pin= :pin" riippumatta siitä, syöttääkö käyttäjä hyväksytyn vai hylätyn syötteen. Tämän voisi korjata (eikä sitä varmaan alkuunkaan olisi, jolloin sillä ei ole väliä) ottamalla syötetty pin-koodi, riippumatta tuloksesta.

Reflektointi:
* Minkälaisissa kohteissa voisi olla sama haavoittuvuus: Etenkin aloittavien kehittäjien sovelluksissa ja jossain massatuotannon sovelluksissa, jos se on huomiovirhe.
* Onko yleinen ja realistinen: Todellakin ainakin ollut yleinen ja realistinen. Toivon, että nykyään harvempi, koska se on myös aika helppo ja nopea korjata, ja koskee yhä tärkeämmäksi nousevaa tietoturvaa. 
* Miten välttää vastaavanlaista haavoittuvuutta: Huolellisuus ja varovaisuus, varsinkin kun käsitellään tietokantaa ja sinne kohdistuvia käskyjä. 
* Muuta opittua: Python on syvältä. TabError: <img width="726" height="196" alt="image of taberror" src="https://github.com/user-attachments/assets/14cc2f0b-c0bf-4340-ba53-12b5b658f016" />


c) dirfuzt-1
Tarvittavat ladattavat asiat: 
* dirfuzt-1. Ohjeet lataamiseen ja asennukseen löytyy lataussivulta. Ladattavissa: https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/
* ffuf. Ladattavissa: `sudo apt-get install ffuf`
* SecLists, common.txt. Ladattavissa: `wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/Web-Content/common.txt`

Koska palvelut (dirfuzt-1 ja ffuf) toimivat täysin lokaalisti ilman internetiä, irrotan tässä välissä yhteyden internetistä turvallisuussyistä. 

Kokeillaan nyt kaikkia common.txt:n rivejä URL-osoitteen päätteenä, käyttäen ffuf:ia. Eli http://127.0.0.2:8000/FUZZ (jossa FUZZ on jokainen yli 4700 avainsanaa common.txt tiedostosta yksitellen). 

	ffuf -w common.txt -u http://127.0.0.2:8000/FUZZ

Tällä saadaan tuhansia rivejä, esim:
<img width="1191" height="521" alt="Example of fuff output" src="https://github.com/user-attachments/assets/1b495e9a-f02f-4212-af86-9c7b2bc6b2e4" />

Tämä ei kuitenkaan vielä auta meitä, koska nyt kaikista tuloksista, oli tulos haluttu tai ei, tulee uusi rivi. Meidän pitää siis suodattaa sisältöä jotenkin niin, että virheellisiä tuloksia ei tulosteta. Katsotaan siis tuloksista, mikä toistuu eniten. Toistuvia kohteita on eniten:
* Status: 200
* Size: 154
* Words: 9
* Lines: 10
* Duration 0ms/1ms

Voidaan suoraan karsia vaihtoehtoja: haluttu tulos pitäisi olla myös toimiva verkkosivu, jolloin sen tulisi sisältää myös "Status: 200". Emme siis voi suodattaa tämän perusteella. Duration kertoo kauanko kokeilu kesti, joka ei kerro välttämättä mitään sivun sisällöstä. Koska esimerkissäkin oli jo vaihtelua ilman, että haluttu tulos löytyi, ei se ole hyvä suodatuskohde.

Valinnoiksi jää siis koko, sanamäärä ja rivimäärä. Näistä voisi ainakin teoriassa valita minkä vain, mutta size on turvallisin, koska esim. riveillä ei välttämättä ole sama tieto, vaikka rivimäärä olisi yhtä iso. Suodatetaan siis tuloksia koon mukaan niin, että kokomerkinnällä "154" ei tulosteta. Ffuf:ssa tämä tehdään parametrilla "-fs". Uusi komento siis:

	ffuf -w common.txt -u http://127.0.0.2:8000/FUZZ -fs 154

<img width="893" height="613" alt="Proof of edited ffuf command working" src="https://github.com/user-attachments/assets/b9d86ba4-b4c8-43f8-b63e-2ecf7480fe96" />

Nyt tuloksia on vain seitsemän. Ohjeissa luki, että etsimme versionhallintaan liittyvää sivua, ja admin-sivua. Tuloksista voisi päätellä, että versionhallinnan sivu on varmaankin `http://127.0.0.2:8000/.git` ja admin-sivu on `http://127.0.0.2:8000/wp-admin`. Käydään tarkistamassa nämä manuaalisesti:

<img width="482" height="171" alt="Proof that .git exists with a flag" src="https://github.com/user-attachments/assets/c8e9191f-31c2-4d69-adf5-330bd435d53e" />
<img width="523" height="176" alt="Proof that wp-admin is an admin page with a flag" src="https://github.com/user-attachments/assets/9e8f01d6-ceb1-432c-9738-a4ede29694a8" />

Versionhallinnan lippu: FLAG{tero-git-3cc87212bcd411686a3b9e547d47fc51}

Admin-sivun lippu: 		FLAG{tero-wpadmin-3364c855a2ac87341fc7bcbda955b580}


d,e)

Ympäristö: Windows -> VirtualBox Debian 13. 7 GB RAM, 5 core. Firefox, Lenovo-kannettava tietokone, kotiverkko

Varovaisuus: Ffufia käyttäessä verkko sammutettuna.

Sovelluksen käynnistys:

Tehdään seuraavat komennot rivi kerrallaan:

	cd challenges/020-your-eyes-only/
	sudo apt-get -y install virtualenv
	virtualenv virtualenv/ -p python3 --system-site-packages
	source virtualenv/bin/activate
	pip install -r requirements.txt
	cd logtin/
	./manage.py makemigrations; ./manage.py migrate
	./manage.py runserver

Nyt pitäisi olla käynnissä server, jossa applikaatio pyörii. Navigoidaan verkossa: `http://127.0.0.1:8000`:

<img width="627" height="427" alt="Assigment d webpage" src="https://github.com/user-attachments/assets/13cf2479-ffdf-4615-8973-cde5af26df19" />

Murtautuminen:
* Mitkä tavat epäonnistuivat:
  * Käytin ensimmäisenä ffufia etsimään mahdollisia sijainteja, jos vaikka löytyisi suojaamaton tapa päästä admin-konsoliin ilman kirjautumista. Etsitään ensin normaalit tulokset: `ffuf -w common.txt -u http://127.0.0.1:8000/FUZZ`. Tuloksia vain yksi, "admin-console". Teoriana se, että common.txt ei sisällä esim. "login", "register", ja "admin-dashboard" osoitteita, koska login ja register ovat tyypillisiä löydettäviä osoitteita, ja admin-dashboard saattaa olla taas vähemmän tyypillinen. Muita osoitteita ei löydy listauksesta ollenkaan, koska ne johtaa "Page Not Found" eli error 404 -sivuun. Kokeilin laittaa manuaalisesti URL-osoitteen perään "admin-console", mutta se vaatii silti kirjautumista. Meidän pitää siis löytää tapa kiertää login/register pyyntö, mahdollisesti rekisteröitymällä vain käyttäjäksi ja katsoa, tarkastaako koodi onko käyttäjä admin vai vain kirjautunut. 
* Mikä tapa onnistui:
  * Rekisteröidyin käyttäjäksi (user, ei admin), ja yritin navigoida UI-elementtien kautta admin-sivulle. Tämä johtaa osoitteeseen `http://127.0.0.1:8000/admin-dashboard`, josta sovellus ilmoitti "403 Forbidden", eli ei oikeuksia. Tämä sivu on siis suojattu oikein niin, että käyttäjä ei pääse sisään. Mitä kuitenkin tapahtuu, jos kokeilen mennä ffuf:lla löydettyyn "admin-consoleen"?
    <img width="735" height="375" alt="Admin console does not check permission" src="https://github.com/user-attachments/assets/9873cc1e-956a-42db-8ea5-07580f6e6662" />

	Admin-console ei nähtävästi tarkista käyttäjän oikeuksia, kuten dashboard. 

* Mikä haavoittuvuus:
  * Käyttäjän oikeustasoa ei käsitellä kaikissa admin-liittyvissä sivuissa. Dashboard tarkistaa, console ei. Millä tahansa kirjautuneella käyttäjällä on siis pääsy "admin-console"-sivulle, on admin tai ei. 
* Miten haavoittuvuutta voi käyttää hyväksi:
  * Riippuen admin consolen toiminnoista, kuka tahansa käyttäjä voisi monitoroida tai muokata käyttäjien dataa, oikeuksia jne. 

Korjaaminen:
* Mikä osio koodista: Kansion logtin/hats tiedostot:
  * `logtin/hats/urls.py` (määrittää näkymät ja niiden osoitteet)
  * `logtin/hats/views.py` (määrittää näkymien luokat ja sitä kautta oikeudet). 
* Miksi tämä on virheellinen: views.py palauttaa boolean arvon dashboardista seuraavalla tavalla `return self.request.user.is_authenticated and self.request.user.is_staff`, mutta admin consolesta vain `return self.request.user.is_authenticated`. Console ei siis tosiaan tarkista ollenkaan, onko käyttäjä "staff", eli admin.
* Miten vika olisi voinut tapahtua: Huomiovirhe? Toisaalta koko rakennekkaan ei tee hirveästi omaan silmään järkeä. Miksi dashboard ja console eivät käytä suoraan samoja oikeusehtoja? Tämä tuntuu selvästi enemmän tahalliselta, kuin vahingolta. 
* Miten korjata:
  * Views.py: Lisätään luokkaan "AdminShowAllView" myös ehto, että käyttäjän pitää olla staff.
  * Toinen vaihtoehto, Urls.py: Käyttää admin consolessa samaa luokkaa "AdminDashboardView", jota dashboard käyttää.
  
  Tässä lisätty "AdminShowAllView"iin ehto, että käyttäjän tulee olla staff. Kuvassa ylhäällä muuttamaton sisältö, alhaalla muutettu:
  
  <img width="897" height="822" alt="Pre and post change code" src="https://github.com/user-attachments/assets/359efb6a-b5b6-45e0-8f8d-81171a429159" />

  Ja todiste siitä, että nyt Admin Console antaa virhekoodin "403 Forbidden", eikä oikeaa sisältöä:

  <img width="564" height="137" alt="proof of fix" src="https://github.com/user-attachments/assets/844c07f3-c0a4-4df1-ae6a-de8fad6d92f2" />

* Aiheutuuko korjauksesta haittavaikutuksia: Ei.

Reflektio:
* Minkälaisissa kohteissa voisi olla sama haavoittuvuus: Paha sanoa. Aloittelijan sovellukset? Jos puhutaan yleisesti tarkistamisen unohtamisesta, voi tapahtua kenelle vain, jos ei pidä tarpeeksi tarkkaa huomiota.
* Onko yleinen ja realistinen: Jos tällä tavalla tekisi, että jokainen admin tiedosto sisältää omat oikeudet (jos niillä saattaisi syystä tai toisesta olla muuttuvaa sisältöä), voisi olla realistinen. En kuitenkaan usko, että kyseinen henkilö pysyisi työpaikan palkkalistalla. En sanoisi yleiseksi, ainakaan jos kyse on siitä, että osat admin-suojauksesta on unohtunut, osa ei. Jos puhutaan yleisesti oikeiden tarkistamisesta, voi se olla jonkin verran yleisempi.  
* Miten välttää haavoittuvuutta: Välttää oikeuksien laittamista useaan eri luokkaan/funktioon yms, mieluummin yksi kunnolla tehty paikka. Jos paikkoja on useampi, sisältö pitäisi kopioida suoraan, jotta tälläisiä virheitä ei tapahtuisi. 
* Muita opetuksia: Ei.

## Lähteet
* Karvinen 2026: Kotitehtävän tehtävänanto. Luettavissa: https://terokarvinen.com/application-hacking/#homework
* OWASP: OWASP Top 10. Luettavissa: https://top10.owasp.org/2021/A01_2021-Broken_Access_Control/
* Karvinen 2023: Find Hidden Web Directories - Fuzz URLs with ffuf. Luettavissa: https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/
* Portswigger: Access control vulnerabilities and privilege escalation. Luettavissa: https://portswigger.net/web-security/access-control
* Karvinen 2006: Raportin kirjoittaminen. Luettavissa: https://terokarvinen.com/2006/raportin-kirjoittaminen-4/
* Karvinen 2024: Hack n Fix. https://terokarvinen.com/hack-n-fix/ (tehtävät a, b, d, e) 
* W3Schools: SQL Parameters. Luettavissa: https://www.w3schools.com/sql/sql_parameterized_queries.asp
* OpenAI:n ilmainen ChatGPT -laaja kielimalli. 20.9.2026. Käytetty SQL-injektion korjaamisessa koodista.
