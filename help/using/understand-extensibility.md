---
title: Informazioni sull'estensione di  [!DNL Asset Compute Service]
description: Quando e come estendere la funzionalità  [!DNL Asset Compute Service]  per eseguire l'elaborazione personalizzata delle risorse.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '229'
ht-degree: 1%

---

# Introduzione all’estensibilità {#introduction-to-extensibilty}

Molti requisiti di rendering, come la conversione in formati e il ridimensionamento di immagini, sono soddisfatti da [Profili di elaborazione in [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/it/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview). I requisiti aziendali più complessi possono richiedere una soluzione personalizzata che soddisfi le esigenze di un&#39;organizzazione. È possibile estendere [!DNL Asset Compute Service] creando applicazioni personalizzate richiamate dai profili di elaborazione in [!DNL Experience Manager]. Queste applicazioni personalizzate soddisfano i [casi d&#39;uso supportati](https://experienceleague.adobe.com/it/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] è disponibile solo per l&#39;utilizzo con [!DNL Experience Manager] come [!DNL Cloud Service].

Le applicazioni personalizzate sono [app Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) headless. L&#39;estensione di [!DNL Asset Compute Service] con applicazioni personalizzate è resa semplice tramite l&#39;[Asset Compute SDK](https://github.com/adobe/asset-compute-sdk) e gli strumenti per sviluppatori di Adobe Developer App Builder. Questi strumenti consentono agli sviluppatori di concentrarsi sulla logica di business. La creazione di applicazioni personalizzate è semplice come la creazione di una semplice azione di Adobe senza server [!DNL I/O Runtime]. È una singola funzione JavaScript di Node.js. L&#39;esempio [dell&#39;applicazione personalizzata di base](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) lo illustra.

## Prerequisiti e requisiti di provisioning {#prerequisites-and-provisioning}

Assicurati di soddisfare i seguenti prerequisiti:

* Gli strumenti di Adobe Developer App Builder sono installati nel computer.
* Organizzazione [!DNL Experience Cloud]. Per ulteriori informazioni, vai a [Avvia il Percorso App Builder](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* L&#39;organizzazione Experience deve avere [!DNL Experience Manager] come [!DNL Cloud Service] abilitato.
* L&#39;organizzazione [!DNL Adobe Experience Cloud] fa parte del programma [!DNL Adobe Developer App Builder] Developer Sneak Peek. Vai a [come richiedere l&#39;accesso](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* Assicurati che lo sviluppatore disponga del ruolo di sviluppatore o delle autorizzazioni di amministratore nell’organizzazione.
* Verificare che l&#39;Adobe [[!DNL aio-cli]](https://github.com/adobe/aio-cli) sia installato localmente.

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
