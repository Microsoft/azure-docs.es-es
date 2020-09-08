---
title: 'Preguntas más frecuentes sobre la transición desde Cloud Partner Portal al Centro de partners: marketplace comercial de Microsoft'
description: Respuestas a las preguntas más frecuentes sobre la transición de ofertas de Cloud Partner Portal al Centro de partners.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: 55b789363397945e01d02d76cb0e573660d18b67
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88607502"
---
# <a name="frequently-asked-questions-about-transitioning-from-the-cloud-partner-portal-to-partner-center"></a>Preguntas más frecuentes sobre la transición de Cloud Partner Portal al Centro de partners

Cloud Partner Portal se ha convertido en el Centro de partners. El Centro de partners ofrece una experiencia simplificada e integrada para publicar nuevas ofertas en [Microsoft AppSource](https://appsource.microsoft.com/) o [Azure Marketplace](https://azuremarketplace.microsoft.com/), y para administrar las ofertas existentes que se han migrado desde Cloud Partner Portal. Todas las funciones de Cloud Partner Portal están disponibles en el Centro de partners. En este artículo se abordan las preguntas más frecuentes sobre esto.

## <a name="what-does-the-transition-to-partner-center-mean-for-me"></a>¿Qué significa la transición al Centro de partners?

Puede seguir haciendo negocios en el Centro de partners:

| Área<img src="" width=200px> | Cambios |
| --- | --- |
| Cuenta | No es necesario crear una cuenta del Centro de partners; puede usar las credenciales de Cloud Partner Portal existentes para iniciar sesión en el Centro de partners, donde podrá administrar la cuenta, los usuarios, los permisos y la facturación. El contrato de publicación y la información de perfil de empresa se migran a la nueva cuenta del Centro de partners, junto con cualquier información de perfil de pago de cuenta, permisos y cuentas de usuario, y ofertas activas. Obtenga más información en [Administración de la cuenta de marketplace comercial en el Centro de partners](partner-center-portal/manage-account.md). |
| Experiencia de administración y publicación de ofertas | Los datos de la oferta se han migrado de Cloud Partner Portal al Centro de partners. Ahora accederá a las ofertas en el Centro de partners, que ofrece una mejor experiencia de usuario y una interfaz intuitiva. Aprenda a [actualizar una oferta existente en el marketplace comercial](partner-center-portal/update-existing-offer.md). |
| Disponibilidad de las ofertas en el marketplace comercial | Sin cambios. Si la oferta está activa en el marketplace comercial, lo seguirá estando. |
| Nuevas compras e implementaciones | Sin cambios. Los clientes pueden seguir comprando e implementando las ofertas sin interrupciones. |
| Pagos | Todas las compras e implementaciones se le seguirán pagando de la forma habitual. Obtenga más información sobre la [Recepción de pagos en el marketplace comercial](partner-center-portal/get-paid.md). |
| Integraciones de API con las [API de Cloud Partner Portal](cloud-partner-portal-api-overview.md) existentes | Todavía se admiten las API de Cloud Partner Portal existentes y las integraciones existentes siguen funcionando. Obtenga más información en [¿Se admitirán las API REST de Cloud Partner Portal?](#are-the-cloud-partner-portal-rest-apis-still-supported) |
| Análisis | Mediante el examen de los análisis en el Centro de partners puede seguir supervisando las ventas, evaluando el rendimiento y optimizando las ofertas en el marketplace comercial. Existen diferencias entre cómo se muestran los informes de análisis en CPP y en el Centro de partners. Por ejemplo, **Información para el vendedor** en CPP tiene una pestaña **Orders & Usage** (Pedidos y uso) que muestra los datos de las ofertas basadas o no basadas en el uso, mientras que, en el Centro de partners, la página **Pedidos** tiene una pestaña aparte para las ofertas de SaaS. Más información en [Acceso a los informes de análisis del marketplace comercial en el Centro de partners](partner-center-portal/analytics.md). |
|||

## <a name="do-i-need-to-create-a-new-account-to-manage-my-offers-in-partner-center"></a>¿Es necesario crear una cuenta para administrar las ofertas en el Centro de partners?

No, la cuenta actual se conservará. Esto significa que, si ya es un partner, puede usar las credenciales existentes de la cuenta de Cloud Partner Portal para iniciar sesión en el Centro de partners. No cree una cuenta, o todas las ofertas creadas en Cloud Partner Portal y que se hayan movido al Centro de partners no se asociarán a ella.

## <a name="what-pages-in-partner-center-correspond-to-pages-i-used-in-the-cloud-partner-portal"></a>¿Qué páginas del Centro de partners corresponden a las páginas que usaba en Cloud Partner Portal?

A continuación se muestran vínculos del Centro de partners de páginas que se suelen usar en Cloud Partner Portal. Si ha guardado como favoritos los vínculos de Cloud Partner Portal, le interesará actualizarlos.

| Página de Cloud Partner Portal <img src="" width=100px>| Vínculo de página de Cloud Partner Portal | Vínculo de página del Centro de partners |
| --- | --- | --- |
| Página Todas las ofertas | [https://cloudpartner.azure.com/#alloffers](https://cloudpartner.azure.com/#alloffers) | [https://partner.microsoft.com/dashboard/commercial-marketplace/overview](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) |
| Página Todos los publicadores | [https://cloudpartner.azure.com/#publishers](https://cloudpartner.azure.com/#publishers) | [https://partner.microsoft.com/dashboard/account/v3/publishers/list](https://partner.microsoft.com/dashboard/account/v3/publishers/list) |
| Perfil de anunciante | [https://cloudpartner.azure.com/#profile](https://cloudpartner.azure.com/#profile) | [https://partner.microsoft.com/dashboard/account/management](https://partner.microsoft.com/dashboard/account/management) |
| Página Usuarios | [https://cloudpartner.azure.com/#users](https://cloudpartner.azure.com/#users) | [https://partner.microsoft.com/dashboard/account/usermanagement](https://partner.microsoft.com/dashboard/account/usermanagement) |
| Página Historial | [https://cloudpartner.azure.com/#history](https://cloudpartner.azure.com/#history) | La característica de Historial todavía no se admite en el Centro de partners. |
| Panel de conclusiones | [https://cloudpartner.azure.com/#insights](https://cloudpartner.azure.com/#insights) | [https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary) |
| Informe de pagos | [https://cloudpartner.azure.com/#insights/payout](https://cloudpartner.azure.com/#insights/payout) | [https://partner.microsoft.com/dashboard/payouts/reports/incentivepayments](https://partner.microsoft.com/dashboard/payouts/reports/incentivepayments) |
|||

## <a name="payout-report-differences"></a>Diferencias en el informe de pago

Estas son las diferencias en el informe de pago entre Cloud Partner Portal (ya retirado) y el Centro de partners actual:

| Cloud Partner Portal | Centro de partners |
| --- | --- |
| **Vínculo**: https://cloudpartner.azure.com/ | **Vínculo**: https://partner.microsoft.com/dashboard/payouts/reports/transactionhistory y https://partner.microsoft.com/dashboard/payouts/reports/incentivepayments |
| **Navegación**: Informes de pago proporcionados en Insights Payout | **Navegación**: Informes de pago proporcionados en el Centro de Partners: icono de pago |
| **Ámbito**:<ul><li>Transacción por artículo de línea visible, para cobros en curso, realizados y pagados.</li><li>Informes: se muestran todos los artículos de línea una vez creado el pedido de compra, incluidos los cobros y la facturación en curso; también se muestran el estado de cobro y los artículos de línea que aún no son válidos para su pago.</li></ul> | **Ámbito**:<ul><li>Muestra los artículos de línea después de que se consideran ganancias válidas.</li><li>Los clientes pagan primero a Microsoft y entonces los ISV pueden ver que se inicia el informe de pago.</li><li>El informe de pago no muestra ni el cobro ni la facturación en curso.</li></ul> |
| **La transacción no está lista para el pago**: Facturación en curso | **La transacción no está lista para el pago**: Siguiente pago estimado: El estado del pago es sin procesar. |
| **Estado del pago**: n/d | **Estado del pago**:<ul><li>Sin procesar: La ganancia es válida para el pago.</li><li>Próximamente: La ganancia se enviará al editor en el siguiente pago mensual.</li><li>Enviado: El pago se ha enviado al banco.</li></ul> |
|||

## <a name="what-about-offers-i-published-in-the-cloud-partner-portal"></a>¿Qué ocurre con las ofertas publicadas en Cloud Partner Portal?

Las ofertas se han movido al Centro de partners y serán accesibles después de que inicie sesión en el Centro de partners, con la excepción de las ofertas Servicio administrado de Dynamics NAV y Cortana Intelligence. Si la oferta estaba activa en el marketplace comercial, lo seguirá estando y los clientes la podrán comprar e implementar sin interrupciones. Vea la siguiente pregunta **¿Qué ofertas se han movido al Centro de partners?** para obtener más información.

## <a name="what-offers-were-moved-to-partner-center"></a>¿Qué ofertas se han movido al Centro de partners?

Todos los tipos de oferta admitidos anteriormente en Cloud Partner Portal se admiten en el Centro de partners, con la excepción de Servicio administrado de Dynamics NAV y Cortana Intelligence.

En el caso de los tipos de oferta admitidos en el Centro de partners, todas las ofertas se han movido con independencia de su estado; también se han movido las ofertas borrador, las eliminadas de la lista y las de versión preliminar.

| Tipo de oferta <img src="" width=150px>| ¿Se ha movido al Centro de partners? <img src="" width=100px>| Pasos siguientes |
| --- | --- | --- |
| SaaS | Sí | Inicie sesión en el Centro de partners para crear ofertas y administrar las que se hayan creado en Cloud Partner Portal. Más información en [Creación de una oferta de SaaS en el marketplace comercial](partner-center-portal/create-new-saas-offer.md). |
| Máquina virtual | Sí | Inicie sesión en el Centro de partners para crear ofertas y administrar las que se hayan creado en Cloud Partner Portal. Más información en [Creación de una oferta de máquina virtual de Azure en Azure Marketplace](partner-center-portal/azure-vm-create-offer.md). |
| Aplicación de Azure | Sí | Inicie sesión en el Centro de partners para crear ofertas y administrar las que se hayan creado en Cloud Partner Portal. Más información en [Creación de una oferta de aplicación de Azure](partner-center-portal/create-new-azure-apps-offer.md). |
| Dynamics 365 Business Central | Sí | Inicie sesión en el Centro de partners para crear ofertas y administrar las que se hayan creado en Cloud Partner Portal. Más información en [Creación de una oferta de Dynamics 365 Business Central](partner-center-portal/create-new-business-central-offer.md). |
| Dynamics 365 for Customer Engagement y Power Apps | Sí | Inicie sesión en el Centro de partners para crear ofertas y administrar las que se hayan creado en Cloud Partner Portal. Más información en [Creación de una oferta de Dynamics 365 for Customer Engagement y PowerApps](partner-center-portal/create-new-customer-engagement-offer.md). |
| Dynamics 365 for Operations | Sí | Inicie sesión en el Centro de partners para crear ofertas y administrar las que se hayan creado en Cloud Partner Portal. Más información en [Creación de una oferta de Dynamics 365 for Operations](partner-center-portal/create-new-operations-offer.md). |
| Aplicación Power BI | Sí | Inicie sesión en el Centro de partners para crear ofertas y administrar las que se hayan creado en Cloud Partner Portal. Más información en [Creación de una aplicación de Power BI para AppSource](partner-center-portal/create-power-bi-app-offer.md). |
| Módulo IoT Edge | Sí | Inicie sesión en el Centro de partners para crear ofertas y administrar las que se hayan creado en Cloud Partner Portal. Más información en [Creación, configuración y publicación de una oferta de módulo de IoT Edge en Azure Marketplace](partner-center-portal/azure-iot-edge-module-creation.md). |
| Contenedor | Sí | Inicie sesión en el Centro de partners para crear ofertas y administrar las que se hayan creado en Cloud Partner Portal. Más información en [Creación de una oferta de contenedor de Azure](partner-center-portal/create-azure-container-offer.md). |
| Servicio de consultoría | Sí | Inicie sesión en el Centro de partners para crear ofertas y administrar las que se hayan creado en Cloud Partner Portal. Más información en [Creación de una oferta de servicios de consultoría](partner-center-portal/create-consulting-service-offer.md). |
| Servicio administrado | Sí | Inicie sesión en el Centro de partners para crear ofertas y administrar las que se hayan creado en Cloud Partner Portal. Más información en [Creación de una oferta de servicio administrado](partner-center-portal/create-new-managed-service-offer.md). |
| Servicio administrado de Dynamics NAV | No | Microsoft ha evolucionado el Servicio administrado de Dynamics NAV a [Dynamics 365 Business Central](https://docs.microsoft.com/dynamics365/business-central/), por lo que sus ofertas se han eliminado de la lista en AppSource. Los clientes ya no pueden detectar estas ofertas y no se han movido al Centro de partners. Para que las ofertas estén disponibles en AppSource, puede adaptarlas a ofertas de Dynamics 365 Business Central y enviarlas al [Centro de partners](https://partner.microsoft.com/). Más información en [Creación de una oferta de Dynamics 365 Business Central](partner-center-portal/create-new-business-central-offer.md). |
| Cortana Intelligence | No | Microsoft ha evolucionado la hoja de ruta del producto para Cortana Intelligence, por lo que sus ofertas activas se han eliminado de la lista en AppSource. Los clientes ya no pueden detectar estas ofertas y no se han movido al Centro de partners. Para que las ofertas estén disponibles en el marketplace comercial, adáptelas a ofertas de software como servicio (SaaS) y envíelas al [Centro de partners](https://partner.microsoft.com/). Más información en [Lista de comprobación de creación de ofertas de SaaS](partner-center-portal/offer-creation-checklist.md). |

## <a name="i-cant-find-my-cloud-partner-portal-offers-in-partner-center"></a>No encuentro mis ofertas de Cloud Partner Portal en el Centro de partners

Lo que ve en el Centro de partners depende de los programas en los que esté inscrito, las cuentas a las que pertenezca y los roles de usuario y permisos que se le hayan asignado. Hay muchos programas del Centro de partners disponibles y puede inscribirse en varios de ellos. También puede acceder a varias cuentas con las mismas credenciales de usuario.

Las ofertas que haya creado en Cloud Partner Portal están disponibles en el Centro de partners, en el programa **Marketplace comercial** y en la cuenta utilizada para crearlas. Para asegurarse de que está viendo el programa y la cuenta correctos, siga los pasos que se indican a continuación. Para obtener otras sugerencias de solución de problemas, vea [Administración de la cuenta del Centro de partners](https://docs.microsoft.com/partner-center/partner-center-account-setup).

### <a name="access-the-right-program-in-partner-center"></a>Acceso al programa correcto en el Centro de partners

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) con las mismas credenciales que usaba para hacerlo en Cloud Partner Portal. En el panel de navegación de la izquierda se muestran las opciones asociadas a los programas en los que está inscrito.

    Por ejemplo, imagine que tiene acceso a tres programas: MPN, Referencias y Marketplace comercial. Al iniciar sesión en el Centro de partners, verá estos tres programas en el panel de navegación.

2. Seleccione **Marketplace comercial** > **Información general** para acceder a las ofertas.

    Si no ve el programa Marketplace comercial en el panel de navegación de la izquierda, es posible que esté en la cuenta equivocada. Siga los pasos que se describen en la sección siguiente para acceder a la cuenta correcta.

    [![Captura de pantalla que muestra el menú Información general del Centro de partners](media/cpp-pc-faq/overview-menu.png "Se muestra el menú Información general del Centro de partners")](media/cpp-pc-faq/overview-menu.png#lightbox)

### <a name="access-the-right-account-in-partner-center"></a>Acceso a la cuenta correcta en el Centro de partners

Si forma parte de varias cuentas, en el Centro de partners verá un botón de selector de cuenta marcado con dos flechas en el menú de navegación de la izquierda. Seleccione el botón de selector de cuenta para ver una lista de todas las cuentas a las que pertenece. Seleccione cualquier cuenta de la lista para cambiar a ella y ver todos los programas e información correspondientes. Si no ve un botón de selector de cuenta en el menú de navegación, significa que solo es miembro de una cuenta.

[![Captura de pantalla que muestra el botón de selección de cuentas del Centro de partners](media/cpp-pc-faq/picker-button.png "Se muestra el botón de selector de cuentas del Centro de partners")](media/cpp-pc-faq/picker-button.png#lightbox)

## <a name="how-do-i-create-new-offers"></a>¿Cómo se crean las ofertas?

Para crear ofertas, acceda al programa Marketplace comercial en el [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace/overview). En la página Información general, seleccione **+ Nueva oferta**.

[![Captura de pantalla que muestra el menú Información general del Centro de partners](media/cpp-pc-faq/new-offer.png "Se muestra el menú Información general del Centro de partners")](media/cpp-pc-faq/new-offer.png#lightbox)

## <a name="i-cant-sign-in-and-need-to-open-a-support-ticket"></a>No puedo iniciar sesión y necesito abrir una incidencia de soporte técnico

Si no puede iniciar sesión en la cuenta, aquí puede abrir una [incidencia de soporte técnico](https://partner.microsoft.com/support/v2/?stage=1).

## <a name="where-are-instructions-for-using-partner-center"></a>¿Dónde están las instrucciones para usa el Centro de partners?

Vaya a la [documentación de marketplace comercial](index.yml) y, después, expanda la seción sobre el **portal de Marketplace comercial en el Centro de partners**. Para ver artículos de ayuda sobre la creación de ofertas en el Centro de partners, expanda **Creación de una nueva oferta**.

## <a name="what-are-the-publishing-and-offer-management-differences"></a>¿Cuáles son las diferencias de administración de publicación y ofertas?

Estas son algunas de las diferencias entre Cloud Partner Portal y el Centro de partners.

### <a name="modular-publishing-capabilities"></a>Funcionalidades de publicación modular

El Centro de partners proporciona una opción de publicación modular que le permite seleccionar los cambios que desea publicar en lugar de publicar siempre todas las actualizaciones a la vez. Por ejemplo, en la pantalla siguiente se muestra que los únicos cambios seleccionados para publicar son los cambios en las **propiedades** y la **descripción de la oferta**. Los cambios que realice en la página de vista previa no se publicarán.

[![Captura de pantalla que muestra la página de revisión y publicación del Centro de partners](media/cpp-pc-faq/review-page.png "Se muestra la página Revisar y publicar del Centro de partners")](media/cpp-pc-faq/review-page.png#lightbox)

Las actualizaciones que no publique se guardan como borradores. Siga usando la vista previa de la oferta para revisar su oferta antes de hacerla pública.

### <a name="enhanced-preview-options"></a>Opciones de vista previa mejoradas

El Centro de partners incluye una [característica de comparación](partner-center-portal/update-existing-offer.md#compare-changes-to-commercial-marketplace-offers) con opciones de filtrado mejoradas. Esto le ofrece la posibilidad de comparar con la versión preliminar y las versiones publicadas de la oferta.

[![Captura de pantalla que muestra la característica de comparación del Centro de partners](media/cpp-pc-faq/compare.png "Se muestra la característica Comparar del Centro de partners")](media/cpp-pc-faq/compare.png#lightbox)

### <a name="branding-and-navigation-changes"></a>Cambios de personalización de marca y navegación

Observará algunos cambios en la personalización de marca. Por ejemplo, las *SKU* aparecen como *Planes* en el Centro de partners:

[![Captura de pantalla que muestra la página Planes del Centro de partners](media/cpp-pc-faq/plans.png "Se muestra la página Planes del Centro de partners")](media/cpp-pc-faq/plans.png#lightbox)

Además, la información que antes se proporcionaba en las páginas **Marketplace** o **Detalles del escaparate electrónico** (servicio de consultoría, aplicación Power BI) de Cloud Partner Portal, ahora se recopila en la página **Descripción de la oferta** del Centro de partners:

[![Captura de pantalla que muestra la página de listado de ofertas del Centro de partners](media/cpp-pc-faq/offer-listing.png "Se muestra la página Descripción de la oferta del Centro de partners")](media/cpp-pc-faq/offer-listing.png#lightbox)

La información que antes se proporcionaba para las SKU en una única página de Cloud Partner Portal es posible que ahora se recopile en varias páginas del Centro de partners:

- Página de configuración del plan
- Página de descripción del plan
- Página de disponibilidad del plan
- Página de configuración técnica del plan, como se muestra aquí: ![![Captura de pantalla que muestra la página de configuración técnica del Centro de partners.](media/cpp-pc-faq/technical-configuration.png "Muestra la página de configuración técnica del Centro de partners")](media/cpp-pc-faq/technical-configuration.png#lightbox)

El identificador de la oferta ahora se muestra en la barra de navegación izquierda de la oferta:

![Se muestra la ubicación del id. de la oferta en el Centro de partners](media/cpp-pc-faq/offer-id.png)

### <a name="stop-selling-an-offer"></a>Detención de la venta de una oferta

Puede solicitar [dejar de vender una oferta](partner-center-portal/update-existing-offer.md#stop-selling-an-offer-or-plan) en Marketplace directamente desde el portal del Centro de partners. La opción está disponible en la página **Información general de la oferta** de la oferta.

[![Captura de pantalla que muestra la página para dejar de vender una oferta del Centro de partners](media/cpp-pc-faq/stop-sell.png "Se muestra la página del Centro de partners para dejar de vender una oferta")](media/cpp-pc-faq/stop-sell.png#lightbox)
<br><br>

## <a name="are-the-cloud-partner-portal-rest-apis-still-supported"></a>¿Se siguen admitiendo las API REST de Cloud Partner Portal?

Las API de Cloud Partner Portal se integran con el Centro de partners y seguirán funcionando. La transición al Centro de partners presenta pequeños cambios. Revise la tabla siguiente para asegurarse de que el código sigue funcionando en el Centro de partners.

| API <img src="" width=100px>| Descripción del cambio | Impacto |
| --- | --- | --- |
| POST Publish, GoLive, Cancel | En el caso de las ofertas migradas, el encabezado de respuesta tendrá un formato diferente, pero continuará funcionando de la misma manera, lo que denota una ruta de acceso relativa para recuperar el estado de la operación. | Al enviar cualquiera de las solicitudes POST correspondientes para una oferta, el encabezado de ubicación tendrá uno de los dos formatos, según el estado de migración de la oferta: <ul><li>Ofertas no migradas: `/api/operations/{PublisherId}${offerId}$2$preview?api-version=2017-10-31`</li><li>Ofertas migradas: `/api/publishers/{PublisherId}/offers/{offereId}/operations/408a4835-0000-1000-0000-000000000000?api-version=2017-10-31`</li></ul>|
| Operación GET | En el caso de las ofertas que anteriormente admitían un campo "Dirección de correo electrónico para notificaciones" en la respuesta, este campo quedará en desuso y ya no se devolverá para las ofertas migradas. | En el caso de las ofertas migradas, ya no se enviarán notificaciones a la lista de correos electrónicos especificados en las solicitudes. En su lugar, el servicio de API se alineará con el proceso de correo electrónico de notificaciones del Centro de partners para enviar correos electrónicos. En concreto, las notificaciones del progreso de la operación se enviarán a la dirección de correo electrónico establecida en la sección de información de contacto del vendedor de la configuración de la cuenta en el Centro de partners.<br><br>Asegúrese de que la dirección de correo electrónico establecida en la sección de información de contacto del vendedor en la [configuración de la cuenta](https://partner.microsoft.com/dashboard/account/management) en el Centro de partners es la correcta para recibir notificaciones. |
|||
