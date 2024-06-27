---
title: Sviluppo per [!DNL Asset Compute Service]
description: Creare applicazioni personalizzate utilizzando [!DNL Asset Compute Service].
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '1507'
ht-degree: 0%

---

# Sviluppare un’applicazione personalizzata {#develop}

Prima di iniziare a sviluppare un’applicazione personalizzata:

* Assicurati che tutte le [prerequisiti](/help/using/understand-extensibility.md#prerequisites-and-provisioning) sono soddisfatte.
* Installare [strumenti software richiesti](/help/using/setup-environment.md#create-dev-environment).
* Consulta [configurare l’ambiente](setup-environment.md) per essere certi di essere pronti per creare un&#39;applicazione personalizzata.

## Creare un’applicazione personalizzata {#create-custom-application}

Assicurati di avere [Adobe aio-cli](https://github.com/adobe/aio-cli) installato localmente.

1. Per creare un&#39;applicazione personalizzata: [creare un progetto App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli). Per farlo, esegui `aio app init <app-name>` nel tuo terminale.

   Se non hai già effettuato l’accesso, questo comando richiede a un browser di accedere al [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis) con il tuo Adobe ID. Consulta [qui](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli) per ulteriori informazioni sull’accesso da cli.

   L&#39;Adobe consiglia di effettuare prima l&#39;accesso. In caso di problemi, segui le istruzioni [per creare un&#39;app senza effettuare l&#39;accesso](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. Dopo aver effettuato l’accesso, segui le istruzioni riportate nella CLI e seleziona la `Organization`, `Project`, e `Workspace` da utilizzare per l’applicazione. Scegli il progetto e l’area di lavoro creati al momento della [configurare l’ambiente](setup-environment.md). Quando richiesto `Which extension point(s) do you wish to implement ?`, assicurati di selezionare `DX Asset Compute Worker`:

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyAdobe Developer App BuilderProject
   ? Which extension point(s) do you wish to implement ? (Press <space> to select, <a>
   to toggle all, <i> to invert selection)
   ❯◯ DX Experience Cloud SPA
   ◯ DX Asset Compute Worker
   ```

1. Quando richiesto con `Which Adobe I/O App features do you want to enable for this project?`, seleziona `Actions`. Assicurati di deselezionare `Web Assets` come risorse web utilizza diversi controlli di autenticazione e autorizzazione.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. Quando richiesto `Which type of sample actions do you want to create?`, assicurati di selezionare `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Seguire le altre istruzioni visualizzate e aprire la nuova applicazione in Visual Studio Code o nell&#39;editor di codice preferito. Contiene lo scaffolding e il codice di esempio per un’applicazione personalizzata.

   Leggi qui le informazioni sulla [componenti principali di un’app App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-an-app-builder-application).

   L’applicazione modello sfrutta le funzionalità di Adobe [SDK ASSET COMPUTE](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) per il caricamento, il download e l’orchestrazione delle rappresentazioni delle applicazioni, in modo che gli sviluppatori debbano implementare solo la logica dell’applicazione personalizzata. All&#39;interno del `actions/<worker-name>` cartella, la `index.js` file è il punto in cui aggiungere il codice personalizzato dell&#39;applicazione.

Consulta [esempi di applicazioni personalizzate](#try-sample) esempi e idee per applicazioni personalizzate.

### Aggiungi credenziali {#add-credentials}

Quando effettui l’accesso durante la creazione dell’applicazione, la maggior parte delle credenziali di App Builder viene raccolta nel file ENV. Tuttavia, l’utilizzo dello strumento per sviluppatori richiede credenziali aggiuntive.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Credenziali di archiviazione dello strumento per sviluppatori {#developer-tool-credentials}

Lo strumento che consente agli sviluppatori di valutare le app personalizzate utilizzando [!DNL Asset Compute service] richiede l’utilizzo di un contenitore di archiviazione cloud. Questo contenitore è essenziale per memorizzare i file di test e per la ricezione e la presentazione delle rappresentazioni prodotte dalle app.

>[!NOTE]
>
>Questo contenitore è separato dall’archiviazione cloud di [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]. Si applica solo per lo sviluppo e il testing con lo strumento per sviluppatori Asset Compute.

Assicurati di avere accesso a [contenitore di archiviazione cloud supportato](https://github.com/adobe/asset-compute-devtool#prerequisites). Questo contenitore viene utilizzato collettivamente da vari sviluppatori per progetti diversi, quando necessario.

#### Aggiungi credenziali al file ENV {#add-credentials-env-file}

Inserisci le credenziali successive per lo strumento di sviluppo in `.env` file. Il file si trova nella directory principale del progetto App Builder:

1. Aggiungi il percorso assoluto al file della chiave privata creato durante l’aggiunta di servizi al progetto App Builder:

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Scarica il file da Adobe Developer Console. Vai alla directory principale del progetto e fai clic su &quot;Scarica tutto&quot; nell’angolo in alto a destra. Il file viene scaricato con `<namespace>-<workspace>.json` come nome del file. Effettua una delle operazioni seguenti:

   * Rinomina il file come `console.json` e spostarlo nella directory principale del progetto.
   * Facoltativamente, puoi aggiungere il percorso assoluto al file JSON dell’integrazione di Adobe Developer Console. Questo file è lo stesso [`console.json`](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user) file scaricato nell’area di lavoro del progetto.

     ```conf
     ASSET_COMPUTE_INTEGRATION_FILE_PATH=
     ```

1. Aggiungi le credenziali di archiviazione S3 o Azure. È sufficiente accedere a una sola soluzione di archiviazione cloud.

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

>[!TIP]
>
>Il `config.json` il file contiene le credenziali. Dall’interno del progetto, aggiungi il file JSON al tuo `.gitignore` per impedire la condivisione. Lo stesso vale per `.env` e `.aio` file.

## Eseguire l’applicazione {#run-custom-application}

Prima di eseguire l’applicazione con lo strumento per sviluppatori Asset Compute, configura correttamente [credenziali](#developer-tool-credentials).

Per eseguire l’applicazione nello strumento per sviluppatori, utilizza `aio app run` comando. Distribuisce l’azione in Adobe [!DNL I/O Runtime]e avvia lo strumento di sviluppo sul computer locale. Questo strumento viene utilizzato per testare le richieste dell’applicazione durante lo sviluppo. Di seguito è riportato un esempio di richiesta di rappresentazione:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>Non utilizzare il `--local` contrassegno con `run` comando. Non funziona con [!DNL Asset Compute] applicazioni personalizzate e lo strumento per sviluppatori Asset Compute. Le applicazioni personalizzate vengono richiamate da [!DNL Asset Compute] servizio che non può accedere alle azioni in esecuzione nei computer locali dello sviluppatore.

Consulta [qui](test-custom-application.md) come testare ed eseguire il debug dell’applicazione. Al termine dello sviluppo dell&#39;applicazione personalizzata, [distribuire l’applicazione personalizzata](deploy-custom-application.md).

## Prova l’applicazione di esempio fornita da Adobe {#try-sample}

Di seguito sono riportati alcuni esempi di applicazioni personalizzate:

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [operaio-animale-immagini](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Applicazione personalizzata modello {#template-custom-application}

Il [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) è un&#39;applicazione modello. Genera una copia trasformata semplicemente copiando il file di origine. Il contenuto di questa applicazione è il modello ricevuto al momento della scelta `Adobe Asset Compute` nella creazione dell&#39;app aio.

Il fascicolo dell&#39;applicazione, [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) utilizza [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) per scaricare il file di origine, orchestra ogni elaborazione della rappresentazione e carica di nuovo le rappresentazioni risultanti nell’archiviazione cloud.

Il [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) definito all’interno del codice dell’applicazione, è il punto in cui eseguire tutta la logica di elaborazione dell’applicazione. Il callback della rappresentazione in `worker-basic` copia semplicemente il contenuto del file di origine nel file della copia trasformata.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## Chiamare un’API esterna {#call-external-api}

Nel codice dell’applicazione, puoi effettuare chiamate API esterne per facilitare l’elaborazione dell’applicazione. Di seguito è riportato un esempio di file di applicazione che richiama un’API esterna.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

Ad esempio, il [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) effettua una richiesta di recupero di un URL statico da Wikimedia utilizzando [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) libreria.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### Trasmettere i parametri personalizzati {#pass-custom-parameters}

Potete trasmettere parametri definiti personalizzati attraverso gli oggetti di rendering. È possibile farvi riferimento all’interno dell’applicazione in [`rendition` istruzioni](https://github.com/adobe/asset-compute-sdk#rendition). Un esempio di oggetto di rendering è:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

Di seguito è riportato un esempio di file di applicazione che accede a un parametro personalizzato:

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

Il `example-worker-animal-pictures` trasmette un parametro personalizzato [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) per determinare quale file recuperare da Wikimedia.

## Supporto di autenticazione e autorizzazione {#authentication-authorization-support}

Per impostazione predefinita, le applicazioni personalizzate di Asset compute sono fornite con i controlli di autorizzazione e autenticazione per il progetto App Builder. Attivato impostando `require-adobe-auth` annotazione a `true` nel `manifest.yml`.

### Accedere ad altre API Adobe {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

Aggiungi i servizi API alla [!DNL Asset Compute] Area di lavoro della console creata durante la configurazione. Questi servizi fanno parte del token di accesso JWT generato da [!DNL Asset Compute Service]. Il token e le altre credenziali sono accessibili all’interno dell’azione dell’applicazione `params` oggetto.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Trasmettere le credenziali per i sistemi di terze parti {#pass-credentials-for-tp}

Per gestire le credenziali per altri servizi esterni, trasmettile come parametri predefiniti sulle azioni. Vengono crittografati automaticamente in transito. Per ulteriori informazioni, consulta [creazione di azioni nella guida per gli sviluppatori di Adobe I/O Runtime](https://developer.adobe.com/runtime/docs/guides/using/creating_actions/). Quindi impostale utilizzando le variabili di ambiente durante la distribuzione. È possibile accedere a questi parametri in `params` all&#39;interno dell&#39;azione.

Imposta i parametri di default all&#39;interno del `inputs` nel `manifest.yml`:

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

Il `$VAR` espressione legge il valore da una variabile di ambiente denominata `VAR`.

Durante lo sviluppo, puoi assegnare il valore nella `.env` file. Il motivo è perché `aio` importa automaticamente le variabili di ambiente da `.env` insieme alle variabili impostate dalla shell di origine. In questo esempio, la proprietà `.env` il file ha un aspetto simile al seguente:

```CONF
#...
SECRET_KEY=secret-value
```

Per la distribuzione di produzione è possibile impostare le variabili di ambiente nel sistema CI, ad esempio utilizzando i segreti nelle azioni GitHub. Infine, accedi ai parametri predefiniti all’interno dell’applicazione in quanto tale:

```javascript
const key = params.secretKey;
```

## Ridimensionamento delle applicazioni {#sizing-workers}

Un’applicazione viene eseguita in un contenitore in Adobe [!DNL I/O Runtime] con [limiti](https://developer.adobe.com/runtime/docs/guides/using/system_settings/) che possono essere configurati tramite `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

A causa dell’elaborazione intensiva eseguita dalle applicazioni Asset Compute, è necessario regolare questi limiti per ottenere prestazioni ottimali (abbastanza grandi da gestire le risorse binarie) ed efficienza (senza sprecare risorse a causa della memoria contenitore non utilizzata).

Il timeout predefinito per le azioni in fase di esecuzione è di un minuto, ma può essere aumentato impostando `timeout` limite (in millisecondi). Se prevedi di elaborare file di dimensioni maggiori, aumenta questo tempo. Considera il tempo totale necessario per scaricare l’origine, elaborare il file e caricare la rappresentazione. Se un’azione si interrompe per timeout, ovvero non restituisce l’attivazione prima del limite di timeout specificato, Runtime elimina il contenitore e non lo riutilizza.

Le applicazioni di Asset compute per natura tendono ad essere collegate alla rete e al disco. Il file di origine deve essere scaricato per primo. L’elaborazione richiede spesso molte risorse, quindi le rappresentazioni risultanti vengono caricate di nuovo.

È possibile specificare la memoria allocata a un contenitore di azioni in megabyte utilizzando `memorySize` parametro. Attualmente, questo parametro definisce anche la quantità di accesso alla CPU che il contenitore ottiene e, soprattutto, è un elemento chiave del costo di utilizzo di Runtime (i contenitori più grandi costano di più). Utilizzare un valore maggiore quando l&#39;elaborazione richiede una quantità maggiore di memoria o CPU, ma prestare attenzione a non sprecare risorse, poiché più grandi sono i contenitori, minore è la velocità effettiva complessiva.

Inoltre, è possibile controllare la concorrenza delle azioni all’interno di un contenitore utilizzando `concurrency` impostazione. Questa impostazione corrisponde al numero di attivazioni simultanee ottenute da un singolo contenitore (della stessa azione). In questo modello, il contenitore delle azioni è simile a un server Node.js che riceve più richieste simultanee, fino a tale limite. Il valore predefinito `memorySize` in fase di esecuzione è impostato su 200 MB, ideale per azioni App Builder di dimensioni inferiori. Ad Asset compute, questo valore predefinito può essere eccessivo a causa della maggiore complessità dell&#39;elaborazione locale e dell&#39;utilizzo del disco. Alcune applicazioni, a seconda dell’implementazione, potrebbero non funzionare correttamente con le attività simultanee. L’SDK di Asset compute garantisce che le attivazioni siano separate scrivendo file in diverse cartelle univoche.

Eseguire test delle applicazioni per trovare i numeri ottimali per `concurrency` e `memorySize`. Contenitori più grandi = un limite di memoria più alto potrebbe consentire una maggiore concorrenza, ma potrebbe anche sprecare per un traffico più basso.
