# Design systém šachových artefaktů (v6)

Specifikace pro tvorbu interaktivních šachových artefaktů (simulace partií, zahájení, chytáků, taktických motivů, **koncovek a úloh**). Každý nový artefakt musí vizuálně i strukturálně odpovídat referenčním artefaktům `londynsky-system-kompletni.jsx` (více variant, rošáda), `damsky-gambit-kompletni.jsx` (plynulá animace figur) a `italska-partie-kompletni.jsx` (čtyři barevné režimy, čtyři varianty se společným začátkem).

**Changelog v6:** **plynulý pohyb figur je povinný v každém artefaktu a u každého zdroje dat** — spouštěčem byl Stockfish analyzátor, kde se deska překreslovala z FEN a nebylo vidět, odkud figura táhla. Nově: **skok na vzdálený tah vždy ukáže poslední tah** dvoufázově — skokem na tah před cílem, pak animace (§12.4); oddělení **logického kroku** a **kroku desky** (§12.4, §6); **identita figur z FEN / PGN / enginu** přes `buildTimeline` s pojistkou `syncToFen` (§12.5); **pravidla pro vanilla JS artefakty** bez Reactu — trvalá vrstva figur, mapa uzlů podle `id` (§12.6); **tahy klepnutím** s kratší animací (§12.7); **jediná tabulka časování `MOTION`** (§12.3); **braní mimochodem** jako standard přes `clear` (§6, §12); rozšířené kontrolní seznamy (§9, §12.10); nová migrace v5 → v6 (§23). Soubor specifikace se od v6 jmenuje **bez čísla verze** (`design-system-sachovych-artefaktu.md`) — verze drží git tagy (§21.1). Technicky: odstraněno zdvojené escapování (`\[`, `\*`, `\_`) z v5, které se zobrazovalo v ukázkách kódu.

**Changelog v5:** systém přestává být jen na zahájení. Nově **start z libovolné pozice (FEN)** a **proměna pěšce** (§13) — bez toho nešlo postavit koncovky; **větvené vedlejší varianty** se sémantickými druhy `variant` / `trap` / `calc`, nejvýš jedna úroveň zanoření (§14); **vrstvy zvýraznění** `zone` / `keys` / `arrows` s pevným barevným významem (§15); **počítadlo závodu** pro tempově kritické pozice (§16); **režim úlohy** s maskovanými žetony tahů (§17); **odznaky série a sdílený glosář** pro artefakty vydávané jako řada (§18); **statické mini-diagramy** v záložce Koncepty (§19); **rozhodovací tabulka funkcí** podle typu artefaktu (§20); **povinná patička s verzí design systému** (§21). Přepsáno **§10** — pořadí variant se nově řídí účelem artefaktu (repertoárový vs. chytákový), ne jedním univerzálním pravidlem. Rozšířen kontrolní seznam §9.

**Changelog v4:** **čtyři barevné režimy místo dvou** — dvě barevné rodiny (teplá a modrá), každá v tmavé a světlé variantě: **Modrá břidlice** (tmavá) a **Modrý papír** (světlá) (§2); přepínač zůstává **jedno tlačítko se symboly ☀/☾**, cykluje všemi čtyřmi režimy a **mění barvu symbolu** podle aktivního režimu — token `switchIcon` (§2, §5); kontrola čitelnosti figur ve všech čtyřech režimech (§9).

**Changelog v3:** plynulá animace tahů jako povinný standard (§12); přechod z polohového modelu desky na model figur s identitou (§6); renderování figur jako pohyblivých skupin `<g>` (§4); FEN se generuje z odděleného pomocníka `boardAt()` (§11).

**Changelog v2:** oprava otáčení souřadnic (§4), podpora rošády v datovém modelu (§6), pravidla pro přepínač variant (§10), export FEN a PGN jako dvě tlačítka na konci (§11), vynucená textová podoba figur proti emoji renderingu (§4).

---

## 1. Technologie

* **React, jeden soubor `.jsx`**, default export `App` bez povinných props
* Pouze `useState`, `useMemo` a `useRef` (časovače tlačítek „Zkopírováno", `requestAnimationFrame` pro dvoufázový skok §12.4), žádné externí knihovny
* **Výjimka — nástroje** (analyzátor, trenažér s enginem): smí být čisté HTML + JS s knihovnou pro pravidla (např. chess.js) a enginem; vzhled, režimy a animace se ale řídí touto specifikací beze změny (§12.6)
* **Inline styly** (JS objekty), žádný Tailwind
* Šachovnice jako **SVG** generované v JS (žádné obrázky)
* **Animace přes CSS `transition`** na `transform` a `opacity` — žádné animační knihovny; `requestAnimationFrame` jen pro spuštění dvoufázového skoku (§12.4), nikdy pro samotný pohyb
* Fonty přes Google Fonts import v `<style>` bloku
* Artefakt vždy vytvořit jako **skutečný soubor** (create_file + present_files), nikdy jako markdown blok
* Název souboru: česky, kebab-case, bez diakritiky (např. `past-v-italske-partii.jsx`)

## 2. Barevné režimy (čtyři + přepínač)

Vše čerpá z objektu `THEMES = { dark, light, blue, blueLight }`; komponenta drží `mode` ve stavu a `const T = THEMES[mode]`. **Žádná barva se nezapisuje natvrdo mimo tento objekt** — jinak nový režim nikdy nebude vypadat dobře.

Každý režim má **stejnou sadu 22 tokenů** (včetně `switchIcon` — barvy symbolu přepínače). Přidání režimu = přidání jednoho klíče do `THEMES`, nic víc.

|Režim|Klíč|Kdy ho zvolit|
|-|-|-|
|Tmavý luxus|`dark`|**výchozí** — večerní studium, zlato na tmavém dřevě|
|Světlý|`light`|denní světlo, tisk, screenshoty do dokumentů|
|Modrá břidlice|`blue`|chladná alternativa k tmavému; noční modrá deska|
|Modrý papír|`blueLight`|denní světlo v modré rodině; shodný s webovou aplikací pro šachová zahájení|

### Tmavý režim (výchozí)

|Token|Hex|Použití|
|-|-|-|
|bg|`#1A1208`|pozadí stránky|
|headerGrad|`linear-gradient(180deg,#221508,#1A1208)`|hlavička|
|border|`#3A2A10`|rámečky, oddělovače panelů|
|panel|`#2A1C0A`|karty, panely, tlačítka|
|divider|`#251A08`|vnitřní oddělovače řádků|
|gold|`#D4A84B`|primární akcent, nadpisy, aktivní prvky|
|goldDim|`#7A5820`|eyebrow texty, popisky KAPITÁLKAMI|
|textBody|`#C8A060`|běžný text komentářů|
|muted|`#806040`|sekundární text, neaktivní prvky|
|boardWrap|`#110C04`|podklad kolem desky|
|boardFrame|`#3A2410`|rám šachovnice|
|sqLight / sqDark|`#D6AF7B` / `#8A5A28`|světlá / tmavá pole (teplé dřevo)|
|coord|`#8A5820`|souřadnice a–h, 1–8|
|wFill / wStroke|`#F7EAD0` / `#2A1A08`|bílé figury (výplň / obrys)|
|bFill / bStroke|`#150D04` / `#C89858`|černé figury (výplň / obrys)|
|lastFrom / lastTo|`rgba(255,215,0,0.22)` / `rgba(255,215,0,0.38)`|zvýraznění posledního tahu|
|mark|`#C0392B`|červené kroužky hrozeb/cílů|
|switchIcon|`#E8D5A9`|**béžový** symbol ☀ v přepínači|

### Světlý režim

|Token|Hex|
|-|-|
|bg|`#F1E7D4`|
|headerGrad|`linear-gradient(180deg,#EADCC2,#F1E7D4)`|
|border|`#C9B48C`|
|panel|`#FBF4E4`|
|divider|`#E4D5B6`|
|gold|`#8A5A18`|
|goldDim|`#A5854E`|
|textBody|`#5A4526`|
|muted|`#96805C`|
|boardWrap|`#E2D3B4`|
|boardFrame|`#7A5220`|
|sqLight / sqDark|`#F0DCB2` / `#B5813F`|
|coord|`#7A5A28`|
|wFill / wStroke|`#FDF7E8` / `#3A2508`|
|bFill / bStroke|`#221302` / `#B58A48`|
|lastFrom / lastTo|`rgba(190,130,15,0.25)` / `rgba(190,130,15,0.45)`|
|mark|`#B03020`|
|switchIcon|`#6B4423` — **hnědý** symbol ☾|

### Modrý režim — „Modrá břidlice"

Chladný protipól tmavého luxusu: břidlicově modré panely, deska ve stylu Lichess (šedomodrá), akcent světle modrý. Červené kroužky hrozeb jsou tu nahrazeny **teplou oranžovočervenou**, protože na modrém podkladu je vidět mnohem líp než čistá červená.

|Token|Hex|
|-|-|
|bg|`#101A24`|
|headerGrad|`linear-gradient(180deg,#16232F,#101A24)`|
|border|`#24384A`|
|panel|`#1A2836`|
|divider|`#16222E`|
|gold *(akcent)*|`#7FB3D5`|
|goldDim|`#46708C`|
|textBody|`#A8C4D8`|
|muted|`#6B8598`|
|boardWrap|`#0A1219`|
|boardFrame|`#24384A`|
|sqLight / sqDark|`#DEE3E6` / `#8CA2AD`|
|coord|`#5C7E96`|
|wFill / wStroke|`#F4F8FA` / `#12222E`|
|bFill / bStroke|`#0E1A24` / `#9CC0D8`|
|lastFrom / lastTo|`rgba(127,179,213,0.22)` / `rgba(127,179,213,0.42)`|
|mark|`#E4572E`|
|switchIcon|`#8FC7E8` — **ledově modrý** symbol ☀|

### Světlý modrý režim — „Modrý papír"

Světlá polovina modré rodiny: skoro bílé pozadí, tmavě modrý inkoust, deska v chladné modrošedé. Rozdíl proti **světlému** režimu je v teplotě, ne v jasu: světlý je krémově zlatý (teplá rodina), Modrý papír je studený a technický. Dvojice `blue` + `blueLight` k sobě patří stejně jako `dark` + `light`.

|Token|Hex|
|-|-|
|bg|`#F4F6F9`|
|headerGrad|`linear-gradient(180deg,#FAFBFD,#F4F6F9)`|
|border|`#C9D5E4`|
|panel|`#FFFFFF`|
|divider|`#E4EBF3`|
|gold *(akcent)*|`#22508F`|
|goldDim|`#7C8FA8`|
|textBody|`#2C3E56`|
|muted|`#8595A8`|
|boardWrap|`#E6ECF3`|
|boardFrame|`#C3D0E0`|
|sqLight / sqDark|`#E8EEF6` / `#8CA6C4`|
|coord|`#6F8299`|
|wFill / wStroke|`#FFFFFF` / `#1B2A44`|
|bFill / bStroke|`#1B2430` / `#B9CBDD`|
|lastFrom / lastTo|`rgba(34,80,143,0.18)` / `rgba(34,80,143,0.36)`|
|mark|`#B03A2E`|
|switchIcon|`#22508F` — **tmavě modrý** symbol ☾|

### Přepínač režimů

**Jedno kulaté tlačítko 34 px vpravo nahoře v hlavičce se symboly ☀ / ☾**, které cykluje čtyřmi režimy a mění barvu symbolu podle tokenu `switchIcon`.

```jsx
const MODES = [
  { key: 'dark',  label: 'Tmavý luxus',     icon: '☀' },
  { key: 'light', label: 'Světlý',          icon: '☾' },
  { key: 'blue',  label: 'Modrá břidlice',  icon: '☀' },
  { key: 'blueLight', label: 'Modrý papír', icon: '☾' },
];
const next = () => MODES[(MODES.findIndex(m => m.key === mode) + 1) % MODES.length];
```

**Pravidla:**

* **Symbol** patří k *aktuálnímu* režimu: tmavé režimy (`dark`, `blue`) nesou ☀, světlé (`light`, `blueLight`) ☾
* **Barva symbolu** = `T.switchIcon`
* Pořadí cyklu je vždy `dark → light → blue → blueLight → dark`, aby bylo předvídatelné
* `title` a `aria-label` pojmenovávají **cíl** kliknutí: `Přepnout na: Modrá břidlice`
* Rámeček a pozadí tlačítka berou `T.border` a `T.panel`
* výchozí `useState('dark')`
* přepnutí režimu **nesmí** resetovat `step`, `vIdx`, `flipped` ani stav režimu úlohy (§17)

## 3. Typografie

* **Nadpisy, názvy tahů, záložky:** `'Playfair Display', Georgia, serif` (400/700)
* **Text komentářů:** `'Crimson Text', Georgia, serif`, 15 px, line-height 1.68
* **Notace, souřadnice, žetony tahů:** `monospace`, 11 px
* **Eyebrow / štítky sekcí:** 11–12 px, `letterSpacing: 3–5`, KAPITÁLKY, barva goldDim
* Import: `@import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;700&family=Crimson+Text:ital@0;1&display=swap');`

## 4. Šachovnice (SVG)

* Konstanty: `SQ = 46` (pole), `P = 26` (okraj pro souřadnice), šířka `P*2 + 8*SQ`
* Řádek 0 = 8. řada (černý nahoře), sloupec 0 = sloupec a; pomocníci `px(c)`, `py(r)` vracejí **levý horní roh pole** a počítají s otočením (`flipped`)
* **Pořadí vrstev v SVG (aktualizováno ve v5):**
`rám → pole → zvýraznění posledního tahu → zone → keys → souřadnice → skupina figur → kroužky marks → arrows`
Figury musí být NAD plošným zvýrazněním a POD kroužky hrozeb; šipky jsou úplně nahoře. Význam vrstev viz §15.
* Figury: **Unicode plné glyfy** `♚♛♜♝♞♟` pro obě strany, **každý s příponou `\uFE0E`** (VARIATION SELECTOR-15 — vynucená textová podoba; bez ní může iOS/WebKit vykreslit figury jako barevné emoji a barvení fill/stroke přestane fungovat), 34 px, explicitní `fontFamily="serif"`, barva přes `fill` + `stroke` s `paintOrder="stroke"`, `userSelect: 'none'`
* **Figury se nekreslí po polích, ale jako samostatné pohyblivé skupiny `<g>`** — viz §12. Glyf uvnitř skupiny sedí na souřadnicích 0,0 (`textAnchor="middle"`, `dominantBaseline="central"`, `y={1}` jako optická korekce) a o polohu se stará `transform` na skupině.
* **Souřadnice a otočení desky:** popisek sloupce `c` se kreslí na `x = px(c) + SQ/2` s textem `fileL(c)`; popisek řady `r` na `y = py(r) + SQ/2` s textem `rankL(r)`. Funkce `px()`/`py()` už otočení řeší — text se NESMÍ indexovat podruhé přes `flipped ? 7-c : c` (dvojitá inverze = popisky se při otočení nezmění, což je chyba). Správné chování: neotočeno a–h zleva a 1 dole; otočeno h–a zleva a 8 dole. **Test před odevzdáním: otoč desku a zkontroluj, že pole a1 má u sebe popisky „a" a „1" v obou orientacích.**
* Zvýraznění posledního tahu: poloprůhledný obdélník na výchozím (slabší) a cílovém (silnější) poli; **u rošády se zvýrazňují všechna čtyři pole** (král i věž)
* Hrozby/cíle: **přerušovaný kroužek v barvě `mark`** (`strokeDasharray="5 4"`, strokeWidth 2.5) na polích z `marks`
* Rám desky: obdélník boardFrame s `rx=4`, obal boardWrap s paddingem 6 a `borderRadius: 8`, `overflowX: auto` (iPhone!)

## 5. Rozvržení stránky (shora dolů)

1. **Hlavička** — na střed: eyebrow (téma), volitelný **odznak série** (§18), hlavní titul (24 px, bold, letterSpacing 2), podtitul s notací linie, volitelný **odznak hodnocení** `+−` / `−+` / `=` (§13.6); vpravo nahoře **kulaté tlačítko ☀/☾** cyklující čtyřmi režimy (§2)
2. **Přepínač variant** (jen pokud je variant více, viz §10)
3. **Šachovnice**
4. **Počítadlo závodu** (volitelné, jen u tempově kritických pozic — §16)
5. **Navigace** — kulatá tlačítka 38 px: `⏮ ◀ [stav] ▶ ⏭ ↕` (↕ = otočit desku); pilulka stavu ukazuje „Začátek" / „Půltah X / Y" / „Konec"
6. **Záložky** — `Strategie | Tah | Historie | Koncepty` (flex, aktivní má podtržení 2 px v barvě gold)
7. **Panel obsahu záložky** — panel s rámečkem, `borderRadius: '0 0 8px 8px'`, minHeight 120
8. **Žetony tahů** — flex-wrap řada tlačítek `⊙` (start) + `1.d4`, `1...d5`, … (monospace, pilulky 12 px radius); tahy po bodu rozvětvení mají zvýrazněnou barvu (goldDim rámeček, textBody písmo); v režimu úlohy jsou maskované (§17)
9. **Karta „CO SI ODNÉST"** — závěrečné shrnutí s 3 přenositelnými lekcemi
10. **Export FEN / PGN** — dvě pilulková tlačítka (§11)
11. **Patička s verzí** — jeden řádek drobným písmem (§21)

Obal obsahu: `maxWidth: 480, margin: '0 auto', padding: '0 12px'` — optimalizováno pro iPhone. Tlačítko přepínače má `position: absolute` (top 14, right 14), hlavička `position: relative` a titulek dostatečný postranní padding, aby se na úzkém displeji nepřekrývaly.

## 6. Datový model

```js
const MOVES = [
  { m: [zŘádek, zSloupec, naŘádek, naSloupec],  // řádek 0 = 8. řada
    san: 'Db6!',          // česká notace (viz §8)
    title: 'Výjimka z pravidla o dámě',   // krátký pedagogický titulek
    comment: '…',         // 3–6 vět, viz §7
    marks: [[6,1]],       // volitelně: pole hrozeb/cílů
    m2: [0,7,0,5],        // volitelně: druhý přesun v témže tahu — POUZE pro rošádu (věž)
    promote: 'Q',         // volitelně: proměna pěšce (§13.2)
    zone: { r0,c0,r1,c1 },// volitelně: obdélník polí (§15)
    keys: [[4,3],[4,5]],  // volitelně: klíčová pole (§15)
    arrows: [{ from:[6,4], to:[4,4], kind:'plan' }], // volitelně (§15)
    race: { w: 4, b: 5, note: '…' },   // volitelně (§16)
    side: [ … ],          // volitelně: vedlejší varianta (§14)
    hidden: true },       // volitelně: tah se maskuje v režimu úlohy (§17)
];
```

* **Rošáda:** zapisuje se jako tah krále v `m` + tah věže v `m2`; `san: 'O-O'` / `'O-O-O'`. Malá rošáda bílého: `m:[7,4,7,6], m2:[7,7,7,5]`; černého: `m:[0,4,0,6], m2:[0,7,0,5]`. Velká rošáda bílého: `m:[7,4,7,2], m2:[7,0,7,3]`; černého: `m:[0,4,0,2], m2:[0,0,0,3]`.
* **Pozice se počítá jako seznam figur s identitou:** `positionAt(moves, upTo)` vrací pole objektů `{ id, side, t, r, c, alive }` — viz §12. Matice 8×8 zůstává jen jako interní pomocník `boardAt()` pro generování FEN.
* **Výchozí pozice:** základní postavení přes `initPieces()`, libovolná pozice přes `piecesFromFEN(START_FEN)` — viz §13.1
* **Proměna pěšce:** klíč `promote` (§13.2) — od v5 součást standardu
* **Braní mimochodem (en passant)** — od v6 standard: tah nese `clear: [r,c]` = pole braného pěšce (jiné než cílové); `positionAt` na něm figuru označí `alive: false` (§12.1). Příklad: bílý pěšec e5 bere d5 mimochodem → `m: [3,4,2,3], clear: [3,3], san: 'exd6'`
* `STRATEGY` / `variant.strategy` — 2–4 odstavce úvodní strategie (záložka Strategie); víceodstavcový text psát s `\n\n` a renderovat přes `split('\n\n')`
* `CONCEPTS` — pole `{ name, text, diagram? }` s vysvětlením klíčových pojmů (sdílené všemi variantami; mini-diagram viz §19)
* `LESSONS` — pole 3 řetězců pro kartu „CO SI ODNÉST"
* `SERIES` — volitelně `{ name, index, total }` pro odznak série (§18)
* Stav: `step` (0 = výchozí pozice; logický krok pro texty a žetony), `board` = `{ step, instant }` (co je právě nakreslené na desce, §12.4), `flipped`, `tab`, `mode`, případně `vIdx` (varianta), `sideIdx` (vedlejší varianta, §14), `puzzle` + `revealed` (§17)

## 7. Interakce

* `go(s)`: ohraničí krok na 0..total, **při kroku > 0 automaticky přepne na záložku „Tah"** a desku posune podle §12.4 (±1 animovaně, vzdálený skok dvoufázově, krok 0 skokem) — všechna navigace (tlačítka, žetony, Historie) jde přes `go()`, nikdy přímo přes `setStep`
* Záložka **Tah**: ikona figury v rámečku + titulek `1...Db6! — Výjimka z pravidla o dámě` + řádek `Černý · Dáma d8 → b6` + komentář; u rošády se do řádku přidá i přesun věže; u proměny se přidá `→ Dáma`
* Záložka **Historie**: klikatelné řádky (číslo tahu, glyf figury, SAN, z→na), klik skočí na daný krok; **nehrané tahy se zobrazují ztlumeně** (`opacity: 0.45`), aktuální má podbarvení `divider`
* Figuru pro popisky tahu `i` získat přes pomocníka `pieceBefore(i)` = `positionAt(moves, i)` a najít živou figuru na výchozím poli
* Krok 0 zobrazuje v „Tah" výzvu „Stiskni ▶ pro první tah" (v režimu úlohy místo toho zadání úlohy — §17)
* Přepnutí varianty resetuje `step` na 0, `tab` na „Strategie" a zavírá otevřenou vedlejší variantu; přepnutí barevného režimu neresetuje nic

## 8. Obsahová pravidla (pedagogika)

* **Vše česky** — UI, komentáře, notace
* **Česká notace:** K = král, D = dáma, V = věž, S = střelec, J = jezdec; pěšec bez písmene; `x` braní, `O-O` rošáda, `=D` proměna, `!` silný tah, `!?` zajímavý, `?` chyba, `??` hrubá chyba
* Každý tah má **titulek** (proč je tah zajímavý) a **komentář 3–6 vět**, který vysvětluje *proč*, ne jen *co*; tato délka je ověřeně „tak akorát". U opakovaných/známých tahů (společný začátek variant) stačí 1–2 věty s odkazem na podrobnou variantu.
* U klíčových tahů vysvětlit vedlejší varianty a pasti přímo v komentáři (např. „8.exd4? f6! 9.Sg3 e5!") nebo je vytáhnout do větve `side` (§14)
* Odborný termín při prvním výskytu česky s anglickým ekvivalentem v závorce: mezitah (zwischenzug), vidlička (fork), přední hlídka (outpost)…
* **Zugzwang, opozice a klíčová pole** se v koncovkách pojmenovávají výslovně — jsou to místa, kde se úloha láme
* `marks` používat u tahů, kde je hrozba/cíl (napadená figura, slabé pole, pole vidličky)
* Propojovat s Radovanovými principy: „Proč soupeř zahrál ten tah?", centrální kontrola, včasná rošáda, tempo
* Závěrečná karta: **3 přenositelné lekce** + poznámka o relevanci pro úroveň ~400–600
* Tón: tykání, přátelský, občas řečnická otázka, žádné povýšenectví

## 9. Šachová přesnost a kontrola před odevzdáním

* U každého tahu ověřit legálnost: výchozí pole obsahuje správnou figuru, cílové pole je dosažitelné (jezdec ±(1,2)/(2,1), diagonály střelců pole po poli, volná cesta u dlouhých tahů)
* Braní pěšcem jen šikmo VPŘED (bílý nahoru = klesající index řádku); pěšec nikdy nebere do strany ani dozadu
* Rošáda: pole mezi králem a věží musí být prázdná; v ukázkových liniích nerošovat přes napadená pole
* `marks` musí odpovídat skutečně kontrolovaným polím (jezdec na d4 kontroluje b3, b5, c2, c6, e2, e6, f3, f5)
* **Otočení desky:** zkontrolovat popisky souřadnic v obou orientacích (viz §4)
* **Barevné režimy:** projít **všechny čtyři** a ověřit, že (a) bílá i černá figura je rozeznatelná na světlém i tmavém poli, (b) zvýraznění posledního tahu je vidět, ale nepřebíjí figuru, (c) kroužek `mark` je zřetelný proti desce, (d) souřadnice jsou čitelné, (e) nikde neprosvítá barva zapsaná natvrdo mimo `THEMES`, (f) symbol přepínače je ve své barvě `switchIcon` dobře vidět proti `panel` daného režimu, (g) **vrstvy `zone`, `keys` a `arrows` jsou čitelné a nepřebíjejí figury** (§15)
* **Animace:** projít celou variantu tam i zpět a ověřit, že (a) žádná figura „neteleportuje", (b) braná figura se plynule vytratí a při kroku zpět zase objeví, (c) při rošádě se hýbe král i věž, (d) přepnutí varianty pozici přepne skokem, ne přeletem figur (§12), (e) **proměna nemění identitu figury** — glyf se přebarví, pohyb doběhne (§13.2), (f) **klik na vzdálený žeton / řádek Historie / ⏭ ukáže poslední tah jako pohyb** z výchozího pole (§12.4), (g) zvýraznění posledního tahu se objeví současně s pohybem, ne dřív, (h) braní mimochodem nechá zhasnout správného pěšce, (i) u nástrojů s daty z FEN / PGN / enginu projít vložené PGN tam i zpět a ověřit totéž (§12.5–12.6), (j) rychlé opakované klikání neskončí v rozporu mezi deskou a textem
* **Start z FEN (§13):** artefakt startuje z `START_FEN`, ne ze základního postavení; **číslo prvního tahu a strana na tahu odpovídají FEN** (§13.3); počty figur sedí se zadaným řetězcem — počítat pole po poli
* **PGN u nestandardního startu** obsahuje `[SetUp "1"]` a `[FEN "…"]` a naimportuje se na Lichess do správné pozice (§13.4)
* **Odznak hodnocení** (`+−` / `−+` / `=`) sedí se skutečným výsledkem linie (§13.6)
* **Vedlejší varianty (§14):** každá větev je legální nad pozicí, ze které odbočuje; návrat do hlavní linie vrátí desku do stavu před odbočkou
* **Režim úlohy (§17):** maskované žetony neprozrazují tah ani počtem znaků; po odhalení se stav dá vrátit zpět
* **Patička (§21):** obsahuje verzi design systému, podle které byl artefakt skutečně postaven (ne tu nejnovější), název souboru sedí s `ARTEFAKT.nazev` a řádek je čitelný ve všech čtyřech režimech
* FEN ze screenshotů nikdy nečíst — pozice stavět jen z ověřené posloupnosti tahů nebo z FEN dodaného Radovanem

## 10. Varianty šablony podle typu obsahu

* **Jedna linie (zahájení, taktický motiv, koncovka):** bez přepínače variant
* **2 varianty (chyták/past, srovnání):** dvě pilulky vedle sebe (`flex: 1`), každá s názvem + notací větve
* **3+ variant:** mřížka pilulek 2×2 (`flexWrap`, `flex: '1 1 45%'`). Pořadí dlaždic = pořadí čtení (vlevo nahoře → vpravo dole) a řídí se **účelem artefaktu**:

  **A · Repertoárový** (otázka „co mám hrát?") — zahájení, systém, plán:

  1. ideál / hlavní plán za hráče, jehož repertoár se učí
  2. hlavní teoretická linie
  3. nejčastější obrana soupeře
  4. trest za typickou chybu v té obraně

  **B · Chytákový** (otázka „jak se to láme?") — past, taktický motiv:

  1. past v ideálním průběhu
  2. hlavní obrana
  3. lepší obrana
  4. plán proti obraně

  Rozhodovací kritérium: pokud artefakt slouží k přípravě na vlastní partie, je repertoárový (A); pokud ukazuje jeden konkrétní úder, je chytákový (B).

  Společné pro obě: **první dlaždice = to, co uživatel uvidí nejčastěji; poslední = nejpodmíněnější linie** (závisí na chybě soupeře). Varianta s trestem musí v komentáři jmenovitě odkázat na variantu se správnou obranou a naopak — jinak si uživatel odnese past bez protijedu.

* **Společný začátek variant:** definovat jednou jako `COMMON = [...]` a v každé variantě `moves: [...COMMON, ...vetev]`; komentáře společných tahů stručné, plné vysvětlení v hlavní variantě; `BRANCH_AT = COMMON.length` pro barevné odlišení žetonů
* **Rozbor celé partie:** stejná šablona; u dlouhých partií komentovat jen klíčové momenty, ostatní tahy jednořádkově
* **Koncovka / úloha:** typicky jedna hlavní linie + vedlejší varianty přes `side` (§14), ne přes přepínač variant — hlavní linie je jen jedna, alternativy jsou odbočky, ne rovnocenné volby
* Referenční artefakty: `italska-partie-kompletni.jsx` (typ A, čtyři varianty), `londynsky-system-kompletni.jsx` (struktura variant), `damsky-gambit-kompletni.jsx` (animace); pro nástroje ve vanilla JS `stockfish-analyzator` po přestavbě na v6 (§23)

## 11. Export FEN a PGN

Na úplném konci stránky, **pod kartou „CO SI ODNÉST"**, jsou pouze **dvě pilulková tlačítka** vedle sebe (`flex: 1`, gap 8) — žádné textové boxy s FEN/PGN ani vysvětlující odstavec. Každé tlačítko má tučný hlavní popisek a malý podtitulek: „Kopírovat FEN" / „aktuální pozice" a „Kopírovat PGN" / „celá varianta". Po kliknutí tlačítko na 1,5 s zobrazí „✓ Zkopírováno" (časovač držet v `useRef`, aby ho rychlé opakované kliknutí nerozbilo).

* **FEN** = aktuální pozice dle `step`, generuje `fenAt(moves, step)` nad pomocníkem `boardAt(moves, step)`. Strana na tahu z parity `step` **posunuté podle výchozího FEN** (§13.3); rošádová práva KQkq se ruší pohybem Z výchozího pole krále/věže i braním NA něm (kontrolovat i `m2`) a u nestandardního startu se berou z `START_FEN`; en passant = pole PŘESKOČENÉ pěšcem při dvojkroku v bezprostředně posledním tahu; počítadlo půltahů se nuluje braním nebo tahem pěšce; číslo tahu = `fullmoveStart + floor((step + offset)/2)`.
* **PGN** = celá aktuální varianta, generuje `pgnOf(variantName, moves)`. Tagy `[Event]`, `[Site "Claude artefakt"]`, `[Result "*"]`, movetext končí `*`. **U nestandardního startu navíc `[SetUp "1"]` a `[FEN "…"]`** (§13.4). **Notace se převádí z české na anglickou** mapou `{ J→N, S→B, D→Q, V→R, K→K }` (Lichess/Chess.com české notaci nerozumí); tahy pěšců a `O-O` se nemění, u proměny se překládá i písmeno za rovnítkem (`=D` → `=Q`), anotace `!`, `!?` zůstávají.
* **Kopírování:** `navigator.clipboard.writeText` s fallbackem přes skryté `textarea` + `document.execCommand('copy')`.

---

## 12. Animace tahů (povinné od v3, rozšířeno ve v6)

Figury musí po šachovnici **plynule klouzat**, ne přeskakovat. Ověřená hodnota: **`transform 0.45s cubic-bezier(0.25, 0.8, 0.35, 1)`** — dost pomalé, aby šlo tah očima sledovat, dost rychlé, aby proklikávání variant neotravovalo.

**Proč je to povinné všude (poučení z v5):** v5 počítala s animací jen v React artefaktech s vlastním seznamem tahů. Stockfish analyzátor (vanilla JS, pozice z FEN a z enginu) desku po každém kroku překreslil celou znovu — figury přeskakovaly a u kliknutí na tah v tabulce nebylo vidět, **odkud figura táhla**. Zvýraznění `lastFrom` / `lastTo` samo nestačí: dvě podbarvená pole v plné pozici se přehlédnou, pohyb figury ne. Od v6 proto platí: **každá šachovnice v každém artefaktu animuje každý tah, který vede do zobrazené pozice** — bez ohledu na framework, zdroj dat (vlastní `MOVES`, PGN, FEN, engine, klepnutí uživatele) a způsob navigace.

### 12.1 Základní princip: identita figur

Animace funguje jen tehdy, když se mezi dvěma kroky **znovupoužije stejný DOM uzel** figury. To vylučuje kreslení desky po polích (`key={r}-${c}`) i mazání a nové vykreslení celé SVG — při tahu by zanikl jeden uzel a vznikl jiný a prohlížeč nemá co animovat.

Proto se drží **seznam figur s trvalým `id`**:

```js
function initPieces() {
  const back = ['R','N','B','Q','K','B','N','R'];
  const ps = [];
  const cnt = {};
  const add = (side, t, r, c) => {
    const k = `${side}${t}`;
    cnt[k] = (cnt[k] || 0) + 1;
    ps.push({ id: `${k}${cnt[k]}`, side, t, r, c, alive: true });
  };
  back.forEach((t, c) => add('b', t, 0, c));
  for (let c = 0; c < 8; c++) add('b', 'P', 1, c);
  for (let c = 0; c < 8; c++) add('w', 'P', 6, c);
  back.forEach((t, c) => add('w', t, 7, c));
  return ps;                               // 32 figur, každá s unikátním id
}

function positionAt(moves, upTo) {
  const ps = START_FEN ? piecesFromFEN(START_FEN) : initPieces();
  const at = (r, c) => ps.find(p => p.alive && p.r === r && p.c === c);
  for (let i = 0; i < upTo; i++) {
    const mv = moves[i];
    const [fr, fc, tr, tc] = mv.m;
    const cap = mv.clear ? at(mv.clear[0], mv.clear[1]) : at(tr, tc);  // clear = braní mimochodem (§6)
    if (cap) cap.alive = false;             // braná figura zůstává v poli, jen zhasne
    const pc = at(fr, fc);
    if (pc) { pc.r = tr; pc.c = tc; if (mv.promote) pc.t = mv.promote; }
    if (mv.m2) {                            // rošáda: druhý přesun (věž)
      const [f2r, f2c, t2r, t2c] = mv.m2;
      const rk = at(f2r, f2c);
      if (rk) { rk.r = t2r; rk.c = t2c; }
    }
  }
  return ps;
}
```

**Klíčová pravidla:**

* `key` v Reactu (resp. klíč v mapě uzlů ve vanilla JS, §12.6) je **vždy `p.id`**, nikdy index ani souřadnice
* braná figura se **nemaže** ze seznamu — dostane `alive: false`, zůstane na posledních souřadnicích a vyblede; jinak by zmizela skokem a při kroku zpět by se objevila bez přechodu
* pořadí uvnitř tahu: nejdřív zneškodnit branou figuru, pak posunout táhnoucí (a případně proměnit), pak `m2`
* `positionAt` je čistá funkce nad krokem → obalit `useMemo(() => positionAt(moves, boardStep), [vIdx, sideIdx, boardStep])` (`boardStep` viz §12.4)
* **Braní mimochodem je od v6 součást standardu** přes klíč `clear` (§6) — už to není „model neumí"

### 12.2 Renderování pohyblivé figury (React)

```jsx
<g key={`pieces-${vIdx}-${sideIdx}`}>
  {pieces.map(p => (
    <g key={p.id} style={{
      transform: `translate(${px(p.c) + SQ/2}px, ${py(p.r) + SQ/2}px)`,
      transition: instant || reduce ? 'none' : MOTION.move,
      opacity: p.alive ? 1 : 0,
      pointerEvents: 'none',
    }}>
      <text textAnchor="middle" dominantBaseline="central" y={1}
        fontSize="34" fontFamily="serif" paintOrder="stroke"
        fill={p.side === 'w' ? T.wFill : T.bFill}
        stroke={p.side === 'w' ? T.wStroke : T.bStroke}
        strokeWidth="1.2"
        style={{ userSelect: 'none' }}>{GLYPH[p.t]}</text>
    </g>
  ))}
</g>
```

* **CSS `transform` ve `style`, ne SVG atribut `transform`** — atribut se v Safari na iPhonu nepřechází spolehlivě; CSS varianta ano. Jednotky `px` jsou povinné.
* `translate` míří na **střed pole** (`px(c) + SQ/2`), proto glyf uvnitř sedí na 0,0 s `dominantBaseline="central"`
* `opacity` se přechází kratší dobu (0,35 s) než pohyb — braná figura zmizí dřív, než na ni útočník dorazí
* `pointerEvents: 'none'` — figury nejsou klikatelné; klepání na desku (§12.7) chytá vrstva polí pod nimi
* barvy figur berou tokeny `wFill/wStroke/bFill/bStroke` aktuálního režimu; **při změně režimu se `fill` nesmí animovat**
* `instant` je příznak „tahle změna se nemá animovat" — viz §12.4

### 12.3 Časování (jediný zdroj pravdy)

```js
const EASE = 'cubic-bezier(0.25, 0.8, 0.35, 1)';
const MOTION = {
  move: `transform 0.45s ${EASE}, opacity 0.35s ease`,   // tah v navigaci ▶ ◀, tah enginu, skok (§12.4)
  user: `transform 0.22s ${EASE}, opacity 0.18s ease`,   // tah, který uživatel sám udělal klepnutím (§12.7)
};
```

|Situace|Přechod|Proč|
|-|-|-|
|▶ / ◀ o jeden půltah (vpřed i vzad)|`MOTION.move`|základní případ — uživatel sleduje, odkud a kam|
|skok na libovolný tah (žeton, Historie, řádek tabulky, bod grafu, ⏭)|skokem na tah před ním, pak `MOTION.move`|viz §12.4|
|tah soupeře / enginu jako odpověď na tah uživatele|`MOTION.move`|uživatel ho nečeká, musí ho vidět|
|tah uživatele klepnutím na desce|`MOTION.user`|uživatel ví, odkud táhl; dlouhá animace by zdržovala|
|⏮, krok 0, přepnutí varianty, otevření/zavření větve, načtení nové partie nebo FEN|bez animace (`instant`)|pozice spolu nesouvisí, přelet figur by mátl|
|otočení desky ↕|`MOTION.move`|čitelný „převrat" desky|
|změna barevného režimu|žádný přechod barev|mění se jen barvy|

Jiné časy než v této tabulce se nepoužívají — i drobná odchylka mezi artefakty série je vidět.

### 12.4 Skok na vzdálený tah: vždy ukázat poslední tah

Do v5 se při skoku přes více tahů nechaly figury letět přímo do cílové pozice. V praxi (klik na řádek tabulky v analyzátoru, na žeton, na bod v grafu) se tím ztratí to nejdůležitější: **který tah do pozice vedl a odkud figura přišla**. Při skoku se dvě tři figury pohnou naráz a oko nepozná, která táhla naposledy.

**Pravidlo v6:** na cílový krok `k > 0` se přejde **dvoufázově**:

1. deska se **bez animace** postaví do pozice `k − 1` (tah před cílem),
2. v dalším snímku se **animovaně** zahraje tah `k` — figura viditelně odjede ze svého pole.

Výjimky: posun o ±1 se animuje přímo (i dozadu — figura se vrátí na výchozí pole); cíl `k = 0` a všechny situace „bez animace" z tabulky §12.3 se přepnou skokem.

Proto se od v6 odděluje **logický krok** (texty, žetony, záložky — mění se hned) od **kroku desky** (co je zrovna nakreslené):

```jsx
const [step, setStep] = useState(0);                         // logický krok
const [board, setBoard] = useState({ step: 0, instant: true });  // krok desky
const raf = useRef(0);

const go = (s) => {
  s = Math.max(0, Math.min(total, s));
  cancelAnimationFrame(raf.current);
  setStep(s);
  if (s > 0) setTab('tah');
  if (s === 0) { setBoard({ step: 0, instant: true }); return; }
  if (Math.abs(s - board.step) <= 1) { setBoard({ step: s, instant: false }); return; }
  setBoard({ step: s - 1, instant: true });                  // fáze 1: skokem na tah před cílem
  raf.current = requestAnimationFrame(() =>                  // dvojitý rAF = fáze 1 je opravdu vykreslená
    raf.current = requestAnimationFrame(() => setBoard({ step: s, instant: false })));
};

const pieces = useMemo(() => positionAt(moves, board.step), [vIdx, sideIdx, board.step]);
const instant = board.instant;
```

* **Zvýraznění `lastFrom` / `lastTo`, `marks`, `zone`, `keys`, `arrows` a počítadlo závodu** se berou z `board.step`, ne ze `step` — musí se objevit spolu s pohybem figury, ne o snímek dřív
* `requestAnimationFrame` je od v6 povolený **výhradně** pro tuto dvoufázovou změnu; samotná animace zůstává čisté CSS `transition`
* rychlé opakované klikání: `cancelAnimationFrame` ruší rozjetou fázi 2, takže deska nikdy neskončí v jiném kroku, než ukazují texty
* v režimu úlohy (§17) se odhalení tahu chová jako posun o +1

### 12.5 Identita figur z FEN, PGN a enginu (`buildTimeline`)

Artefakty, které netahají z vlastního pole `MOVES`, ale z dat zvenku (PGN vložené uživatelem, pozice z chess.js, hlavní varianta enginu, FEN), **nesmí stavět figury z FEN pro každý krok znovu** — každý krok by dostal nové identity a figury by přeskakovaly. Místo toho se identity předpočítají jednou pro celou partii:

```js
const FILES = 'abcdefgh';
const sq2rc = (sq) => [8 - Number(sq[1]), FILES.indexOf(sq[0])];   // 'e4' → [4, 4]

// jeden krok: prev = seznam figur (včetně mrtvých), mv = { from:'e2', to:'e4', promo:'q' } (UCI / chess.js)
function applyMove(prev, mv) {
  const ps = prev.map(p => ({ ...p }));
  const at = (r, c) => ps.find(p => p.alive && p.r === r && p.c === c);
  const [fr, fc] = sq2rc(mv.from), [tr, tc] = sq2rc(mv.to);
  const pc = at(fr, fc);
  if (!pc) return ps;                                           // nemá nastat — ohlídá syncToFen
  let cap = at(tr, tc);
  if (!cap && pc.t === 'P' && fc !== tc) cap = at(fr, tc);      // braní mimochodem
  if (cap) cap.alive = false;
  if (pc.t === 'K' && Math.abs(tc - fc) === 2) {                // rošáda: věž jede taky
    const rk = at(fr, tc > fc ? 7 : 0);
    if (rk) rk.c = tc > fc ? 5 : 3;
  }
  pc.r = tr; pc.c = tc;
  if (mv.promo) pc.t = mv.promo.toUpperCase();                  // id zůstává (§13.2)
  return ps;
}

// pojistka: srovná seznam s FEN, kdyby data obsahovala něco, co applyMove nečeká
let extraId = 0;
function syncToFen(ps, fen) {
  const want = piecesFromFEN(fen);                              // jen side, t, r, c
  const live = ps.filter(p => p.alive);
  const used = new Set();
  const rest = [];
  for (const w of want) {                                       // 1) sedí přesně → ponechat
    const hit = live.find(p => !used.has(p) && p.side === w.side && p.t === w.t && p.r === w.r && p.c === w.c);
    if (hit) used.add(hit); else rest.push(w);
  }
  for (const w of rest) {                                       // 2) nejbližší volná figura stejného druhu
    const cand = live.filter(p => !used.has(p) && p.side === w.side && p.t === w.t)
      .sort((a, b) => Math.max(Math.abs(a.r - w.r), Math.abs(a.c - w.c))
                    - Math.max(Math.abs(b.r - w.r), Math.abs(b.c - w.c)))[0];
    if (cand) { used.add(cand); cand.r = w.r; cand.c = w.c; }
    else { const n = { id: `x${++extraId}`, side: w.side, t: w.t, r: w.r, c: w.c, alive: true };
           ps.push(n); used.add(n); }                           // 3) nová figura se objeví prolnutím
  }
  live.forEach(p => { if (!used.has(p)) p.alive = false; });    // 4) co v FEN není, zhasne
  return ps;
}

// celá partie: timeline[k] = figury po k půltazích
function buildTimeline(startFen, moves) {                       // moves: [{ from, to, promo, after }]
  const tl = [piecesFromFEN(startFen)];
  moves.forEach(mv => tl.push(syncToFen(applyMove(tl[tl.length - 1], mv), mv.after)));
  return tl;
}
```

* `timeline` se počítá **jednou** po načtení partie (nebo po přidání tahu v analýze) — navigace pak jen vybírá `timeline[k]`
* protože mrtvé figury v seznamu zůstávají, **krok zpět vrátí branou figuru prolnutím** a identity sedí v obou směrech
* `syncToFen` je pojistka, ne hlavní mechanismus: při správných datech nic nemění. Když přesto zasáhne (neznámý formát, Chess960), deska zůstane **správná** a nanejvýš jedna figura se přesune „divně" — nikdy nevznikne špatná pozice
* volná analýza (uživatel táhne sám): nový tah = `timeline.push(syncToFen(applyMove(last, mv), fenPo))`; „Vrátit tah" = `timeline.pop()` a deska se animovaně vrátí
* hlavní variantu enginu (PV) lze přehrát stejně: `buildTimeline(fen, pvMoves)` — nová timeline = přepnutí **skokem**, pak animace po tazích

### 12.6 Vanilla JS artefakty (HTML bez Reactu)

Nástroje typu analyzátor jsou často čisté HTML + JS. Pravidla §12.1–12.5 platí beze změny, jen místo Reactu drží uzly figur **mapa podle `id`** a SVG se **nikdy nemaže celé**:

```js
// SVG se staví jednou, ve vrstvách (pořadí §4):
// gBoard (rám, pole, souřadnice) → gUnder (lastFrom/lastTo, zone, keys) → gPieces → gOver (marks, šach, arrows)
const nodes = new Map();                                        // id → <g>

function renderPieces(list, { instant = false, speed = MOTION.move } = {}) {
  const seen = new Set();
  for (const p of list) {
    seen.add(p.id);
    let g = nodes.get(p.id);
    if (!g) {                                                   // nová figura: postavit bez přechodu
      g = el('g', { 'class': 'piece' });
      g.appendChild(el('text', { 'text-anchor': 'middle', 'dominant-baseline': 'central', y: 1 }));
      gPieces.appendChild(g); nodes.set(p.id, g);
      g.style.transition = 'none';
    } else {
      g.style.transition = instant || REDUCE ? 'none' : speed;
    }
    const { x, y } = center(p.r, p.c);                          // střed pole, respektuje flipped
    g.style.transform = `translate(${x}px, ${y}px)`;
    g.style.opacity = p.alive ? 1 : 0;
    const t = g.firstChild;
    t.setAttribute('class', 'pc ' + (p.side === 'w' ? 'pw' : 'pb'));
    t.textContent = GLYPH[p.t] + '\uFE0E';
  }
  for (const [id, g] of nodes) if (!seen.has(id)) { g.remove(); nodes.delete(id); }
}

function showStep(k, { reset = false } = {}) {                  // stejná logika jako go() v §12.4
  cancelAnimationFrame(rafId);                                  // reset = nová partie / nová timeline
  const from = S.boardStep;
  if (k === 0 || reset) { draw(k, { instant: true }); return; }
  if (Math.abs(k - from) <= 1) { draw(k); return; }
  draw(k - 1, { instant: true });
  gPieces.getBoundingClientRect();                              // vynutí vykreslení fáze 1
  rafId = requestAnimationFrame(() => draw(k));
}
// draw(k, opt) = S.boardStep = k; překreslí gUnder + gOver pro krok k a zavolá renderPieces(timeline[k], opt)
```

* `gBoard` se překresluje jen při otočení desky; `gUnder` a `gOver` se smějí mazat a stavět znovu při každém kroku; **`gPieces` nikdy**
* `getBoundingClientRect()` mezi fázemi je nutný — bez něj prohlížeč obě změny sloučí a fáze 1 se neprojeví
* nový uzel se postaví s `transition: 'none'`, aby nepřiletěl z rohu desky (0,0)
* CSS: `.piece { will-change: transform; }` — plynulejší na iPhonu

### 12.7 Tahy klepnutím na desku

U artefaktů, kde uživatel táhne sám (volná analýza, trenažér):

* klepnutí na pole chytá vrstva polí (`gBoard`), figury mají `pointer-events: none`
* po platném tahu se figura přesune s `MOTION.user` (0,22 s); odpověď enginu nebo soupeře, pokud přijde, s `MOTION.move` až **po doběhnutí** tahu uživatele (nejdřív za 250 ms), aby dva pohyby nesplynuly v jeden
* nelegální klepnutí figurou nehýbe — jen zruší výběr

### 12.8 Zvláštní případy

|Situace|Chování|Řešení|
|-|-|-|
|**Rošáda**|král i věž se hýbou zároveň|`m2` v `positionAt` / detekce v `applyMove`; obě dvojice polí zvýraznit (§4)|
|**Proměna pěšce**|glyf se mění uprostřed pohybu|`promote` / `promo` přepíše `p.t`, `id` zůstává → figura dojede a přebarví se (§13.2)|
|**Braní mimochodem**|braný pěšec zhasne na jiném poli, než kam táhne útočník|`clear` v `positionAt`, detekce v `applyMove`|
|**Skok přes více tahů**|nejdřív skokem na tah před cílem, pak animovaný poslední tah|§12.4|
|**Přepnutí varianty / větve, nová partie**|pozice se přepne skokem|`key` skupiny `pieces-${vIdx}-${sideIdx}` (React), `instant` (vanilla)|
|**Otočení desky (↕)**|všechny figury naráz přejedou na zrcadlené pozice|ponechat — je to čitelný „převrat" desky|
|**Změna barevného režimu**|mění se jen barvy, ne pozice|žádná animace `fill`|

### 12.9 Přístupnost

```js
const REDUCE = typeof window !== 'undefined'
  && window.matchMedia
  && window.matchMedia('(prefers-reduced-motion: reduce)').matches;
// React:   transition: instant || REDUCE ? 'none' : MOTION.move
// vanilla: g.style.transition = instant || REDUCE ? 'none' : speed
```

Při `REDUCE` zůstává dvoufázový skok (§12.4) zachovaný — fáze 2 prostě proběhne bez přechodu. Zvýraznění `lastFrom` / `lastTo` je v tom případě jediné vodítko, proto nesmí chybět nikdy.

### 12.10 Co animaci rozbíjí (kontrolní seznam)

* ❌ `key` odvozený od pozice nebo indexu místo `p.id`
* ❌ mazání brané figury ze seznamu místo `alive: false`
* ❌ SVG atribut `transform="translate(x,y)"` místo CSS `style.transform`
* ❌ chybějící jednotky `px` v `translate()`
* ❌ nové pole figur vytvořené při každém renderu bez `useMemo`
* ❌ vykreslování figur uvnitř smyčky po polích desky (matice 8×8)
* ❌ **nová identita figury při proměně** (nový `id` = teleport)
* ❌ **stavění figur z FEN pro každý krok znovu** (`new Chess(fen).board()` → `<text>`) místo `buildTimeline` (§12.5)
* ❌ **mazání celé SVG** (`while (svg.firstChild) svg.removeChild(…)`) při každém kroku (§12.6)
* ❌ **skok na vzdálený tah jedním přechodem** bez fáze „tah před cílem" (§12.4)
* ❌ zvýraznění posledního tahu počítané ze `step` místo z kroku desky (bliknutí o snímek dřív)

---

## 13. Start z libovolné pozice (nové ve v5)

Do v4 uměl systém jen zahájení: `initPieces()` postavil 32 figur na základní postavení. Koncovky, studie a úlohy začínají kdekoli — proto tato sekce.

### 13.1 `piecesFromFEN`

```js
const START_FEN = '1k6/8/P6p/2P2Kp1/8/8/8/8 b - - 0 1';

function piecesFromFEN(fen) {
  const [board] = fen.split(' ');
  const ps = [];
  const counter = {};                       // kolik figur daného druhu už bylo
  board.split('/').forEach((row, r) => {
    let c = 0;
    for (const ch of row) {
      if (/\d/.test(ch)) { c += Number(ch); continue; }
      const side = ch === ch.toUpperCase() ? 'w' : 'b';
      const t = ch.toUpperCase();
      const k = `${side}${t}`;
      counter[k] = (counter[k] || 0) + 1;
      ps.push({ id: `${k}${counter[k]}`, side, t, r, c, alive: true });
      c += 1;
    }
  });
  return ps;                                 // id je stabilní, protože pořadí čtení FEN je pevné
}
```

`positionAt(moves, upTo)` začíná `piecesFromFEN(START_FEN)` místo `initPieces()`. Pravidlo `key = p.id` (§12.1) zůstává beze změny — animace na tom stojí.

**Zlaté pravidlo:** dokud není ověřený FEN, artefakt se nestaví. Odhadovat pozici ze slovního popisu nebo screenshotu je zakázané (§9). Nejčastější chyba: zkopírovaná pozice **z prostředka lekce** místo z jejího začátku → nesedí počty figur. Postup je vždy: přetočit na začátek lekce → zkopírovat FEN → spočítat figury proti řetězci → vložit celý řetězec včetně pole „na tahu".

### 13.2 Proměna pěšce

```js
{ m: [1,0,0,0], san: 'a8=D', promote: 'Q', title: '…', comment: '…' }
```

V `positionAt` po přesunu: `if (mv.promote) pc.t = mv.promote;`. Glyf se přebarví okamžitě, figura si **ponechá `id`**, takže animace posledního kroku doběhne. Do řádku v záložce Tah se přidá `→ Dáma`.

### 13.3 Číslování tahů a strana na tahu

Žetony tahů i záložka Historie **nesmí** začínat od „1.". Číslo se bere z `fullmove` ve FEN a parita z pole „na tahu":

```js
const [, sideToMove, , , , fullmove] = START_FEN.split(' ');
const startFull = Number(fullmove);
const offset = sideToMove === 'b' ? 1 : 0;     // černý na tahu = lichý půltah

const numLabel = (i) => {
  const half = i + offset;
  const no = startFull + Math.floor(half / 2);
  return half % 2 === 0 ? `${no}.` : `${no}...`;
};
```

Pokud studie začíná černým tahem, první žeton je `1...f4`, ne `1.f4`. Stejný `offset` se použije i pro určení strany na tahu ve FEN exportu (§11) a pro barvu řádků v Historii.

### 13.4 PGN musí nést pozici

Bez těchto tagů Lichess partii naimportuje ze základního postavení a rozsype ji:

```
[Event "…"]
[Site "Claude artefakt"]
[SetUp "1"]
[FEN "1k6/8/P6p/2P2Kp1/8/8/8/8 b - - 0 1"]
[Result "*"]
```

Movetext u startu černým tahem začíná `1...`. Převod české notace na anglickou (§11) platí i pro písmeno za rovnítkem (`=D` → `=Q`).

### 13.5 Odznak hodnocení

U koncovek a úloh je hodnocení pozice součástí zadání, ne detailem. V hlavičce vedle podtitulu stojí malá pilulka:

|Odznak|Význam|Barva|
|-|-|-|
|`+−`|bílý vyhrává|`gold`, rámeček `goldDim`|
|`−+`|černý vyhrává|`gold`, rámeček `goldDim`|
|`=`|remíza|`muted`, rámeček `border`|

Text pilulky je monospace 11 px, padding `2px 8px`, `borderRadius: 10`. **Odznak musí sedět se skutečným výsledkem hlavní linie** — kontroluje se v §9.

### 13.6 Otázka pro uživatele

Artefakty typu koncovka/úloha mají v záložce Strategie **první odstavec formulovaný jako otázka** („Bílý je na tahu. Najdi jediný tah, který drží remízu."). Bez ní artefakt jen předvádí řešení, místo aby učil hledat.

---

## 14. Vedlejší varianty (nové ve v5)

Do v4 se alternativy vysvětlovaly jen slovně v komentáři. To stačí u zahájení, ale u koncovek je potřeba **ukázat na desce, co se stane po chybě** — a pak se vrátit zpět do hlavní linie.

### 14.1 Datový model

```js
{ m: [5,2,4,2], san: 'Kc5!', title: '…', comment: '…',
  side: [{
    kind: 'trap',                       // 'variant' | 'trap' | 'calc'
    label: 'A co 2.Kc6?',               // text tlačítka, vždy otázka nebo krátký popis
    moves: [ { m:[…], san:'Kc6?', title:'…', comment:'…' }, … ],
    verdict: 'Remíza — pěšec doběhne o tempo dřív.',
  }],
}
```

### 14.2 Druhy větví a jejich role

|`kind`|Kdy použít|Barva pilulky|
|-|-|-|
|`variant`|rovnocenná alternativa, taky hratelná|rámeček `border`, text `textBody`|
|`trap`|chyba a její trest — „co se stane, když…"|rámeček `mark`, text `mark`|
|`calc`|čistě výpočetní odbočka (kolik tahů, kdo je dřív)|rámeček `goldDim`, text `gold`|

### 14.3 Pravidla

* **Nejvýš jedna úroveň zanoření.** Větev nesmí mít vlastní `side`. Co se nevejde, patří do komentáře slovně, nebo do samostatného artefaktu.
* Větev se otevírá **tlačítkem v záložce Tah**, ne v žetonech — žetony patří hlavní linii
* Po otevření se nad deskou objeví **pruh s názvem větve a tlačítkem „← Zpět do hlavní linie"**; pruh má barvu podle `kind`
* Uvnitř větve funguje navigace stejně (`⏮ ◀ ▶ ⏭`), ale krok 0 větve = pozice **před** tahem, ze kterého větev odbočuje
* Návrat obnoví `step` na hodnotu před odbočením — uživatel nesmí „vypadnout" jinam, než odkud odbočil
* `verdict` se zobrazí na konci větve jako zvýrazněný řádek — jednou větou, co si z odbočky odnést
* `key` skupiny figur se rozšíří na `pieces-${vIdx}-${sideIdx}`, aby se pozice přepnula skokem (§12.3)

---

## 15. Vrstvy zvýraznění (nové ve v5)

Kroužky `marks` z v2 zůstávají, ale samy nestačí: koncovky potřebují ukázat **oblast** (čtverec pěšce), **pole, o která se hraje** a **směr plánu**. Význam barev je pevný a nesmí se míchat.

|Vrstva|Tvar|Barva|Význam|
|-|-|-|-|
|`zone`|obdélník polí, výplň 10 %, přerušovaný obrys|`gold`|prostor / geometrie (čtverec pěšce, plovoucí čtverec, prostor krále)|
|`keys`|plný kruh `r = SQ/2 − 8`, krytí 22 %|`gold`|klíčová pole, o která se pozice hraje|
|`marks`|přerušovaný kroužek (`5 4`, šířka 2,5)|`mark`|konkrétní hrozba nebo cíl v tomto tahu|
|`arrows`|čára se šipkou, šířka 3,5, krytí 75 %|`gold` = plán, `mark` = hrozba|pohyb, manévr, směr útoku|

**Pevná sémantika:** `gold` = moje geometrie a můj plán, `mark` = soupeřova hrozba a cíl útoku. Nikdy naopak — uživatel se barvu naučí jednou a musí platit ve všech artefaktech série.

```jsx
// zone
<rect x={px(c0)} y={py(r0)}
  width={(c1 - c0 + 1) * SQ} height={(r1 - r0 + 1) * SQ}
  fill={T.gold} fillOpacity={0.10}
  stroke={T.gold} strokeWidth="2" strokeDasharray="6 5" />

// arrow
<line x1={px(fc)+SQ/2} y1={py(fr)+SQ/2} x2={px(tc)+SQ/2} y2={py(tr)+SQ/2}
  stroke={kind === 'threat' ? T.mark : T.gold} strokeWidth="3.5"
  strokeOpacity="0.75" markerEnd="url(#arrowhead)" />
```

**Pravidla:**

* `zone` a `keys` se počítají v souřadnicích řádek/sloupec a otočení řeší `px()`/`py()` — nikdy neindexovat podruhé
* `markerEnd` definovat v `<defs>` **zvlášť pro každou barvu** (`arrowhead-gold`, `arrowhead-mark`), protože SVG marker nedědí `stroke` rodiče
* Naráz nejvýš **dvě šipky a jedna zóna**. Víc = deska přestává být čitelná; zbytek patří do komentáře.
* Všechny vrstvy jsou vázané na konkrétní tah (`moves[i]`), ne na globální stav — při kroku zpět zmizí samy

---

## 16. Počítadlo závodu (nové ve v5)

U pěšcových koncovek se rozhoduje o jediné tempo. Slovní „bílý je o tah dřív" se hůř chápe než číslo.

```js
{ m: […], san: 'a5!', race: { w: 4, b: 5, note: 'Bílý mění o tempo dřív — proto tenhle tah, ne a4.' } }
```

Zobrazuje se jako pruh **pod deskou, nad navigací** (§5.4), jen u tahů, které `race` mají:

```
┌────────────────────────────────────┐
│  ZÁVOD   Bílý 4 tahy · Černý 5     │
│  Bílý mění o tempo dřív…           │
└────────────────────────────────────┘
```

* Čísla monospace 15 px v barvě `gold`, štítek `ZÁVOD` v `goldDim` kapitálkách, `note` v `textBody` 13 px
* Vedoucí strana má číslo tučně; při rovnosti obě `muted`
* Pruh **nesmí** být trvale na obrazovce — jen v tazích, kde se závod skutečně počítá, jinak zevšední

---

## 17. Režim úlohy (nové ve v5)

Aktivní hledání tahu učí víc než proklikávání hotového řešení. Režim úlohy je **volitelný** a zapíná se pilulkou v pruhu se záložkami.

### 17.1 Chování

* Stav: `puzzle` (bool, výchozí `false`) a `revealed` (index posledního odhaleného půltahu)
* Zapnutý režim **maskuje** v žetonech tahy, které mají `hidden: true`, na `1.■` — maska má **vždy stejnou šířku bez ohledu na délku tahu**, aby neprozradila, jestli jde o pěšce nebo dámu
* Záložka Tah zobrazuje místo komentáře výzvu: *„Najdi tah bílého. Až budeš mít odpověď, odhal ho."* + tlačítko **Odhalit tah**
* Odhalení posune `revealed`, přehraje tah s animací a odemkne komentář; `◀` se dá vrátit zpět
* `⏭` je v režimu úlohy neaktivní (jinak by prozradil celé řešení jedním klikem); zůstává `⏮` a `◀`
* Vypnutí režimu odhalí vše a vrátí běžné chování — nic se neresetuje

### 17.2 Co maskovat

`hidden: true` patří **jen tahům strany, která řeší** (typicky bílý). Soupeřovy odpovědi zůstávají viditelné — jinak úloha přestává být šachová a stává se hádankou.

---

## 18. Série artefaktů (nové ve v5)

Artefakty vydávané jako řada (např. **Pěšcové koncovky — 8 lekcí**) musí být poznat na první pohled a nesmí si vysvětlovat stejný pojem pokaždé jinak.

### 18.1 Odznak série

```js
const SERIES = { name: 'Pěšcové koncovky', index: 3, total: 8 };
```

V hlavičce **pod eyebrow, nad titulem**: pilulka `Pěšcové koncovky · 3 / 8`, monospace 11 px, barva `goldDim`, rámeček `border`. Pokud `SERIES` chybí, pilulka se nevykreslí.

### 18.2 Sdílený glosář

Pojmy, které se v sérii opakují (opozice, zugzwang, klíčová pole, čtverec pěšce, volný pěšec), se **napříč artefakty formulují doslova stejně**. Definice se drží v jednom bloku, který se do každého artefaktu kopíruje beze změny:

```js
const GLOSSARY = {
  opozice: { name: 'Opozice', text: '…' },
  zugzwang: { name: 'Zugzwang', text: '…' },
  // …
};

const CONCEPTS = [
  GLOSSARY.opozice,
  GLOSSARY.zugzwang,
  { name: 'Chráněný volný pěšec', text: '…' },   // pojem specifický pro tuto lekci
];
```

Sdílené pojmy jdou **první**, lekci vlastní pojmy za nimi. Přeformulovat sdílenou definici „líp" v jednom artefaktu je chyba — konzistence je pro učení cennější než stylistika.

---

## 19. Mini-diagramy v záložce Koncepty (nové ve v5)

Abstraktní pojem se pochopí rychleji s obrázkem. Koncept proto může nést vlastní **statický diagram**:

```js
{ name: 'Čtverec pěšce', text: '…',
  diagram: { fen: '8/8/8/3k4/8/8/4P3/4K3 w - - 0 1',
             zone: { r0:2, c0:4, r1:6, c1:7 },
             caption: 'Král je uvnitř čtverce — pěšce dohoní.' } }
```

**Pravidla:**

* `SQ = 22`, **bez souřadnic, bez rámu, bez animace a bez interakce** — je to obrázek, ne druhá deska
* Vykresluje se pod textem konceptu, zarovnaný vlevo, `maxWidth: 176`
* Podporuje jen `zone` a `keys`, ne `arrows` ani `marks` — v malém měřítku by se ztratily
* `caption` je 12 px `muted` kurzívou pod diagramem
* Nejvýš **dva** diagramy na artefakt; ostatní pojmy zůstávají textové

---

## 20. Rozhodovací tabulka funkcí

Ne každý artefakt potřebuje všechno. Tabulka říká, co zapnout podle typu:

|Funkce|Zahájení|Past / motiv|Koncovka|Rozbor partie|Nástroj (analyzátor)|
|-|-|-|-|-|-|
|Čtyři barevné režimy (§2)|✅|✅|✅|✅|✅|
|Animace figur (§12) — včetně skoku §12.4|✅|✅|✅|✅|✅|
|Přepínač variant (§10)|✅ typ A|✅ typ B|⚪ zřídka|❌|❌|
|Start z FEN (§13.1)|❌|⚪|✅|⚪|✅|
|Proměna pěšce (§13.2)|❌|⚪|✅|⚪|✅|
|Odznak hodnocení (§13.5)|❌|⚪|✅|⚪|❌|
|Otázka pro uživatele (§13.6)|⚪|✅|✅|❌|❌|
|Vedlejší varianty `side` (§14)|⚪|✅|✅|⚪|❌|
|`zone` / `keys` (§15)|❌|⚪|✅|❌|❌|
|`arrows` (§15)|⚪|✅|⚪|⚪|✅ tah enginu|
|Počítadlo závodu (§16)|❌|❌|✅ jen pěšcové|❌|❌|
|Režim úlohy (§17)|❌|✅|✅|❌|❌|
|Odznak série (§18)|⚪|⚪|✅|❌|❌|
|Mini-diagramy (§19)|⚪|⚪|✅|❌|❌|

✅ zapnout · ⚪ podle obsahu · ❌ nezapínat

**Pravidlo úspornosti:** zapnutá funkce, která v daném artefaktu nic nevysvětluje, škodí — rozptyluje a zdražuje údržbu. Když váháš, nech ji vypnutou.

---

## 21. Verzování artefaktů a patička (nové ve v5)

Každý artefakt musí být zvenčí poznat, podle které verze design systému vznikl. Bez toho nejde po roce rozhodnout, jestli je artefakt v pořádku, nebo jen starý.

### 21.1 Kde verze být má — a kde ne

|Místo|Verdikt|Proč|
|-|-|-|
|**Patička v artefaktu**|✅ **povinné**|cestuje s obsahem, vidí ji uživatel i budoucí opravář, nejde ji rozpojit od kódu|
|**Konstanta na začátku souboru**|✅ **povinné**|jediný zdroj pravdy, ze kterého se patička generuje|
|Název souboru|❌ **nepoužívat**|při upgradu na novou verzi by bylo nutné soubor přejmenovat → rozbité odkazy, dvě kopie vedle sebe, nejasno, která je živá|
|Komentář v kódu|⚪ volitelné|neuškodí, ale sám nestačí — uživatel ho neuvidí|

**Totéž platí pro samotnou specifikaci:** je to jediný soubor `design-system-sachovych-artefaktu.md` v git repozitáři. Číslo verze nese jen nadpis dokumentu, changelog a **git tag** (`v5`, `v6`, …); starší verze se nečtou z kopií vedle sebe, ale z historie — na GitHubu přes záložku *Tags*, lokálně `git show v5:design-system-sachovych-artefaktu.md`, rozdíl verzí `git diff v5 v6`. (Do v5 se verze psala do názvu souboru; od v6 už ne.)

### 21.2 Konstanta

Hned pod importy, nad `THEMES`:

```js
const ARTEFAKT = {
  nazev: 'italska-partie-kompletni',   // shodné s názvem souboru, bez přípony
  ds: 'v6',                            // verze design systému, podle které byl POSTAVEN
  vznik: '2026-08',                    // rok a měsíc vzniku
  revize: null,                        // '2026-11' při větší úpravě, jinak null
};
```

`ds` se **nepřepisuje jen proto, že vyšla nová verze specifikace.** Mění se teprve tehdy, když je artefakt skutečně přestavěný a projde kontrolním seznamem §9 pro danou verzi. Falešné „v6" v patičce je horší než poctivé „v4".

### 21.3 Patička

Úplně dole, **pod tlačítky FEN / PGN**, jeden řádek na střed:

```jsx
<div style={{
  marginTop: 18, textAlign: 'center',
  fontFamily: 'monospace', fontSize: 10, letterSpacing: 0.5,
  color: T.muted, opacity: 0.75,
}}>
  design systém {ARTEFAKT.ds} · {ARTEFAKT.nazev} · {ARTEFAKT.vznik}
  {ARTEFAKT.revize && ` · rev. ${ARTEFAKT.revize}`}
</div>
```

Vykreslí se například:

```
design systém v6 · italska-partie-kompletni · 2026-08
```

**Pravidla:**

* barva `muted` s krytím 0,75 — má být přečtitelná, ale nesmí soupeřit s kartou „CO SI ODNÉST"
* 10 px monospace, `letterSpacing: 0.5`; **žádný rámeček, žádné pozadí, žádná ikona**
* při revizi se přidá `· rev. 2026-11`, původní `vznik` zůstává — je vidět, jak artefakt stárl
* patička je jediná část artefaktu, která **není** česky formátovaná do věty; je to technický štítek, ne text

### 21.4 Kdy zvyšovat `ds`

|Situace|`ds`|`revize`|
|-|-|-|
|Oprava překlepu v komentáři|beze změny|beze změny|
|Oprava chyby v tazích, doplnění varianty|beze změny|doplnit|
|Přestavba podle novější specifikace + kontrola §9|zvýšit|doplnit|
|Nový artefakt|aktuální verze|`null`|

---

## 22. Migrace v4 → v5

Existující artefakty se **nemusí** přepisovat. Pokud se ale artefakt otevírá kvůli opravě, doplní se při té příležitosti:

1. Kontrola, že `positionAt` používá `if (mv.promote)` — jeden řádek, umožní budoucí proměnu
2. `key` skupiny figur rozšířit na `pieces-${vIdx}-${sideIdx}`, i když `sideIdx` zatím není
3. Pořadí variant přerovnat podle nového §10 (typ A vs. B)
4. Doplnit konstantu `ARTEFAKT` a patičku (§21) — u nepřestavěného artefaktu s `ds: 'v4'`
5. Kontrolní seznam §9 projít v celém rozsahu, ne jen v části, které se oprava týkala

**Zpětná kompatibilita zápisu:** dřívější dokumenty (např. `zadani-artefaktu-pawn-endgame-bootcamp.md`) odkazují na „§13.5 zone" a „§13.6 odznak hodnocení". Ve v5 se `zone` přesunula do **§15** (jako součást vrstev zvýraznění) a odznak hodnocení je **§13.5** (posunulo se o jedno nahoru, protože PGN se sloučilo do §13.4). Při čtení starších zadání se řídit v5.

---

## 23. Migrace v5 → v6

Změna ve v6 se týká jen pohybu figur, obsah artefaktů zůstává. Artefakty postavené podle v5 (React s `positionAt`) už plynule animují krok po kroku — chybí jim jen dvoufázový skok. Při nejbližší opravě:

1. Rozdělit stav na `step` (logický) a `board = { step, instant }` (deska) a přepsat `go()` podle §12.4
2. Zvýraznění posledního tahu, `marks`, vrstvy §15 a počítadlo závodu přepojit ze `step` na `board.step`
3. Časy přechodů nahradit konstantou `MOTION` (§12.3)
4. Pokud linie obsahuje braní mimochodem, doplnit `clear` (§6)
5. Kontrolní seznam §9 (body animace a–j) projít celý, pak `ds: 'v6'` a `revize` (§21.4)

**Nástroje ve vanilla JS** (Stockfish analyzátor) se přestavují celé podle §12.5–12.6: `drawBoard()` rozdělit na čtyři trvalé vrstvy, figury kreslit přes `renderPieces(timeline[k])`, `timeline` stavět jednou po načtení partie a ve volné analýze prodlužovat o každý tah.

**Zpětná kompatibilita:** v6 nemění datový model tahů (`m`, `m2`, `promote`, `side`, …) — přibyl jen volitelný klíč `clear`. Artefakt podle v5 je proto dál platný; v patičce nechat poctivě `ds: 'v5'`, dokud neprojde přestavbou.

---

*Konec specifikace v6.*
