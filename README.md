# Analiza fairness modelu na danych Census Income

**Autor:** Daniel Sadowski, nr indeksu: 259314  
**Temat:** nr 4 - Analiza fairness modelu ML  
**Kurs:** Aspekty prawne, społeczne i etyczne w AI, PWr 2025/2026

---

## Wymagania i uruchomienie

Python 3.12+. Zalecane środowisko wirtualne:

```bash
python -m venv venv
source venv/bin/activate

pip install -r requirements.txt
```

Uruchomienie notebooka:

```bash
jupyter notebook notebooks/analiza_fairness_lsac.ipynb
```

## Cel projektu

Celem projektu jest zbadanie czy standardowe modele ML dyskryminują grupy demograficzne oraz - co ważniejsze - jak trudne jest usunięcie tej dyskryminacji i jakim kosztem. Intuicja podpowiada że model trenowany na danych zawierających atrybuty wrażliwe (płeć, rasa) będzie stronniczy, ale mniej oczywiste jest to że próba korekcji jednej definicji fairness nieuchronnie pogarsza inną. W tym repo pokazuję ten trade-off na konkretnych liczbach.

## Powiązanie z projektem grupowym

Projekt grupowy dotyczy wykrywania napadów padaczkowych na danych EEG. 
Problem fairness jest tam analogiczny: model trenowany na niejednorodnych danych klinicznych 
może mieć istotnie różny recall dla różnych podgrup pacjentów - dokładnie tak jak model 
bazowy w tym projekcie miał recall 0.53 dla kobiet vs. 0.64 dla mężczyzn. W kontekście 
medycznym taka różnica to nie kwestia etyczna w abstrakcyjnym sensie, lecz realne ryzyko 
przeoczenia napadu u konkretnej grupy pacjentów.

Dodatkowe powiązanie dotyczy wyjaśnialności: projekt grupowy pokazuje że metody XAI 
nie zgadzają się ze sobą (Integrated Gradients vs. Attention Rollout vs. GradientSHAP), 
a faithfulness jest niska. To ten sam problem co z fairness - optymalizacja jednej metryki 
psuje inną. Nie możesz jednocześnie mieć modelu dokładnego, wyjaśnialnego i sprawiedliwego 
bez świadomego zarządzania tymi trade-offami.

## Wyniki

Porównanie trzech modeli na zbiorze testowym (9207 rekordów, atrybut wrażliwy: płeć):

| Model | Accuracy | ROC-AUC | DPD (płeć) | EOD (płeć) |
|---|---|---|---|---|
| Bazowy (Logistic Regression) | 0.850 | 0.902 | 0.183 | 0.108 |
| ThresholdOptimizer [demographic_parity] | 0.795 | - | 0.015 | 0.194 |
| ThresholdOptimizer [equalized_odds] | 0.766 | - | 0.093 | 0.018 |

- **DPD (Demographic Parity Difference)** - różnica w odsetku pozytywnych predykcji między grupami; 0 = brak dyskryminacji
- **EOD (Equalized Odds Difference)** - różnica w TPR i FPR między grupami; 0 = brak dyskryminacji

Model bazowy osiąga najwyższą dokładność, ale przypisuje kobietom wynik >50K o 18 p.p. rzadziej niż mężczyznom przy tych samych pozostałych cechach. Korekcja demographic parity redukuje DPD do 0.015, lecz EOD rośnie do 0.194. Korekcja equalized odds działa odwrotnie. Szczegółowe wykresy: [`wyniki/porownanie_modeli.png`](wyniki/porownanie_modeli.png), [`wyniki/eda_grupy_wrazliwe.png`](wyniki/eda_grupy_wrazliwe.png).

## Wnioski merytoryczne

### 1. Model bazowy dyskryminuje - i jest to mierzalne

DPD = 0.183 oznacza że model przypisuje kobietom wynik >50K o 18 p.p. rzadziej niż mężczyznom przy identycznych pozostałych cechach. To nie jest artefakt - to reprodukcja nierówności historycznych obecnych w danych treningowych (dane z 1994 roku).

### 2. Korekcja fairness jest możliwa, ale ma cenę

Żadna z zastosowanych korekcji nie poprawia jednocześnie DPD i EOD. To nie błąd implementacji - to konsekwencja twierdzenia o niemożliwości (Chouldechova 2017): przy niezbalansowanych klasach nie można jednocześnie spełnić demographic parity i equalized odds. Regulator który wymaga "fairness" bez wskazania konkretnej definicji stwarza lukę prawną którą można wypełnić dowolnie.

### 3. AI Act - system wysokiego ryzyka

Zgodnie z **AI Act, Art. 6 w związku z Załącznikiem III, pkt 1(b)**, systemy AI stosowane do podejmowania decyzji o dostępie do zatrudnienia lub edukacji klasyfikowane są jako systemy wysokiego ryzyka. Model tego typu wdrożony przez pracodawcę lub instytucję finansową podlegałby obowiązkom z **Rozdziału III AI Act**: ocena zgodności, rejestracja w bazie EU, dokumentacja techniczna i audyt przed wdrożeniem. Nasze DPD = 0.183 byłoby konkretnym wskaźnikiem wymagającym wyjaśnienia w dokumentacji.

### 4. RODO Art. 22 - zakaz zautomatyzowanych decyzji

**Art. 22 RODO** zakazuje podejmowania decyzji wywołujących skutki prawne wyłącznie w oparciu o zautomatyzowane przetwarzanie, bez udziału człowieka. Model predykcji dochodu stosowany do oceny zdolności kredytowej lub rekrutacji bez human-in-the-loop naruszałby ten przepis. Co istotne, prawo do wyjaśnienia decyzji (art. 13-14 RODO) jest trudne do realizacji gdy model wykazuje bias - wyjaśnienie "model uznał że nie kwalifikujesz się" jest niewystarczające jeśli decyzja jest skorelowana z płcią.

### 5. Pośrednia dyskryminacja a proxy features

Model nie musi używać płci explicite żeby dyskryminować - cechy takie jak `marital-status` czy `relationship` są silnie skorelowane z płcią i mogą pośrednio odtwarzać bias po usunięciu atrybutu wrażliwego. Dyrektywa 2000/43/EC oraz **Kodeks pracy Art. 18(3a)** zakazują dyskryminacji pośredniej, co oznacza że usunięcie cechy `sex` z modelu nie jest wystarczającym środkiem zaradczym - wymagana jest audyt całego pipeline'u danych.

## Ograniczenia

- Analiza dotyczy wyłącznie atrybutu płci - rasa została wstępnie zbadana w EDA, ale nie weszła do porównania modeli ze względu na silną nierównowagę grup (White: 86% danych).
- Dataset pochodzi z 1994 roku - wnioski socjoekonomiczne są historyczne, nie współczesne.
- ThresholdOptimizer działa post-hoc - nie zmienia reprezentacji grup w treningu. Podejścia in-processing (np. Reductions, Adversarial Debiasing) mogłyby dać lepszy trade-off accuracy/fairness.
- Projekt nie obejmuje analizy proxy features - usunięcie atrybutu `sex` z modelu i zbadanie czy bias pozostaje przez zmienne skorelowane byłoby naturalnym rozszerzeniem.

## Źródła

- [Adult Census Income Dataset - OpenML](https://www.openml.org/d/1590) - dane użyte w analizie
- [fairlearn - dokumentacja](https://fairlearn.org/) - biblioteka do pomiaru i korekcji fairness
- [AI Act - tekst rozporządzenia](https://eur-lex.europa.eu/legal-content/PL/TXT/?uri=CELEX:32024R1689) - Rozporządzenie PE i Rady (UE) 2024/1689
- [RODO - Art. 22](https://eur-lex.europa.eu/legal-content/PL/TXT/?uri=CELEX:02016R0679-20160504) - Rozporządzenie PE i Rady (UE) 2016/679
- [Chouldechova A. (2017) - Fair prediction with disparate impact](https://arxiv.org/abs/1703.00056) - twierdzenie o niemożliwości jednoczesnej fairness
- [Hardt M. et al. (2016) - Equality of Opportunity in Supervised Learning](https://arxiv.org/abs/1610.02413) - definicja equalized odds
- [Dyrektywa 2000/43/EC](https://eur-lex.europa.eu/legal-content/PL/TXT/?uri=CELEX:32000L0043) - zakaz dyskryminacji pośredniej
