---
title: 'Terminología y preguntas más frecuentes de los análisis de marketplace comercial: Centro de partners'
description: Obtenga respuestas a las preguntas más frecuentes sobre el análisis de marketplace comercial en el Centro de partners. En este artículo se incluye un diccionario de datos para la terminología de análisis.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 02/17/2020
author: shganesh-dev
ms.author: shganesh
ms.openlocfilehash: f0f14bf24bd867344ec72c86a6fd517085b66d1f
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87317559"
---
# <a name="commercial-marketplace-analytics-terminology-and-common-questions"></a>Preguntas comunes y terminología de análisis de marketplace comercial

En este artículo se abordan las preguntas más frecuentes sobre los mensajes de análisis en el Centro de partners y también se proporciona un diccionario de terminología de análisis.

## <a name="common-questions"></a>Preguntas frecuentes

**No puedo ver mis datos de análisis en el Centro de partners. Cuando accedo a estas páginas, veo el mensaje siguiente. ¿Por qué ocurre esto?**

![Todavía no hay datos para las ofertas](./media/analytics-faq-no-data.png)

Por qué podría recibir este mensaje:

- Actualmente no existe ninguna adquisición para las ofertas publicadas en Marketplace. Esto puede significar que las ofertas están activas en Marketplace y obtienen vistas de los clientes en las páginas de presentación del producto, pero que los clientes aún no han realizado acciones para comprarlas e implementarlas.
- La publicación de la oferta podría estar en curso y no activa aún. Los clientes solo pueden adquirir ofertas en directo. Para comprobar el estado de las ofertas, consulte la información general en el [panel Analizar](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/summary). Para más información, consulte [Panel Resumen del análisis de marketplace comercial](./summary-dashboard.md).
- Sus ofertas pueden aparecer como **Ponerse en contacto conmigo**, que son ofertas solo de anuncio y no pueden ser adquiridas por los clientes en Marketplace. Aunque estas ofertas generan clientes potenciales y se comparten con usted, no se crean pedidos para estas ofertas, ya que no se pueden comprar. Para consultar el tipo de anuncio de la oferta, vaya a la página de configuración.

**Sé que tengo datos de análisis, pero aparece el mensaje siguiente:**

![Sin datos para el intervalo de fechas proporcionado](./media/analytics-faq-data-range.png)

Si recibe este mensaje, significa que tiene datos de análisis pero no hay datos para el intervalo de fechas seleccionado. Seleccione un intervalo de fechas diferente o un intervalo de fechas personalizado para ver los datos desde 2010. Para más información, consulte la sección Intervalo de fechas de [Panel Resumen del análisis de marketplace comercial](./summary-dashboard.md).

## <a name="dictionary-of-data-terms"></a>Diccionario de términos de datos

| Nombre del atributo | Informes | Definición|
|---|---|---|
| Azure License Type (Tipo de licencia de Azure) | Cliente, pedido | Tipo de contrato de licencia que los clientes utilizan para comprar Azure. También conocido como Canal. |
| Tipo de licencia de Azure: Proveedor de soluciones en la nube | Cliente, pedido | El cliente final adquiere Azure y su oferta de Marketplace a través de su proveedor de soluciones en la nube, que actúa como su revendedor.|
| Tipo de licencia de Azure: Enterprise | Cliente, pedido | El cliente final adquiere Azure y su oferta de Marketplace a través de un contrato Enterprise, firmado directamente con Microsoft.|
| Tipo de licencia de Azure: Enterprise a través de revendedor  | Cliente, pedido | El cliente final adquiere Azure y su oferta de Marketplace a través de un revendedor que le facilita el contrato Enterprise con Microsoft.|
| Tipo de licencia de Azure: Pago por uso| Cliente, pedido | El cliente final adquiere Azure y la oferta de Marketplace a través de un contrato de pago por uso firmado directamente con Microsoft.|
| Nombre de la instancia de nube| Pedido de| Microsoft Cloud donde se produjo la implementación de una VM.|
| Nombre de la instancia en la nube: Azure Global| Pedido de| Nube pública global de Microsoft.|
| Nombre de la instancia en la nube: Azure Government | Pedido de| Nubes de Microsoft específicas de la Administración Pública para una de las administraciones siguientes: China, Alemania o Estados Unidos de América.| |
| Ciudad del cliente| Customer| Nombre de ciudad proporcionado por el cliente. La ciudad podría ser diferente de la ciudad de la suscripción a Azure del cliente.|
| Customer Communication Language (Idioma de comunicación del cliente)  | Customer| Idioma preferido por el cliente para la comunicación.|
| Customer Company Name (Nombre de la empresa del cliente) | Cliente, pedido | Nombre de empresa proporcionado por el cliente. El nombre podría ser diferente de la ciudad de la suscripción a Azure del cliente.|
| País o región del cliente | Cliente, pedido | Nombre del país o región proporcionado por el cliente. Podría ser diferente del país o región de la suscripción a Azure del cliente.|
| Customer Email (Correo electrónico del cliente)| Customer| Dirección de correo electrónico proporcionada por el cliente final. El correo electrónico podría ser diferente de la dirección de correo electrónico de la suscripción a Azure del cliente.|
| Customer First Name (Nombre del cliente)| Customer| Nombre proporcionado por el cliente. El nombre podría ser diferente del nombre proporcionado en la suscripción a Azure del cliente.|
| Customer ID | Cliente, pedido | Identificador único asignado a un cliente. Un cliente puede no tener ninguna suscripción a Azure Marketplace o bien tener varias.|
| Customer Postal Code (Código postal del cliente)  | Customer| Código postal proporcionado por el cliente. El código podría ser diferente del código postal proporcionado en la suscripción a Azure del cliente.|
| Customer State (Estado del cliente)| Customer| Estado (dirección) proporcionado por el cliente. El estado podría ser diferente del estado proporcionado en la suscripción a Azure del cliente.|
| Date Acquired (Fecha de adquisición)| Customer| Primera fecha en que el cliente adquirió cualquier oferta publicada por usted.|
| Date Lost (Fecha de pérdida)| Customer| Última fecha en que el cliente canceló la última de todas las ofertas que había comprado anteriormente.|
| Cliente nuevo  | Pedido de| El valor identificará a un nuevo cliente que adquiere una o varias de sus ofertas por primera vez (o no). El valor es "Sí" si es el mismo mes natural de la "Fecha de adquisición". El valor es "No" si el cliente ha comprado cualquiera de las ofertas antes del mes natural notificado. |
| Versión preliminar de SKU| Pedido de| El valor le permitirá saber si ha etiquetado la SKU como "versión preliminar". El valor es "Sí" si la SKU se ha etiquetado en consecuencia y solo las suscripciones de Azure autorizadas por usted pueden implementar y usar esta imagen. El valor es "No" si la SKU no se ha identificado como "versión preliminar". |
| Participación de contacto promocional| Customer| El valor le permitirá saber si el cliente ha optado de manera proactiva por el contacto promocional de los anunciantes. En este momento, no presentamos la opción a los clientes, por lo que hemos indicado "No" de forma general. Una vez implementada esta característica, comenzaremos a actualizarla en consecuencia.|
| Marketplace License Type (Tipo de licencia de Marketplace)| Pedido de| Método de facturación de la oferta de Marketplace.|
| Tipo de licencia de Marketplace: Facturado a través de Azure| Pedido de| Microsoft es su agente para esta oferta de Marketplace y factura a los clientes en su nombre. (Tarjeta de crédito de pago por uso o factura de Enterprise)|
| Tipo de licencia de Marketplace: Traiga su propia licencia | Pedido de| La máquina virtual requiere una clave de licencia proporcionada por el cliente para la implementación. Microsoft no factura a los clientes por mostrar sus ofertas de esta manera a través de Marketplace.|
| Tipo de licencia de Marketplace: Gratuito| Pedido de| La oferta está configurada para ser gratis para todos los usuarios. Microsoft no factura a los clientes por su uso de esta oferta.|
| Tipo de licencia de Marketplace: Microsoft como revendedor  | Pedido de| Microsoft es su revendedor para esta oferta de Marketplace.|
| Identificador de la suscripción a Marketplace | Cliente, pedido | Identificador único asociado a la suscripción a Azure que el cliente usó para comprar su oferta de Marketplace. Antes era el GUID de la suscripción a Azure.|
| Nombre de la oferta  | Pedido de| Nombre de la oferta de Marketplace.|
| Tipo de oferta  | Pedido de| Tipo de la oferta de Microsoft Marketplace.|
| Tipo de oferta: Aplicación administrada  | Pedido de | Use el tipo de oferta Aplicación de Azure: aplicación administrada cuando se requieran las condiciones siguientes: Se implementa una solución basada en suscripciones para al cliente con una máquina virtual o una solución completa basada en IaaS. Usted o su cliente requieren que la solución la administre un asociado. |
| Tipo de oferta: Aplicación de Azure| Pedido de | Use el tipo de oferta Aplicación de Azure: plantilla de solución cuando la solución requiera automatización adicional de la implementación y la configuración más allá de una VM simple.|
| Tipo de oferta: Servicio de consultoría| Pedido de| Los servicios de consultoría de Azure Marketplace ayudan a conectar a los clientes con servicios que les permitan mejorar y extender el uso que hacen de Azure.|
| Tipo de oferta: Contenedor | Pedido de| Utilice el tipo de oferta Contenedor cuando la solución sea una imagen de contenedor de Docker aprovisionada como un servicio de contenedor de Azure basado en Kubernetes.|
| Tipo de oferta: Dynamics 365 Business Central| Pedido de| Use este tipo de oferta cuando la solución se integra con Dynamics 365 for Finance and Operations.|
| Tipo de oferta: Dynamics 365 for Customer Engagement | Pedido de| Use este tipo de oferta cuando la solución se integra con Dynamics 365 for Customer Engagement.|
| Tipo de oferta: Módulo IoT Edge | Pedido de| Los módulos de Azure IoT Edge son las unidades de cálculo más pequeñas que administra IoT Edge y pueden contener servicios de Microsoft (por ejemplo, Azure Stream Analytics), servicios de terceros o su propio código específico de la solución. |
| Tipo de oferta: Aplicación de Power BI | Pedido de| Utilice el tipo de oferta Aplicación de Power BI al implementar una aplicación integrada con Power BI.|
| Tipo de oferta: Aplicación SaaS| Pedido de| Use el tipo de oferta de aplicación SaaS para permitir que el cliente compre su solución técnica basada en SaaS como suscripción.||
| Tipo de oferta: Máquina virtual | Pedido de | Use el tipo de oferta de máquina virtual cuando implemente un dispositivo virtual para la suscripción asociada con el cliente.|
| Tipo de oferta: Extensión de Visual Studio Marketplace  | Pedido de| Tipo de oferta disponible previamente para desarrolladores de la extensión Azure DevOps. En el futuro, los desarrolladores de la extensión Azure DevOps podrán vender su extensión directamente a los clientes. Las ofertas de extensión se pueden configurar como de pago o con una versión de evaluación. |
| Order Cancel Date (Fecha de cancelación de pedido)| Pedido de| Fecha en que se canceló el pedido de Marketplace.|
| Identificador del pedido| Pedido de| Identificador único del pedido del cliente para el servicio de Marketplace. Las ofertas basadas en el uso de máquinas virtuales no están asociadas con un pedido.|
| Order Purchase Date (Fecha de compra del pedido)| Pedido de| Fecha en que se creó el pedido de Marketplace.|
| Order Status (Estado del pedido)| Pedido de| Estado de un pedido de Marketplace en el momento en que se actualizaron por última vez los datos.|
| Estado del pedido: Active  | Pedido de| El cliente ha realizado un pedido y no lo ha cancelado.|
| Estado del pedido: Cancelado | Pedido de| El cliente ha realizado un pedido y posteriormente lo ha cancelado.|
| Correo electrónico del proveedor| Customer| Dirección de correo electrónico del proveedor que participa en la relación entre Microsoft y el cliente final. Si el cliente es una empresa a través de un revendedor, será el revendedor. Si participa un proveedor de soluciones en la nube (CSP), será el CSP.|
| Nombre del proveedor| Customer| Nombre del proveedor que participa en la relación entre Microsoft y el cliente final. Si el cliente es una empresa a través de un revendedor, será el revendedor. Si participa un proveedor de soluciones en la nube (CSP), será el CSP.|
| SKU| Pedido de| Nombre de la SKU (ahora denominada plan) que se define durante la publicación. Una oferta puede tener muchas SKU (planes), pero una SKU solo puede estar asociada a una oferta.|
| Trial End Date (Fecha de fin de la prueba)| Pedido de| La fecha del período de prueba de este pedido va a finalizar o ha finalizado.|
|||

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre los informes de análisis disponibles en el marketplace comercial del Centro de partners, consulte [Análisis del marketplace comercial en el Centro de partners](./analytics.md).
- Para ver los gráficos, las tendencias y los valores de datos agregados que resuman la actividad de la oferta en el marketplace, consulte [Panel Resumen de los análisis de marketplace comercial](./summary-dashboard.md).
- Para información sobre los pedidos en un formato gráfico que se pueda descargar, consulte [Panel Pedidos de los análisis de marketplace comercial](./orders-dashboard.md).
- Para información sobre las métricas de uso y facturación de las ofertas de máquina virtual, consulte [Panel Uso de los análisis de marketplace comercial](./usage-dashboard.md).
- Para información detallada sobre los clientes, incluidas las tendencias de crecimiento, consulte [Panel Cliente de los análisis de marketplace comercial](./customer-dashboard.md).
- Para ver una lista de las solicitudes de descarga de los últimos 30 días, consulte [Panel Descargas de los análisis de marketplace comercial](./downloads-dashboard.md).
- Para tener una vista consolidada de comentarios de los clientes sobre las ofertas de Azure Marketplace y AppSource, consulte [Panel Calificaciones y opiniones de los análisis de marketplace comercial](./ratings-reviews.md).
