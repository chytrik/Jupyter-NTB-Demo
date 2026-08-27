# Predictive Maintenance — Predikce poruch průmyslového zařízení

## Co notebook ukazuje

Prediktivní údržba na syntetických IoT datech z CNC strojů. Notebook prochází celým ML
cyklem od explorace dat po vyčíslení business dopadu — bez potřeby GPU.

## Prerekvizity

- Standardní Data Science workbench image (pandas, scikit-learn, matplotlib)
- Žádný model serving — vše běží lokálně na CPU

## Syntetická data (`data/equipment_sensors.csv`)

| Parametr | Hodnota |
|----------|---------|
| Záznamy | 5 760 |
| Strojů | 8 (CNC-001 až CNC-008) |
| Období | 6 měsíců (červenec–prosinec 2025) |
| Frekvence | 4 čtení/den (každých 6 hodin) |
| Failure rate | ~7 % |

### Senzory (features)

| Senzor | Jednotka | Chování před poruchou |
|--------|----------|----------------------|
| `temperature_c` | °C | Roste |
| `vibration_mm_s` | mm/s | Roste |
| `pressure_bar` | bar | Klesá |
| `rpm` | ot/min | Klesá |
| `power_kw` | kW | Roste |
| `acoustic_db` | dB | Roste |

Doplňkové features: `operating_hours`, `machine_age_years`, `days_since_last_maintenance`.

**Target:** `failure_within_7d` — binární (1 = porucha nastane do 7 dní).

### Jak byla data generována

Každý stroj má 1–3 náhodné poruchy. Před každou poruchou probíhá 14denní degradace —
teplota a vibrace graduálně rostou, tlak klesá. K tomu sezónní efekt, denní cyklus
a realistický šum. Korelace mezi senzory jsou záměrné (příkon závisí na teplotě a vibracích).

## Průběh notebooku

1. **Explorace dat** — distribuce, statistiky, target balance
2. **Vizualizace degradace** — 4 senzory CNC-001 v čase, červené zóny = pre-failure okna
3. **Feature engineering** — z 12 raw features na ~39 (rolling mean/std 24h a 3d, delty, cross-sensor interakce jako temp×vibrace, příkon/otáčky)
4. **Trénink 3 modelů** — Logistic Regression, Random Forest, Gradient Boosting
   - Time-based split: train červenec–říjen, test listopad–prosinec
   - Class balancing (imbalanced dataset, 7 % positives)
5. **ROC + Precision-Recall křivky** — vizuální porovnání modelů
6. **Feature importance** — které features nejvíc přispívají k predikci
7. **Business impact** — EUR kalkulace úspor (reaktivní 15k/porucha vs prediktivní 2k)
8. **Timeline** — pravděpodobnost poruchy vs. reálné senzory pro CNC-002

## Klíčové výsledky

| Metrika | Hodnota |
|---------|---------|
| Nejlepší model | Gradient Boosting / Random Forest |
| AUC | ~0.95+ |
| Úspora nákladů | ~60–80 % oproti reaktivní údržbě |
| Klíčové features | Rolling statistiky a cross-sensor interakce > raw hodnoty |

## Návaznost na OpenShift AI

Notebook ukazuje cestu od dat k modelu. Další kroky v produkci:

1. **Model Registry** — registrace natrénovaného modelu s verzí a metadaty
2. **Model Serving** — nasazení přes KServe pro real-time scoring
3. **Observe & monitor** — sledování data driftu a kvality predikcí
4. **Pipelines** — automatizovaný retraining při detekci driftu
5. **Governance** — kdo trénoval, na jakých datech, kdo schválil deployment
