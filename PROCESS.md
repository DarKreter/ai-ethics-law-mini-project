# Dokumentacja procesu

Ten plik dokumentuje **jak** pracowałem/am nad mini-projektem - jakie narzędzia AI wykorzystałem, jakie prompty pisałem, jakie decyzje podjąłem i co nie zadziałało.

> **PROCESS.md jest tak samo ważny jak kod.** Prowadzący ocenia świadome korzystanie z narzędzi AI - to jest kurs o aspektach AI.

---

## Narzędzia AI

[Lista narzędzi AI użytych w projekcie]

| Narzędzie | Do czego używałem |
|-----------|-------------------|
| Claude | Planowanie projektu, generowanie kodu notebooka iteracyjnie, pisanie README.md |


## Prompty

Każdy z poniższych promptów inicjował sesję wielopromptową - Claude dostarczał kolejne komórki/sekcje, ja weryfikowałem wyniki i korygowałem kierunek w kolejnych wiadomościach.

### Planowanie projektu
```
You are great programmer, good AI engineer and data scientist.

Mam do wykonania listę na zajęcia na uczelni. Lista wygląda tak.

[...]

Chciałbym zrobić ten projekt. Mój konkretny temat jest wybrany i wpisany w README.
Jest tutaj dużo zasad których mam się trzymać i tego żeby nie generować wszystkiego LLMem tylko pisać promtpy. Ale projekt jest chyba dosyć prosty może stworzyć taki działający notebook z uwagami plus opisać jakich promptów używałem.

Jak zaplanowałbyś pracę nad tym, co zrobił po kolei i czemu którą rzecz w jaki sposób. Bądź krytyczny i pokazuj swój tok rozumowania i gdzie w wymaganiach jest wymóg przez który tak robisz albo co akurat spełniasz.

Jeśli czegoś nie jesteś pewien zapytaj mnie i podyskutujemy. Ale pamiętaj, że najważniejsze jest, zeby mieć ten projekt zrobiony. Chce to zrobić na takie Good, nie ma co robić excellent.
```
**Kontekst:** Punkt startowy - chciałem zobaczyć plan zanim zacznę pisać cokolwiek, żeby nie tracić czasu na złe podejście.

### Generowanie notebooka
```
No dobra to lecimy w notebook, zeby jak najsprawniej mieć go zrobionego, a przy okazji będę dokumentował prompty i jakoś to tam się bedzie dało opisać.
Projekt grupowy najlepiej opisuje ta prezentacja: [...]
Tutaj jeszcze w pliku AGENTS.md mam takie fajne zasady których powinniśmy się trzymać: [...]

Zaczynamy pracę od notebooka jupyterowego, nie generuj mi kodu całości na raz. Pracujemy iteracyjnie, dostarczasz mi kolejnych komórek według ustalonego planu, ja jestem w stanie je zweryfikować czy robią to co chce, zeby robiły i czy faktycznie działają. Nie ma sensu generować od razu całości, bo jak coś z początku nie działa albo zrobisz nie tak jak chce, to potem cała reszta musi być przegenerowana. Więc pracujemy w pętli sprzężenia zwrotnego i skupiamy się wyłącznie na notebooku na ten moment. Wnioski będziemy pisać dopiero jak dostaniemy konkretne wyniki.
```
**Kontekst:** Wymuszenie pracy iteracyjnej

### Pisanie README
```
Świetnie, tak więc po ciężkiej walce mamy zamknięty notebook. Teraz pora napisać README. Taki jest template: [...]
Pamiętaj o tym wymaganiu: [...]

To jest ważny dokument i to co tu piszemy musimy przedyskutować. Tak samo jako notebooka nie pisz tego wszystkiego naraz, bo wyjdzie słabo. Napiszmy to krok po kroku. Pisz swoje propozycje jak opisałbyś kolejne sekcje, albo na czym powinienem się skupić, żeby to opracować, ja będę to weryfikował albo pisał trochę po swojemu i krok po kroku dojdziemy do tego co chce przekazać i jak to napisać.
```
**Kontekst:** Ten sam wzorzec co przy notebooku.

## Decyzje

1. **Zmiana datasetu z LSAC na Adult Census Income** - LSAC wymagał rejestracji na SEAPHE, brak otwartego mirrora. Adult jest szerzej cytowany w literaturze fairness (Hardt 2016, Mehrabi 2021), więc zmiana nie była kompromisem.
2. **Binaryzacja atrybutu `race`** - grupy poza White stanowią łącznie ~14% danych, metryki fairness dla tak małych grup są statystycznie niewiarygodne. Zdecydowałem się na White/Non-White i skupienie analizy na płci.
3. **ThresholdOptimizer zamiast podejść in-processing** - post-hoc korekcja jest prostsza do zrozumienia i bezpośrednio ilustruje trade-off między metrykami, co było celem projektu.

## Co nie zadziałało

1. **Dataset LSAC** - docelowy dataset z tematu nr 4 wymagał rejestracji na platformie SEAPHE i nie był dostępny przez żadne otwarte mirror. Zastąpiony przez Adult Census Income jako uznany benchmark fairness.

## Iteracje

1. **v1** - planowany notebook na danych LSAC, analiza fairness przyjęć na studia prawnicze
2. **v2** - zmiana datasetu na Adult Census Income po braku dostępu do LSAC
3. **v3 (finalna)** - notebook z EDA, modelem bazowym i dwoma wariantami korekcji ThresholdOptimizer, porównanie DPD/EOD, wnioski prawne powiązane z AI Act i RODO
