---
title: "[!DNL Asset Compute Service] API HTTP"
description: "[!DNL Asset Compute Service] API HTTP per creare applicazioni personalizzate."
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '2862'
ht-degree: 2%

---

# API HTTP [!DNL Asset Compute Service] {#asset-compute-http-api}

L’utilizzo dell’API è limitato a scopi di sviluppo. L’API viene fornita come contesto durante lo sviluppo di applicazioni personalizzate. [!DNL Adobe Experience Manager] as a [!DNL Cloud Service] utilizza l&#39;API per passare le informazioni di elaborazione a un&#39;applicazione personalizzata. Per ulteriori informazioni, vedere [Utilizzare i microservizi delle risorse e i profili di elaborazione](https://experienceleague.adobe.com/it/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] è disponibile solo per l&#39;utilizzo con [!DNL Experience Manager] come [!DNL Cloud Service].

Qualsiasi client dell&#39;API HTTP [!DNL Asset Compute Service] deve seguire questo flusso di alto livello:

1. È stato eseguito il provisioning di un client come progetto [!DNL Adobe Developer Console] in un&#39;organizzazione IMS. Ogni client separato (sistema o ambiente) richiede un proprio progetto separato per separare il flusso di dati dell’evento.

1. Un client genera un token di accesso per l&#39;account tecnico utilizzando l&#39;autenticazione [JWT (Service Account)](https://developer.adobe.com/developer-console/docs/guides/).

1. Un client chiama [`/register`](#register) una sola volta per recuperare l&#39;URL del diario.

1. Un client chiama [`/process`](#process-request) per ogni risorsa per la quale desidera generare rappresentazioni. La chiamata è asincrona.

1. Un client esegue regolarmente il polling del diario per [ricevere eventi](#asynchronous-events). Riceve eventi per ogni rendering richiesto quando il rendering viene elaborato correttamente (`rendition_created` tipo di evento) o se si verifica un errore (`rendition_failed` tipo di evento).

Il modulo [adobe-asset-compute-client](https://github.com/adobe/asset-compute-client) semplifica l&#39;utilizzo dell&#39;API nel codice Node.js.

## Autenticazione e autorizzazione {#authentication-and-authorization}

Tutte le API richiedono l’autenticazione tramite token di accesso. Le richieste devono impostare le seguenti intestazioni:

1. Intestazione `Authorization` con token Bearer, che è il token dell&#39;account tecnico, ricevuto tramite [JWT Exchange](https://developer.adobe.com/developer-console/docs/guides/) dal progetto Adobe Developer Console. Gli [ambiti](#scopes) sono documentati di seguito.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. Intestazione `x-gw-ims-org-id` con ID organizzazione IMS.

1. `x-api-key` con l&#39;ID client del progetto [!DNL Adobe Developers Console].

### Ambiti {#scopes}

Verifica che il token di accesso sia conforme ai seguenti ambiti:

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

Questi ambiti richiedono la sottoscrizione del progetto [!DNL Adobe Developer Console] ai servizi `Asset Compute`, `I/O Events` e `I/O Management API`. La ripartizione dei singoli ambiti è la seguente:

* Base
   * ambiti: `openid,AdobeID`

* Asset compute
   * metascope: `asset_compute_meta`
   * ambiti: `asset_compute,read_organizations`

* Adobe [!DNL `I/O Events`]
   * metascope: `event_receiver_api`
   * ambiti: `event_receiver,event_receiver_api`

* Adobe [!DNL `I/O Management API`]
   * metascope: `ent_adobeio_sdk`
   * ambiti: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registrazione {#register}

Ogni client di [!DNL Asset Compute service] - un progetto [!DNL Adobe Developer Console] univoco sottoscritto al servizio - deve [registrarsi](#register-request) prima di elaborare le richieste. Il passaggio di registrazione restituisce il giornale di registrazione eventi univoco necessario per recuperare gli eventi asincroni dall’elaborazione della rappresentazione.

Al termine del ciclo di vita, un client può [annullare la registrazione](#unregister-request).

### Registra richiesta {#register-request}

Questa chiamata API configura un client [!DNL Asset Compute] e fornisce l&#39;URL del giornale di registrazione eventi. Questo processo è un’operazione idempotente e deve essere chiamato solo una volta per ogni client. Può essere richiamato nuovamente per recuperare l’URL del diario.

| Parametro | Valore |
|--------------------------|------------------------------------------------------|
| Metodo | `POST` |
| Percorso | `/register` |
| Intestazione `Authorization` | Tutte le [intestazioni relative all&#39;autorizzazione](#authentication-and-authorization). |
| Intestazione `x-request-id` | Facoltativo, impostato dai client per un identificatore end-to-end univoco delle richieste di elaborazione tra i sistemi. |
| Corpo della richiesta | Deve essere vuoto. |

### Registra risposta {#register-response}

| Parametro | Valore |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Intestazione `X-Request-Id` | Uguale all&#39;intestazione della richiesta `X-Request-Id` o generata in modo univoco. Da utilizzare per identificare le richieste tra sistemi, o richieste di supporto, o entrambe. |
| Corpo della risposta | Un oggetto JSON con `journal`, `ok` o `requestId` campi. |

I codici di stato HTTP sono:

* **200 riuscito**: quando la richiesta è riuscita. L&#39;URL `journal` riceve notifiche sui risultati dell&#39;elaborazione asincrona avviata tramite `/process`. Avvisa `rendition_created` eventi al completamento o `rendition_failed` eventi se il processo non riesce.

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401 Non autorizzato**: si verifica quando la richiesta non dispone di [autenticazione](#authentication-and-authorization) valida. Un esempio potrebbe essere un token di accesso non valido o una chiave API non valida.

* **403 Non consentito**: si verifica quando la richiesta non dispone di [autorizzazione](#authentication-and-authorization) valida. Un esempio potrebbe essere un token di accesso valido, ma il progetto Adobe Developer Console (account tecnico) non è abbonato a tutti i servizi richiesti.

* **429 Troppe richieste**: si verifica quando questo client o in altro modo sovraccarica il sistema. I client devono riprovare con un backoff [esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff). Il corpo è vuoto.
* **4xx errore**: errore del client e registrazione non riuscita. Di solito viene restituita una risposta JSON come questa, anche se ciò non è garantito per tutti gli errori:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx error**: si verifica quando si è verificato un altro errore sul lato server e la registrazione non è riuscita. Di solito viene restituita una risposta JSON come questa, anche se ciò non è garantito per tutti gli errori:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

### Annulla registrazione richiesta {#unregister-request}

Questa chiamata API annulla la registrazione di un client [!DNL Asset Compute]. Dopo l&#39;annullamento della registrazione non sarà più possibile chiamare `/process`. L&#39;utilizzo della chiamata API per un client non registrato o per un client non ancora registrato restituisce un errore `404`.

| Parametro | Valore |
|--------------------------|------------------------------------------------------|
| Metodo | `POST` |
| Percorso | `/unregister` |
| Intestazione `Authorization` | Tutte le [intestazioni relative all&#39;autorizzazione](#authentication-and-authorization). |
| Intestazione `x-request-id` | Facoltativo. I client possono impostarlo per un identificatore end-to-end univoco delle richieste di elaborazione tra i sistemi. |
| Corpo della richiesta | Vuoto. |

### Annulla registrazione risposta {#unregister-response}

| Parametro | Valore |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Intestazione `X-Request-Id` | Uguale all&#39;intestazione della richiesta `X-Request-Id` o generata in modo univoco. Utilizza per identificare le richieste tra i sistemi o le richieste di supporto. |
| Corpo della risposta | Un oggetto JSON con `ok` e `requestId` campi. |

I codici di stato sono:

* **200 operazione riuscita**: si verifica quando la registrazione e il giornale di registrazione vengono trovati e rimossi.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401 Non autorizzato**: si verifica quando la richiesta non dispone di [autenticazione](#authentication-and-authorization) valida. Un esempio potrebbe essere un token di accesso non valido o una chiave API non valida.

* **403 Non consentito**: si verifica quando la richiesta non dispone di [autorizzazione](#authentication-and-authorization) valida. Un esempio potrebbe essere un token di accesso valido, ma il progetto Adobe Developer Console (account tecnico) non è abbonato a tutti i servizi richiesti.

* **404 Non trovato**: questo stato viene visualizzato quando le credenziali fornite vengono annullate o non sono valide.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **429 Troppe richieste**: si verifica quando il sistema è sovraccarico. I client devono riprovare con un backoff [esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff). Il corpo è vuoto.

* **4xx errore**: si verifica quando si è verificato un altro errore del client e l&#39;annullamento della registrazione non è riuscito. Di solito viene restituita una risposta JSON come questa, anche se ciò non è garantito per tutti gli errori:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx error**: si verifica quando si è verificato un altro errore sul lato server e la registrazione non è riuscita. Di solito viene restituita una risposta JSON come questa, anche se ciò non è garantito per tutti gli errori:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

## Richiesta di elaborazione {#process-request}

L&#39;operazione `process` invia un processo che trasforma una risorsa di origine in più rappresentazioni, in base alle istruzioni contenute nella richiesta. Le notifiche di completamento (tipo evento `rendition_created`) o di errori (tipo evento `rendition_failed`) vengono inviate a un giornale di registrazione eventi che deve essere recuperato una volta con [`/register`](#register) prima di effettuare un numero qualsiasi di richieste `/process`. Le richieste formate in modo errato hanno immediatamente esito negativo con un codice di errore 400.

I riferimenti binari vengono eseguiti utilizzando URL, ad esempio URL prefirmati di Amazon AWS S3 o URL SAS di archiviazione Azure Blob. Utilizzato sia per leggere la risorsa `source` (`GET` URL) che per scrivere le rappresentazioni (`PUT` URL). Il client è responsabile della generazione di questi URL prefirmati.

| Parametro | Valore |
|--------------------------|------------------------------------------------------|
| Metodo | `POST` |
| Percorso | `/process` |
| Tipo MIME | `application/json` |
| Intestazione `Authorization` | Tutte le [intestazioni relative all&#39;autorizzazione](#authentication-and-authorization). |
| Intestazione `x-request-id` | Facoltativo. I client possono impostare un identificatore end-to-end univoco per tenere traccia delle richieste di elaborazione tra i sistemi. |
| Corpo della richiesta | Deve essere nel formato JSON della richiesta del processo come descritto di seguito. Fornisce istruzioni su quale risorsa elaborare e quali rappresentazioni generare. |

### Elabora JSON richiesta {#process-request-json}

Il corpo della richiesta di `/process` è un oggetto JSON con questo schema di alto livello:

```json
{
    "source": "",
    "renditions" : []
}
```

I campi disponibili sono:

| Nome | Tipo | Descrizione | Esempio |
|--------------|----------|-------------|---------|
| `source` | `string` | URL della risorsa di origine elaborata. Facoltativo, in base al formato di rendering richiesto (ad esempio, `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | Descrizione della risorsa di origine elaborata. Vedi la descrizione di [campi oggetto Source](#source-object-fields) di seguito. Facoltativo in base al formato di rendering richiesto (ad esempio, `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Rappresentazioni da generare dal file di origine. Ogni oggetto di rendering supporta una [istruzione di rendering](#rendition-instructions). Obbligatorio. | `[{ "target": "https://....", "fmt": "png" }]` |

`source` può essere un `<string>` visualizzato come URL oppure un `<object>` con un campo aggiuntivo. Le seguenti varianti sono simili:

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### Campi oggetto Source {#source-object-fields}

| Nome | Tipo | Descrizione | Esempio |
|-----------|----------|-------------|---------|
| `url` | `string` | URL della risorsa di origine da elaborare. Obbligatorio. | `"http://example.com/image.jpg"` |
| `name` | `string` | Nome del file di risorse Source. Se non viene rilevato alcun tipo MIME, è possibile utilizzare un&#39;estensione di file nel nome. Ha priorità rispetto al nome file specificato nel percorso URL. Inoltre, ha priorità rispetto al nome del file nell&#39;intestazione `content-disposition` della risorsa binaria. Impostazione predefinita: &quot;file&quot;. | `"image.jpg"` |
| `size` | `number` | Dimensione file risorse Source in byte. Ha la precedenza sull&#39;intestazione `content-length` della risorsa binaria. | `10234` |
| `mimetype` | `string` | Tipo MIME del file di risorse Source. Ha la precedenza sull&#39;intestazione `content-type` della risorsa binaria. | `"image/jpeg"` |

### Un esempio completo di richiesta `process` {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## Risposta processo {#process-response}

La richiesta `/process` restituisce immediatamente un risultato positivo o negativo in base alla convalida della richiesta di base. L’elaborazione effettiva delle risorse avviene in modo asincrono.

| Parametro | Valore |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Intestazione `X-Request-Id` | Uguale all&#39;intestazione della richiesta `X-Request-Id` o generata in modo univoco. Utilizza per identificare le richieste tra i sistemi o le richieste di supporto. |
| Corpo della risposta | Un oggetto JSON con `ok` e `requestId` campi. |

Codici di stato:

* **200 completato**: se la richiesta è stata inviata correttamente. JSON di risposta include `"ok": true`:

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400 Richiesta non valida**: se la richiesta non è strutturata correttamente, ad esempio se nel payload JSON mancano i campi obbligatori. JSON di risposta include `"ok": false`:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401 Non autorizzato**: quando la richiesta non dispone di [autenticazione](#authentication-and-authorization) valida. Un esempio potrebbe essere un token di accesso non valido o una chiave API non valida.
* **403 Non consentito**: se la richiesta non dispone di [autorizzazione](#authentication-and-authorization) valida. Un esempio potrebbe essere un token di accesso valido, ma il progetto Adobe Developer Console (account tecnico) non è abbonato a tutti i servizi richiesti.
* **429 Troppe richieste**: si verifica quando il sistema è sovraccarico, a causa di questo particolare client o della domanda complessiva. I client possono riprovare con un backoff [esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff). Il corpo è vuoto.
* **4xx errore**: quando si è verificato un altro errore del client. Di solito viene restituita una risposta JSON come questa, anche se ciò non è garantito per tutti gli errori:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx errore**: quando si è verificato un altro errore lato server. Di solito viene restituita una risposta JSON come questa, anche se ciò non è garantito per tutti gli errori:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

La maggior parte dei client è probabilmente incline a ritentare la stessa richiesta con [backoff esponenziale](https://en.wikipedia.org/wiki/Exponential_backoff) per qualsiasi errore *eccetto* problemi di configurazione come 401 o 403 o richieste non valide come 400. A parte la regolare limitazione delle tariffe tramite risposte 429, un’interruzione temporanea del servizio o una limitazione potrebbero causare errori 5xx. Sarebbe quindi consigliabile riprovare dopo un certo periodo di tempo.

Tutte le risposte JSON (se presenti) includono `requestId`, che è lo stesso valore dell&#39;intestazione `X-Request-Id`. L’Adobe consiglia di leggere dall’intestazione perché è sempre presente. `requestId` viene restituito anche in tutti gli eventi relativi all&#39;elaborazione delle richieste come `requestId`. I client non devono fare alcuna supposizione sul formato di questa stringa. È un identificatore di stringa opaco.

## Consenso alla post-elaborazione {#opt-in-to-post-processing}

[Asset Compute SDK](https://github.com/adobe/asset-compute-sdk) supporta un set di opzioni di post-elaborazione di base per le immagini. I processi di lavoro personalizzati possono acconsentire in modo esplicito alla post-elaborazione impostando il campo `postProcess` sull&#39;oggetto rendering su `true`.

I casi d’uso supportati sono:

* Ritaglio è una rappresentazione di un rettangolo i cui limiti sono definiti da crop.w, crop.h, crop.x e crop.y. I dettagli di ritaglio sono specificati nel campo `instructions.crop` dell&#39;oggetto di rendering.
* Ridimensionare le immagini utilizzando la larghezza, l&#39;altezza o entrambe. `instructions.width` e `instructions.height` lo definiscono nell&#39;oggetto di rendering. Per ridimensionare utilizzando solo la larghezza o l&#39;altezza, impostate un solo valore. Il servizio di elaborazione conserva le proporzioni.
* Imposta la qualità per un&#39;immagine JPEG. `instructions.quality` lo definisce nell&#39;oggetto di rendering. Un livello di qualità pari a 100 rappresenta la qualità più elevata, mentre un numero inferiore indica una diminuzione della qualità.
* Creazione di immagini interlacciate. `instructions.interlace` lo definisce nell&#39;oggetto di rendering.
* Impostate DPI per regolare le dimensioni di rendering per la pubblicazione desktop regolando la scala applicata ai pixel. `instructions.dpi` lo definisce nell&#39;oggetto di rendering per modificare la risoluzione dpi. Tuttavia, per ridimensionare l&#39;immagine in modo che abbia le stesse dimensioni a una risoluzione diversa, utilizzare le istruzioni `convertToDpi`.
* Ridimensiona l’immagine in modo che la larghezza o l’altezza di cui è stato eseguito il rendering rimanga invariata rispetto all’originale alla risoluzione di destinazione specificata (DPI). `instructions.convertToDpi` lo definisce nell&#39;oggetto di rendering.

## Risorse filigrana {#add-watermark}

[Asset Compute SDK](https://github.com/adobe/asset-compute-sdk) supporta l&#39;aggiunta di una filigrana ai file immagine PNG, JPEG, TIFF e GIF. La filigrana viene aggiunta seguendo le istruzioni per il rendering nell&#39;oggetto `watermark` del rendering.

La filigrana viene eseguita durante la post-elaborazione della rappresentazione. Per applicare una filigrana alle risorse, il processo di lavoro personalizzato [opta per la post-elaborazione](#opt-in-to-post-processing) impostando il campo `postProcess` dell&#39;oggetto di rendering su `true`. Se il lavoratore non acconsente, la filigrana non viene applicata, anche se l’oggetto filigrana è impostato sull’oggetto di rendering nella richiesta.

## Istruzioni per la rappresentazione {#rendition-instructions}

Di seguito sono riportate le opzioni disponibili per l&#39;array `renditions` in [`/process`](#process-request).

### Campi comuni {#common-fields}

| Nome | Tipo | Descrizione | Esempio |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | Il formato di destinazione delle rappresentazioni può anche essere `text` per l&#39;estrazione del testo e `xmp` per l&#39;estrazione dei metadati XMP come XML. Visualizza [formati supportati](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support) | `png` |
| `worker` | `string` | URL di [applicazione personalizzata](develop-custom-application.md). Deve essere un URL `https://`. Se questo campo è presente, la copia trasformata viene creata da un&#39;applicazione personalizzata. Qualsiasi altro campo di rendering impostato viene quindi utilizzato nell’applicazione personalizzata. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | L’URL in cui deve essere caricata la rappresentazione generata utilizzando HTTP PUT. | `http://w.com/img.jpg` |
| `target` | `object` | Informazioni sul caricamento di URL prefirmati in più parti per la rappresentazione generata. Queste informazioni sono per [AEM / Caricamento binario diretto Oak](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) con questo [comportamento di caricamento multipart](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Campi:<ul><li>`urls`: array di stringhe, uno per ogni URL parte prefirmato</li><li>`minPartSize`: dimensione minima da utilizzare per una parte = url</li><li>`maxPartSize`: dimensione massima da utilizzare per una parte = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Facoltativo. Il client controlla lo spazio riservato e lo trasmette così come avviene per gli eventi di rendering. Consente a un client di aggiungere informazioni personalizzate per identificare gli eventi di rendering. Non deve essere modificata o utilizzata nelle applicazioni personalizzate, in quanto i client sono liberi di modificarla in qualsiasi momento. | `{ ... }` |

### Campi specifici della rappresentazione {#rendition-specific-fields}

Per un elenco dei formati di file attualmente supportati, vedere [formati di file supportati](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support).

| Nome | Tipo | Descrizione | Esempio |
|-------------------|----------|-------------|---------|
| `*` | `*` | È possibile aggiungere campi personalizzati avanzati che un&#39;applicazione [personalizzata](develop-custom-application.md) comprende. | |
| `embedBinaryLimit` | `number` in byte | Quando la dimensione del file della rappresentazione è inferiore al valore specificato, viene inclusa nell’evento inviato al termine della creazione. La dimensione massima consentita per l’incorporamento è 32 KB (32 x 1024 byte). Se le dimensioni di una copia trasformata superano il limite di `embedBinaryLimit`, la copia viene inserita in una posizione nell&#39;archiviazione cloud e non viene incorporata nell&#39;evento. | `3276` |
| `width` | `number` | Larghezza in pixel. solo per le rappresentazioni di immagini. | `200` |
| `height` | `number` | Altezza in pixel. solo per le rappresentazioni di immagini. | `200` |
|                   |          | Le proporzioni vengono sempre mantenute se: <ul> <li> Sono specificati sia `width` che `height`, quindi l&#39;immagine si adatta alle dimensioni mantenendo le proporzioni </li><li> Se si specifica solo `width` o `height`, l&#39;immagine risultante utilizza la dimensione corrispondente mantenendo le proporzioni</li><li> Se `width` o `height` non è specificato, viene utilizzata la dimensione in pixel dell&#39;immagine originale. Dipende dal tipo di origine. Per alcuni formati, ad esempio i file PDF, viene utilizzata una dimensione predefinita. Può essere presente un limite di dimensioni massimo.</li></ul> | |
| `quality` | `number` | Specificare la qualità jpeg nell&#39;intervallo compreso tra `1` e `100`. Applicabile solo alle rappresentazioni di immagini. | `90` |
| `xmp` | `string` | Utilizzato solo dal writeback dei metadati dell&#39;XMP, l&#39;XMP con codifica base64 viene riscritto nella rappresentazione specificata. | |
| `interlace` | `bool` | Creare PNG o GIF interlacciato o progressive JPEG impostandolo su `true`. Non ha alcun effetto su altri formati di file. | |
| `jpegSize` | `number` | Dimensione approssimativa del file JPEG in byte. Sostituisce qualsiasi impostazione `quality`. Non ha alcun effetto su altri formati. | |
| `dpi` | `number` oppure `object` | Impostare x e y DPI. Per semplicità, può anche essere impostato su un singolo numero, che viene utilizzato sia per x che per y. Non ha alcun effetto sull’immagine stessa. | `96` oppure `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` oppure `object` | x e y DPI ricampiona i valori mantenendo le dimensioni fisiche. Per semplicità, può anche essere impostato su un singolo numero, che viene utilizzato sia per x che per y. | `96` oppure `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Elenco di file da includere nell&#39;archivio ZIP (`fmt=zip`). Ogni voce può essere una stringa URL o un oggetto con i campi:<ul><li>`url`: URL per scaricare il file</li><li>`path`: archivia il file nel percorso specificato nel file ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Gestione duplicata per archivi ZIP (`fmt=zip`). Per impostazione predefinita, più file memorizzati nello stesso percorso nel file ZIP generano un errore. Se si imposta `duplicate` su `ignore`, viene archiviata solo la prima risorsa e il resto viene ignorato. | `ignore` |
| `watermark` | `object` | Contiene istruzioni sulla [filigrana](#watermark-specific-fields). |  |

### Campi specifici della filigrana {#watermark-specific-fields}

Il formato PNG viene utilizzato come filigrana.

| Nome | Tipo | Descrizione | Esempio |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Scala della filigrana, tra `0.0` e `1.0`. `1.0` significa che la filigrana ha la scala originale (1:1) e i valori più bassi ne riducono la dimensione. | Un valore di `0.5` indica metà delle dimensioni originali. |
| `image` | `url` | URL del file PNG da utilizzare per la filigrana. | |

## Eventi asincroni {#asynchronous-events}

Al termine dell&#39;elaborazione di una copia trasformata o quando si verifica un errore, un evento viene inviato a un Adobe [!DNL `I/O Events Journal`]. I client devono ascoltare l&#39;URL del diario fornito tramite [`/register`](#register). La risposta del journal include un array `event` costituito da un oggetto per ogni evento, di cui il campo `event` include il payload dell&#39;evento effettivo.

Il tipo di Adobe [!DNL `I/O Events`] per tutti gli eventi di [!DNL Asset Compute Service] è `asset_compute`. Il giornale di registrazione è sottoscritto automaticamente solo a questo tipo di evento e non è necessario filtrare ulteriormente in base al tipo di evento [!DNL Adobe Developer]. I tipi di evento specifici del servizio sono disponibili nella proprietà `type` dell&#39;evento.

### Tipi di evento {#event-types}

| Evento | Descrizione |
|---------------------|-------------|
| `rendition_created` | Inviato per ogni rappresentazione elaborata e caricata correttamente. |
| `rendition_failed` | Inviato per ogni rappresentazione che non è stata elaborata o caricata. |

### Attributi evento {#event-attributes}

| Attributo | Tipo | Evento | Descrizione |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Timestamp in cui l’evento è stato inviato nel formato esteso semplificato [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601), definito da JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | ID della richiesta originale a `/process`, come intestazione `X-Request-Id`. |
| `source` | `object` | `*` | `source` della richiesta `/process`. |
| `userData` | `object` | `*` | `userData` della rappresentazione dalla richiesta `/process` se impostata. |
| `rendition` | `object` | `rendition_*` | Oggetto di rendering corrispondente passato in `/process`. |
| `metadata` | `object` | `rendition_created` | Le proprietà [metadata](#metadata) del rendering. |
| `errorReason` | `string` | `rendition_failed` | Errore di rappresentazione [motivo](#error-reasons), se presente. |
| `errorMessage` | `string` | `rendition_failed` | Testo che fornisce maggiori dettagli sull’eventuale errore della rappresentazione. |

### Metadati {#metadata}

| Proprietà | Descrizione |
|--------|-------------|
| `repo:size` | Dimensione della rappresentazione in byte. |
| `repo:sha1` | Il digest sha1 della rappresentazione. |
| `dc:format` | Tipo MIME della rappresentazione. |
| `repo:encoding` | La codifica charset della rappresentazione nel caso sia un formato basato su testo. |
| `tiff:ImageWidth` | Larghezza della rappresentazione in pixel. Presente solo per le rappresentazioni di immagini. |
| `tiff:ImageLength` | Lunghezza della rappresentazione in pixel. Presente solo per le rappresentazioni di immagini. |

### Motivi di errore {#error-reasons}

| Motivo | Descrizione |
|---------|-------------|
| `RenditionFormatUnsupported` | Il formato di rappresentazione richiesto non è supportato per l’origine specificata. |
| `SourceUnsupported` | L’origine specifica non è supportata anche se il tipo è supportato. |
| `SourceCorrupt` | Dati di origine danneggiati. Include file vuoti. |
| `RenditionTooLarge` | Impossibile caricare la rappresentazione utilizzando gli URL prefirmati forniti in `target`. Le dimensioni effettive del rendering sono disponibili come metadati in `repo:size` e vengono utilizzate dal client per rielaborare il rendering con il numero corretto di URL prefirmati. |
| `GenericError` | Qualsiasi altro errore imprevisto. |
