# Konfiguracija
## Uvod

Konfiguracija i parametrizacija su sinonimi u smislu ovog dokumenta i oba izraza se ravnopravno koriste.

Pod konfiguracijom se podrazumevaju podaci na osnovu čijih se različitih vrednosti aplikacija ponaša na različite načine, čime se omogućava da se isti scenariji obrađuju različito, ili čak da postoje različtii scenariji koji rešavaju isti zadatak.

U ovom dokumentu pod konfiguracijom se podrazumeva opis celog modula konfiguracije koji se sastoji od: 
- microservice koji se bavi svim aspektima obrade podataka koji se koriste da bi ostali microservice radili prema zahtevima. Da bi dokument bio lakše čitljiv microservice koji se bavi ovom tematikom (i koji se opisuje u ovom dokumentu) zovemo **Config MS** 
- front aplikacija koje služe da korisnik može podašavati parametre na user frendly način 
- batch i pomoćnih aplikacija koje omogućavaju: prenose konfiguracije od Badin preko partnera do krajnjih korisnika i od testa na produkciju; testiranje integriteta konfiguracija

## Definicije

## Krajnji korisnici

Krajnji korisnik je firma koja koristi softver za svoje potrebe. Ukoliko ta firma ima svoje korisnike njih ćemo zvati klijenti u ovom dokumentu.

## User
 
User je zaposleni pojedinac koji koristi aplikaciju. Preciznije, user ne mora da bude zaposlen kod krajnjeg korisnika, ali mora biti ovlašten za korišćenje aplikacije od menadžmenta krajnjeg korisnika.

## Tenant

Konfiguracija podržava multitenant instalacije. To podrazumeva da na istoj instalaciji mogu da rade potpuno nezavisni krajnji korisnici koji nemaju nikakvih dodirnih tačaka. 

### Učesnici u proizvodnji i dorada softvera

Lanac učesnika u proizvodnji softvera obuhvata:
- Badin soft - kao osnovni proizvođač svih sistema koji će korisiti ovaj sistem. 
- Partneri - jedan ili više partnera u lancu čiji je zadatak da kastomizuju softver za drugog partnera ili za krajnjeg korisnika. Zadnji u lancu partnera može biti zadužen i za implementaciju mada implementaciju može raditi i sam krajnji korisnik
- krajnji korisnik - može da vrši kastomizaciju softera, može i implementaciju, ali sigurno radi konfiguraciju. Krajnji korisnik takođe može imati više mesta gde se radi proizvodnja i dorada softvera ali je samo jedno mesto gde se može izvršiti implementacija.

Pod kastomizacijom softver u smislu ovog člana se podrazumeva mogućnost prilagođavanja ponašanja softvera prema potrebama korišćenjem konfiguracije ali i izmenama u kodu prema definisanim pravilima od strane Badin soft ili za potpuno dodavanje novog koda koji rešava scenarije koje nisu obuhvaćeni inicijalnom proizovdnjom (od strane Badin softa) ali takođe prema pravilima koje je podesio Badin soft.

### Okruženje

Okruženje ima isti sistem konfiguracije i ponašanja na jednoj instalaciji i za jednog tenanta. Okruženje se identifikuje šifrom okruženja. 
Podrazumeva se da su tenant-i na jednoj instalaciji različita okružanja ali se ona identifikuju šifrom tenanta.  

### Mesto unosa i izmene

Pri definicije konfiguracije razlikujemo prvi unos i kasnije izmene. Mesto gde se prvi put unosi naziva se Mesto unosa (i to može biti Badin, bilo koji Partner u nizu, ili krajnji korisnik) a sva ostala mesta su mesta izmene (i mogu biti bilo koji učesnik u nizu posle Mesta unosa).

Potpuno je drugčaiji način rada pri prvom unosu konfiguracije i prilikom njegove modifikacije. Razlog je to što je potrebno ispuniti zadatak da se u novim verzijama prenosi sve ono što je izmenjeno na izvoru, osim onoga što je menjano na ciljnoj lokaciji. U ovom sistemu će biti urađena i dodatna opcija da ciljno mesto bude obavešteno o promenama izvornog parametra koji je promenjen na ciljnom mestu. 

## Ciljevi i zadaci sistema konfiguracije

### Efikasnost

Glavni fokus u smislu efikasnosti je da API u Config MS koji daje parametre ostalim MS. Ukoliko je potrebno smanjuje se efikasnost u deficijama i izmenama parametara da bi se povećala efikasnost u usluživanju ostalih MS. 

### Omogućavanje različitih parametara na jednom sistemu

#### Različiti parametri kod različitih učesnika u  lancu proizvodnji softvera

Potrebno je omogućiti da svaki učesnik može da promeni konfiguraciju, ali tako da sledeća verzija konfiguracije od prethodnog učesnika ne utiče na promene kod učesnika, ali **obavezno** menja sve promenjene i nove konfiguracije. 
Ovo se zove **statička promena konfiguracije**.



#### Različite konfiguracije prema okruženju

Potrebno je omogućiti da se može dobiti različita konfiguracije prema okruženju gde se zahtev za konfiguraciju realizuuje. Okruženja mogu biti različita zbog želje da se jedna aplikacija ponaša drugačije kada se zahtev dolazi (ovo su najčešći primeri upotrebe):
- sistem radi u različitim zemljama
- sistem radi u različitim regionima
- postoje posebne organizacione celine
- želi se dozvoliti da pojedini korisnici aplikacije (user) 
- posotoji potreba za korišćenjem aplikaije u uslovima koji nisu redovni (recimo posotji potreba da se vraćamo u prošlost mnogo više nego što je to dozovljeno u redovnom poslovanju)
Ovo se zove **dimačka promena konfiguracije**

### Prenos konfiguracije sa jednog mesta na drugo

Potrebno je omogućiti da se prenosi konfiguracija sa jednog mesta na drugo. Uobičajeno je da se prenosi konfiguracija po liniji proizovdnje softvera, ali takođe i sa testnog okruženja na produkciono okruženje. Konfiguracija se može prenositi cela ili u delovima (u kom slučaju se kombinuje sa konfiguracijom na ciljanom mestu). Prenos mora biti potpun (da se ne ispusti ni jedna promena) i bezbedan (da kritični podaci nisu dosupni neovlašćenim osobama).

### Referenciranje i nasleđivanje

Potrebno je omogućiti da se svaki elemenat konfiguracije referencira na bilo koji drugi elemenat konfiguracije i da od njega nasleđuje svaku promenu. Ukoliko je parametar složen (objekat ili niz) potrebno je omogućiti da bilo koji elemenat (bez obzira na dubinu) može da referencira bilo koji element iz takođe složenog parametra.

### Vremenska dimenzija parametara

Svi parametri moraju da imaju period važenja (početak i kraj). Vremenska dimenzija je u sekundama. 

## Opis sistema

### Modularnost

Config MS je centralizovan i zajednički MS za sve module u okviru jedne aplikacije. I kao takav je potrebno da bude instaliran na svim mestima. Međutim konfiguracija je stritno raspoređena po modulima i potrebno je instalirati samo onu konfiguraciju za module koji su instalirani. 

### Glavna podela sistema konfiguracije

S obzirom da su zahtevi za sistem kongiruacije zahtevni, a često i kontradiktorni da bi bilo moguće odgovoriti na zahteve potpuno je razdvojen sistem konfiguracije na sledeće delove:
- definicioni deo sistema čiji je zadatak da omogući izmenu i definisanje parametara, da se tako definisani parametri mogu prenositi sa jedne instalacije na drugu instalaciju, i da proizvede rezultujuću konfiguraciju koja se koristi u drugom delu sistema. 
- izvršna konfiguracija - ovaj deo sistema je zadužen za efikasno odgovaranje na zahteve o vrednosti parametara putem API. Zahteve postavljaju drugi MS. 

Fokus na efikasnost je u Izvršnoj konfiguraciji dok je u definicionom delu sistema fokus na user frendly definicije. 

Pod definicioni deo sistema spada i kompletna podrška za prenos i testiranje sistema. 

### Vrste parametara

Parametri mogu biti:
- Jednostavni
- Složeni 
- Šifarnici
- Tabele odlučivanja (Decision table) 
- Specijalni parametri

Jednostavni parametri kao vrednost imaju skalarnu vrednost tipa: 
- string  (string)
- datum (date)
- datum i vreme (time) 
- ceo broj (int)
- raconalni broj (double)

Složeni parametri su:
- object
- array (niz skalarnih vrednosti istog tipa)
- array of object (niz objekara različitog tipa)

Šifarnici su liste podataka koji služe kao enumeracije u modulu koji je vlasnik šifarnika. Postoji nekoliko tipova šifarnika koji su obrađeni u ovoj konfiguraciji:
- enumeracija - sadrži samo šifru
- lista - sadrži šifru i opis (naziv) šifre
- lista sa vrednosti - sadrži šifru, vrednost šifre i opis (nazi šifre)
Šifarnici su vlasništvo modula, tenanta i okruženje. 

Tabele odlučivanja (slično kao u DMN 1.1 standardu) su proizvoljne tabele koje imaju nekoliko ulaznih kolona i nekoliko izlaznih kolona. U ulaznim kolonima se definišu izrazi koji ako su zadovoljeni daju rezultat vrednosti u izlaznim kolonama. Ukoliko prvi red rezultat izraza nije istinit, prelazi se na sledeći red. Ukoliko nije zadovoljen ni jedan red reziltat je definisan pod ostalo, ili ako nije ni on definisan, podiže se greška. Rezultat odluke se koristi za menjanje toka procesa, ali se može koristi i za druge svrhe (izračunavanje...)

Specijalni parametri su opisani u sledećem poglavlju.

Parametar se identifikuje nazivom. Pored naziva parametar ima i opis čiji je zadatak da približi user-ima čemu služi parametar. Pored ovih podataka postoje podaci kojim se definiše koja je vrsta paramtera i koji su tipovi dozvoljeni.

#### Specijalni parametri 

Specijalni parametri su u podaci i tabele koje po svojoj prirodi se ne bi mogli baš svrstati u konfiguraciju, više su možda i microservisi ali su dosta potrebni ostalim delovima konfiguracije pa su zato dodati u konfiguraciju. Moguće je da neke od ovih specijalnih parametara čuva neki drugi microservice koji ne pripada sistemu kome pripada konfiguracije (ne pripada Badin ili Badin partnerima). U tom slučaju se radi integracija i umesto podataka iz konfiguracije se koirste podaci iz third party service. 

##### Organizaciona struktura

Organizaciona struktura je hijerahijska slika svake organizacije koja se sastoji od delova ili timova. Ukoliko je organizacija flat i ne postoje timovi (u tom slučaju je verovatno i veoma mala) i dalje ima organizacionu strukturu ali je ona jednostavna sastoji se samo od jednog organizacionog dela.

Organizaciona struktura u konfiguraciji podrzaumeva da ima dve hirerahije:
- first line nadređena (nekad poznata kao primarna, vertikalna, osnovna...)
- second line nadređena (ponekad poznaka kao sekundarna, horizontalna, teritojialna...)
Jedna organizacija mora imati barem jednu hijerarhiju (first line) dok drugi nivo hijerarhije nije obavezan.

Jedna organizaciona jedinica može imati samo jednu first i second line nadređenu organizacionu jedinicu. 

Jedna organizaciona jedinica ima jednog menadžera. Najčešće organizaciona jedinica ima flat strukturu, ali to nije uslov. Može imati i grupe u okviru sebe ili pojedince sa posebnim ovlašćenjima u timu,  ali ako oni ne predstavljaju oficijalnu hijerarhiju vode se kao jedna organizaciona jedinica. Nadređeni menadžer menadžeru organizacionu jedinice je menadžer nadređene organizacione jedinice. To znači da svaki menadžer ima first line i second line nadređenog menadžera. 
Treba obraitti pažnju da se ovde priča o 1st level naređenoj. Pored 1st level naređenosti može postojati 2nd line (kao i 3rd...) u zavisnosti od dubine organizacione strukture.

Organizacione jedinice ali i zaposleni mogu imati dodatne hijerahije ili grupisanja ali to nije predmet ove organizacione strukture. 

### Izvršna konfiguracija 

#### API 

##### API: GET Vrednost parametara

Ovaj API ima zadatak da vrati vrednost parametra koji se traži. Odnosi se na jednostavne i slođene parametre.
Ulazni parametri:
- modul 
- tenant
- okruženje 
- vreme (ukoliko je izostavljeno podrazumeva se sadašnje vreme)
- naziv parametra
- putanja do elementa parametra - ovo se koristi ukoliko se iz složenog parametra želi dobiti samo jedan elemenat. Primer ako imamo parametar Granične vrednost koji je JSON u obliku:
    {maxValue: 1000,
     minValue: 10,
     curency: {code: 891, label: EUR} a pozivom se đeli dobiti šifra valuta poziv bi bio /curency/code

Izlazni parametar:
- JSON sa vrednošću parametra. Za jednostavne parametre JSON je u obliku { value: vrednost parametra} dok je za slo\ene parametre JSON koji reprezentuje složeni tip. 

##### API: POST Setuj vrednost parametara

Ovaj API ima zadatak da setuje vrednost parametra ukoliko je za parametar dozvoljeno da se menja od strane modula. Generalno parametri se menjaju kroz konfiguracioni sistem koristeći front aplikaciju, ali je moguće za određene parametre (koji se tako podese) menjati kroz modul programski kroz ovaj API. Odnosi se na jednostavne i složene parametre.

Ulazni parametri:
- modul
- tenant
- okruženje
- naziv parametra
- vreme važenja od - do
- vrednost (JSON)

##### API: GET Šifarnik celi

Ovaj API vreća podatke iz šifarnika koji je ulazni parameter. Izlaz je JSON a u zavisnosti kog je tipa šifarnik, izlazna lista će imati jednu ili više lista. 

Ulazni parametri:
- modul 
- tenant
- okruženje 
- vreme (ukoliko je izostavljeno podrazumeva se sadašnje vreme)
- šifra šifarnika

Izlaz:
- JSON sa listom (array) šifara koji su takođe pojedinačni JSON a sadržaj je:
    - šifra
    - opis - osim za enumeracija tip šifarnika
    - vrednost - samo za liste sa vrednošću

##### API: GET Testiranje šifre

Ovaj API služi da se testira da li postoji šifra u šifarniku koji su ulazni parametri. Izlaz je true ako postoji. 

Ulazni parametri:
- modul 
- tenant
- okruženje 
- vreme (ukoliko je izostavljeno podrazumeva se sadašnje vreme)
- šifra šifarnika
- šifra

Izlaz:
- JSON sa atributom value koji može imati vrednost true ako postoji šifra i false ako ne postoji tražena šifra u šifarniku


### Database

Podaci u bazi za ovaj deo podsistema je optimizovana za davanje brzih odgovora za sadašnje vreme, pa su podaci podeljeni u tri tabele: prošlo vreme (oni koji su prestali da važe), važećii podaci i budući podaci (koji će važiti od određenog trenutka). Ukoliko se potraži podatak u vremenu koji ne odgovara važećem podatku pretražuju se druge tabele već prema vremenima koja su u zahtevu. S obzirom da je očekivano da zahtevi se uglavnom daju u sadašnjem vremenu (veliki procenat zahteva) ova optimizacija je opravdana. 

### Definicije konfiguracije

#### Definicije parametara

#### Razlika u definicija u prvom unosu i sledećim izmenama

Jedan od glavnih zadataka je da se definicija parametara prenose kroz lanac učesnika a da se pri tome održi mogućnost izmene podataka u novoj verziji od strane prethodnog učesnika u lancu a istovremeno se održe i promene koje je napravi trenutni učesnik u lancu. Ovo se može raditi na nivou celog objekta (npr. da ako se izmeni jenda šifra u šifarniku više se ne prenosi ceo šifarnik ili ako se izmeni jedan atribut u složenom parametru ne prenosi se ceo paramtar) ali u ovom sistemu se planira mogućnost izmene na najnižem nivou, što znači da se prenosi izmenjeni objekat osim dela koji je izmenjen.

Zbog takvog zahteva postoje razlike prilikom prvog unosa i prilikom izmena kod sledećeg u lancu učesnika u izmeni i doradi softvera. Dok je prvi unos prilagođen da se definiše podatak, a zatim i unese, izmena kod sledećih učesnika u lancu je u stvari definicija izmene da bi u sledećoj verziji konfiguracije izmene ostale. Naravno, i UX je različit u ta dva scenarija. 


##### Jedinstvenost identifikatora

Zbog potrebe da pojedini identifikatori budu jedinstveni kroz ceo lanac proizvođača softvera uvodi se jedinstvena oznaka učesnika u proizvodnji softvera. Oznaku određuje Badin, a ona se dopisuje kao sufix u obliku __*identifikator*

##### Statusi pri unosu/izmeni parametara

Postoje dve vrste statusa pri unosu/izmeni parametra. Prvi se odnosi na faze unosa ondosno spremnosti za distribuciju i produkciju, a druga vrsta statusa je da li je to novi ili izmenjen parametar.

U procesu unosa/izmene parametara postoji nekoliko faza/statusa:
- priprema (PREPARE) - u ovoj fazi parametar se nalazi u trenutku kada se želi unositi/menjati komplikovaniji parametar, a ne želi se uticati na ostale delove sistema. U ovom statusu parametar je nedivljiv za ostale funkcionalnosti osim za unos/izmenu.
- test (TEST) - u ovoj fazi parametar je spreman za testiranje na mestu unosa i uzima se u obzir za generisanje rezultujućeg parametra ukoliko je status okruženja TEST. U ovom statusu se može menjati, ali u tom slučaju odmah se menja i rezultujući parametar pa izmena u ovom statusu pa je potrebno oni koji rade promenu budu za to obučeni. Takođe iz ovog statusa se može vratiti u PREPARE status.
- spreman za distribuciju (READY) - u ovoj fazi parametar je spreman za distribuciju i može se prenositi sve do produkcije. Izmene su moguće u hitnim slučajevima, ali se ne preperučuju. Svi parametri na produkciji moraju biti u ovom statusu, osim u slučajevima hitne intervencije.  


##### Unos novog parametra

Novi parametar se unosi samo na mestu unosa (što je i logično pošto je to prvi unos). 

Parametar se ientifikuje imenom (jedinstven identifikator). Pored imena unose se sledeći atributi parametra:
- naziv - tekstualni naziv parametara. U stvari ovo je kratak opis parametra.
- opis - tekstualni opis koji treba detaljnije da opiše čemu služi 
- tip parametara 
- opis definicije parametra koji služi (pogledati posebno poglavlje)
- vrednost parametra u zavisnosti od tipa parametra, a u skladu sa opisom definicije
    - za proste parametre unosi se vrednost
    - za složene paremetre gradi se JSON objekat sa atributima (koji takođe mogu biti JSON objekat)
    - za šifarnike se unosi tip šifarnika pa u zavisnosti od tipa unose se podaci prema tipu šifara
    - za  tabele odlučivanja unosi se tabela prema definiciji (pogledati posebno poglavlje)
- vreme unosa

#### Opis definicije parametara

Jedna od funkcionalnosti ovog dela sistema je opis definicije parametara koja sliži kontroli unosa čime se postiže sigurniji rad aplikacije i povećava UX. 

##### Jednostavni parametri

Za jednostavne parametre se definiše tip parametra:
- ceo broj (integer) - uz mogućnost ograničenja:
    - da bude pozitivan 
    - da bude pozitivan i nula
    - da bude u rangu
- decimalni broj - uz mogućnosti ograničenja iste kao za ceo broj
- datum - uz mogućnosti ograničenja:
    - da bude u prošlosti
    - da bude u prošlosti i danas
    - da bude u budućnosti 
    - da bude u budućnosti i danas
    - da bude u rangu (razlike u odnosu na današnji dan)
- vreme - uz mogućnosti ograničenja ista kao za datum

##### Složeni parametri - Object

Složeni paraemtri su u stvari objekat (koji je definisan u JSON obliku). Objekat je u stvari rekurzivna definicija gde ćlan objekta može biti takođe biti objekat. 

Objekat može imati jedan ili više nivoa. Pod nivoom se podrazumeva dubina pojavljivanja složenih objekata (object, array). Nivo se povećava kada član u objektu nije skalarni tip već složeni tip.

Za svaki nivo se definiše sledeće:
- obavezni članovi (njihova imena) - članovi koji se moraju naći u objektu i imati vrednost (vrednost NULL je regularna vrednost). Ukoliko se ne nađe član u objektu a definisan je kao obavezan biće prijavljena greška. Za svaki član se definiše tip:
    - skalarni (ceo broj, decimalni broj, datum i vreme)
    - objekat (rekurzivna definicija)
    - niz (koji ima elemente koji mogu biti skalarni tip ili objekat)
- mogući članovi (njihova imena ) - članovi koji se mogu naći u objektu. Ukoliko ovi članovi nisu definisani u objektu smatraju se kao undefined i kao takvi se mogu tretirati u objektu. Tipovi su identični tipovima u obaveznim članovima.
- da li je dozvoljeno dodavanje članova - ovo se odnosi na mogućost da neko od sledećih u lancu učesnika može dodati član ili ne. Sledeći učesnik može ovo da promeni samo ako je dozvoljeno u nedozvoljeno a ako se unese zabrana dodavanja članova ona se više ne može promeniti. 

Za niz pogledati sledeće poglavlje.

Za skalarne tipove definicija je opisana u jednostavnim parametrima. 

##### Složeni parametri - Array i array of object

Za niz se definiše:
- tip elementa - tip elementa može biti skalarni (definicija opisana u jednostavnim parametrima) ili objekat (opisana u Složenim parametrima - Object)
- minimalni broj elemenata
- maksimalni broj elemenata (ukoliko je minimalni i maksimimalni broj elemenata isti tada je to u stvari fiksan broj elemenata)

Elementu niza se pristupa rednim brojm elementa a on počinje sa 0 (0 je prvi element u nizu).

##### Šifarnici

Šifarnici su definisani sa sledećim atributima: 
- tip šifarnika (enumeracija, sa šifrom, sa vrednošću)
- tip šifre (ceo broj, pozitivan ceo broj, string određene dužine)
- dužina naziva (od - do)
- minimalni i maksimalni broj elemenata u šifarniku (maksimalni može biti bez ograničenja)
- da li je moguće dodavati elemente - da li je dozvoljeno da sledeći učesnici u lancu mogu da dodaju elemente u šifarnik
- da li je moguće brisati elemente - da li je dozvoljeno da sledeći učesnici u lancu mogu brisati elemente 
- da li je moguće menjati vrednost - da li je dozvoljeno da sledeći učesnici u lancu mogu da menjaju polje vrednost (samo za šifarnike sa vrednošću)

Za svaku šifru se mogu unositi i sledeći podaci: 
- da li se vrednost (za šifarnike sa vrednošću) može menjati. Ovo polje ima značenje samo u slučaju da je za šifarnik dozovoljena izmena vrednosti. 
- ograničenja za šifre u zavisnosti od tipa isto kao za jednostavne parametre

##### Tabele odlučivanja

Tabele odlučivanja se identifikuju:
- imenom tabele
- ulaznim kolonama, a za svaku ulaznu kolonu se definiše
    - tip kolone - isto kao kod jednostavnih parametara
    - ograničenja pri unosu isto kao kod jednostavnih parametara
    - dodatno ograničenje ako je parametar iz šifarnika provera se da li postoji vrednost u šifarniku
    - posebno ograničenje ukoliko je vrednost kolone iz  API unosi se definicija API (koja može zavisiti od ulaznih parametara)
    - relacije parametara (veće, manje, veće jednako, manje jednako, jednako(može da se izostavi), između (tada je potrebno uneti dva parametra))
- izlaznim kolonama, a za svaku kolonu se definiše
    - tip kolone - istko kao kod jednosavnih parametara
    - ograničenja pri unosu istko kao kod jednostavnih parametara
    - dodatno ohraničenje ako je parametar iz šifarnika proverava se da li psotoji vrednost u šifarniku
    - posebno ograničenje ukoliko je vrednost koloe iz API unosi se definicija API (koja može zavisiti od ulaznih parametara)
    - ograničenje kao zavisnost ulaznih kolona
