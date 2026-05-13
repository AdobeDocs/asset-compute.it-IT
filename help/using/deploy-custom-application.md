---
title: Distribuisci  [!DNL Asset Compute Service] applicazione personalizzata
description: Distribuisci  [!DNL Asset Compute Service] applicazione personalizzata.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
TQID: https://experienceleague.adobe.com/JN29pTaNB93DKALUqIbXhwswlzHiYQSowFZAbgHA5TA
product_v2:
  - id: d09181b5-a36a-43de-ba01-36641440bc43
  - id: fd1f54a9-f50c-467d-8956-cebbaf4f3eb8
role_v2:
  - id: ff6a42d2-313e-452e-93a6-792e4fad9ff8
source-git-commit: 2510f77fed8d0f0708e09f32d0b13a437d2ede4f
workflow-type: tm+mt
source-wordcount: 210
ht-degree: 7%

---

# Distribuire un&#39;applicazione personalizzata {#deploy-custom-application}

Per distribuire l&#39;applicazione, utilizzare il comando [distribuzione app aio](https://github.com/adobe/aio-cli#aio-appdeploy). Nel terminale, il comando visualizza un URL per accedere all’applicazione personalizzata. L&#39;URL è in formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Per ottenere lo stesso URL senza ridistribuire l&#39;applicazione, utilizzare il comando [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action).

Utilizza l&#39;URL in un [profilo di elaborazione in [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/it/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use) per integrare l&#39;applicazione con [!DNL Experience Manager] as a [!DNL Cloud Service].

Assicurati che il progetto e l&#39;area di lavoro di App Builder corrispondano all&#39;ambiente [!DNL Experience Manager] come [!DNL Cloud Service] in cui desideri utilizzare l&#39;azione. Dispone di diversi ambienti per lo sviluppo, la gestione temporanea e la produzione. Puoi verificare l&#39;ambiente controllando le credenziali di `AIO_runtime_*` definite nel file ENV nella radice dell&#39;applicazione Adobe Developer App Builder. Ad esempio, per distribuire in un&#39;area di lavoro `Stage`, `AIO_runtime_namespace` è in formato `xxxxxx_xxxxxxxxx_stage`. Per l&#39;integrazione con [!DNL Experience Manager] come ambiente di produzione [!DNL Cloud Service], utilizzare gli URL dell&#39;applicazione dall&#39;area di lavoro Adobe Developer App Builder `Production`.

>[!CAUTION]
>
>Non utilizzare un&#39;area di lavoro personale negli ambienti [!DNL Experience Manager] critici.

>[!MORELIKETHIS]
>
>* [Comprendere e gestire gli ambienti in [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/it/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/manage-environments).
