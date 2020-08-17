---
title: Comentarios de revisión de las ofertas de aplicaciones de Azure - marketplace comercial de Microsoft
description: Cómo controlar los comentarios de la oferta de la aplicación de Azure del equipo de revisión de Microsoft Azure Marketplace. Puede acceder a los comentarios desde Azure DevOps con las credenciales del Centro de partners.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 11/11/2019
ms.author: mingshen
author: mingshen-ms
ms.openlocfilehash: aecc64c5fa923bc29d34efd3969c2c12f30153c4
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320109"
---
# <a name="handling-review-feedback-for-azure-application-offers"></a>Control de los comentarios de revisión de las ofertas de aplicaciones de Azure

En este artículo se explica cómo acceder a los comentarios del equipo de revisión de Microsoft Azure Marketplace desde [Azure DevOps](https://azure.microsoft.com/services/devops/). Si se detectan problemas críticos en su oferta de la aplicación de Azure durante el paso de **revisión de Microsoft**, puede iniciar sesión en este sistema para la información detallada sobre estos problemas (comentarios de revisión). Después de corregir todos los problemas, debe volver a enviar la oferta para proceder a su publicación en Azure Marketplace. En el siguiente diagrama se ilustra cómo se relacionan estos comentarios con la publicación.

![Proceso de comentarios de revisión](./media/review-feedback-process.png)

Normalmente, a los problemas de revisión se hace referencia con una solicitud de incorporación de cambios (PR). Cada solicitud de incorporación de cambios está vinculada a un elemento de Azure DevOps con los detalles sobre el problema. La imagen siguiente muestra un ejemplo de la experiencia del Centro de partners si se detectan problemas durante las revisiones. 

![Estado de publicación](./media/publishing-status.png)

La solicitud de incorporación de certificados que contiene detalles específicos sobre el envío se mencionará en el vínculo "Ver informe de certificación". Para situaciones complejas, los equipos de revisión y soporte técnico también pueden escribir por correo electrónico.

## <a name="azure-devops-access"></a>Acceso de Azure DevOps

Todos los usuarios con acceso al rol "desarrollador" en el Centro de partners podrán ver los elementos de la solicitud de incorporación de cambios a los que se hace referencia en los comentarios de revisión.

## <a name="reviewing-the-pull-request"></a>Revisión de la solicitud de incorporación de cambios

Use el procedimiento siguiente para revisar los problemas que se documentan en la solicitud de incorporación de cambios.

1. En las secciones de **Revisión de Microsoft** del formulario Pasos de publicación, seleccione un vínculo de solicitud de incorporación de cambios para iniciar el explorador e ir a la página **Información general** (página principal) de dicha solicitud. En la siguiente imagen se muestra un ejemplo de página principal para un problema crítico de una oferta de la aplicación Contoso. Esta página contiene información resumida útil acerca de los problemas de revisión que se detectaron en la aplicación de Azure.

    [![Página principal de la solicitud de incorporación de cambios](./media/pr-home-page-thumb.png)](./media/pr-home-page.png)
    <br/> *Haga clic en la imagen para expandirla.*

1. (Opcional) En el lado derecho de la ventana, en la sección **Directivas**, seleccione el mensaje del problema (en este ejemplo: **Policy Validation failed**) (Error durante la validación de directivas) para investigar los detalles de bajo nivel del problema, incluidos los archivos de registro asociados. Normalmente, los errores aparecen en la parte inferior de los archivos de registro.

1. En el menú en el lado izquierdo de la página principal, seleccione **Archivos** para mostrar la lista de archivos que componen los recursos técnicos de esta oferta. Los revisores de Microsoft deberían haber agregado comentarios que describen los problemas críticos detectados. En el ejemplo siguiente, se detectaron dos problemas.

    [![Página principal de la solicitud de incorporación de cambios](./media/pr-files-page-thumb.png)](./media/pr-files-page.png)
    <br/> *Haga clic en la imagen para expandirla.*

1. Seleccione cada nodo de comentario en el árbol de la izquierda para ver el comentario en el contexto del código circundante. Corrija el código fuente en el proyecto de equipo para corregir el problema que se describe en el comentario.

>[!Note]
>No puede editar los recursos técnicos de su oferta dentro del entorno de Azure DevOps del equipo de revisión. Para los publicadores, este es un entorno de solo lectura para el código fuente contenido. No obstante, puede dejar las respuestas a los comentarios para beneficio del equipo de revisión de Microsoft.

   En el ejemplo siguiente, el publicador ha revisado, corregido y respondido al primer problema.

   ![Primera respuesta de revisión y comentario](./media/first-comment-reply.png)

## <a name="next-steps"></a>Pasos siguientes

Después de corregir los problemas críticos que se documentan en las solicitudes de incorporación de cambios de la revisión, debe [volver a publicar la oferta de la aplicación de Azure](./create-new-azure-apps-offer.md#publish).
