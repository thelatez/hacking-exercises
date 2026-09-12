# h6 - Onkohan tämä turvallinen käyttää? 

### Tehtävänanto
* Tutki Tapo C200-kameran ohjelmiston turvallisuutta
* Kirjoita raportti siitä, miten ja mitä haavoittuvuuksia löysit. Voiko haavoittuvuuksia käyttää hyödyksi?

## Vastaus

Tehtävän aloittamista varten pitää ladata Tapo C200:n laiteohjelmisto. Sen saa ladattua vapaasti myös internetistä, mutta tässä tapauksessa käytän opettajan jakamaa laiteohjelmistoa. Kyseessä on "Tapo C200" kameran v3-laiteohjelmisto. Huomiona se, että uudempia versioita löytyy, mm. v5, mutta tässä tehtävässä ei ole tarkoitus aiheuttaa oikeita ongelmia, jolloin tutkimme tiedetysti viallista versiota v3. Lisäksi pitää ladata myös "tp-link-decrypt" github-repositorio, jossa on työkalu jolla kaivaa laitteesta avaimet. Suorittaminen kannattaa tehdä linux-ympäristössä, mieluusti "Debian", "Ubuntu" tai "Kali" distribuutiolla. 

Lataukset selkeämmin:
* Tapo C200-laiteohjelmisto. Mennään tiedostonimellä "dump-tapo-c200v3-1.4.2.bin"
* tp-link-decrypt -repositorion työkalu. Saatavilla: https://github.com/robbins/tp-link-decrypt.

### Vaiheet:

Tehdään kansio "tapo" ja siirrytään sen sisään. Siirretään C200:n laiteohjelmisto kansion sisään.
Kloonataan tp-link-decrypt kansioon, ja siirrytään sen sisään:

    git clone https://github.com/robbins/tp-link-decrypt
    cd tp-link-decrypt

Ajetaan "preinstall.sh".

    ./preinstall.sh

Jos virheitä esiintyy, voi joutua itse lataamaan esimerkiksi python-riippuvuudet "jefferson" ja "ubi_reader".

Kun preinstall on tehty, suoritetaan:

    ./extract_keys.sh

Kun ohjelma kysyy: "[INFO] Do you want to run binwalk in quiet mode? [yes/no]": kirjoita "no". Onnistumisen jälkeen suoritetaan

    make

Ja sen jälkeen decryptauksen voi viimeistellä:

    cd ..
    tp-link-decrypt/bin/tp-link-decrypt dump-tapo-c200v3-1.4.2.bin

Jos kaikki tapahtui onnistuneesti, sinulla pitäisi nyt olla tiedosto "dump-tapo-c200v3-1.4.2.bin.dec".


## Lähteet
* Kurssin moodle sivu, "Sovellusten hakkerointi ja haavoittuvuudet - ICI012AS3A-3004 - 2026p1 - Tero ja Lari - to 14:00", välilehti "Hardware hacking". 
* https://github.com/robbins/tp-link-decrypt (valmis työkalu, jolla saa kaivettua avaimet purkamiseen)
* https://www.evilsocket.net/2025/12/18/TP-Link-Tapo-C200-Hardcoded-Keys-Buffer-Overflows-and-Privacy-in-the-Era-of-AI-Assisted-Reverse-Engineering (esimerkki siitä, kuinka ohjelmiston saa auki, ja mitä haavoittuvuuksia voi löytää)
* https://quentinkaiser.be/security/2025/07/25/rooting-tapo-c200/ (toinen esimerkki siitä, miten esimerkiksi root-salasanan voi saada selville)
