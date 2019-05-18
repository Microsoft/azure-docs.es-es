---
title: Información general y conceptos clave de Azure API Management | Microsoft Docs
description: Obtenga información acerca de las API, los productos, los roles, los grupos y otros conceptos clave de API Management.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 11/15/2017
ms.author: apimpm
ms.custom: mvc
ms.openlocfilehash: 3934dbb01c3b9f3219c5e649d97371850b8daadc
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65230629"
---
# <a name="about-api-management"></a>Acerca de API Management

API Management (APIM) es una manera de crear puertas de enlace de API coherentes y modernas para servicios de back-end existentes.

API Management ayuda a las organizaciones a publicar API para desarrolladores externos, asociados e internos para liberar el potencial de sus datos y servicios. Todas y cada una de las empresas pretenden extender sus operaciones como una plataforma digital creando nuevos canales, buscando nuevos clientes y estrechando la relación con los existentes. API Management proporciona las competencias esenciales para garantizar un programa de API de éxito mediante compromisos con desarrolladores, información detallada empresarial, análisis, seguridad y protección. Puede usar Azure API Management para tomar cualquier back-end e iniciar un programa de API completo basado en él.

Este artículo proporciona información general de escenarios comunes que implican a APIM.  También ofrece una breve descripción de los componentes principales del sistema de APIM. El artículo, a continuación, proporciona una descripción más detallada de cada uno.

## <a name="overview"></a>Información general

Para usar API Management, los administradores crean API. Cada API consta de una o varias operaciones y se puede agregar a uno o varios productos. Para usar una API, los desarrolladores se suscriben a un producto que contiene esa API y después pueden llamar a la operación de la API cumpliendo cualquier directiva de uso que pueda estar en vigor. Entre los escenarios habituales se incluyen los siguientes:

* **Protección de la infraestructura móvil** mediante el control de acceso con claves de API, la prevención de ataques de denegación de servicio mediante la limitación o el uso de directivas de seguridad avanzadas como la validación de tokens JWT.
* **Habilitación de los ecosistemas de asociados de fabricantes de software independiente** mediante la oferta de una incorporación rápida de asociados a través del portal para desarrolladores y la creación de una fachada de API para desacoplar desde implementaciones internas que no son adecuada para el consumo de los asociados.
* **Ejecución de un programa de API interno** mediante la oferta de una ubicación centralizada para la organización a fin de comunicar acerca de la disponibilidad y los últimos cambios en las API y el control de acceso en función de cuentas de organizaciones, todo ello basado en un canal seguro entre la puerta de enlace de la API y el back-end.

El sistema consta de los siguientes componentes:

* La **puerta de enlace de la API** es el extremo que:
  
  * Acepta llamadas de API y las enruta a los back-end.
  * Comprueba las claves de API, los tokens JWT, los certificados y otras credenciales.
  * Aplica cuotas de uso y límites de frecuencia.
  * Transforma la API sobre la marcha sin modificaciones de código.
  * Almacena en caché las respuestas de back-end donde se instalaron.
  * Registra los metadatos de llamada para fines de análisis.
* **Azure Portal** es la interfaz administrativa donde se configura el programa de API. Utilícelo para:
  
  * definir o importar el esquema de API
  * empaquetar las API en productos
  * establecer directivas, como cuotas o transformaciones, en las API
  * obtener información del análisis
  * administrar usuarios
* El **portal para desarrolladores** actúa como la presencia web principal para desarrolladores, donde estos pueden:
  
  * leer documentación de la API
  * probar una API a través de la consola interactiva
  * crear una cuenta y suscribirse para obtener claves de API
  * obtener acceso a análisis sobre su propio uso

Para más información, consulte las notas del producto en formato PDF [Cloud-based API Management: Harnessing the Power of APIs](https://j.mp/ms-apim-whitepaper) (API Management basado en la nube: aprovechamiento de la versatilidad de las API. Estas notas del producto introductorias sobre la administración de API por CITO Research cubren: 
 
 * Desafíos y requisitos comunes de API
 * Desacoplamiento de API y presentación de fachadas
 * Puesta en marcha de los desarrolladores rápidamente
 * Protección de acceso
 * Análisis y métricas
 * Obtención de control e información con una plataforma de API Management
 * Uso de soluciones de nube frente a locales
 * Azure API Management
 
## <a name="apis"></a>API y operaciones
Las API son el fundamento de una instancia del servicio Administración de API. Cada API representa un conjunto de operaciones disponibles para los desarrolladores. Cada API contiene una referencia a un servicio back-end que implementa la API y sus operaciones se asignan a las operaciones implementadas por dicho servicio. Las operaciones de API Management son altamente configurables, con control sobre asignación de direcciones URL, parámetros de consulta y ruta de acceso, contenidos de solicitudes y respuestas y almacenamiento en caché de respuestas de operaciones. En la API o en el ámbito de operación individual, también se pueden implementar directivas de límite de tasa, cuotas y restricción de direcciones IP.

Para más información, consulte [Creación de API][How to create APIs] e [Incorporación de operaciones a una API][How to add operations to an API].

## <a name="products"> </a> Productos
Los productos son la forma de presentar las API a los desarrolladores. Los productos en Administración de API tienen una o varias API y se configuran con un título, una descripción y términos de uso. Los productos pueden ser de tipo **Abierto** o **Protegido**. Para poder usar los productos protegidos es necesario suscribirse antes a ellos, mientras que los productos abiertos pueden usarse sin suscripción. Cuando un producto está preparado para que lo usen los desarrolladores, se puede publicar. Una vez publicado, los desarrolladores pueden verlo (y, en el caso de los productos protegidos, suscribirse a él). La aprobación de la suscripción se configura en el ámbito de producto y puede requerir la aprobación del administrador o aprobarse automáticamente.

Los grupos se usan para administrar la visibilidad de productos a los desarrolladores. Los productos conceden visibilidad a los grupos y los desarrolladores pueden ver los productos visibles a los grupos a los que pertenecen y suscribirse a ellos. 

## <a name="groups"></a> Grupos
Los grupos se usan para administrar la visibilidad de productos a los desarrolladores. API Management tiene los siguientes grupos invariables del sistema:

* **Administradores** : los administradores de la suscripción de Azure son miembros de este grupo. Los administradores controlan las instancias del servicio Administración de API y crean las API, las operaciones y los productos que usan los desarrolladores.
* **Desarrolladores** : los usuarios del portal para desarrolladores autenticados se incluyen en este grupo. Los desarrolladores son los clientes que compilan aplicaciones con sus API. Los desarrolladores, después de que se les concede acceso al portal para desarrolladores, crean aplicaciones que llaman a las operaciones de una API.
* **Invitados** : a este grupo pertenecen los usuarios del portal para desarrolladores no autenticados como, por ejemplo, clientes potenciales que visitan el portal para desarrolladores de una instancia de API Management. Se les concede determinado acceso de solo lectura, como por ejemplo la posibilidad de ver API pero no llamarlas.

Además de estos grupos del sistema, los administradores pueden crear grupos personalizados o [aprovechar los grupos externos en inquilinos de Azure Active Directory asociados](api-management-howto-aad.md). Los grupos personalizados y externos pueden usarse junto con grupos del sistema en la concesión a los desarrolladores de visibilidad y acceso a productos de la API. Por ejemplo, podría crear un grupo personalizado para los desarrolladores afiliados a una organización asociada específica y permitirles el acceso a las API a partir de un producto que contenga solo las API relevantes. Un usuario puede ser miembro de más de un grupo.

Para más información, consulte [Creación y uso de grupos][How to create and use groups].

## <a name="developers"> </a> Desarrolladores
Los desarrolladores representan las cuentas de usuario de una instancia del servicio API Management. Los desarrolladores pueden ser creados por administradores o invitados por estos y también pueden suscribirse desde el [Portal para desarrolladores][Developer portal]. Cada desarrollador es miembro de uno o varios grupos y se puede suscribir a los productos que conceden visibilidad a esos grupos.

Cuando los desarrolladores se suscriben a un producto, se les concede la clave principal y secundaria para dicho producto. Esta clave se usa cuando se realizan llamadas en las API del producto.

Para más información, consulte [Creación o invitación de desarrolladores][How to create or invite developers] y [Asociación de grupos a desarrolladores][How to associate groups with developers].

## <a name="policies"> </a> Directivas
Las directivas son una eficaz funcionalidad de API Management que permite a Azure Portal cambiar el comportamiento de la API a través de la configuración. Las directivas son una colección de declaraciones que se ejecutan secuencialmente en la solicitud o respuesta de una API. Entre las declaraciones más usadas se encuentran la conversión de formato de XML a JSON y la limitación de tasa de llamadas para restringir el número de llamadas entrantes de un desarrollador, pero también hay muchas otras directivas disponibles.

Las expresiones de directiva pueden utilizarse como valores de atributos o valores de texto en cualquiera de las directivas de API Management, a menos que la directiva especifique lo contrario. Algunas directivas como [Flujo de control](/azure/api-management/api-management-advanced-policies#choose) y [Establecer variable](/azure/api-management/api-management-advanced-policies#set-variable) se basan en expresiones de directiva. Para más información, consulte [Directivas avanzadas](/azure/api-management/api-management-advanced-policies#AdvancedPolicies) y [Expresiones de directiva](/azure/api-management/api-management-policy-expressions).


Para obtener una lista completa de las directivas de API Management, consulte [Referencia de la directiva][Policy reference]. Para más información acerca del uso y la configuración de directivas, consulte [Directivas de API Management][API Management policies]. Para ver un tutorial sobre la creación de un producto con directivas de cuota y límite de tasas, consulte [Creación y definición de configuraciones de productos avanzadas][How create and configure advanced product settings].


## <a name="developer-portal"> </a> Portal para desarrolladores
El portal para desarrolladores es el lugar en el que los desarrolladores pueden aprender acerca de sus API, ver operaciones y llamarlas y suscribirse a productos. Los clientes potenciales pueden visitar el portal para desarrolladores, ver API y operaciones y suscribirse. La dirección URL del portal para desarrolladores se encuentra en Azure Portal para la instancia del servicio API Management.

Puede personalizar el aspecto y apariencia del portal para desarrolladores agregando contenido personalizado, personalizando estilos e incorporando su toque diferenciador.

## <a name="api-management-and-the-api-economy"></a>API Management y la economía de la API

Para más información acerca de API Management, vea la siguiente presentación de la conferencia Microsoft Ignite 2017.

> [!VIDEO https://channel9.msdn.com/Events/Ignite/Microsoft-Ignite-Orlando-2017/BRK2186/player]
> 
> 

## <a name="next-steps"></a>Pasos siguientes

Complete la guía de inicio rápido siguiente y empiece a usar Azure API Management:

> [!div class="nextstepaction"]
> [Creación de una instancia de Azure API Management](get-started-create-service-instance.md)

[APIs and operations]: #apis
[Products]: #products
[Groups]: #groups
[Developers]: #developers
[Policies]: #policies
[Developer portal]: #developer-portal

[How to create APIs]: api-management-howto-create-apis.md
[How to add operations to an API]: api-management-howto-add-operations.md
[How to create and publish a product]: api-management-howto-add-products.md
[How to create and use groups]: api-management-howto-create-groups.md
[How to associate groups with developers]: api-management-howto-create-groups.md#associate-group-developer
[How create and configure advanced product settings]: transform-api.md
[How to create or invite developers]: api-management-howto-create-or-invite-developers.md
[Policy reference]: api-management-policy-reference.md
[API Management policies]: api-management-howto-policies.md
[Create an API Management service instance]: get-started-create-service-instance.md




