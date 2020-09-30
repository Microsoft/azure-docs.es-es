---
title: Actualización de una oferta existente de marketplace comercial
description: En este artículo se describe cómo realizar actualizaciones en una oferta de marketplace comercial existente, por ejemplo, editar y eliminar un borrador, cancelar una solicitud de publicación, dejar de vender una oferta o plan y sincronizar audiencias privadas.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: keferna
ms.author: keferna
ms.date: 01/16/2020
ms.openlocfilehash: f83f5da03d2db5354b020ce7d0c3c8d70f1830a0
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89650097"
---
# <a name="update-an-existing-offer-in-the-commercial-marketplace"></a>Actualización de una oferta existente en el marketplace comercial

Puede ver las ofertas existentes en la pestaña **Información general** del [portal de Marketplace comercial](https://partner.microsoft.com/dashboard/commercial-marketplace/offers) en Centro de partners.

Para actualizar una oferta existente que actualmente está activa en el marketplace comercial, siga estos pasos:

1. Seleccione el nombre de la oferta que desea actualizar. El estado de la oferta puede aparecer como **Preview** (Versión preliminar), **Live** (Activa), **Publish in progress** (Publicación en curso), **Draft** (Borrador), **Attention needed** (Requiere atención) o **Not available** (No disponible) (si anteriormente ha elegido dejar de vender la oferta). Una vez seleccionado, se abrirá la página **Información general de la oferta** para dicha oferta.
2. Seleccione **Update** (Actualizar) en la tarjeta de la página de información general de la oferta, o el elemento de menú del panel de navegación izquierdo correspondiente al área que quiere actualizar. Puede que desee actualizar alguno de estos elementos: **Configuración de la oferta**, **Propiedades**, **Descripción de la oferta**, **Vista previa**, **Configuración técnica** , **Información general del plan** o **Versión de prueba**.
3. Realice los cambios y, después seleccione **Save draft** (Guardar borrador). Repita este proceso hasta que completar todos los cambios.

## <a name="review-and-publish-an-updated-offer"></a>Revisión y publicación de una oferta actualizada

Cuando esté listo para publicar la oferta actualizada, seleccione **Review and publish** (Revisar y publicar) en cualquier página. Se abrirá la página **Review and publish** (Revisar y publicar). En esta página puede realizar las siguientes acciones:

- Consultar el estado de finalización de las secciones de la oferta que ha actualizado: 
    - **Cambios sin publicar**: La sección se ha actualizado y completado. Se han proporcionado todos los datos necesarios y no ha habido errores en las actualizaciones.
    - **Incompletos**: Las actualizaciones realizadas en la sección han introducido errores que tienen que corregirse o es necesario proporcionar más información.
- Proporcionar información adicional al equipo de pruebas de certificación para asegurarse de que las pruebas se realizan sin problemas.
- Seleccione **Publish** (Publicar) para publicar la oferta actualizada.  Le enviaremos un correo electrónico cuando haya alguna versión preliminar de la oferta actualizada disponible para su revisión y aprobación.

> [!IMPORTANT]
> Debe revisar la versión preliminar de la oferta una vez que esté disponible y seleccionar **Transmitir** para publicar su oferta actualizada para los destinatarios previstos (públicos o privados).

## <a name="add-a-plan-to-an-existing-offer"></a>Agregar un plan a una oferta existente

Para realizar cambios en un plan dentro de una oferta existente que ya haya publicado:

1. Con la página de **Información general de la oferta** de la oferta existente abierta, vaya a la página de **Información general del plan** y, a continuación, seleccione **Crear un nuevo plan**.
1. Cree un nuevo plan de acuerdo con las [directrices](./create-new-saas-offer.md#plan-overview) mediante el **modelo de precios de planes existentes**.
1. Seleccione **Guardar borrador** después de cambiar el nombre del plan.
1. Seleccione **Publicar** cuando esté listo para publicar las actualizaciones. Se abrirá la página **[Revisar y publicar](#review-and-publish-an-updated-offer)** proporcionando un estado de finalización para las actualizaciones.

## <a name="update-a-plan-within-an-existing-offer"></a>Actualización de un plan dentro de una oferta existente

Para realizar cambios en un plan dentro de una oferta existente que ya haya publicado:

1. Con la página de **Información general de la oferta** de la oferta existente abierta, elija el plan que quiera cambiar. Si el plan no es accesible desde la lista de **Información general del plan**, seleccione **Ver todos los planes**.
1. Seleccione los valores de **Name** (Nombre), **Pricing model** (Modelo de precios) o **Availability** (Disponibilidad) del plan. *En la actualidad, los planes solo están disponibles en inglés (Estados Unidos)* .
1. Seleccione **Save draft** (Guardar borrador) después de realizar los cambios necesarios en el nombre, la descripción, el público o la disponibilidad del plan.
1. Si está listo para publicar sus actualizaciones, seleccione **Review and publish** (Revisar y publicar). Se abrirá la página **[Revisar y publicar](#review-and-publish-an-updated-offer)** proporcionando un estado de finalización para las actualizaciones.
1. Seleccione **Publish** (Publicar) para publicar el plan actualizado. Le enviaremos un correo electrónico cuando haya alguna versión preliminar de la oferta actualizada disponible para su revisión y aprobación.

## <a name="offer-a-virtual-machine-plan-at-a-new-price"></a>Oferta de un plan de máquina virtual a un nuevo precio

Una vez publicado un plan de máquina virtual, no se puede cambiar su precio. Para ofrecer el mismo plan a un precio diferente, debe ocultar el plan y crear otro con el precio actualizado. En primer lugar, oculte el plan con el precio que quiere cambiar:

1. Con la página de **Información general de la oferta** de la oferta existente abierta, elija el plan que quiera cambiar. Si el plan no es accesible desde la lista de **Información general del plan**, seleccione **Ver todos los planes**.
1. Active la casilla **Ocultar plan**. Guarde el borrador antes de continuar.

Ahora que ha ocultado el plan con el precio anterior, cree una copia de ese plan con el precio actualizado:

1. En el Centro de partners, vuelva a **Plan overview** (Información general del plan).
2. Seleccione **Crear nuevo plan**. Rellene los campos **Id. de plan** y **Nombre del plan** y, luego, seleccione **Crear**.
1. Para volver a usar la configuración técnica del plan que ha ocultado, active la casilla **Reuse technical configuration** (Reutilizar la configuración técnica). Lea [Información general del plan](azure-vm-create-offer.md#plan-overview) para más información.
    > [!IMPORTANT]
    > Si selecciona **This plan reuses technical configuration from another plan** (Este plan reutiliza la configuración técnica de otro plan), no podrá dejar de vender el plan principal más adelante. No use esta opción si quiere dejar de vender el plan principal.
3. Complete todas las secciones necesarias del nuevo plan, incluido el nuevo precio.
1. Seleccione **Guardar borrador**.
1. Cuando haya completado todas las secciones necesarias del nuevo plan, seleccione **Review and publish** (Revisar y publicar). Esta acción envía la oferta para su revisión y publicación. Lea [Revisión y publicación de una oferta en el marketplace comercial](../review-publish-offer.md) para más información.

## <a name="compare-changes-to-commercial-marketplace-offers"></a>Comparación de cambios en las ofertas de marketplace comercial

Puede auditar los cambios que realice en una oferta [publicada](#compare-changes-to-published-offer) o de [versión preliminar](#compare-changes-to-a-preview-offer) antes de que estén en directo utilizando **Comparar**.

> [!NOTE]
> Una oferta publicada es una oferta que se ha publicado correctamente en estado Versión preliminar o En directo.

A continuación encontrará información general sobre auditorías:

- Puede usar **Compare** (Comparar) en cualquier momento del proceso de edición.
- Seleccione un campo en la página **Compare** (Comparar) para ir al valor que desea modificar.
- Si está listo para publicar sus actualizaciones, seleccione **Review and publish** (Revisar y publicar).
- Para ver los valores de todos los campos, hasta de los que no están actualizados, seleccione el filtro **Todos los campos**. Para modificar los filtros de estos campos, seleccione **Modified fields** (Campos modificados) y, después, seleccione uno de los siguientes filtros:
    - El filtro **Removed values** (Valores eliminados) muestra los campos que ha publicado y que está eliminando completamente.
    - El filtro **Added values** (Valores agregados) muestra los campos que no publicó originalmente y agrega ahora.
    - El filtro **Edited values** (Valores editados) muestra campos que se habían publicado, pero ahora ha actualizado el contenido.

      >[!NOTE]
      > Si alguno de estos filtros no está disponible, indica que no ha realizado ninguna actualización de ese tipo.

- Para ver solo los valores que no se han actualizado, seleccione el campo **Unchanged fields** (Campos sin cambios). Los valores de los campos que se muestran para la versión de borrador y la publicada serán los mismos.

  ![Los filtros para comparar las actualizaciones de una oferta publicada o de versión preliminar](./media/compare-changes-marketplace.png)

>[!NOTE]
> Las siguientes páginas no admiten actualmente **Compare** (Comparar):
>- Audiencia de revendedores de CSP
>- Configuración técnica de la versión de prueba
>- Listas de Marketplace de la versión de prueba
>- Venta conjunta
>- Archivos complementarios

### <a name="compare-changes-to-published-offer"></a>Comparación de cambios en ofertas publicadas

Siga las instrucciones que aparecen a continuación para comparar los cambios de la oferta publicada:

1. Seleccione **Compare** (Comparar) en la barra de comandos de la página. La página **Compare** (Comparar) proporciona versiones en paralelo de los cambios guardados de esta oferta y de la oferta de Marketplace publicada.
2. Si se accede a **Compare** (Comparar) desde una página concreta de la oferta, la predeterminado muestra solo los cambios que se han realizado en esa página.
    - Si desea comparar los cambios en todas las páginas, cambie la página en **Seleccione una página que comparar**.  
    - Si desea comparar los cambios realizados en la oferta en todas las páginas, seleccione **Todas las páginas**.

Al ser la predeterminada, **Compare** (Comparar) compara los nuevos cambios en la oferta de Marketplace activa.

Recuerde volver a publicar la oferta después de realizar actualizaciones para que los cambios surtan efecto.

### <a name="compare-changes-to-a-preview-offer"></a>Comparación de cambios en una oferta de versión preliminar

Si tiene cambios en versión preliminar que no están activos, puede comparar los cambios nuevos con la versión preliminar de la oferta de Marketplace.

Siga las instrucciones que se indican a continuación para comparar los cambios nuevos con la versión preliminar de la oferta de Marketplace:

1. Seleccione **Compare** (Comparar) en la barra de comandos de la página.
2. Seleccione la lista desplegable **Con** y cambie la opción de **Live offer** (Oferta en directo) a **Preview offer** (Oferta en versión preliminar).
3. La página **Compare** (Comparar) proporciona versiones en paralelo que muestran los cambios.

>[!NOTE]
>Si la oferta aún no está activa, pero la ha publicado en versión preliminar, no tiene la opción de compararla con una oferta activa.

Recuerde volver a publicar la oferta después de realizar actualizaciones para que los cambios surtan efecto.

## <a name="delete-a-draft-offer"></a>Eliminación de una oferta borrador

Puede eliminar una oferta borrador (una que no se ha publicado), seleccione **Eliminar borrador** en la página **Información general de la oferta**. Esta opción no estará disponible si ha publicado previamente la oferta.

Después de confirmar que desea eliminar la oferta borrador, la oferta ya no estará visible ni accesible en Centro de partners y se abrirá la página **Todas las ofertas**.

## <a name="delete-a-draft-plan"></a>Eliminación de una plan borrador

Para eliminar un plan que no se ha publicado, seleccione **Eliminar borrador** desde la página **Información general del plan**. Esta opción no estará disponible si ha publicado previamente la oferta.

Después de confirmar que desea eliminar el plan borrador, el plan ya no estará visible ni accesible en Centro de partners.

## <a name="cancel-publishing"></a>Cancelación de la publicación

Para cancelar una oferta con el estado **Publicación en curso**:

1. Seleccione el nombre de la oferta para abrir la página **Información general de la oferta**.
1. Seleccione **Cancelar publicación** en la esquina superior derecha de la página.
1. Confirme que desea detener la publicación de la oferta.

Si desea publicar la oferta más adelante, tendrá que volver a empezar el proceso de publicación.

> [!NOTE]
> La publicación de una oferta solo se puede detener si esta aún no ha llegado al paso de aprobación por parte del publicador. Después de seleccionar **Publicar** ya no tendrá la opción de cancelar la publicación.

## <a name="stop-selling-an-offer-or-plan"></a>Detención de la venta de una oferta o un plan

Son varios los motivos por los que podría decidir eliminar el catálogo de ofertas del marketplace comercial de Microsoft. La eliminación de ofertas garantiza que los nuevos clientes ya no puedan adquirir ni implementar la oferta, pero no afecta a los clientes existentes.

Para dejar de vender una oferta publicada, seleccione **Stop selling** (Dejar de vender) desde la página **Overview** (Información general) de la oferta.

Tras confirmar que quiere dejar de vender la oferta, al cabo de unas horas ya no estará visible en el marketplace comercial y ningún nuevo cliente podrá descargarla.

Para detener la venta de un plan, seleccione **Dejar de vender** desde la página **Información general del plan**. La opción de dejar de vender un plan solo está disponible si tiene más de un plan en la oferta. Puede elegir dejar de vender un plan sin que otros planes dentro de la oferta se vean afectados. Una vez que confirme que quiere dejar de vender el plan, debe volver a publicar la oferta para que se aplique el cambio. Después de que se vuelva a publicar la oferta, el plan dejará de estar visible en el marketplace y ningún cliente nuevo podrá descargarlo.

Los clientes que hayan adquirido anteriormente la oferta o el plan, podrán seguir usándolos. Aunque pueden volver a descargarlos, no recibirán actualizaciones si más adelante actualiza y vuelve a publicar la oferta o el plan.

Una vez completada la solicitud para dejar de vender la oferta o el plan, los seguirá viendo en el portal de marketplace comercial del Centro de partners con el estado **Not available** (No disponible).

Si decide volver a mostrar o vender esta oferta o este plan, siga las instrucciones para [actualizar una oferta existente](#update-an-existing-offer-in-the-commercial-marketplace). No olvide que después de realizar cualquier cambio tiene que volver a **publicar** la oferta o el plan.

## <a name="remove-offers-from-existing-customers"></a>Eliminación de ofertas de clientes existentes

Para eliminar ofertas de los clientes existentes, [presente una solicitud de soporte técnico](https://aka.ms/marketplacepublishersupport). En la lista de temas de soporte técnico, seleccione **Marketplace comercial** > **Eliminación de ofertas o aplicaciones** y envíe la solicitud. El equipo de soporte técnico le guiará en el proceso de eliminación de la oferta.

## <a name="sync-private-plan-audiences"></a>Sincronización de audiencias de plan privado

Si su oferta incluye uno o varios planes que están configurados para que solo estén disponibles a una audiencia privada restringida, puede actualizar solo la audiencia que tenga acceso a ese plan privado sin publicar otros cambios en la oferta. 

Para actualizar y sincronizar la audiencia privada de sus planes:

- Modifique la audiencia en uno o varios planes privados mediante el botón + **Agregar un ID** o **Importar clientes (csv)** y, a continuación, guarde los cambios.
- Seleccione **Sync private audience** (Sincronizar audiencia privada) desde la página **Información general del plan**.

**Sync private audience** (Sincronizar audiencia privada) publicará solo los cambios para sus audiencias privadas, sin publicar ninguna otra actualización que haya podido realizar en la oferta borrador.

## <a name="next-steps"></a>Pasos siguientes

- [Comprobación del estado de publicación de la oferta de marketplace comercial](./publishing-status.md)
