# h6 - Onkohan tämä turvallinen käyttää? 

### Tehtävänanto
* Tutki Tapo C200-kameran ohjelmiston turvallisuutta
* Kirjoita raportti siitä, miten ja mitä haavoittuvuuksia löysit. Voiko haavoittuvuuksia käyttää hyödyksi?

## Vastaus

### Vaiheet:

Lataukset:
* Tapo C200-laiteohjelmisto. Mennään tiedostonimellä "Tapo-C200v3_en_1.4.2.bin"
* Kameran dump-tiedosto. Mennään tiedostonimellä "dump-tapo-c200v3-1.4.2.bin"
* tp-link-decrypt -repositorion työkalu. Saatavilla: https://github.com/robbins/tp-link-decrypt.

Tehdään kansio "tapo" ja siirrytään sen sisään. 

    mkdir tapo
    cd tapo

Ladataan C200:n laiteohjelmisto:

    aws s3 cp s3://download.tplinkcloud.com/firmware/Tapo_C200v3_en_1.4.2_Build_250313_Rel.40499n_up_boot-signed_1747894968535.bin Tapo_C200v3_en_1.4.2.bin --no-sign-request

Huom. vaatii AWS CLI:n.
    
Siirretään C200:n kameradump kansion tapo sisään (löytyy moodlesta), ja kloonataan tp-link-decrypt kansioon, ja siirrytään sen sisään:

    git clone https://github.com/robbins/tp-link-decrypt
    cd tp-link-decrypt

Ajetaan "preinstall.sh".

    ./preinstall.sh

Jos virheitä esiintyy, voi joutua itse lataamaan esimerkiksi python-riippuvuudet "jefferson" ja "ubi_reader".

Kun preinstall on tehty, suoritetaan:

    ./extract_keys.sh

Kun ohjelma kysyy: "[INFO] Do you want to run binwalk in quiet mode? [yes/no]": kirjoita "no". Onnistumisen jälkeen suoritetaan:

    make

Jos tämä onnistui, nyt kansiossa bin tulisi olla "tp-link-decrypt" -tiedosto. Sillä voi nyt decryptata C200:n firmwaren:

    cd ..
    tp-link-decrypt/bin/tp-link-decrypt Tapo_C200v3_en_1.4.2.bin

Jos kaikki tapahtui onnistuneesti, sinulla pitäisi nyt olla tiedosto "Tapo_C200v3_en_1.4.2.bin.dec".

<img width="568" height="63" alt="Proof of success" src="https://github.com/user-attachments/assets/4240ad2a-57df-4caf-9189-3539c55aeb9f" />




## Lähteet
* Kurssin moodle sivu, "Sovellusten hakkerointi ja haavoittuvuudet - ICI012AS3A-3004 - 2026p1 - Tero ja Lari - to 14:00", välilehti "Hardware hacking". 
* https://github.com/robbins/tp-link-decrypt (valmis työkalu, jolla saa kaivettua avaimet purkamiseen)
* https://www.evilsocket.net/2025/12/18/TP-Link-Tapo-C200-Hardcoded-Keys-Buffer-Overflows-and-Privacy-in-the-Era-of-AI-Assisted-Reverse-Engineering (esimerkki siitä, kuinka ohjelmiston saa auki, ja mitä haavoittuvuuksia voi löytää)
* https://quentinkaiser.be/security/2025/07/25/rooting-tapo-c200/ (toinen esimerkki siitä, miten esimerkiksi root-salasanan voi saada selville)
