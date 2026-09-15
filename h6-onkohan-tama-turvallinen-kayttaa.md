# h6 - Onkohan tämä turvallinen käyttää? 

### Tehtävänanto
* Tutki Tapo C200-kameran ohjelmiston turvallisuutta
* Kirjoita raportti siitä, miten ja mitä haavoittuvuuksia löysit. Voiko haavoittuvuuksia käyttää hyödyksi?

## Vastaus

### Lataukset/valmistelu:
* Tapo C200-laiteohjelmisto. Mennään tiedostonimellä "Tapo-C200v3_en_1.4.2.bin"
* Kameran dump-tiedosto. Mennään tiedostonimellä "dump-tapo-c200v3-1.4.2.bin". <sub>(En tarjoa tälle mitään julkista latausmetodia, koska en tiedä onko se sallittua).</sub>
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

Jos tämä onnistui, nyt kansiossa bin tulisi olla "tp-link-decrypt" -tiedosto. Sillä voi nyt decryptata C200:n firmwaren.

### 1. Decrypt firmware image

    cd ..
    tp-link-decrypt/bin/tp-link-decrypt Tapo_C200v3_en_1.4.2.bin

Jos decryptaus tapahtui onnistuneesti, sinulla pitäisi nyt olla tiedosto "Tapo_C200v3_en_1.4.2.bin.dec".

<img width="568" height="63" alt="Proof of success" src="https://github.com/user-attachments/assets/4240ad2a-57df-4caf-9189-3539c55aeb9f" />

### 2. Analyse the image file

Tehdään analysointi binwalkilla.

    binwalk Tapo_C200v3_en_1.4.2.bin.dec

<img width="1084" height="402" alt="First part of binwalk command result" src="https://github.com/user-attachments/assets/de3a5d28-9e17-4977-9ebc-3d3aeb6d2a1b" />

<img width="1083" height="174" alt="Ending of binwalk command result" src="https://github.com/user-attachments/assets/6358b2f9-08fe-467e-9c4f-b9192dc0f3fc" />
<sub>Huom. välistä jätetty rivejä kompressoitua xz-dataa.</sub> 

    
Käyn läpi vain tärkeimmät huomiot: 
* "Android bootimg" sisältää 0 tavua kokoisen kernelin, ja massiivisen ramdisk-koon. Tämä on mitä luultavammin false positive.
* "uImage header" on tunnetusti yhdistetty U-Boot:iin, joka joka on yleinen sulautettujen järjestelmien bootloader. Erityisesti kertoo:
  * CPU:n arkkitehtuuri on MIPS vs. esim. ARM, yleinen nykypäivän puhelinten ja sulautettujen järjestelmien arkkitehtuuri
  * Image type kertoo tämän olevan kernel image.
  * Image name kertoo, että valmistaja on "Ingenic", linux kernel versiolla 3.10.14.
  * Image size kertoo kernelin olevan kompressoituna noin 1.3 MB.
* "JBOOT STAG header" väittää kokonsa olevan noin 1.8 GB. Tämä ei ole mahdollista, joten luultavasti toinen false positive.
* "Squashfs filesystem" on kompressoitu, erittäin tunnettu read-only käyttöjärjestelmä Linux-pohjaisissa sulautetuissa järjestelmissä. Järjestelmä käyttää xz-kompressiota. Kokona noin 3 MB kompressoituna.

Lähes varmaa on siis se, että etsimämme asiat (esim. salasanat) löytyvät, jos Squashfs:n extractaa. 

### 3. Extract rootfs from the dump file

Nyt kyse on dump-tiedostosta, eli opettajan antamasta kameran dumpista (dump-tapo-c200v3-1.4.2.bin). Tämä järjestelmä toimii myös squashfs:llä, jonka voi myös tarkistaa suorittaen "binwalk <tiedostonnimi>". Helpoin tapa extractaa olisi tehdä "binwalk -e dump-tapo-c200v3-1.4.2.bin", mutta tällä ei valitettavasti tule haluttua tulosta. Koska binwalk ei suoraan löydä oikeaa muistialuetta kokonaan, pitää se löytää manuaalisesti:

    

Tämä luo kansion "_dump-tapo-c200v3-1.4.2.bin.extracted".

### 4. Extract rootfs from the image file

Tämän voi extractata "helpommalla tavalla", koska meitä ei kiinnosta niin paljoa rakenne ja sen sisältö, vaan avain, joka sijaitsee kernelissä. Muistellaan ensin kernelin sijainti aikaisemmin tehdystä "binwalk Tapo_C200v3_en_1.4.2.bin.dec"-komennosta. Alla sama kuva uusiksi:

<img width="1084" height="402" alt="First part of binwalk command result" src="https://github.com/user-attachments/assets/de3a5d28-9e17-4977-9ebc-3d3aeb6d2a1b" />

Kuvasta näkee, että "uImage header" (kernel) on 64 tavua, ja se on kompressoitu "lzma":lla. UImagen jälkeen tulee lzma kompressoitu alue, joka on luultavimmin se, missä kernelin data sijaitsee. 

Extractataan seuraavaksi decryptattu firmware:

    binwalk -e Tapo_C200v3_en_1.4.2.bin.dec 
    cd _Tapo_C200v3_en_1.4.2.bin.dec.extracted

Seuraavaksi yritämme purkaa kernelin ulos:

    dd if=../Tapo_C200v3_en_1.4.2.bin.dec of=kernel.lzma bs=1 skip=$((0x20440)) count=4200000
    python3 -c "
    import lzma
    data = open('kernel.lzma','rb').read()
    dec = lzma.LZMADecompressor(format=lzma.FORMAT_ALONE)
    open('kernel.bin','wb').write(dec.decompress(data))
    "
    file kernel.bin

Nyt kansiossa pitäisi myös olla tiedosto "kernel.bin". Yritämme etsiä kernelistä avaimen, ja toisten ihmisten aikaisempien tutkimusten perusteella avain alkaa yleensä "TP-LINK", jolloin etsitään alkuun vastaavanlaista sisältöä:

    strings -n 8 kernel.bin | grep "^TP_LINK"

Tämä palauttaa minulle avaimen plaintekstinä: "TP_LINK88i667gnt". Jotta avain saadaan käytettävään muotoon, tehdään komento:

    echo -n "TP_LINK88i667gnt" | xxd -p | tr -d '\n'; echo

Tästä palautuu "54505f4c494e4b383869363637676e74". Avaimen tulee olla 32-merkkiä pitkä, ja tämä on. Mennään sillä oletuksella, että saatu avain on toimiva. Avaimen kanssa tarvitaan myös "IV". IV on kuitenkin kaikissa TP-Link laitteissa sama, laitteesta ja versiosta lukuunottamatta. IV:n löysi netistä. Key ja IV siis:
* Key == 54505f4c494e4b383869363637676e74
* IV  == 55aadeadc0de4c494e5558457854aa55




### 5. Search available applications

Aloitetaan firmwaren versiosta, kohdasta 4. 

### 6. Finding the root password

Aloitetaan tiedostolla "Tapo_C200v3_en_1.4.2.bin.dec". Tämä on siis se decryptattu firmware-tiedosto, ei kohdassa 4 extractattu.

## Lähteet
* Kurssin moodle sivu, "Sovellusten hakkerointi ja haavoittuvuudet - ICI012AS3A-3004 - 2026p1 - Tero ja Lari - to 14:00", välilehti "Hardware hacking". 
* https://github.com/robbins/tp-link-decrypt (valmis työkalu, jolla saa kaivettua avaimet purkamiseen)
* https://www.evilsocket.net/2025/12/18/TP-Link-Tapo-C200-Hardcoded-Keys-Buffer-Overflows-and-Privacy-in-the-Era-of-AI-Assisted-Reverse-Engineering (esimerkki siitä, kuinka ohjelmiston saa auki, ja mitä haavoittuvuuksia voi löytää)
* https://quentinkaiser.be/security/2025/07/25/rooting-tapo-c200/ (toinen esimerkki siitä, miten esimerkiksi root-salasanan voi saada selville)
* ilmainen OpenAI:n ChatGPT -laaja kielimalli. Käytetty 12.-14.9.2026. Saatavilla: chatgpt.com
* ilmainen Anthropic:n Claude Sonnet 5 -laaja kielimalli, effort tasoilla Medium ja High. Käytetty 14.-15.9.2026. Saatavilla: https://claude.ai/new
