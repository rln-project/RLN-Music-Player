# RLN Music Player 1.2.0 · Fase 1

La base è il sorgente Global 1.1.0 build 51, presente in `14177275b62e0e5d1e2ffdb79dca2620bce972e0`. Il workflow [Global build 51](https://github.com/rln-project/RLN-Music-Player/actions/runs/45076540466) ha verificato il blob sorgente `b712d28c534d798dff58af1f647ab97bba4ff0fe`. Questo ramo usa lo stesso archivio di partenza e conserva il layout che i workflow estraggono.

## Diagnostica

- `PlaybackDiagnostics.kt` mantiene in memoria gli ultimi 160 eventi con tempi monotoni e un numero di traccia locale al processo. Il report distingue buffering, riproduzione, errori, variazioni di rete, connessione Cast e decisioni già prese dal player per attesa, nuovo tentativo, fallback o salto.
- `PlaybackService.kt` osserva i callback e i rami già esistenti. `SongSources.kt` annota i rifiuti HTTP di URL YouTube senza assegnarli al brano corrente, perché il caricamento può essere in prefetch.
- `DiagnosticRedactor.kt` oscura gli URL completi e i formati di credenziali riconosciuti. La redazione si applica al buffer dell'app, ai percorsi di logging RLN e Media3 interessati, ai nuovi crash salvati e al testo finale esportato. Il report omette il dump completo delle preferenze di riproduzione.
- Il comando **Export diagnostics** usa il selettore di condivisione Android. Il report può ancora contenere informazioni sul dispositivo e dettagli sull'ascolto presenti nei log. Va riletto prima di condividerlo.

## Controlli

I test JVM `DiagnosticRedactorTest` e `PlaybackDiagnosticsTest` coprono credenziali in intestazioni, eccezioni e URL, idempotenza della redazione, durata del buffering, correlazione per traccia e limite del buffer. `AppLogTest` verifica il comportamento del registro esistente.

Per la verifica Android usare `./gradlew :core:testDebugUnitTest :app:testDebugUnitTest` dopo aver estratto il sorgente in una directory di lavoro. Le prove su dispositivo da eseguire prima del rilascio sono:

1. Riprodurre uno stream, passare fra Wi-Fi e dati mobili ed esportare il report. Cercare transizioni di rete, buffering ed esito. Usare credenziali di prova per verificare la redazione.
2. Ripetere con Google Cast, radio HLS, un file locale e un download offline. Confrontare il comportamento della riproduzione con Global build 51.
3. Aprire ed esportare il report da smartphone e tablet, in verticale e orizzontale. Verificare l'aggiornamento in-app nel normale ciclo di validazione firmato.

Le impostazioni di firma e il workflow di rilascio rimangono quelli della build 51. Questa fase usa il numero di build di partenza e non definisce il numero definitivo di 1.2.0.
