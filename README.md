<div align="center">

# AirBridge

### D&R Airlite → UDP Bridge per Windows

**Controlla la tua regia attraverso i pulsanti e gli eventi del mixer D&R Airlite.**

![Versione](https://img.shields.io/badge/versione-1.4.1-385BCA?style=flat-square)
![Piattaforma](https://img.shields.io/badge/piattaforma-Windows%2010%2F11-555?style=flat-square)
![Protocollo](https://img.shields.io/badge/protocollo-UDP-1E936C?style=flat-square)

**[Scarica l'ultima versione](../../releases/latest) · [Segnala un problema](../../issues)**

</div>

---

## Cos'è AirBridge?

AirBridge è un'applicazione Windows che collega il mixer **D&R Airlite** a un sistema di automazione o regia controllabile via UDP.

Invece di simulare la pressione di tasti sulla tastiera del PC, come fa il Virtual Keyboard Mapper, AirBridge associa agli eventi del mixer **uno o più comandi UDP configurabili**, eventualmente separati da un ritardo.

Può inoltre gestire direttamente i **16 LED dei pulsanti Control** del mixer: colore fisso oppure dinamico, con stati ON e OFF personalizzabili per ogni pulsante.

```text
        D&R Airlite
             │ USB
             ▼
       Airlite Control
             │ UDP
             ▼
          AirBridge
          ├── Eventi PRESS / RELEASE e moduli
          ├── Regole e sequenze con delay
          ├── Gestione LED rosso / verde / spento
          └── Log e diagnostica
             │ UDP: password!codice
             ▼
         Regia / software di destinazione
```

> **Nota:** AirBridge è un progetto indipendente, non ufficiale e non affiliato a D&R Electronica. Per comunicare con il mixer USB è necessario mantenere in esecuzione **Airlite Control**.

## Funzionalità

- **Mappatura pulsanti Control:** 16 tasti `1A–8A` e `1B–8B`, con azioni distinte per `PRESS` e `RELEASE`.
- **Eventi dei moduli:** ON/OFF, fader e CUE dei canali, con possibilità di associare regole UDP.
- **Sequenze UDP:** più codici per evento e ritardo configurabile in millisecondi tra gli step.
- **Formato di uscita:** `password!codice`, inviato come testo UTF-8 all'IP e alla porta scelti dall'utente.
- **LED personalizzabili (modalità C):** fisso (spento, rosso, verde) oppure dinamico con colore ON/OFF e stato iniziale indipendenti.
- **Sincronizzazione LED:** aggiornamento dei 16 LED all'avvio e quando si applica una modifica alla configurazione.
- **Spegnimento LED:** invio del comando OFF a tutti i 16 LED quando si preme **Ferma** o si esce dal programma.
- **Funzionamento in background:** icona nell'area di notifica (system tray), minimizzazione e avvio automatico opzionale con Windows.
- **Diagnostica:** log degli eventi Airlite e dei comandi UDP, contatore RX e visualizzazione HEX dei pacchetti per il debug.
- **Nessuna DLL D&R richiesta:** la comunicazione con Airlite Control avviene tramite UDP.

## Download e requisiti

Scarica `AirBridge.exe` dagli **Assets** dell'[ultima release](../../releases/latest).

**Requisiti:**

- Windows 10 o 11;
- mixer **D&R Airlite** collegato e riconosciuto dal PC;
- software **Airlite Control** installato e in esecuzione;
- rete UDP raggiungibile verso l'applicazione di destinazione.


> Il programma non include il software ufficiale Airlite Control, da ottenere separatamente tramite i canali del produttore.

## Configurazione iniziale

1. Avvia **Airlite Control** e assicurati che rilevi il mixer.
2. **Chiudi completamente** il *Virtual Keyboard Mapper* ufficiale, anche dall'area di notifica: potrebbe utilizzare la stessa porta UDP di ricezione.
3. Avvia `AirBridge.exe`.
4. Nella schermata **Panoramica** verifica gli indirizzi e le porte Airlite.
5. Inserisci **IP**, **porta** e **password** del software di destinazione.
6. Premi **Salva e applica** e poi **Avvia AirBridge**.
7. Nella schermata **Regole e comandi** associa gli eventi del mixer ai codici UDP desiderati.
8. In **Mixer Airlite + LED** imposta la modalità e i colori dei singoli pulsanti.

### Parametri Airlite predefiniti

| Funzione | Valore predefinito |
| --- | --- |
| Indirizzo Airlite Control | `127.0.0.1` |
| Ricezione eventi Airlite (CUSTOM) | UDP `19550` |
| Invio comandi LED verso Airlite Control | UDP `19551` |
| IP e porta della regia | Configurabili |
| Formato comando verso la regia | `password!codice` |

Il listener UDP di AirBridge è dedicato **esclusivamente agli eventi Airlite**: non è presente un listener generico o una funzione di apprendimento UDP.

## Esempio di mappatura

Si vuole associare al tasto `1A` un comando di avvio e al rilascio un comando separato:

| Evento Airlite | Azione UDP | Attesa |
| --- | --- | ---: |
| `AIRLITE_1A_PRESS` | `password!100` | 0 ms |
| `AIRLITE_1A_PRESS` | `password!101` | 250 ms |
| `AIRLITE_1A_RELEASE` | `password!102` | 0 ms |

Nel programma si inseriscono **solo i codici** (`100`, `101`, `102`): la password e il separatore `!` vengono aggiunti automaticamente in trasmissione.

### Gestione dei LED

Ogni pulsante può essere configurato indipendentemente:

- **Fisso:** LED sempre spento, rosso oppure verde durante il funzionamento di AirBridge.
- **Dinamico:** un colore per lo stato OFF, uno per lo stato ON e uno stato iniziale configurabile. Ogni `PRESS` valido alterna lo stato; `RELEASE` non cambia il LED.

Esempio: per `1A` si può scegliere **verde = OFF** e **rosso = ON**. All'avvio viene applicato lo stato configurato; dopo una pressione il LED passa all'altro colore.

Quando premi **Ferma** oppure selezioni **Esci** dalla tray, AirBridge invia un comando OFF per ciascuno dei 16 LED. La **X** della finestra, invece, nasconde il programma nella tray senza interromperne il funzionamento e senza spegnere i LED.

> I comandi verso il mixer sono UDP: l'invio non costituisce una conferma della ricezione. Una terminazione forzata del processo o l'arresto improvviso del PC può impedire lo spegnimento finale dei LED.

## Log e risoluzione problemi

Apri **Log e debug** per verificare il traffico. In **Panoramica** puoi attivare anche il **log HEX dei pacchetti RX**.

**Il mixer non genera eventi PRESS/RELEASE?**

1. Controlla che **Airlite Control** sia in esecuzione.
2. Assicurati che il **Virtual Keyboard Mapper** ufficiale sia chiuso.
3. Verifica la porta di ricezione `19550/UDP` e gli eventuali conflitti di porta/firewall.
4. Premi e rilascia un pulsante del mixer e osserva il contatore **RX Airlite**.
5. Se RX resta a zero, AirBridge non riceve pacchetti. Se RX aumenta ma non compaiono eventi riconosciuti, controlla il log HEX: potrebbe essere necessario adattare la decodifica alla versione firmware in uso.

**Le regole vengono riconosciute ma la regia non reagisce?** Verifica IP, porta, password, codice e connettività UDP sul sistema destinatario.

Percorsi utili:

```text
%APPDATA%\AirBridge\settings.json
%LOCALAPPDATA%\AirBridge\logs\airbridge.log
```

La password è protetta tramite **DPAPI** nel profilo Windows ed è oscurata nei log applicativi.

## Stato del progetto

**Versione 1.4.1 — 11 novembre 2026**

- Sincronizzazione dei LED all'avvio e quando si applicano modifiche.
- Spegnimento di tutti i LED su **Ferma** e **Esci**.
- Rimozione delle precedenti funzioni di apprendimento e ricezione UDP generica.
- Comunicazione Airlite tramite UDP, senza DLL del produttore.

**Stato dei test:** sono stati eseguiti test software e la gestione dei colori LED è stata verificata sul mixer. La ricezione degli eventi **PRESS/RELEASE sull'hardware reale** richiede ancora conferma con questa versione: si consiglia di considerare il rilascio *sperimentale* finché non è completato il collaudo.

## Feedback e segnalazioni

Per segnalare un problema usa la sezione [Issues](../../issues) del repository, indicando:

- versione di AirBridge e di Windows;
- versione di Airlite Control, se nota;
- evento atteso e comportamento osservato;
- porzione di log pertinente (anche HEX, se necessario).

**Non pubblicare password, indirizzi riservati o altri dati sensibili nei log allegati.**

---

<sub>AirBridge è un progetto indipendente. D&R Airlite e gli altri nomi di prodotto citati appartengono ai rispettivi titolari.</sub>
