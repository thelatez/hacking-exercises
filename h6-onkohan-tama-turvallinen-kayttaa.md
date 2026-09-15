# h6 - Onkohan tämä turvallinen käyttää? 

### Tehtävänanto

Tehtävänanto tunnilta:
1. Decrypt the firmware image            // Decryptaa firmware (laiteohjelmisto)
2. Analyse the image file                // Analysoi laiteohjelmistoa
3. extract rootfs from the dump file     // Extractaa rootfs kameran dump-tiedostosta
4. extract rootfs from the image file    // Extractaa rootfs laiteohjelmistosta
5. search available applications         // Etsi saatavilla olevia sovelluksia
6. analyse and try to open root password // Analysoi ja yritä avata rootin salasana

Lisäksi:
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

Nyt kyse on dump-tiedostosta, eli opettajan antamasta kameran dumpista (dump-tapo-c200v3-1.4.2.bin). Tämä järjestelmä toimii myös squashfs:llä, jonka voi myös tarkistaa suorittaen "binwalk (tiedostonnimi)". Helpoin tapa extractaa olisi tehdä "binwalk -e dump-tapo-c200v3-1.4.2.bin", mutta tällä ei valitettavasti tule haluttua tulosta. Koska binwalk ei suoraan löydä oikeaa muistialuetta kokonaan, pitää se löytää manuaalisesti. Myös key pitää löytää itse. Key voidaan saada selville esim. firmwaresta. Aloitetaan keyn löytämisellä.

Muistellaan ensin kernelin sijainti aikaisemmin tehdystä "binwalk Tapo_C200v3_en_1.4.2.bin.dec"-komennosta. Alla sama kuva uusiksi:

<img width="1084" height="402" alt="First part of binwalk command result" src="https://github.com/user-attachments/assets/de3a5d28-9e17-4977-9ebc-3d3aeb6d2a1b" />

Kuvasta näkee, että "uImage header" (kernel) on 64 tavua, ja se on kompressoitu "lzma":lla. UImagen jälkeen tulee lzma kompressoitu alue, joka on oletettavasti se alue, missä kernelin data sijaitsee. 

Extractataan decryptattu firmware (tämä on myös tehtävä 4):

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

Nyt kansiossa pitäisi myös tiedosto "kernel.bin". Yritämme etsiä kernelistä avaimen, ja toisten ihmisten aikaisempien tutkimusten perusteella avaimen voi löytää etsimällä "TP-LINK" kernelistä. Etsitään kernelistä "^TP_LINK":

    strings -n 8 kernel.bin | grep "^TP_LINK"

Tämä palauttaa minulle avaimen pelkkänä tekstinä: "TP_LINK88i667gnt". Jotta avain saadaan käytettävään muotoon, tehdään komento:

    echo -n "TP_LINK88i667gnt" | xxd -p | tr -d '\n'; echo

Tästä palautuu "54505f4c494e4b383869363637676e74". Avaimen tulee olla 32-merkkiä pitkä, ja tämä on. Mennään sillä oletuksella, että saatu avain on toimiva. Avaimen kanssa tarvitaan myös "IV". IV on kuitenkin kaikissa TP-Link laitteissa sama, laitteesta ja versiosta lukuunottamatta. IV:n löysi netistä. Key ja IV siis:
* Key == 54505f4c494e4b383869363637676e74
* IV  == 55aadeadc0de4c494e5558457854aa55

Etsitään nyt oikea squashfs:

    cat > scan.sh << 'EOF'
    #!/bin/bash
    FILE="dump-tapo-c200v3-1.4.2.bin"
    KEY="54505f4c494e4b383869363637676e74"
    IV="55aadeadc0de4c494e5558457854aa55"
    STEP=256
    START=$((0x1B0000))
    END=$((0x200000))
    
    for ((offset=START; offset<END; offset+=STEP)); do
      magic=$(dd if="$FILE" bs=1 skip=$offset count=512 status=none \
        | openssl enc -aes-128-cfb1 -d -nosalt -nopad -K "$KEY" -iv "$IV" 2>/dev/null \
        | head -c4 | xxd -p)
      if [ "$magic" = "68737173" ]; then
        printf "*** FOUND at offset 0x%X ***\n" "$offset"
      fi
    done
    echo "scan done"
    EOF
    chmod +x scan.sh
    ./scan.sh        

Tästä pitäisi tulostua: "*** FOUND at offset 0x1C0000 ***". Nyt siis tiedämme että oikea squashfs -tiedostojärjestelmä alkaa osoitteesta 0x1C0000. Emme vielä tiedä sen loppua, joten teemme kokeiluksi ensin riittävän laajan alueen:

    FILE="dump-tapo-c200v3-1.4.2.bin"
    KEY="54505f4c494e4b383869363637676e74"
    IV="55aadeadc0de4c494e5558457854aa55"
    START=$((0x1C0000))
    
    dd if="$FILE" of=real_rootfs.bin bs=1 skip=$START count=3000000 status=progress
    
    dd if=real_rootfs.bin bs=512 count=1 | openssl enc -aes-128-cfb1 -d -nosalt -nopad \
      -K "$KEY" -iv "$IV" | dd of=real_rootfs.bin bs=512 count=1 conv=notrunc
    
    file real_rootfs.bin

Tästä pitäisi tulostua kutakuinkin: "real_rootfs.bin: Squashfs filesystem, little endian, version 4.0, xz compressed, 2132764 bytes, 457 inodes, blocksize: 65536 bytes, created: Thu Mar 13 03:15:03 2025". Jos tulos ei ole "Squashfs filesystem", jotain on väärin.

Etsitään seuraavaksi oikea koko:

    python3 -c "
    import struct
    with open('real_rootfs.bin','rb') as f:
        data = f.read(48)
    bytes_used = struct.unpack('<Q', data[40:48])[0]
    print('exact filesystem size:', bytes_used, hex(bytes_used))
    "    

Tästä pitäisi tulostua kutakuinkin "exact filesystem size: 2132764 0x208b1c". Nyt siis tiedämme, että oikea koko on 2132764 bittiä, ja päätösosoite on 0x208b1c. Extractataan nyt oikea tiedostojärjestelmä:

    truncate -s 2132764 real_rootfs.bin
    rm -rf real_squashfs-root
    unsquashfs -d real_squashfs-root real_rootfs.bin

Nyt sinulla pitäisi olla oikea squashfs -tiedostojärjestelmä, "real_squashfs-root", josta löytyy esimerkiksi passwd, eli rootin salasana.

<img width="493" height="64" alt="Real_squashfs-root folder contents" src="https://github.com/user-attachments/assets/caa3be10-ac09-4ecc-a23d-6a51af6e66a8" />


### 4. Extract rootfs from the image file

Tämän voi extractata "helpommalla tavalla", koska meillä on jo "kunnolla tehty" dump-tiedoston extractaus. Tätä voidaan myös käyttää vertailukohtana.

HUOM! Tässä on ohje jos vaihetta 3 ei tehnyt. Siinä extractattiin jo firmware. Edellytyksenä kuitenkin vaihe 1.

    binwalk -e Tapo_C200v3_en_1.4.2.bin.dec

Jos osion 3 oli jo tehnyt tai edellisen ohjeen, seuraavaksi:

    cd _Tapo_C200v3_en_1.4.2.bin.dec.extracted/squashfs-root/

Nyt käsissä on myös yksinkertainen, ei täydellinen squashfs käyttöjärjestelmä. Täältä puuttuu paljon olennaisia osia, kuten root salasana ja paljon oikeita toimintoja. Tämäkin kansio kuitenkin sisältää tietoa.

### 5. Search available applications

| Tiedosto | Lähde | Vaaditut oikeudet |
| --- | --- | --- |
| main | squashfs_root (overlay) | root (muita oikeustasoja ei käytetä) |
| hostapd | squashfs_root (overlay) | root |
| init -skriptit | real_squashfs-root (OS) | root |

### 6. Finding the root password

Tiedämme jo, että dumpista kaivettu Squashfs sisältää nyt kaiken. Siirrytään siis tapo-kansion juuresta yleiseen salasanojen tallennuspaikkaan, "etc".

    cd real_squashfs-root/etc/ && ls

Näemme, että kansio sisältää tunnettuja sijanteja, "shadow" ja "passwd". Katsotaan niiden sisälle:

    more passwd && more shadow

Poikkeuksellisesti, rootin salasana löytyy "passwd" tiedostosta, joka on "vanha" tapa. Nyt meillä on käsissämme rootin hash: "$1$ciCib83f$p1yofmGYSQxu8OI2M8/Mz.". Alun "$1$" tarkoittaa MD5-crypt:iä, "ciCib83f" on sen 'suola', ja loput "p1yofmGYSQxu8OI2M8/Mz." on itse hash. 

Hashin voisi saada selville esimerkiksi hashcat -bruteforce-hyökkäyksellä, mutta se vie valtavasti aikaa ja käytännössä vaatii GPU:n, joten ratkaistaan se tällä kertaa hieman "huijaamalla".

Verkosta löytyy tietoa, että "Realtek"-pohjaisella Tapo-laitteella, rootin vakiosalasana on ollut "slprealtek" (https://github.com/nervous-inhuman/tplink-tapo-c200-re). Tämä koostuu raportoidusta rootin shell promptista "root@SLP" + Realtek pohjainen infrastruktuuri. Koska meillä on "Ingenic" pohjainen prosessorin infastruktuuri, voi olla pääteltävissä, että salasana olisi "slpingenic". Selvitetään se python-skriptillä:

    python3 -c "
    import subprocess
    target = '\$1\$ciCib83f\$p1yofmGYSQxu8OI2M8/Mz.'
    result = subprocess.run(['openssl','passwd','-1','-salt','ciCib83f','slpingenic'],
                             capture_output=True, text=True).stdout.strip()
    print('match' if result == target else 'no match')
    "

Tämän tulos on "match", eli rootin salasana on **slpingenic**. 

### Mitä haavoittuvuuksia ja miten ne voi löytää?

Aloitetaan selkeimmällä ongelmalla:

root salasana
* **Ongelmat:** Salasana on staattinen, toistuva hash monessa laitteessa. Kryptograafisesti "kestävä", mutta ei siltikään turvallinen (10 merkkiä pitkä, kaikki pieniä kirjaimia, ei numeroita/erikoismerkkejä). Salasana on tallennettu yksinkertaisesti "passwd" tiedostoon, joka on "legacy" tapa. Uudempi tapa "shadow":n sisään tallentamisellekin olisi mahdollinen ja osittain toteutettu, mutta ei kuitenkaan. 
* **Miten korjata:** Salasana saisi olla vähintäänkin dynaaminen, eli ei toistu useassa laitteessa. Salasana voisi pohjatua esimerkiksi uniikkiin laitetunnukseen. Tallentaminen pitäisi tehdä huoleellisemmin, ei vain tunnetussa hakemistossa esillä.

Verkkoyhteydet
* **Ongelmat:** Kaikki yhteydet (HTTP/RTSP/...) hoitaa squashfs-root/bin/main -tiedosto, root oikeuksilla. Koska kaikki hoidetaan rootilla, pääsy millä tahansa yhteydellä järjestelmään tarkoittaa sitä, että oikeudet ovat suoraan root, eikä esim. user josta pitäisi saada "eskaloitua" oikeuksia ylöspäin. 
* **Miten korjata:** Koska tiedostoissa jo mainitaan "admin", "user" yms, pitäisi toimintoja suorittaa minimioikeuksilla, eikä aina root-tason skripteillä. 


### Fiilikset tehtävästä
Loppuun valtava burn-out, siitä syystä esim. kohta 5 ja haavoittuvuusten kuvaus minimit. Työhön käytetty taas kivat osuudet päivistä la-ti, suuri osa hinkatessa edes takaisin tekoälyn kanssa, kun yritti saada roottia selville keinolla ja toisella. Pakko sanoa, että tehtävään ei varmaan ollut riittävää lähtötasoa, ja oppiminenkin jäi siitä syystä pienemmäksi kun mitä toivoisi. Ei ole muutenkaan mitään kokemusta, kaikenlaisista asioista, joista olisi ollut hyötyä esimerkiksi juurikin haavoittuvuusten löytämisessä (esim. se Ghidra, joka (jos ryhmiä olisi ollut vain yksi) olisi jo tutumpi). 

Liitän kuvaksi lopputilanteen tapo-kansiosta jossa tehtävä + kokeilut tehtiin (osa kokeiluista poistettu jo aikaisemmin).

<img width="1508" height="78" alt="image" src="https://github.com/user-attachments/assets/5307073b-ecd7-447f-b46f-0c67696906b6" />


## Lähteet
* Kurssin moodle sivu, "Sovellusten hakkerointi ja haavoittuvuudet - ICI012AS3A-3004 - 2026p1 - Tero ja Lari - to 14:00", välilehti "Hardware hacking". 
* https://github.com/robbins/tp-link-decrypt (valmis työkalu, jolla saa kaivettua avaimet purkamiseen)
* https://www.evilsocket.net/2025/12/18/TP-Link-Tapo-C200-Hardcoded-Keys-Buffer-Overflows-and-Privacy-in-the-Era-of-AI-Assisted-Reverse-Engineering (esimerkki siitä, kuinka ohjelmiston saa auki, ja mitä haavoittuvuuksia voi löytää)
* https://quentinkaiser.be/security/2025/07/25/rooting-tapo-c200/ (toinen esimerkki siitä, miten esimerkiksi root-salasanan voi saada selville)
* ilmainen OpenAI:n ChatGPT -laaja kielimalli. Käytetty 12.-14.9.2026. Saatavilla: chatgpt.com
* ilmainen Anthropic:n Claude Sonnet 5 -laaja kielimalli, effort tasoilla Medium ja High. Käytetty 14.-15.9.2026. Saatavilla: https://claude.ai/new
* https://github.com/nervous-inhuman/tplink-tapo-c200-re (toinen root salasana)
