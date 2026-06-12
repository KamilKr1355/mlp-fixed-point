# Raport z optymalizacji sieci neuronowych

Projekt polegał na zbadaniu wpływu kwantyzacji na rozmiar pamięciowy, czas wnioskowania oraz skuteczność modeli uczenia maszynowego. Kwantyzacja to technika optymalizacji polegająca na zamianie precyzji wag z formatu zmiennoprzecinkowego FP32 na całkowitoliczbowy format INT8. Zastosowanie takich zmian jest kluczowe przy wdrażaniu sztucznej inteligencji na urządzenia wbudowane oraz telefony komórkowe.

## Metodologia i środowisko testowe

Do realizacji zadań wykorzystano bibliotekę PyTorch. Testy przeprowadzono w dwóch etapach na dwóch różnych zbiorach obrazów. Pierwszy zbiór to MNIST, który zawiera czarnobiałe obrazki z odręcznie napisanymi cyframi. Zastosowano tutaj prostą sieć wielowarstwową typu MLP. Drugi zbiór to CIFAR10. Zawiera on dużo bardziej złożone, kolorowe zdjęcia różnych obiektów. Do tego zadania zbudowano rozbudowaną sieć konwolucyjną typu CNN. 

Modele były początkowo trenowane w pełnej precyzji FP32. Następnie wprowadzono dwie metody konwersji do formatu INT8. Pierwszą z nich była metoda PTQ, a drugą QAT.

## Opis wdrożonych metod optymalizacji

Podejście bazowe to standardowy model operujący na liczbach FP32. Stanowił on bezpośredni punkt odniesienia dla pozostałych testów.

Drugie podejście to PTQ. Konwersja na format INT8 odbywała się tutaj po całkowitym zakończeniu uczenia modelu. Aby proces przebiegł poprawnie, gotowy model był dodatkowo kalibrowany na małej paczce danych testowych w celu ustalenia odpowiednich zakresów liczbowych.

Trzecie podejście to QAT. W tym wariancie bierzemy bazowy model FP32, włączamy symulację kwantyzacji i trenujemy go przez kilka dodatkowych epok. Dzięki temu sieć na bieżąco koryguje wagi i uczy się rekompensować błędy wynikające z utraty precyzji. Docelowa konwersja do stałoprzecinkowego formatu INT8 następuje na samym końcu.

## Wyniki eksperymentu dla zbioru MNIST

W przypadku prostej sieci klasyfikującej czarnobiałe cyfry różnice w skuteczności pomiędzy poszczególnymi wariantami okazały się pomijalne.

| Wersja modelu | Dokładność testowa | Rozmiar w pamięci | Czas inferencji |
| :--- | :--- | :--- | :--- |
| Model FP32 | 97.83% | 440.24 KB | 1996.3 ms |
| Model PTQ | 97.82% | 119.45 KB | 1981.4 ms |
| Model QAT | 98.11% | 119.45 KB | 2012.8 ms |

Model bazowy FP32 osiągnął niemal 98 procent dokładności. Zastosowanie konwersji PTQ zmniejszyło ten wynik o zaledwie jedną setną procenta. Model trenowany metodą QAT poradził sobie z kolei najlepiej i przekroczył barierę 98 procent.

Główną zaletą optymalizacji okazała się znaczna redukcja zapotrzebowania na pamięć. Dzięki zastosowaniu formatu INT8 rozmiar modelu spadł niemal czterokrotnie, zajmując nieco ponad 100 kilobajtów. Czas potrzebny na wygenerowanie odpowiedzi na procesorze pozostał na bardzo podobnym poziomie we wszystkich trzech wariantach.

## Wyniki eksperymentu dla zbioru CIFAR10 i analiza regularyzacji

Bardziej wymagający zbiór kolorowych obrazków znacznie lepiej obnażył koszty związane z kompresją sieci.

| Wersja modelu | Dokładność testowa | Rozmiar w pamięci | Czas inferencji | Dokładność na szumie |
| :--- | :--- | :--- | :--- | :--- |
| Model FP32 | 79.31% | 2485.08 KB | 28685.5 ms | 38.36% |
| Model PTQ | 79.13% | 638.80 KB | 18704.6 ms | 39.25% |
| Model QAT | 77.10% | 638.80 KB | 20038.7 ms | 42.70% |

Standardowy model FP32 zajmował blisko dwa i pół megabajta. Obydwie metody kwantyzacji pozwoliły zmniejszyć tę wartość do około 638 kilobajtów. Warto zauważyć, że przy większej sieci CNN zastosowanie wariantu PTQ widocznie skróciło czas wnioskowania z prawie 29 do niecałych 19 sekund.

Pojawiły się jednak wyraźne różnice w dokładności predykcji. Metoda PTQ zachowała wynik bardzo bliski oryginałowi na poziomie 79 procent. Model uczony z użyciem QAT wypadł gorzej osiągając 77 procent poprawnych klasyfikacji.

Najciekawsze wnioski płyną jednak z testów odpornościowych. Do zbioru testowego celowo dodano silny szum Gaussa. Model bazowy FP32 kompletnie sobie nie poradził, a jego skuteczność spadła do 38 procent. Wersja PTQ zadziałała nieznacznie lepiej. Zdecydowanym zwycięzcą tego testu okazał się model QAT. Utrzymał on trafność przewidywań na poziomie blisko 43 procent. Dowodzi to, że ciągła symulacja błędów ułamkowych w trakcie uczenia działa jak bardzo silna regularyzacja. Sieć uczy się ignorować drobne zakłócenia i zyskuje o wiele większą odporność na zniekształcenia danych.

## Podsumowanie i wnioski końcowe

Redukcja rozmiaru parametrów sieci poprzez kwantyzację jest wysoce opłacalnym procesem. Oszczędność pamięci rzędu 75 procent to rewelacyjny wynik ułatwiający tworzenie lekkich aplikacji.

Każda z przetestowanych metod ma swoje idealne zastosowanie. Konwersja PTQ sprawdza się świetnie, gdy posiadamy gotowy model i chcemy błyskawicznie zmniejszyć jego rozmiar bez odczuwalnych spadków trafności na czystych danych. Z kolei trening QAT jest bardziej wymagający obliczeniowo, ale daje wymierne korzyści przy wdrażaniu sztucznej inteligencji w środowisku naturalnym. Taki model może radzić sobie odrobinę słabiej na idealnych danych laboratoryjnych, ale jego ogólna stabilność działania i odporność na zniekształcenia wizualne rośnie w zauważalny sposób.