# 🏃‍♂️ GPX Runner Analytics & Strava Race Predictor

Una web application **100% client-side** (zero backend, zero tracciamento) per analizzare tracce GPS di corsa (`.gpx`) e prevedere i tempi gara su **5 km, 10 km, Mezza Maratona, Maratona** e distanze personalizzate partendo dall'archivio esportato da **Strava** (`.zip` o file `.tcx`/`.gpx`).

> ⚡ **Nota sullo sviluppo (Vibe Coded Project):**  
> Questa dashboard è stata realizzata in modalità *vibe coding* — combinando architettura software, dominio sportivo/fisiologico della corsa e iterazione assistita da AI per calibrare gli algoritmi su dati reali di allenamento e gara.

---

## ✨ Funzionalità Principali

### 1. 📊 Analisi Singolo GPX (`Modalità 1`)
* **Riepilogo Istantaneo & Tracciato GPS:** Calcolo di distanza, tempo totale, passo medio, dislivello positivo smussato ($D+$) e visualizzazione mappa interattiva su Leaflet/OpenStreetMap.
* **Taglio Inizio/Fine (Trimming):** Slider interattivi per tagliare riscaldamento iniziale, soste al semaforo o passeggiata finale per distanza o per tempo.
* **Split Regolari & Costruttore Fartlek:** Analisi per frazioni fisse (`1 km`, `500 m`, `400 m`, `1 min`, ecc.) oppure costruzione di allenamenti misti (es. *2 km Riscaldamento + 6×(1' Veloce + 1' Recupero) + Defaticamento*).
* **Indice di Oscillazione Interna ($\pm\%$):** Misura la regolarità del passo *all'interno* di ciascun singolo blocco, distinguendo una ripetuta corsa a ritmo costante da una corsa a strappi.

### 2. 🏆 Proiezioni Gara da Archivio Strava (`Modalità 2`)
* **Lettura Diretta dell'Export `.zip` di Strava:** Estrazione in-memory nel browser tramite `JSZip` e `pako` (decompressione automatica di `activities.csv` e delle tracce `.tcx.gz` / `.gpx.gz`).
* **Separazione Multi-Sport & Cross-Training:** Scarta automaticamente gli altri sport dal calcolo del passo gara, ma accredita il volume aerobico del **Calcio / Cross-Training (pesato al 65%)** e dei riscaldamenti brevi ($< 2\text{ km}$) nel calcolo della tenuta muscolare sulle lunghe distanze.
* **Rilevatore Ciclico di Fartlek & Segmenti Continui:** Scansiona le tracce secondo per secondo per estrarre sia i migliori segmenti continui ($\ge 1\text{ km}$) sia gli allenamenti a intervalli brevi (tramite la mediana delle ripetute distinte), evitando che il riscaldamento o i recuperi lenti abbassino la stima del potenziale aerobico.

---

## 🧮 Modello Matematico e Fisiologico

Il motore predittivo combina quattro modelli classici della letteratura dell'endurance per evitare sia le stime troppo pessimistiche (causate dalla media con i fondi lenti) sia i falsi positivi (causati da picchi GPS isolati).

### 1. Smussamento Altimetrico e GAP (Grade Adjusted Pace)
L'altitudine GPS grezza viene filtrata con una media mobile a 7 punti e una soglia di isteresi di $3.5\text{ m}$ per eliminare il rumore satellitare in pianura. Il passo viene poi convertito in **GAP** (passo equivalente in pianura) secondo il modello metabolico di Minetti (~$3.3\text{ s/km}$ per ogni $10\text{ m}$ di $D+$ al km):

$$\text{GAP}_{\text{sec/km}} = \text{Passo}_{\text{grezzo}} - \left( \frac{D+_{\text{totale}}}{D_{\text{km}} \cdot 10} \right) \times 3.3$$

### 2. Stima della Cilindrata Aerobica ($\text{VDOT}$ di Daniels & Gilbert)
Per ogni prestazione di distanza $D$ (in metri) e tempo equivalente in piano $T$ (in minuti), con velocità $v = D / T$ (in $\text{m/min}$), il consumo di ossigeno teorico ($\text{VO}_2$) e la frazione sostenibile ($\text{\%VO}_2\max$) sono calcolati tramite le equazioni di Daniels & Gilbert:

$$\text{VO}_2(v) = -4.60 + 0.182258 \cdot v + 0.000104 \cdot v^2$$

$$\%\text{VO}_2\max(T) = 0.8 + 0.1894393 \cdot e^{-0.012778 \cdot T} + 0.2989558 \cdot e^{-0.1932605 \cdot T}$$

$$\text{VDOT} = \frac{\text{VO}_2(v)}{\%\text{VO}_2\max(T)}$$

*(Se sono presenti i dati di frequenza cardiaca media $\text{FC}_{\text{media}}$ rispetto a $\text{FC}_{\max}$, gli sforzi sub-massimali corsi tra il $65\%$ e il $91\%$ della $\text{FC}_{\max}$ ricevono un accredito proporzionale alla riserva cardiaca).*

### 3. Rilevamento Ciclico Fartlek (Mediana delle Ripetute) e Bonus Costanza
Per evitare che gli allenamenti a intervalli vengano penalizzati dalle fasi di recupero lento:
* La traccia viene suddivisa in finestre consecutive di $30\text{ s}$ basate sulla distanza geodetica continua (formula dell'emisenoverso / *Haversine*).
* Una sessione viene riconosciuta come **Fartlek** solo se presenta $\ge 3$ cicli distinti di spinta/recupero in cui i blocchi veloci sono almeno il $12\%$ più rapidi della mediana dell'uscita e adiacenti a blocchi di recupero almeno il $30\%$ più lenti.
* Il passo delle ripetute viene estratto tramite la **mediana** dei blocchi veloci (robusta agli outlier del GPS) e convertito in passo $10\text{ km}$ equivalente.
* L'indice $\text{VDOT}_{\text{eff}}$ finale è ancorato al **Picco Base Certificato** (con decadimento temporale oltre i 21 giorni) potenziato da un **Bonus Costanza** (fino a $+1.5\%$) derivato dai migliori allenamenti di qualità e dal volume settimanale.

### 4. Proiezione sulle Distanze con Esponente Dinamico di Riegel
Dal tempo base sui $10\text{ km}$ ($T_{10\text{k}}$, ricavato invertendo numericamente per bisezione la funzione $\text{VDOT}$), il tempo $T_2$ su una distanza target $D_2$ viene calcolato con la legge di potenza di Peter Riegel:

$$T_2 = T_{10\text{k}} \cdot \left( \frac{D_2}{10000} \right)^k$$

dove l'**esponente di fatica $k$** non è fisso al valore standard $1.06$ (valido solo per atleti d'élite ad alto chilometraggio), ma viene adattato dinamicamente in base al **Volume Settimanale Equivalente** ($V_{\text{eq}} = \text{km}_{\text{corsa}} + 0.65 \cdot \text{km}_{\text{calcio/cross}}$):

$$k = \text{clamp}\Big( 1.06 + \max\big(0,\; (45 - V_{\text{eq}}) \cdot 0.0005\big),\; 1.055,\; 1.090 \Big)$$

Per le distanze superiori ai $15\text{ km}$ (Mezza Maratona e Maratona), se il lungo massimo recente ($L_{\max}$) è inferiore al $68\%$ della distanza di gara, viene applicato un ulteriore coefficiente prudenziale di tenuta muscolare.

---

## 🔒 Privacy

L'applicazione non utilizza alcun server di backend: l'intero parsing dei file `.zip`, `.csv`, `.gpx` e `.tcx` avviene localmente nel browser dell'utente. Nessun file viene caricato in rete.