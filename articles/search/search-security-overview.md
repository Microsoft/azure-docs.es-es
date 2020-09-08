---
title: Introducción a la seguridad
titleSuffix: Azure Cognitive Search
description: Azure Cognitive Search es compatible con SOC 2, HIPAA y otras certificaciones. Conexión y cifrado de datos, autenticación y acceso a identidades mediante identificadores de seguridad de usuarios y grupos en expresiones de filtro.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 08/01/2020
ms.custom: references_regions
ms.openlocfilehash: c9f0f496bfdb31e0c7cb45a07c87ea238d031e34
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88928775"
---
# <a name="security-in-azure-cognitive-search---overview"></a>Seguridad en Azure Cognitive Search: información general

En este artículo se describen las principales características de seguridad de Azure Cognitive Search que pueden proteger el contenido y las operaciones.

+ En la capa de almacenamiento, el cifrado en reposo está integrado en todo el contenido administrado por el servicio guardado en el disco, incluidos los índices, los mapas de sinónimos y las definiciones de indizadores, orígenes de datos y conjuntos de aptitudes. Azure Cognitive Search también admite la incorporación de claves administradas por el cliente (CMK) para el cifrado complementario del contenido indizado. En el caso de los servicios creados después del 1 de agosto de 2020, el cifrado de CMK se extiende a los datos de los discos temporales, para el cifrado doble completo del contenido indizado.

+ La seguridad de entrada protege el punto de conexión del servicio de búsqueda en niveles crecientes de seguridad: desde las claves de API de la solicitud hasta las reglas de entrada del firewall o los puntos de conexión privados que protegen totalmente el servicio de la red pública de Internet.

+ La seguridad de salida se aplica a los indexadores que extraen contenido de orígenes externos. En el caso de las solicitudes salientes, configure una identidad administrada para que busque en un servicio de confianza al acceder a los datos de Azure Storage, Azure SQL, Cosmos DB u otros orígenes de datos de Azure. Una identidad administrada es un sustituto de las credenciales o las claves de acceso en la conexión. La seguridad de salida no se trata en este artículo. Para obtener más información acerca de esta capacidad, vea [Configuración de una conexión de indexador a un origen de datos mediante una identidad administrada](search-howto-managed-identities-data-sources.md).

Vea este vídeo de resumen para obtener información general sobre la arquitectura de seguridad y cada categoría de características.

> [!VIDEO https://channel9.msdn.com/Shows/AI-Show/Azure-Cognitive-Search-Whats-new-in-security/player]

<a name="encryption"></a>

## <a name="encrypted-transmissions-and-storage"></a>Transmisiones y almacenamiento cifrados

En Azure Cognitive Search, el cifrado empieza por las conexiones y las transmisiones y se extiende al contenido almacenado en el disco. En el caso de los servicios de búsqueda en la red pública de Internet, Azure Cognitive Search escucha en el puerto HTTPS 443. Todas las conexiones de cliente a servicio usan el cifrado TLS 1.2. Las versiones anteriores (1.0 o 1.1) no se admiten.

En el caso de los datos que administra internamente el servicio de búsqueda, en la tabla siguiente se describen los [modelos de cifrado de datos](../security/fundamentals/encryption-models.md). Algunas características, como el almacén de conocimiento, el enriquecimiento incremental y la indización basada en indizador, leen o escriben en estructuras de datos de otros servicios de Azure. Esos servicios tienen sus propios niveles de compatibilidad de cifrado independientes de Azure Cognitive Search.

| Modelo | Claves&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Requisitos&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Restricciones | Se aplica a |
|------------------|-------|-------------|--------------|------------|
| Cifrado del servidor | Claves administradas por Microsoft | Ninguno (integrados) | Ninguna, disponible en todos los niveles, en todas las regiones, para el contenido creado después del 24 de enero de 2018. | Contenido (índices y mapas de sinónimos) y definiciones (indizadores, orígenes de datos, conjuntos de aptitudes) |
| Cifrado del servidor | Claves administradas por el cliente | Azure Key Vault | Disponible en niveles facturables, en todas las regiones, para el contenido creado después de enero de 2019. | Contenido (índices y mapas de sinónimos) en discos de datos |
| Cifrado doble del servidor | Claves administradas por el cliente | Azure Key Vault | Disponible en niveles facturables, en regiones seleccionadas, en servicios de búsqueda después del 1 de agosto de 2020. | Contenido (índices y mapas de sinónimos) en discos de datos y discos temporales |

### <a name="service-managed-keys"></a>Claves administradas por el servicio

El cifrado administrado por el servicio es una operación interna de Microsoft que se basa en [Azure Storage Service Encryption](../storage/common/storage-service-encryption.md) con [cifrado AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) de 256 bits. Se produce automáticamente en todas las indizaciones, incluidas las actualizaciones incrementales a índices que no están totalmente cifrados (creados antes de enero de 2018).

### <a name="customer-managed-keys-cmk"></a>Claves administradas por el cliente (CMK)

Las claves administradas por el cliente requieren un servicio facturable adicional, Azure Key Vault, que puede estar en otra región, pero en la misma suscripción que Azure Cognitive Search. Al habilitar el cifrado de CMK aumenta el tamaño de los índices y empeora el rendimiento de las consultas. Según las observaciones hechas hasta la fecha, puede esperar un aumento del 30 al 60% en los tiempos de consultas, aunque el rendimiento real variará según la definición del índice y los tipos de consultas. Debido a este impacto en el rendimiento, se recomienda habilitar esta característica solo en los índices que realmente la necesitan. Para obtener más información, vea [Configuración de claves administradas por el cliente para el cifrado de datos en Azure Cognitive Search](search-security-manage-encryption-keys.md).

<a name="double-encryption"></a>

### <a name="double-encryption"></a>Cifrado doble

En Azure Cognitive Search, el cifrado doble es una extensión de CMK. Se entiende que se trata de un cifrado doble (una vez mediante CMK y otra mediante claves administradas por el servicio) y de ámbito general, que engloba el almacenamiento a largo plazo que se escribe en un disco de datos y el almacenamiento a corto plazo escrito en discos temporales. La diferencia entre CMK antes del 1 de agosto de 2020 y después, y lo que convierte a CMK en una característica de cifrado doble en Azure Cognitive Search, es el cifrado adicional de datos en reposo en discos temporales.

El cifrado doble está disponible actualmente en los nuevos servicios creados en estas regiones después del 1 de agosto:

+ Oeste de EE. UU. 2
+ Este de EE. UU.
+ Centro-sur de EE. UU.
+ US Gov - Virginia
+ US Gov: Arizona

<a name="service-access-and-authentication"></a>

## <a name="inbound-security-and-endpoint-protection"></a>Seguridad de entrada y protección de puntos de conexión

Las características de seguridad de entrada protegen el punto de conexión del servicio de búsqueda a través de niveles crecientes de seguridad y complejidad. En primer lugar, todas las solicitudes requieren una clave de API para el acceso autenticado. En segundo lugar, puede establecer opcionalmente las reglas de firewall que limitan el acceso a direcciones IP específicas. Para la protección avanzada, una tercera opción consiste en habilitar Azure Private Link para proteger el punto de conexión de servicio de todo el tráfico de Internet.

### <a name="public-access-using-api-keys"></a>Acceso público mediante claves de API

De forma predeterminada, se tiene acceso a un servicio de búsqueda a través de la nube pública mediante la autenticación basada en claves para el acceso de administrador o de consulta al punto de conexión del servicio de búsqueda. Una clave de API es una cadena que se compone de letras y números generados aleatoriamente. El tipo de clave (administrador o consulta) determina el nivel de acceso. El envío de una clave válida se considera una prueba de que la solicitud se origina desde una entidad de confianza.

Hay dos niveles de acceso al servicio de búsqueda, habilitados por las siguientes claves API:

+ Clave de administración (permite el acceso de lectura y escritura para operaciones de [crear-leer-actualizar-eliminar](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) en el servicio de búsqueda)

+ Clave de consulta (permite el acceso de solo lectura a la colección de documentos de un índice)

Las *claves de administración* se crean cuando se aprovisiona el servicio. Hay dos claves de administración, designadas como *principal* y *secundaria*, pero en realidad son intercambiables. Todos los servicios tienen dos claves de administración, con el fin de que se pueda dejar de usar una de ellas sin perder el acceso al servicio. También puede [regenerar la clave de administrador](search-security-api-keys.md#regenerate-admin-keys) periódicamente para seguir los procedimientos recomendados de seguridad de Azure, pero no se puede agregar al número total de claves de administrador. Hay un máximo de dos claves de administrador por servicio de búsqueda.

Las *claves de consulta* se crean a medida que son necesarias y están diseñadas para las aplicaciones cliente que emiten consultas. Puede crear hasta 50 claves de consulta. En el código de la aplicación, especifique la dirección URL de búsqueda y una clave de API de consulta para permitir el acceso de solo lectura a la colección de documentos de un índice específico. Juntos, el punto de conexión, una clave de API para el acceso de solo lectura y un índice de destino definen el nivel de acceso y ámbito de la conexión desde la aplicación cliente.

Se requiere autenticación en cada solicitud, y cada solicitud se compone de una clave obligatoria, una operación y un objeto. Cuando se encadenan, los dos niveles de permisos (completo y de solo lectura) y el contexto (por ejemplo, una operación de consulta en un índice) son suficientes para proporcionar seguridad en la gama completa de las operaciones del servicio. Para más información acerca de las claves, vea [Create and manage api-keys](search-security-api-keys.md) (Creación y administración de claves de API).

### <a name="ip-restricted-access"></a>Acceso con restricción de IP

Para controlar aún más el acceso al servicio de búsqueda, puede crear reglas de firewall de entrada que permitan el acceso a una dirección IP específica o a un intervalo de direcciones IP. Todas las conexiones de cliente deben realizarse a través de una dirección IP permitida, o la conexión se denegará.

Puede usar el portal para [configurar el acceso de entrada](service-configure-firewall.md).

Como alternativa, puede usar las API REST de administración. La versión de API 2020-03-13, con el parámetro [IpRule](/rest/api/searchmanagement/2019-10-01-preview/createorupdate-service#IpRule), le permite restringir el acceso a su servicio mediante la identificación (de forma individual o en un intervalo) de las direcciones IP a las que quiera otorgar acceso a su servicio de búsqueda.

### <a name="private-endpoint-no-internet-traffic"></a>Punto de conexión privado (sin tráfico de Internet)

Los [puntos de conexión privados](../private-link/private-endpoint-overview.md) para Azure Cognitive Search permiten a un cliente de una [red virtual](../virtual-network/virtual-networks-overview.md) obtener acceso de forma segura a los datos de un índice de búsqueda a través de un [vínculo privado](../private-link/private-link-overview.md).

El punto de conexión privado usa una dirección IP del espacio de direcciones de la red virtual para las conexiones al servicio de búsqueda. El tráfico de red entre el cliente y el servicio de búsqueda atraviesa la red virtual y un vínculo privado de la red troncal de Microsoft, lo que elimina la exposición a la red pública de Internet. Una red virtual permite establecer una comunicación segura entre recursos, con su red local y con Internet.

Aunque esta solución es la más segura, el uso de servicios adicionales supone un costo adicional, por lo que debe comprender claramente las ventajas antes de profundizar en ella. Para más información sobre los costos, consulte la [página de precios](https://azure.microsoft.com/pricing/details/private-link/). Para obtener más información sobre cómo funcionan conjuntamente estos componentes, vea el vídeo que se encuentra en la parte superior de este artículo. La opción de punto de conexión privado se trata a partir del minuto 5:48 del vídeo. Para obtener instrucciones sobre cómo configurar el punto de conexión, consulte [Creación de un punto de conexión privado para una conexión segura a Azure Cognitive Search](service-create-private-endpoint.md).

## <a name="index-access"></a>Acceso a los índices

En Azure Cognitive Search, un índice individual no es un objeto protegible. En su lugar, el acceso a un índice se determina en la capa de servicio (acceso de lectura o escritura al servicio), junto con el contexto de una operación.

Para el acceso del usuario final, puede estructurar las solicitudes de consulta para conectarse con una clave de consulta, lo que hace que cualquier solicitud sea de solo lectura e incluir el índice específico utilizado por la aplicación. En una solicitud de consulta, no existe el concepto de combinación de índices ni de acceso simultáneo a varios índices para que todas las solicitudes tengan un único índice de destino por definición. Por lo tanto, la construcción de la solicitud de consulta misma (una clave y un índice único de destino) define el límite de seguridad.

Los accesos de administrador y de desarrollador a los índices no se diferencian: ambos necesitan tener acceso de escritura para crear, eliminar y actualizar objetos administrados por el servicio. Cualquier persona con una clave de administración del servicio puede leer, modificar o eliminar cualquier índice en el mismo servicio. Para la protección contra la eliminación accidental o malintencionada de índices, su control de código fuente interno para los recursos de código es la solución para revertir una eliminación o modificación de índices no deseada. Azure Cognitive Search incluye conmutación por error dentro del clúster para garantizar la disponibilidad, pero no almacena ni ejecuta el código propietario utilizado para crear o cargar los índices.

Para soluciones multiinquilino que requieren límites de seguridad en el nivel de índice, estas soluciones suelen incluir un nivel intermedio que los clientes utilizan para controlar el aislamiento de índices. Para más información sobre los casos de uso de varios inquilinos, consulte [Modelos de diseño de aplicaciones SaaS para varios inquilinos y Azure Cognitive Search](search-modeling-multitenant-saas-applications.md).

## <a name="user-access"></a>Acceso de usuarios

El modo en que un usuario tiene acceso a un índice y otros objetos viene determinado por el tipo de clave de API de la solicitud. La mayoría de los desarrolladores crean y asignan [*claves de consulta*](search-security-api-keys.md) para las solicitudes de búsqueda del lado cliente. Una clave de consulta concede acceso de solo lectura al contenido que se puede buscar en el índice.

Si necesita tener un control por usuario pormenorizado de los resultados de la búsqueda, puede crear filtros de seguridad en las consultas, que devuelven documentos asociados con una identidad de seguridad determinada. En lugar de funciones predefinidas y asignaciones de roles, el control de acceso basado en identidades se implementa como un *filtro* que recorta los resultados de búsqueda de documentos y contenido en función de las identidades. La tabla siguiente describe dos enfoques para recortar el contenido no autorizado de los resultados de la búsqueda.

| Enfoque | Descripción |
|----------|-------------|
|[Recorte de seguridad basado en filtros de identidad](search-security-trimming-for-azure-search.md)  | Documenta el flujo de trabajo básico para implementar el control de acceso de identidades de usuario. Trata la incorporación de identificadores de seguridad a un índice y, a continuación, se explica el filtrado por ese campo para no incluir el contenido prohibido en los resultados. |
|[Recorte de seguridad basado en identidades de Azure Active Directory](search-security-trimming-for-azure-search-with-aad.md)  | En este artículo se amplía el artículo anterior y se proporcionan los pasos necesarios para recuperar identidades de Azure Active Directory (AAD), uno de los [servicios gratuitos](https://azure.microsoft.com/free/) de la plataforma de nube de Azure. |

## <a name="administrative-rights"></a>Derechos administrativos

El [control de acceso basado en rol de Azure (Azure RBAC)](../role-based-access-control/overview.md) es un sistema de autorización basado en [Azure Resource Manager](../azure-resource-manager/management/overview.md) para el aprovisionamiento de recursos de Azure. En Azure Cognitive Search, Resource Manager se usa para crear o eliminar el servicio, administrar las claves de API y escalar el servicio. Como tal, las asignaciones de roles de Azure determinan quién puede realizar esas tareas, independientemente de si se usa el [portal](search-manage.md), [PowerShell](search-manage-powershell.md) o las [API de REST de administración](/rest/api/searchmanagement/search-howto-management-rest-api).

En cambio, los derechos de administrador sobre el contenido hospedado en el servicio, como la capacidad de crear o eliminar un índice, se confieren a través de las claves de API, tal como se describe en la [sección anterior](#index-access).

> [!TIP]
> Mediante el uso de mecanismos de aplicación en todo el sistema de Azure, puede bloquear una suscripción o un recurso para evitar la eliminación accidental o no autorizada del servicio de búsqueda por parte de usuarios con derechos de administrador. Para obtener más información, vea [Bloqueo de recursos para impedir eliminación inesperada](../azure-resource-manager/management/lock-resources.md).

## <a name="certifications-and-compliance"></a>Certificaciones y cumplimiento

Azure Cognitive Search tiene la certificación compatible con varios estándares globales, regionales y específicos del sector para la nube pública y Azure Government. Para obtener la lista completa, descargue las notas del producto de las [**ofertas de cumplimiento de** Microsoft Azure](https://azure.microsoft.com/resources/microsoft-azure-compliance-offerings/).

En el caso del cumplimiento, puede usar [Azure Policy](../governance/policy/overview.md) para implementar los procedimientos recomendados de alta seguridad de [Azure Security Benchmark](../security/benchmarks/introduction.md). Azure Security Benchmark es una colección de recomendaciones de seguridad codificadas en controles de seguridad que se asignan a acciones clave que se deben realizar para mitigar las amenazas a los servicios y los datos. Actualmente hay 11 controles de seguridad, entre los que se incluyen [Seguridad de red](../security/benchmarks/security-control-network-security.md), [Registro y supervisión](../security/benchmarks/security-control-logging-monitoring.md) y [Protección de datos](../security/benchmarks/security-control-data-protection.md), por nombrar algunos.

Azure Policy es una capacidad integrada en Azure que ayuda a administrar el cumplimiento de varios estándares, incluidos los de Azure Security Benchmark. En el caso de los bancos de pruebas conocidos, Azure Policy proporciona definiciones integradas que ofrecen tanto criterios como una respuesta accionable que aborda el no cumplimiento.

En Azure Cognitive Search, actualmente hay una definición integrada. Es para el registro de diagnóstico. Con ella integrada, puede asignar una directiva que identifique cualquier servicio de búsqueda que no tenga registro de diagnóstico y lo active. Para obtener más información, vea [Controles de Cumplimiento normativo de Azure Policy para Azure Cognitive Search](security-controls-policy.md).

## <a name="see-also"></a>Consulte también

+ [Aspectos básicos de la seguridad de Azure](../security/fundamentals/index.yml)
+ [Azure Security](https://azure.microsoft.com/overview/security)
+ [Azure Security Center](../security-center/index.yml)