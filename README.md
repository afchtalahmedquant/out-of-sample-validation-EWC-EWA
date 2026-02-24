# out-of-sample-validation-EWC-EWA

# Pair Trading EWC / EWA — Validation Rigoureuse d'une Stratégie Quantitative

> Stratégie de Pair Trading sur les ETFs Canada (EWC) vs Australie (EWA),  
> testée en trois étapes : Grid Search, Walk Forward Analysis, Validation Out-of-Sample.

---

## Contexte

Mon premier post LinkedIn présentait un backtest sur EWC/EWA de 1996 à 2025 avec des paramètres choisis à l'instinct et un Filtre de Kalman dynamique déjà en place. Les résultats semblaient positifs.

Ce repo documente ce qui se passe quand on challenge ces résultats sérieusement.

---

## Hypothèse de la Paire

EWC (Canada) et EWA (Australie) partagent une structure économique similaire — deux économies orientées matières premières avec un secteur financier dominant. L'idée : leur spread de prix devrait revenir vers la moyenne.

---

## Ce qui a été ajouté :



- **Grid Search** — 27 combinaisons de paramètres testées systématiquement (entrée, sortie, stop-loss)
- **Walk Forward Analysis** — optimisation sur une fenêtre, test sur la fenêtre suivante jamais vue
- **Random Forest** — tentative d'amélioration par ML (filtre ou sizing des positions)

Note : la période a été réduite de 1996–2025 à 2014–2026 pour une raison simple — le Walk Forward Analysis avec grid search sur 27 fenêtres tourne plusieurs heures sur un seul PC. Contrainte matérielle, pas statistique.

---

## Résultats

### Grid Search — In-Sample (2014–2022) — 27 combinaisons validées

| Entry | Exit | Stop-Loss | Sharpe | Sortino | Calmar | Max DD | Win Rate |
|-------|------|-----------|--------|---------|--------|--------|----------|
| **1.1** | **0.9** | **4.0** | **0.786** | **0.967** | **0.422** | **-11.62%** | **39.9%** |
| 1.1 | 0.9 | 3.0 | 0.765 | — | — | -12.50% | — |
| 2.0 | 0.9 | 4.0 | 0.701 | — | — | -10.41% | — |
| 1.75 | 0.9 | 4.0 | 0.673 | — | — | -11.81% | — |
| 1.1 | 0.7 | 4.0 | 0.487 | — | — | -12.55% | — |

**Observation** : exit=0.9 domine systématiquement. entry=1.1 génère plus de trades avec un meilleur Sharpe mais un drawdown légèrement plus élevé.

<img width="1251" height="952" alt="image" src="https://github.com/user-attachments/assets/94b11763-8dd7-4bc1-8d8d-56e971fa94d1" />
<img width="1251" height="952" alt="image" src="https://github.com/user-attachments/assets/94b11763-8dd7-4bc1-8d8d-56e971fa94d1" />


---

### Walk Forward Analysis — 4 Fenêtres (Train: 521j / Test: 126j)

| Fenêtre | Train Sharpe | Test Sharpe | Dégradation | Max DD | Entry | Exit |
|---------|-------------|-------------|-------------|--------|-------|------|
| 1 | 1.633 | 1.352 | -17.2% | -5.74% | 2.0 | 0.7 |
| 2 | 1.795 | 2.395 | +33.4% | -0.40% | 2.0 | 0.9 |
| 3 | 1.787 | 1.069 | -40.2% | -0.73% | 2.0 | 0.9 |
| 4 | 1.892 | -1.635 | -186.4% | -7.96% | 1.1 | 0.9 |

**Résumé :**
- Sharpe moyen test : **0.795**
- Sharpe médian test : **1.210**
- Max DD moyen : **-3.71%**
- Ratio Test/Train moyen : **0.47**
- **3/4 fenêtres positives**

La fenêtre 4 (période incluant 2023+) s'effondre à -1.635. Cohérent avec la rupture de cointégration post-2022.

<img width="1395" height="868" alt="image" src="https://github.com/user-attachments/assets/328f3899-b4f7-486f-a8df-835435a3e799" />
<img width="1395" height="868" alt="image" src="https://github.com/user-attachments/assets/328f3899-b4f7-486f-a8df-835435a3e799" />


---

### Cointégration — Le Problème Central

| Période | p-value Engle-Granger | Résultat |
|---------|----------------------|----------|
| 2014–2026 (complète) | 0.181 | Non cointégrée |
| 2014–2022 (training) | 0.025 | Modérée (95%) |
| Fenêtres WFA test (moy.) | > 0.4 | Non cointégrées |
| 2023–2026 (out-of-sample) | 0.437 | Non cointégrée |

Le spread reste **stationnaire sur toutes les périodes** (ADF p < 0.001). Mais la relation de long terme est cassée post-2022.

---

### Machine Learning (Random Forest)

| Mode | Trades | Sharpe | Return Total | Verdict |
|------|--------|--------|--------------|---------|
| Sans ML (baseline) | 548 | 0.515 | +36.5% | Référence |
| Filter (binaire) | 3 | -0.26 | -1.42% | Inutilisable |
| Sizing (continu) | 225 | 0.524 | +36.5% | Neutre |

**Modèle :** Random Forest, 100 arbres, max_depth=5  
**Accuracy test : 70.3%** — trompeuse (déséquilibre de classes)  
**AUC : 0.49** — ne prédit pas mieux que le hasard en mode binaire

---

### Validation Out-of-Sample — 2023 à 2026 (sans ML)

Paramètres : entry=1.1, exit=0.9, stop=4.0

| Métrique | Valeur |
|----------|--------|
| Annual Return | 2.22% |
| Total Return | 6.79% |
| Sharpe Ratio | 0.731 |
| Sortino Ratio | 0.761 |
| Calmar Ratio | 0.493 |
| Max Drawdown | -4.51% |
| Win Rate | 38.27% |
| Profit Factor | 1.219 |

Malgré la perte de cointégration post-2022, le spread reste mean-révertant. Sharpe 0.73 en out-of-sample — mais fragile, à monitorer.

![WhatsApp Image 2026-02-24 at 12 40 26](https://github.com/user-attachments/assets/d9ce4473-fb7c-4c98-8d8c-e7ebc5a7dd1b)
![WhatsApp Image 2026-02-24 at 12 40 26](https://github.com/user-attachments/assets/d9ce4473-fb7c-4c98-8d8c-e7ebc5a7dd1b)


---

## Conclusion

Les résultats du premier post n'étaient pas de la chance — la relation existait et généraient de l'alpha sur 3/4 fenêtres WFA (Sharpe moyen 0.795). Mais le régime a changé après 2022. La stratégie n'est pas opérationnelle en l'état sans un filtre de régime basé sur la p-value de cointégration.


---

## Installation

```bash
pip install yfinance numpy pandas matplotlib statsmodels pykalman scikit-learn
```

---



---

*Projet de recherche personnel. Pas un conseil financier.*
