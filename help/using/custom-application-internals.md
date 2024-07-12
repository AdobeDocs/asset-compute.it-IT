---
title: Comprendere il funzionamento di un’applicazione personalizzata
description: Funzionamento interno dell'applicazione personalizzata  [!DNL Asset Compute Service]  per comprendere il funzionamento.
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '691'
ht-degree: 0%

---

# Interni di un’applicazione personalizzata {#how-custom-application-works}

Utilizza l’illustrazione seguente per comprendere il flusso di lavoro end-to-end quando una risorsa digitale viene elaborata da un client utilizzando un’applicazione personalizzata.

![Flusso di lavoro applicazione personalizzato](assets/customworker.svg)

*Figura: passaggi necessari durante l&#39;elaborazione di una risorsa utilizzando l&#39;Adobe [!DNL Asset Compute Service].*

## Registrazione {#registration}

Il client deve chiamare [`/register`](api.md#register) una volta prima della prima richiesta a [`/process`](api.md#process-request) in modo da poter impostare e recuperare l&#39;URL del journal per la ricezione di eventi di Adobe [!DNL I/O Events], ad Adobe l&#39;Asset compute.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

La libreria JavaScript [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) può essere utilizzata nelle applicazioni NodeJS per gestire tutti i passaggi necessari dalla registrazione all&#39;elaborazione alla gestione asincrona degli eventi. Per ulteriori informazioni sulle intestazioni richieste, vedere [Autenticazione e autorizzazione](api.md).

## Elaborazione {#processing}

Il client invia una richiesta [elaborazione](api.md#process-request).

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

Il client è responsabile della corretta formattazione delle rappresentazioni con URL prefirmati. La libreria JavaScript [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) può essere utilizzata nelle applicazioni NodeJS per pre-firmare gli URL. Attualmente la libreria supporta solo l’archiviazione BLOB di Azure e i contenitori AWS S3.

La richiesta di elaborazione restituisce un `requestId` che può essere utilizzato per il polling di [!DNL Adobe I/O] eventi.

Di seguito è riportato un esempio di richiesta di elaborazione personalizzata dell’applicazione.

```json
{
    "source": "https://www.adobe.com/some-source-file.jpg",
    "renditions" : [
        {
            "worker": "https://my-project-namespace.adobeioruntime.net/api/v1/web/my-namespace-version/my-worker",
            "name": "rendition1.jpg",
            "target": "https://some-presigned-put-url-for-rendition1.jpg",
        }
    ],
    "userData": {
        "my-asset-id": "1234567890"
    }
}
```

[!DNL Asset Compute Service] invia le richieste di rendering dell&#39;applicazione personalizzata all&#39;applicazione personalizzata. Utilizza un POST HTTP per l’URL dell’applicazione fornito, che è l’URL dell’azione web protetta di App Builder. Tutte le richieste utilizzano il protocollo HTTPS per massimizzare la sicurezza dei dati.

L&#39;[SDK di Asset compute](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) utilizzato da un&#39;applicazione personalizzata gestisce la richiesta HTTP POST. Gestisce inoltre il download dell&#39;origine, il caricamento delle rappresentazioni, l&#39;invio dell&#39;Adobe [!DNL I/O Events] e la gestione degli errori.

<!-- TBD: Add the application diagram. -->

### Codice applicazione {#application-code}

Il codice personalizzato deve fornire solo un callback che accetta il file di origine disponibile localmente (`source.path`). `rendition.path` è la posizione in cui inserire il risultato finale di una richiesta di elaborazione di risorse. L&#39;applicazione personalizzata utilizza il callback per trasformare i file di origine disponibili localmente in un file di rendering utilizzando il nome passato (`rendition.path`). Un&#39;applicazione personalizzata deve scrivere in `rendition.path` per creare una copia trasformata:

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

// worker() is the entry point in the SDK "framework".
// The asynchronous function defined is the rendition callback.
exports.main = worker(async (source, rendition) => {

    // Tip: custom worker parameters are available in rendition.instructions.
    console.log(rendition.instructions.name); // should print out `rendition.jpg`.

    // Simplest example: copy the source file to the rendition file destination so as to transfer the asset as is without processing.
    await fs.copyFile(source.path, rendition.path);
});
```

### Scarica i file sorgente {#download-source}

Un’applicazione personalizzata tratta solo i file locali. [Asset Compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) gestisce il download del file di origine.

### Creazione rappresentazione {#rendition-creation}

L&#39;SDK chiama una funzione di callback [copia trasformata](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) asincrona per ogni copia trasformata.

La funzione di callback ha accesso agli oggetti [source](https://github.com/adobe/asset-compute-sdk#source) e [rendition](https://github.com/adobe/asset-compute-sdk#rendition). `source.path` esiste già ed è il percorso della copia locale del file di origine. `rendition.path` è il percorso in cui deve essere archiviata la rappresentazione elaborata. A meno che il flag [disableSourceDownload](https://github.com/adobe/asset-compute-sdk#worker-options-optional) non sia impostato, l&#39;applicazione deve utilizzare esattamente `rendition.path`. In caso contrario, l’SDK non è in grado di individuare o identificare il file di rappresentazione e genera un errore.

L&#39;eccessiva semplificazione dell&#39;esempio viene eseguita per illustrare e concentrarsi sull&#39;anatomia di un&#39;applicazione personalizzata. L&#39;applicazione copia semplicemente il file di origine nella destinazione della copia trasformata.

Per ulteriori informazioni sui parametri di callback della rappresentazione, vedere [Asset Compute SDK API](https://github.com/adobe/asset-compute-sdk#api-details).

### Carica rappresentazioni {#upload-rendition}

Dopo la creazione e l&#39;archiviazione di ogni rendering in un file con il percorso fornito da `rendition.path`, l&#39;[Asset Compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) carica ogni rendering in un archivio cloud (AWS o Azure). Un’applicazione personalizzata ottiene più rappresentazioni contemporaneamente se, e solo se, la richiesta in ingresso ha più rappresentazioni che puntano allo stesso URL dell’applicazione. Il caricamento nell’archiviazione cloud viene eseguito dopo ogni rendering e prima di eseguire il callback per il rendering successivo.

`batchWorker()` ha un comportamento diverso. Elabora tutte le rappresentazioni e solo dopo che tutte sono state elaborate, le carica.

## [!DNL Adobe I/O] eventi {#aio-events}

L&#39;SDK invia l&#39;Adobe [!DNL I/O Events] per ogni rappresentazione. Questi eventi sono di tipo `rendition_created` o `rendition_failed` a seconda del risultato. Per ulteriori informazioni, vedere [Asset compute di eventi asincroni](api.md#asynchronous-events).

## Ricevi [!DNL Adobe I/O] eventi {#receive-aio-events}

Il client esegue il polling del giornale di registrazione Adobe [!DNL I/O Events] in base alla logica di utilizzo. L&#39;URL iniziale del diario è quello specificato nella risposta API `/register`. Gli eventi possono essere identificati utilizzando `requestId` presente negli eventi ed uguale a quello restituito in `/process`. Ogni rendering ha un evento separato che viene inviato non appena il rendering è stato caricato (o non è riuscito). Quando riceve un evento corrispondente, il client può visualizzare o gestire in altro modo le rappresentazioni risultanti.

La libreria JavaScript [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) semplifica il polling del journal utilizzando il metodo `waitActivation()` per ottenere tutti gli eventi.

```javascript
const events = await assetCompute.waitActivation(requestId);
await Promise.all(events.map(event => {
    if (event.type === "rendition_created") {
        // get rendition from cloud storage location
    }
    else if (event.type === "rendition_failed") {
        // failed to process
    }
    else {
        // other event types
        // (could be added in the future)
    }
}));
```

Per informazioni dettagliate su come ottenere gli eventi del diario, vedere l&#39;Adobe [[!DNL I/O Events] API](https://developer.adobe.com/events/docs/guides/api/journaling_api/).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
