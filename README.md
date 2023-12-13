# Mini Projekat - Prvi domaći zadatak iz MIPS-a

## Autor

- [@Aleksandar Đokić 605-2021](https://www.github.com/caojasamalex)

## Šema
![Sema](https://i.ibb.co/XDZFzsx/Screenshot-2023-12-13-134608.png)

## Hardver

**MCU - STM32F103C6T6A**

**2x Step Motor - 86HBP80AL4**
**2x Microstep Driver - DM556**
![Step Motor i Drajver](https://i.ibb.co/S7VSWGj/image.png)

**Digitalni temperaturni senzor - TMP102**
![Temperaturni senzor](https://cdn.sparkfun.com//assets/parts/1/0/5/9/3/13314-02a.jpg)

**Optokapler - PC817**\
![Optokapler](https://robu.in/wp-content/uploads/2018/12/PC817-DIP-4-Transistor-Output-Optocoupler-Pack-of-5-ICs-6.jpg)

**Induktivni NPN Proximity Senzor - LN30Y-Z15NK**
![Proximity senzor](https://www.ep-solutions.rs/uploads/prodimgs/b/xurui-induktivni-senzor-m30-detekcija-15mm-npn-no-10-30-vdc-ln30y-z15nk.jpg)

**Napajanja - 3.3 VDC, 5 VDC, 24 VDC**

**Tranzistori - 8x NPN**

## Hardverska podešavanja Microstep-ova
Za motor koji okreće držače epruveta podešen je microstep 4, što znači da umesto 200 stepova po revoluciji (Jer je motor 1.8 stepeni po stepu) biće potrebno 800 stepova po revoluciji. Ovaj motor će raditi u više brzina. Prva brzina se koristi kada je potrebno zagrejati epruvete, tada su nam potrebni visoki obrtaji. Druga brzina se koristi kada se nakon zagrevanja vraćamo u početni položaj. Treća brzina se koristi kada od nultog položaja idemo ka željenoj epruveti.

Microstep 4 podešavamo hardverski uključivanjem/isključivanjem prekidača SW5, SW6, SW7 i SW8 na samom Microstep drajveru. (U ovom slučaju -> **SW5-ON, SW6-OFF, SW7-ON, SW8-ON**)

Za motor koji treba da izbaci epruvetu iz držača nije podešavan microstep već se koristi default 200 stepova po revoluciji. Microstep drajver ovog motora podešavamo tako da su svi već pomenuti prekidači ukljuceni.

## Način rada

Motor 1 će prvo okretati držače epruveta na visokom broju obrtaja kako bi se postigla željena temperatura (Broj slova u imenu i prezimenu * 5 = 75 u mom slučaju).

Digitalni temperaturni senzor TMP102 će očitavati temperaturu i kada temperatura bude u opsegu od 75-1 do 75+1 motor prestaje sa okretanjem.

Nakon toga, krug sa držačima se vraća u nulti položaj uz pomoć induktivnog proximity senzora koji će signalizirati kada je nulti položaj postignut. (Držač nulte epruvete će imati metalni deo koji će biti na < 15mm udaljenosti od induktivnog senzora. Kada se on pozicionira ispred senzora, senzor će preko optokaplera dovesti 3.3 V na ulaz PB1 mikrokontrolera)

Kada se vrati u početni položaj, potrebno je da korisnik unese željenu supstancu (od 0 do 4) i onda firmware obradjuje dalje korake.

Supstance su podeljene na sledeći način: redni_broj_slova_u_azbuci % 5 pa u mom slučaju imamo sledeći niz supstanci: **[1, 3, 2, 2, 1, 1, 1, 0, 1, 0, 1, 3, 2, 0, 3]**

Unos željene supstance se mikrokontroleru šalje preko UART-a, a sama dalja obrada i način rada je moguće videti u samom kodu.

Nakon unosa, prolazi se kroz nizove supstanci i niz dostupnosti epruvete (U nultom slucaju sve epruvete su dostupne pa je niz sastavaljen od 15 jedinica; Kada se epruveta izbaci jedinica se setuje na nulu)

Kada se nadje dostupna epruveta, ako je njen indeks u nizu veći od 7, to znači da će se motor okretati u suprotnom smeru jer je to najbliža putanja.

Kretanje se vrši tako što se drajveru šalje (53 * broj_epruveta_koji_treba_da_se_prodje) stepova kako bi stigao do odgovarajuće.

Kada stigne do odgovarajuće, motor 2, će izvršiti rotaciju i poluga koja je montirana na njega će izbaciti epruvetu iz držača i ubaciti je u kutiju.