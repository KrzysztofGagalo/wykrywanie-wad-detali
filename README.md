# Wykrywanie wadliwych detali na taśmociągu

Zadanie rekrutacyjne (zadanie 5, wariant 2): system wizyjny wykrywający detale wadliwe (rysy, pęknięcia, braki materiału).

Rozwiązanie to klasyfikacja zdjęć na dwie klasy, **dobry** i **wadliwy**, siecią ResNet-18 douczoną metodą transfer learning.

## Zawartość

- `wykrywanie_wad.ipynb` – notebook z całym procesem: dane, podział, augmentacja, model, uczenie, ocena, strefa „niepewny". Wyniki i wykresy są zapisane w pliku.

## Dane

Publiczny zbiór zdjęć odlewów z Kaggle: [Casting product image data for quality inspection](https://www.kaggle.com/datasets/ravirajsinh45/real-life-industrial-dataset-of-casting-product).
Autor zbioru: Ravirajsinh Dabhi, licencja CC BY-NC-ND 4.0 (użycie niekomercyjne).
Używane są tylko oryginalne zdjęcia 512×512 (519 dobrych, 781 wadliwych). Zbiór nie jest częścią repozytorium; notebook pobiera go przy pierwszym uruchomieniu do folderu `dane/`.

## Wyniki na zbiorze testowym (195 zdjęć)

| Metryka | Wynik |
|---|---|
| czułość | 98,3% |
| precyzja | 100% |
| F1 | 99,1% |

Wynik pochodzi z jednego przebiegu uczenia (6 epok, podział 70 / 15 / 15).

## Uruchomienie

```bash
pip install -r requirements.txt
```

```bash
jupyter notebook wykrywanie_wad.ipynb
```
