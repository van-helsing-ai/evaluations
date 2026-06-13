Salut! Aceasta este o întrebare de arhitectură software excelentă și atinge o nuanță conceptuală pe care mulți o trec cu vederea.

Răspunsul scurt este **nu**: respectarea strictă a principiului SRP (Single Responsibility) **nu înseamnă automat** că o clasă respectă și OCP (Open/Closed).

Cele două principii abordează fațete diferite ale designului: SRP se concentrează pe _coeziune_ (ce face clasa și care sunt granițele ei), în timp ce OCP se concentrează pe _extensibilitate_ (cum gestionează clasa variațiile și evoluția în timp a codului).

Iată de ce pot exista situații frecvente în care o clasă are o singură responsabilitate perfect definită, dar tot necesită încălcarea OCP pentru funcționalități noi.

### Diferența de paradigmă

- **SRP (Motivul schimbării):** O clasă trebuie să se ocupe de un singur domeniu logic. Dacă se schimbă regulile de emitere a facturilor, ar trebui să modifici clasa de facturare, nu pe cea care trimite emailuri. Are un singur „motiv” general pentru a fi atinsă.
- **OCP (Închis la modificare, deschis la extensie):** Când apare o cerință nouă în acel domeniu specific, ar trebui să poți adăuga funcționalitatea scriind cod nou, _fără_ să modifici codul existent și deja testat.

### Exemplu practic: SRP respectat, OCP încălcat

Să ne gândim la un serviciu backend (de exemplu, o componentă dintr-o aplicație NestJS) care se ocupă cu analiza și calculul costurilor pentru consultanță juridică sau fiscală.

Avem clasa `CostCalculator`. Responsabilitatea ei este unică și clară: calculează costurile pe baza tipului de speță. Prin urmare, **respectă perfect SRP**. Nu interacționează cu baza de date, nu randează interfețe și nu se ocupă de validări HTTP.

```typescript
class CostCalculator {
    calculate(caseType: string, baseHours: number): number {
        if (caseType === "Civil") {
            return baseHours * 100;
        } else if (caseType === "Comercial") {
            return baseHours * 150;
        }
        return baseHours * 80; // Standard
    }
}
```

**Unde apare problema?**
Ce se întâmplă dacă mâine trebuie să adaugi o logică nouă pentru spețele de tip `Fiscal`?
Va trebui să intri direct în metoda `calculate` și să mai adaugi un `else if`. Deși schimbarea ține strict de „calculul costurilor” (deci conceptual ești în continuare pe aceeași unică responsabilitate a clasei), prin faptul că _modifici_ structura internă a unei metode existente, **încalci OCP**. Clasa nu a fost concepută să fie „închisă la modificări”.

### Cum le aducem în armonie?

Pentru ca o clasă să le respecte pe amândouă, intervenția se face de obicei prin **polimorfism și abstractizare**. În loc să centralizezi toate detaliile care variază într-un singur bloc de cod, le delegi.

Folosind un _Strategy Pattern_ și injecția de dependențe, codul s-ar transforma astfel:

```typescript
interface CostStrategy {
    isMatch(caseType: string): boolean;
    calculate(baseHours: number): number;
}

class CostCalculator {
    // Injectăm dinamic o listă de strategii
    constructor(private readonly strategies: CostStrategy[]) {}

    calculate(caseType: string, baseHours: number): number {
        const strategy = this.strategies.find((s) => s.isMatch(caseType));
        if (!strategy) throw new Error("Tip de speță necunoscut");

        return strategy.calculate(baseHours);
    }
}
```

Acum:

1. **SRP este păstrat:** `CostCalculator` are o singură responsabilitate – orchestrează logica de aplicare a costurilor.
2. **OCP este respectat:** Dacă apare o speță nouă, creezi o clasă nouă (ex. `FiscalStrategy` care implementează `CostStrategy`) și o înregistrezi în modulul tău. Nu mai atingi și nu mai modifici nicio linie de cod din clasa de bază `CostCalculator`.

---

În concluzie, SRP te asigură că nu construiești un „God Object” (o clasă care face de toate). OCP este pasul următor, care îți garantează că arhitectura acelei responsabilități unice este flexibilă și imună la regresii (bug-uri neintenționate) atunci când business-ul impune scenarii noi.
