# Zadání: Náhoda za volantem (`0_cafe_mini_uncertainty`)

| | |
|---|---|
| ID | `0_cafe_mini_uncertainty` |
| Blok | C |
| Lokace / sublokace | Park / Ulice u kavárny → Zastávka MHD |
| Odpovídá | Ondra |
| Adresa (hlavní hra ji už zná) | https://0-cafe-mini-uncertainty.vercel.app |
| Stav podkladů | překopírováno z podkladů, „chce to ještě dotáhnout“ |
| Navazuje | Náhoda v davu mizí (`0_cafe_mini_shift_LLN`), Murphyho toast (`0_cafe_mini_LLN`) |

Shrnutí z přehledu: mřížka 10×10 nebo 20×20 aut, hráč tipuje, která nabourají. Opakovat 3×.
Pak slider výše škody 5 tis.–500 tis. Kč. Finále: animace 1 000 aut.

## Text zadání (převzato z podkladů, `minihry.xlsx`)

1. Bylo by 10×10 / 20×20 aut a hráč by měl tipovat, která nabourají. Označí je a pak se
   náhodně vyberou nějaká „reálně“ nabouraná (např. v počtech 3, 5, 4).
2. Třikrát to zopakovat. Na to navázat: „Konkrétní auto netrefíš. Ale víš, že nějaká vždy
   nabourají.“ „Teď už máš nějakou zkušenost – netipuj, jaká auta, ale kolik aut asi nabourá.“
3. Zase projet simulaci (klidně dát počet 3–5 jiný než tip). „Počet není vždy stejný, ale bývá
   více či méně podobný.“
4. „Ale ještě něco je nejisté… Tipni si: jak velká bude škoda.“
5. Zobrazí se postupně těch 3–5 aut po nehodě. K tomu slider 5 tis. – 500 tis. Kč.
6. Potom „odhalení reality“ různých hodnot, např. 12 tis., 310 tis., 95 tis.
7. A komentář strýčka: „Nejen že nevíme, které auto nabourá… ale ani jak velká bude škoda.“
8. Ještě finální ilustrace z 1 000 aut – pár jich nabourá, každé má jinou škodu… bez počítání,
   jen animace, aby bylo vidět kdo + kolik (nebo to projet 3krát)… bez součtů, průměrů atd.…
   šlo by i navázat na ZVČ (zákon velkých čísel).

## Otevřené (doplní autor po dohodě s Pavlem)

- Jak se počítá `successRate` (0–1). Hra nemá jednu „správnou“ odpověď; možnost je např.
  `1` = dokončeno a podrobnosti (tipy, skutečnost) ve výstupním `payload`.
- Jestli má hra nějaký vstupní `payload` / `level` (např. velikost mřížky).

Jak hru napojit na hlavní hru, otestovat a odevzdat: **Návod pro autory miniher** (pošle Pavel).
