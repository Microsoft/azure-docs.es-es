---
title: Prueba y publicación de una oferta de SaaS en el marketplace comercial de Microsoft
description: Use el Centro de partners para enviar la oferta de SaaS a publicación, obtener una versión preliminar de ella, probarla y, luego, publicarla en el marketplace comercial de Microsoft.
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: mingshen-ms
ms.author: mingshen
ms.date: 09/02/2020
ms.openlocfilehash: bef59ea9e7de77e7f9a80cc3950762ea70238b87
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89380503"
---
# <a name="how-to-test-and-publish-a-saas-offer-to-the-commercial-marketplace"></a>Prueba y publicación de una oferta de SaaS en el marketplace comercial

En este artículo se explica cómo usar el Centro de partners para enviar la oferta de SaaS a publicación, obtener una vista previa de ella, probarla y, luego, publicarla en el marketplace comercial. Ya debe haber creado una oferta que quiera publicar.

## <a name="submit-your-offer-for-publishing"></a>Envío de la oferta a publicación

1. Inicie sesión en el panel de marketplace comercial en el [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace/overview).
1. En la página **Información general**, seleccione la oferta que quiere publicar.
1. En la esquina superior derecha del portal, seleccione **Revisar y publicar**.
2. Asegúrese de que la columna de **Estado** de cada página indica **Completa**. Los tres estados posibles que se muestran son:

   - **No iniciado**: la página está incompleta.
   - **Incompleta**: falta información necesaria en la página o hay errores que deben corregirse. Tendrá que volver a la página anterior y actualizarla.
   - **Completa**: la página está completa. Se han proporcionado todos los datos necesarios y no hay ningún error.

1. Si alguna de las páginas tiene un estado que no sea **Completa**, seleccione el nombre de la página, corrija el problema, guarde la página y, luego, elija de nuevo **Revisar y publicar** para volver a esta página.
1. Después de completar todas las páginas, en el cuadro de **Notas para la certificación**, proporcione instrucciones de prueba al equipo de certificación para asegurarse de que la aplicación se ha probado correctamente. Proporcione cualquier nota complementaria que resulte útil para entender la aplicación.
1. Para iniciar el proceso de publicación de la oferta, seleccione **Publicar**. Aparece la página **Información general de la oferta**, que muestra el **estado de publicación** de la oferta.

El estado de publicación de la oferta cambiará a medida que avance por el proceso de publicación. Para más información sobre este proceso, consulte [Pasos de validación y publicación](review-publish-offer.md#validation-and-publishing-steps).

## <a name="preview-and-test-your-offer"></a>Obtener una versión preliminar de la oferta y probarla

Cuando la oferta esté lista para su aprobación, le enviaremos un correo electrónico para solicitarle que revise y apruebe la versión preliminar de ella. También se puede actualizar la página **Offer overview** (Información general de la oferta) en el explorador para ver si la oferta ha alcanzado la fase de aprobación del editor. Si es así, el botón **Transmitir en directo** y los vínculos de versión preliminar estarán disponibles. Habrá un vínculo para la versión preliminar de Microsoft AppSource, la versión preliminar de Azure Marketplace, o ambas, según las opciones que elija al crear la oferta. Si elige vender la oferta mediante Microsoft, cualquiera que sea agregado a la audiencia de versión preliminar podrá probar la adquisición y la implementación de esta para asegurarse de que cumple sus requisitos durante esta etapa.

En la captura de pantalla siguiente se muestra la página **Información general de la oferta** de una oferta de SaaS, con dos vínculos de versión preliminar en el botón **Publicar**. Los pasos de validación que verá en esta página varían en función de las selecciones realizadas al crear la oferta.

![Muestra la página de Información general de la oferta para una oferta en el Centro de partners. Se muestran los vínculos de versión preliminar y el botón de Transmitir en directo.](media/publish-status-publisher-signoff.png)

Siga estos pasos para obtener una versión preliminar de la oferta.

1. En la página **Información general de la oferta**, seleccione un vínculo de versión preliminar en el botón **Publicar**. 

1. Para validar el flujo de compra y configuración de un extremo a otro, compre la oferta mientras está en versión preliminar. En primer lugar, informe a Microsoft con una [incidencia de soporte técnico](https://aka.ms/marketplacesupport) para asegurarse de que no se procesen cargos.

1. Si su oferta de SaaS admite la [facturación de uso medido mediante el servicio de medición de marketplace comercial](./partner-center-portal/saas-metered-billing.md), revise y siga los procedimientos recomendados de prueba que se detallan en [API de facturación según uso de Marketplace](./partner-center-portal/marketplace-metering-service-apis.md#development-and-testing-best-practices).

1. Revise y siga las instrucciones de prueba que se indican en [API de suministro de SaaS v2 en el marketplace comercial de Microsoft](./partner-center-portal/pc-saas-fulfillment-api-v2.md#development-and-testing) para que la oferta se integre correctamente con las API antes de publicarla.

1. Si después obtener la versión preliminar de la oferta y probarla necesita realizar cambios, puede editarla y volver a enviarla para publicar una nueva versión preliminar. Para obtener más información, consulte [Actualización de una oferta existente en el marketplace comercial](./partner-center-portal/update-existing-offer.md).

## <a name="publish-your-offer-live"></a>Publicación de la oferta

Después de completar todas las pruebas de la versión preliminar, seleccione **Publicar** para publicar la oferta en el marketplace comercial.

   > [!TIP]
   > Si la oferta ya está activa en el marketplace comercial, las actualizaciones que realice no se publicarán hasta que seleccione **Publicar**.

Ahora que ha decidido que la oferta esté disponible en el marketplace comercial, se realizan una serie de comprobaciones de validación finales para garantizar que la oferta activa está configurada igual que en la versión preliminar. Para más información sobre estas comprobaciones de validación, consulte [Fase de publicación](review-publish-offer.md#publish-phase).

Después de finalizar estas comprobaciones de validación, la oferta estará activa en el marketplace.

## <a name="next-step"></a>Paso siguiente

[Acceso a los informes de análisis de marketplace comercial en el Centro de partners](./partner-center-portal/analytics.md)