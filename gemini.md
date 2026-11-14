Aspetto,Pregi,Difetti
Costo,Molto economico (circa 20€ - 35€ totali per tutti i componenti elettronici).,Richiede l'acquisto di più moduli separati e cavetteria.
Qualità Audio,"Ottima sulle sorgenti digitali grazie ai DAC I2S dedicati (evitando il DAC interno dell'ESP32, molto rumoroso).",La Radio FM soffre facilmente di interferenze elettromagnetiche generate dal Wi-Fi dell'ESP32 se i cavi non sono ben schermati.
Spotify,Funziona come uno speaker smart commerciale (controllabile da smartphone/PC senza usare il Bluetooth).,"Richiede obbligatoriamente un account Spotify Premium. Inoltre, le API di Spotify cambiano spesso, richiedendo aggiornamenti al firmware."
Complessità Software,Esistono già ottimi progetti pronti su GitHub (es. StreamCore32 o Euphonium).,Unire un firmware complesso basato su ESP-IDF (necessario per Spotify) con semplici librerie Arduino (spesso usate per la radio FM) richiede buone competenze di programmazione C++.
Consumi,Basso consumo energetico; alimentabile facilmente via USB-C o batteria LiPo.,"Il Wi-Fi attivo per Spotify assorbe parecchi picchi di corrente (fino a 300-400 mA), richiedendo un'alimentazione stabile."

Aggiungi il dab. Accensione istantanea del sistema. Eventuale sveglia.

Progetto Radio ESP32 Spotify FM
https://share.gemini.google/OUywT9nLrniZ
Creato con Auto 14 novembre 2026 alle ore 15:31
Data di pubblicazione: 14 novembre 2026 alle ore 20:08
Progetto radio fm esp32 mp3 spotify diy. Progetto con pregi e difetti

Realizzare un sistema audio "all-in-one" con ESP32 che integri Radio FM, lettore MP3 e Spotify è fattibile, ma richiede una scelta attenta dell'hardware perché lo streaming di Spotify è pesante per un microcontrollore.   
Makerfabs

1. Architettura Hardware Consigliata
Per gestire tutto senza blocchi o problemi di memoria, la configurazione ideale prevede:

Microcontrollore: ESP32-S3 (con almeno 8 MB di PSRAM) oppure ESP32-WROVER. La memoria PSRAM aggiuntiva è indispensabile per il buffering audio di Spotify.

Audio Digitale (MP3 e Spotify): Un DAC I2S esterno come il PCM5102A (per uscita cuffie/amplificatore) o il MAX98357A (che integra già un amplificatore mono da 3W per un piccolo altoparlante).   
YouTube

Radio FM: Un modulo sintonizzatore controllabile via bus I2C, come l'RDA5807M (consigliato per qualità e supporto RDS) o il TEA5767.

Storage MP3: Uno slot per scheda MicroSD collegato via SPI (spesso già integrato nelle schede di sviluppo o nei display TFT).   
YouTube

Interfaccia: Un display OLED I2C (es. SSD1306 da 0.96" o 1.3") o un TFT SPI più un encoder rotativo per cambiare stazione, brano e volume.   
YouTube

2. Come funzionano le 3 sorgenti audio
Lettore MP3: Si utilizza la libreria ESP32-audioI2S. Legge i file dalla MicroSD e invia il flusso decodificato al DAC I2S.   
Makerfabs

Spotify: Non esiste un'app nativa autonoma su ESP32. Si usa il protocollo Spotify Connect tramite librerie open-source come cspot o firmware completi come Squeezelite-ESP32 / StreamCore32. Il dispositivo appare tra gli altoparlanti disponibili sull'app Spotify del telefono, ma lo streaming avviene direttamente dal router all'ESP32.   
GitHub

Radio FM: Il modulo RDA5807M riceve il segnale analogico e lo invia direttamente all'amplificatore, mentre l'ESP32 si limita a inviare i comandi di cambio frequenza via I2C. (Nota: servirà un piccolo mixer resistivo o un selettore analogico per unire l'audio del modulo FM con quello del DAC I2S).

3. Pregi e Difetti del Progetto
Aspetto	Pregi	Difetti
Costo	Molto economico (circa 20€ - 35€ totali per tutti i componenti elettronici).	Richiede l'acquisto di più moduli separati e cavetteria.
Qualità Audio	Ottima sulle sorgenti digitali grazie ai DAC I2S dedicati (evitando il DAC interno dell'ESP32, molto rumoroso).	La Radio FM soffre facilmente di interferenze elettromagnetiche generate dal Wi-Fi dell'ESP32 se i cavi non sono ben schermati.
Spotify	Funziona come uno speaker smart commerciale (controllabile da smartphone/PC senza usare il Bluetooth).	Richiede obbligatoriamente un account Spotify Premium. Inoltre, le API di Spotify cambiano spesso, richiedendo aggiornamenti al firmware.
Complessità Software	Esistono già ottimi progetti pronti su GitHub (es. StreamCore32 o Euphonium).	Unire un firmware complesso basato su ESP-IDF (necessario per Spotify) con semplici librerie Arduino (spesso usate per la radio FM) richiede buone competenze di programmazione C++.
Consumi	Basso consumo energetico; alimentabile facilmente via USB-C o batteria LiPo.	Il Wi-Fi attivo per Spotify assorbe parecchi picchi di corrente (fino a 300-400 mA), richiedendo un'alimentazione stabile.
💡 Consiglio pratico
Se vuoi semplificarti la vita ed evitare il circuito di commutazione audio tra FM e DAC digitale, valuta di sostituire la Radio FM analogica con le Web Radio (streaming IP). Con la stessa libreria che legge gli MP3 puoi ascoltare migliaia di stazioni radio italiane e internazionali via Wi-Fi con qualità digitale e zero disturbi d'antenna.   
YouTube

Vuoi che ti prepari uno schema dei collegamenti (pinout) tra i componenti o preferisci esplorare i progetti firmware già pronti da caricare sull'ESP32?

Aggiungi il dab. Accensione istantanea del sistema. Eventuale sveglia.

Aggiungere DAB+, accensione istantanea e sveglia trasforma il progetto da un semplice player a una vera radiosveglia hi-tech, ma cambia radicalmente l'architettura hardware ed energetica.

1. Integrazione del DAB+ (e Radio FM insieme)
Un microcontrollore ESP32 non ha la potenza di calcolo per decodificare il segnale radio digitale DAB+ via software. Serve un chip tuner dedicato che invia l'audio già decodificato (in analogico o via bus digitale I2S).

Modulo consigliato: Si4684 o Si4688 (di Silicon Labs), reperibili su breakout board su AliExpress/eBay (cerca "DAB+ FM module Si4684" o moduli basati su chip Keystone T1 / T4B).

Il grande vantaggio: Questi moduli integrano sia DAB+ che FM sullo stesso chip e si controllano via I2C o UART. Elimini quindi il modulo FM separato (RDA5807M) e gestisci entrambe le bande da un'unica interfaccia.

Audio: Molti moduli DAB+ escono in analogico (Line Out) oppure in I2S. Se escono in analogico, ti servirà un piccolo multiplexer audio (es. chip CD4052 o TS5A22362) comandato dall'ESP32 per commutare l'amplificatore tra il modulo DAB/FM e il DAC di Spotify/MP3.

2. Accensione Istantanea del Sistema
Per avere un'accensione immediata (meno di 0,5 secondi per l'audio locale) senza attendere il boot completo e la riconnessione al Wi-Fi:

Niente spegnimento totale (Soft-Off / Light Sleep): Non tagliare l'alimentazione con un interruttore meccanico. L'ESP32 deve restare in Light Sleep (consumo ridottissimo, ~0.8 mA) con il display spento e l'amplificatore messo in MUTE tramite il pin dedicato (es. il pin SD sul MAX98357A).

Boot a priorità differenziata: Alla pressione del tasto Power, l'ESP32 si risveglia in pochi millisecondi, riattiva subito l'amplificatore e l'ultima sorgente locale (DAB+, FM o MP3 dalla MicroSD), mentre la connessione al Wi-Fi e il client Spotify vengono avviati in background su un core separato.

Attenzione al modulo DAB: I tuner DAB impiegano naturalmente 1–2 secondi per agganciare il multiplex digitale alla prima accensione, mentre l'FM e l'MP3 partono all'istante.

3. Funzione Sveglia Affidabile
Affidarsi solo all'orologio interno dell'ESP32 è rischioso: se va via la corrente, l'ora si azzera e la sveglia non suona.

Orologio RTC dedicato: Aggiungi un modulo DS3231 (collegato sullo stesso bus I2C del display e del tuner). Ha una batteria a bottone tampone (CR2032) che mantiene l'ora esatta per anni anche senza corrente.

Risveglio hardware: Il DS3231 ha un pin di interrupt (SQW/INT) programmabile: all'ora della sveglia, invia un segnale fisico all'ESP32 che lo risveglia istantaneamente dal sonno profondo.

Suoneria di sicurezza (Fallback): Se imposti la sveglia con Spotify o una stazione DAB, programma il firmware affinché, se dopo 5 secondi non c'è connessione Wi-Fi o segnale radio, riproduca automaticamente un file MP3 di emergenza salvato sulla MicroSD.

Nuovi Pregi e Difetti
Aspetto	Pregi	Difetti / Criticità
Integrazione Radio	Un solo modulo (es. Si4684) gestisce DAB+, FM e RDS, semplificando il cablaggio radio.	I moduli DAB+ costano circa 15€ - 25€ da soli (raddoppiando il budget) e richiedono una buona antenna telescopica dedicata per la banda VHF Band III.
Reattività	Grazie al Light Sleep e al DS3231, la sveglia è precisa al secondo e l'interfaccia risponde subito.	Il sistema deve restare sempre collegato alla presa di corrente (sconsigliato l'uso a batteria per via del consumo in standby dei moduli radio e del Wi-Fi).
Complessità Firmware	Puoi gestire display, RTC e radio su un core dell'ESP32 e Spotify/Wi-Fi sull'altro (usando FreeRTOS).	Non esiste un firmware "pronto all'uso" che unisca DAB+, Spotify Connect e sveglia: dovrai scrivere tu il codice di controllo combinando le varie librerie.
Come preferisci procedere con il progetto?

Genera la lista completa dei componenti (BOM) e i collegamenti

Mostra l'architettura software e le librerie da usare
Aggiungi un display touch per i comandi ed anche i knob regilatori di volume

Integrare un display touch e dei knob fisici (encoder rotativi) per il volume offre il meglio di entrambi i mondi: la flessibilità visiva di un'interfaccia grafica per scorrere le playlist di Spotify o le stazioni DAB/FM, e la comodità immediata di una manopola fisica per regolare il volume senza dover guardare lo schermo.

1. Come cambia la configurazione con Touch e Knob
Display Touch (TFT LCD):

Invece del semplice OLED monocromatico, si passa a un display a colori come il ILI9341 da 2.4" o 2.8" (risoluzione 320x240) con touch screen integrato (controller XPT2046) o a display capacitivi più moderni basati su chip ESP32-S3-BOX / display con driver ST7789.

A cosa serve: Mostrare le copertine degli album di Spotify (cover art), la lista delle stazioni radio DAB con i loghi o i nomi dei programmi, l'orologio della sveglia e i pulsanti di controllo riproduzione (Play/Pause, Salta, Cambia Sorgente).

Knob Regolatori di Volume (Encoder Rotativi):

Si utilizzano uno o due encoder rotativi con pulsante a pressione integrato (tipo KY-040).

A cosa servono: Il primo knob gestisce il volume principale (ruotando regola il volume, premendolo attiva il Mute). Un secondo knob opzionale può servire per la sintonizzazione rapida delle stazioni DAB/FM o per scorrere i brani senza usare il touch.

2. Impatto sul Microcontrollore e sui Pin dell'ESP32
Aggiungere un display TFT touch e i controlli fisici richiede un numero elevato di pin GPIO. Per questo motivo la scelta dell'ESP32-S3 è obbligatoria:

Bus SPI Dedicato: Il display TFT e il lettore MicroSD condividono lo stesso bus SPI ad alta velocità (richiede circa 5-6 pin dedicati).

Linee Touch e I2C: Il touch screen usa altri pin SPI o I2C, mentre il modulo DAB/FM, l'RTC DS3231 e l'OLED (se presente) sfruttano il bus I2C (SDA/SCL).

Interrupt per gli Encoder: Gli encoder rotativi richiedono pin che supportano gli interrupt hardware (attachInterrupt) per evitare di perdere impulsi quando si gira velocemente la manopola.

3. Pregi e Nuove Criticità della Configurazione Completa
Aspetto	Pregi	Criticità / Atttenzioni
Ergonomia e Usabilità	Interfaccia moderna e ricca di informazioni (grazie al touch a colori) unita alla risposta tattile immediata della manopola del volume.	Gestione dei pin: L'ESP32 ha molti pin, ma con WiFi, Bluetooth, SPI (Display + SD), I2C (Tuner DAB + RTC) e GPIO per gli encoder, i pin liberi si esauriscono in fretta.
Grafica e Risorse	Con librerie come LVGL (Light and Versatile Graphics Library) puoi creare UI bellissime con menu fluidi direttamente sull'ESP32-S3.	LVGL richiede molta RAM e cicli di CPU: va configurato sfruttando il doppio core dell'ESP32 (un core gestisce la grafica e l'interfaccia, l'altro l'audio e il Wi-Fi).
Costo e Cablaggio	I componenti restano economici nel complesso (un display TFT touch costa circa 8€-12€).	Aumentano notevolmente i collegamenti elettrici (cablaggio "a ragno" sconsigliato); è quasi d'obbligo l'uso di una PCB personalizzata (prototypation board / breadboard estesa) per evitare falsi contatti.
Vuoi che ti mostri lo schema completo di tutti i componenti (BOM) e l'assegnazione dei pin per l'ESP32-S3?

Sì
Genera la lista completa dei componenti con display touch ILI9341, encoder rotativi, modulo DAB/FM Si4684, DAC e RTC DS3231, specificando l'assegnazione dei pin per l'ESP32-S3.

Ecco la lista completa dei componenti (BOM - Bill of Materials) e la mappatura dei pin (Pinout) ottimizzata per evitare conflitti di memoria, bus e interrupt sull'ESP32-S3.

1. Lista Completa dei Componenti (BOM)
Categoria	Componente Specifico	Funzione e Note Tecniche	Costo Stimato
Cervello (MCU)	ESP32-S3-DevKitC-1 (N16R8)	Versione con 16MB Flash e 8MB PSRAM (indispensabile per buffering Spotify e grafica touch LVGL).	8€ - 12€
Display & Touch	TFT LCD 2.8" ILI9341 (320x240) con Touch XPT2046	Schermo a colori con slot MicroSD integrato sul retro (molto comodo per leggere gli MP3).	10€ - 15€
Radio DAB+ / FM	Modulo DAB+ / FM Si4684 (o Si4688)	Breakout board con uscita audio analogica (Line Out) e controllo I2C/SPI. Richiede antenna telescopica VHF/FM.	18€ - 25€
Audio Digitale	DAC I2S PCM5102A	Converte in alta qualità l'audio digitale di Spotify e MP3.	4€ - 7€
Selettore Audio	Multiplexer Analogico CD4052BE (o modulo relè bistabile)	Commuta l'audio in ingresso all'amplificatore tra il DAC PCM5102A (Spotify/MP3) e il Si4684 (DAB/FM).	1€ - 3€
Amplificatore	Modulo PAM8403 (3W+3W) o TPA3116D2	Amplificatore stereo in classe D per pilotare gli altoparlanti. Dotato di pin Mute/Shutdown per l'accensione istantanea.	2€ - 6€
Orologio / Sveglia	Modulo RTC DS3231 + Batteria CR2032	Mantiene l'ora esatta senza corrente e invia l'impulso fisico di sveglia all'ESP32.	2€ - 4€
Controlli Fisici	2x Encoder Rotativi KY-040	
Knob 1: Volume (rotazione) + Mute/Power (pressione).


Knob 2: Sintonizzazione/Scorrimento (rotazione) + Selezione (pressione).

3€ - 5€
Alimentazione	Alimentatore 5V 3A (USB-C o Jack) + Step-down 3.3V (AMS1117 / MP1584)	Garantisce corrente stabile per i picchi del Wi-Fi, lo schermo retroilluminato e l'amplificatore audio.	6€ - 10€
(Costo totale stimato dell'elettronica: circa 55€ - 85€, esclusi altoparlanti e case).

2. Mappatura dei Pin (Pinout ESP32-S3)
L'ESP32-S3 permette di rimappare quasi tutti i bus su qualsiasi pin libero. Questa configurazione evita i pin riservati alla memoria interna Flash/PSRAM (GPIO 26–37) e i pin di strapping critici (GPIO 0, 3, 45, 46).

A. Bus SPI Condiviso (Display ILI9341 + Touch XPT2046 + MicroSD)
Tutti e tre i dispositivi condividono le linee dati principali (MOSI, MISO, SCK) ma hanno un pin Chip Select (CS) separato.

SPI_MOSI (SDI): GPIO 11

SPI_MISO (SDO): GPIO 13

SPI_SCK (CLK): GPIO 12

TFT_CS (Chip Select Schermo): GPIO 10

TFT_DC (Data/Command): GPIO 9

TFT_RST (Reset): GPIO 14

TFT_BL (Retroilluminazione PWM): GPIO 21 (per regolare la luminosità di notte)

TOUCH_CS (Chip Select Touch): GPIO 15

TOUCH_IRQ (Interrupt Touch): GPIO 16

SD_CS (Chip Select MicroSD): GPIO 17

B. Bus I2C Condiviso (Tuner DAB/FM Si4684 + Orologio RTC DS3231)
I2C_SDA: GPIO 8 (con resistenza di pull-up da 4.7kΩ verso 3.3V)

I2C_SCL: GPIO 18 (con resistenza di pull-up da 4.7kΩ verso 3.3V)

DAB_RST (Reset Tuner Si4684): GPIO 4

DAB_INT (Interrupt Tuner Si4684): GPIO 5

RTC_SQW (Sveglia / Interrupt DS3231): GPIO 1 (pin RTC-capable per risvegliare l'ESP32 dal Light/Deep Sleep)

C. Bus Audio I2S (DAC PCM5102A per Spotify e MP3)
I2S_BCLK (Bit Clock): GPIO 38

I2S_LRCK (Word Select / LRC): GPIO 39

I2S_DOUT (Data Out): GPIO 40

D. Controlli Fisici (2x Encoder Rotativi)
ENC1_CLK (Volume A): GPIO 6

ENC1_DT (Volume B): GPIO 7

ENC1_SW (Tasto Mute/Accensione): GPIO 2 (pin RTC-capable per accensione istantanea dallo standby)

ENC2_CLK (Tuning/Menu A): GPIO 41

ENC2_DT (Tuning/Menu B): GPIO 42

ENC2_SW (Tasto Conferma): GPIO 47

E. Gestione Audio e Accensione Istantanea
AUDIO_MUX_SEL (Controllo CD4052): GPIO 48 (LOW = Audio da DAC Spotify/MP3, HIGH = Audio da Radio DAB/FM)

AMP_MUTE (Controllo PAM8403/TPA3116): GPIO 19 (Mette istantaneamente in muto l'amplificatore in standby ed evita il "bump" all'accensione)

3. Accorgimenti Pratici per il Cablaggio
Isolamento dei disturbi (Ground Loop): Alimenta il modulo DAB/FM Si4684 e il DAC PCM5102A attraverso un regolatore lineare a basso rumore (es. AMS1117-3.3 dedicato con condensatori da 100nF e 10µF), separato dall'alimentazione del display TFT. Eviterai ronzii di fondo e interferenze sulla ricezione radio.

Livelli Logici: L'ESP32-S3 lavora rigorosamente a 3.3V. Assicurati che i pin I2C e SPI del modulo Si4684 e del display ILI9341 lavorino a 3.3V (quasi tutti i moduli moderni lo fanno, ma verifica che il regolatore onboard del display sia bypassato o compatibile).

Antenna DAB/FM: Collega il cavo dell'antenna telescopica direttamente al connettore SMA/U.FL del modulo Si4684 tenendolo il più lontano possibile dai cavi SPI del display, che emettono disturbi ad alta frequenza.

Fai una immagine dei componenti e collegamenti

A technical block diagram schematic titled "SCHEMA DI COLLEGAMENTO ESP32-S3 RADIO DAB+, FM, SPOTIFY, MP3, SVEGLIA" on a clean white background, illustrating the complex DIY electronics project. At the center is a large rectangular ESP32-S3-DevKitC-1 (N16R8) development board, labeled "MCU ESP32-S3," with all specified GPIO pins clearly indicated with numbers (e.g., GPIO 11, GPIO 13, GPIO 12, GPIO 9, GPIO 10, GPIO 8, GPIO 18, GPIO 5, GPIO 4, GPIO 1, GPIO 38, GPIO 39, GPIO 40, GPIO 48, GPIO 19, GPIO 6, GPIO 7, GPIO 2, GPIO 41, GPIO 42, GPIO 47) and functions (SPI_MOSI, SPI_MISO, SPI_SCK, TFT_DC, TFT_CS, I2C_SDA, I2C_SCL, DAB_INT, DAB_RST, RTC_SQW, I2S_BCLK, I2S_LRCK, I2S_DOUT, AUDIO_MUX_SEL, AMP_MUTE, ENC1_CLK, ENC1_DT, ENC1_SW, ENC2_CLK, ENC2_DT, ENC2_SW). The diagram shows logical connections using colored wires.

To the left, the shared SPI bus branches:
1.  **TFT LCD 2.8" ILI9341 with Touch XPT2046 & MicroSD:** Labeled "DISPLAY TFT TOUCH & SD," connected to SPI_MOSI, MISO, SCK, TFT_CS, DC, RST, BL (GPIO 21), TOUCH_CS (GPIO 15), TOUCH_IRQ (GPIO 16), and SD_CS (GPIO 17).

To the right, connected to the I2C bus (GPIO 8, 18):
1.  **Modulo DAB+ / FM Si4684 (con Antenna Telescopica):** Labeled "TUNER DAB+/FM," connected to I2C_SDA, SCL, DAB_RST, DAB_INT, and an antenna symbol labeled "ANTENNA VHF/FM."
2.  **Modulo RTC DS3231:** Labeled "RTC DS3231" with its CR2032 battery, connected to I2C_SDA, SCL, and RTC_SQW (GPIO 1).

Bottom connections:
1.  **DAC I2S PCM5102A:** Labeled "DAC I2S" connected to I2S_BCLK, LRCK, DOUT.
2.  **Multiplexer Analogico CD4052BE:** Labeled "AUDIO MUX CD4052," connected to AUDIO_MUX_SEL (GPIO 48), receiving inputs from "DAC L/R" and "TUNER L/R," and sending "OUT L/R" to the amplifier.
3.  **Amplificatore PAM8403:** Labeled "AMPLIFICATORE PAM8403," connected to AMP_MUTE (GPIO 19) and outputting to "ALTOPARLANTE L" and "ALTOPARLANTE R."
4.  **2x Encoder Rotativi KY-040:** Labeled "ENCODER VOLUME" (connected to ENC1_CLK, DT, SW) and "ENCODER TUNING" (connected to ENC2_CLK, DT, SW).

Top section:
1.  **Alimentazione:** "ALIMENTATORE 5V 3A" feeding a block labeled "REGOLATORE AMS1117 3.3V," providing power (3.3V and GND lines) to the ESP32-S3, TFT, Tuner, RTC, DAC, Mux, and Amp.

The diagram uses various colors for signal lines (yellow for SPI, blue for I2C, green for Audio, purple for Control, orange for Interrupt, red for 3.3V power, black for GND). All components are drawn as labeled modules with pins, clear text labels for every connection, component name, and GPIO number. The background is white with a light grey grid, and the style is a clean, professional electronics schematic., AI generated, generata con l'AI
Si può montare tutto senza pcb come esercizio didattico?

Sì, assolutamente! Per un esercizio didattico e di prototipazione, puoi montare tutto su breadboard usando cavetti DuPont, senza dover progettare o stampare una PCB.

Tuttavia, data la presenza di segnali radio ad alta frequenza (DAB/FM) e bus veloci (SPI, I2S), ci sono alcune regole pratiche fondamentali per evitare che il prototipo non funzioni o faccia troppo rumore:

1. Cosa ti serve per il montaggio senza PCB
2 Breadboard standard da 830 punti (affiancate o collegate insieme): l'ESP32-S3 è largo e occupa quasi tutta la larghezza di una singola breadboard, lasciando pochi fori liberi ai lati.

Set di cavetti DuPont (Maschio-Maschio e Maschio-Femmina) corti (preferibilmente da 10 cm, evitando quelli lunghi da 20-30 cm che fungono da antenne per i disturbi).

Alcuni condensatori ceramici da 100nF (0.1µF) ed elettrolitici da 10µF - 100µF da inserire direttamente sulle linee di alimentazione della breadboard per stabilizzare la corrente.

2. Accorgimenti critici su Breadboard (per non impazzire)
Linee di alimentazione divise: Molte breadboard lunghe hanno le linee rosse e blu (+ e -) interrotte a metà. Ricordati di fare un ponticello al centro, altrimenti metà dei componenti non riceverà corrente.

Qualità dei cavetti DuPont: I cavi economici spesso hanno contatti ballerini. Se lo schermo TFT resta bianco o la MicroSD non viene letta, nel 90% dei casi è un cavetto SPI lento o difettoso.

Interferenze tra Display e Radio DAB/FM: Posiziona il modulo radio Si4684 a un'estremità della breadboard e il display TFT all'estremità opposta. I cavi del bus SPI del display emettono disturbi elettromagnetici che possono "coprire" il segnale radio DAB+.

Connessioni I2S corte: I tre fili che vanno dall'ESP32 al DAC PCM5102A (BCLK, LRCK, DOUT) trasportano segnali audio digitali ad alta frequenza: tienili il più corti possibile e lontani dai cavi dell'antenna.

Massa comune (GND) solida: Assicurati che tutti i moduli (ESP32, Display, DAC, Tuner, RTC, Amplificatore) abbiano un collegamento diretto e ben saldo alla linea di massa (GND) comune.

3. Consiglio per procedere a step (Metodo Didattico)
Non montare tutto insieme al primo colpo, altrimenti in caso di errore sarà difficile capire cosa non va. Procedi per moduli:

Step 1: Monta solo ESP32-S3 + Display Touch + Encoder e testa l'interfaccia grafica e i controlli.

Step 2: Aggiungi il DAC PCM5102A + Amplificatore e prova a riprodurre un MP3 dalla MicroSD o lo streaming Spotify.

Step 3: Integra l'RTC DS3231 e testa la sveglia e lo standby (Light Sleep).

Step 4: Aggiungi per ultimo il modulo DAB/FM Si4684 e il multiplexer audio CD4052, lavorando sul posizionamento dell'antenna per ridurre i disturbi.

Una volta che tutto funziona su breadboard, se vorrai renderlo definitivo e robusto, potrai trasferire il circuito su una scheda millefori saldando i componenti.

Crea la sequenza di montaggio come se fissero i passi di.un corso. Ogni step deve avere indicato, materiale, obiettivo e test di funzionamento.

Ecco la sequenza di montaggio strutturata come un corso pratico passo-passo. Questo approccio modulare (consigliato per la didattica) permette di verificare il corretto funzionamento di ogni singolo sottosistema prima di passare al successivo.

Modulo 1: Setup di Base e Interfaccia Grafica
Obiettivo: Inizializzare l'ESP32-S3, configurare il bus SPI, accendere il display TFT touch e verificare la risposta dei comandi tattili e fisici.

Materiale necessario:

Scheda ESP32-S3-DevKitC-1 (N16R8)

Display TFT LCD 2.8" ILI9341 (con Touch XPT2046 e MicroSD)

2x Encoder Rotativi KY-040

Breadboard e cavetti DuPont

Alimentatore 5V e cavo USB-C

Passi di montaggio:

Posiziona l'ESP32-S3 e il display sulla breadboard.

Collega i pin del bus SPI condiviso (GPIO 11 per MOSI, GPIO 13 per MISO, GPIO 12 per SCK) dall'ESP32 al display.

Collega i pin di controllo del display (TFT_CS su GPIO 10, TFT_DC su GPIO 9, TFT_RST su GPIO 14, TFT_BL su GPIO 21) e del touch (TOUCH_CS su GPIO 15, TOUCH_IRQ su GPIO 16).

Collega i due encoder rotativi ai pin dedicati (GPIO 6, 7, 2 per il primo; GPIO 41, 42, 47 per il secondo).

🧪 Test di funzionamento: Carica uno sketch di test con la libreria LVGL o Adafruit grafiche. Verifica che lo schermo si accenda, che toccando i pulsanti virtuali sullo schermo il sistema risponda e che ruotando le manopole (encoder) i valori a schermo cambino correttamente.

Modulo 2: Riproduzione Audio Locale (MP3 & SD Card) e DAC
Obiettivo: Integrare la lettura dei file MP3 dalla scheda MicroSD e convertire il flusso audio digitale in analogico di alta qualità tramite il DAC I2S.

Materiale necessario:

Il circuito del Modulo 1

DAC I2S PCM5102A

Scheda MicroSD (formattata in FAT32 con qualche file MP3 di test)

Amplificatore PAM8403 e un piccolo altoparlante

Passi di montaggio:

Inserisci la MicroSD nel lettore sul retro del display TFT e collega il pin SD_CS a GPIO 17.

Collega il DAC PCM5102A all'ESP32 usando il bus I2S (I2S_BCLK su GPIO 38, I2S_LRCK su GPIO 39, I2S_DOUT su GPIO 40).

Collega l'uscita analogica del DAC ai canali del modulo amplificatore PAM8403 e collega l'altoparlante.

Collega il pin AMP_MUTE dell'ESP32 al pin di controllo dell'amplificatore.

🧪 Test di funzionamento: Scrivi un codice che legga un file MP3 dalla MicroSD, lo invii via I2S al DAC e regoli il volume tramite il primo encoder. Verifica che l'audio esca pulito dall'altoparlante e che la manopola del volume funzioni senza fruscii anomali.

Modulo 3: Gestione del Tempo e Sveglia (RTC DS3231)
Obiettivo: Integrare l'orologio in tempo reale per mantenere l'ora esatta e impostare una funzione sveglia sicura anche in assenza di alimentazione di rete.

Materiale necessario:

Il circuito dei Moduli 1 e 2

Modulo RTC DS3231 (con batteria CR2032 inserita)

Passi di montaggio:

Collega il modulo RTC DS3231 al bus I2C dell'ESP32 (I2C_SDA su GPIO 8, I2C_SCL su GPIO 18), condividendo le linee con eventuali altri dispositivi I2C, ricordando le resistenze di pull-up.

Collega il pin di interrupt RTC_SQW del DS3231 al GPIO 1 dell'ESP32.

🧪 Test di funzionamento: Imposta l'ora corrente tramite un programma di prova. Scollega l'alimentazione principale per un minuto, poi ricollegala: verifica che l'orologio sul display mantenga l'ora corretta grazie alla batteria tampone.

Modulo 4: Ricezione Radio DAB+ / FM e Commutazione Audio
Obiettivo: Aggiungere la sintonizzazione delle stazioni radio digitali (DAB+) e analogiche (FM) e gestire il multiplexer audio per alternare la radio alle sorgenti digitali (MP3/Spotify).

Materiale necessario:

Il circuito dei Moduli 1, 2 e 3

Modulo DAB+/FM Si4684 con antenna telescopica

Multiplexer analogico CD4052BE

Passi di montaggio:

Collega il modulo Si4684 al bus I2C (SDA su GPIO 8, SCL su GPIO 18), al pin di reset (DAB_RST su GPIO 4) e di interrupt (DAB_INT su GPIO 5). Monta l'antenna telescopica sul modulo radio.

Inserisci il multiplexer CD4052BE: collega i suoi ingressi rispettivamente all'uscita analogica del DAC (Spotify/MP3) e all'uscita audio del modulo Si4684 (Radio).

Collega l'uscita del multiplexer all'amplificatore PAM8403.

Collega il pin di controllo del multiplexer (AUDIO_MUX_SEL) al GPIO 48 dell'ESP32 per decidere via software quale sorgente ascoltare.

🧪 Test di funzionamento: Seleziona la modalità "Radio FM/DAB" dall'interfaccia touch. Verifica che l'antenna agganci le stazioni, che il multiplexer commuti correttamente l'audio senza sovrapposizioni e che l'ascolto sia limpido.

Modulo 5: Integrazione Spotify Connect e Modalità Standby (Light Sleep)
Obiettivo: Configurare la connettività Wi-Fi, integrare il protocollo Spotify Connect per lo streaming cloud e implementare il risparmio energetico con accensione istantanea tramite l'encoder o la sveglia RTC.

Materiale necessario:

Il sistema completo assemblato nei moduli precedenti

Rete Wi-Fi domestica e un account Spotify Premium

Passi di montaggio:

Integra nel firmware la libreria per Spotify Connect (es. basata su cspot/Squeezelite) configurando le credenziali di rete.

Implementa la gestione del doppio core dell'ESP32-S3 (Core 0 per Wi-Fi/Audio di rete, Core 1 per UI grafica e controlli).

Programma la modalità Light Sleep quando si preme il tasto di spegnimento: spegni la retroilluminazione del display (GPIO 21 a LOW), metti in MUTE l'amplificatore (GPIO 19) e mantieni vigili i pin di interruzione dell'encoder di accensione (GPIO 2) e dell'RTC (GPIO 1).

🧪 Test di funzionamento finale:

Apri l'app Spotify sul telefono e verifica che il dispositivo "ESP32 Radio" compaia tra gli speaker disponibili; avvia la riproduzione e controlla la fluidità dell'audio.

Premi la manopola per mandare il sistema in standby (consumo ridotto). Premila di nuovo (o attendi l'orario della sveglia): verifica che il sistema si riattivi istantaneamente riprendendo l'ultima funzione attiva.

Si può usare un account spotify gratis

No, non è possibile utilizzare un account Spotify Free (gratuito) per questo tipo di progetto su ESP32.

Per lo streaming su microcontrollori tramite protocollo Spotify Connect (sfruttando librerie come cspot o simili), è obbligatorio disporre di un account Spotify Premium.

Perché è richiesto Spotify Premium?
Limitazioni delle API ufficiali: Spotify non permette il controllo e lo streaming diretto tramite API esterne su dispositivi hardware custom (come l'ESP32) per gli account gratuiti, che sono vincolati all'uso delle app ufficiali con pubblicità.

Protocollo Spotify Connect: Questo protocollo di streaming cloud richiede le credenziali e le funzioni di autenticazione avanzate (gestione dei token cifrati) che Spotify abilita esclusivamente per gli utenti abbonati (Premium).

Alternative gratuite se non hai Spotify Premium:
Se preferisci non sottoscrivere un abbonamento Premium, puoi comunque riempire la tua MicroSD di file MP3 o configurare il modulo radio per ascoltare le Web Radio in streaming Wi-Fi (gratuitamente), oltre ovviamente alla radio FM e DAB+ terrestre.

Usare la versione web di spotify?

Usare la versione web di Spotify (il player da browser web, come Google Chrome o Firefox) non è possibile direttamente sull'ESP32, per motivi tecnici legati alle risorse del microcontrollore. Ecco perché:

Mancanza di un Browser: L'ESP32 non esegue un sistema operativo completo (come Windows, macOS o Linux) e non possiede un browser web in grado di caricare pagine, interpretare JavaScript o eseguire lo script di riproduzione di Spotify Web Player.

Come funziona l'ESP32: I progetti fai-da-te per ESP32 non "navigano" su Spotify come faresti da PC. Usano invece librerie di terze parti (come cspot) che simulano un dispositivo Spotify Connect via codice, comunicando direttamente con i server di Spotify tramite API. Anche con questa modalità, come detto, l'autenticazione richiede comunque i token di un account Spotify Premium.

Esistono alternative se usi un account Free?
Se il tuo obiettivo è riprodurre musica da piattaforme di streaming o Web App senza avere Spotify Premium, sull'ESP32 puoi valutare:

Internet Web Radio / Streaming HTTP: Tramite la libreria audio puoi sintonizzarti gratuitamente su migliaia di stazioni radio in streaming IP (es. flussi Icecast o Shoutcast).

Bluetooth Audio Sink: Puoi fare in modo che l'ESP32 funzioni come una normale cassa Bluetooth. In questo modo puoi connettere il tuo smartphone all'ESP32 e riprodurre qualsiasi sorgente audio dal telefono (inclusa la versione web gratuita di Spotify aperta sul computer o sul telefono), inviando l'audio in streaming Bluetooth direttamente all'ESP32 e poi al DAC.

Le web radio che si ascoltano da browser sono utilizzabili?

Sì, assolutamente! Le web radio che si ascoltano normalmente da browser (tramite i siti ufficiali delle emittenti, come RaiPlay Sound, RTL 102.5, Deejay, ecc.) sono perfettamente utilizzabili sull'ESP32, ed è una delle soluzioni più semplici e completamente gratuite.

Ecco come funzionano e perché si adattano benissimo al tuo progetto:

Come funzionano su ESP32?
A differenza di Spotify (che richiede un protocollo chiuso e account Premium), la stragrande maggioranza delle web radio trasmette flussi audio aperti basati su protocolli standard HTTP / HTTPS, Icecast o Shoutcast (formati MP3 o AAC).

L'ESP32-S3, grazie alla libreria audio AudioI2S, è perfettamente in grado di:

Connettersi a Internet tramite Wi-Fi.

Scaricare il flusso audio in streaming direttamente dall'indirizzo URL della radio (lo stesso link che usa il browser).

Decodificarlo in tempo reale e inviarlo al DAC I2S per l'ascolto sull'altoparlante.

Vantaggi rispetto a Spotify:
Completamente gratuito: Non serve alcun abbonamento.

Nessun blocco API: Le stazioni radio forniscono direttamente i link ai flussi streaming pubblici.

Zero problemi di account: Puoi inserire la lista delle tue stazioni preferite direttamente nel codice dell'ESP32 (o farle scegliere tramite il display touch).

Come integrarle nel progetto:
Nel firmware dell'ESP32 puoi memorizzare una lista di URL di streaming (es. [http://icestreaming.rai.it/](http://icestreaming.rai.it/)... per le radio RAI). Con l'interfaccia touch (creata con LVGL) potrai visualizzare i loghi o i nomi delle stazioni e cambiarle con un semplice tap o usando la seconda manopola (encoder) che avevamo previsto per la sintonizzazione!

Il progetto prevede la memorizzazione delle stazioni fm?

Sì, è assolutamente previsto e fortemente consigliato!

Nel contesto del nostro progetto, la memorizzazione delle stazioni (i cosiddetti "preset" o preferiti) si divide in due parti: la parte software (facile da implementare) e la parte hardware (già gestita dai componenti che abbiamo scelto).

Ecco come funziona e come si integra:

1. Come funziona la memorizzazione delle stazioni
Stazioni FM e DAB+: Il modulo radio che abbiamo scelto (il Si4684) possiede funzioni interne di scansione delle frequenze. Nel firmware dell'ESP32-S3 puoi programmare una funzione di "Scansione automatica" (Auto-Tune) che cerca tutte le emittenti con segnale forte e le salva in un array o nella memoria Flash interna dell'ESP32 (usando la libreria Preferences).

Web Radio e Streaming: Anche per le web radio via Wi-Fi puoi creare una lista fissa o modificabile di URL preferiti.

Interfaccia Touch (LVGL): Sul display TFT 2.8" puoi creare una schermata con dei pulsanti numerati (es. Preset 1: RTL 102.5, Preset 2: Radio Deejay, ecc.) che richiamano istantaneamente la stazione salvata con un semplice tap.

2. Dove vengono salvate? (Memoria Flash dell'ESP32)
L'ESP32-S3 ha a disposizione una memoria Flash da 16MB (nella versione DevKit che abbiamo inserito nella lista componenti).

Una piccolissima parte di questa memoria viene utilizzata per salvare in modo permanente le preferenze dell'utente: le stazioni radio salvate, l'ultima sorgente ascoltata (DAB, FM, MP3 o Web Radio) prima dello spegnimento e persino il livello del volume.

In questo modo, quando spegni la radio (o va in Light Sleep), le stazioni memorizzate non si perdono neanche togliendo l'alimentazione principale.

Vuoi che aggiungiamo questa funzione di "Salvataggio Preset" tra le caratteristiche software da sviluppare nel corso?

Rimuovi la funzione di spotify

Certamente, rimuoviamo completamente la parte di Spotify (visto anche l'obbligo dell'account Premium e l'incompatibilità con la versione Free). Il sistema diventa così un vero e proprio ricevitore ibrido incentrato su Radio DAB+, FM, Web Radio via Wi-Fi, lettore MP3 da MicroSD e Sveglia.

Ecco come si modifica il progetto di conseguenza:

1. Cosa cambia nella Lista Componenti (BOM)
Addio a cspot / Spotify Connect: Non serve più l'architettura di streaming cloud complessa.

Semplificazione software: L'ESP32-S3 userà solo le librerie per il display touch (LVGL), la lettura file (SD/MP3), i flussi HTTP per le Web Radio e le librerie I2C/I2S per il sintonizzatore Si4684 e il DAC PCM5102A.

Il resto della componentistica (Display TFT, Encoder, RTC DS3231, DAC, Multiplexer CD4052, Amplificatore e alimentatore) resta esattamente identico.

2. Le Sorgenti Audio Finali del Sistema
Eliminando Spotify, le sorgenti gestibili tramite l'interfaccia touch e i knob diventano 4:

Radio DAB+ (Digitale): Audio limpido, ricezione digitale terrestre e salvataggio dei preset preferiti.

Radio FM (Analogica): Con supporto RDS e scansione automatica delle frequenze.

Web Radio (Wi-Fi): Streaming gratuito illimitato di migliaia di stazioni italiane e internazionali tramite i flussi HTTP/Icecast inseriti nel codice (o selezionabili da touch).

Lettore MP3: Riproduzione di musica locale salvata sulla scheda MicroSD inserita nel retro del display.

3. Vantaggi di questa configurazione "Spotify-Free"
Costo azzerato per gli abbonamenti: Non serve alcun account a pagamento.

Codice molto più leggero e stabile: Meno consumo di RAM e minor carico sul processore dell'ESP32-S3, con un avvio ancora più rapido e zero problemi di aggiornamento token API (che Spotify cambia spesso).

Gestione stazioni semplificata: Puoi memorizzare facilmente sia i preset FM/DAB che la lista delle tue Web Radio preferite direttamente nella memoria Flash dell'ESP32.

Vuoi che aggiorniamo la struttura del corso o il dettaglio dei pinout rimuovendo i riferimenti a Spotify?

Aggiorna il corso

Ecco il corso pratico aggiornato per la realizzazione della Radio DIY con ESP32-S3, rimuovendo completamente la parte di Spotify e mantenendo le 4 sorgenti audio (DAB+, FM, Web Radio e Lettore MP3 locale), la gestione dei preset, il display touch, gli encoder e la sveglia RTC.

Modulo 1: Setup di Base, Display Touch e Controlli Fisici
Obiettivo: Inizializzare l'ESP32-S3, configurare il bus SPI, accendere il display TFT touch (con slot SD integrato) e verificare la risposta dei comandi tattili e delle manopole.

Materiale necessario:

Scheda ESP32-S3-DevKitC-1 (N16R8)

Display TFT LCD 2.8" ILI9341 (con Touch XPT2046 e MicroSD)

2x Encoder Rotativi KY-040

Breadboard e cavetti DuPont

Alimentatore 5V e cavo USB-C

Passi di montaggio:

Posiziona l'ESP32-S3 e il display sulla breadboard.

Collega i pin del bus SPI condiviso (GPIO 11 per MOSI, GPIO 13 per MISO, GPIO 12 per SCK) dall'ESP32 al display.

Collega i pin di controllo del display (TFT_CS su GPIO 10, TFT_DC su GPIO 9, TFT_RST su GPIO 14, TFT_BL su GPIO 21) e del touch (TOUCH_CS su GPIO 15, TOUCH_IRQ su GPIO 16).

Collega i due encoder rotativi ai pin dedicati (GPIO 6, 7, 2 per il primo; GPIO 41, 42, 47 per il secondo).

🧪 Test di funzionamento: Carica uno sketch di test grafico (es. libreria LVGL). Verifica che lo schermo si accenda, che il touch risponda ai comandi e che ruotando le manopole i valori sullo schermo varino correttamente.

Modulo 2: Riproduzione Audio Locale (MP3 & SD Card) e DAC
Obiettivo: Integrare la lettura dei file MP3 dalla scheda MicroSD e convertire il flusso audio digitale in analogico tramite il DAC I2S e l'amplificatore.

Materiale necessario:

Il circuito del Modulo 1

DAC I2S PCM5102A

Scheda MicroSD (formattata in FAT32 con file MP3 di test)

Amplificatore PAM8403 e altoparlante

Passi di montaggio:

Inserisci la MicroSD nel lettore sul retro del display TFT e collega il pin SD_CS a GPIO 17.

Collega il DAC PCM5102A all'ESP32 tramite il bus I2S (I2S_BCLK su GPIO 38, I2S_LRCK su GPIO 39, I2S_DOUT su GPIO 40).

Collega l'uscita analogica del DAC agli ingressi dell'amplificatore PAM8403 e collega l'altoparlante.

Collega il pin AMP_MUTE dell'ESP32 al controllo di mute dell'amplificatore (GPIO 19).

🧪 Test di funzionamento: Avvia la riproduzione di un brano MP3 dalla MicroSD. Verifica che l'audio esca pulito dall'altoparlante e che la manopola del volume regoli correttamente l'ascolto.

Modulo 3: Gestione del Tempo, Sveglia e Salvataggio Preset
Obiettivo: Integrare l'orologio in tempo reale (RTC) per la sveglia autonoma e configurare la memoria Flash per salvare i preset delle stazioni preferite.

Materiale necessario:

Il circuito dei Moduli 1 e 2

Modulo RTC DS3231 (con batteria CR2032)

Passi di montaggio:

Collega il modulo RTC DS3231 al bus I2C dell'ESP32 (I2C_SDA su GPIO 8, I2C_SCL su GPIO 18) con le relative resistenze di pull-up.

Collega il pin di interrupt RTC_SQW del DS3231 al GPIO 1 dell'ESP32 per consentire il risveglio hardware in orario di sveglia.

Nel firmware, implementa la libreria Preferences per memorizzare nella Flash dell'ESP32 le frequenze FM/DAB e gli URL delle Web Radio salvati dall'utente.

🧪 Test di funzionamento: Imposta l'ora, scollega l'alimentazione per verificare la batteria tampone dell'RTC, e testa la scrittura/lettura dei preset radiofonici nella memoria interna.

Modulo 4: Ricezione Radio DAB+ / FM e Commutazione Audio
Obiettivo: Integrare il modulo sintonizzatore Si4684 per radio digitale e analogica e il multiplexer audio per gestire le sorgenti.

Materiale necessario:

Il circuito dei Moduli 1, 2 e 3

Modulo DAB+/FM Si4684 con antenna telescopica

Multiplexer analogico CD4052BE

Passi di montaggio:

Collega il modulo Si4684 al bus I2C (SDA su GPIO 8, SCL su GPIO 18), al pin di reset (DAB_RST su GPIO 4) e di interrupt (DAB_INT su GPIO 5). Monta l'antenna telescopica.

Collega il multiplexer CD4052BE interponendolo tra l'uscita del DAC (MP3/Web Radio) e l'uscita del Si4684 (Radio), mandando l'uscita comune all'amplificatore.

Collega il pin di controllo del multiplexer (AUDIO_MUX_SEL) al GPIO 48 dell'ESP32.

🧪 Test di funzionamento: Seleziona la modalità FM/DAB dal display touch, effettua una scansione delle frequenze e verifica la memorizzazione dei preset e la limpidezza dell'audio sintonizzato.

Modulo 5: Integrazione Web Radio Wi-Fi e Standby (Light Sleep)
Obiettivo: Configurare la connessione Wi-Fi per lo streaming delle Web Radio gratuite e ottimizzare i consumi con la modalità di accensione istantanea.

Materiale necessario:

Il sistema completo assemblato nei moduli precedenti

Rete Wi-Fi domestica

Passi di montaggio:

Aggiungi nel codice la gestione Wi-Fi e la lista degli URL delle Web Radio preferite gestibili tramite l'interfaccia touch.

Configura la modalità Light Sleep: alla pressione prolungata del tasto di spegnimento, disattiva la retroilluminazione del display (GPIO 21 a LOW), metti in muto l'amplificatore e mantieni attivi i pin di risveglio (GPIO 2 dell'encoder e GPIO 1 dell'RTC).

🧪 Test di funzionamento finale:

Seleziona una Web Radio dalla lista touch e verifica lo streaming fluido tramite Wi-Fi.

Metti il sistema in standby e verifica che si riattivi istantaneamente alla pressione della manopola o all'orario della sveglia programmata.