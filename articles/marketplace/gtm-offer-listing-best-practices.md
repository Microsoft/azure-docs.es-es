---
title: 'Procedimientos recomendados para la publicación de ofertas: marketplace comercial de Microsoft'
description: Obtenga información sobre los procedimientos recomendados sobre la publicación para la comercialización de sus ofertas de Microsoft AppSource y Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 07/06/2020
ms.openlocfilehash: b7bcdaf9e3935c01a0906ecfaa1570e4bc7caba9
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87798127"
---
# <a name="offer-listing-best-practices"></a>Procedimientos recomendados para la publicación de ofertas

En este artículo se ofrecen sugerencias para crear ofertas de marketplace comercial y participar en ellas. En las tablas siguientes se describen los procedimientos recomendados para completar la información de la oferta en el Centro de partners. Para obtener un análisis del rendimiento de las ofertas, vaya al [Panel de información de Marketplace](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/marketplaceinsights) en el Centro de partners. 

## <a name="storefront-offer-details"></a>Detalles de la oferta del escaparate

| Configuración del escaparate | Procedimiento recomendado |
|:--- |:--- |  
| Nombre de la oferta | Para las aplicaciones, proporcione un título claro que incluya palabras clave de búsqueda para ayudar a los clientes a detectar su oferta. <br> <br> En el caso de los servicios de consultoría, siga este formato: [nombre de la oferta: [duración] [tipo de oferta] (por ejemplo, Contoso: implementación en 2 semanas) <br> <br> Tenga en cuenta las directrices de la marca: siga las [Directrices de marca y marca comercial de Microsoft](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general.aspx), así como cualquier otra directriz específica de cada producto, cuando se haga referencia a las marcas comerciales de Microsoft y a nombres de software, productos y servicios de Microsoft. |
| Descripción de la oferta | Proporcione una descripción clara de la propuesta de valor de la oferta en las primeras oraciones.  Tenga en cuenta que estas oraciones se pueden usar en los resultados del motor de búsqueda. Los componentes principales de la propuesta de valor deben incluir lo siguiente: <ul> <li>Descripción del producto o de la solución. </li> <li> Rol de usuario que se beneficia del producto o de la solución. </li> <li> Necesidad o preocupación del cliente que aborda el producto o la solución. </li> </ul> <br> Cuando sea posible, use un vocabulario estándar del sector o palabras que se basen en ventajas.  No se base exclusivamente en las características y funcionalidades para vender su producto.  En su lugar, céntrese en el valor que proporciona. <br> <br> En el caso de los anuncios de servicios de consultoría, debe indicar claramente los servicios profesionales que proporciona. |

En las tablas de esta sección se ofrecen sugerencias para crear ofertas atractivas en Marketplace. Para escribir y configurar estos materiales de marketing, vaya al [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace/). 

## <a name="storefront-listing-details"></a>Detalles de la publicación de productos del escaparate

Las categorías y las industrias de un escaparate diferente serán aplicables a distintos tipos de oferta.

| Escaparate | Categorías <br>por Escaparate | Categorías <br>por Escaparate | Industrias <br> por AppSource |
| :------------------- |:----------------:|:------:|:-------------:|
| **Tipo de oferta**   |  **Azure Marketplace**  | **AppSource**  |
| Azure App | X | |
| Contenedor | X | |
| Servicios de consultoría | | | X |
| Dynamics 365 for Customer Engagement & Power Platform | | X | X |
| Dynamics 365 Finance & administración de cadenas de suministros | | X | X | 
| Dynamics 365 Business Central | | X | X |
| Módulos de IoT Edge | X | |
| Power BI | | X | X |
| SaaS | X | X | X |
| Máquina virtual de Azure |  X |    |

### <a name="categories"></a>Categorías

Microsoft AppSource y Azure Marketplace son dos escaparates que satisfacen requisitos únicos de los clientes. Azure Marketplace se dirige a los profesionales de TI y a los desarrolladores que crean soluciones en Azure.  Microsoft AppSource se dirige a los usuarios empresariales que buscan aplicaciones SaaS empresariales o del sector, complementos de Dynamics 365, complementos de Microsoft 365 y aplicaciones de Power Platform.

Las categorías y subcategorías se asignan a cada escaparate en función de la audiencia de destino. La oferta se publicará en Microsoft AppSource o Azure Marketplace en función del tipo de oferta, las capacidades de transacción de la oferta y la selección de categoría/subcategoría. 

Seleccione las categorías y subcategorías que mejor se alineen con su oferta y con la audiencia prevista. Puede seleccionar:

* Máximo de dos (2) categorías, incluidas una categoría principal y una secundaria (opcional).
* Máximo de dos (2) subcategorías para cada categoría principal o secundaria. Si no se selecciona ninguna subcategoría, solo se podrá detectar en la categoría seleccionada.

[!INCLUDE [categories and subcategories](./includes/categories.md)]

**IMPORTANTE: ofertas de SaaS y complementos de Microsoft 365**

Consulte la [guía por tipo de oferta](https://docs.microsoft.com/azure/marketplace/partner-center-portal/commercial-marketplace-overview) para obtener información específica sobre cómo las capacidades de transacción pueden afectar a la forma en que los clientes de marketplace pueden ver y comprar su oferta. En el caso de las ofertas de SaaS, la funcionalidad de transacción de la oferta, así como la selección de categoría, determinará el escaparate donde se publicará la oferta:

| Oferta de SaaS    | Oferta de SaaS   | Oferta de SaaS  | Oferta de SaaS   | Oferta de SaaS   | Oferta de SaaS   | Oferta de SaaS    | Escaparate aplicable | Escaparate aplicable |
|:-------------:|:---:|:--------:|:---------:|:--:|:--:|:---:|:---------------------:|:-------------:|
| Facturación de uso medido | Complementos de Microsoft 365 | Ponerse en contacto conmigo | Transacción (al menos 1 plan) | Plan solo privado | Plan solo público | Planes públicos y privados | AppSource | Azure Marketplace |
|  | X |  |  |  |  |  | X |  |
| X |  |  | X | X |  |  |  | X |
| X |  |  | X |  | X |  |  | X |
| X |  |  | X |  |  | X |  | X<sup>2</sup> |
|  |  |  | X | X |  |  |  | X |
|  |  |  | X |  | X |  | X<sup>1</sup> | X<sup>1</sup> |
|  |  |  | X |  |  | X | X<sup>1</sup> | X<sup>1,2</sup> |
|  |  | X |  |  |  |  | x<sup>1</sup> | X<sup>1</sup> | 

1. En función de la selección de categoría/subcategoría y del sector
2. Las ofertas con planes privados se publicarán en Azure Portal


### <a name="industries"></a>Industrias

La selección de sector solo se aplica a las ofertas publicadas en AppSource y a los servicios de consultoría publicados en Azure Marketplace.  Seleccione sectores o verticales si la oferta aborda las necesidades específicas del sector, con lo que se destacan las funcionalidades específicas del sector en la descripción de la oferta. Puede seleccionar hasta dos (2) sectores, así como dos (2) verticales por cada sector seleccionado.

>[!Note]
>En el caso de las ofertas de servicio de consultoría de Azure Marketplace, no hay ninguna vertical del sector.

| **Industrias** |  **Segmentos verticales** |
| :------------------- | :----------------|
| **Agricultura** | |
| **Arquitectura y construcción** | |
| **Automoción** | |
| **Distribución** | Venta al por mayor <br> Envío de paquetes |  
| **Education** | Educación superior <br> Educación primaria y secundaria / K-12 <br> Bibliotecas y museos |
| **Servicios financieros** | Banca y mercados de capital <br> Seguros | 
| **Gobierno** |  Defensa e inteligencia <br> Gobierno civil <br> Justicia y seguridad pública |
| **Atención sanitaria** | Responsable de los pagos de mantenimiento <br> Proveedor de asistencia sanitaria <br> Productos farmacéuticos | 
| **Hostelería y viajes** | Viajes y transporte <br> Hoteles y ocio <br> Servicio de restaurantes y hostelería | 
| **Fabricación y recursos** | Química y agroquímica <br> Fabricación discreta <br> Sector energético | 
| **Elementos multimedia y comunicaciones** | Elementos multimedia y entretenimiento <br> Telecomunicaciones | 
| **Otras industrias del sector público** | Silvicultura y pesca <br> Sin ánimo de lucro | 
| **Servicios profesionales** | Servicios profesionales de partner <br> Información legal | 
| **Bienes inmuebles** | |

Sector solo para Microsoft AppSource:

| **Sector** |  **Segmentos verticales** |
| :------------------- | :----------------|
| **Minoristas y bienes de consumo** | Minoristas <br> Bienes de consumo |

### <a name="applicable-products"></a>Productos aplicables

Seleccione los productos aplicables con los que funciona la aplicación para que la oferta se muestre con los productos seleccionados en AppSource.

### <a name="search-keywords"></a>Palabras clave de búsqueda

Pueden ayudar a los clientes a encontrar su oferta cuando realicen búsquedas Identifique las principales palabras clave de búsqueda de su oferta, agréguelas en el resumen de la oferta y la descripción, así como en la sección de palabras claves de los detalles de la oferta.

## <a name="storefront-marketing-details"></a>Detalles de marketing del escaparate
| Configuración del escaparate | Procedimiento recomendado |
|:--- |:--- |  
| Logotipo de la oferta (formato PNG, entre 216 × 216 y 350 x 350 píxeles): página de detalles de la aplicación | Diseñe y optimice su logotipo para un medio digital:<br>Cargue el logotipo en formato PNG para la página de publicación de detalles de la aplicación de la oferta. El Centro de partners cambiará su tamaño a los tamaños de logotipo necesarios. |
| Logotipo de la oferta (formato .png, 48 x 48 píxeles): página de búsqueda | El Centro de partners generará este logotipo a partir del logotipo grande que ha cargado. Opcionalmente, puede reemplazarlo por una imagen diferente. |
| Documentos de "Más información" | Incluya recursos auxiliares de ventas y marketing en "Más información", como:<ul><li>notas del producto</li><li> folletos</li><li>listas de comprobación,</li><li> presentaciones de PowerPoint</li></ul><br>Guarde todos los archivos en formato PDF. El objetivo debe ser informar a los clientes acerca del producto, no venderlo.<br><br>Agregue un vínculo a la página principal de la aplicación en todos los documentos e incluya parámetros de URL para ayudarlo a realizar un seguimiento de los visitantes y las evaluaciones. |
| Videos: solo AppSource, servicios de consultoría y ofertas de SaaS | Los vídeos más atractivos comunican el valor de la oferta en forma de narración:<ul> <li> Asegúrese de que el héroe de la historia es el cliente, no su negocio. </li> <li> El vídeo debe abordar los objetivos y retos más importantes del cliente de destino. </li> <li> Longitud recomendada: de 60 a 90 segundos.</li> <li> Incorpore palabras clave de búsqueda que usan el nombre de los vídeos. </li> <li> Considere la posibilidad de agregar otros vídeos como, por ejemplo, de instrucciones, de introducción o de testimonios de clientes. </li> </ul> |
| Capturas de pantalla (1280&nbsp;&times;&nbsp;720) | Agregue hasta cinco capturas de pantalla:<br>Incorpore palabras clave de búsqueda en los nombres de archivos. |

## <a name="link-to-your-offer-page-from-your-website"></a>Vinculación a la página de la oferta desde su sitio web

Al vincular el distintivo de AppSource o de Marketplace del sitio web a la publicación en Marketplace, puede incluir los siguientes parámetros de consulta al final de la dirección URL para obtener análisis e informes detallados:
* **src**: incluya el origen desde donde se enruta el tráfico a AppSource; por ejemplo, el sitio web, LinkedIn o Facebook.
* **mktcmpid**: el identificador de la campaña de marketing, que puede contener hasta 16 caracteres en cualquier combinación de letras, números, guiones bajos y guiones (por ejemplo, *blogpost_12*).

La siguiente dirección URL de ejemplo contiene los dos parámetros de consulta anteriores: `https://appsource.microsoft.com/product/dynamics-365/mscrm.04931187-431c-415d-8777-f7f482ba8095?src=website&mktcmpid=blogpost_12`

Al agregar estos parámetros a la dirección URL de AppSource, puede revisar la eficacia de la campaña en el panel de análisis del [Centro de partners](https://partner.microsoft.com/dashboard/commercial-marketplace/).

## <a name="next-steps"></a>Pasos siguientes

Más información acerca de las [ventajas de Marketplace comercial](./gtm-your-marketplace-benefits.md).

Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership) para crear y configurar la oferta.

