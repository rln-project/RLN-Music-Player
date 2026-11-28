# RLN Music Player 1.2.0 · Fase 1

La base è il sorgente Global 1.1.0 build 51, presente in `14177275b62e0e5d1e2ffdb79dca2620bce972e0`. Il workflow [Global build 51](https://github.com/rln-project/RLN-Music-Player/actions/runs/45076540466) ha verificato il blob sorgente `b712d28c534d798dff58af1f647ab97bba4ff0fe`. Questo ramo usa lo stesso archivio di partenza e conserva il layout che i workflow estraggono.

## Diagnostica

- `PlaybackDiagnostics.kt` mantiene in memoria gli ultimi 160 eventi con tempi monotoni e un numero di traccia locale al processo. Il report distingue buffering, riproduzione, errori, variazioni di rete, connessione Cast e decisioni già prese dal player per attesa, nuovo tentativo, fallback o salto.
- `PlaybackService.kt` osserva i callback e i rami già esistenti. `SongSources.kt` annota i rifiuti HTTP di URL YouTube senza assegnarli al brano corrente, perché il caricamento può essere in prefetch.
- `DiagnosticRedactor.kt` oscura gli URL completi e i formati di credenziali riconosciuti. La redazione si applica al buffer dell'app, ai percorsi di logging RLN e Media3 interessati, ai nuovi crash salvati e al testo finale esportato. Il report omette il dump completo delle preferenze di riproduzione.
- Il comando **Export diagnostics** usa il selettore di condivisione Android. Il report può ancora contenere informazioni sul dispositivo e dettagli sull'ascolto presenti nei log. Va riletto prima di condividerlo.

## Controlli

I test JVM `DiagnosticRedactorTest` e `PlaybackDiagnosticsTest` coprono credenziali in intestazioni, eccezioni e URL, idempotenza della redazione, durata del buffering, correlazione per traccia e limite del buffer. `AppLogTest` verifica il comportamento del registro esistente.

I sei test mirati delle tre classi sono passati con JUnit 4 su JVM. Il confronto degli archivi estratti ha confermato che le modifiche al sorgente riguardano solo i file di diagnostica e i relativi test in `core`.

### SDK e build Global di test

Il pacchetto ufficiale è `platforms;android-37.0`. Con la sola impostazione `compileSdk = 37`, AGP nel precedente ambiente locale cercava il target `android-37`. Impostare anche `compileSdkMinor = 0` in `:app` e `:core` seleziona esplicitamente la piattaforma installata. I livelli API restano 37 per la compilazione, 36 per il target runtime e 26 come minimo. In locale serve un JDK 21 completo di `javac`.

Il workflow del ramo diagnostica esegue le suite JVM debug e release di `:app` e `:core`, firma la build universale con la chiave RLN e verifica il certificato contro la build 51. Controlla l'identità del pacchetto, la versione, i livelli SDK, le quattro ABI e la presenza delle librerie audio native. La prova di aggiornamento installa la build 51 e poi la 52 sullo stesso AVD. L'artefatto viene caricato in Actions dopo questi controlli.

Le quattro suite JVM sono passate in locale e su GitHub Actions. In locale i report contano 2.248 casi, 2.218 passati e 30 saltati: per ogni variante, 14 esportazioni di screenshot README che richiedono immagini esterne e un test che richiede relay live. L’activity di test Compose è disponibile nel manifest solo durante l’invocazione Gradle delle suite. La validazione verifica anche la sua assenza dall’APK firmato.

Per la verifica Android usare `./gradlew :core:testDebugUnitTest :app:testDebugUnitTest` dopo aver estratto il sorgente in una directory di lavoro. Le prove su dispositivo da eseguire prima del rilascio sono:

1. Riprodurre uno stream, passare fra Wi-Fi e dati mobili ed esportare il report. Cercare transizioni di rete, buffering ed esito. Usare credenziali di prova per verificare la redazione.
2. Ripetere con Google Cast, radio HLS, un file locale e un download offline. Confrontare il comportamento della riproduzione con Global build 51.
3. Aprire ed esportare il report da smartphone e tablet, in verticale e orizzontale. Verificare l'aggiornamento in-app nel normale ciclo di validazione firmato.

Il [workflow di validazione riuscito](https://github.com/rln-project/RLN-Music-Player/actions/runs/46085943944) ha verificato la firma originale, le quattro ABI e l'aggiornamento in sede 51 → 52 su Android 15 x86_64. Entrambe le installazioni sono riuscite, il timestamp della prima installazione è rimasto identico e l'app si è avviata dopo l'aggiornamento. Il [pacchetto di test Actions](https://github.com/rln-project/RLN-Music-Player/actions/runs/46085943944/artifacts/15239811056) contiene l'APK universale e il relativo checksum.

La build di test usa `versionName=1.2.0` e `versionCode=52`. Il workflow di rilascio e la configurazione della firma sono quelli della build 51.
