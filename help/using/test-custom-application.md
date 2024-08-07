---
title: Test ed esecuzione del debug  [!DNL Asset Compute Service] dell'applicazione personalizzata
description: Test ed esecuzione del debug  [!DNL Asset Compute Service] dell'applicazione personalizzata.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '775'
ht-degree: 0%

---

# Testare ed eseguire il debug di un&#39;applicazione personalizzata {#test-debug-custom-worker}

## Eseguire unit test per un&#39;applicazione personalizzata {#test-custom-worker}

Installa [Docker Desktop](https://www.docker.com/get-started) nel computer. Per testare un processo di lavoro personalizzato, eseguire il comando seguente nella radice dell&#39;applicazione:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Questo comando esegue un framework di unit test personalizzato per azioni dell&#39;applicazione di Asset compute nel progetto come descritto di seguito. È collegato tramite una configurazione nel file `package.json`. È inoltre possibile disporre di unit test JavaScript, ad esempio Jest. `aio app test` esegue entrambi.

Il plug-in [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) è incorporato come una dipendenza di sviluppo nell&#39;app dell&#39;applicazione personalizzata, pertanto non è necessario installarlo nei sistemi di build/test.

### Framework di test delle unità applicative {#unit-test-framework}

L&#39;Asset compute di framework di unit test dell&#39;applicazione consente di testare le applicazioni senza scrivere codice. Si basa sul principio di origine per il rendering del file delle applicazioni. È necessario configurare una determinata struttura di file e cartelle per definire i casi di test con file di origine di test, parametri facoltativi, rappresentazioni previste e script di convalida personalizzati. Per impostazione predefinita, le rappresentazioni vengono confrontate per l&#39;uguaglianza dei byte. Inoltre, i servizi HTTP esterni possono essere facilmente presi in giro utilizzando semplici file JSON.

### Aggiungi test {#add-tests}

I test sono previsti all&#39;interno della cartella `test` al livello principale del progetto. I test case per ogni applicazione devono trovarsi nel percorso `test/asset-compute/<worker-name>`, con una cartella per ogni test case:

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

Guarda [esempio di applicazioni personalizzate](https://github.com/adobe/asset-compute-example-workers/) per alcuni esempi. Di seguito è riportato un riferimento dettagliato.

### Output di prova {#test-output}

La directory `build` nella directory principale dell&#39;app Adobe Developer App Builder contiene i risultati dettagliati dei test e i registri dell&#39;applicazione personalizzata. Questi dettagli vengono visualizzati anche nell&#39;output del comando `aio app test`.

### Mascherare i servizi esterni {#mock-external-services}

È possibile simulare chiamate di servizio esterne all&#39;interno delle azioni creando `mock-<HOST_NAME>.json` file per gli scenari di test. HOST_NAME è l&#39;host specifico che si intende imitare. Un esempio di caso d’uso è un’applicazione che effettua una chiamata separata a S3. La nuova struttura di test sarà simile alla seguente:

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

Il file fittizio è una risposta HTTP in formato JSON. Per ulteriori informazioni, consulta [questa documentazione](https://www.mock-server.com/mock_server/creating_expectations.html). Se ci sono più nomi host da simulare, definire più file `mock-<mocked-host>.json`. Di seguito è riportato un esempio di file fittizio per `google.com` denominato `mock-google.com.json`:

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

L&#39;esempio `worker-animal-pictures` contiene un [file fittizio](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) per il servizio Wikimedia con cui interagisce.

#### Condivisione di file tra test case {#share-files-across-test-cases}

L&#39;Adobe consiglia di utilizzare i symlink relativi se si condividono `file.*`, `params.json` o `validate` script in più test. Sono supportati con Git. Assicurati di assegnare ai file condivisi un nome univoco, in quanto potrebbero essere presenti file diversi. Nell’esempio seguente i test si combinano e corrispondono a pochi file condivisi e ai loro:

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### Test degli errori previsti {#test-unexpected-errors}

I casi di test degli errori non devono contenere un file `rendition.*` previsto e devono definire il file `errorReason` previsto all&#39;interno del file `params.json`.

>[!NOTE]
>
>Se un test case non contiene un file `rendition.*` previsto e non definisce il `errorReason` previsto all&#39;interno del file `params.json`, si presume che si tratti di un errore con qualsiasi `errorReason`.

Struttura del test case di errore:

```json
<error_test_case>/
    file.jpg
    params.json
```

File di parametri con motivo errore:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

Visualizza un elenco completo e una descrizione di [motivi di errore Asset compute](https://github.com/adobe/asset-compute-commons#error-reasons).

## Eseguire il debug di un&#39;applicazione personalizzata {#debug-custom-worker}

Nei passaggi seguenti viene illustrato come eseguire il debug dell&#39;applicazione personalizzata utilizzando Visual Studio Code. Consente di visualizzare registri live, punti di interruzione di hit e codice step-through, nonché il ricaricamento in tempo reale delle modifiche al codice locale a ogni attivazione.

`aio` preconfigurato automatizza molti di questi passaggi. Vai alla sezione Debug dell&#39;applicazione nella [documentazione di Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app). Per il momento, i passaggi seguenti includono una soluzione alternativa.

1. Installa la versione più recente di [wskdebug](https://github.com/apache/openwhisk-wskdebug) da GitHub e la [ngrok](https://www.npmjs.com/package/ngrok) facoltativa.

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Aggiungi delle impostazioni utente al file JSON. Continua a utilizzare il debugger del codice di Visual Studio precedente. Il nuovo ha [alcuni problemi](https://github.com/apache/openwhisk-wskdebug/issues/74) con wskdebug: `"debug.javascript.usePreview": false`.
1. Chiudere tutte le istanze di app aperte tramite `aio app run`.
1. Distribuire il codice più recente utilizzando `aio app deploy`.
1. Eseguire solo lo strumento di sviluppo Asset Compute utilizzando `aio asset-compute devtool`. Tienilo aperto.
1. Nell&#39;editor di codice di Visual Studio aggiungere la seguente configurazione di debug a `launch.json`:

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   Recupera `ACTION NAME` dall&#39;output di `aio app deploy`.

1. Selezionare `wskdebug worker` dalla configurazione di esecuzione/debug e premere l&#39;icona di riproduzione. Attendere che venga avviato finché non viene visualizzato **[!UICONTROL Pronto per le attivazioni]** nella finestra **[!UICONTROL Console di debug]**.

1. Fare clic su **[!UICONTROL esegui]** in Devtool. È possibile visualizzare le azioni in esecuzione nell&#39;editor di codice di Visual Studio e i registri iniziano a essere visualizzati.

1. Imposta un punto di interruzione nel codice. Quindi esegui di nuovo e dovrebbe colpire.

Eventuali modifiche al codice vengono caricate in tempo reale e diventano effettive non appena si verifica l’attivazione successiva.

>[!NOTE]
>
>Per ogni richiesta nelle applicazioni personalizzate sono presenti due attivazioni. La prima richiesta è un’azione web che si richiama in modo asincrono nel codice SDK. La seconda attivazione è quella che colpisce il codice.
