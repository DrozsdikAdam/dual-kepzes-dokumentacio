# Projektösszefoglaló: Duális Képzési Rendszer Backend API

## Áttekintés

A projekt egy nagyméretű, vállalati szintű kooperatív és duális képzési rendszert kiszolgáló REST API backend szolgáltatás. A rendszer elsődleges célja a felsőoktatási intézmények, a partnercégek és a hallgatók közötti együttműködés teljes életciklusának digitális koordinálása, az adminisztratív folyamatok automatizálása, valamint a jelentkezések és partnerségi szerződések kezelése szerepkör-alapú hozzáférés-vezérlés mellett.

---

## Főbb funkcionális modulok

### 1. Többszintű jogosultság- és szerepkörkezelés (RBAC / ABAC)
* Különböző szintű felhasználók (Hallgatók, Céges Mentorok, Céges Adminisztrátorok, Egyetemi Koordinátorok és Rendszeradminisztrátorok) kezelése.
* Egyedi jogosultsági rendszer, amely a statikus szerepkörök (Role-Based Access Control) mellett a dinamikus tulajdonosi viszonyokat (Attribute-Based Access Control) is ellenőrzi (például dokumentumok, jelentkezések és cégprofilok biztonságos szerkesztésénél).

### 2. Jelentkezési és felvételi csővezeték (Application Pipeline)
* Strukturált jelentkezési munkafolyamat, amely lehetővé teszi a hallgatók számára az aktív céges pozíciókra való jelentkezést és a szükséges dokumentumok (önéletrajz, motivációs levél) csatolását.
* Szigorú, állapottípusokhoz kötött átmenet-ellenőrzés (Submitted, Accepted, Rejected, No Response, Retracted), amely megakadályozza az érvénytelen munkafolyamat-lépéseket.

### 3. Duális partnerségek életciklus-kezelése (Partnership Management)
* Sikeres céges felvételi esetén a rendszer automatikusan elindítja a duális partnerségi munkafolyamatot.
* Egyetemi referensek (koordinátorok) intelligens, szak- és cégalapú hozzárendelése az adminisztrációs folyamatok felügyeletéhez.
* Szerződéskötési státuszok nyomon követése állapógéppel (Pending Mentor, Pending University, Active, Finished, Terminated).

### 4. Adminisztratív jelentéskészítés és statisztikai modulok
* Összetett adataggregációs végpontok a rendszer aktuális teljesítményének elemzésére.
* Statisztikák és trendek biztosítása hallgatói eloszlásról, szakok népszerűségéről, céges jelentkezési arányokról és referensi leterheltségről.

### 5. Aszinkron értesítési és háttérfolyamat-motor
* Beépített belső értesítési rendszer a fontos munkafolyamat-változások (pl. státuszváltás, új jelentkezés) valós idejű vagy aszinkron jelzésére.
* Decoupled (leválasztott) háttérfolyamatok az API terhelésének csökkentésére.

---

## Alkalmazott technológiák és könyvtárak

### Alkalmazás-infrastruktúra és nyelv
* **Node.js**: Eseményvezérelt, skálázható szerveroldali futtatókörnyezet.
* **TypeScript**: Típusbiztos architektúra megvalósítása a kontrollerektől a szolgáltatásokon át az adatbázis-lekérdezésekig.
* **Express (v5)**: Könnyűsúlyú és flexibilis webes keretrendszer az API végpontok kialakításához és middleware-láncok kezeléséhez.

### Adatkezelés és perzisztencia
* **PostgreSQL**: Relációs adatbázis-kezelő a komplex üzleti entitások és kapcsolatok biztonságos tárolására.
* **Prisma ORM**: Modern, deklaratív sémadefiníció és típusbiztos adat-hozzáférési réteg, amely támogatja az adatbázis-migrációk automatizálását és a seed-elést.

### Háttérfolyamatok és üzenetsorok
* **BullMQ & Redis**: Elosztott feladatsor (job queue) kezelése aszinkron feladatokhoz (pl. háttérben történő levélküldés és dokumentum-továbbítás), növelve a rendszer válaszidejét és hibatűrését.

### Biztonság és adatvédelem
* **JWT (JSON Web Token)**: Stateless munkamenet- és tokenalapú hitelesítés.
* **bcryptjs**: Biztonságos, sózott jelszó-hashelés.
* **Helmet & CORS**: Biztonsági HTTP fejlécek beállítása és kereszt-eredetű erőforrás-megosztás (Cross-Origin Resource Sharing) szabályozása.
* **Express Rate Limit**: Végpont-alapú kérésszám-korlátozás a túlterheléses (DoS) támadások elleni védelem érdekében.

### Fájl- és képkezelés
* **AWS SDK (S3)**: Felhőalapú objektumtár integráció a csatolt állományok és képek biztonságos, skálázható tárolásához.
* **Sharp & Multer**: Multipart form-data kérések feldolgozása, valamint nagy teljesítményű képoptimalizálás és méretezés a szerveroldalon.
* **Nodemailer**: SMTP alapú e-mail küldő modul sablonkezeléssel és csatolmány-támogatással.

### Tesztelés és dokumentáció
* **Jest & Supertest**: Unit és integrációs tesztek futtatása a végpontok és az üzleti logika automatizált validációjához.
* **Swagger / OpenAPI**: Interaktív, automatikusan generált API dokumentáció és végpont-tesztelési felület biztosítása.

---

## Építészeti és szoftverfejlesztési mérföldkövek

### 1. Logikai törlés (Soft Delete) architektúra
Egyedi Prisma kiterjesztés fejlesztése a törlési műveletek logikai szűrésére. A rendszer automatikusan kezeli a rekordok elrejtését a lekérdezések során anélkül, hogy az adatbázisból fizikailag eltávolítaná azokat, megőrizve ezzel a történeti adatkonzisztenciát és a kapcsolati integritást.

### 2. GDPR-kompatibilis fájlfeldolgozás
Biztonságos fájlkezelési folyamat implementálása, amely során a hallgatói dokumentumok nem perzisztálódnak a helyi szerverlemezen. A kérések feldolgozása memóriapufferben történik, ahonnan közvetlenül titkosított csatornán keresztül kerülnek elküldésre vagy tárolásra, minimalizálva a személyes adatok szivárgásának kockázatát.

### 3. Rétegelt architektúra (Layered Architecture)
A tiszta kód (Clean Code) irányelveinek megfelelő szigorú rétegződés alkalmazása:
* **Route réteg**: A végpontok konfigurációja és a middleware-ek (hitelesítés, validáció, rate limit) bekötése.
* **Controller réteg**: A HTTP request-response ciklus kezelése, bejövő paraméterek átadása és válaszformázás.
* **Service réteg**: Az üzleti logika kizárólagos helye, tranzakcionális adatbázis-műveletek végrehajtása.
* **Validation (Schema) réteg**: Deklaratív bemeneti adatvalidáció (Zod segítségével), amely garantálja, hogy csak tiszta és biztonságos adatok érjék el a szolgáltatási réteget.
