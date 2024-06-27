---
title: Distribuisci [!DNL Asset Compute Service] applicazione personalizzata
description: Distribuisci [!DNL Asset Compute Service] applicazione personalizzata.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '170'
ht-degree: 0%

---

# Distribuire un&#39;applicazione personalizzata {#deploy-custom-application}

Per implementare l’applicazione, utilizza [distribuzione app aio](https://github.com/adobe/aio-cli#aio-appdeploy) comando. Nel terminale, il comando visualizza un URL per accedere all’applicazione personalizzata. L’URL è nel formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Per ottenere lo stesso URL senza ridistribuire l’applicazione, utilizza [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) comando.

Utilizza l’URL in una [Profilo di elaborazione in [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/it/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use) per integrare l’applicazione con [!DNL Experience Manager] as a [!DNL Cloud Service].

Assicurati che il progetto e l’area di lavoro di App Builder corrispondano agli [!DNL Experience Manager] as a [!DNL Cloud Service] dell’ambiente in cui desideri utilizzare l’azione. Dispone di diversi ambienti per lo sviluppo, la gestione temporanea e la produzione. Puoi verificare l’ambiente controllando `AIO_runtime_*` credenziali definite nel file ENV nella directory principale dell’applicazione Adobe Developer App Builder. Ad esempio, per distribuire in un `Stage` workspace, il `AIO_runtime_namespace` è del formato `xxxxxx_xxxxxxxxx_stage`. Integrare con [!DNL Experience Manager] as a [!DNL Cloud Service] Ambiente di produzione, utilizza gli URL dell’applicazione dal tuo Adobe Developer App Builder `Production` Workspace.

>[!CAUTION]
>
>Non utilizzare un&#39;area di lavoro personale in ambienti critici [!DNL Experience Manager] ambienti.

>[!MORELIKETHIS]
>
>* [Comprendere e gestire gli ambienti in [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/manage-environments).
