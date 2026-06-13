# Întrebarea 1: Relația dintre principiile SOLID

Cum se leagă Single Responsibility Principle (SRP) de Open/Closed Principle (OCP)? Dacă o clasă respectă cu adevărat regula SRP de a avea un singur motiv pentru a se schimba, înseamnă automat că respectă și OCP (fiind închisă la modificări )? Sau pot exista situații în care o clasă are o singură responsabilitate bine definită, dar tot necesită o încălcare a OCP pentru a i se adăuga noi funcționalități?

**Răspuns:**

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

---

# Întrebarea 2: Factory Method vs. Abstract Factory în practică

Nu înțeleg foarte clar diferența practică dintre Factory Method, care delegă crearea către subclase folosind o interfață comună, și Abstract Factory, care creează familii de obiecte relaționate. Materialul oferă exemplul clasic cu elemente de UI pentru Windows și Mac. Ai putea să-mi dai un exemplu practic aplicat strict pe zona de **backend**, care să clarifice de ce aș folosi Abstract Factory în loc de mai multe Factory Methods independente?

**Răspuns:**

Salutare! Este o întrebare excelentă. Diferența principală subliniată în materialele de curs este următoarea: **Factory Method** creează _un singur tip_ de obiect, delegând decizia de instanțiere către subclase, în timp ce **Abstract Factory** creează _familii de obiecte relaționate_ care trebuie neapărat să funcționeze împreună.

Pentru a clarifica _de ce_ ai folosi Abstract Factory în loc de mai multe Factory Methods independente, îți voi oferi un exemplu practic de backend.

_Pe baza cunoștințelor generale (nu din materialele de curs):_

Să ne imaginăm că dezvolți o aplicație backend cu o **arhitectură multi-cloud** care trebuie să comunice cu diverse servicii (de exemplu, AWS sau Microsoft Azure). Pentru ca un modul din aplicația ta să funcționeze, are nevoie simultan de trei componente distincte:

1. Un serviciu de stocare a fișierelor (`IStorage`)
2. O bază de date (`IDatabase`)
3. Un sistem de mesagerie (`IMessageQueue`)

### Abordarea 1: Mai multe Factory Methods independente

Dacă ai folosi trei Factory Methods separate, codul clientului ar arăta cam așa:

```java
IStorage storage = StorageFactory.create("AWS");         // Îți dorești AWS S3
IDatabase db = DatabaseFactory.create("Azure");          // Ops! O greșeală de configurare returnează Azure SQL
IMessageQueue queue = QueueFactory.create("AWS");        // AWS SQS
```

**Problema:** Un dezvoltator sau o configurare greșită poate amesteca serviciile. Aplicația ta va încerca acum să citească din AWS S3, să scrie date în Azure SQL și să trimită mesaje din nou în AWS SQS. Această abordare introduce probleme majore de latență (comunicare cross-cloud), costuri neprevăzute de trafic de rețea și potențiale erori de securitate. Nimic din cod nu garantează că aceste trei obiecte fac parte din aceeași "familie".

### Abordarea 2: Abstract Factory

Aici intervine Abstract Factory, care are rolul exact de a asigura **compatibilitatea** și faptul că familiile de obiecte **lucrează împreună** fără dependențe hard-codate.

În loc de trei fabrici independente, vei crea un `ICloudInfrastructureFactory` (fabrica abstractă):

```java
public interface ICloudInfrastructureFactory {
    IStorage createStorage();
    IDatabase createDatabase();
    IMessageQueue createMessageQueue();
}
```

Apoi, vei avea fabrici concrete pentru fiecare furnizor: `AWSInfrastructureFactory` și `AzureInfrastructureFactory`. Codul tău de backend va arăta astfel:

```java
// Configurarea inițială la nivel de aplicație alege familia:
ICloudInfrastructureFactory factory = new AWSInfrastructureFactory();

// Mai târziu în cod, modulul tău își creează instanțele fără să îi pese de cloud-ul exact:
IStorage storage = factory.createStorage();       // Ești garantat că primești S3
IDatabase db = factory.createDatabase();          // Ești garantat că primești AWS RDS
IMessageQueue queue = factory.createMessageQueue(); // Ești garantat că primești SQS
```

### Concluzie

Dacă ai nevoie doar de un singur obiect izolat, de exemplu pentru un sistem de procesare a plăților (cu mai multe tipuri de plată), folosești **Factory Method**.

În schimb, folosești **Abstract Factory** atunci când obiectele tale fac parte dintr-un ecosistem și sunt obligate să comunice perfect între ele. Abstract Factory protejează codul, acționând ca o "fabrică de fabrici", împiedicându-te să asamblezi un "Frankenstein" din obiecte incompatibile de la familii (în cazul nostru, furnizori cloud) diferite.

---

# Întrebarea 3: Strategy Pattern vs. State Pattern

La nivel de implementare, pattern-urile Strategy și State par aproape identice, ambele bazându-se pe încapsularea comportamentului și delegarea acestuia către obiecte specifice. Totuși, materialul menționează că Strategy schimbă algoritmii la nivel de client, iar State alterează comportamentul pe baza stării interne (precum un Finite State Machine). În practică, cine dictează "schimbarea" în cazul fiecărui pattern (contextul intern sau clientul extern), și cum decid care dintre ele este potrivit pentru un anumit scenariu?

**Răspuns:**

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

---

# Întrebarea 4: Concepte suprapuse: Facade vs. API Gateway

Ambele concepte, Facade Pattern (din zona structurală a design pattern-urilor) și API Gateway (din arhitectura de microservicii), au ca scop principal ascunderea complexității unui sistem intern și oferirea unui punct unic și simplificat de intrare. Există vreo diferență fundamentală de design între ele, sau API Gateway-ul este pur și simplu un Facade Pattern aplicat la nivel de infrastructură distribuită?

**Răspuns:**

Observația ta este excelentă și atinge un punct esențial în evoluția arhitecturilor software. Răspunsul scurt este: **Da, la nivel pur conceptual, un API Gateway este o manifestare a modelului Facade, dar aplicat la scară distribuită.** Ambele ascund complexitatea și expun o interfață unificată.

Totuși, răspunsul detaliat este că **„trecerea graniței” rețelei schimbă fundamental regulile jocului**, transformând API Gateway-ul dincolo de o simplă fațadă de cod, într-o componentă critică de infrastructură.

Iată diferențele fundamentale de design și responsabilitate între cele două concepte:

### 1. Spațiul de Execuție (In-Process vs. Out-of-Process)

- **Facade Pattern:** Trăiește în spațiul de memorie al aplicației (In-Process). De exemplu, într-un backend scris în NestJS, un Facade ar putea fi un `Service` dedicat care injectează alte trei servicii mai granulare (ex: `UserService`, `BillingService`, `NotificationService`) pentru a expune o singură metodă curată către `Controller`. Apelurile între ele sunt simple apeluri de funcții/metode.
- **API Gateway:** Este un proces complet separat, adesea rulând pe propriul său cluster de servere (Out-of-Process). Comunicarea între Gateway și microserviciile din spate se face exclusiv prin rețea, ceea ce introduce latență, serializare/deserializare (ex: JSON) și necesitatea gestionării conexiunilor.

### 2. Responsabilitățile de Infrastructură (Cross-Cutting Concerns)

- **Facade Pattern:** Scopul său este strict organizarea codului, reducerea cuplajului (coupling) și respectarea principiilor SOLID în interiorul aceluiași domeniu (sau monolit).
- **API Gateway:** Fiind punctul unic de intrare dintr-o rețea externă în rețeaua internă, preia o serie de „heavy lifting-uri” operaționale pe care un Facade clasic nu le face:
- **Autentificare și Autorizare:** Validarea token-urilor (ex: JWT) înainte ca request-ul să atingă serviciile interne.
- **Rate Limiting și Throttling:** Protejarea microserviciilor de atacuri DDoS sau de un volum prea mare de trafic.
- **SSL Termination:** Decriptarea traficului HTTPS la nivelul Gateway-ului pentru a reduce încărcarea procesorului pe microservicii.

### 3. Traducerea Protocoalelor și Compunerea Datelor

- **Facade Pattern:** Operează, în general, cu structuri de date sau obiecte native din limbajul de programare respectiv.
- **API Gateway:** Funcționează adesea ca un traducător universal. Poate primi un request HTTP REST sau GraphQL din partea unui client de mobile, dar intern poate orchestra apeluri folosind protocoale mult mai eficiente pentru comunicarea server-to-server, cum ar fi gRPC, sau poate plasa mesaje într-un message broker (RabbitMQ/Kafka).

### 4. Gestionarea Erorilor (The Fallacies of Distributed Computing)

- **Facade Pattern:** Erorile sunt excepții de cod (Stack Traces). Gestionarea lor se face prin blocuri standard `try/catch`. Dacă un sub-sistem pică, pică toată tranzacția în memorie.
- **API Gateway:** Trebuie să gestioneze complexitatea rețelelor distribuite: timeout-uri, partiții de rețea, sau servicii care răspund lent. Aici intervin pattern-uri adiționale de reziliență precum **Circuit Breaker**, **Retry Mechanisms** și strategii de **Fallback**, care sunt indispensabile în designul unui Gateway, dar irelevante pentru un Facade clasic.

---

### Comparație Rapidă

| Caracteristică     | Facade Pattern (GoF)                  | API Gateway                                            |
| ------------------ | ------------------------------------- | ------------------------------------------------------ |
| **Natură**         | Pattern structural de design (Cod)    | Pattern arhitectural distribuit (Infrastructură)       |
| **Latență**        | Neglijabilă (apeluri în memorie)      | Notabilă (I/O de rețea)                                |
| **Scop Principal** | Decuplarea claselor, API curat intern | Rutare, securitate, agregare rețea                     |
| **Protocoale**     | Native limbajului (metode/obiecte)    | HTTP, WebSocket, gRPC, TCP                             |
| **Evoluție**       | Poate fi refactorizat extrem de ușor  | Schimbările necesită planificare și deployment separat |

În concluzie, poți privi un API Gateway ca pe un **Facade arhitectural cu „superputeri” de rețelistică și securitate**. Dacă te afli la începutul unui proiect și ai un monolit modular, un Facade Pattern bine implementat îți va permite să extragi mult mai ușor ulterior logica respectivă în microservicii aflate în spatele unui API Gateway, atunci când scara proiectului o va cere.

---

# Întrebarea 5: Compromisurile integrării asincrone

Alegerea Shared Database Integration oferă un nivel foarte scăzut de latență, tranzacții sigure și o consistență imediată a datelor. Cu toate acestea, în sistemele distribuite moderne și de mare volum se recomandă trecerea la Message Broker Integration, chiar dacă acest lucru adaugă latență și complexitate ("eventual consistency"). Care sunt limitele tehnice reale ale Shared Database Integration de ocolit cărora arhitecții sunt dispuși să accepte o implementare bazată pe broker de mesaje?

**Răspuns:**

Ai surprins perfect esența unuia dintre cele mai mari compromisuri din arhitectura sistemelor distribuite. Deși **Shared Database Integration** (baza de date partajată) oferă confortul tranzacțiilor ACID (Atomicitate, Consistență, Izolare, Durabilitate) și elimină latența de rețea dintre servicii pentru sincronizarea datelor, la scară largă această abordare se transformă într-un anti-șablon.

Arhitecții de sistem preferă complexitatea unui **Message Broker** și modelul _eventual consistency_ pentru a ocoli următoarele limite tehnice severe ale bazelor de date partajate:

### 1. Epuizarea resurselor și efectul de "Noisy Neighbor"

O bază de date unică are resurse finite (CPU, memorie, IOPS pe disc). Într-o arhitectură cu bază de date partajată, dacă un serviciu (de exemplu, cel de _Raportare_) rulează o interogare analitică greoaie, acesta va consuma majoritatea resurselor disponibile. Rezultatul este că un alt serviciu critic (de exemplu, cel de _Checkout_ sau _Plăți_) va experimenta latențe uriașe sau va pica complet, chiar dacă procesează date complet diferite. Prin decuplare, un serviciu nu mai poate acapara resursele altuia.

### 2. Limitele de "Connection Pooling"

Bazele de date relaționale clasice (cum ar fi PostgreSQL, MySQL) sunt concepute să gestioneze un număr limitat de conexiuni TCP concurente și persistente. În sistemele moderne cu auto-scalare, unde pot apărea sute sau mii de instanțe efemere de microservicii în momentele de vârf, pool-ul de conexiuni al bazei de date partajate se epuizează rapid. Un Message Broker, în schimb, este conceput nativ pentru a gestiona zeci de mii de conexiuni concurente (producători și consumatori) printr-un throughput masiv.

### 3. Contenția la nivel de blocaj (Lock Contention)

Pentru a menține "consistența imediată", baza de date folosește mecanisme de locking (pe rând sau pe tabelă). Într-un sistem de mare volum unde mai multe servicii încearcă să scrie sau să actualizeze date simultan în aceleași tabele, acestea vor intra într-o coadă de așteptare. Latența inițială "scăzută" a bazei de date explodează din cauza timpului pierdut așteptând eliberarea blocajelor (sau apar erori de tip _deadlock_ care invalidează tranzacțiile).

### 4. Cuplajul strâns la nivel de schemă (Schema Coupling)

Aceasta este probabil cea mai mare frână în agilitatea dezvoltării. Dacă două sau mai multe servicii depind de aceeași structură de tabelă, orice modificare de schemă (adăugarea, ștergerea sau modificarea unei coloane) necesită o coordonare strictă între multiple echipe de dezvoltare. Aceasta anulează practic avantajul principal al microserviciilor: capacitatea de a dezvolta și implementa (deploy) complet independent.

### 5. Single Point of Failure (Extinderea "Blast Radius-ului")

Dacă baza de date partajată necesită mentenanță, cade din motive hardware sau este coruptă, **întregul sistem** pică instantaneu. Trecerea la un arhitectură bazată pe mesaje este adesea însoțită de șablonul _Database-per-Service_. Astfel, dacă serviciul de recenzii cade pentru că și-a pierdut baza de date, serviciul de comenzi poate continua să funcționeze neîntrerupt, oferind o degradare grațioasă a aplicației.

### 6. Imposibilitatea persistenței poliglote

Nu toate datele au aceeași formă. Datele relaționale se potrivesc într-un RDBMS, documentele JSON complexe într-un NoSQL (ex: MongoDB), rețelele de relații într-un Graph Database (ex: Neo4j), iar căutările textuale complexe într-un motor precum Elasticsearch. O bază de date partajată obligă toate serviciile să folosească aceeași tehnologie (abrodarea "one size fits all"), ceea ce duce la performanțe suboptime pentru anumite cazuri de utilizare.

---

**Concluzie:**
Trecerea la **Message Broker Integration** nu se face pentru performanța absolută a unei singure operațiuni, ci pentru **supraviețuirea la scară a întregului sistem**. Acceptând "eventual consistency", arhitecții sacrifică tranzacțiile simple și imediate pentru a obține scalabilitate independentă, reziliență la defecte (prin asincronism și retry-uri) și autonomie totală pentru echipele de dezvoltare.
