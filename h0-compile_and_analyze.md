# h0 - Compile and Analyze

### Tehtävänanto omin sanoin:
* Luo oma lyhyt sovellus, esim. hello world käyttäen C/C++
* Käännä ohjelma binarykoodiksi, analysoi sitä

## Tulos:
Sovellus, joka käännetään:

  	#include <stdio.h>
	char* tekstiulkona = "ulkoarvo";
	int intulkona = 123;
  	int main() {
          printf("Kohta1\n");
          volatile char* volatilearvo = "Volatile nakyy";
          if (intulkona == 2) {
                  printf("Ehto nakyy");
          }
          static char* staticarvo = "Static nakyy";
          int eiloydy = 789;
          printf("Kohta2\n");
  	}
Käännetyn sovelluksen suorittaminen antaa seuraavat tulokset:
* Kohta1
* Kohta2
  
Saadaan siis sovelluksen oikea toiminnallisuus esiin; printataan kaksi "kohtaa". 

Binäärin tarkastelemalla kuitenkin selviää ohjelmasta enemmänkin. Käytetään "strings" binäärin tutkimiseen (strings ohjelman_nimi).
Pieni osio tuloksesta: 
  
	ulkoarvo
  	Kohta1
  	Volatile nakyy
  	Ehto nakyy
  	Kohta2
  	Static nakyy
  	;*3$"
  	GCC: (Debian 14.2.0-19) 14.2.0

Tästä saamme selville esimerkiksi sen, että: globaali char* (kuin string) muuttujan arvo saadaan käsiksi, mutta globaalin int-tyyppisen muuttujan arvoa ei. Samoin kaikki "volatile", "static" alkuiset muuttujat tulevat näkyviin. Myös koodissa olevan mahdottoman ehtorakenteen tulos saadaan ulos. HUOM! jos laittaa esim. "if (0) ..." se ei tule näkyviin. Tämä lienee johtuu kääntäjän automaattisesta siivoamisesta. Tästä syystä myös "int eiloydy" ja sen arvo eivät löydy binääristä. 

Lopusta löytyvä "GCC: ..." kertoo kääntäjän (compiler) version. Tämä on digitaalinen jälki, jota voidaan käyttää esim. virusten rikostutkinnassa. 

Pointtina se, että esimerkiksi salasanat tai API-avaimet eivät ole turvallisessa tilassa binäärissä, vaikka niitä ei käytettäisi koko ohjelman suorittamisen aikana. 

### Lähteet:
Työssä käytetty 20.8.2026 ilmaista Google Gemini -tekoälymallia. Tekoälymallia käytetty ymmärtämään paremmin strings-työkalua ja binäärin rakennetta. Prompteja esimerkiksi: "linux what is strings", "I would like to know how to show C variables in strings".
