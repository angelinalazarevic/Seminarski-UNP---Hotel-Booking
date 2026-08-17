# Hotel Booking - analiza otkazivanja rezervacija

Seminarski rad iz predmeta **Uvod u nauku o podacima**.

**Autori:**

* Milena Đukić 40/2020
* Angelina Lazarević 66/2022

**Profesor:** dr Branko Arsić  
**Institut za matematiku i informatiku, Prirodno-matematički fakultet, Univerzitet u Kragujevcu**

## Opis projekta

Cilj rada je analiza karakteristika povezanih sa otkazivanjem hotelskih rezervacija i izgradnja modela mašinskog učenja za predikciju promenljive `is_canceled`.

Projekat obuhvata:

* upoznavanje i analizu skupa podataka;
* obradu nedostajućih i netipičnih vrednosti;
* eksplorativnu analizu podataka i vizualizaciju;
* statističko testiranje;
* feature engineering;
* pripremu podataka za mašinsko učenje;
* poređenje više klasifikacionih algoritama;
* grupnu stratifikovanu unakrsnu validaciju;
* proveru uticaja izbora manjeg broja karakteristika;
* podešavanje hiperparametara;
* evaluaciju konačnog modela na izdvojenom test skupu;
* analizu važnosti karakteristika;
* analizu grešaka modela.

## Skup podataka

Korišćen je **Hotel Booking Demand** skup podataka koji sadrži informacije o rezervacijama za dva tipa hotela: `Resort Hotel` i `City Hotel`.

Početni skup sadrži **119.390 rezervacija i 36 promenljivih**.

Ciljna promenljiva je:

* `is_canceled = 0` - rezervacija nije otkazana;
* `is_canceled = 1` - rezervacija je otkazana.

Tokom pripreme podataka analizirane su i obrađene nedostajuće i netipične vrednosti. Uklonjene su karakteristike koje predstavljaju direktne identifikatore ili mogu dovesti do curenja podataka, a kreirane su i nove karakteristike namenjene daljoj analizi i modeliranju.

Nakon čišćenja skup sadrži **119.208 rezervacija**. Za modeliranje su korišćene **34 karakteristike dostupne u trenutku rezervisanja**, dok su `assigned_room_type`, `booking_changes` i `days_in_waiting_list` izostavljene jer njihove konačne vrednosti nisu pouzdano poznate odmah nakon kreiranja rezervacije.

### Izvor podataka

* Kaggle: [Hotel Booking](https://www.kaggle.com/datasets/mojtaba142/hotel-booking)

## Struktura projekta

```text
.
├── data/
│   ├── hotel_booking.csv
│   ├── hotel_booking_cleaned.csv
│   └── hotel_booking_model_ready.csv
│
├── notebooks/
│   └── hotel_booking_analysis.ipynb
│
├── reports/
│   ├── hotel_booking_analysis.html
│   └── Predlog skupa podataka za seminarski rad.pdf
│
├── README.md
└── requirements.txt
```

Opis najvažnijih fajlova:

* `hotel_booking.csv` - originalni skup podataka;
* `hotel_booking_cleaned.csv` - skup nakon čišćenja podataka;
* `hotel_booking_model_ready.csv` - skup nakon feature engineering-a;
* `hotel_booking_analysis.ipynb` - kompletna analiza, od učitavanja podataka do evaluacije konačnog modela;
* `hotel_booking_analysis.html` - HTML verzija završne analize;
* `requirements.txt` - spisak Python biblioteka potrebnih za pokretanje projekta.

## Korišćene tehnologije

Projekat je realizovan u programskom jeziku **Python**.

Najvažnije korišćene biblioteke:

* pandas
* NumPy
* Matplotlib
* Seaborn
* SciPy
* scikit-learn
* XGBoost

## Pokretanje projekta

Klonirati repozitorijum:

```bash
git clone https://github.com/angelinalazarevic/Seminarski-UNP---Hotel-Booking.git
cd Seminarski-UNP---Hotel-Booking
```

Kreirati virtuelno okruženje:

```bash
python -m venv .venv
```

Na Windows PowerShell-u aktivirati okruženje:

```powershell
.\.venv\Scripts\Activate.ps1
```

Instalirati potrebne biblioteke:

```bash
python -m pip install -r requirements.txt
```

Pokrenuti Jupyter Notebook:

```bash
jupyter notebook
```

Zatim otvoriti:

```text
notebooks/hotel_booking_analysis.ipynb
```

Za potpunu reprodukciju rezultata preporučuje se pokretanje notebook-a od početka opcijom **Restart Kernel and Run All Cells**.

## Mašinsko učenje

U početnom poređenju korišćeni su sledeći klasifikacioni modeli:

* Dummy Classifier;
* Logistic Regression;
* Decision Tree;
* Random Forest;
* XGBoost.

Podaci su podeljeni na trening i test skup u približnom odnosu 80:20 korišćenjem `StratifiedGroupKFold`, tako da isti rezervacioni profil ne može biti prisutan u oba skupa, uz očuvanje približno istog odnosa otkazanih i neotkazanih rezervacija.

Modeli su zatim poređeni na trening skupu pomoću petostruke grupne stratifikovane unakrsne validacije. Kao glavna metrika za izbor modela korišćen je **F1-score**, uz praćenje accuracy, precision, recall i ROC-AUC vrednosti.

Najbolji početni rezultat ostvario je **XGBoost**, sa prosečnim CV F1 rezultatom **0.8060** i ROC-AUC vrednošću **0.9401**.

Za XGBoost su zatim proverene varijante sa:

* svih 34 dozvoljena prediktora;
* 20 najvažnijih prediktora;
* 10 najvažnijih prediktora.

Model sa svih 34 prediktora ostvario je najbolji F1 rezultat, pa smanjenje broja karakteristika nije primenjeno u konačnom modelu.

Nakon toga je izvršeno podešavanje XGBoost hiperparametara pomoću `RandomizedSearchCV`. Podešeni model je u petostrukoj grupnoj unakrsnoj validaciji povećao prosečan F1 sa **0.8060** na **0.8131**, pa je izabran kao konačni model.

## Konačni rezultati

Kao konačni model izabran je **podešeni XGBoost sa svih 34 dozvoljena prediktora**.

Na izdvojenom test skupu ostvarene su sledeće performanse:

| Metrika           | Vrednost |
| ----------------- | -------: |
| Accuracy          |   0.8727 |
| Precision         |   0.8420 |
| Recall            |   0.8083 |
| F1-score          |   0.8248 |
| ROC-AUC           |   0.9470 |
| Average Precision |   0.9221 |

Matrica konfuzije na test skupu pokazala je:

* 13.662 tačno prepoznate neotkazane rezervacije;
* 7.145 tačno prepoznatih otkazanih rezervacija;
* 1.341 neotkazanu rezervaciju pogrešno označenu kao otkazanu;
* 1.695 propuštenih otkazanih rezervacija.

Pored ukupnih metrika izvršena je analiza važnosti karakteristika i analiza pogrešno klasifikovanih rezervacija. Kao posebno važne karakteristike izdvojile su se `deposit_type` i `country`, dok se među značajnijim pojavljuju i `agent`, `market_segment`, `lead_time` i informacije o posebnim zahtevima.
