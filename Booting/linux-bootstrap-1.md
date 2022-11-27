# **Avviamento del Kernel. Parte 1.**

# Dal bootloader al kernel

Se hai letto il mio [post_precedente](https://0xax.github.io/categories/assembler/), potresti aver notato che sono stato coinvolto con la programmazione di basso livello per un po' di tempo. Ho scritto un po' di articoli sulla programmazione in assembly per x86_64 Linux e, allo stesso tempo, ho iniziato a immergermi nel codice sorgente del Kernel Linux.  

Ho un grande interesse nel capire come funzionano le cose a basso livello, coem vengono eseguiti i programmi sul computer, coem solo posizionati in memoria, come il kernel gestisce i processi e la memoria, come funziona la pila protocollare di internet a basso livello e molte altre cose. Cos', ho deciso di scrivere un'altra serie di post sul kernel Linux per l'architettura **x86_64**.

Nota che non sono un hacker professionale del kernel e non scrivo codice per il kernel al lavoro. E' solo un hobby. Semplicemente mi piacciono le cose a basso livello ed e' interessante capire come funziona quel mondo. Quidni se trovi qualcosa di confusionario, o hai domande o suggerimenti, pingami su Twitter [0xAX](https://twitter.com/0xAX), inviami una [email](mailto:anotherworldofworld@gmail.com) o crea un issue. Lo apprezzo molto.

*Questa non è documentazione ufficiale ma solo apprendimenti personale e condivisione delle mie conoscenze*

**Conoscenze richieste**

* Conoscenza del C
* Conoscenza dell'assembly (sintassi AT&T)

In ogni caso, se stai soltanto iniziando a imparare questi strumenti, ti spiegero alcune parti nei posts. 

Ok, questa è la fine dell'introduzione, immergiamoci nel kernel Linux e nel mondo low-level!

Ho iniziato a scrivere questi post al tempo del kernel linux 3.18 e molte cose sono cambiate da quel periodo. Se ci sono dei cambiamenti, aggiornerò i miei post a dovere.

# Il Bottone d'Accensione Magico. Cosa succede poi?

Sebbene questa serie di post sia sul kernel Linux, non partiremo direttamente sal codice del kernel. Appena premi il magico bottone d'accensione sul tuo laptop o desktop, esso inizia a funzionare. La motherbouard invia un segnale al *power supply*. Dopo aver ricevuto questo segnale, il power supply fornisce la giusta quantità di elettricità al computer e invia alla motherboard il [power_good_signal](https://en.wikipedia.org/wiki/Power_good_signal). Quindi la motherboard porva ad avviare la CPU.  
La CPU resetta tutti i resti di dati che erano present inei registri e imposta dei valori predefiniti per ciascuno di questi.

Per la CPU 80386 o le successive, alcuni dei valori predefiniti sono i seguenti:
```
IP            0xfff0
CS selector   0xf000
CS base       0xffff0000
```

Questi valori ci serviranno fra un attimo.

I processori iniziano a lavorare in [real_mode](https://en.wikipedia.org/wiki/Real_mode).

> **N.B: REAL MODE**  
>
> In breve la real mode è la versione più intuitiva. Il nome deriva dal fatto che ogni indirizzo corrisponde a quello fisico (reale). Quindi non c'è paging. E' disponibile solo 1 MByte. Ho accesso diretto a tutta questa memoria, a tutti gli indirizzi di I/O e periferiche Hardware.  
> Non ho però il multitasking e i livelli di privilegio

Prima di procedere però, proviamo a capire la [segmentazione_di_memoria](https://en.wikipedia.org/wiki/Memory_segmentation) in questa modalità. La segmentazione è supportata da tutti i processori x86 a partire dalla CPU 8086.
La CPU `8086` ha un bus di indirizzi di 20 bit, che significa che può lavorare con `1 MByte` di spazio indirizzi. Ma ha solo registri a `16-bit`, quindi 64 KByte di indirizzamento massimo.

Qui entra in gioco la segmentazione della memoria, che serve per usare tutto lo spazio di indirizzamento disponibile. Tutta la memoria è divisa in segmenti di dimensioni fisse di `65535 byte` (64KiB).  
L'indirizzo contenuto nei registri a 16-bit (detto *offset*) mi servirà a **selezionare un certo byte nel segmento**.  
Poi ci sono vari registri da 16-bit (CS, SS, DS, ES, FS, GS, ...) che mi permettono di selezionare il segmento. Questi registri sono detti appunto *Segment Selector*

Per calcolare l'indirizzo finale quindi faccio questo:
```
IndirizzoFisico = SegmentSelector * 2^4 + Offset
```

Moltiplico per 2^4 il registro selettore perchè così lo estendo a 20 bit, poi nei 16 bit inferiori ci sommo Offset.

### **Esempi**

* La sintassi è `CS:IP`, ad esempio `0x2000:0x0010`, in questo caso l'indirizzo fisico corrispondente sarebbe:
    ```
    0x2000 << 4 + 0x0010 = 0x20010
    ```

* Se però avessi `0xffff:0xffff`, otterrei come risultato `0x10ffef` che è `65520` byte oltre 1 MegaByte. Ottengo questo? perchè "shiftando" il selettore a 20 bit, esso potenzialmente può indirizzare `2^20 = 1MegaByte`, tuttavia se gli sommo un valore non nullo (l'offset), supero 1 MegaByte. In questo caso gli sommo `64KiB = 65536 byte` ma supero 1MB di "solo" 65520 perchè i `16 byte` di differenza sono dati dal fatto che shiftando il selettore gli inserisco degli 0 e non degli 1.

### **Caricamento del BIOS**

Ma perchè proprio l'indirizzo 0xfffffff0? Non potevano metterne un altro di default? La risposta è sì, ma la convenzione è che gli indirizzi finali dello spazio indirizzabile **non sono di RAM ma di ROM**. Questa ROM è quella che **contiene il BIOS**. 

Siccome la ROM è di solito più lenta della RAM, per prima cosa si *carica in RAM* una versione compressa del BIOS, la si decomprime e la si avvia. 

Ecco i primissimi passi fatti da [coreboot](https://www.coreboot.org/) (il source code in [src/cpu/x86/reset16.inc](https://review.coreboot.org/plugins/gitiles/coreboot/+/refs/heads/4.11_branch/src/cpu/x86/16bit/reset16.inc)): 

```assembly
    .section ".reset", "ax", %progbits
    .code16
.globl	_start
_start:
    .byte  0xe9
    .int   _start16bit - ( . + 2 )
    ...
```
Questa sezione viene chiamata ".reset" (verrà usato nel codice successivo, quello del linker).  

Possiamo trovare la `jmp` (il suo opcode è *0xe9*) e l'indirizzo di salto `_start16bit - ( . + 2)`

Viene specificato (`.code16`) che si lavora su 16 bit perchè il computer non è ancora entrato in protected mode e quindi è obbligato a lavorare così.

Questo codice è *posizionato in memoria dal linker*, all'indirizzo `0xfffffff0` ([src/cpu/x86/16bit/reset16.ld](https://review.coreboot.org/plugins/gitiles/coreboot/+/refs/heads/4.11_branch/src/cpu/x86/16bit/reset16.ld)):

```
SECTIONS {
    /* Trigger an error if I have an unusable start address */
    _bogus = ASSERT(_start16bit >= 0xffff0000, "_start16bit too low. Please report.");
    _ROMTOP = 0xfffffff0;
    . = _ROMTOP;
    .reset . : {
        *(.reset);
        . = 15;
        BYTE(0x00);
    }
}
```

Per la spiegazione approfondita di questo codice si guardi la mia [altra_repository](https://github.com/GiacomoPauletti/linker-scripting-bm/blob/main/note.md#esempio)

