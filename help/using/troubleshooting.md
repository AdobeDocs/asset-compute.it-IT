---
title: Risoluzione dei problemi [!DNL Asset Compute Service]
description: Risolvere i problemi ed eseguire il debug delle applicazioni personalizzate utilizzando  [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '273'
ht-degree: 0%

---

# Risoluzione dei problemi {#troubleshoot}

Di seguito sono riportati alcuni suggerimenti generici per la risoluzione dei problemi relativi al servizio Asset Compute:

* Verificare che l&#39;applicazione JavaScript non si arresti all&#39;avvio. Tali arresti anomali sono in genere correlati a una libreria mancante o a una dipendenza.
* Verificare che nel file `package.json` dell&#39;applicazione venga fatto riferimento a tutte le dipendenze da installare.
* Assicurati che eventuali errori derivanti dalla pulizia in caso di errore non generino errori che nascondono il problema originale.

* Quando si avvia lo strumento per sviluppatori per la prima volta con una nuova integrazione [!DNL Asset Compute Service], la prima richiesta di elaborazione potrebbe non riuscire se il journal degli eventi Asset Compute non è completamente configurato. Attendi che il giornale di registrazione venga configurato prima di inviare un’altra richiesta.
* Assicurarsi che tutte le API richieste (Asset compute, Adobe [!DNL I/O Events], Gestione eventi e Runtime) siano incluse nell&#39;Adobe [!DNL `I/O Project`] e in Workspace per evitare errori di richiesta `/register` o `/process`.

## Accedi ai problemi per Adobe [!DNL aio-cli] {#login-via-aio-cli}

In caso di problemi durante l&#39;accesso a [!DNL Adobe Developer Console] [tramite l&#39;Adobe [!DNL aio-cli]](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli), aggiungere manualmente le credenziali necessarie per lo sviluppo, il test e la distribuzione dell&#39;applicazione personalizzata:

1. Passa al progetto e all&#39;area di lavoro Adobe Developer App Builder in [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis) e premi **[!UICONTROL Scarica]** dall&#39;angolo in alto a destra. Apri questo file e salva JSON in un luogo sicuro sul computer.

1. Passa al file ENV nell’applicazione Adobe Developer App Builder.

1. Aggiungere le credenziali dell&#39;Adobe [!DNL I/O Runtime]. Ottieni le credenziali dell&#39;Adobe [!DNL I/O Runtime] dal JSON scaricato. Credenziali in `project.workspace.services.runtime`. Aggiungere le credenziali di runtime [!DNL Adobe I/O] nelle variabili `AIO_runtime_XXX`:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Aggiungi il percorso assoluto al JSON scaricato nel passaggio 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Configura il resto delle [credenziali richieste](develop-custom-application.md) necessarie per lo strumento per sviluppatori.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
