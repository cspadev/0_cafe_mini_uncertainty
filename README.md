# Náhoda za volantem (`0_cafe_mini_uncertainty`)

Minihra pro Aktuárskou kariérní hru, Level 0, blok C (Park / Ulice u kavárny, Zastávka MHD). Verze 9: verze 8.3 po
auditech (likvidátorský sprint, test barvy, srovnání deníku s daty pojišťoven), z níž se část s 1 000 auty oddělila
do samostatné minihry `0_cafe_mini_shift_LLN` (Náhoda v davu mizí).

Hráč na parkovišti se 100 auty s povinným ručením zjistí, že konkrétní nehodu předpovědět nejde. Počet nehod v kole
odhadnout umí: před každým kolem zvolí tip a šířku pásma (úzké, střední, široké). Na stovce aut počet hodně kolísá,
proto se vyplatí široké pásmo. Výši škody odhaduje podle hlášení o nehodě (místo, rychlost, zranění; barva auta je
falešná stopa). V likvidátorském sprintu pak třídí hlášení do přihrádek na čas o závod se strýčkem a z výsledku vznikne
deník, podle kterého odhaduje rozložení škod.

Strýček hraje jako soupeř s lidskými chybami: u stovky aut tipuje jako minule (setrvačnost) a bere široké pásmo, u výše
škody čte hlášení dobře, ale u červených aut přidává polovinu a ve sprintu je posouvá o přihrádku výš (pověra),
u rozložení dává 5/4/1 podle dat pojišťoven. Ve sprintu jsou vždy dvě červená auta s úplně jinou škodou (test barvy),
takže pověru hráč uvidí na vlastních číslech. Hráč má navíc kartu **Likvidátor** (1×): u jednoho hlášení ukáže odhad
škody ±30 %, u hlášení se zraněním ±50 % (škoda na zdraví zůstává nejistá i po prohlídce).

Navazuje minihra `0_cafe_mini_shift_LLN` (Náhoda v davu mizí, zákon velkých čísel na 10 až 10 000 autech). Délka asi
3 minuty. Zadání, repliky a audit: dokumenty „Náhoda za volantem (0_cafe_mini_uncertainty)“ a „Audit minihry Náhoda
za volantem“ v projektu Aktuárská hra.

## Pro hlavní hru (souhrn podle návodu pro autory)

| | |
|---|---|
| ID minihry | `0_cafe_mini_uncertainty` |
| `level` | nepoužívá se (jedna obtížnost), jakákoli hodnota i chybějící `level` se ignoruje |
| `payload` vstup | `seed` celé číslo 1 až 2147483647 (výchozí náhodně), `duel` true / false (výchozí true); vše nepovinné, neplatné hodnoty se nahradí výchozími |
| `successRate` | $\min(1;\ \max(0;\ \text{body}) / 42)$, body za odhady ve fázích 2, 3a, 3b a 3c (nejvýš 54) |
| `payload` výstup | `{ version, skillPoints, duel, clover, f1, f2, f3a, sprint, f3c, f3cShare, cards, decisions, durationSec }`, podrobně v kapitole Výstup |
| Protokol | verze 1, `src/minihra-protokol.ts` beze změn z návodu (2. 11. 2026) |
| Build | `npm run build`, výstup ve složce `dist` (Vite) |

## Průběh

| Část | Co hráč dělá | Rozhodnutí |
|---|---|---|
| 1. Které auto | 2 kola: označí 5 aut, po pátém se kolo spustí samo | jen pro čtyřlístek, do bodů se nepočítá |
| 2. Kolik aut ze 100 | 3 kola: před každým kolem tip a šířka pásma, od 2. kola měkký časovač 8 s (plus doba na přečtení strýčkovy repliky, nejvýš 5 s) | 6 |
| 3a. Výše škody | 3 hlášení o nehodě, tip na logaritmickém posuvníku, karta Likvidátor | 3 + karta |
| 3b. Likvidátorský sprint | zkouška nanečisto, pak 8 hlášení; každé zatřídit do přihrádky podle 4 krátkých indicií, rychlost se počítá do bodů jako v Kahootu | 8 |
| 3c. Deník a rozložení | deník z 11 hlášení (3a a sprint), pak 3 kola: rozdělení 10 nehod do přihrádek, mezi koly lze změnit; nakonec srovnání podílu velkých škod: deník, 30 nehod ze 3c, data pojišťoven | 3 |
| Souhrn | body po částech, strýčkova věta o frekvenci a severitě, tlačítko Pokračovat | |

## Vstupy

| | |
|---|---|
| `level` | nepoužívá se, minihra má jednu obtížnost; hodnota se ignoruje |
| `payload.seed` | celé číslo 1 až 2147483647 pro opakovatelný průběh, výchozí náhodně |
| `payload.duel` | `false` vypne strýčka jako soupeře (a kartu Likvidátor), výchozí `true` |

Chybějící nebo neplatné hodnoty se nahradí výchozími. Minihra běží i bez `level` a bez `payload`.
Se semínkem je opakovatelné vše, co ovlivňuje body (počty, škody, barvy aut, odhad likvidátora).

Příklady pro tester: `{}`, `{"seed": 42}`, `{"seed": 42, "duel": false}`. Ověřené okrajové případy: chybějící
`level` i `payload`, `level` jako text nebo `NaN`, `payload` jako text, pole nebo `null`, nesmyslné hodnoty
(`{"seed": -5, "duel": "x"}`), druhý `INIT` uprostřed hry (hra začne znovu od úvodu), trojité ťuknutí
na Pokračovat (odejde právě jeden `FINISH`).

## Bodování

**Počty (fáze 2).** Pásmo kolem tipu má poloviční šířku $\max(m, \lfloor r \cdot \text{tip} \rfloor)$ aut.

| Pásmo | $r$ | $m$ | V pásmu | Mimo | Příklad tip 4 |
|---|---|---|---|---|---|
| Úzké | 0,15 | 0 | +4 | −1 | přesně 4 |
| Střední | 0,30 | 1 | +2 | −1 | 3 až 5 |
| Široké | 0,75 | 2 | +1 | 0 | 1 až 7 |

Očekávané body za kolo při tipu podle skutečné frekvence (simulace): úzké −0,04, střední 0,64, **široké 0,95**.

**Výše škody (fáze 3a).** Nejvýš 1,25krát víc nebo míň než skutečnost 3 body, nejvýš 1,5krát 2 body, nejvýš 2krát 1 bod.

**Likvidátorský sprint (fáze 3b), bodování jako v Kahootu.** Hlášení má jen 4 krátké indicie (kde, rychlost, co se
stalo, zranění) a barvu auta. Na každé hlášení je 9 s. Správná odpověď do 3 s dá 3 body, do 6 s 2 body, do 9 s 1 bod;
chyba i vypršený čas 0 bodů. Strýček se hodnotí stejně (odpovídá za 1,8 až 3,8 s, u červeného auta tipuje o přihrádku
výš). Sprint začíná zkouškou nanečisto bez času a bez bodů. Správná přihrádka se určuje podle skutečné škody, takže
i dokonalé čtení hlášení trefí asi 94 % (jednotlivá škoda zůstává nejistá). Pravidlo férovosti: kdo zvolí přihrádku
typickou pro scénář hlášení, ale skutečná škoda padne jinam (asi 6 % hlášení), dostane 1 bod a vysvětlení „Podle
hlášení správně, tahle škoda ale vyšla jinak“; série správných odpovědí se nepřeruší. Platí i pro strýčka.

Test barvy: ve sprintu jsou červená vždy právě dvě auta, ťukanec s nejmenší škodou a nejdražší velká bouračka (když
ve sprintu žádná není, nejdražší bouračka). Strýček se na ťukanci splete vždy, po sprintu se obě auta ukážou vedle sebe
(„Stejná barva, úplně jiná škoda“).

**Rozložení (fáze 3c).** Shoda 10 z 10 3 body, 9 z 10 2 body, 8 z 10 1 bod. Deník ukazuje pod přihrádkami dva pruhy,
podíl na počtu nehod a podíl na penězích z 11 viděných hlášení, a strýček to shrne čísly (např. „Velké nehody jsou jen
2 z 11, ale stojí 67 % všech peněz.“). Fáze 3a obsahuje vždy jednu velkou nehodu, takže pointa vyjde v každé hře.
Deník ale velké škody nadhodnocuje (v průměru 15 % nehod a 51 % peněz proti 9 % a 42 % v celém rozdělení), proto
u přihrádek vidí hráč vedle počtů z deníku i podíly pojišťoven (48 / 43 / 9 %) a strýček řekne, že deník je malý.
Po 3. kole se ukážou tři pruhy: deník, 30 nehod ze 3c a data pojišťoven (9 % nehod, 42 % peněz), s vysvětlením, že
jedna velká nehoda byla v deníku schválně a na pár desítkách nehod podíl pořád kolísá.

Fáze 1 se nepočítá (čistá náhoda), první trefa dává čtyřlístek. Skóre duelu = body za odhady. Při shodě rozhoduje menší
součet chyb tipů ve fázi 2, pak ve fázi 3a.

**`successRate`** $= \min\left(1, \dfrac{\max(0, \text{body})}{42}\right)$. Nejvyšší možný součet je 54, 42 bodů
odpovídá 95. percentilu optimální a rychlé hry (simulace 8 000 her, 42 a víc má asi 2 % her).

Simulace 8 000 her na typ hráče (čtenář pomalý: odpověď ve sprintu asi za 4 s a 88 % správně, běžný: 2,6 s a 95 %,
rychlý: 1,6 s a 97 %; mimo sprint hrají jako učící se hráč, ve fázi 3c podle podílů pojišťoven 5/4/1):

| Typ hráče | Výhry nad strýčkem | Body ve sprintu | successRate průměr |
|---|---|---|---|
| Naivní (tipuje a ťuká náhodně) | 0,3 % | asi 8 | 0,27 |
| Pomalý čtenář | 46 % | asi 15 | 0,68 |
| Běžný hráč | 92 % | asi 20 | 0,80 |
| Rychlý hráč | 99 % | asi 22 | 0,86 |
| Strýček | | asi 16,5 | |

AUC successRate naivní proti pomalému čtenáři 0,996. Bez části s 1 000 auty rozhoduje duel ještě víc rychlost čtení ve
sprintu; cíl auditu 55 až 70 % výher splňuje nejvíc pomalý čtenář. Rychlost strýčka (`UNCLE_SPRINT_MS` v
`src/game/sim.ts`) nebo váhu rychlosti doladit podle skutečných časů odpovědí z playtestu.

## Výstup

```json
{
  "version": 9,
  "skillPoints": { "points": 33, "ref": 42, "max": 54 },
  "duel": { "player": 33, "uncle": 30, "winner": "player", "tieBreak": null },
  "clover": 0,
  "f1": { "crashes": [4, 2], "hits": 0, "uncleHits": 0 },
  "f2": [{ "tip": 3, "band": "siroke", "crashes": 3, "points": 1, "uncleTip": 2, "uncleBand": "siroke", "unclePoints": 1 }],
  "f3a": [{ "scenario": "park-auto", "color": "červená", "tip": 11000, "actual": 14000, "points": 2, "uncleTip": 20000, "unclePoints": 2, "likvidator": false }],
  "sprint": {
    "items": [{ "scenario": "park-auto", "color": "červená", "actual": 15000, "bin": 0, "reportBin": 0, "answer": 0, "ms": 2313, "points": 3, "uncleAnswer": 1, "uncleMs": 2482, "unclePoints": 0 }],
    "points": 21, "unclePoints": 12, "colorTest": [2, 5]
  },
  "f3c": [{ "alloc": [5, 4, 1], "counts": [5, 5, 0], "overlap": 9, "points": 2, "uncleOverlap": 9, "unclePoints": 2 }],
  "f3cShare": { "counts": [15, 13, 2], "money": [214000, 820000, 690000] },
  "cards": { "likvidator": { "used": true, "car": 1 } },
  "decisions": 21,
  "durationSec": 180
}
```

- `f2` má 3 položky (kola), `f3a` 3 položky (hlášení), `sprint.items` 8 položek, `f3c` 3 položky (kola).
- Ve sprintu je `bin` správná přihrádka podle skutečné škody (0 ťukanec, 1 bouračka, 2 velká bouračka), `reportBin`
  přihrádka typická pro scénář hlášení (zdroj bodu za smůlu), `answer` volba hráče (`null` = vypršel čas) a `ms` doba
  odpovědi.
- `sprint.colorTest` jsou indexy dvou červených aut v `sprint.items` (test barvy), `null`, pokud ve sprintu chybí ťukanec
  nebo dražší hlášení.
- `f3cShare` jsou počty a součty škod (Kč) ťukanců, bouraček a velkých bouraček za všech 30 nehod fáze 3c.
- `band` je `uzke`, `stredni` nebo `siroke`.
- `duel`, `cards` a strýčkovy hodnoty jsou `null`, když je strýček vypnutý (`payload.duel = false`).
- `winner` je `player`, `uncle` nebo `tie`; `tieBreak` říká, co rozhodlo remízu na body (`f2`, `f3a`).
- `clover` = 1, pokud hráč ve fázi 1 trefil aspoň jedno nabourané auto (návrh: hlavní hra připíše čtyřlístek).
- Částky jsou v Kč, `crashes` jsou počty nabouraných aut v kole.
- `scenario` je ID hlášení (12 scénářů, 4 v každé přihrádce), seznam a typické škody jsou v `src/game/sim.ts`.

Při neočekávané chybě minihra pošle `ERROR` s kódem `INTERNAL`.

## Model

- Frekvence: binomické rozdělení $\mathrm{Bi}(100;\ 0{,}035)$ na parkoviště (data povinného ručení pro osobní auta,
  3,5 nehody na 100 aut za kolo, tj. za pojistné období), bez ořezu (ve fázi 1 aspoň 1 nehoda).
- Výše škody v deníku a ve fázi 3c: lognormální, $\sigma = 1{,}2$, useknuto na 5 tis. až 2 mil. Kč,
  $\mu = 10{,}2512$, takže podmíněný průměr je 62 tis. Kč. Podíly přihrádek 48 / 43 / 9 %, velké škody nesou asi 42 %
  peněz.
- Fáze 3a a sprint: 12 scénářů s typickou škodou 7 až 320 tis. Kč, škoda = typická × lognormální šum $\sigma = 0{,}4$
  (useknuto na 5 až 500 tis. Kč). Ve fázi 3a vždy jeden scénář z každé přihrádky, ve sprintu typ podle podílů
  48 / 43 / 9 %. Deník z 11 hlášení proto obsahuje v průměru 15 % velkých nehod s 51 % peněz (simulace 20 000 her).
- Karta Likvidátor: odhad = skutečná škoda × lognormální šum, $\sigma = 0{,}12$ a rozpětí ±30 % u škod na vozidlech,
  $\sigma = 0{,}2$ a ±50 % u hlášení se zraněním (rozpětí obsahuje skutečnou škodu asi v 97 %, resp. 96 % případů).

## Vývoj

```bash
npm install
npm run dev        # http://localhost:5173/?payload={"seed":42}
npm run build      # výstup do dist (Vite), tak ho staví Vercel
```

Komunikace s hlavní hrou je v `src/minihra-protokol.ts` (protokol verze 1, beze změn z návodu pro autory).
Simulace, bodování a strýček jsou v `src/game/sim.ts`, vstupy v `src/game/config.ts`.

`VITE_STANDALONE_PREVIEW=1` je jen pro sdílený náhled mimo hlavní hru (minihra pak nečeká na `INIT` ani když běží v
cizím iframu). Ve Vercelu pro ostrou verzi se nenastavuje.

## Kontrolní seznam

- Ovládání prstem na výšku, šířka 360 až 430 px, bez vodorovného posouvání. Bez drag & drop.
- Tlačítka a ovládací prvky mají aspoň 44 px. Výjimka: auta na parkovišti 10 × 10 mají na šířce 360 px asi 32 px.
  Přesnost tu nevadí, protože každé auto má stejnou šanci, takže přeťuknutí nic nezkazí. Druhé ťuknutí výběr ruší.
- Během běhu kola se stránka uvnitř minihry sama posune, aby parkoviště bylo vidět nad spodní lištou.
- Animace se řídí časem (CSS a `performance.now`), na 120 Hz displeji běží stejně rychle. Respektuje
  `prefers-reduced-motion`.
- Texty česky a genderově neutrálně (hráč), strýček mluví v mužském rodě.
- Písma jsou součástí projektu (Baloo 2, Nunito, sady latin a latin-ext včetně velkých písmen s diakritikou).
- Žádné přihlašování, klíče, `localStorage`, cookies, `alert()` ani zvuky. Grafika je kreslená v kódu (SVG).
- Hra má pevný konec: souhrn a tlačítko Pokračovat, které pošle právě jeden `FINISH`. V samostatném režimu se tlačítko
  Hrát znovu odemkne až po 1,2 s.
