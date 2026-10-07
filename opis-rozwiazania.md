# Zadanie 5, wariant 2: wykrywanie detali wadliwych

## Pomysł w jednym zdaniu

Uczę sieć neuronową rozróżniać dwie klasy zdjęć: **detal dobry** i **detal wadliwy** (rysa, pęknięcie, brak materiału).
To zwykła klasyfikacja obrazów.

## 1. Dane

### Skąd

- Kamera nad taśmą robi zdjęcie każdego detalu.
- Zbieram zdjęcia przez kilka dni, żeby trafiły się różne partie i różne pory dnia.
- Detale wadliwe są rzadkie, więc dokładam te, które odrzuciła kontrola jakości.
- Cel na start: ok. 1000 zdjęć dobrych i 200–300 wadliwych.

### Etykiety

- Pracownik kontroli jakości przypisuje każdemu zdjęciu: `dobry` albo `wadliwy`.
- Zdjęcia trafiają do dwóch folderów, to wystarczy do uczenia.

### Przygotowanie

- przycięcie zdjęcia do samego detalu
- zmiana rozmiaru na 224×224 pikseli
- normalizacja wartości pikseli

### Augmentacja (sztuczne powiększenie zbioru)

Wadliwych zdjęć jest mało, więc z każdego robię kilka wariantów:

- obrót, odbicie lustrzane
- lekka zmiana jasności i kontrastu

Dzięki temu model widzi więcej przykładów i jest odporniejszy na ułożenie detalu i oświetlenie.

## 2. Model

### Co wybieram

Gotowa sieć konwolucyjna **ResNet-18**, wytrenowana wcześniej na zbiorze ImageNet, z wymienioną ostatnią warstwą na 2 klasy (**transfer learning**).

### Dlaczego

- Sieć konwolucyjna to standard dla obrazów: sama uczy się, jakie cechy (krawędzie, faktury) są ważne.
- Sieć wytrenowana wcześniej już „umie patrzeć", więc wystarczy mało własnych zdjęć i krótkie uczenie.
- ResNet-18 jest mała i szybka, zdąży ocenić detal, zanim przejedzie po taśmie.

### Jak uczę

1. Wczytuję sieć z gotowymi wagami.
2. Podmieniam ostatnią warstwę na wyjście: dobry / wadliwy.
3. Uczę kilkanaście epok, obserwując wynik na zbiorze walidacyjnym.
4. Zatrzymuję uczenie, gdy wynik na walidacji przestaje się poprawiać (żeby uniknąć przeuczenia).

### Niezbalansowane klasy

Dobrych zdjęć jest dużo więcej niż wadliwych. Żeby model nie nauczył się odpowiadać zawsze „dobry":

- nadaję klasie `wadliwy` większą wagę w funkcji straty
- (albo) częściej losuję zdjęcia wadliwe podczas uczenia

## 3. Jak mierzę skuteczność

### Podział danych

| Zbiór | Udział | Do czego |
|---|---|---|
| uczący | 70% | uczenie modelu |
| walidacyjny | 15% | dobór ustawień i progu |
| testowy | 15% | końcowa ocena, używany tylko raz |

W każdym zbiorze zachowuję tę samą proporcję dobrych do wadliwych.

### Metryki

Sama dokładność (accuracy) jest myląca: jeśli 95% detali jest dobrych, model mówiący zawsze „dobry" ma 95% dokładności i nie wykrywa niczego. Dlatego patrzę na:

- **macierz pomyłek**: ile dobrych i wadliwych oceniono poprawnie, a ile błędnie
- **czułość (recall)**: jaki procent wadliwych detali model wykrył. To najważniejsza liczba, bo przepuszczona wada trafia do klienta.
- **precyzja**: jaki procent detali oznaczonych jako wadliwe faktycznie był wadliwy. Niska precyzja = wyrzucamy dobre detale.
- **F1**: jedna liczba łącząca czułość i precyzję

## 4. Gdy model nie jest pewny albo się myli

### Niepewność

Model zwraca prawdopodobieństwo, że detal jest wadliwy (0–100%). Zamiast jednego progu stosuję dwa:

- poniżej 20% → **dobry**, jedzie dalej
- powyżej 80% → **wadliwy**, odrzut
- pomiędzy → **niepewny**, do sprawdzenia przez człowieka

Progi 20% i 80% to przykład; właściwe wartości dobieram na zbiorze walidacyjnym.

### Pomyłki

- Gorsza jest przepuszczona wada niż odrzucony dobry detal, więc progi ustawiam ostrożnie (wolę więcej sprawdzeń ręcznych).
- Zdjęcia detali niepewnych i błędnie ocenionych zapisuję razem z decyzją człowieka.
- Co jakiś czas dodaję je do zbioru uczącego i trenuję model ponownie. System z czasem się poprawia.
- Kontroler co jakiś czas sprawdza losowe detale uznane za dobre, żeby wiedzieć, czy model nie zaczął przepuszczać wad.

## 5. Kod

Notebook w PyTorch: https://github.com/KrzysztofGagalo/wykrywanie-wad-detali

- Dane: publiczny zbiór zdjęć odlewów z Kaggle („casting product image data for quality inspection"), 519 zdjęć dobrych i 781 wadliwych.
- Model: ResNet-18 z transfer learning, 6 epok, podział 70 / 15 / 15.
- Wynik na zbiorze testowym (195 zdjęć): czułość 98,3%, precyzja 100%, F1 99,1%.
- Model przepuścił 2 wadliwe detale ze 117 i w obu przypadkach był pewny swojej odpowiedzi, więc nie trafiły do strefy „niepewny". Dlatego potrzebna jest wyrywkowa kontrola detali uznanych za dobre.
- W tym zbiorze wadliwych zdjęć jest więcej niż dobrych, odwrotnie niż na prawdziwej linii.
