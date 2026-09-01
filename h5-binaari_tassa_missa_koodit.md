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

Huomiona se, että vastaus voi tulla aika epäselvänä läpi. Tässä kuitenkin nyt esillä se, kuinka ajatusprosessi ja toiminta eteni, enkä riisunut ongelmia pois.

<img width="502" height="535" alt="image" src="https://github.com/user-attachments/assets/a6fb6766-1723-400c-b8fc-cbc565f6092a" />

Alkuun katsoin ohjelman lähdekoodia komennolla: "more gdb_example1.c". Sieltä koodin toiminallisuus ja virhe eivät tulleet heti selviksi, joten suoritin myös ohjelman komennolla "./gdb.example1", jonka tuloksena oli: 
    
    Khoor/#zruog1
    Segmentation fault

Nopealla silmäyksellä koodista ja tulostuksesta ilmenee selvitettävää: "register" -avainsana, bad_messagen arvo NULL ja "Segmentation fault" tuloksessa. Segmentation fault ei ole koodin toiminnallisuus, jolloin tämä kertoo siis virheestä, joka koodissa tapahtuu.
* Register-avainsana tarkoittaa, että se muuttuja tallennetaan mieluummin esimerkiksi prosessorin rekisteriin, eikä RAM-muistiin. Käyttötarkoituksena nopeuttaa arvon hakua muistista, ja ei pitäisi olla merkitystä tehtävään.
* NULL-arvo tarkoittaa tyhjää arvoa. Koska tehtävässä NULL ei ole "NULL", se on siis myös avainsana, eikä tekstiä kuten good_messagen arvo. Tämä aiheuttaa lähes varmasti segmentation faultin.
* Segmentation fault tarkoittaa käytännössä sitä, että ohjelma yrittää esim. päästä sisään muistiin paikkaan, johon sillä ei ole oikeutta. Koska segmentation fault ilmenee NULL -arvon yhteydessä, ohjelmalla ei siis ole pääsyä sinne, missä NULL arvo sijaitsee muistissa. 

Selvittääkseni mistä "Segmentation fault" johtui, kokeilin myös suorittaa ohjelman ilman "bad_message", sekä eri "register int i":n arvolla. Esimerkiksi i = 123:n tulos oli: "����꧛����ߩ", ja muutaman muun oli esimerkiksi: "Jgnnq."yqtnf0" ja "Rovvy6*�y|vn8". Sain tästä selville sen, että i:n arvolla on väliä tulostuksen sisällölle. Tämän tein myös lähinnä siksi, että haluttu tuloste on yhä hieman epäselvä. Ohjelman funktio "print_scrambled" kertoo, että sanasta "Hello, world." halutaan sekoitettu versio, mutta en ole varma haluttiinko ohjelmasta vain kirjainten järjestysten sekoitus, vai oliko  "Khoor/#zruog1" tosiaan hyvä tulos. Mennään kuitenkin sillä idealla eteenpäin, että good_messagen tuloste on haluttu versio, jolloin korjattavaa on vain "segmentation fault". 

Tässä vaiheessa oli pakko käyttää tekoälyä ymmärtämään ensin koko koodin toiminnallisuus, koska en ole käyttänyt C:tä aikaisemmin. Sieltä sainkin selville ne asiat, joihin jäin jumiin: 
* bad_message ja good_messagen "arvot" ovat osoittimia (pointer), ei suoraan mitä koodissa lukee
* Koska C -kielessä ei ole merkkijonoja (string), on merkkijonot kirjaimen (char) taulukkoja. Esim. siis good_message on: [H, e, l, l, o, ,,  , w, o, r, l, d, ., \0], jossa "\0" on tyypillinen null-päätösmerkki.
* Tähti (*) itsessään tarkoittaa "dereference", eli pointerista -> arvoon. " *message " tarkoittaa siis sitä arvoa, ei enää muistiosoitetta. Koska message on taulukko kirjaimia, *message palauttaa YHDEN merkin, ei koko jonoa. Tämä oli kenties suurin jumituskohta.
* "printf("%c", (*message)+i)" tarkoittaa käytännössä: ota *message:lla kirjain (esim. H), ja lisää siihen i (3). Koska kirjaimet ovat C:ssä ASCII muodossa, lisäämme oikeasti H (72) +3 = 75, joka on K. Tästä syystä tulos on siis "Khoor...". Printf:n alussa oleva "%c" kertoo, että tämä näytetään kirjaimena, eikä kokonaislukuna (integer), jolloin se olisi ollut 75.
* "while (*++message)": Tätä voidaan ajatella niin, että " *++message ":sta suoritetaan ensin ++message. Tämä "siirtää" osoittimen seuraavaan kirjaimeen, esim. H -> e. Sen jälkeen tehdään uuden arvon *message, jolloin saadaan itse arvo "e", eikä sen osoitin. Tällä hoidetaan myös suorittamisen lopettaminen: päätösmerkin "\0" arvo on 0, joka lopettaa ohjelman, koska "while (0)" on aina epätosi.

### Miksi ohjelma siis kaatuu, voiko sen korjata?
Ohjelma kaatuu, koska bad_message on pointer arvoon NULL. C-kielessä tämä tarkoittaa sitä, että se ei osoita mihinkään. Jos yrität "dereferensoida" NULL-osoittajaa, osoittaa se osoitteeseen "0", joka on käyttöjärjestelmän hallittavissa, eikä ohjelman. Yksi segmentation faultin syistä olikin juuri se, että ohjelmalla ei ole oikeutta sijaintiin. 

Ohjeet ovat mitättömät, jolloin on vaikea tietää suoraan miten ohjelma halutaan "korjata". Yksi vaihtoehto olisi vain poistaa NULL-osoittava bad_message, mutta tämä ei välttämättä ole käytännöllinen ratkaisu oikeassa ohjelmassa. Muutetaankin koodia mieluummin niin, että NULL-osoittavaa ei vain käsitellä. Tässä muokattu koodi:

    #include "stdio.h"
    
    void print_scrambled(char *message)
    {
      register int i = 3;
      if (message) {
        do {
          printf("%c", (*message)+i);
        } while (*++message);
          printf("\n");
      }
    }
    
    int main()
    {
      char * bad_message = NULL;
      char * good_message = "Hello, world.";
    
      print_scrambled(good_message);
      print_scrambled(bad_message);
    }

Ja tulos GNU Debuggerista:

<img width="741" height="140" alt="image" src="https://github.com/user-attachments/assets/6b407813-a4e0-4c76-8404-70ea9a4e3e96" />

Tämä toimii siksi, että lisätty "if(message)" katsoo, onko messagella pointeria. Normaalissa tapauksessa on, mutta NULL tapauksessa ei.

## Lab2:

Kansiossa on seuraavat tiedostot: Makefile (jolla voi kääntää ohjelman passtr), passtr, passtr2o, passtr.c, README.md (joka sisältää ohjeita tehtävän tekemiseen).

Tehtävässä halutaan, että ohjelmasta **passtr2o** löydetään salasana, ja mitä luultavasti oikean salasanan syöttäessä printattu lippu (flag). Aloitin ihan ensiksi tutustumalla passtr -ohjelmaan, suorittamalla sen (./passtr). Tästä ohjelmasta meillä on kuitenkin myös lähdekoodi, joten en pyöri sen parissa kauaa. Passtr -ohjelman salasanan saikin helposti saataville, muun muassa komennolla "strings passtr", tai katsomalla suoraan lähdekoodista. 

Seuraavaksi tietenkin piti myös kokeilla passtr -ohjelman salasanaa passtr2o tehtävään, mutta tehtävä ei ole sentään niin helppo. Koska passtr2o:sta ei ole lähdekoodia, katsoin myös sen sisällön strings-komennolla. Sieltäkään ei kuitenkaan saanut enää salasanaa, vain sen, että "What's the password?" rivin jälkeen tuli jälleen "%19s" (kuten passtr) ohjelmassa. Tämä kertoo siis, että jokin muuttuja on määritelty olemaan korkeintaan 19 merkkiä pitkä oleva string. Salasana on siis luultavasti korkeintaan 20 merkkiä päätösmerkin (\0) kanssa. Lisäksi strings-komennon avulla selvisi, että ohjelma sisältää "check_password" funktion. Tämä funktio on luultavasti se funktio, joka vertaa syötettä salasanaan.

GDB:lläkään ei salasanaa löydä niin helposti, koska lähdekoodin puuttuminen rajoittaa debuggerin toimintoja huomattavasti. Muun muassa breakpointteja ja watchpointteja ei saa juurikaan laitettua paitsi tunnettuihin funktioihin, joita yleensä on vain main, nyt myös check_password. Jotta ohjelmaa voi siis tutkia esim. disassemblerilla, pitää ohjelman olla käynnissä. Tehdään siis seuraavat asiat, kun ohjelma on avattu komennolla "gdb passtr2o"

    break main
    run
    disassemble

(huom. en laita breakpointtia funktiolle "check_password", koska alkukokeiluiden yhteydessä se ei toiminut). Nyt meillä on ohjelmasta Assembly-ohjeita. Tästä on kuitenkin vielä aika vaikea löytää mikä riveistä edes saattaisi sisältää etsimämme salasanan, saatika sitten miten sen saa annetulla informaatiolla selville.

Käytin tässä aika paljon aikaa tutkiessa eri assemblyn ohjeita, kuten mov, call, "jne", "cmp", "or", "jle", "je", "test", "sub" ja niin edespäin. Netistä löysin muun muassa seuraavia GDB:n ominaisuuksia, joilla voi tutkia rekistereitä:

    info registers
    x/s $<rekisteri> , esim. x/s $rsi

Jos haluaa myös nähdä assemblyn rivit uudestaan, voi käyttää komentoa:

    x/i $pc

Tämä näyttää nykyisen rivin, rivien määrää voi myös nostaa laittamalla i:n eteen jonkin numeron, esim. 10.
    
Kuitenkaan vielä salasanan löytäminen ei ole helppoa/mahdollista, koska emme ole edes syöttäneet vielä salasanaa. Laitetaan siis breakpoint johonkin väliin, missä salasana on jo syötetty, mutta ohjelman suoritus ei ole vielä päättynyt. Trial-and-error kokeiluden jälkeen huomasin, että riville "x5555555550fb <main+123>:	call   0x55555555525a <mAsdf3a>" oli paras sijainti. Hakasuluissa (<>) oleva "mAsdf3a" vaikutti olevan joku funktio, koska muistin sen olevan "strings" komennon listalla, ja siihen pystyikin laittamaan breakpointin suoraan nimellä.

    break mAsdf3a

Tässä vaiheessa poistinkin aikaisemman breakpointin mainista, ja käynnistin debuggaamisen uudelleen.

Nyt salasanan pystyi antamaan, ja ohjelma ei vielä keskeytynyt. Annoin salasanaksi "testisalasana". Tässä vaiheessa kävin myös katsomassa uusiksi rekisterit, ja yllätyksellisesti rekisterit rsi ja rbx sisälsivät syötteeni. Lisäksi rekisteri rdi sisälti myös kiinnostavan stringin, "anLTj4u8". Myös tämän muistin nähneeni "strings"-in avulla. Kokeilin myös tässä vaiheessa salasanaksi rdi:n sisältöä, mutta se ei kelvannut. (Rekisterit rsi, rbx ja rdi pitivät huomiotani eniten, koska ne olivat ainoat rekisterit, jotka sisälsivät suhteellisen selkeää tietoa. Muut olivat joko tyhjiä, tai epäselviä tuloksia, kuten %rip: "ATUH\211\375SH\...".

<img width="313" height="124" alt="image" src="https://github.com/user-attachments/assets/7075092d-5802-4b51-a6f0-7ccc3e63fe1e" />

Breakpointtia ei juuri pystynyt laittamaan myöhempään vaiheeseen ohjelmaa, koska silloin heti salasanan kysymisen jälkeen sovelluksen suoritus olisi päättynyt siihen, että virheellinen salasana. Koodissa on siis jokin ehto, joka lopettaa suorittamisen ajoissa, ja tietylle koodiriville ei siis ikinä päästä. Tästä syystä jäin hieman jumiin, joten jouduin kysymään tekoälyltä apua lähinnä Assemblyn tulkitsemiseen. Tuloksena olikin se, että koodi vertailee eräässä vaiheessa kahden stringin pituutta toisiinsa. Jos pituudet eivät ole samat, ohjelma päättyy. No, ainoat vertailtavat stringit joihin minulla oli vaikutusta oli tietenkin syötetty salasana, ja löydetty "anLTj4u8". Tekoäly suositteli kokeilemaan, mitä tapahtuu jos salasana on 8-merkkiä pitkä, kuten löydetty teksti. Alla kuva Assemblyn kohdasta, jossa vertailu tapahtuu:

<img width="672" height="215" alt="image" src="https://github.com/user-attachments/assets/266e6a1f-1621-44ff-bdc6-f025c9553542" />

Kun ohjelma kutsuu "call" strlen-funktiota, se laskee stringin pituuden. Se tapahtuu kuvassa kahdesti. Niiden tulos tallennetaan "mov" avulla johonkin muuttujaan yms. Loppupäässä oleva "cmp" on compare eli vertaile. Se vertailee tulosten pituutta. Sen jälkeen tuleva "jne" (jump if not equal) tarkoittaa sitä, että koodi hyppää toiseen kohtaan, jos edellä oleva compare ei ollut equal, muuten toiminta jatkuu lineaarisesti. Jos salasana ei siis ole 8-merkkiä pitkä, ohjelma hyppää osoitteeseen "0x5555555552af <mAsdf3a+85>". Tämä sattuu olemaan aivan koodin loppupäässä, jossa käy jo "siivousoperaatiot", ja ihan hetken päästä myös ret (return), eli ohjelman lopetus.

Nyt tiedetään, että seuraava breakpoint kannattaa laittaa vertailun jälkeen, ja syötteeksi antaa 8 merkkiä pitkä salasana. Annetaan salasanaksi vaikkapa salasana, ja breakpointiksi muistiosoite "0x555555555281", joka on heti "jle" -ohjeen jälkeen. Teen tämän kokonaan uudessa gdb-instanssissa jotta poistuu aikaisempi breakpoint ja kesken jäävä ohjelma saadaan nopeasti suljettua. Tehdään siis: quit -> gdb passtr2o.

    break *0x555555555281
    run
    salasana

Nyt rekisterit ovat muuttuneet hieman, esimerkiksi "salasana" löytyy rekistereistä rbx ja rdi, ja ohjelman teksti rekisteristä rbp. Katsotaan taas ohjelmakoodia, "x/30i $pc".

<img width="674" height="610" alt="image" src="https://github.com/user-attachments/assets/2221372b-769b-4172-9543-52be4992734a" />

Tässä vaiheessa selkeyttäisi, jos rekisterit ja niiden sisältö on kirjattuna ylös. Tässä siis nykyinen tilanne päätellen aikaisempia tapahtumia (ja Assemblyä) sekä avattavia rekistereitä:
* %rbp = "anLTj4u8"
* %rbx = "salasana"
* %rax = indeksi (esim. jokutaulukko[indeksi], hakee siis kirjaintaulukosta (stringistä) tietyn kirjaimen indeksin kohdalla).
* %ecx = "salasana" -stringin kirjain kohdassa indeksi
* %edx = "anLTj4u8" -stringin kirjain kohdassa indeksi

Rivi "0x555555555290 <mAsdf3a+54>:	test   $0x1,%al" tarkoittaa: "Onko indeksi tällä hetkellä parillinen/pariton?" ja sen pohjalta tehdään päätös "je":n avulla: jos parillinen, lisätään kirjaimen arvoon 3 (0x555555555299 <mAsdf3a+63>:	add    $0x3,%edx), jos pariton, vähennetään sen sijaan 7 (0x555555555294 <mAsdf3a+58>:	sub    $0x7,%edx). Oletan siis, että tämä on loop, joka käy koko kirjainketjun rekisteristä edx ("anLTj4u8"), läpi ja nostaa tai laskee kirjainten ASCII-arvoa ehdon mukaan. Lopussa oleva cmp vertaa %ecx (syöttämämme salasanaa) ja uutta %edx (muokattu "anLTj4u8").

Seuraavalla komennolla saa 8 kirjainta ja niiden ASCII luvut %rbp:stä:

    x/8cb $rbp

Lisäämme siis parillisiin 3, vähennämme parittomista 7. Esim. siis "a" on 97, jolloin siitä tulee 100, "n" on 110, jolloin siitä tulee 103 jne. Salasana on siis **dgOMm-x1**, Lippu: **FLAG{Lari-rsvRDx04WMBZpuwg4qfYwzdcvVa0oym}**

<img width="690" height="86" alt="image" src="https://github.com/user-attachments/assets/ff4a1f9b-ba65-4894-b7b0-5633f3f4e44f" />


## Lab3:

Tehtävässä pitää valita yksi crackme:stä. Katsoin Noran sivustoilta tehtävistä hieman lisätietoa, ja ensimmäiset kaksi olivat liian helppoja, kolmas taas turhan vaikea? Otin kuitenkin kolmannen, tiedoston **crackme03e.64**. En käytä lähdekoodia tehtävän ratkaisemiseen.

Aloitetaan taas tyypillisellä tavalla (huom. koska ohjelma haluaa heti parametrinä salasanan eikä kysy sitä jälkikäteen, lisätään se "run" jälkeen):

    gdb crackme03e.64
    break main
    run salasana

<img width="805" height="758" alt="image" src="https://github.com/user-attachments/assets/9784b392-3b1b-4d6b-ae7f-4c5b08fe5b54" />


Tämä lähdekoodi näyttääkin lab2 verrattuna hieman monimutkaisemmalta. Ohjelma sisältää nimittäin enemmän mov, movb, movabs ja lea -ohjeita, jotka voivat kaikki liittyä tavalla tai toisella datan siirtämiseen tai käsittelyyn. Ensin lähden ohjelmasta etsimään suuntaa, mihin haluamme päästä. Esimerkiksi puolivälistä eteenpäin ohjelmasta löytyy kolme kommentoitua (#...) muistiosoitetta. Nämä ovat tyypillisesti printtejä, jolloin etsin näistä ensin hyvän tuloksen printin (ylhäältä alas):
* x/s 0x555555556034 => "No, %s is not correct.\n"
* x/s 0x555555556004 => "Need exactly one argument."
* x/s 0x55555555601f => "Yes, %s is correct!\n"

Tästä saadaan selville se, että rivi jolla printataan "Yes..." on haluamamme paikka. Se on siis: "0x000055555555521c <+143>:	lea    0xdfc(%rip),%rdi". Sitten katsomme ylempää, mitkä ehdot täytyy täyttyä, jotta pääsemme hyppäämään oikeaan tulokseen.

<img width="676" height="402" alt="image" src="https://github.com/user-attachments/assets/3594d563-2261-476c-89be-d6bd0af38488" />

Kuvassa alin rivi on nyt "0x00005555555551e5 <+88>:	jne    0x555555555219 <main+140>". Se vie hyvin lähelle haluamaamme osoitetta, vain 3 ohjetta edellä. Voimme heti ohjeista ottaa ensimmäisen jne-ohjeen ja sitä edeltävät pois tarkemmasta huomiosta, koska ne johtavat vastaukseen "Need exactly one argument". Se siis vasta käsittelee syötettyjen argumenttien määrää, ei sisältöä. Siihen väliin voisi siis laittaa esimerkiksi breakpointin, joka ei kuitenkaan nyt ole tarpeellista.

Siitä seuraava jne johtaa väärään salasanaan, jolloin meidän pitää varmistaa, että sitä ennen oleva "cmp" -vertailukäsky palauttaa tosi. Kyseinen cmp vertaileekin "$0x8" ja "‰rax", eli käytännössä katsoo, onko rax koko 8. Rax taas tässä tarkoituksessa tarkoittaa "call strlen" kanssa strlenin palautusarvoa, eli rdi:n pituutta. Tyypillisesti rdi varsinkin ennen "call" on ensimmäinen funktion parametri -> strlen(rdi). %rax tai %eax (rax:n alimmat 32-bittiä) ovat varsinkin funktion jälkeen tyypillisiä palautussijainteja. Esim. jos funktio palauttaa 8 ja sitä seuraa %rax, rax pitäisi olla 8. Koodi siis selvittää, onko rdi:n pituus 8, jos ei -> väärä salasana. Halutussa tilanteessa rdi:n pituus on siis 8.

No, mikä on rdi, eli rbx? Sitä varten voimme mennä eteenpäin ohjelmassa, vaikkapa juuri ennen, kuin tärkeä "cmp" suoritetaan:

<img width="511" height="175" alt="image" src="https://github.com/user-attachments/assets/53ace03e-c8f8-4a86-8bc8-58ac1fd0cd2e" />

Tästä voimme nyt nähdä, että rax on tosiaan 8, ja rdi sekä rbx ovat molemmat "salasana", se minkä syötin. Tuleva cmp ja jne eivät tule siis olemaan ongelma, sillä jatkamme niiden ohi 8-merkkisellä salasanalla "salasana". Siirryn samalla juuri ennen siirtymistä "check_pw" -funktioon. Katson vielä rekisterit rdx, rsi ja rbx, sillä niitä käsitellään juuri ennen salasanan tarkistusfunktiota. 
* x/s $rbx => "salasana"
* x/s $rsi => "password"
* x/8bx $rdx => "0x03 0x05 0x02 0x04 0x01 0x00 0x03 0x01" (tarvittu tähän hieman apua, koska tarvitiin "x/8bx")
  
Tämä tehtävä näyttää toimivan suuremmaksi osaksi samalla tavalla kuin lab2:ssa ollut pala tekstiä. Tätä "password" luultavasti muunnetaan jollain luvuilla, jolloin siitä tulee oikeasti haluttu salasana. Sitten sitä verrataan "salasana":aan. Tämän varmistamiseksi meidän pitää kuitenkin myös käydä "check_pw":n sisällä:

    break check_pw
    c

Nyt, kun olemme pysähtyneenä check_pw-funktion sisään, katsotaan jälleen ohjeita: 

<img width="779" height="360" alt="image" src="https://github.com/user-attachments/assets/cf5b8677-51b7-445d-8642-751902f9fdd5" />

Alun "mov $0x0, %eax" tarkoittaa, että eax-rekisteriin siirretään arvo 0, jolloin sen arvo on siis 0. Koska eax on rax:n alimmat 32-bittiä, myös rax on nyt 0. 

Seuraava ohje "movzbl (%rdx,%rax,1),%ecx" tarkoittaa käytännössä rdx + rax * 1, eli rdx + 0*1 = rdx. Tämä tallennetaan ecx:ään. Huom. rdx:ssä sijaitsee "pointer" muistipaikkaan, missä itse arvo sijaitsee. Koska rdx oli outo string numeroita, Pelkkä rdx (tai rdx indeksillä 0) tarkoittaa siis ensimmäistä tavua, "0x03" tai vain 03. 

Sen jälkeen tuleva "add (%rsi,%rax,1),%cl" tarkoittaa, että lisätään cl-rekisterin osoittamaan paikkaan rsi:n osoittama arvo. Tässä tapauksessa, koska "indeksi" (rax,1) on edelleen 0, tarkoittaa rsi:n ensimmäistä tavua, kirjainta "p". Tämä muutetaan on ASCII numeroarvo. Cl ei nyt kuitenkaan sisällä 70, vaan 73, koska %cl sattuu olemaan %ecx:n lopputavuja. 

Sitten tuleva "cmp %cl,(%rdi,%rax,1)" tekee käytännössä 73 == %rdi indeksillä 0, eli 73 == %rdi :n ensimmäinen tavu. Aikaisemmin laitoin, että meidän salasana oon rbx, mutta nyt uudelleen katsomalla, rbx sisältää pointterin "salasana" stringiin. Rivi siis vertaa "73" ja "s", eli 73. Koska ne ovat samoja, seuraavaa "jne" ei käynnistetä. Jos kirjaimet eivät ole samat, esim. 73 == 70 eli epätosi, koodi hyppää kohtaan "mov 0x0,%eax", "ret", eli palauttaa 0, huono tulos. Tässä funktion ainoa epäonnistumisen kohta. 

"cmpb   $0x0,0x1(%rsi,%rax,1)" tarkoittaa, että verrataan "0x0" eli 0 ja %rsi indeksillä +1, eli RSI + 1 (aikaisemmin oli RSI + 0). Seuraava "je" tarkoittaa jump if equal, eli hyppää kohtaan "mov 0x1,eax" "ret". Eli jos viimeinen merkki on 0, palauta 1, eli hyvä tulos. Tämä on siksi, että string loppuu päätösmerkkiin "0\" joka on 0. Jos ehto ei täyty, jatkamme:

"add 0x1,%eax" eli lisää eax/rax +1. Tämä on siis indeksin korotus, koodissa esim. "i++". 

Sen jälkeen verrataan taas 0 ja rdi + indeksi. Jos tämä on tosi (tulos 0), palauttaa ohjelma 1, eli hyvä tulos. Muuten, aloitetaan ohjelma uudelta loopilta.

### No miten se salasana nyt saadaan siis selville?

Yksinkertaisesti, voidaan lisätä jokaiseen RSI kirjaimeen saman indeksin RDX (rsi on "password", RDX on "0x03 0x05 0x02 0x04 0x01 0x00 0x03 0x01". Esim. rsi[0] + rdx[0] => 70 + 3 = 73 = "s". rsi[1] + rdx[1] => "a" -> 97 + 5 = 102 = "f"... == **sfuwxoue**. 

<img width="611" height="37" alt="image" src="https://github.com/user-attachments/assets/c9f7f303-b2dc-4482-ad2d-1925d665397a" />


## Lähteet
* Lab1: https://en.wikipedia.org/wiki/Register_(keyword) (mikä on register-avainsana)
* Lab1: https://www.scaler.com/topics/segmentation-fault-in-c-cpp/ (mikä on segmentation fault)
* Lab1: https://stackoverflow.com/questions/26362340/printing-null-pointer-gives-segmentation-fault-core-dumped (miksi null-pointer johtaa segmentation faultiin)
* Lab1: Käytetty ilmaista OpenAI:n ChatGPT -laajaa kielimallia 31.8.2026. Syötteenä: "Tässä koulutehtävän koodi, voisitko selittää sen toiminnallisuuden, ilman, että ratkaiset sen ongelmia."
* Lab2: Käytetty ilmaista OpenAI:n ChatGPT -laajaa kielimallia 1.9.2026. Mallia käytetty ymmärtämään tiettyjä osia assembly-koodista ja GDB käskyjä, kuitenkin määräten, että antaa vain pieniä ohjeita, eikä ratkaise tehtävää suoraan.
* Lab2: https://www.rapidtables.com/convert/number/ascii-hex-bin-dec-converter.html (paikka jossa laskin lab2 salasanan)
