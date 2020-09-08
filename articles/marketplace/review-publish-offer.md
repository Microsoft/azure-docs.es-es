---
title: Procedimiento para revisar y publicar una oferta en el marketplace comercial de Microsoft
description: Use el centro de partners para enviar su oferta a versión preliminar, obtener una versión preliminar de la oferta y, a continuación, publicarla en el marketplace comercial de Microsoft.
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: mingshen-ms
ms.author: mingshen
ms.date: 08/12/2020
ms.openlocfilehash: eff1db3aca2cab77d18698634b84e8e1b1c99f7e
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89375894"
---
# <a name="how-to-review-and-publish-an-offer-to-the-commercial-marketplace"></a>Procedimiento para revisar y publicar una oferta en el marketplace comercial

Este artículo le muestra cómo usar el Centro de partners para enviar su oferta a publicación, obtener una versión preliminar de la oferta y, a continuación, publicarla en el marketplace comercial de Microsoft. También se explica cómo comprobar el estado de la publicación a medida que avanza a través de los pasos de publicación. Ya debe haber creado una oferta que quiera publicar.

## <a name="offer-status"></a>Estado de la oferta

Puede revisar el estado de la oferta en la pestaña **Información general** del panel del marketplace comercial en el [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace/overview). Se muestra uno de los siguientes indicadores de estado en la columna de **Estado** de cada oferta.

| Estado | Descripción |
| ------------ | ------------- |
| Borrador | La oferta se ha creado pero no se va a publicar. |
| Publicación en curso | Se está realizando el proceso de publicación para la oferta. |
| Atención necesaria | Se ha detectado un problema crítico durante la certificación o durante otra fase de publicación. |
| Versión preliminar | Nosotros certificamos la oferta, misma que está en espera de la comprobación final del anunciante. Seleccione **Transmitir** para publicar la oferta en vivo. |
| En vivo | La oferta está activa en Marketplace y los clientes la pueden ver y adquirir. |
| Detener venta pendiente | El anunciante ha seleccionado "Detener venta" en la oferta o el plan, pero aún no se ha completado la acción. |
| No disponible en el marketplace | Una oferta publicada anteriormente se ha quitado del Marketplace. |
|||

## <a name="validation-and-publishing-steps"></a>Pasos de validación y publicación

Cuando esté listo para enviar una oferta para su publicación, seleccione **Revisar y publicar** en la esquina superior derecha del portal. En la página **Revisar y publicar** se muestra el estado de cada página de la oferta, que puede ser uno de los siguientes:

   - **No iniciado**: la página está incompleta.
   - **Incompleta**: falta información necesaria en la página o hay errores que deben corregirse. Tendrá que volver a la página anterior y actualizarla.
   - **Completa**: la página está completa. Se han proporcionado todos los datos necesarios y no hay ningún error.

Si alguna de las páginas tiene un estado distinto de **Completo**, debe corregir el problema de la página y, a continuación, volver a la página **Revisar y publicar** para confirmar que el estado es ahora **Completo**. Algunos tipos de oferta requieren pruebas. En este caso, verá un campo **Notas para la certificación** donde deberá proporcionar instrucciones para la realización de pruebas al equipo de certificación, además de todas las notas adicionales que pueden resultar útiles para comprender la aplicación.

Cuando todas las páginas estén completas y haya escrito las notas de prueba aplicables, seleccione **Publicar** para iniciar los procesos de validación y publicación. Las fases y la secuencia general pueden variar en función del tipo de oferta que esté publicando. En la tabla siguiente se muestra un posible flujo de publicación. Cada una de las fases se explica con más detalle en las secciones siguientes.

| Fase | Qué ocurre |
| ------------ | ------------- | ------------- |
| [Validación automatizada](#automated-validation-phase) | Procesamos un conjunto de validaciones automatizadas. |
| [Certificación](#certification-phase) | Llevamos a cabo validaciones manuales. |
| [Creación de versión preliminar](#preview-creation-phase) | La página del anuncio de la versión preliminar de la oferta está disponible para cualquiera que tenga el vínculo de versión preliminar. Si su oferta se va a vender a través de Microsoft (transactable), solo la audiencia que haya especificado en la página de **Audiencia de versión preliminar** de su oferta puede comprar y obtener acceso a la oferta para realizar pruebas. |
| [Aprobación del publicador](#publisher-sign-off-phase) | Le enviaremos un correo electrónico con una solicitud para que pueda obtener una versión preliminar y aprobar su oferta. |
| [Publicar](#publish-phase) | Ejecutamos una serie de pasos para comprobar que la oferta de versión preliminar se publica en directo en el marketplace comercial. |
|||

## <a name="automated-validation-phase"></a>Fase de validación automatizada

El primer paso del proceso de publicación es realizar un conjunto de validaciones automatizadas. Cada paso de validación corresponde a una característica que eligió al crear la oferta. Para que la oferta pueda avanzar al siguiente paso del proceso de publicación, se deben realizar todas las comprobaciones de validación.

- **Configuración del flujo de compra de la oferta (>10 min)**

   La oferta debe poderse satisfacer cuando los clientes la compran mediante Azure Portal. Este paso solo es aplicable para las ofertas de venta mediante Microsoft.

- **Validación de los datos de la versión de prueba (~5 min)**

   Los datos proporcionados se validan en la página de configuración técnica de la oferta. Se pone a prueba y se aprueba la funcionalidad de la versión de prueba. Este paso solo es aplicable para las ofertas con una versión de prueba habilitada.

- **Aprovisionamiento de la versión de prueba (~30 min)**

    Después de validar los datos y la funcionalidad de la versión de prueba en el paso anterior, se implementan y replican las instancias de la versión de prueba para que estén listas para su uso por parte de los clientes. Este paso solo es aplicable para las ofertas con una versión de prueba habilitada.

- **Registro y validación de la administración de clientes potenciales (>15 min)**

    Se confirma que el sistema de administración de clientes potenciales puede recibir clientes potenciales en función de los detalles que se proporcionaron en la página **Configuración de la oferta**. Este paso solo es aplicable para las ofertas con la administración de clientes potenciales habilitada.

## <a name="certification-phase"></a>Fase de certificación

Antes de que se publiquen, las ofertas enviadas al marketplace comercial se deben certificar. Las ofertas se someten a pruebas rigurosas, algunas automáticas y otras manuales. Para más información, consulte las [directivas de certificación de marketplace comercial](https://aka.ms/commercial-marketplace-certification-policies).

### <a name="types-of-validation-that-take-place-during-certification"></a>Tipos de validación que se realizan durante la certificación

El proceso de certificación incluye tres niveles de validación para cada oferta enviada.

- Aptitud de la empresa anunciante
- Validación del contenido
- Validación técnica

#### <a name="publisher-business-eligibility"></a>Aptitud de la empresa anunciante

En cada tipo de oferta se comprueba un conjunto de criterios de aptitud básicos. Estos criterios pueden incluir el estado de MPN del editor, las competencias y su grado, etc.

#### <a name="content-validation"></a>Validación del contenido

Se comprueba la calidad y pertinencia de la información que se introdujo al crear la oferta. Con estas comprobaciones se revisan las entradas de los detalles del anuncio en marketplace, los precios, la disponibilidad, los planes asociados, y demás. Para cumplir los criterios de anuncios de Microsoft AppSource y Azure Marketplace, validaremos que la oferta incluya:

- un título que describa la oferta con precisión
- descripciones bien escritas que proporcionen información general detallada y una propuesta de valor
- capturas de pantallas y vídeos de calidad
- una explicación de cómo la oferta usa las plataformas y las herramientas de Microsoft.

Para más información sobre los criterios de validación del contenido, consulte las [directivas generales de los anuncios](https://aka.ms/commercial-marketplace-certification-policies#100-general).

#### <a name="technical-validation"></a>Validación técnica

Durante la validación técnica, la oferta (paquete o binario) se somete a las siguientes comprobaciones.

- Examen en busca de malware
- Monitorización de las llamadas de red
- Análisis de los paquetes
- Análisis exhaustivo de la funcionalidad de la oferta

La oferta se prueba en varias plataformas y con distintas versiones para garantizar su solidez.

### <a name="certification-failure-report"></a>Informe de error de certificación

Si se produce algún error en las comprobaciones del anuncio, técnicas o de las directivas, o si no es apto para enviar una oferta de ese tipo, se genera un informe de error de certificación, que se le envía por correo electrónico.

Este informe contiene descripciones de las directivas con error, junto con notas de revisión. Revise este informe de correo electrónico, solucione los problemas, actualice la oferta según sea necesario y vuelva a enviarla mediante el [portal de marketplace comercial](https://partner.microsoft.com/dashboard/commercial-marketplace/offers) del Centro de partners. Puede volver a enviar las ofertas las veces que sea necesario hasta que se supere la certificación.

## <a name="preview-creation-phase"></a>Fase de versión preliminar

Durante la fase de creación de la versión preliminar, creamos una versión de la oferta que será accesible solo a la audiencia que especificó en la página de **Audiencia de versión preliminar** de su oferta, si la hay. La versión preliminar de la oferta no estará disponible para ningún usuario ajeno a la versión preliminar de la audiencia hasta que publique la oferta en directo.

> [!NOTE]
> No utilice la audiencia de versión preliminar para otorgar visibilidad sobre la oferta a personas ajenas a la organización. Use la opción de Oferta privada en su lugar. En este momento, la oferta no se ha probado y validado por completo y no está lista para distribuirse fuera de la organización.

## <a name="publisher-sign-off-phase"></a>Fase de aprobación del editor

Cuando la oferta esté lista para su revisión y aprobación, se le enviará un correo electrónico para solicitarle que revise y apruebe la versión preliminar de la oferta. También se puede actualizar la página **Offer overview** (Información general de la oferta) en el explorador para ver si la oferta ha alcanzado la fase de aprobación del editor. Si es así, el botón **Transmitir en directo** y los vínculos de versión preliminar estarán disponibles.

En la captura de pantalla siguiente se muestra la página de **Información general de la oferta** para una oferta de SaaS. Los pasos de validación que verá en esta página varían en función del tipo de oferta y de las selecciones realizadas al crear la oferta.

![Muestra la página de Información general de la oferta para una oferta en el Centro de partners. Se muestran los vínculos de versión preliminar y el botón de Transmitir en directo.](./media/publish-status-publisher-signoff.png)

### <a name="previewing-and-approving-your-offer"></a>Obtención de la versión preliminar y aprobación de la oferta

> [!IMPORTANT]
> Para validar el flujo de compra y configuración de un extremo a otro, compre la oferta mientras está en versión preliminar. En primer lugar, informe a Microsoft con una [incidencia de soporte técnico](https://aka.ms/marketplacesupport) para asegurarse de que no se procesarán cargos.

En la página **Información general de la oferta**, verá vínculos de versión preliminar bajo el botón **Transmitir**. Habrá un vínculo para la versión preliminar de AppSource, la versión preliminar de Azure Marketplace o ambos según las opciones que elija al crear la oferta. Si elige vender la oferta mediante Microsoft, cualquiera que sea agregado a la audiencia de versión preliminar podrá probar la adquisición y la implementación de esta para asegurarse de que cumple sus requisitos durante esta etapa.

Después de aprobar la versión preliminar, seleccione **Transmitir** para publicar su oferta en el marketplace comercial. 

Si desea realizar cambios después obtener la versión preliminar de la oferta, puede editar y volver a enviar la solicitud de publicación. Si la oferta ya está activa y disponible para el público en el marketplace, las actualizaciones que realice no se activarán hasta que seleccione **Transmitir*. Para obtener más información, consulte [Actualización de una oferta existente en el marketplace comercial](./partner-center-portal/update-existing-offer.md).

## <a name="publish-phase"></a>Fase de publicación

Ahora que ha decidido publicar la oferta, lo que hará que esté disponible en el marketplace comercial, hay una serie de comprobaciones de validación finales que se realizan para garantizar que la oferta activa está configurada igual que en la versión preliminar.

- **Configuración del flujo de compra de la oferta (>10 min)**

    La oferta debe poderse satisfacer cuando los clientes la compran mediante Azure Portal. Este paso solo es aplicable para las ofertas de venta mediante Microsoft.

- **Validación de los datos de la versión de prueba (~5 min)**

    Los datos proporcionados se validan en la página de configuración técnica de la oferta. Se pone a prueba y se aprueba la funcionalidad de la versión de prueba. Este paso solo es aplicable para las ofertas con una versión de prueba habilitada.

- **Aprovisionamiento de la versión de prueba (~30 min)**

    Las instancias de la versión de prueba se implementan y replican para que estén listas para su uso por parte de los clientes. Este paso solo es aplicable para las ofertas con una versión de prueba habilitada.

- **Registro y validación de la administración de clientes potenciales** (>15 min)

    Se confirma que el sistema de administración de clientes potenciales puede recibir clientes potenciales en función de los detalles proporcionados en la página **Offer setup** (Configuración de la oferta). Este paso solo es aplicable para las ofertas con la administración de clientes potenciales habilitada.

- **Publicación final (>30 minutos)**

    Existe la garantía de que la oferta estará públicamente disponible en el marketplace.

Después de finalizar estas comprobaciones de validación, la oferta estará activa en el marketplace.

## <a name="next-steps"></a>Pasos siguientes

[Acceso a los informes de análisis de marketplace comercial en el Centro de partners](partner-center-portal/analytics.md)
