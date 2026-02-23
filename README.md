## I. Android általános bemutatása (nagyvonalakban)

### Alapvető fogalmak:

- **Android projekt**: modul(ok)ból álló fejlesztési egység, mely modulok lehetnek `library` jellegű
  modulok (projekten belül definiált, vagy repositoryból letöltött), és `application` jellegű (
  futtatható) modulok.
- **Modul**: egy önállóan fordítható egység, amelynek lehetnek függőségei más modulokra.
    - **Library modul**: olyan modul, amely nincs applikációként kezelve, tehát nincs entry point
      meghatározva.
    - **Application modul**: (futtatható applikáció) van entrypoint és `applicationId` meghatározva
      a modul konfigjaiban,. (ebből adódik, hogy más applikáció típusú modult nem lehet függőségként
      beállítani számára, mert akkor több entry point lenne az applikációnak, összeakadnának)
- **Alkalmazás**: egy applikáció modul (és függőségeinek) buildelt bináris kódja (.apk) ami
  andoridos telefonra fel lehet telepíteni. Egy android projektben több alkalmazás is lehet (= több
  application jellegű modul), amik külön apk-ként telepíthetőek, és lehetnek akár közös függőségeik
  is (pl `core` library modul).
- **Application class**: (`android.app.Application`) az alkalmazás életciklusát követő osztály,
  amely az alkalmazás életciklusának eseményeit kezeli (pl. indulás, leállítás, stb.).
- **Application Context** Az `Application` class-ból érjük el, az alkalmazás teljes életciklusán át
  él, lehetővé teszi, hogy hozzáférjünk az Android operációs rendszer különböző szolgáltatásaihoz,
  mint például a rendszer erőforrások, hálózati szolgáltatások, adatbázisok, és más globális
  funkciók
- **Activity Context**: az `Activity` osztályból érjük el, az adott activity életciklusa alatt él (
  ezért nem szabad az üzleti logikába beinjektálni, mert memóriaszivárgás lesz belőle), és az
  activity UI elemekhez való hozzáférést biztosítja.
- **Fő komponens**: Android alkalmazás fő részei, amikhez az androoid oprendszer közvetlenül tud
  kapcsolódni, ezért is kell az `AndroidManifest.xml`-ban regisztrálni őket:
    - Activity class: Az alkalmazás egy képernyőjét reprezentálja. (mindig kell egy entry point
      activity class-t meghatározni, ami az applikáció indulásakor elindul)
        - FONTOS: rugalmasan kezeli az oprendszer: nem biztosított, hogy az alkalmazás életciklusa
          alatt folyamatosan a memóriában maradnak, ezért több életciklusuk van, amikhez tudunk
          hozzárendelni callback-eket (pl. onCreate, onStart, onResume, onPause, onStop, onDestroy),
          ezért ide UI logikát szabad csak írni, üzleti logikát nem, mert nem garantált, hogy a
          következő életciklusban is elérhető lesz az adott állapot, az UI elemek, meg statikus
          elemek, nem veszik el fontos adat stb.
        - lehetnek "alegységei" is: **Fragment**: UI szempopntból azt tudja mint az activity, csak
          nem önállóan fut, hanem egy activity részeként, és lehet több fragment is egy
          activity-ben, és egy fragment is több activity-ben.
    - Service class: Háttérben futó komponens, amely hosszú ideig fut, anélkül, hogy felhasználói
      felületet biztosítana.
    - Broadcast Receiver class: Az alkalmazás értesítéseket fogad és reagál rájuk.
    - Content Provider class: Az alkalmazás adatokat oszt meg más alkalmazásokkal.
- **Intent**: olyan objektum, amely egy másik
  komponenshez (Activity, Service, Broadcast Receiver, Content Provider) irányítja az alkalmazás
  vezérlését, ez akár lehet egy másik applikáció komponense is egyébként. Az Intentek lehetnek
  implicit (nem specifikálják a célt) vagy explicit (megadják a
  célt).
- **Plugin**: pluginok: kiegészítők, amelyek segítségével a Gradle build rendszer képes az adott
  modulokat build-elni és JVM bytekóddá alakítani (mivel az alap környezet androidhoz a JAVA, így
  buildnél JVM bytecode-t kell generálni) pl.:
    - maga a kotlin programozási nyelv is egy pluginon keresztül
      értelmeződik (`org.jetbrains.kotlin.android`), ami a build során JVM bytecode-t generál a
      kotlin forráskódból.
    - annotációs pluginok: annotációk használatát teszi lehetővé, amelyek segítségével pl. a HILT
      plugin képes az Dependency injection-t megvalósítani annotácik alapján.
    - `com.android.application`: application moduloknál kell megadni, ez a plugin azért felel, hogy
      az adott modulból készüljön egy futtatható .apk fájl
- **Gradle build**: a Gradle build egy automatizált folyamat, amely során a Gradle build rendszer a
  projekt forráskódját, függőségeit (library, plugin), tesztjeit és egyéb aspektusait kezelve
  készíti el a végső futtatható .apk fájlt. A Gradle build rendszer a modulok `build.gradle`
  fájlokban definiált konfigurációk alapján végzi a build folyamatot.

### Android projekt felépítése

- `{root}/settings.gradle` - kiindulópontja egy android projektnek, itt definiáljuk a projektben
  található modulokat, (gyárilag egy darab `app` modul)
  amiket a build során figyelmebe vesz. pl: `include ':core'` -> `{root}/core` mappában lévő modul)
- ezekben a modulokban kötelező lennie két (config) fájlnak:

1. `{root}/{module}/build.gradle`: aminek a tartalma az adott modulra vonatkozóan:

- Dependencies: Függőségek meghatározása (pl. library-k, amelyek szükségesek az alkalmazás
  futtatásához).
- Build Settings: Build konfigurációk, mint például a compileSdkVersion, minSdkVersion,
  targetSdkVersion, verziószámok stb.
- Plugins: Pluginok meghatározása
- Build Variants: Különböző build variánsok (pl. debug, release) beállítása.

2. `{root}/{module}/src/main/AndroidManifest.xml` A modul (alkalmazás modul esetében magának az
   alkalmazásnak) metaadatait tartalmazza, és információt nyújt az Android rendszernek:
    - Application Components: az alkalmazás fő komponenseinek deklarálása (activity, service,
      broadcast receiver, content provider).
    - Permissions: Az alkalmazás által igényelt jogosultságok meghatározása (pl. internet
      hozzáférés, kamera használat).
    - App Information: (application modulnál)  Alapvető információk az alkalmazásról, mint például
      az alkalmazás neve, ikonjai, témája stb.
    - Intent Filters: Az alkalmazás által kezelt intentek meghatározása.

### HILT Dependency injection plugin

A memóriabiztonságos context kezeléshez, és nem mellesleg dependency inversion elvnek megfelelő
működéshez valamilyen
dependency injection keretrendszerre van szükség. A HILT-t használjuk, mert ez az Android által is
ajánlott.  
Doksi: [HILT](https://developer.android.com/training/dependency-injection/hilt-android)

Használata (elég sok mindent tud lsd. fent a doksit, ez csak egy része, amit legtöbbszr használunk):

- Az aktuális `Activity vagy Fragment`-be (vagy `Service` (háttérművelet) -be) kell
  a `@AndroidEntryPoint` annotációt beírni, így lesz elérhető az adott `Activity vagy Fragment`
  alatt, (minden pillanatban valamilyen Activity / Fragment van a képernyőn)
- automatikus injektálás konstruktorba:

``` 
    class AuthRepository @Inject constructor(
        private val sessionDataUtil: SessionDataUtilInterface,
    ) 
```

itt látszik, hogy interface-t (`SessionDataUtilInterface`) adtunk meg, ami nyilván nem lehetne
paraméterként beinjektálni, mivel abstract osztály, de a HILT ezt a konkrét osztályra fogja
cserélni:  
a HILT konfigurációs class-ban a `SessionDataUtilInterface` -hoz hozzá lett rendelve
a `SessionDataUtil` konkrét osztály, és így a
hilt a megadott konkrét osztályt fogja példányosítani, ahol a `SessionDataUtilInterface`-t
hivatkozzák meg.  
Konfiguráció osztály pl. core modul: `/core/di/CoreModule.kt`: (tetszőleges helyen jellemzően
modulonként
egy ilyen @Module konfigurációs class-t kell definiálni)

Így megvalósul a dependency inversion elv, mert a `AuthRepository` osztály nem tudja, hogy milyen
konkrét osztályt kapott csak azt, hogy milyen interface-nek felel meg a kapott osztály.

```
@Module
@InstallIn(SingletonComponent::class)
class CoreModule {
    @Provides
    @Singleton
    fun provideSessionDataUtil(
        sessionDataUtil: SessionDataUtil,
    ): SessionDataUtilInterface {
        return sessionDataUtil
    }
```

ApplicationContext injektálása:

``` 
class SessionDataUtil @Inject constructor(
    @ApplicationContext private val context: Context,
) 
```

### Android architektúra rétegei

1. UI réteg:

- Activity/Fragment többek között: Az alkalmazás egy képernyőjét reprezentálja, az alkalmazás egyik
  fő komponense, amely
  felhasználói felületet biztosít.
- ViewModel: UI és Domain réteg közötti kapcsolatot biztosítja, UI logika, UI állapotok kezelése (
  Activity fragment-te ellentétben a memóriában marad az alkalmazás életciklusa alatt)
- View: Az Activity vagy Fragment UI elemeinek megjelenítése, és az UI elemekkel való interakció
  kezelése. (pl. gombnyomásra való reagálás)
- Layout: Az Activity vagy Fragment UI elemeit és elemeinek elrendezését definiálja
- Toast: Rövid értesítési üzenetek megjelenítése az alkalmazás felhasználója számára.

2. Domain réteg többek között:

- ViewModel: UI és Domain réteg közötti kapcsolatot biztosítja, UI logika, UI állapotok kezelése
- LiveData: Élő adatokat reprezentáló osztály, amely figyeli az adatok változását, és értesíti a
  megfigyelőket. (ezek a megfigyelők jellemzően UI elemek, pl. TextView) Ez azt jelenti hogy nem
  direktben határozzuk meg a UI adatokat, hanem a LiveData figyeli az adatok változását, és ha
  változás van, akkor értesíti a megfigyelőket, és azok frissítik az UI-t.
  a livedata-kat bármilyen művelettel frissíthetjük (pl. adatbázisból, API-ból, stb.)
- UseCase / Interactor: Az alkalmazás üzleti logikáját tartalmazó osztály, amely az adatok
  feldolgozását és az üzleti szabályok végrehajtását végzi. (php környezetben ezeket a service-nek
  nevezzük)

3. Data réteg:

- DataSource: Az adatok forrását reprezentáló osztály, amely az adatok lekérdezését és tárolását
  végzi. (pl. adatbázis, API, fájl stb.)
- Repository: Az adatok (DataSource)-ből való lekérdezését és tárolását végző osztály, amely az
  adatokat a
  Domain réteg számára biztosítja.
- SharedPreferences: Kulcs-érték párok tárolására szolgáló osztály, amely az alkalmazás életciklusa
  alatt érvényes adatok tárolására használható.
- SQLite, Firebase: lokális és távoli (központi) adatbázisok

pl. egy adat lekérdezési folyamat: az UI által kezdeményezés - backend API hívás - aztán vissza az
UI megjelenítésig:
Frissítés gombra nyomva lekérdezzük, és megjelentjük az új híreket:

- activity: NewsActivity
- res/layout/news.xml layoutban definiáljuk a "news" textview UI elemet és a refreshButton gomb UI
  elemet
- NewsActivity-hez kapcsoljuk a news.xml layout-ot
- NewsActivity-hez kapcsoljuk a NewsViewModel osztályt
- NewsViewModel-ben létrehozzuk a híreket tartalmazó newsData string LiveData-t
- NewsActivity-ben a news textview-t a viewmodelben lévő LiveData-hoz kötjük, tehát ha frissül a
  newsData livedata, akkor automatikusan frissül a news" textview UI elem a a newsData értékével
- NewsActivity-ben a refreshButton UI elemre regisztrálunk egy observert, ha megnyomják, akkor hívja
  meg az activityhez kapcsolt NewsViewModel checkNews() metódusát.
- a NewsViewModel a getNews() metódusban a híreket lekérjük az NewsRepository-ból
- az NewsRepository API hívással lekéri az adatot, és visszaadja az adatot az NewsViewModel-nek
- NewsViewModel getNews() frissíti a newsData LiveData-t
- a newsData LiveData frissülésére a NewsActivity-ben feliratkozott "news" textview UI elem is
  frissül az új hírekkel

## II. XXXXX projekt felépítése

A XXXXXX általános funkciói illetve egyéb hasznos util-ek a `core` "library" típusú modulban
vannak implementálva,

A `core` modul mellé lehet "applikáció" típúsú modulokat készíteni, pl (app_driver - sofőr
applikáció), amik a core modul funkcióit használhatják. Ebből következik, hogy ehhez a projekthez
több applikációt is tudunk készíteni, amik külön apk-ként telepíthetőek, tehát a telefonunkon is
különböző applikációként jelennek meg.

Fontos, hogy a `core` modul nem függhet egyik alkalmazás modultól sem, tehát nem "kérdezhet ki" a
modulon kívüli namespace-ekbe.

### Core modul funkciók

az alábbiakban a `core` modulban található funkciókat mutatom be, hozzá példákat a használatukra.

#### Session Data

Kicsit csalóka az elnevezés, mert nincs olyan androidban, hogy "session", egy applikációnak lehet
ilyen
key/value jellegű tárolót kialakítani a `SharedPreferences` segítségével, (ez technikailag fájlba
írja/olvassa az értékeket,
és ehhez a fájlhoz csak az adott applikáció fér hozzá.)

- SessionDataUtilInterface
  A `SessionDataUtilInterface` hez tartozó konkrét osztály intézi a ennek a kezelését, lehet
  titkosítva is
  tárolni adatot,
  `SessionDataUtilInterface::setEncriptedData()` mivel érzékeny adatot nem szabad
  `SharedPreferences`-ben a tárolóban tárolni, így mentés előtt titkosítjuk.
- Authentikácó megszűnésénél a tároló tartalmát is explicit töröljük, így valósul meg a session
  jelleg.
- `SessionDataUtilInterface::setFlashMessage()` flash üzenetnek szánt stringeket tudunk menteni
  amit `getFlashMessage()` -vel tudunk előhozni és a flash jelleg miatt ilyenkor automatikusan törli
  is.
  Ezen keresztül tudunk egy átirányítás előtt üzenni, vagy exception dobás előtt üzenni a következő
  Activity-nek (képernyőnek)
  (persze csak ha a következő `Activity` megvizsgálja, hogy a SessionDataUtilInterface::
  getFlashMessage()
  -ben van-e tartalom.)

Ha lekértük a tokent, elmentjük a sessionbe:

```
    sessionDataUtil.setEncriptedData("token", token)
```

a sessionbe lementett tokent felhasználjuk az api hívsoknál:

```
    val request = Request.Builder()
            .url(BuildConfig.BASE_URL + path")
            .addHeader("Authorization", "Bearer " + tokenFromSession)
```

Flash message beállítása:

```
   sessionDataUtil.setFlashMessage("Sikeres kijelentkezés")
```

Flash message lekérése:

```    
  val flashData: String = sessionData.getFlashMessage()
        if (flashData.isNotEmpty()) {
            _flashMessage.value = flashData
        }        
```

#### Authentikáció

Teljeskörűen meg van oldva az authentikáció, mind a logikai, mind a frontend része:

- `AuthRepositoryInterface`
  végzi az authentikáviós API műveleteket + a tokent is menti a  (`SessionDataUtilInterface`)-en
  keresztül
  hogy az applikációban mindenhol hozzáférhessünk a tokenhez, további methodok:
    - ::login()
    - ::logout()
    - ::isTokenValid()
- `LoginActivity` / `LoginViewModel` pedig a bejelentkezési felületet (képernyőt), és login form
  feldolgozását végzi.

Ellenőrizzük, hogy érvényes--e a "session"-be (SharedPreferences) lévő token, ha nem,
akkor a LoginActivity-t indítjuk, és ha sikeres a login, bekerül a "session"-be a token:

``` 
    val isTokenValid = authRepository.isTokenValid()
    withContext(Dispatchers.Main) {
        _navigateToLogin.value = !isTokenValid
        _checkAuth.value = false
    }
```

```
    mainViewModel.navigateToLogin.observe(this, Observer { needLogin ->
            if (needLogin) {
                val intent = Intent(this, LoginActivity::class.java)
                startActivity(intent)
            }
    })
```

#### ApiDataSourceInterface

Általános HTTP api kliens osztály, egyszerűsíti az API kommunikációt, az egyes respository-k így
könnyen
tudnak API-n keresztül adatot lekérni, vagy adatot küldeni.

- a tokent automatikusan hozzáfűzi a kéréshez
- vizsgálja a status code-ot, a válasz üzenet message-t,
- lekezeli a hibákat.
- Stringben adja vissza a válasz `data` jsont:
    - `fun get(path: String, getParams: Map<String, String> = emptyMap()): String`
    - `fun post(path: String, body: String, params: Map<String, String>): String`

Lekérjük az alkalmazottak listáját (a GET param-okat lehet az URL-hez is írni, és második
paraméterként is megadni, sőt keverni is a kettőt):

```    
   val employee = apiDataSource.get("api/general/employee/employee-list?limit=50", mapOf("page" to "1"))
```

#### Signature Activity

Egy rajzolható felületet biztosít, és a rajzot elmenti .png fájlba, majd a fájl lokációját
visszaadja a hívó Activity-nek.

- `SignatureActivity` - a rajzolható felület
- `SignatureViewModel` - a rajzolás mentését végzi

``` 
    private val signatureLauncher = registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result ->
        if (result.resultCode == Activity.RESULT_OK) {
            val data = result.data?.getStringExtra("signature_location")
            binding!!.textHome.text = "Aláírás png fájl:\n $data"
        }
    }
    
    
    signatureButton.setOnClickListener {
        val intent = Intent(activity, SignatureActivity::class.java)
        signatureLauncher.launch(intent)
    }
    
```

#### GLog

Egy egyszerű logoló osztály, amely a `Log` osztályt terjeszti ki, ugyanúúgy statikus methodokat
használ,
így nenm kell példányosítani, itt azon felül, hogy a gyáeri `Log` osztályt használja,
lehetőségünk van Logstash-ba stb elküldeni a logokat, ha akarjuk.

``` 
    GLog.d("Debug üzenet")
    GLog.e("Hibaüzenet")
    GLog.i("Információs üzenet")
```

#### Hibakezelés

Az uncaught exception-öket (fő szálon lehetséges elkapni csak) a `GastroExceptionHandler::
handleException()` - hez tudjuk irányítani. (az adott applicatoin modul fő `Application` osztályában kell ezt beállítani:
`Thread.setDefaultUncaughtExceptionHandler { thread, exception -> GastroExceptionHandler.handleException(exception) }` )

Ez intézi a logolást és egy rövid hibaüzenettel tér vissza, amit ki tudnk íratni a usernek jellemzően Toast
message-be. Ez a hibaüzenet tartalmaz hibakódot is (4 karakter hosszú random betűk), hogy a user tudjon
valami egyedi azonosítót megadni a hibekeresésnél.

Ezek a saját exception-öket így kezeli le:
- `BackendErrorException` - visszaadott string: "**Központi hiba! hibakód: HJRT**"
- `UnauthorizedException` - visszaadott string: "**Nincs jogosultság!**"
- `ShowMessageException` - visszaadott string amit az exceptionben megadtunk  

Gyári exception-öknél csak a `SocketTimeoutException` kezeljük le külön:
- `SocketTimeoutException`-t visszaadott string: "**Hálózati hiba (időtúllépés)!**"
- Minden más exception: "**Hiba! hibakód: GRZS**"

### Egy applikáció (application modul) felépítése (Sofőr applikáció)

#### Entry point Activity: **MainActivity**

Kaptt egy gyár elrendezést oldalsó menüvel (NavigationDrawer), ez a MainActivity layoutjában van
intézve. A bal felső sarokba kiiratjuk a user nevét és beosztását.
Ez az egy Activity van az egész applikációban, ami  (Single Page Application)-höz
hasonlóan aloldalakat dinamikusan cseréli menüpont kiválasztástól függően: (aloldal = teljes
képeryős fragment)

- HomeFragment
- EmployeeFragment

Ez azért is előnyös, mert a MainActivity-ben lehet minden aloldalra érvényes műveleteket definiálni,
pl authentikáció ellenőrzés, flashMessage kezelés, stb. így ezeket nem kell minden al Fragmentben
külön kiépíteni, mivel az őket tároló MainActivity-ben már megvannak.

Az egyes fragmentekhez tartozó viewmodelben kérdezzük le a repositoryk-ból az adatokat, pl a
munkatársakat lsd.:

- ui/employee/EmployeeFragment
- ui/employee/EmployeeViewModel,
- data/employee/EmployeeRepository
- data/employee/Employee (data class)
- data/employee/EmployeeList (data class)
- res/layout/fragment_employee.xml (layout)

#### Többnyelvűség

Az ui elemeken meg lehet adni direktbe a szövegeket, de ez nem jó gyakorlat, mert kizárja a
többnyelvűség lehetőségét.
Helyette a res/value/string.xml -ben kell definiálni az egyes stringeket, és a ui elemekben
@id/string/... -vel könnyen tudunk rá hivatkozni, és később csak a string.xml -t kell lefordítani,
és
az android automatikusan alkalmazza a megfelelő nyelvi fájlt a telefonon bellított nyelv alapján.



