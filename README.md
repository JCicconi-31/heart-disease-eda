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
   - Ottimizzazione degli iperparametri della *Logistic Regression* tramite `GridSearchCV` orientata espressamente a massimizzare la metrica di **Recall** (`scoring='recall'`).
   - Selezione della regolarizzazione $L_1$ (Lasso) con $C = 0.001$, riducendo i Falsi Negativi.

---

## 📊 Risultati e Confronto Modelli

### Baseline (Notebook 02 - Test Set 20%)

| Modello | Accuracy | Precision | Recall (Sensibilità) | F1-Score | ROC-AUC |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Logistic Regression (Baseline)** | **0.803** | 0.800 | **0.848** | **0.824** | **0.870** |
| **Random Forest** | 0.754 | 0.765 | 0.788 | 0.776 | 0.858 |
| **KNN (k=5)** | 0.787 | **0.812** | 0.788 | 0.800 | 0.812 |

### Modello Ottimizzato (Notebook 03 - Test Set 20%)

| Modello | Accuracy | Recall (Sensibilità) | ROC-AUC | Note |
| :--- | :---: | :---: | :---: | :--- |
| **Logistic Regression (Ottimizzata)** | 0.754 | **0.939** | **0.870** | **+9.1% di Recall** rispetto al baseline |

### 🩺 Perché la Logistic Regression Ottimizzata è il modello migliore
- **Priorità Clinica alla Recall (0.939)**: in ambito diagnostico, l'errore più critico è il Falso Negativo (un paziente malato classificato erroneamente come sano). Con la ricerca guidata su `scoring='recall'`, la Logistic Regression intercetta quasi il **94%** dei soggetti a rischio (rispetto all'84.8% del baseline).
- **Trade-off consapevole**: l'Accuracy passa dall'80.3% al 75.4%, accettando un lieve incremento di Falsi Positivi pur di abbattere drasticamente i Falsi Negativi.
- **Interpretabilità**: la regolarizzazione $L_1$ isola le variabili a maggiore impatto predittivo.

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
│   └── 03_hyperparameter_tuning.ipynb # Stratified 5-Fold CV e GridSearchCV guidata su Recall
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
