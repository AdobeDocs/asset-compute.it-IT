---
title: Applicazione [!DNL Asset Compute Service] personalizzata Deployed.
description: Applicazione [!DNL Asset Compute Service] personalizzata Deployed.
translation-type: tm+mt
source-git-commit: 1c2a1dc41296bf26c432c51b5afa20cb07a4c5c5
workflow-type: tm+mt
source-wordcount: '201'
ht-degree: 8%

---


# Distribuzione di un&#39;applicazione personalizzata {#deploy-custom-application}

Per distribuire l&#39;applicazione, utilizza il comando [di distribuzione](https://github.com/adobe/aio-cli#aio-appdeploy) app aio. Nel terminale, il comando visualizza un URL per accedere all&#39;applicazione personalizzata. L&#39;URL è in un formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Per ottenere lo stesso URL senza ridistribuire l&#39;applicazione, utilizzare [`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action) il comando.

Utilizzate l&#39;URL in un profilo di [elaborazione in  Experience Manager come Cloud Service](https://docs.adobe.com/content/help/it-IT/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) per integrare l&#39;applicazione [!DNL Experience Manager] come Cloud Service.

Assicurati che il progetto Firefly e l’area di lavoro corrispondano al [!DNL Experience Manager] come ambiente Cloud Service in cui desideri usare l’azione. Dispone di ambienti diversi per lo sviluppo, l&#39;installazione e la produzione. È possibile verificare l&#39;ambiente controllando `AIO_runtime_*` le credenziali definite all&#39;interno del file ENV nella radice dell&#39;applicazione Firefly. Ad esempio, per eseguire la distribuzione in un’ `Stage` area di lavoro, il formato `AIO_runtime_namespace` è `xxxxxx_xxxxxxxxx_stage`. Per l&#39;integrazione con [!DNL Experience Manager] come ambiente di produzione Cloud Service, utilizzate gli URL dell&#39;applicazione presenti nell&#39;area di lavoro `Production` Firefly.

>[!CAUTION]
>
>Non utilizzate un&#39;area di lavoro personale in [!DNL Experience Manager] ambienti critici.

>[!MORELIKETHIS]
>
>* [Comprendere e gestire gli ambienti in  Experience Manager come Cloud Service](https://docs.adobe.com/content/help/it-IT/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).
