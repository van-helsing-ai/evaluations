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
