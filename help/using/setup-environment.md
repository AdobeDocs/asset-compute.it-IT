---
title: Imposta l'ambiente di sviluppo richiesto per  [!DNL Asset Compute Service]
description: Configurazione dell'ambiente di sviluppo per  [!DNL Asset Compute Service]  per iniziare a creare e testare il codice personalizzato.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: db38b9dc27505aa7e04cf58a646005fc2e0e8782
workflow-type: tm+mt
source-wordcount: '359'
ht-degree: 1%

---

# Configurare un ambiente per sviluppatori {#create-dev-environment}

Per creare una configurazione che consenta lo sviluppo per [!DNL Asset Compute Service], seguire i requisiti e le istruzioni seguenti.

1. [Acquisire l&#39;accesso e le credenziali](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials) per [!DNL Adobe Developer App Builder].

1. [Configurare l&#39;ambiente locale](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up) e gli strumenti richiesti.

1. Altri strumenti utili per iniziare a sviluppare in modo fluido sono:

   * [Git](https://git-scm.com/)
   * [Docker Desktop](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) (v14 LTS, versioni dispari non consigliate) e [NPM](https://www.npmjs.com). L&#39;utente di OS X HomeBrew può eseguire `brew install node` per installare entrambi. In caso contrario, scaricalo dalla [pagina di download di NodeJS](https://nodejs.org/it/)
   * Un IDE adatto a NodeJS, ad Adobe [Visual Studio Code (VS Code)](https://code.visualstudio.com), in quanto è l&#39;IDE supportato per il debugger. È possibile utilizzare qualsiasi altro IDE come editor di codice, ma l’utilizzo avanzato (ad esempio, debugger) non è ancora supportato
   * Installa l&#39;Adobe più recente [[!DNL aio-cli]](https://github.com/adobe/aio-cli) (`aio`)
   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. Assicurati di soddisfare i [prerequisiti](/help/using/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## Configurare un progetto App Builder {#create-App-Builder-project}

1. Verificare che nell&#39;organizzazione [!DNL Experience Cloud] sia presente un ruolo di amministratore di sistema o sviluppatore. Un amministratore di sistema, nell&#39;[Admin Console](https://adminconsole.adobe.com/overview), imposta questo ruolo.

1. Accedi a [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis). Assicurarsi di far parte della stessa organizzazione [!DNL Experience Cloud] di [!DNL Experience Manager] come integrazione [!DNL Cloud Service]. Per ulteriori informazioni su Adobe Developer Console, consulta la [documentazione della console](https://developer.adobe.com/developer-console/docs/guides/).

1. [Crea un progetto App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app/). Fai clic su **[!UICONTROL Crea nuovo progetto]** > **[!UICONTROL Progetto da modello]**. Seleziona App Builder. Viene creato un nuovo progetto App Builder con due aree di lavoro: `Production` e `Stage`. Aggiungere altre aree di lavoro, ad esempio `Development`, in base alle esigenze.

1. Nel progetto App Builder, seleziona un’area di lavoro e abbonati ai servizi necessari, ad Asset compute. Fare clic su **Aggiungi al progetto** > **API** e aggiungere i servizi `Asset Compute`, `IO Events` e `IO Events Management`. Quando si aggiunge la prima API, viene richiesto di creare una chiave privata. Salva queste informazioni sul computer quando necessario per testare l&#39;applicazione personalizzata con lo strumento per sviluppatori.

   >[!NOTE]
   >
   >JWT è obsoleto e la chiave privata non è disponibile per il download. Durante l’aggiornamento degli strumenti di test, tieni presente che è possibile distribuire i processi di lavoro personalizzati creati con OAuth, ma che devtools non funzionerebbe.

## Passaggio successivo {#next-step}

Una volta configurato l&#39;ambiente, puoi [creare un&#39;applicazione personalizzata](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
