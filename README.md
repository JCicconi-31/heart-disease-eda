# Heart Disease: EDA, Data Cleaning & Classification Modeling

Pipeline completa di pulizia dati, analisi esplorativa e modellazione predittiva basata sul dataset clinico *Heart Disease* (studio di Cleveland).  
L'obiettivo è duplice: verificare l'integrità del dato eliminando artefatti di campionamento e confrontare modelli di Machine Learning per intercettare pazienti a rischio cardiovascolare, con focus prioritario sulla **Recall clinica (Sensibilità)**.

---

## 📌 Struttura del Progetto

1. **Notebook 01 - EDA & Data Cleaning**:
   - **Ispezione iniziale**: 1025 record e 13 predittori clinici, assenza di missing value formali.
   - **Deduplicazione**: rimozione di 723 record duplicati; il campione effettivo scende a 302 pazienti unici.
   - **Analisi Outlier**: identificazione tramite metodo IQR su colesterolo e pressione sistolica; i valori estremi (fino a 200 mmHg e 564 mg/dl) sono stati preservati perché clinicamente plausibili.
   - **Analisi di correlazione**: evidenziato il forte potere separatore della risposta allo sforzo (`exang`, `oldpeak`, `thalach`) rispetto a fattori ematici isolati (`chol`, `fbs`).
   - **Export**: esportazione del dataset pulito in `data/heart_cleaned.csv`.

2. **Notebook 02 - Baseline Classification Models**:
   - Suddivisione stratificata 80/20 (`stratify=y`) su 302 osservazioni.
   - Standardizzazione delle feature continue (`StandardScaler`) calcolata rigorosamente solo sul training set per prevenire Data Leakage.
   - Addestramento e confronto tra tre algoritmi: *Logistic Regression*, *Random Forest* e *K-Nearest Neighbors (KNN)*.

3. **Notebook 03 - Hyperparameter Tuning & Stratified CV**:
   - Implementazione di **Stratified 5-Fold Cross-Validation** e `Pipeline` di Scikit-Learn per evitare Data Leakage nei fold.
   - **Diagnosi di un errore precedente**: ottimizzare la sola Recall (`scoring='recall'`) scelse $C = 0.001$, un modello quasi degenere (specificità in CV 0.373, 83.0% di predizioni "malato" contro una prevalenza del 54.4%).
   - **Selezione del modello** (`C`, `l1_ratio`) con `GridSearchCV` su `average_precision`: scelti $C = 0.03$, regolarizzazione $L_2$ (`l1_ratio=0`); la griglia è piatta e nessun coefficiente è a zero, quindi non c'è evidenza di selezione delle feature.
   - **Scelta della soglia separata dal modello**: `TunedThresholdClassifierCV` solo sul training set, massimizzando la precisione con Recall ≥ 0.90 → soglia **0.283**.
   - Test set usato una sola volta; CV annidata con media ± deviazione standard.

---

## 📊 Risultati e Confronto Modelli

### Baseline (Notebook 02 - Test Set 20%)

| Modello | Accuracy | Precision | Recall (Sensibilità) | F1-Score | ROC-AUC |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Logistic Regression (Baseline)** | **0.803** | 0.800 | **0.848** | **0.824** | **0.870** |
| **Random Forest** | 0.754 | 0.765 | 0.788 | 0.776 | 0.858 |
| **KNN (k=5)** | 0.787 | **0.812** | 0.788 | 0.800 | 0.812 |

### Modello Ottimizzato (Notebook 03 - Test Set 20%, 61 pazienti: 33 malati, 28 sani)

| Modello | Accuracy | Precision | Recall | F2 | ROC-AUC | PR-AUC | FN | FP |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Baseline, soglia 0.5 | 0.803 | 0.800 | 0.848 | 0.838 | 0.871 | 0.895 | 5 | 7 |
| Ottimizzato, soglia 0.5 | 0.787 | 0.778 | 0.848 | 0.833 | 0.882 | 0.901 | 5 | 8 |
| **Ottimizzato, soglia calibrata (0.283)** | 0.787 | 0.738 | **0.939** | **0.891** | 0.882 | 0.901 | **2** | 11 |

Il baseline del notebook 03 usa una `Pipeline` che scala tutte le feature, quello del notebook 02 solo le continue: per questo la ROC-AUC è 0.871 qui e 0.870 sopra (le altre metriche coincidono).

**Perché ottimizzare la sola Recall era un errore e come l'ho risolto:** la Recall non penalizza i falsi positivi, quindi `scoring='recall'` premia il modello che chiama "malato" quasi tutti (`C=0.001`: Recall in CV 1.000 ma specificità 0.373); ho separato la selezione del modello (`average_precision`) dalla scelta della soglia (CV sul solo training, Recall ≥ 0.90).

### Cosa dicono (e non dicono) i numeri
- **Effetto della soglia**: rispetto alla soglia 0.5 la soglia calibrata recupera 3 malati (FN da 5 a 2, Recall 0.939) al prezzo di 3 falsi positivi in più (da 8 a 11). ROC-AUC e PR-AUC non cambiano, perché la soglia non modifica il ranking.
- **Incertezza**: in CV annidata sul training la Recall con soglia calibrata è 0.962 ± 0.060 ma la specificità è 0.391 ± 0.368, quindi il vincolo Recall ≥ 0.90 ha un costo in falsi positivi elevato e instabile su questo campione (baseline: specificità 0.764 ± 0.161).
- **Significatività**: con 61 pazienti nel test set, le differenze piccole tra i modelli non sono statisticamente significative (es. baseline e ottimizzato a soglia 0.5 differiscono di un solo falso positivo).
- **Regolarizzazione**: il modello scelto è $L_2$ senza coefficienti a zero; non c'è selezione delle feature.

---

## ⚠️ Limiti

- Campione ridotto (302 pazienti), derivato da una singola coorte di studio.
- La correlazione e i pesi lineari identificano associazioni statistiche, non relazioni causali dirette.

---

## 📂 Struttura della Repository

```text
heart-disease-eda/
├── data/
│   ├── heart.csv                  # Dataset grezzo iniziale (Kaggle)
│   └── heart_cleaned.csv          # Dataset deduplicato e pulito (302 righe)
├── notebooks/
│   ├── 01_eda_and_cleaning.ipynb  # Pulizia, gestione outlier ed EDA
│   ├── 02_classification_models.ipynb # Preprocessing anti-leakage e benchmark modelli
│   └── 03_hyperparameter_tuning.ipynb # Selezione su average_precision, soglia calibrata in CV, CV annidata
├── .gitignore                     # Esclusione cache, .idea e virtualenv
├── LICENSE                        # Licenza MIT
├── README.md                      # Documentazione del progetto
└── requirements.txt               # Dipendenze d'ambiente
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
