---
title: Introduzione alla [!DNL Asset Compute Service]
description: '"[!DNL Asset Compute Service] è un servizio di elaborazione delle risorse nativo per il cloud che riduce la complessità e migliora la scalabilità".'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '278'
ht-degree: 1%

---

# Panoramica di [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] è un servizio scalabile ed estensibile di [!DNL Adobe Experience Cloud] per elaborare le risorse digitali. Può trasformare immagini, video, documenti e altri formati di file in diverse rappresentazioni, tra cui miniature, testo e metadati estratti e archivi.

Gli sviluppatori possono collegare applicazioni per risorse personalizzate (o processi di lavoro personalizzati) per risolvere i casi di utilizzo personalizzati. Il servizio funziona sull&#39;Adobe [!DNL I/O Runtime]. È estendibile tramite [!DNL Adobe Developer App Builder] app headless scritte in Node.js. Possono eseguire operazioni personalizzate, come chiamare API esterne per eseguire operazioni sulle immagini o sfruttare [!DNL Adobe Sensei] supporto.

[!DNL Adobe Developer App Builder] è un framework per generare e distribuire applicazioni web personalizzate su Adobe [!DNL I/O Runtime] per estendere le soluzioni Adobe Experience Cloud. Per creare applicazioni personalizzate, gli sviluppatori possono sfruttare [!DNL React Spectrum] (Adobe UI toolkit), creare microservizi, eventi personalizzati e orchestrare API. Consulta [documentazione di Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview/).

>[!NOTE]
>
>Attualmente, il [!DNL Asset Compute Service] utilizzabile solo tramite [!DNL Experience Manager] as a [!DNL Cloud Service]. Gli amministratori creano profili di elaborazione che possono richiamare [!DNL Asset Compute Service] per trasferire le risorse da elaborare. Consulta [utilizzo dei microservizi delle risorse e dei profili di elaborazione](https://experienceleague.adobe.com/it/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

## Casi d’uso supportati di [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] supporta alcuni casi d’uso aziendali comuni, ad esempio l’elaborazione di base delle immagini, conversioni specifiche di applicazioni di Adobe e creazione di applicazioni personalizzate che orchestrano requisiti aziendali complessi.

È possibile utilizzare [!DNL Asset Compute] servizio web per generare miniature per diversi tipi di file, rendering di immagini di alta qualità per [formati di file supportati](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support). Consulta [casi d’uso supportati tramite configurazione personalizzata](https://experienceleague.adobe.com/it/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>Il servizio non fornisce l’archiviazione delle risorse. Gli utenti lo forniscono e forniscono riferimenti alle posizioni dei file di origine e di rendering nell’archiviazione cloud.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Panoramica dell’elaborazione delle risorse con i microservizi per le risorse in [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/it/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview).
>* [Documentazione di Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview).
>* [Formati di file supportati per l’elaborazione](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support).

<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
