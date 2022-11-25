# linux-insides

Un libro in progresso sul kernel linux e il suo funzionamento.

**L'obiettivo è semplice** - condividere la mia modesta conoscenza su come funziona il kernel linux e aiutre persone interessate a questo argomento o comunque ad argomenti di basso livello. Sentitevi liberi di leggere tutto il libro, vi consiglio di partire da [qui](SUMMARY.md).  

**Domande/Consigli**: Non abbiate paura a farmi domande o a chiedermi un consiglio pingandomi su twitter [0xAX](https://twitter.com/0xAX), aggiungendo un issue oppure inviandomi una [email](mailto:anotherworldofworld@gmail.com).

Potete generare eBooks e PDF - [documentazione](https://github.com/GitbookIO/gitbook/blob/master/docs/ebook.md)

Visto che ci sono, aggiungo qualche informazione personale. Io sono il traduttore della versione originale, quella di [0xAX](https://github.com/0xAX/linux-insides). Ho deciso per passione personale di provare a dare un mio contributo traducendolo in italiano. Sebbene non sia di sicuro così esperto come lo è il creatore, potete comunque contattare anche me a [questa_email](mailto:giacomo.pauletti@gmail.com).

Aggiungerò inoltre riflessioni personali quando le ritengo necessarie per la mia comprensione.


# Mailing List

Abbiamo una Google Group mailing list per imparare il codice sorgente del kernel linux. Ecco alcune istruzioni su come usarla.

### **Join**

Invia un email con qualsiasi contenuto e oggetto a `kernelhacking+subscribe@googlegroups.com`. Poi riceverai una email di conferma. Rispondi con qualsiasi contenuto e poi hai finito.

> Se hai un account Google, puoi anche aprire la [pagina di archivio](kernelhacking+subscribe@googlegroups.com) e schiaccia su **Apply to join group**. Verrai accettato automaticamente.

### **Inviare emails alla mailing list**

Semplicemente invia una email a `kernelhacking@googlegroups.com`. L'utilizzo base è lo stesso di quello delle altre mailing lists fornite da mailman.

### **Archivi**

https://groups.google.com/forum/#!forum/kernelhacking

## In altre lingue

* [Originale](https://github.com/0xAX/linux-insides)
* [Brasiliano_Portoghese](https://groups.google.com/forum/#!forum/kernelhacking)
* [Cinese](https://github.com/MintCN/linux-insides-zh)
* [Giapponese](https://github.com/tkmru/linux-insides-ja)
* [Coreano](https://github.com/junsooo/linux-insides-ko)
* [Russo](https://groups.google.com/forum/#!forum/kernelhacking)
* [Spagnolo](https://groups.google.com/forum/#!forum/kernelhacking)
* [Turco](https://groups.google.com/forum/#!forum/kernelhacking)

## Docker

Se vuoi eseguire la tua copia del libro con gibook in un container locale:
1. Abilita la feature sperimentale di docker tramite un editor di testo
    ```
    sudo vim /usr/lib/systemd/system/docker.service
    ```

    Then add --experimental=true to the end of the ExecStart
    i aggiungici add --experimental=true alla fine della linea che termina con ExecStart=/usr/bin/dockerd -H fd://
    Diventa cioè *ExecStart=/usr/bin/dockerd -H fd:// --experimental=true*
    Poi salva

    Poi th serve ricaricare il daemon di docker:

    ```
    systemctl daemon-reload
    systemctl restart docker.service
    ```

2. Esegui l'immagine di docker
    ``` 
    make run
    ```
3. Apri la tua copia locale del libro a questo url: http://localhost:4000 oppure esegui `make browse`

## Contributi

Se volete contribuire a questa repository fate pure una pull request o aprite un issue. Altrimenti andate alla repository originale e vedete le condizioni per apportare cambiamenti.

## Autori

[@0xAX](https://twitter.com/0xAX)
[Io](https://github.com/GiacomoPauletti) solo per traduzione e qualche aggiunta