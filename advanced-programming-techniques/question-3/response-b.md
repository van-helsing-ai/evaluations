Salut! Ai observat un detaliu excelent. Din punct de vedere structural (în diagramele UML), pattern-urile **Strategy** și **State** sunt aproape identice, bazându-se ambele pe compoziție și delegare pentru a schimba comportamentul la runtime. Totuși, intenția lor și modul în care se produce „schimbarea” sunt complet diferite.

Iată cum se diferențiază cele două pe baza materialelor de curs, precum și câteva detalii care să te ajute în alegerea pattern-ului potrivit:

### 1. Cine dictează "schimbarea"?

- **În cazul Strategy:** Schimbarea este dictată de **clientul extern**. Clientul știe ce fel de comportament dorește și furnizează contextului obiectul „Strategie” potrivit. De exemplu, clientul alege algoritmul la runtime, decuplând astfel logica de business de implementarea algoritmului.
    - _Notă din cunoștințe generale (în afara materialului):_ Strategiile, de regulă, nu știu una de existența celeilalte și nu se schimbă singure între ele.
- **În cazul State:** Schimbarea este dictată din **interior** (de către context sau de către însăși starea curentă). Obiectul își alterează comportamentul atunci când „starea sa internă se schimbă”, dând impresia că și-a modificat clasa.
    - _Notă din cunoștințe generale (în afara materialului):_ Obiectele de tip „State” au de obicei o referință către context și cunosc regulile de tranziție; ele pot declanșa singure trecerea la o altă stare, pe baza unor evenimente sau inputuri.

### 2. Cum decizi pe care să îl folosești într-un scenariu?

Conform materialelor cursului, alegerea trebuie să se bazeze pe scopul problemei pe care încerci să o rezolvi:

**Alege STRATEGY când:**

- Ai **mai mulți algoritmi care pot rezolva aceeași problemă** și dorești ca ei să fie complet interschimbabili (Memory Trick: „Strategy = Swappable algorithms”).
- Vrei să selectezi/schimbi algoritmul dinamic, pentru a evita un set lung de `if/else`-uri în clasa client.
- _Scenariu practic:_ Un sistem de plată unde utilizatorul alege (clientul dictează) dacă plătește cu Card, PayPal sau Crypto.

**Alege STATE când:**

- Modelezi un **Finite State Machine (FSM)** sau trebuie să gestionezi o tranziție clară de tip workflow.
- Comportamentul unui obiect se schimbă drastic pe măsură ce ciclul său de viață progresează (Memory Trick: „State = Object changes behavior with state”).
- _Scenariu practic:_ Un automat de vândut produse (Vending Machine) sau un bancomat (ATM). Un bancomat reacționează diferit dacă e în starea `Așteaptă_PIN`, `Fără_Bani` sau `Tranzacție_În_Curs`.

**Sinteză rapidă pentru examene:**

- Dacă problema menționează **„interchangeable algorithms”** sau selecție a algoritmului la runtime, gândește-te la **Strategy**.
- Dacă problema cere **„alter behavior when state changes”** sau menționează automate cu stări finite, mergi pe **State**.

Sper că aceste diferențe de nuanță clarifică confuzia! Dacă vrei să discutăm un exemplu de cod sau un scenariu specific la care te gândești, sunt aici să ajut.
