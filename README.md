# Heart Disease: EDA e Data Cleaning

Pulizia ed analisi esplorativa del dataset *Heart Disease*, derivato dallo studio di Cleveland.
L'obiettivo è capire quanto sono affidabili i dati, correggere le anomalie del campione e vedere quali variabili fisiologiche si associano di più alla variabile target.

---

## Perché questo progetto

Molti notebook su Kaggle applicano subito un modello al dataset grezzo (1025 record). Basta però un controllo preliminare per accorgersi che il dataset è pieno di righe duplicate: i pazienti unici sono molti meno. Prima di esplorare o modellare, quindi, i dati vanno ripuliti.

## Cosa ho fatto

1. **Ispezione iniziale**: dimensioni, tipi di dato, valori mancanti (nessuno).
2. **Deduplicazione**: rimosse 723 righe duplicate, il campione scende a 302 pazienti unici.
3. **Outlier**: identificati con il metodo IQR su colesterolo e pressione sistolica, poi valutati caso per caso.
4. **Analisi bivariata e correlazioni**: quali variabili separano meglio i due gruppi del target.
5. **Export**: salvataggio del dataset pulito, pronto per la fase di modellazione.

## Risultati principali

Dalla matrice di correlazione di Pearson e dai confronti tra i due gruppi emerge questo (le correlazioni sono rispetto a `target`, dove 1 è la classe positiva del dataset):

- **`exang` (angina da sforzo), r = -0.44**: chi non ha angina da sforzo ha molto più spesso `target = 1`.
- **`oldpeak` (depressione del tratto ST), r = -0.43**: mediana 0.0 nel gruppo `target = 1`, circa 1.0 nell'altro.
- **`thalach` (frequenza cardiaca massima), r = +0.42**: mediana di circa 153-155 bpm nel gruppo `target = 1`, circa 142 bpm nell'altro.
- **`cp` (tipo di dolore toracico), r = +0.43**: i dolori atipici e non anginosi sono più frequenti nel gruppo `target = 1` rispetto al dolore tipico.

Colesterolo (`chol`, r = -0.08) e glicemia a digiuno (`fbs`, r = -0.03) non mostrano una correlazione lineare utile con il target. Le variabili legate alla risposta allo sforzo sembrano quindi più informative di un singolo valore ematico, almeno in questo campione di 302 persone.

**Outlier.** Li ho tenuti: pressione fino a 200 mmHg e colesterolo fino a 564 mg/dl sono valori estremi ma plausibili dal punto di vista clinico, non sembrano errori di inserimento.

## Limiti

- Campione piccolo (302 pazienti), da una sola popolazione.
- Pearson misura solo relazioni lineari e tratta come numeriche anche variabili categoriche come `cp`.
- Correlazione non è causalità: nessuna delle associazioni qui sopra va letta come un effetto.

## Struttura del repository

```text
heart-disease-eda/
├── data/
│   ├── heart.csv                  # dataset grezzo (Kaggle)
│   └── heart_cleaned.csv          # dataset pulito, 302 record unici
├── notebooks/
│   └── 01_eda_and_cleaning.ipynb  # pulizia e visualizzazioni
├── .gitignore                     # esclude .idea e virtualenv
├── README.md
└── requirements.txt               # dipendenze
```
## Setup e Riproducibilità

Clonare la repository:
```bash
git clone https://github.com/JCicconi-31/heart-disease-eda.git
cd heart-disease-eda
```

Per windows: 
```bash
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Per macOS/Linux
```bash
python -m venv .venv
source .venv/bin/activate
```

Installare le dipendenze:
```bash
pip install -r requirements.txt
```

Eseguire l'analisi: Aprire ed eseguire il notebook ```notebooks/01_eda_and_cleaning.ipynb``` con PyCharm o Jupyter
