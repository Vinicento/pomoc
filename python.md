# PYTHON — ŚCIĄGA NA KOLOKWIUM
**Dwa przykładowe zestawy • 3 schematy: analiza danych / pliki + JSON / klasy**  
Kopiuj gotowy wzorzec i zmieniaj **nazwy, warunki, wzory, nazwy plików i klucze słownika**.

## 0. MINIMUM DO ZAPAMIĘTANIA

| Co chcesz zrobić? | Najprostszy zapis |
|---|---|
| Przejść po liście | `for x in lista:` |
| Zamienić tekst na liczbę | `float(x)` lub `int(x)` |
| Dodać element do listy | `lista.append(x)` |
| Długość / suma / min / max | `len(a)`, `sum(a)`, `min(a)`, `max(a)` |
| Średnia | `sum(a) / len(a)` (gdy lista niepusta) |
| Usunąć kropki i przecinki | `tekst.replace('.', '').replace(',', '')` |
| Małe litery / słowa | `tekst.lower()`, `tekst.split()` |
| Rozdzielić wiersz po znaku | `wiersz.split(',')`, `wiersz.split(':')` |
| Policzyć wystąpienia | `if x in licznik: licznik[x] += 1` / `else: licznik[x] = 1` |
| 3 najczęstsze (słownik) | `sorted(d.items(), key=lambda x: -x[1])[:3]` |
| Czy coś występuje w linii? | `if 'WARNING' in linia:` |
| Sformatować napis | `f'Wynik: {wynik}'` |
| Przypisać pola obiektu | `self.nazwa = nazwa` |
| Zwrócić wartość | `return wynik` (a `print` tylko wyświetla!) |

**Różnica:** `print(x)` = wyświetla; `return x` = oddaje wynik funkcji.  
**Wcięcia:** wszystko należące do `def`, `for`, `if`, `with`, `class` musi być wcięte.

---

## 1. LISTA LICZB — średnia, minimum, maksimum (zestaw PDF, zad. 1)

**Schemat:** konwersja → odrzucenie błędnych → statystyki → słownik.

```python
# zad1.py
def analizuj_pomiary(pomiary):
    dobre = []
    for x in pomiary:
        x = float(x)                  # działa dla 12.5 i "12.5"
        if x >= 0:                   # odrzuć ujemne
            dobre.append(x)

    if not dobre:                    # bez dzielenia przez zero
        return {"srednia": None, "minimum": None, "maksimum": None}

    return {
        "srednia": sum(dobre) / len(dobre),
        "minimum": min(dobre),
        "maksimum": max(dobre)
    }

print(analizuj_pomiary([12.5, -99, "14.2", 10, "-5.0", 13.3]))
# {'srednia': 12.5, 'minimum': 10.0, 'maksimum': 14.2}
```

**Zmieniasz przy analogicznym zadaniu:** `x >= 0` na inny warunek; dodajesz np. `"liczba": len(dobre)`; średnia zawsze `sum(...) / len(...)`.

## 2. ANALIZA TEKSTU — 3 najczęstsze słowa (zestaw JPG, zad. 1)

**Schemat:** małe litery → usuń interpunkcję → słowa → słownik zliczeń → sortowanie.

```python
# zad1.py
def analizuj_tekst(tekst):
    tekst = tekst.lower().replace(".", "").replace(",", "")
    slowa = tekst.split()            # lista słów
    licznik = {}                     # słowo -> liczba wystąpień

    for slowo in slowa:
        if slowo in licznik:
            licznik[slowo] += 1
        else:
            licznik[slowo] = 1

    return sorted(licznik.items(), key=lambda x: -x[1])[:3]

print(analizuj_tekst("Ala ma kota. Ala ma psa, Ala ma rybę."))
# [('ala', 3), ('ma', 3), ('kota', 1)]
```

**Rozszyfruj:** `.items()` = pary `(słowo, liczba)`; `x[1]` = liczba; `-x[1]` = od największej; `[:3]` = pierwsze 3.  
**Wariant:** usuwanie większej liczby znaków: `for znak in '.,!?;:': tekst = tekst.replace(znak, '')`.

---

## 3. PLIK TXT Z LOGAMI → JSON (zestaw JPG, zad. 2)

**Schemat:** czytaj linia po linii → zlicz WARNING → wyciągnij liczbę po `=` → maksimum → zapisz JSON.

```python
# zad2.py
import json

liczba_warning = 0
maksimum = None

with open("logi_zad2.txt", encoding="utf-8") as plik:
    for linia in plik:
        if "WARNING" in linia:
            liczba_warning += 1

        if "odczyt_sensora=" in linia:
            fragment = linia.split("odczyt_sensora=")[1]
            odczyt = float(fragment.split()[0])
            if maksimum is None or odczyt > maksimum:
                maksimum = odczyt

wynik = {"liczba_warning": liczba_warning, "maksimum": maksimum}
with open("wyniki.json", "w", encoding="utf-8") as plik:
    json.dump(wynik, plik, indent=4, ensure_ascii=False)
```

**Jak wyciąga liczbę?** Dla `... odczyt_sensora=83.3 V. ...`:  
`split('odczyt_sensora=')[1]` → `"83.3 V. ..."` → `.split()[0]` → `"83.3"` → `float(...)` → `83.3`.  
**Jeśli analogiczne zadanie:** podmień `WARNING`, `odczyt_sensora=`, nazwę JSON i wymagane klucze.

## 4. PLIK CSV (przecinek LUB dwukropek) → JSON (zestaw PDF, zad. 2)

**Schemat:** odczyt wierszy → wykryj separator → pola → filtr + suma → zapisz JSON.

```python
# zad2.py
import json

with open("produkty.csv", encoding="utf-8-sig") as plik:
    wiersze = plik.read().strip().splitlines()

separator = "," if "," in wiersze[0] else ":"
do_zamowienia = []
wartosc = 0

for wiersz in wiersze[1:]:            # [1:] pomija nagłówek
    pola = wiersz.split(separator)
    nazwa = pola[0]
    cena = float(pola[1])
    ilosc = int(pola[2])

    if ilosc < 5:
        do_zamowienia.append(nazwa)
    wartosc += cena * ilosc

raport = {"do_zamowienia": do_zamowienia,
         "wartosc_calkowita": wartosc}
with open("raport.json", "w", encoding="utf-8") as plik:
    json.dump(raport, plik, indent=4, ensure_ascii=False)
```

**Uwaga do źródła:** w PDF nagłówek ma 3 pola (`nazwa,cena,ilosc`), ale przykład `Chleb,4.50,10,15` ma 4. Kod zakłada 3 kolumny zgodne z opisem; ewentualną nadmiarową kolumnę ignoruje.  
**Wariant:** zamiast `ilosc < 5` dowolny warunek; zamiast `cena * ilosc` dowolny wzór. Jeśli CSV ma inne kolumny — zmień indeksy `pola[0]`, `pola[1]`, `pola[2]`.

---

## 5. KLASA PROSTOKĄT (zestaw JPG, zad. 3)

**Schemat:** `__init__` zapisuje pola → metody liczą → metoda zmienia pole → metoda wypisuje.

```python
# zad3.py
class Prostokat:
    def __init__(self, kolor, dlugosc, szerokosc):
        self.kolor = kolor
        self.dlugosc = dlugosc
        self.szerokosc = szerokosc

    def oblicz_pole(self):
        return self.dlugosc * self.szerokosc

    def oblicz_obwod(self):
        return 2 * (self.dlugosc + self.szerokosc)

    def zmien_dlugosc(self, nowa_dlugosc):
        self.dlugosc = nowa_dlugosc

    def wypisz(self):
        print(f"Prostokąt koloru {self.kolor}. "
              f"Obwód={self.oblicz_obwod()} pole={self.oblicz_pole()}")

moj_prostokat = Prostokat("Czerwony", 5, 4)
moj_prostokat.zmien_dlugosc(8)
moj_prostokat.wypisz()
# Prostokąt koloru Czerwony. Obwód=24 pole=32
```

**Ważna niezgodność w JPG:** opis konstruktora podaje kolejność `dlugosc, szerokosc, kolor`, a użycie to `Prostokat("Czerwony", 5, 4)`. Tu przyjęto **kolejność z przykładu użycia**. Gdy egzaminator wymaga kolejności z opisu, zmień tylko nagłówek na `def __init__(self, dlugosc, szerokosc, kolor):` i wywołanie na `Prostokat(5, 4, "Czerwony")`.

## 6. KLASA SAMOCHÓD (zestaw PDF, zad. 3)

```python
# zad3.py
class Samochod:
    def __init__(self, marka, model, rok_produkcji, przebieg):
        self.marka = marka
        self.model = model
        self.rok_produkcji = rok_produkcji
        self.przebieg = przebieg

    def jedz(self, km):
        self.przebieg += km

    def czy_klasyk(self):
        return 2025 - self.rok_produkcji > 25

    def wyswietl_info(self):
        print(f"Samochód: {self.marka} {self.model}, "
              f"Przebieg: {self.przebieg} km")

    def oblicz_sredni_roczny_przebieg(self):
        lata = 2025 - self.rok_produkcji
        return self.przebieg / lata if lata > 0 else 0

auto = Samochod("Toyota", "Corolla", 2017, 200000)
auto.jedz(500)
auto.wyswietl_info()
print("Klasyk:", auto.czy_klasyk())
print("Przebieg roczny:", auto.oblicz_sredni_roczny_przebieg())
# Samochód: Toyota Corolla, Przebieg: 200500 km
# Klasyk: False
# Przebieg roczny: 25062.5
```

**Uwaga:** używamy **2025**, bo tak nakazuje zadanie, nawet gdy aktualny rok jest inny.  
**Dlaczego `return`?** Zadanie wywołuje `print(auto.oblicz_sredni_roczny_przebieg())`; gdy metoda robi tylko `print`, zewnętrzny `print` pokaże też `None`.

---

## 7. UNIWERSALNE FRAGMENTY — WSTAW DO ANALOGICZNEGO ZADANIA

```python
# LICZNIK
ile = 0
for x in lista:
    if x > 10:
        ile += 1

# FILTR
wyniki = []
for x in lista:
    if x >= 0:
        wyniki.append(x)

# SUMA WARTOŚCI
suma = 0
for x in lista:
    suma += x

# MAXIMUM LICZONE PO KOLEI (bez wczytywania całej listy)
maksimum = None
for x in lista:
    if maksimum is None or x > maksimum:
        maksimum = x

# WYCINANIE: lista[0] pierwszy, lista[-1] ostatni
# lista[1:] wszystko oprócz pierwszego; lista[:3] trzy pierwsze

# ZAPIS TEKSTU (gdyby nie wymagali JSON)
with open("wynik.txt", "w", encoding="utf-8") as plik:
    plik.write(str(suma))
```

## 8. NAJCZĘSTSZE BŁĘDY (SPRAWDŹ PRZED ODDANIEM)

1. `=` przypisanie, `==` porównanie; `if x < 5:` wymaga dwukropka.
2. `self.` przed atrybutami obiektu; `self.metoda()` do wywołania innej metody.
3. `float("14.2")` działa, `float("14,2")` nie — najpierw `.replace(",", ".")`.
4. `sorted(d)` sortuje **klucze**; `sorted(d.items(), key=lambda x: -x[1])` sortuje po **licznościach**.
5. `read()` wczytuje cały plik; `for linia in plik:` czyta po jednej linii.
6. `"w"` nadpisuje plik; `"r"` czyta; nazwę pliku zapisuj dokładnie jak w zadaniu.
7. `json.dump(dane, plik, indent=4)` zapisuje JSON; `print(dane)` **nie** zapisuje pliku.
8. `min`, `max`, średnia wymagają danych; dla pustej listy obsłuż szczególny przypadek.
9. Skrypty na egzaminie: **osobno `zad1.py`, `zad2.py`, `zad3.py`**. Każdy plik samodzielny.

**Jak podejść do nowego zadania:** (1) Ustal, co wchodzi i co ma wyjść. (2) Wybierz jeden z 3 schematów. (3) Zmień nazwy i warunki. (4) Sprawdź przykład z polecenia. (5) Upewnij się, czy wymagają `print`, `return` czy zapisu do pliku.

*Źródła: przesłane materiały „PPD_kolokwium_gr16 (1) (1).pdf” oraz „gr_5 (1).jpg”.*

