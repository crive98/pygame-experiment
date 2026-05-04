# Era Tactics

> Nome provvisorio del progetto.  
> Gioco strategico tattico a turni dinamici su griglia, con mappe procedurali, progressione tecnologica per ere, truppe evolvibili e combattimenti basati su visibilità, terreno e iniziativa.

---

## Indice

1. [Descrizione del gioco](#1-descrizione-del-gioco)
2. [How to play](#2-how-to-play)
3. [Struttura della partita](#3-struttura-della-partita)
4. [Mappa e territori](#4-mappa-e-territori)
5. [Terreni](#5-terreni)
6. [Schieramento](#6-schieramento)
7. [Unità e truppe](#7-unità-e-truppe)
8. [Generale](#8-generale)
9. [Timeline, iniziativa e costo azione](#9-timeline-iniziativa-e-costo-azione)
10. [Movimento](#10-movimento)
11. [Attacco](#11-attacco)
12. [Distanza di Chebyshev](#12-distanza-di-chebyshev)
13. [Visibilità e informazioni nascoste](#13-visibilità-e-informazioni-nascoste)
14. [Danni](#14-danni)
15. [Eliminazione dei generali e ribelli](#15-eliminazione-dei-generali-e-ribelli)
16. [CPU e comportamento deterministico](#16-cpu-e-comportamento-deterministico)
17. [Economia e ricompense](#17-economia-e-ricompense)
18. [Esperienza e livelli](#18-esperienza-e-livelli)
19. [Progressione permanente](#19-progressione-permanente)
20. [Ere tecnologiche](#20-ere-tecnologiche)
21. [DAG tecnologico completo](#21-dag-tecnologico-completo)
22. [Evoluzione delle truppe](#22-evoluzione-delle-truppe)
23. [Catene evolutive corrette](#23-catene-evolutive-corrette)
24. [Regole consolidate](#24-regole-consolidate)
25. [Controlli](#25-controlli)

---

# 1. Descrizione del gioco

**Era Tactics** è un gioco strategico tattico su griglia in cui il giocatore guida un esercito attraverso diverse ere tecnologiche, partendo da una civiltà primitiva fino ad arrivare a tecnologie futuristiche come intelligenza artificiale, droni, nanotecnologie e guerra orbitale.

Ogni partita si svolge su una mappa generata casualmente. La mappa è composta da caselle, e ogni casella rappresenta una specifica tipologia di terreno, come pianura, foresta, montagna, mare, collina, palude, strada o fortezza.

Il gioco può prevedere fino a quattro fazioni contemporaneamente:

```text
Giocatore
CPU 1
CPU 2
CPU 3
```

Ogni fazione schiera un esercito composto da:

```text
1 Generale obbligatorio
N truppe selezionate dal proprio roster
```

Lo scopo della partita è eliminare tutti i generali avversari. Quando il generale di una fazione viene sconfitto, quella fazione viene eliminata dalla partita e le sue truppe superstiti diventano **ribelli**. I ribelli non possono vincere la partita, ma continuano ad agire e attaccano chiunque vedano.

Il gioco non usa turni classici per fazione. Non esiste quindi una sequenza del tipo:

```text
Muove tutto il giocatore
Muove tutta la CPU 1
Muove tutta la CPU 2
Muove tutta la CPU 3
```

Al contrario, ogni unità ha una propria posizione nella timeline. Agisce sempre l'unità con il costo azione più basso. Dopo aver mosso o attaccato, il suo costo azione aumenta. Questo genera un sistema dinamico in cui le unità più veloci agiscono più spesso, mentre quelle più lente agiscono meno frequentemente.

La profondità tattica nasce da:

```text
terreno
visibilità
costo movimento
costo attacco
ampiezza attacco
ampiezza visibilità
scelta del territorio iniziale
composizione dell'esercito
progressione tecnologica
evoluzione delle truppe
esperienza individuale delle unità
```

Non esistono counter rigidi del tipo "cavalleria forte contro arcieri" o "arcieri deboli contro fanteria". Le unità non hanno ruoli meccanici obbligatori. Il giocatore può usare ogni truppa come preferisce. Un arciere può essere usato come esca, un cavaliere come esploratore, un guerriero come blocco difensivo o il generale come centro avanzato di visibilità, se il giocatore accetta il rischio.

---

# 2. How to play

## 2.1 Obiettivo principale

L'obiettivo di ogni partita è:

```text
eliminare tutti i generali avversari
```

Il giocatore perde se il proprio generale viene sconfitto.

La partita termina quando:

```text
il giocatore elimina tutti i generali nemici
oppure
il generale del giocatore viene eliminato
```

## 2.2 Fasi principali

Ogni partita segue questo ciclo:

```text
1. Il giocatore si trova in una determinata era tecnologica.
2. La dimensione della mappa viene determinata dall'era corrente.
3. La mappa viene generata casualmente.
4. Il gioco individua territori plausibili per lo schieramento.
5. Il giocatore sceglie il proprio territorio iniziale.
6. Il giocatore seleziona le truppe da schierare in base al territorio scelto.
7. Le CPU scelgono territorio e truppe.
8. Tutti i giocatori ricevono lo stesso budget di schieramento.
9. La timeline delle unità viene inizializzata.
10. Agisce sempre l'unità con costo azione più basso.
11. Ogni unità può muovere di una casella oppure attaccare.
12. La partita procede fino all'eliminazione dei generali.
13. A fine partita vengono assegnate monete ed esperienza.
14. Fuori partita il giocatore può acquistare truppe, upgrade, tecnologie ed evoluzioni.
```

## 2.3 Strategia di base

Il giocatore non sceglie le truppe prima di conoscere la mappa. Prima viene generata la mappa, poi viene scelto il territorio, e solo dopo si decide quali truppe schierare.

Questo significa che la composizione dell'esercito deve adattarsi al territorio.

Esempi:

```text
Territorio costiero:
utile schierare navigatori o unità capaci di controllare il mare.

Territorio montuoso:
utile schierare unità con buona visibilità o attacco a distanza.

Territorio boschivo:
utile schierare fanteria, esploratori o unità non troppo penalizzate dalla foresta.

Territorio in pianura:
utile schierare cavalleria o unità mobili.

Territorio con molte strettoie:
utile schierare truppe resistenti e unità ad ampia gittata.
```

---

# 3. Struttura della partita

La struttura della partita è progettata intorno a tre elementi:

```text
mappa procedurale
scelta tattica dello schieramento
timeline dinamica delle unità
```

## 3.1 Sequenza completa della partita

```text
1. Determinazione dell'era corrente del giocatore.
2. Determinazione della dimensione della mappa in base all'era.
3. Generazione casuale della mappa.
4. Analisi della mappa.
5. Individuazione di territori plausibili per lo schieramento.
6. Scelta del territorio da parte del giocatore.
7. Scelta delle truppe da schierare.
8. Schieramento del generale obbligatorio.
9. Schieramento delle truppe selezionate.
10. Scelta e schieramento delle CPU.
11. Inizializzazione del costo azione di tutte le unità.
12. Ordinamento della timeline.
13. Attivazione dell'unità con costo azione minore.
14. Movimento o attacco dell'unità attiva.
15. Aggiornamento del costo azione dell'unità.
16. Riordinamento della timeline.
17. Ripetizione fino alla conclusione della partita.
18. Calcolo delle ricompense.
19. Calcolo dell'esperienza.
20. Ritorno alla gestione permanente del giocatore.
```

## 3.2 Numero di giocatori

Una partita può includere:

```text
2 giocatori totali: giocatore + 1 CPU
3 giocatori totali: giocatore + 2 CPU
4 giocatori totali: giocatore + 3 CPU
```

Il numero massimo previsto è:

```text
1 giocatore umano + 3 CPU
```

## 3.3 Budget equo

Tutti i partecipanti alla stessa partita devono avere lo stesso budget di schieramento.

Regola:

```text
Tutti i giocatori della partita hanno lo stesso budget.
```

Questa regola evita squilibri strutturali. La difficoltà non deve derivare dal fatto che la CPU abbia più punti schieramento, ma da:

```text
scelte tattiche
composizione dell'esercito
territorio iniziale
uso del terreno
visibilità
tecnologie disponibili
livello delle truppe
evoluzioni
```

---

# 4. Mappa e territori

## 4.1 Dimensione della mappa

La dimensione della mappa cresce con l'era tecnologica.

La mappa finale massima è:

```text
32x32
```

Nelle prime ere, però, si parte da mappe più piccole. Questo evita che le prime partite siano troppo lunghe o complesse.

Progressione consigliata:

| Era | Dimensione mappa |
|---|---:|
| Primitiva | 10x10 |
| Antica | 12x12 |
| Classica | 16x16 |
| Medievale | 18x18 |
| Rinascimentale | 20x20 |
| Industriale | 24x24 |
| Moderna | 28x28 |
| Futuristica | 32x32 |

## 4.2 Generazione casuale

La mappa viene generata casualmente a inizio partita.

La generazione non dovrebbe produrre una distribuzione completamente caotica. La mappa deve essere giocabile e tatticamente interessante.

Una generazione plausibile dovrebbe:

```text
creare biomi coerenti
evitare territori completamente isolati
evitare spawn impossibili
garantire percorsi tra le aree principali
creare zone strategiche
distribuire ostacoli naturali
distribuire mare, montagne e foreste in modo leggibile
```

## 4.3 Territori iniziali plausibili

Dopo la generazione della mappa, il gioco individua automaticamente territori plausibili per lo schieramento.

Un territorio plausibile dovrebbe avere:

```text
spazio sufficiente per generale e truppe
accesso ad almeno una direzione di espansione
distanza ragionevole dagli altri territori iniziali
assenza di isolamento totale
terreni attraversabili per almeno una parte significativa delle truppe
possibilità tattiche interessanti
```

Esempi di territori plausibili:

```text
angolo nord-ovest con pianura e foresta
penisola costiera
altopiano montuoso accessibile
zona centrale protetta da colline
area vicina a una strada
```

## 4.4 Scelta del territorio prima delle truppe

Il giocatore sceglie il territorio prima di scegliere le truppe.

Questa regola è centrale.

Il territorio non è solo estetica. Determina quali truppe sono più adatte.

Esempi:

```text
Se il territorio ha molto mare vicino:
le unità navali diventano più utili.

Se il territorio è circondato da foreste:
la cavalleria può essere penalizzata.

Se il territorio ha montagne:
unità con alta visibilità o attacco a distanza possono essere molto efficaci.

Se il territorio è pianeggiante:
le unità mobili possono esprimersi meglio.
```

---

# 5. Terreni

## 5.1 Terreni base

I terreni previsti sono:

```text
Pianura
Foresta
Montagna
Mare
Collina
Palude
Strada
Fortezza/Base
```

## 5.2 Funzione del terreno

Ogni terreno modifica tre aspetti fondamentali:

```text
movimento
ampiezza attacco
ampiezza visibilità
```

Il terreno non modifica direttamente il danno.

Questa distinzione è molto importante:

```text
Il terreno influenza come ti muovi, quanto vedi e quanto lontano puoi attaccare.
Il terreno non riduce direttamente i danni subiti.
```

## 5.3 Modificatori per tipo di truppa

Ogni terreno può modificare ogni tipologia di truppa in modo diverso.

Esempio concettuale:

```text
Foresta:
- Fanteria: movimento leggermente penalizzato
- Cavalleria: movimento molto penalizzato
- Arcieri: ampiezza attacco ridotta
- Esploratori: penalità ridotta o nulla
- Mezzi pesanti: movimento molto penalizzato

Montagna:
- Fanteria: attraversabile lentamente
- Cavalleria: non attraversabile
- Arcieri: ampiezza attacco aumentata
- Unità navali: non attraversabile
- Unità aeree: attraversabile senza problemi

Mare:
- Unità terrestri: non attraversabile
- Navigatori: attraversabile
- Unità aeree: attraversabile
- Unità anfibie future: attraversabile con penalità
```

## 5.4 Caselle non attraversabili

Alcune truppe non possono entrare in determinati terreni.

Esempi:

```text
Cavalleria antica non può entrare in montagna.
Fanteria non può entrare in mare.
Navigatori non possono entrare in montagna.
Carri armati possono essere bloccati da montagne o paludi estreme.
```

Se una casella non è attraversabile da una truppa, non compare tra le mosse disponibili.

---

# 6. Schieramento

## 6.1 Punti schieramento

Lo schieramento usa un sistema a punti.

Ogni partita fornisce un budget di schieramento.

Ogni unità ha un costo.

Esempio:

```text
Budget partita: 20

Guerriero: 3
Arciere: 4
Cavaliere: 5
Navigatore: 4
Esploratore: 2
Generale: obbligatorio
```

Il generale è obbligatorio e non deve essere sbloccato.

## 6.2 Budget uguale per tutti

Tutti i giocatori hanno lo stesso budget.

Esempio:

```text
Giocatore: 20 punti
CPU 1: 20 punti
CPU 2: 20 punti
CPU 3: 20 punti
```

## 6.3 Budget legato all'era

Il budget aumenta con l'era.

Progressione indicativa:

| Era | Budget indicativo |
|---|---:|
| Primitiva | 8 |
| Antica | 12 |
| Classica | 18 |
| Medievale | 24 |
| Rinascimentale | 30 |
| Industriale | 36 |
| Moderna | 44 |
| Futuristica | 54 |

Il budget può essere ulteriormente modificato da tecnologie logistiche, ma sempre in modo uguale per tutte le fazioni nella partita oppure gestito come livello di difficoltà coerente.

---

# 7. Unità e truppe

## 7.1 Proprietà concettuali di una truppa

Ogni truppa possiede:

```text
nome
classe
HP
attacco
difesa
costo movimento corrente
costo attacco corrente
ampiezza attacco
ampiezza visibilità
costo schieramento
livello
esperienza
possibili evoluzioni
```

Ogni truppa non deve avere:

```text
era tecnologica come proprietà individuale principale
forte contro
debole contro
ruolo meccanico obbligatorio
bonus difensivo terreno
bonus offensivo terreno
```

L'era è una proprietà globale del giocatore, non della singola truppa.

Una truppa può essere sbloccata da una tecnologia appartenente a una certa era, ma l'era non deve essere trattata come proprietà meccanica individuale della truppa.

## 7.2 Costo movimento e costo attacco

Ogni unità ha due costi distinti:

```text
costo movimento corrente
costo attacco corrente
```

Questo è importante perché una truppa può essere veloce a muoversi ma lenta ad attaccare.

Esempio:

```text
Arciere:
movimento medio
attacco lento perché deve caricare e mirare

Cavaliere:
movimento rapido
attacco meno frequente

Guerriero:
movimento medio/lento
attacco regolare
```

## 7.3 Ruoli non vincolanti

Le unità possono avere una descrizione di ruolo, ma il ruolo non deve imporre regole rigide.

Esempi di ruolo descrittivo:

```text
unità mobile
unità da tiro
unità resistente
unità navale
unità esplorativa
unità d'assedio
```

Ma il giocatore può usarle liberamente.

---

# 8. Generale

## 8.1 Disponibilità del generale

Il generale è sempre disponibile.

Regola fondamentale:

```text
Il generale non si compra.
Il generale non si sblocca.
Il generale è sempre obbligatorio.
```

Il giocatore deve poter giocare fin dall'inizio. Per questo il generale non può essere vincolato a una tecnologia.

## 8.2 Upgrade automatico del generale

Il generale si aggiorna automaticamente quando il giocatore passa di era.

Catena automatica:

```text
Generale Tribale
→ Generale Antico
→ Generale Classico
→ Generale Medievale
→ Generale Rinascimentale
→ Generale Industriale
→ Generale Moderno
→ Generale Futuristico
```

Questo passaggio è automatico e segue l'era corrente del giocatore.

## 8.3 Funzione del generale

Il generale è:

```text
unità obbligatoria
condizione di sconfitta
centro di comando visivo
unità con ampiezza visibilità molto alta
```

Il generale non fornisce bonus alle altre truppe, almeno nella versione iniziale.

## 8.4 Visibilità del generale

Il generale deve avere un'ampiezza di visibilità estremamente ampia rispetto alle altre unità.

Questo crea un dilemma tattico:

```text
tenerlo indietro e vedere meno
oppure
portarlo avanti per aumentare la visibilità, rischiando la partita
```

## 8.5 Sconfitta del generale

Quando il generale di una fazione arriva a 0 HP:

```text
la fazione viene eliminata
le truppe superstiti diventano ribelli
```

---

# 9. Timeline, iniziativa e costo azione

## 9.1 Sistema di iniziativa

Il gioco non usa turni per fazione.

Usa una timeline basata sul costo azione delle singole unità.

Ogni unità ha durante la partita:

```text
costo azione partita
```

All'inizio della partita:

```text
costo azione partita = costo movimento corrente
```

Agisce sempre l'unità con:

```text
costo azione partita minore
```

## 9.2 Progressione lineare

Il costo azione non raddoppia.

La progressione è lineare.

Se un'unità ha costo movimento corrente 2, senza modificatori terreno la sua progressione sarà:

```text
2 → 4 → 6 → 8 → 10
```

Esempio:

```text
Cavalleria:
costo movimento corrente = 2

Inizio partita:
costo azione = 2

Dopo prima azione movimento:
costo azione = 4

Dopo seconda azione movimento:
costo azione = 6

Dopo terza azione movimento:
costo azione = 8
```

## 9.3 Movimento e attacco aggiornano il costo in modo diverso

Quando una truppa muove:

```text
costo azione += costo movimento corrente + modificatore terreno destinazione
```

Quando una truppa attacca:

```text
costo azione += costo attacco corrente
```

## 9.4 Esempio di timeline

Unità in campo:

```text
Cavaliere A: costo azione 2
Guerriero B: costo azione 5
Arciere C: costo azione 6
Generale D: costo azione 7
```

Ordine iniziale:

```text
1. Cavaliere A
2. Guerriero B
3. Arciere C
4. Generale D
```

Il Cavaliere A muove in pianura.

```text
costo movimento corrente: 2
modificatore pianura: 0

nuovo costo azione = 2 + 2 + 0 = 4
```

Nuovo ordine:

```text
Cavaliere A: 4
Guerriero B: 5
Arciere C: 6
Generale D: 7
```

Il Cavaliere agisce ancora.

Muove in foresta.

```text
costo movimento corrente: 2
modificatore foresta per cavalleria: +3

nuovo costo azione = 4 + 2 + 3 = 9
```

Nuovo ordine:

```text
Guerriero B: 5
Arciere C: 6
Generale D: 7
Cavaliere A: 9
```

Questo sistema produce naturalmente unità rapide che agiscono spesso, ma vengono rallentate dai terreni ostili.

---

# 10. Movimento

## 10.1 Regola base

Quando una truppa agisce può muovere di una sola casella.

```text
Una azione movimento = spostamento di 1 casella.
```

## 10.2 Casella di destinazione

Il modificatore di terreno viene applicato alla casella di destinazione.

Regola:

```text
Il costo movimento viene aumentato o diminuito dal terreno in cui l'unità entra.
```

Motivazione:

```text
Ci metti più tempo quando entri in una foresta, in una palude o in montagna.
Non quando ne esci.
```

## 10.3 Formula movimento

```text
costo_azione += costo_movimento_corrente + modificatore_terreno_destinazione
```

Il costo effettivo non dovrebbe mai scendere sotto 1.

```text
costo_effettivo_movimento = max(1, costo_movimento_corrente + modificatore_terreno_destinazione)
```

Quindi:

```text
costo_azione += costo_effettivo_movimento
```

## 10.4 Esempi

Cavaliere:

```text
costo movimento corrente = 2
```

Movimento in pianura:

```text
modificatore pianura = 0
costo effettivo = 2
```

Movimento in foresta:

```text
modificatore foresta = +3
costo effettivo = 5
```

Movimento su strada:

```text
modificatore strada = -1
costo effettivo = 1
```

---

# 11. Attacco

## 11.1 Regola base

Quando una truppa agisce può attaccare un bersaglio visibile entro la propria ampiezza di attacco.

Ogni truppa ha:

```text
ampiezza attacco
```

Non esistono:

```text
min_range
max_range
```

## 11.2 Ampiezza attacco

L'ampiezza attacco indica la massima distanza entro cui una truppa può colpire.

Esempi:

```text
Guerriero: 1
Cavaliere: 1
Arciere: 3
Balestriere: 3
Cecchino: 5
Artiglieria: 6
Missili: 8
```

## 11.3 Ampiezza minima

L'ampiezza minima è sempre:

```text
1
```

Anche se il terreno riduce l'ampiezza, il valore finale non può scendere sotto 1.

```text
ampiezza_attacco_finale = max(1, ampiezza_base + modificatore_terreno)
```

## 11.4 Costo dell'attacco

Quando una truppa attacca:

```text
costo_azione += costo_attacco_corrente
```

Il costo attacco è separato dal costo movimento.

Esempio:

```text
Arciere:
costo movimento = 5
costo attacco = 8

Motivo:
si muove a velocità media, ma richiede tempo per caricare e mirare.
```

## 11.5 Terreno e attacco

Il terreno può modificare l'ampiezza di attacco.

Esempio:

```text
Arciere in pianura:
ampiezza 3

Arciere in foresta:
ampiezza 2

Arciere in montagna:
ampiezza 4
```

Il terreno non aumenta il danno.

Non esistono bonus del tipo:

```text
+1 danno da montagna
+2 danni da fortezza
```

---

# 12. Distanza di Chebyshev

## 12.1 Scelta della distanza

Per attacco e visibilità si usa la distanza di Chebyshev.

Formula:

```text
distanza = max(abs(x1 - x2), abs(y1 - y2))
```

## 12.2 Forma dell'area

La distanza di Chebyshev genera un'area quadrata.

Ampiezza 1:

```text
X X X
X U X
X X X
```

Ampiezza 2:

```text
X X X X X
X X X X X
X X U X X
X X X X X
X X X X X
```

Dove:

```text
U = unità
X = casella inclusa nel range
```

## 12.3 Uso della distanza

La distanza di Chebyshev si usa per:

```text
attacco
visibilità
controllo dei bersagli visibili
comportamento dei ribelli
priorità CPU
```

Il movimento resta di una casella per azione, secondo le caselle adiacenti consentite.

---

# 13. Visibilità e informazioni nascoste

## 13.1 Visibilità reale

La visibilità è una meccanica effettiva.

Ogni truppa ha:

```text
ampiezza visibilità
```

I nemici sono visibili solo se si trovano entro la visibilità di almeno una truppa del giocatore.

Regola:

```text
Una truppa nemica fuori dalla visibilità di tutte le tue unità non viene mostrata sulla mappa.
```

## 13.2 Attacco e visibilità

Per attaccare un nemico, il nemico deve essere visibile.

Quindi una truppa deve soddisfare entrambe le condizioni:

```text
bersaglio entro ampiezza attacco
bersaglio visibile da almeno una propria unità
```

Non è necessario che il bersaglio sia visto dalla stessa unità che attacca, salvo futura scelta di design più restrittiva.

## 13.3 Differenza tra visibilità e attacco

Una truppa può vedere più lontano di quanto attacchi, oppure attaccare quasi quanto vede.

Esempio:

```text
Arciere:
ampiezza visibilità = 4
ampiezza attacco = 3
```

Quindi l'arciere può vedere nemici a distanza 4, ma può attaccare solo fino a distanza 3.

## 13.4 Generale e visibilità

Il generale ha visibilità molto ampia.

Esempio indicativo:

```text
Guerriero: visibilità 3
Arciere: visibilità 4
Cavaliere: visibilità 4
Generale: visibilità 7 o 8
```

## 13.5 Terreno e visibilità

Il terreno può modificare la visibilità.

Esempi:

```text
Foresta: riduce visibilità
Montagna: aumenta visibilità
Collina: aumenta leggermente visibilità
Palude: riduce visibilità
Fortezza: aumenta visibilità
```

Formula:

```text
visibilità_finale = max(1, visibilità_base + modificatore_terreno)
```

---

# 14. Danni

## 14.1 Formula danni

Il danno è calcolato così:

```text
danno = max(1, attacco - difesa)
```

Se la difesa è maggiore o uguale all'attacco, il danno è comunque 1.

Esempi:

```text
Attacco 5, difesa 2:
danno = 3

Attacco 4, difesa 4:
danno = 1

Attacco 3, difesa 6:
danno = 1
```

## 14.2 Nessun bonus terreno al danno

Il terreno non modifica direttamente il danno.

Non esistono:

```text
bonus attacco da terreno
bonus difesa da terreno
```

Il terreno modifica:

```text
movimento
ampiezza attacco
ampiezza visibilità
attraversabilità
```

## 14.3 Niente counter rigidi

Non esistono regole come:

```text
cavalleria forte contro arcieri
lancieri forti contro cavalleria
arcieri forti contro fanteria
```

Le situazioni favorevoli emergono dal posizionamento.

Un cavaliere può essere efficace contro un arciere se riesce ad avvicinarsi. Ma se il cavaliere deve attraversare foreste o viene visto in anticipo, può essere svantaggiato.

---

# 15. Eliminazione dei generali e ribelli

## 15.1 Eliminazione del generale

Quando il generale di un giocatore arriva a 0 HP:

```text
quel giocatore viene eliminato dalla partita
```

## 15.2 Truppe superstiti

Le truppe superstiti del giocatore eliminato diventano ribelli.

## 15.3 Regole dei ribelli

I ribelli:

```text
non appartengono a nessuna fazione
non possono vincere la partita
restano nella timeline di iniziativa
mantengono statistiche e visibilità originali
attaccano chiunque vedano
```

## 15.4 Comportamento dei ribelli

Regola base scelta:

```text
I ribelli attaccano chiunque vedano.
```

Priorità deterministica consigliata:

```text
1. Se vedono uno o più generali, attaccano il generale più vicino.
2. Se ci sono più generali alla stessa distanza, attaccano quello con meno HP.
3. Se non vedono generali, attaccano la truppa visibile più vicina.
4. Se ci sono più truppe alla stessa distanza, attaccano quella con meno HP.
5. Se non vedono nessuno, non attaccano oppure si muovono con logica semplice.
```

## 15.5 Implicazioni strategiche

Eliminare un generale non significa semplicemente rimuovere un esercito dalla mappa.

Le sue truppe diventano pericolose per tutti.

Questo crea decisioni interessanti:

```text
conviene eliminare subito il generale nemico?
oppure conviene prima indebolire le sue truppe?
i ribelli possono ostacolare altri nemici?
posso sfruttare i ribelli come zona pericolosa?
```

---

# 16. CPU e comportamento deterministico

## 16.1 CPU base

Per la prima versione, la CPU deve essere deterministica.

Non serve una IA complessa.

La CPU decide solo per l'unità attiva.

## 16.2 Regole CPU

Priorità consigliata:

```text
1. Se può vedere e attaccare un generale nemico, lo attacca.
2. Se può eliminare una truppa nemica visibile, la attacca.
3. Se può attaccare una truppa nemica visibile, attacca quella con meno HP.
4. Se non può attaccare, si muove verso il generale nemico visibile più vicino.
5. Se non vede generali, si muove verso l'area inesplorata o verso il centro della mappa.
```

## 16.3 CPU e visibilità

Idealmente, la CPU non deve barare.

Regola consigliata:

```text
La CPU conosce solo ciò che le sue truppe possono vedere.
```

Per una versione iniziale può avere una conoscenza approssimativa della direzione generale del nemico, ma non la posizione esatta di unità fuori visibilità.

## 16.4 Sfidanti coerenti con l'era

Gli avversari CPU usano truppe coerenti con l'era corrente del giocatore.

Esempi:

```text
Era Primitiva:
guerrieri tribali, lanciatori primitivi, zattere.

Era Medievale:
balestrieri, cavalieri pesanti, trabocchi.

Era Moderna:
soldati moderni, carri armati, aerei, lanciamissili.

Era Futuristica:
droni, soldati potenziati, robotica, armi a energia.
```

---

# 17. Economia e ricompense

## 17.1 Monete

Partecipando alle battaglie si guadagna denaro sotto forma di monete.

Regole:

```text
Partecipazione: +5 monete
Ogni generale eliminato: +50 monete
Vittoria: +100 monete
```

Formula:

```text
monete = 5 + 50 × generali_eliminati + 100 × vittoria
```

Dove:

```text
vittoria = 1 se il giocatore vince
vittoria = 0 se il giocatore perde
```

## 17.2 Esempi

| Risultato | Calcolo | Monete |
|---|---:|---:|
| Perde senza eliminare generali | 5 | 5 |
| Perde eliminando 1 generale | 5 + 50 | 55 |
| Perde eliminando 2 generali | 5 + 100 | 105 |
| Vince eliminando 1 generale | 5 + 50 + 100 | 155 |
| Vince eliminando 2 generali | 5 + 100 + 100 | 205 |
| Vince eliminando 3 generali | 5 + 150 + 100 | 255 |

Esempio esplicito:

```text
Il giocatore elimina 3 generali e vince.

monete = 5 + 50 × 3 + 100
monete = 255
```

## 17.3 Usi delle monete

Le monete possono essere usate per:

```text
comprare nuove truppe
ricercare tecnologie
acquistare upgrade
evolvere truppe esistenti
```

---

# 18. Esperienza e livelli

## 18.1 Come si guadagna esperienza

Le truppe guadagnano esperienza solo in battaglia.

Più precisamente:

```text
Le truppe guadagnano esperienza in base alle eliminazioni che effettuano.
```

Non basta partecipare.

Non basta essere schierati.

Serve eliminare unità nemiche.

## 18.2 Fattori che determinano l'esperienza

L'esperienza dipende da:

```text
tipo di unità eliminata
livello dell'unità eliminata
era corrente
differenza tecnologica
se l'unità eliminata è un generale
```

## 18.3 Formula concettuale

```text
XP = max(
    1,
    round(
        (XP_base + bonus_livello + bonus_generale)
        × moltiplicatore_era
        × moltiplicatore_differenza_tecnologica
    )
)
```

## 18.4 XP minima

L'esperienza finale non può mai essere inferiore a 1.

Regola:

```text
XP minima per eliminazione = 1
```

## 18.5 Moltiplicatore differenza tecnologica

Il moltiplicatore di differenza tecnologica non deve mai essere negativo.

Esempio:

| Differenza tecnologica | Moltiplicatore |
|---|---:|
| Bersaglio molto inferiore | 0.25 |
| Bersaglio inferiore | 0.50 |
| Stessa fascia | 1.00 |
| Bersaglio superiore | 1.25 |
| Bersaglio molto superiore | 1.50 |

Questa scelta evita risultati negativi.

## 18.6 Moltiplicatore era

Esempio possibile:

| Era | Moltiplicatore XP |
|---|---:|
| Primitiva | 1.0 |
| Antica | 1.2 |
| Classica | 1.3 |
| Medievale | 1.4 |
| Rinascimentale | 1.5 |
| Industriale | 1.6 |
| Moderna | 1.8 |
| Futuristica | 2.0 |

## 18.7 Bonus generale

Eliminare un generale dà un bonus esperienza significativo.

Esempio:

```text
bonus_generale = +30 XP
```

## 18.8 Esempio completo

Un arciere elimina un generale di livello 3 in Era Medievale.

```text
XP_base_generale = 20
bonus_livello = 2 × (3 - 1) = 4
bonus_generale = 30
moltiplicatore_era = 1.4
moltiplicatore_differenza_tecnologica = 1.0

XP = round((20 + 4 + 30) × 1.4 × 1.0)
XP = round(75.6)
XP = 76
```

## 18.9 Livelli

Quando una truppa sale di livello, ottiene benefici alle statistiche.

Possibili benefici:

```text
+1 HP
+1 attacco
+1 difesa
-1 costo movimento corrente
-1 costo attacco corrente
+1 ampiezza visibilità
```

La crescita deve essere bilanciata per classe.

Esempi:

```text
Fanteria:
più probabilità di HP, difesa, attacco.

Tiro a distanza:
più probabilità di attacco, visibilità, riduzione costo attacco.

Cavalleria:
più probabilità di HP, attacco, riduzione costo movimento.

Esploratori:
più probabilità di visibilità e movimento.

Artiglieria:
più probabilità di attacco e ampiezza, ma costo attacco resta alto.
```

## 18.10 Nessun permadeath

Per ora non esiste permadeath.

Regola:

```text
Le truppe sconfitte tornano disponibili dopo la partita.
```

Il permadeath può essere riservato a una futura modalità competitiva, hardcore o multiplayer.

---

# 19. Progressione permanente

Fuori dalla battaglia, il giocatore gestisce la propria progressione.

Può usare le monete per:

```text
comprare truppe
ricercare tecnologie
comprare upgrade
evolvere truppe compatibili
```

## 19.1 Roster

Le truppe vengono acquistate singolarmente.

Esempio:

```text
Arciere #1
Arciere #2
Guerriero #1
Cavaliere #1
```

Ogni truppa può avere livello ed esperienza propri.

## 19.2 Upgrade

Gli upgrade sono separati dall'esperienza.

L'esperienza migliora la singola unità.

Gli upgrade possono migliorare classi, rami o sistemi.

Esempi:

```text
Addestramento al tiro:
migliora le unità da tiro.

Armature leggere:
migliora la fanteria.

Logistica:
aumenta budget o riduce costi.

Navigazione:
migliora unità navali.

Ricognizione:
aumenta visibilità.
```

## 19.3 Tecnologie

Le tecnologie sono organizzate in un DAG per ere.

Il giocatore può ricercare solo tecnologie dell'era corrente.

Quando tutte le tecnologie dell'era corrente sono completate:

```text
il passaggio all'era successiva è automatico
```

---

# 20. Ere tecnologiche

## 20.1 Proprietà globale

L'era tecnologica è una proprietà globale del giocatore.

Non è una proprietà individuale delle truppe.

L'era determina:

```text
dimensione della mappa
budget di schieramento
tecnologie ricercabili
truppe acquistabili
evoluzioni disponibili
tipo di sfidanti CPU
complessità delle mappe
versione automatica del generale
```

## 20.2 Passaggio automatico

Il passaggio di era è automatico.

Regola:

```text
Quando tutte le tecnologie dell'era corrente sono ricercate, il giocatore passa alla prossima era.
```

## 20.3 Effetti del passaggio era

Quando il giocatore passa di era:

```text
aumenta la dimensione della mappa
aumenta il budget di schieramento
si aggiorna automaticamente il generale
si sbloccano nuove tecnologie
si sbloccano nuove truppe
si sbloccano nuove evoluzioni
le CPU usano truppe della nuova era
possono comparire mappe più complesse
```

---

# 21. DAG tecnologico completo

Legenda:

```text
A <- B
```

significa:

```text
A richiede B
```

```text
A <- B + C
```

significa:

```text
A richiede sia B sia C
```

Nota importante:

```text
Le tecnologie di comando non sbloccano il generale.
Il generale è sempre disponibile e si aggiorna automaticamente con l'era.
Le tecnologie di comando possono sbloccare upgrade tattici, logistici o di visibilità.
```

---

## 21.1 Era Primitiva

```text
Fuoco
Pietra Scheggiata
Raccolta Organizzata

Caccia <- Pietra Scheggiata
Pelle e Ossa <- Caccia

Linguaggio Tribale <- Fuoco

Ruota <- Pietra Scheggiata
Addomesticamento <- Caccia
Zattere <- Raccolta Organizzata

Tattiche di Branco <- Linguaggio Tribale + Caccia
Sentieri <- Ruota + Addomesticamento

Villaggio <- Fuoco + Linguaggio Tribale + Raccolta Organizzata
```

### Sblocchi principali

```text
Pietra Scheggiata → Guerriero Tribale
Caccia → Lanciatore Primitivo
Addomesticamento → Esploratore Tribale
Zattere → Navigatore Primitivo
Pelle e Ossa → upgrade HP truppe primitive
Sentieri → bonus movimento su alcuni terreni
Villaggio → acquisto stabile truppe primitive
```

### Generale

```text
Generale Tribale disponibile automaticamente dall'inizio.
```

---

## 21.2 Era Antica

```text
Agricoltura
Metallurgia del Rame
Scrittura
Arco Semplice
Allevamento
Navigazione Costiera

Ceramica <- Agricoltura

Armi di Rame <- Metallurgia del Rame

Burocrazia <- Scrittura + Agricoltura

Carri <- Ruota + Allevamento

Mura Primitive <- Ceramica + Metallurgia del Rame

Formazioni Militari <- Scrittura + Armi di Rame

Commercio <- Ceramica + Navigazione Costiera

Comando Organizzato <- Burocrazia + Formazioni Militari
```

### Sblocchi principali

```text
Armi di Rame → Guerriero di Rame
Arco Semplice → Arciere Antico
Allevamento → Cavalleria Leggera
Carri → Carro da Guerra
Navigazione Costiera → Barca da Guerra
Mura Primitive → più fortezze/base nelle mappe
Formazioni Militari → upgrade fanteria
Burocrazia → bonus budget schieramento
Commercio → bonus economici
Comando Organizzato → upgrade tattici/comando, non generale
```

### Generale

```text
Passaggio automatico a Generale Antico quando entri nell'Era Antica.
```

---

## 21.3 Era Classica

```text
Ferro
Matematica

Armi di Ferro <- Ferro

Armature di Ferro <- Ferro + Formazioni Militari

Ingegneria <- Matematica + Ferro

Catapulte <- Ingegneria

Strade Militari <- Ingegneria + Burocrazia

Tattiche di Fanteria <- Armi di Ferro + Comando Organizzato

Cavalleria Organizzata <- Allevamento + Armi di Ferro

Arco Composito <- Arco Semplice + Matematica

Porti <- Navigazione Costiera + Commercio

Triremi <- Porti + Armi di Ferro

Codici Militari <- Tattiche di Fanteria + Burocrazia

Medicina Empirica <- Agricoltura + Scrittura
```

### Sblocchi principali

```text
Armi di Ferro → Guerriero di Ferro
Armature di Ferro → Fanteria Corazzata
Catapulte → Catapulta
Strade Militari → più strade e bonus movimento
Cavalleria Organizzata → Cavaliere Classico
Arco Composito → Arciere Composito
Porti → infrastrutture navali
Triremi → Trireme
Codici Militari → upgrade comando/tattica, non generale
Medicina Empirica → upgrade recupero/gestione truppe
```

### Generale

```text
Passaggio automatico a Generale Classico quando entri nell'Era Classica.
```

---

## 21.4 Era Medievale

```text
Acciaio Primordiale

Armi d'Acciaio <- Acciaio Primordiale

Cavalleria Pesante <- Cavalleria Organizzata + Armi d'Acciaio

Staffe e Selle <- Cavalleria Organizzata

Balestra <- Arco Composito + Ingegneria

Castelli <- Mura Primitive + Ingegneria

Assedio Avanzato <- Catapulte + Castelli

Gilde Artigiane <- Commercio + Acciaio Primordiale

Cartografia Medievale <- Matematica + Porti

Navigazione d'Altura <- Porti + Cartografia Medievale

Disciplina Feudale <- Codici Militari + Castelli

Logistica Feudale <- Strade Militari + Gilde Artigiane

Medicina Monastica <- Medicina Empirica + Gilde Artigiane

Fortificazioni Costiere <- Castelli + Navigazione d'Altura
```

### Sblocchi principali

```text
Armi d'Acciaio → Fanteria d'Acciaio
Cavalleria Pesante → Cavaliere Pesante
Staffe e Selle → bonus movimento cavalleria
Balestra → Balestriere
Castelli → fortezze evolute
Assedio Avanzato → Trabocco
Gilde Artigiane → sconti upgrade/evoluzioni
Cartografia Medievale → bonus visibilità/mappa
Navigazione d'Altura → Nave Medievale
Disciplina Feudale → upgrade comando/tattica, non generale
Logistica Feudale → bonus budget
Medicina Monastica → upgrade recupero
Fortificazioni Costiere → mappe costiere più complesse
```

### Generale

```text
Passaggio automatico a Generale Medievale quando entri nell'Era Medievale.
```

---

## 21.5 Era Rinascimentale

```text
Polvere da Sparo

Archibugi <- Polvere da Sparo

Cannoni <- Polvere da Sparo + Assedio Avanzato

Eserciti Professionali <- Disciplina Feudale + Logistica Feudale

Cartografia Globale <- Cartografia Medievale + Navigazione d'Altura

Caravelle <- Cartografia Globale + Navigazione d'Altura

Banche <- Commercio + Gilde Artigiane

Scienza Sperimentale <- Matematica + Banche

Fortificazioni a Stella <- Castelli + Cannoni

Addestramento al Tiro <- Archibugi + Eserciti Professionali

Cavalleria con Armi da Fuoco <- Cavalleria Pesante + Archibugi

Medicina Rinascimentale <- Medicina Monastica + Scienza Sperimentale

Logistica Commerciale <- Banche + Cartografia Globale

Navi da Guerra a Vela <- Caravelle + Cannoni
```

### Sblocchi principali

```text
Archibugi → Archibugiere / Tiratore ad Archibugio secondo ramo
Cannoni → Cannone
Eserciti Professionali → upgrade comando/tattica, non generale
Cartografia Globale → mappe con più mare/coste
Caravelle → Caravella
Banche → bonus economia
Scienza Sperimentale → prerequisito era industriale
Fortificazioni a Stella → fortezze anti-cannone
Addestramento al Tiro → upgrade unità da tiro
Cavalleria con Armi da Fuoco → Dragone
Medicina Rinascimentale → upgrade recupero/HP
Logistica Commerciale → bonus budget/economia
Navi da Guerra a Vela → Galeone
```

### Generale

```text
Passaggio automatico a Generale Rinascimentale quando entri nell'Era Rinascimentale.
```

---

## 21.6 Era Industriale

```text
Meccanica <- Scienza Sperimentale

Motore a Vapore <- Meccanica

Produzione di Massa <- Meccanica + Banche

Ferrovie <- Motore a Vapore + Logistica Commerciale

Fucili a Retrocarica <- Archibugi + Produzione di Massa

Artiglieria Moderna <- Cannoni + Produzione di Massa

Corazzate <- Navi da Guerra a Vela + Motore a Vapore

Telegrafo <- Ferrovie + Scienza Sperimentale

Medicina Industriale <- Medicina Rinascimentale + Produzione di Massa

Logistica Industriale <- Ferrovie + Telegrafo

Motore a Combustione <- Meccanica + Produzione di Massa

Ricognizione Organizzata <- Telegrafo + Cartografia Globale

Stato Maggiore <- Eserciti Professionali + Telegrafo

Mitragliatrici Primitive <- Fucili a Retrocarica + Produzione di Massa
```

### Sblocchi principali

```text
Motore a Vapore → logistica industriale
Produzione di Massa → riduzione costo truppe/evoluzioni
Ferrovie → mappe con infrastrutture rapide
Fucili a Retrocarica → Fuciliere
Artiglieria Moderna → Artiglieria
Corazzate → Corazzata
Telegrafo → bonus coordinamento/visibilità strategica
Medicina Industriale → upgrade recupero/HP
Logistica Industriale → bonus budget
Motore a Combustione → prerequisito mezzi moderni
Ricognizione Organizzata → Esploratore Moderno / Tiratore Industriale
Stato Maggiore → upgrade comando/tattica, non generale
Mitragliatrici Primitive → Mitragliere
```

### Generale

```text
Passaggio automatico a Generale Industriale quando entri nell'Era Industriale.
```

---

## 21.7 Era Moderna

```text
Radio <- Telegrafo

Motorizzazione <- Motore a Combustione

Blindati <- Motorizzazione + Produzione di Massa

Carri Armati <- Blindati + Artiglieria Moderna

Aviazione <- Motore a Combustione + Scienza Sperimentale

Aerei da Combattimento <- Aviazione + Mitragliatrici Primitive

Bombardamento <- Aviazione + Artiglieria Moderna

Radar <- Radio + Aviazione

Missilistica <- Radar + Bombardamento

Fanteria Moderna <- Fucili a Retrocarica + Radio

Cecchini <- Fanteria Moderna + Ricognizione Organizzata

Portaerei <- Corazzate + Aviazione

Medicina da Campo Moderna <- Medicina Industriale + Radio

Comando Integrato <- Radio + Stato Maggiore

Logistica Meccanizzata <- Motorizzazione + Radio
```

### Sblocchi principali

```text
Radio → comunicazioni moderne
Motorizzazione → Fanteria Motorizzata
Blindati → Autoblindo
Carri Armati → Carro Armato
Aviazione → Aereo da Ricognizione
Aerei da Combattimento → Caccia
Bombardamento → Bombardiere
Radar → bonus visibilità/anti-aereo
Missilistica → Lanciamissili
Fanteria Moderna → Soldato Moderno
Cecchini → Cecchino
Portaerei → Portaerei
Medicina da Campo Moderna → upgrade recupero
Comando Integrato → upgrade comando/tattica, non generale
Logistica Meccanizzata → bonus budget
```

### Generale

```text
Passaggio automatico a Generale Moderno quando entri nell'Era Moderna.
```

---

## 21.8 Era Futuristica

```text
Informatica Avanzata <- Radio + Radar

Intelligenza Artificiale <- Informatica Avanzata

Droni Militari <- Intelligenza Artificiale + Aviazione

Robotica <- Informatica Avanzata + Motorizzazione

Nanotecnologie <- Informatica Avanzata + Medicina da Campo Moderna

Sciami di Droni <- Droni Militari + Produzione di Massa

Esoscheletri <- Robotica + Medicina da Campo Moderna

Materiali Intelligenti <- Nanotecnologie + Robotica

Stealth Avanzato <- Materiali Intelligenti + Radar

Energia Avanzata <- Informatica Avanzata + Materiali Intelligenti

Armi a Energia <- Energia Avanzata + Materiali Intelligenti

Missili Intelligenti <- Missilistica + Intelligenza Artificiale

Guerra Orbitale <- Energia Avanzata + Missili Intelligenti

Comando Autonomo <- Intelligenza Artificiale + Comando Integrato

Logistica Autonoma <- Robotica + Sciami di Droni

Nanoriparazione <- Nanotecnologie + Logistica Autonoma

Dominio Sensoriale <- Stealth Avanzato + Droni Militari
```

### Sblocchi principali

```text
Informatica Avanzata → prerequisito sistemi futuri
Intelligenza Artificiale → Droni Base
Droni Militari → Drone da Ricognizione
Robotica → Unità Robotica
Nanotecnologie → Nanomedicina / Nano-sciami
Sciami di Droni → Sciame Drone
Esoscheletri → Soldato Potenziato
Materiali Intelligenti → upgrade HP/difesa avanzati
Stealth Avanzato → Cecchino Stealth / Unità Stealth
Energia Avanzata → prerequisito armi avanzate
Armi a Energia → Fanteria a Energia
Missili Intelligenti → Lanciamissili Intelligente
Guerra Orbitale → Batteria Orbitale
Comando Autonomo → upgrade comando/tattica, non generale
Logistica Autonoma → budget massimo / supporto automatizzato
Nanoriparazione → upgrade recupero avanzato
Dominio Sensoriale → massimo bonus visibilità
```

### Generale

```text
Passaggio automatico a Generale Futuristico quando entri nell'Era Futuristica.
```

---

# 22. Evoluzione delle truppe

## 22.1 Modello scelto

Il gioco usa il modello C:

```text
Quando una nuova unità avanzata viene sbloccata, puoi:
1. comprarla nuova;
2. oppure evolvere una truppa esistente compatibile.
```

## 22.2 Vantaggi dell'evoluzione

Evolvere una truppa veterana può:

```text
mantenere parte del livello
mantenere parte dell'esperienza
costare meno rispetto all'acquisto di una nuova unità avanzata
premiare l'uso continuativo della stessa truppa
```

## 22.3 Requisiti possibili

Una evoluzione può richiedere:

```text
tecnologia specifica
monete
livello minimo della truppa
classe compatibile
```

Esempio:

```text
Arciere Composito → Balestriere

Richiede:
Balestra
Arciere Composito livello almeno 2
Costo evoluzione: monete
```

## 22.4 Nessuna duplicazione tra rami

Le catene evolutive devono essere disgiunte.

Non deve accadere che due rami diversi evolvano nella stessa unità, perché questo crea ambiguità su:

```text
upgrade applicabili
classe di appartenenza
progressione
bilanciamento
interfaccia del roster
```

È stato quindi corretto il problema per cui sia fanteria sia tiro a distanza potevano evolvere in Moschettiere e Fuciliere.

---

# 23. Catene evolutive corrette

## 23.1 Generale

```text
Generale Tribale
→ Generale Antico
→ Generale Classico
→ Generale Medievale
→ Generale Rinascimentale
→ Generale Industriale
→ Generale Moderno
→ Generale Futuristico
```

Nota:

```text
Questa catena è automatica con il passaggio d'era.
Non richiede acquisto, tecnologia specifica o evoluzione manuale.
```

---

## 23.2 Fanteria da mischia / fanteria principale

```text
Guerriero Tribale
→ Guerriero Primitivo
→ Guerriero di Rame
→ Guerriero di Ferro
→ Fanteria Corazzata
→ Fanteria d'Acciaio
→ Picchiere
→ Fante di Linea
→ Fante Industriale
→ Soldato Moderno
→ Soldato Potenziato
→ Fanteria a Energia
```

Caratteristiche del ramo:

```text
robusta
versatile
buoni HP
buona difesa
ampiezza attacco generalmente bassa o media
adatta al controllo del territorio
```

---

## 23.3 Tiro a distanza / precisione

```text
Lanciatore Primitivo
→ Fromboliere
→ Arciere Antico
→ Arciere Composito
→ Balestriere
→ Tiratore ad Archibugio
→ Tiratore Scelto
→ Tiratore Industriale
→ Cecchino
→ Cecchino Stealth
→ Operatore di Precisione Orbitale
```

Caratteristiche del ramo:

```text
alta ampiezza attacco
buona o alta visibilità
HP inferiori
difesa inferiore
costo attacco più alto
posizionamento fondamentale
```

Questo ramo non evolve più in Moschettiere o Fuciliere.

---

## 23.4 Fanteria da fuoco

Ramo separato, introdotto con le armi da fuoco.

```text
Archibugiere
→ Moschettiere
→ Fuciliere
→ Fanteria Meccanizzata
→ Soldato d'Assalto
→ Soldato Esoscheletro
```

Caratteristiche del ramo:

```text
attacco medio-alto
ampiezza attacco media
più resistente del ramo precisione
meno visibilità del ramo precisione
più offensiva della fanteria principale
```

---

## 23.5 Mobilità terrestre

```text
Esploratore Tribale
→ Esploratore
→ Ricognitore
→ Cavalleria Leggera
→ Cavaliere Classico
→ Cavaliere Pesante
→ Dragone
→ Ricognitore Motorizzato
→ Autoblindo
→ Carro Armato
→ Unità Robotica Pesante
```

Caratteristiche del ramo:

```text
alta mobilità
buona pressione tattica
utile per esplorazione e manovre
molto dipendente dal terreno
```

---

## 23.6 Navale

```text
Navigatore Primitivo
→ Barca Rudimentale
→ Barca da Guerra
→ Trireme
→ Nave Medievale
→ Caravella
→ Galeone
→ Corazzata
→ Portaerei
→ Nave Autonoma
```

Caratteristiche del ramo:

```text
controllo del mare
utilità su mappe costiere o arcipelaghi
può diventare fondamentale nelle ere avanzate
```

---

## 23.7 Assedio / artiglieria

```text
Catapulta
→ Trabocco
→ Cannone
→ Artiglieria
→ Lanciamissili
→ Lanciamissili Intelligente
→ Batteria Orbitale
```

Caratteristiche del ramo:

```text
ampiezza attacco molto alta
costo attacco alto
movimento basso
forte controllo di zona
vulnerabile se esposta
```

Nota:

```text
Fromboliere resta nel ramo tiro a distanza.
Non evolve in Catapulta.
```

Questo evita un'ulteriore sovrapposizione tra rami.

---

## 23.8 Aereo / droni

```text
Aereo da Ricognizione
→ Caccia
→ Bombardiere
→ Drone da Ricognizione
→ Drone da Combattimento
→ Sciame Drone
```

Caratteristiche del ramo:

```text
alta mobilità
interazione particolare col terreno
alta visibilità
potenziale attacco ad ampia distanza
forte nelle ere moderne e futuristiche
```

---

# 24. Regole consolidate

Questa è la versione compatta del regolamento finale discusso finora.

```text
La mappa viene generata casualmente.
La dimensione della mappa dipende dall'era tecnologica globale del giocatore.
Dopo la generazione vengono individuati territori iniziali plausibili.
Il giocatore sceglie il territorio.
Solo dopo sceglie le truppe da schierare.
Tutti i giocatori hanno lo stesso budget di schieramento.
Il generale è sempre disponibile.
Il generale si aggiorna automaticamente con il passaggio d'era.
Le CPU usano truppe coerenti con l'era tecnologica del giocatore.
La ricerca tecnologica è un DAG.
Il giocatore può ricercare solo tecnologie dell'era corrente.
Quando tutte le tecnologie dell'era corrente sono ricercate, il passaggio d'era è automatico.
Ogni unità ha costo movimento corrente e costo attacco corrente.
A inizio partita, costo azione partita = costo movimento corrente.
Agisce sempre l'unità con costo azione partita minore.
Ogni azione è movimento di una casella oppure attacco.
Quando una unità muove, il costo azione aumenta linearmente.
Il modificatore movimento dipende dalla casella di destinazione.
Quando una unità attacca, il costo azione aumenta del costo attacco corrente.
L'attacco usa ampiezza attacco intera.
La distanza per attacco e visibilità è Chebyshev.
Ampiezza attacco minima = 1.
Il terreno modifica movimento, ampiezza attacco e ampiezza visibilità.
Il terreno non modifica direttamente il danno.
Il danno è max(1, attacco - difesa).
Non esistono counter rigidi.
Non esistono ruoli meccanici obbligatori.
I nemici fuori dalla visibilità delle proprie truppe non sono mostrati.
Il generale ha ampiezza visibilità molto alta.
Il generale sconfitto elimina il giocatore.
Le truppe del giocatore eliminato diventano ribelli.
I ribelli attaccano chiunque vedano.
Le truppe sconfitte tornano disponibili dopo la partita.
Non c'è permadeath nella modalità base.
Le truppe guadagnano esperienza solo tramite eliminazioni.
L'esperienza dipende da tipo unità eliminata, livello, era, differenza tecnologica e generale eliminato.
L'esperienza finale non può mai essere inferiore a 1.
Il denaro si guadagna tramite partecipazione, generali eliminati e vittoria.
Le truppe possono essere comprate nuove oppure evolute da truppe esistenti compatibili.
Le catene evolutive devono evitare duplicazioni tra rami.
```

---

# 25. Controlli

I controlli potranno cambiare durante lo sviluppo, ma una configurazione base consigliata è:

| Azione | Controllo |
|---|---|
| Selezionare casella o unità | Click sinistro |
| Confermare movimento | Click sinistro su casella valida |
| Confermare attacco | Click sinistro su bersaglio valido |
| Annullare selezione | Click destro |
| Aprire/chiudere pannello unità | Tab |
| Attesa/skip dell'unità attiva | Spazio |
| Muovere camera | WASD o frecce direzionali |
| Zoom avanti/indietro | Rotella mouse |
| Mostrare range movimento | M |
| Mostrare range attacco | A |
| Mostrare range visibilità | V |
| Mostrare tutti i range dell'unità | R |
| Aprire menu pausa | Esc |

## 25.1 Loop operativo in battaglia

Il ciclo base del giocatore è:

```text
1. L'unità attiva viene evidenziata.
2. Il giocatore vede le azioni disponibili.
3. Il giocatore può selezionare movimento o attacco.
4. Se sceglie movimento, vengono mostrate le caselle adiacenti valide.
5. Se sceglie attacco, vengono mostrati i bersagli visibili entro ampiezza attacco.
6. Il giocatore conferma.
7. Il costo azione dell'unità viene aggiornato.
8. La timeline viene riordinata.
9. La prossima unità con costo azione minore diventa attiva.
```

---

# Stato del design

Questo README descrive il regolamento e il game design attualmente consolidati.

Il progetto è pensato per essere implementato in Pygame, ma questo documento si concentra sulle dinamiche di gioco e non sull'architettura tecnica.

La priorità di sviluppo consigliata è:

```text
1. Griglia e generazione mappa semplice.
2. Terreni base.
3. Schieramento con budget.
4. Timeline delle unità.
5. Movimento di una casella.
6. Attacco con distanza Chebyshev.
7. Visibilità reale.
8. Generale e condizione di vittoria.
9. Ribelli.
10. Ricompense.
11. Esperienza.
12. Progressione tecnologica.
13. Evoluzioni.
```
