---
title: Comprobar el estado de publicación de su oferta de Marketplace comercial
description: Compruebe el estado de la validación, certificación y vista previa pasos necesarios para publicar una oferta a través de Marketplace comercial en Microsoft Partner Center.
author: mattwojo
manager: evansma
ms.author: mattwoj
ms.service: marketplace
ms.topic: conceptual
ms.date: 05/30/2019
ms.openlocfilehash: 461c9f3f3725ba27410088ca19f1ec050375adf2
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65806152"
---
# <a name="check-the-publishing-status-of-your-commercial-marketplace-offer"></a>Comprobar el estado de publicación de su oferta de Marketplace comercial

Puede ver su actual **estado de publicación** en el **ofrecen información general sobre** pestaña de la [portal Marketplace comercial](https://partner.microsoft.com/dashboard/commercial-marketplace/offers) en Centro de partners.

## <a name="automated-validation"></a>Validación automatizada

El primer paso del proceso de publicación es un conjunto de validaciones automatizadas. Cada paso de validación que se corresponde con una característica que eligió para habilitar la creación de su oferta. Si esa característica no se ha habilitado la validación salta hasta el siguiente paso de publicación. Cada comprobación de validación debe realizarse antes de que el estado de publicación se ha aprobado.

- **Configuración del flujo de compra de la oferta (> 10 min)**

En este paso, nos aseguramos de que su oferta puede cumplirse cuando se adquieren por los clientes a través del portal de Azure. Este paso solo es aplicable para las ofertas de venta a través de Microsoft.

- **Probar la validación de datos de unidad (~ 5 min)**

En este paso, se validan los datos que proporcionó en la sección de configuración técnica de la oferta de la versión de prueba. Funcionalidad de la unidad de prueba ha probado y aprobado. Este paso solo es aplicable para las ofertas con una versión de prueba habilitada.

- **Prueba de unidad de aprovisionamiento (~ 30 min)**

En este paso después de validar los datos y la funcionalidad de la versión de prueba en el paso anterior, se implementa y se replican las instancias de la versión de prueba para que estén listos para el uso del cliente.  Este paso solo es aplicable para las ofertas con una versión de prueba habilitada.

- **Llevar el registro y la validación de administración (> 15 minutos)**

En este paso, confirmamos que su sistema de administración de clientes potenciales puede recibir clientes potenciales según los detalles proporcionados en el programa de instalación de la oferta. Este paso solo es aplicable para las ofertas con administración de clientes potenciales habilitada.

## <a name="certification"></a>Certificación

Antes de que se publica, deben estar certificadas ofertas presentadas a Marketplace en el centro de partners comerciales. Enviar ofertas se someten a pruebas rigurosas, algunos automatizadas y otros manuales, entre ellos una comprobación contra el [directivas de certificación de Azure Marketplace](https://docs.microsoft.com/legal/marketplace/general-policies). Ofrecen envíos deben marcarse aptos para la certificación antes de continuar con el paso siguiente en el flujo de publicación.

### <a name="types-of-validation-that-take-place-during-certification"></a>Tipos de validación que tienen lugar durante la certificación

Hay tres niveles de validación que se incluyen en el proceso de certificación para cada oferta presentada.

- Elegibilidad de negocio de publicador
- Validación de contenido
- Validación técnica

#### <a name="publisher-business-eligibility"></a>Elegibilidad de negocio de publicador

Cada tipo de oferta comprueba un conjunto de criterios de elegibilidad base que debe cumplir el publicador. Estado MPN del publicador, mantenidas las competencias, niveles de competencia, etcetera, pueden incluir criterios de elegibilidad.

#### <a name="content-validation"></a>Validación de contenido

Durante la validación de contenido, se comprueba la información que escribió cuando creó su oferta de calidad y la relevancia. Estas comprobaciones revisará las entradas de la lista de detalles, precios, disponibilidad, los planes asociados, etcetera de marketplace. Para cumplir con el Azure Marketplace o AppSource listado criterios, se validará que la oferta incluye:

- un título que se describe con precisión la oferta;
- descripciones bien escritas que proporcionan una introducción exhaustiva y propuesta de valor;
- capturas de pantalla de calidad y la adjunta vídeos; y
- explicación de cómo utiliza la oferta de herramientas y plataformas de Microsoft.

Más información sobre los criterios de validación de contenido, consulte el [general enumerar directivas](https://docs.microsoft.com/legal/marketplace/general-policies#10-general-listing-policies).

#### <a name="technical-validation"></a>Validación técnica

Durante la validación técnica, la oferta (paquete o binario) se somete a las siguientes comprobaciones.
- Examinar en busca de malware
- Llamadas de red supervisadas
- Analizar el paquete
- Un análisis completo de funcionalidad real de la oferta

La oferta se ha probado en varias versiones y plataformas con el fin de asegurarse de que es sólido.

Revise los detalles de configuración específica necesarios para la oferta en la sección de configuración técnica de este documento.

### <a name="certification-failure-report"></a>Informe de errores de certificación

Tras la finalización de la revisión, si su oferta ha pasado la certificación, a continuación, mueve a lo largo de con el siguiente paso del proceso de publicación. Si su oferta ha dado error cualquiera de los controles de lista, técnico o la directiva, o si no es apto para enviar una oferta de ese tipo, un informe de errores de certificación es genera y envía por correo electrónico para usted.

Este informe contiene descripciones de las directivas con error, junto con notas de revisión. Revise este informe de correo electrónico, solucione cualquier problema, realizar actualizaciones en la oferta donde sea necesario y vuelva a enviar la oferta utilizando el [portal Marketplace comercial](https://partner.microsoft.com/dashboard/commercial-marketplace/offers) en Centro de partners. (Se puede volver a enviar la oferta tantas veces como sea necesario hasta que el paso de certificación).

## <a name="preview-creation"></a>Creación de vista previa

Durante la **obtener una vista previa de creación** paso, creamos una versión de la oferta accesible sólo a la audiencia que especificó en la sección de vista previa de la oferta.

## <a name="publisher-approval"></a>Aprobación del publicador

En este paso, se enviará una solicitud para revisar y aprobar la versión preliminar de la oferta antes del último paso de publicación.

Si ha seleccionado para vender su oferta a través de Microsoft, podrá probar la adquisición e implementación de la oferta para asegurarse de que cumple sus requisitos durante esta fase de aprobación de versión preliminar. La oferta no aún estará disponible en marketplace pública. Después de probar y aprobar esta versión preliminar, deberá seleccionar **Go-Live** en el [ **ofrecen información general** ](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) panel.

Si desea realizar cambios en la oferta durante esta fase de vista previa, puede editar y volver a enviar para publicar una nueva vista previa. Consulte el artículo [actualización de ofertas de marketplace existente](#update-existing-marketplace-offers) para obtener más información sobre los cambios más.

Si la oferta ya está disponible para el público en marketplace y en vivo, todas las actualizaciones que realice no entraré en directo hasta que seleccione **Go-live** en el [ **ofrecen información general** ](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) panel.

### <a name="publish-offer-to-the-public"></a>Publicar la oferta pública

Inicie sesión en el centro de partners y tener acceso a la oferta. Se le redirigirá a la **ofrecen información general sobre** página. En la parte superior de esta página, verá una opción para **lanzarla**. Seleccione **lanzarla,** y después de confirmar, iniciará la oferta de su publicación para el público. Recibirá una notificación por correo electrónico cuando la oferta está activa.

## <a name="publish"></a>Publicar

Ahora que ha seleccionado para **lanzarla** con su oferta, que están disponibles en marketplace, hay una serie de comprobaciones de validación final que se depurará paso a paso a través para asegurarse de que la oferta en vivo se configura igual que la versión preliminar versión de la oferta.

- **Configuración del flujo de compra de la oferta (> 10 min)**

En este paso, nos aseguramos de que su oferta puede cumplirse cuando se adquieren por los clientes a través del portal de Azure. Este paso solo es aplicable para las ofertas de venta a través de Microsoft.

- **Probar la validación de datos de unidad (~ 5 min)**

En este paso, se validan los datos que proporcionó en la sección de configuración técnica de la oferta de la versión de prueba. Funcionalidad de la unidad de prueba ha probado y aprobado. Este paso solo es aplicable para las ofertas con una versión de prueba habilitada.

- **Prueba de unidad de aprovisionamiento (~ 30 min)**

En este paso, se implementa y se replican las instancias de la versión de prueba para que estén listos para el uso del cliente.  Este paso solo es aplicable para las ofertas con una versión de prueba habilitada.

- **Llevar el registro y la validación de administración (> 15 minutos)**

En este paso, confirmamos que su sistema de administración de clientes potenciales puede recibir clientes potenciales según los detalles proporcionados en el programa de instalación de la oferta. Este paso solo es aplicable para las ofertas con administración de clientes potenciales habilitada.

- **Publicar final (> 30 minutos)**

En este paso, nos aseguramos de que su oferta estará disponible públicamente en marketplace.

## <a name="update-existing-marketplace-offers"></a>Actualizar las ofertas de marketplace existente

Si desea realizar cambios en una oferta que ya se ha publicado, deberá actualizar primero la oferta existente y, a continuación, publicarlo de nuevo.

## <a name="next-steps"></a>Pasos siguientes

- [Actualizar una oferta en Marketplace comercial existente](./update-existing-offer.md)
