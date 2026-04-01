# Czemó size_t?

---

#### Co to tak na prawdę oznacza?
```c
int arr[100]
```

###### Zapis rozbity na czynniki
```c
Typ zmiennych w tablicy
 ↓
int arr[100] ← Wielkość tablicy
     ↑
    Nazwa tablicy [zmiennej]
```
`int` nie jest typem zmiennej `arr`<br>
`arr` jest wskaźnikiem (adresem) początku tablicy, której elementy to `int`

```c
int* wskaznik = arr; /*
 ↑
Wskaźnik do inta */
printf("%p", wskaznik); // 0x7ffc17a92610 (przykładowy adres)
```

---

#### Mamy jakiś wskaźnik, ale jak odczytać wartości z tablicy?
Standardowo w taki sposób odczytuje się n-ty element, licząc od zera
```c
wskaznik[n]
```
Ale jak to w rzeczywistości działa?
```c
*(wskaznik + n)
```
`wskaznik` to po prostu liczba, **adres**, do którego możemy dodać inną liczbę, **przesunięcie**

---

#### Co znaczy, że
```c
wskaznik[0] == *wskaznik

// Ponieważ
wskaznik[0] → *(wskaznik + 0) → *wskaznik
```
Przesunięcie w tym przypadku to zero, więc indeksowanie nic nie zmienia
#### Ale w ilości czego jest wyrażane przesunięcie?<br>Ilości bitów, bajtów, a może czegoś innego
Przesuwanie o bity nie jest możliwe, a przesuwanie o bajty prawie nigdy nie jest sensowne<br>
Dlatego przesunięcie jest wyrażone w liczbie elementów o które chcemy przesunąć wskaźnik
##### Kompilator sam obsłuży operacje na konkretnych wartościach
Tak można to zrobić manualnie
```c

//  Konwersja na liczbę by uniknąć standardowego przesuwania
//  ↓
(size_t)wskaznik + n * sizeof(int) /*
                     ↑
      Ilość bajtów które zajmuje n intów
```

---

### Czemu `size_t` zamiast `int`
||Format|Szerokość|
|---|---|---|
|Wskaźnik|Adres, nieujemna liczba całkowita|Zależna od systemu (najczęściej 64 bity)|
|`int`|Liczba całkowita, może przyjmować wartości ujemne|Zależna od kompilatora (najczęściej 32 bity)|


Z tego powodu do operacji na wskaźnikach powinno się używać `size_t`, typu z właściwościami wskaźnika

### Procesor nie może operować na liczbach jeśli
- Mają różne szerokości
- Jedna z nich może przyjmować wartości ujemne, a druga nie

### Problemy `int`
- Potrzebne dodatkowe konwersje przed operacjami
- Mniejszy zakres liczb

---

## Przykład pokazujący działanie pętli na niskim poziomie
###### Standardowy zapis
```c
int arr[100];
for (size_t i = 0; i < 100; i++) {
    printf("%p - %d\n", &arr[i], i);
    arr[i] = i;
}
```
#### Ten kod tworzy tablicę stuelementową<br>Przechodząc po wszystkich polach tablicy
- Wypisuje ich adres i indeks
- Przypisuje im wartość

### Tak na niskim poziomie będzie wyglądać podany kod
```c
int arr[100];
size_t i = 0;
loop:
    printf("%p - %d\n", (arr + i), i);
    *(arr + i) = i;
    i++;
    if (i < 100) goto loop;
```
### Przykładowy wynik tego kodu
```
0x7fffb36c74a0 - 0
0x7fffb36c74a4 - 1
0x7fffb36c74a8 - 2
0x7fffb36c74ac - 3
0x7fffb36c74b0 - 4
0x7fffb36c74b4 - 5
...
0x7fffb36c761c - 95
0x7fffb36c7620 - 96
0x7fffb36c7624 - 97
0x7fffb36c7628 - 98
0x7fffb36c762c - 99
```
