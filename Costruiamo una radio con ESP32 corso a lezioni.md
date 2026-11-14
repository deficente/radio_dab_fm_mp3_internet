# Costruiamo una radio con ESP32: corso a lezioni

*FM, DAB+, internet radio, MP3 da SD/USB, orologio, preset e sveglia: dalla breadboard alla radio finale.*

## Lezione 0 – Ricerca, componenti e criticità

### Cosa esiste già

Nessun progetto trovato unisce tutte le funzioni richieste, ma ci sono ottimi pezzi da cui partire:

- **PE5PVB – SI4684 ESP32 DAB+ Receiver**: open source (KiCad e firmware PlatformIO/Arduino), ESP32 DevKit, chip Si4684, TFT 320×240, microSD, 2 encoder e 3 pulsanti. Richiede PCB e saldature.
- **Elektor Labs – Radio DAB+/FM (Yves Bourdon)**: ESP32 con scheda AVIT a Si4684, TFT a colori, 7 tasti, due amplificatori classe D da 3 W.
- **Instructables – DAB+ minimalista**: ESP32 D1 R32 con DABshield, lista componenti su GitHub.
- **Edzelf/ESP32-Radio**: internet radio con VS1053 e TFT, fino a 100 preset, encoder, MP3 da SD o chiavetta USB (CH376), aggiornamento OTA.
- **schreibfaul1/ESP32-audioI2S**: libreria per web radio e file audio via I2S (MP3, AAC, FLAC), compatibile con PCM5102A, MAX98357A, UDA1334A.
- **Esempi per singole funzioni**: internet radio ESP32-S3 con touch (Arielhh) e FM con TEA5767, encoder KY-040, PAM8403 e OLED (turtushig22-blip).

Il corso li combina: audio digitale come nelle web radio, un solo tuner Si4684 per FM e DAB+ come nei progetti DAB.

### Architettura scelta

| Blocco | Componente scelto | Perché |
| --- | --- | --- |
| Microcontrollore | ESP32-S3 DevKitC-1 con PSRAM (N8R8 o N16R8) | AAC, FLAC e HTTPS richiedono PSRAM; la libreria audio è pensata per chip multi-core |
| Display | TFT SPI 2,8 pollici ILI9341, 320×240 | economico e leggibile; librerie LovyanGFX o TFT\_eSPI |
| Tuner FM + DAB+ | Si4684 (DABshield, scheda AVIT o scheda PE5PVB) | un solo chip per FM con RDS e DAB+ |
| Audio digitale | DAC I2S PCM5102A | uscita stereo di linea, supportato da audioI2S |
| Amplificatore | modulo PAM8403 2×3 W a 5 V | economico, adatto alla cassa di test (upgrade: TPA3116D2) |
| Cassa di test | altoparlante 3 W, 4–8 Ω, in scatola | ascolto e collaudo |
| Orologio | DS3231 I2C con CR2032 | ora e sveglia anche senza Wi-Fi |
| MP3 | modulo microSD SPI; CH376S per chiavetta USB | SD prima, USB poi |
| Selettore sorgenti | modulo 2 relè 5 V (o relè DPDT con transistor) | commuta DAC e tuner |
| Comandi | 2 encoder KY-040 con pulsante, 3 pulsanti | volume, sintonia/menu, modo, preset, standby |
| Alimentazione | alimentatore 5 V 3 A certificato |  |
| Varie | breadboard, jumper, 1000 µF, 100 nF, resistenze, antenna telescopica |  |

### Criticità

1. **Il DAB non è un componente di uso comune.** Le schede con Si4684 sono di nicchia: verifica disponibilità, tempi e prezzi prima di acquistare, e controlla la copertura DAB+ della tua zona. Piano B: FM con Si4732 o TEF6686, DAB come lezione opzionale.
2. **Sorgenti digitali e analogiche insieme.** Il DAC esce in analogico, il tuner pure: serve un selettore (relè) e bisogna evitare i colpi (pop) durante la commutazione.
3. **USB MP3.** Non è plug and play su ESP32: si parte da microSD e si aggiunge la chiavetta con il modulo CH376S (spesso a 5 V: controlla i livelli logici a 3,3 V).
4. **Pin GPIO.** Sull'ESP32-S3 con PSRAM octal i GPIO 35–37 sono occupati e 26–32 servono a flash e PSRAM; 0, 3, 45, 46 sono di strapping; 19 e 20 sono la USB. I pin liberi bastano a malapena: si usa un bus SPI condiviso con CS separati e, se serve, un MCP23017 I2C.
5. **Breadboard e rumore.** Wi-Fi, TFT SPI e amplificatore in classe D generano disturbi che antenna FM/DAB e linee audio captano. Cavi corti, tuner lontano da display e amplificatore, antenna esterna.
6. **Alimentazione.** Picchi Wi-Fi più amplificatore fino a oltre 1 A: alimentatore 5 V 3 A, 1000 µF vicino all'amplificatore, alimentazione dell'amplificatore da morsetto e non dalle rotaie della breadboard.
7. **Accensione e sveglia.** Per suonare, la radio non può essere spenta del tutto: il pulsante di accensione diventa standby (audio e display spenti, orologio e sveglie attivi).
8. **Software e versioni.** Il core ESP32 per Arduino 3.x ha cambiato l'API I2S: fissa le versioni di core e librerie. Alcuni stream HTTPS funzionano meglio in HTTP. Il firmware del Si4684 viene caricato all'avvio: segui il metodo della scheda scelta.
9. **Volume uniforme.** Il volume software vale solo per la sorgente digitale; per FM e DAB si usa il volume del chip. Calibra i livelli per evitare salti al cambio sorgente.
10. **Sicurezza.** Solo alimentatore 5 V certificato; mai lavorare sulla tensione di rete.

### Mappa pin di partenza (da verificare sulla tua scheda)

| Funzione | GPIO |
| --- | --- |
| Bus SPI condiviso (SCK, MOSI, MISO) | 12, 11, 13 |
| CS: TFT, SD, tuner, CH376 | 10, 21, 47, 48 |
| TFT DC; RST del TFT collegato a EN | 9 |
| Tuner RST | 14 |
| I2S BCK, WS, DOUT | 4, 5, 6 |
| Encoder volume (A, B, SW) | 1, 2, 42 |
| Encoder sintonia/menu (A, B, SW) | 7, 15, 16 |
| Pulsanti Standby, Modo, Preset | 17, 18, 8 |
| I2C RTC (SDA, SCL) | 38, 39 |
| Relè selettore; alimentazione/mute ampli | 40, 41 |

Su alcune versioni della scheda il LED RGB di bordo usa il GPIO 38 o 48: è ignorabile.

### Piano delle lezioni

1. Ambiente di sviluppo e ESP32-S3
2. Display TFT
3. Manopole e pulsanti
4. Audio: DAC, amplificatore e cassa di test
5. Internet radio
6. Preset e memoria
7. Orologio
8. MP3 da SD e USB
9. Tuner Si4684: FM
10. DAB+
11. Selettore sorgenti
12. Standby e sveglia
13. Integrazione del firmware
14. Dalla breadboard alla radio finale

## Lezione 1 – Ambiente di sviluppo e ESP32-S3

**Materiale**

- ESP32-S3 DevKitC-1 con PSRAM, cavo USB dati, breadboard, LED e resistenza 330 Ω
- PC con PlatformIO (consigliato) o Arduino IDE

**Istruzioni**

1. Installa PlatformIO e fissa la versione del core ESP32.
2. Scegli la scheda ESP32-S3 DevKitC-1, abilita PSRAM OPI e USB CDC on boot.
3. Monta la scheda a cavallo della breadboard, lasciando libere le colonne di entrambi i lati.
4. Carica un blink con LED su un GPIO libero.
5. Stampa sul monitor seriale memoria flash e PSRAM.
6. Collega il Wi-Fi e stampa l'indirizzo IP.

**Test di funzionamento**

- Il LED lampeggia.
- La PSRAM risulta maggiore di zero (alcuni MB).
- Ottieni un IP e la scheda risponde al ping.

## Lezione 2 – Display TFT

**Materiale**

- TFT ILI9341 2,8 pollici SPI, jumper corti, libreria LovyanGFX o TFT\_eSPI

**Istruzioni**

1. Collega VCC 3,3 V, GND, SCK 12, MOSI 11, CS 10, DC 9; RST ad EN; retroilluminazione secondo il modulo.
2. Imposta i pin nella libreria e parti con SPI a 20 MHz, poi sali per gradi.
3. Disegna una schermata di prova: colori, rettangoli, testo grande.
4. Crea il layout radio: riga sorgente, titolo, orario, barra segnale, volume.
5. Aggiorna solo le aree che cambiano (o usa sprite in PSRAM).

**Test di funzionamento**

- Rosso, verde e blu compaiono nell'ordine giusto.
- Nessun artefatto o scorrimento dell'immagine.
- Aggiornare l'orario non fa sfarfallare lo schermo.

## Lezione 3 – Manopole e pulsanti

**Materiale**

- 2 encoder KY-040, 3 pulsanti, eventuali condensatori 10 nF su A e B

**Istruzioni**

1. Collega encoder volume (1, 2, 42) ed encoder sintonia (7, 15, 16); pulsanti a GND su 17, 18, 8 con INPUT\_PULLUP.
2. Leggi gli encoder con una libreria che usa il contatore hardware (ad esempio ESP32Encoder).
3. Gestisci gli eventi: rotazione, click, doppio click, pressione lunga.
4. Mostra gli eventi sul display.

**Test di funzionamento**

- Ogni scatto vale esattamente più o meno uno, senza rimbalzi.
- La pressione lunga (circa un secondo) viene riconosciuta.
- Muovendo i cavi non nascono eventi fantasma.

## Lezione 4 – Audio: DAC, amplificatore e cassa di test

**Materiale**

- Modulo PCM5102A, modulo PAM8403, altoparlante 3 W, condensatore 1000 µF 10 V, alimentatore 5 V 3 A

**Istruzioni**

1. Collega il DAC: BCK 4, LRCK 5, DIN 6; imposta i ponticelli del modulo (di solito SCK a massa e mute disattivato).
2. Porta l'uscita L/R del DAC all'ingresso dell'amplificatore.
3. Alimenta il PAM8403 direttamente dall'alimentatore, con il 1000 µF vicino ai suoi morsetti.
4. Collega l'altoparlante all'uscita sinistra: mai a massa, le uscite sono a ponte.
5. Genera un tono sinusoidale a 440 Hz via I2S, partendo da volume basso.

**Test di funzionamento**

- Tono pulito, senza fruscio forte.
- Il volume software cambia l'ascolto in modo progressivo.
- Al riavvio non senti botti forti.
- Misura l'assorbimento a 5 V con il multimetro.

## Lezione 5 – Internet radio

**Materiale**

- Circuito della lezione 4, Wi-Fi 2,4 GHz, qualche URL di stream (MP3 o AAC a 128 kbps)

**Istruzioni**

1. Installa la libreria ESP32-audioI2S e imposta i pin I2S.
2. Collega il Wi-Fi e avvia lo stream con connecttohost.
3. Gestisci le callback del nome stazione e dei metadati ICY e mostrali sul display.
4. Controlla il volume con l'encoder 1.
5. Prevedi riconnessione automatica e prova la versione HTTP se uno stream HTTPS non parte.
6. Gli URL si trovano su directory pubbliche di stazioni (ad esempio radio-browser.info).

**Test di funzionamento**

- Ascolto continuo per 30 minuti senza interruzioni.
- Titolo del brano visibile.
- Spegnendo e riaccendendo il Wi-Fi lo stream riparte da solo.

## Lezione 6 – Preset e memoria

**Materiale**

- Nessun componente nuovo

**Istruzioni**

1. Salva le stazioni in un file stations.json su LittleFS (nome, URL, tipo).
2. Salva in NVS (Preferences) ultimo preset, volume e sorgente.
3. Dieci preset: l'encoder 2 sceglie, il pulsante Preset richiama, la pressione lunga salva la stazione corrente nello slot.
4. Aggiungi una pagina web minima per modificare l'elenco e un portale di configurazione per le credenziali Wi-Fi.

**Test di funzionamento**

- Dopo il riavvio tornano ultima stazione e volume.
- I preset salvati restano.
- Puoi modificare l'elenco da browser.

## Lezione 7 – Orologio

**Materiale**

- Modulo DS3231 con CR2032

**Istruzioni**

1. Collega SDA 38, SCL 39 e alimentazione 3,3 V.
2. All'avvio sincronizza l'ora via NTP con il fuso CET-1CEST,M3.5.0,M10.5.0/3 (Italia, ora legale automatica).
3. Scrivi l'ora in UTC nel DS3231 e convertila con il fuso.
4. Senza Wi-Fi leggi l'ora dal DS3231.
5. Mostra l'orario in grande con la data.

**Test di funzionamento**

- Dopo un riavvio senza Wi-Fi l'ora è corretta.
- Cambiando la data a mano il passaggio all'ora legale avviene da solo.
- Dopo una notte la deriva è trascurabile.

## Lezione 8 – MP3 da SD e USB

**Materiale**

- Modulo microSD SPI a 3,3 V con scheda FAT32 e alcuni MP3
- Più avanti: modulo CH376S e chiavetta USB FAT32

**Istruzioni**

1. Collega la SD al bus SPI condiviso con CS 21.
2. Scansiona la cartella MP3 e costruisci l'elenco.
3. Riproduci con connecttoFS; a fine file passa alla traccia successiva.
4. Comandi: sintonia per traccia successiva/precedente, click per pausa.
5. Mostra titolo e tag ID3.
6. Aggiungi il CH376S sullo stesso bus (CS 48), rispettando i livelli a 3,3 V, e riusa lo stesso browser dei file.

**Test di funzionamento**

- Un album intero si ascolta senza interruzioni.
- Salto traccia e pausa funzionano.
- Con la chiavetta l'interfaccia è identica.
- Se compaiono glitch con TFT e SD sullo stesso bus, abbassa il clock SPI.

## Lezione 9 – Tuner Si4684: FM

**Materiale**

- Scheda Si4684 (DABshield, AVIT o PE5PVB), antenna telescopica, cavi corti

**Istruzioni**

1. Collega alimentazione e bus SPI (SCK 12, MOSI 11, MISO 13, CS 47, RST 14) secondo lo schema della scheda.
2. Carica il firmware FM con il metodo previsto dalla scheda.
3. Scansiona la banda FM (87,5–108 MHz) e mostra frequenza e livello di segnale.
4. Attiva l'RDS: nome stazione e radiotext.
5. Porta provvisoriamente l'uscita audio analogica all'ingresso dell'amplificatore.
6. Sintonizza con l'encoder 2 a passi da 100 kHz.

**Test di funzionamento**

- Almeno cinque stazioni locali con segnale buono.
- RDS leggibile.
- Audio pulito con antenna estesa, confronto con una radiolina.

## Lezione 10 – DAB+

**Materiale**

- Stessa scheda, antenna per banda III (telescopica accorciata o dedicata)

**Istruzioni**

1. Carica il firmware DAB.
2. Scansiona i canali della banda III (da 5A a 13F).
3. Elenca multiplex e servizi in ordine alfabetico.
4. Seleziona un servizio e mostra nome, testo dinamico (DLS) e qualità del segnale.
5. Salva l'elenco su LittleFS per non rifare la scansione a ogni avvio; scansione completa su richiesta.

**Test di funzionamento**

- Trovi almeno un multiplex locale.
- I servizi si ascoltano senza scatti.
- Dopo il riavvio l'elenco salvato viene usato senza nuova scansione.

## Lezione 11 – Selettore sorgenti

**Materiale**

- Modulo 2 relè 5 V (oppure 2 relè con transistor BC337 e diodo 1N4148), cavi audio corti

**Istruzioni**

1. Ingresso A = DAC (internet e MP3), ingresso B = tuner (FM e DAB+), uscita verso l'amplificatore.
2. Pilota il relè dal GPIO 40 tramite transistor e diodo.
3. Prima di commutare porta a zero il volume di entrambe le sorgenti, poi commuta, poi riporta il volume.
4. Implementa la macchina a stati delle sorgenti: INTERNET, SD/USB, FM, DAB; il pulsante Modo le scorre.
5. Calibra i livelli perché non ci siano salti di volume.

**Test di funzionamento**

- Ogni sorgente è udibile.
- Nessun colpo nel cambio sorgente.
- Livelli simili tra le quattro sorgenti.

## Lezione 12 – Standby e sveglia

**Materiale**

- MOSFET o interruttore per l'alimentazione dell'amplificatore (GPIO 41)

**Istruzioni**

1. Il pulsante di accensione (GPIO 17) alterna standby e funzionamento: in standby audio fermo, amplificatore spento, display ridotto all'orologio.
2. Sveglie: fino a tre, con giorni, ora, sorgente, preset, volume crescente in 60 secondi e snooze di 9 minuti.
3. Fallback: se lo stream non parte entro 15 secondi, passa a FM o a un tono.
4. Salva le sveglie in NVS e aggiungi il menu impostazioni.

**Test di funzionamento**

- La sveglia scatta dallo standby all'ora impostata.
- Volume crescente e snooze funzionano.
- Con il Wi-Fi spento parte il fallback.
- Le sveglie sopravvivono a un riavvio.

## Lezione 13 – Integrazione del firmware

**Materiale**

- Nessun componente nuovo

**Istruzioni**

1. Organizza il codice in moduli: interfaccia, ingressi, sorgenti audio, configurazione.
2. Usa task FreeRTOS separati per audio e interfaccia.
3. Macchina a stati generale: STANDBY, PLAY(sorgente), MENU, SVEGLIA.
4. Aggiungi aggiornamento OTA, watchdog e log seriale.
5. Gestisci gli errori: Wi-Fi perso, SD assente, tuner che non risponde.

**Test di funzionamento**

- 24 ore di funzionamento senza riavvii e con memoria stabile.
- Scollegare Wi-Fi o SD non blocca l'interfaccia.
- Ogni funzione è raggiungibile dal menu.

## Lezione 14 – Dalla breadboard alla radio finale

**Materiale**

- Scheda millefori o PCB, zoccoli femmina per i moduli, morsettiere, cavo schermato
- Scatola (legno o stampa 3D), pannello frontale, manopole, altoparlanti definitivi
- Presa di alimentazione 5 V con PTC, antenna telescopica con presa

**Istruzioni**

1. Congela schema e mappa pin.
2. Monta tutto su millefori con zoccoli, così ogni modulo si può sostituire.
3. Massa a stella; tuner e antenna lontani da display e amplificatore; cavi audio corti.
4. Fora il frontale: display, due manopole, tre pulsanti.
5. Cabla l'alimentazione e fissa i moduli.
6. Esegui il collaudo finale e documenta lo schema.

**Test di funzionamento finale**

- Funzionano le quattro sorgenti, i preset, l'orario, la sveglia e lo standby.
- Il rumore a volume zero è inudibile.
- Moduli solo tiepidi dopo un'ora di ascolto.
- 24 ore senza riavvii.

Upgrade possibili: TPA3116D2 con casse più grandi, Bluetooth, batteria.

## Fonti consultate

- cnx-software.com – articolo sul ricevitore DAB+ PE5PVB con ESP32 e Si4684 (aprile 2026)
- elektormagazine.com/labs/radio-dab – Radio DAB+/FM di Yves Bourdon
- instructables.com – A Minimalistic DAB Digital Radio
- github.com/Edzelf/ESP32-radio
- github.com/schreibfaul1/ESP32-audioI2S
- github.com/Arielhh/ESP32-Radio-Internet-With-TouchScreen-Display
- github.com/turtushig22-blip/tea5767-radio-tuner
