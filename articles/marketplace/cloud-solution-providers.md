---
title: 'Proveedor de soluciones en la nube: Marketplace comercial de Microsoft'
description: Obtenga información sobre cómo vender sus ofertas a través del canal de partners del programa Proveedor de soluciones en la nube (CSP) de Microsoft en el marketplace comercial.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: keferna
ms.author: keferna
ms.date: 07/14/2020
ms.openlocfilehash: 51f4c9a5748896eff9e9f3ae9719b2e7e7cdd4e7
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88607484"
---
# <a name="cloud-solution-provider-program"></a>Programa Proveedor de soluciones en la nube

En este artículo se explica cómo configurar una oferta para que esté disponible en el programa Proveedor de soluciones en la nube (CSP). Además de publicar sus ofertas a través de [tiendas en línea del marketplace comercial](comparing-appsource-azure-marketplace.md), también puede vender a través del programa CSP para llegar a millones de clientes cualificados de Microsoft a los que el programa presta servicios.

Puede configurar ofertas nuevas o existentes para su disponibilidad en el programa CSP en función de la participación, lo que permite que los partners de CSP vendan sus productos y creen conjuntos de soluciones para los clientes.

Los publicadores son los responsables de ofrecer soporte break-fix a los clientes finales y de proporcionar un mecanismo para que los asociados del programa CSP y los clientes se pongan en contacto con usted para obtener soporte técnico. Un procedimiento recomendado es ofrecer a los asociados del programa CSP documentación de usuario, aprendizaje y notificaciones sobre el mantenimiento o la interrupción del servicio (según corresponda), de modo que los asociados del programa CSP estén equipados para responder a solicitudes de soporte técnico de nivel 1 por parte de los clientes.  

Las ofertas siguientes son aptas para ser vendidas por parte de los asociados del programa CSP:

- Ofertas de transacción de software como servicio (SaaS)
- Máquinas virtuales (VM)
- Plantillas de solución
- Aplicaciones administradas

> [!NOTE]
> De manera predeterminada, los planes de VM de Contenedores y Traiga su propia licencia (BYOL) son aptas para ser vendidas por parte de los asociados del programa CSP.

## <a name="how-to-configure-an-offer"></a>Cómo configurar una oferta

Configure el valor de participación en el programa CSP al crear la oferta en el Centro de partners. [Más información sobre los cambios en la experiencia del publicador](https://www.microsoftpartnercommunity.com/t5/Azure-Marketplace-and-AppSource/Cloud-Marketplace-In-Partner-Center/m-p/9738#M293).

### <a name="partner-center-opt-in"></a>Participación en el Centro de partners

La experiencia de participación se encuentra en el módulo Audiencia de revendedores de CSP:

![Audiencia de revendedores de CSP](media/marketplace-publishers-guide/csp-reseller-audience.png)

Elija una de las tres opciones:

1. Cualquier partner en el programa CSP.
2. Partners específicos en el programa CSP que seleccione.
3. No hay partners en el programa CSP.

#### <a name="option-1-any-partner-in-the-csp-program"></a>Opción 1: cualquier asociado del programa CSP

![cualquier asociado del programa CSP](media/marketplace-publishers-guide/csp-reseller-option-one.png)

 Al elegir esta opción, todos los asociados del programa CSP pueden optar por revender su oferta a sus clientes.

#### <a name="option-2-specific-partners-in-the-csp-program-i-select"></a>Opción 2: asociados específicos del programa CSP que selecciono

![asociados específicos del programa CSP que selecciono](media/marketplace-publishers-guide/csp-reseller-option-two.png)

Al elegir esta opción, autoriza a los asociados del programa CSP que pueden optar por revender su oferta.

Para autorizar a los asociados, haga clic en **Select CSP Partners** (Seleccionar asociados de CSP) y aparecerá un menú donde podrá buscar por el nombre del asociado o el identificador de inquilino de Azure Active Directory (AAD).

![Menú para seleccionar el CSP](media/marketplace-publishers-guide/csp-pop-up-module.png)

Puede aplicar filtros de búsqueda, como **País**, **Competencia** o **Aptitud**.

![Filtros de país o región, competencia y aptitud para la búsqueda de partners](media/marketplace-publishers-guide/csp-add-resellers.png)

Una vez que elija la lista de asociados, seleccione **Agregar**.

![Lista de ejemplo de los asociados autorizados del programa CSP](media/marketplace-publishers-guide/csp-add-resellers-details.png)

En la página de audiencia de revendedores de CSP aparece una tabla que muestra la lista de asociados que seleccionó.

![Tabla con la lista de asociados en la página de audiencia de revendedores de CSP](media/marketplace-publishers-guide/csp-option-two-add-reseller.png)

Seleccione **Guardar borrador** para registrar los cambios.

Si esta oferta no está publicada, deberá publicar su oferta para que esté disponible para los asociados seleccionados.

>[!NOTE]
>Si autoriza a un asociado del programa CSP en una región determinada, este puede vender la oferta a cualquier cliente que pertenezca a esa región en particular. Consulte [Mercados y monedas regionales del programa Proveedor de soluciones en la nube](https://docs.microsoft.com/partner-center/regional-authorization-overview) para más información sobre cómo se clasifican en regiones las ofertas del CSP.

Si está actualizando la lista de CSP de una oferta ya publicada, agregue los asociados adicionales y seleccione **Sync CSP audience** (Sincronizar audiencia de CSP).

Si tiene una oferta que ya tiene una lista de asociados autorizados y quiere usar la misma lista para otra oferta, use **Import/Export**. Vaya a la oferta que tiene la lista de CSP y seleccione **Export CSPs** (Exportar CSP). La función desarrolla un archivo .csv que se puede importar a otra oferta.

#### <a name="option-3-no-partners-in-the-csp-program"></a>Opción 3: ningún asociado del programa CSP

![ningún asociado del programa CSP](media/marketplace-publishers-guide/csp-reseller-option-three.png)

Si elige esta opción, excluye su oferta del programa CSP. Puede cambiar esta selección en cualquier momento.

## <a name="deauthorize-partners-in-the-csp-program"></a>Eliminación de la autorización de los asociados del programa CSP

Si autorizó a un asociado del programa CSP y ese asociado ya revendió el producto a sus clientes, no podrá quitarle la autorización.

Si hay un asociado del programa CSP que todavía no vende su producto a los clientes y quiere quitar el CSP una vez que se publique la oferta, siga estas instrucciones:

1. Vaya a la [página de solicitud de soporte técnico](https://aka.ms/marketplacepublishersupport). Los primeros menús desplegables se rellenan automáticamente.

   > [!NOTE]
   > No cambie las selecciones de menú desplegable rellenadas previamente.

2. En **Select the product version** (Seleccionar la versión del producto), seleccione **Live offer management** (Administración de ofertas en directo).
3. En **Select a category that best describe the issue** (Seleccionar la categoría que mejor describa la incidencia), elija la categoría que hace referencia a su oferta.
4. En **Select a problem that best describes the issue** (Seleccionar un problema que mejor describa la incidencia), seleccione **Update existing offer** (Actualizar la oferta existente).
5. Seleccione **Siguiente** para ir a la **página de detalles de la incidencia** para escribir más detalles sobre el problema.
6. Use **Deauthorize CSP** (Quitar autorización de CSP) como el título de la incidencia y rellene el resto de las secciones obligatorias.

## <a name="navigate-between-options"></a>Navegación entre las opciones

Use esta sección para navegar entre las tres opciones de revendedor de CSP.

### <a name="navigate-from-option-one-any-partner-in-the-csp-program"></a>Navegación desde la opción 1: cualquier asociado del programa CSP

Si la oferta es actualmente **Opción 1: cualquier asociado del programa CSP** y quisiera navegar a cualquiera de las otras dos opciones, use estas instrucciones para crear una solicitud:

1. Vaya a la [página de solicitud de soporte técnico](https://aka.ms/marketplacepublishersupport). Los primeros menús desplegables se rellenan automáticamente.

   > [!NOTE]
   > No cambie las selecciones de menú desplegable rellenadas previamente.

2. En **Select the product version** (Seleccionar la versión del producto), seleccione **Live offer management** (Administración de ofertas en directo).
3. En **Select a category that best describe the issue** (Seleccionar la categoría que mejor describa la incidencia), elija la categoría que hace referencia a su oferta.
4. En **Select a problem that best describes the issue** (Seleccionar un problema que mejor describa la incidencia), seleccione **Update existing offer** (Actualizar la oferta existente).
5. Seleccione **Siguiente** para ir a la **página de detalles de la incidencia** para escribir más detalles sobre el problema.
6. Use **Deauthorize CSP** (Quitar autorización de CSP) como el título de la incidencia y rellene el resto de las secciones obligatorias.

> [!NOTE]
> Si intenta navegar a la opción 2 y un asociado del programa CSP ya revendió la oferta a sus clientes, ese asociado ya está, de manera predeterminada, en la lista de asociados autorizados.  

### <a name="navigate-from-option-two-specific-partners-in-the-csp-program-i-select"></a>Navegación desde la opción 2: asociados específicos del programa CSP que selecciono

Si la oferta es actualmente **Opción 2: asociados específicos del programa CSP que selecciono** y quisiera navegar a la **Opción 1: cualquier asociado del programa CSP**, use estas instrucciones para crea una solicitud:

1. Vaya a la [página de solicitud de soporte técnico](https://aka.ms/marketplacepublishersupport). Los primeros menús desplegables se rellenan automáticamente.

   > [!NOTE]
   > No cambie las selecciones de menú desplegable rellenadas previamente.

2. En **Select the product version** (Seleccionar la versión del producto), seleccione **Live offer management** (Administración de ofertas en directo).
3. En **Select a category that best describe the issue** (Seleccionar la categoría que mejor describa la incidencia), elija la categoría que hace referencia a su oferta.
4. En **Select a problem that best describes the issue** (Seleccionar un problema que mejor describa la incidencia), seleccione **Update existing offer** (Actualizar la oferta existente).
5. Seleccione **Siguiente** para ir a la **página de detalles de la incidencia** para escribir más detalles sobre el problema.
6. Use **Deauthorize CSP** (Quitar autorización de CSP) como el título de la incidencia y rellene el resto de las secciones obligatorias.

 Si la oferta es actualmente **Opción 2: asociados específicos en el programa CSP que selecciono** y quisiera navegar a la **Opción 3: ningún asociado del programa CSP**, solo podrá navegar a esa opción si los asociados del programa CSP a los que autorizó anteriormente todavía no revenden la oferta a los clientes finales. Use estas instrucciones para crear una solicitud:

1. Vaya a la [página de solicitud de soporte técnico](https://aka.ms/marketplacepublishersupport). Los primeros menús desplegables se rellenan automáticamente.

   > [!NOTE]
   > No cambie las selecciones de menú desplegable rellenadas previamente.

2. En **Select the product version** (Seleccionar la versión del producto), seleccione **Live offer management** (Administración de ofertas en directo).
3. En **Select a category that best describe the issue** (Seleccionar la categoría que mejor describa la incidencia), elija la categoría que hace referencia a su oferta.
4. En **Select a problem that best describes the issue** (Seleccionar un problema que mejor describa la incidencia), seleccione **Update existing offer** (Actualizar la oferta existente).
5. Seleccione **Siguiente** para ir a la **página de detalles de la incidencia** para escribir más detalles sobre el problema.
6. Use **Deauthorize CSP** (Quitar autorización de CSP) como el título de la incidencia y rellene el resto de las secciones obligatorias.

### <a name="navigate-from-option-3-no-partners-in-the-csp-program"></a>Navegación desde la opción 3: ningún asociado del programa CSP

Si la oferta es actualmente **Opción 3: ningún asociado del programa CSP**, puede navegar a cualquiera de las otras opciones en cualquier momento.

## <a name="sharing-sales-and-support-materials-with-partners-in-the-csp-program"></a>Uso compartido de materiales de ventas y soporte técnico con asociados en el programa CSP

Para ayudar a que los partners del programa Proveedor de soluciones en la nube representen la oferta e interactúen con su organización de la forma más eficaz posible, debe enviar materiales de soporte técnico y ventas para que estén a disposición de los revendedores. Estos recursos no se expondrán a los clientes en las tiendas en línea del marketplace.

### <a name="partner-center-csp-channel"></a>Canal de CSP del Centro de partners

Si ha participado en el canal de CSP del Centro de partners, los publicadores deben escribir una dirección URL que hospede los materiales de marketing pertinentes y la información de contacto del canal en el canal de CSP en el módulo de lista de ofertas:

![Información sobre documentación y material adjunto de CSP del Centro de partners](media/marketplace-publishers-guide/pc-csp-channel.png)

## <a name="next-steps"></a>Pasos siguientes

Visite la [Guía de publicación de Azure Marketplace y AppSource](marketplace-publishers-guide.md).

Para más información sobre los servicios GTM de Marketplace, consulte [Servicios de comercialización](https://partner.microsoft.com/reach-customers/gtm).

Inicie sesión en el [Centro de Partners](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership) para crear y configurar la oferta.
