Ecco lo schema: l'ESP32-S3 sta a cavallo del canale centrale e i moduli lo circondano, con i comandi in alto e la catena audio in basso.

Lo schema segue la mappa pin della Lezione 0, quindi i GPIO sono quelli già proposti nel corso.

- **Comandi in alto:** display, i due encoder e i pulsanti sono collegati direttamente all'ESP32.
- **Bus SPI condiviso:** su SCK 12, MOSI 11 e MISO 13 stanno display, microSD e tuner, ciascuno con il proprio CS.
- **Audio in corallo:** tuner e DAC arrivano entrambi al relè selettore, che sceglie la sorgente prima dell'amplificatore e della cassa.
- **Alimentazione:** l'alimentatore da 5 V va direttamente ai morsetti del PAM8403, con il condensatore da 1000 µF vicino, e non dalle rotaie della breadboard.
- **Non disegnati:** i fili di alimentazione 3,3 V e massa dei moduli, e il GPIO 41 (accensione e spegnimento dell'amplificatore, come descritto nella Lezione 12).

Vuoi che prepari anche lo schema elettrico dettagliato di una sezione, per esempio DAC e amplificatore, con i numeri dei pin dei moduli?