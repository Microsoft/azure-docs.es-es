---
title: Protección de Azure Functions
description: Obtenga información sobre cómo hacer que el código de función que se ejecuta en Azure sea más seguro frente a ataques comunes.
ms.date: 4/13/2020
ms.topic: conceptual
ms.openlocfilehash: e0c5036681aace103ea69d1e9cc73e96dc30821f
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502688"
---
# <a name="securing-azure-functions"></a>Protección de Azure Functions

En muchos sentidos, la planeación del desarrollo, la implementación y el funcionamiento seguros de las funciones sin servidor es muy similar a la de cualquier aplicación hospedada en la nube o basada en web. [Azure App Service](../app-service/index.yml) proporciona la infraestructura de hospedaje para las aplicaciones de funciones. En este artículo se proporcionan estrategias de seguridad para ejecutar el código de función y cómo App Service puede facilitar la protección de las funciones. 

[!INCLUDE [app-service-security-intro](../../includes/app-service-security-intro.md)]

Para obtener un conjunto de recomendaciones de seguridad que siguen [Azure Security Benchmark](../security/benchmarks/overview.md), vea [Base de referencia de seguridad de Azure para Azure Functions](security-baseline.md).

## <a name="secure-operation"></a>Operación segura 

Esta sección le guía en la configuración y ejecución de la aplicación de funciones de la manera más segura posible. 

### <a name="security-center"></a>Security Center

Security Center se integra con la aplicación de funciones en el portal. Proporciona, de forma gratuita, una evaluación rápida de posibles vulnerabilidades de seguridad relacionadas con la configuración. Las aplicaciones de funciones que se ejecutan en un plan dedicado también pueden usar las características de seguridad en tiempo real de Security Center, a un precio adicional. Para obtener más información, vea [Protección de las aplicaciones web y API de Azure App Service](../security-center/security-center-app-services.md). 

### <a name="log-and-monitor"></a>Registro y supervisión

Una forma de detectar ataques es a través de la supervisión de actividades y el análisis de registro. Functions se integra con Application Insights para recopilar datos de registro, rendimiento y errores para la aplicación de funciones. Application Insights detecta automáticamente anomalías en el rendimiento e incluye herramientas de análisis eficaces que ayudan a diagnosticar problemas y comprender cómo se usan las funciones. Para más información, consulte [Supervisión de Azure Functions](functions-monitoring.md).

Functions también se integra con los registros de Azure Monitor para que pueda consolidar los registros de la aplicación de funciones con eventos del sistema para facilitar el análisis. Puede usar la configuración de diagnóstico para configurar la exportación de streaming de registros y métricas de la plataforma para las funciones al destino que elija, como un área de trabajo de Log Analytics. Para obtener más información, vea [Supervisión de Azure Functions con registros de Azure Monitor](functions-monitor-log-analytics.md). 

En el caso de la detección de amenazas de nivel empresarial y la automatización de respuestas, transmita los registros y eventos a un área de trabajo de Logs Analytics. Después, puede conectar Azure Sentinel a esta área de trabajo. Para obtener más información, vea [¿Qué es Azure Sentinel?](../sentinel/overview.md)  

Para obtener más recomendaciones de seguridad para la observación, vea la [Base de referencia de seguridad de Azure para Azure Functions](security-baseline.md#logging-and-monitoring). 

### <a name="require-https"></a>Requerir HTTPS

De forma predeterminada, los clientes se pueden conectar a puntos de conexión de función mediante HTTP o HTTPS. Debe redirigir HTTP a HTTPs porque HTTPS usa el protocolo SSL/TLS para proporcionar una conexión segura, que está tanto cifrada como autenticada. Para obtener información sobre cómo hacerlo, vea [Aplicación de HTTPS](../app-service/configure-ssl-bindings.md#enforce-https).

Cuando requiera HTTPS, también debe requerir la versión más reciente de TLS. Para obtener información sobre cómo hacerlo, vea [Exigencia de las versiones TLS](../app-service/configure-ssl-bindings.md#enforce-tls-versions).

Para obtener más información, vea [Conexiones seguras (TSL)](../app-service/overview-security.md#https-and-certificates).

### <a name="function-access-keys"></a>Claves de acceso de función

[!INCLUDE [functions-authorization-keys](../../includes/functions-authorization-keys.md)]

#### <a name="system-key"></a>Clave del sistema 

Las extensiones específicas pueden requerir una clave administrada por el sistema para acceder a los puntos de conexión de webhook. Las claves del sistema están diseñadas para puntos de conexión de función específicos de la extensión que llaman los componentes internos. Por ejemplo, el [desencadenador Event Grid](functions-bindings-event-grid-trigger.md) requiere que la suscripción use una clave del sistema cuando se llama al punto de conexión del desencadenador. Durable Functions también usa claves del sistema para llamar a las [API de extensión de Durable Task](durable/durable-functions-http-api.md). 

El ámbito de las claves del sistema viene determinado por la extensión, pero generalmente se aplica a toda la aplicación de funciones. Las claves del sistema solo se pueden crear mediante extensiones específicas y sus valores no se pueden establecer de forma explícita. Como sucede con otras claves, puede generar un valor nuevo para la clave desde el portal o mediante las API de clave.

#### <a name="keys-comparison"></a>Comparación de claves

En la tabla siguiente se comparan los usos de los distintos tipos de claves de acceso:

| Acción                                        | Ámbito                    | Claves válidas         |
|-----------------------------------------------|--------------------------|--------------------|
| Ejecutar una función                            | Función específica        | Función           |
| Ejecutar una función                            | Cualquier función             | Función o host   |
| Llamada a un punto de conexión de administración                        | Aplicación de función             | Host (solo maestro) |
| Llamada a las API de extensión de Durable Task              | Aplicación de funciones<sup>1</sup> | Sistema<sup>2</sup> |
| Llamada a un webhook específico de la extensión (interna) | Aplicación de funciones<sup>1</sup> | Sistema<sup>2</sup> |

<sup>1</sup>Ámbito determinado por la extensión.  
<sup>2</sup>Nombres específicos establecidos por la extensión.

Para obtener más información sobre las claves de acceso, vea el [artículo sobre enlace de desencadenadores HTTP](functions-bindings-http-webhook-trigger.md#obtaining-keys).

### <a name="authenticationauthorization"></a>Autenticación y autorización

Aunque las claves de función pueden proporcionar cierta mitigación del acceso no deseado, la única manera de proteger realmente los puntos de conexión de la función es mediante la implementación de la autenticación positiva de los clientes que acceden a las funciones. Después, puede tomar decisiones de autorización en función de la identidad.  

[!INCLUDE [functions-enable-auth](../../includes/functions-enable-auth.md)]

### <a name="permissions"></a>Permisos

Como sucede con cualquier aplicación o servicio, el objetivo es ejecutar la aplicación de funciones con los mínimos permisos posibles. 

#### <a name="user-management-permissions"></a>Permisos de administración de usuarios

Functions admite el [control de acceso basado en rol de Azure (Azure RBAC)](../role-based-access-control/overview.md) integrado. Los roles de Azure que admite Functions son [Colaborador](../role-based-access-control/built-in-roles.md#contributor), [Propietario](../role-based-access-control/built-in-roles.md#owner) y [Lector](../role-based-access-control/built-in-roles.md#owner). 

Los permisos son eficaces en el nivel de la aplicación de funciones. El rol Colaborador es necesario para realizar la mayoría de las tareas de nivel de la aplicación de funciones. Solo el rol Propietario puede eliminar una aplicación de funciones. 

#### <a name="organize-functions-by-privilege"></a>Organización de funciones por privilegio 

Las cadenas de conexión y otras credenciales almacenadas en la configuración de la aplicación proporcionan a todas las funciones de la aplicación de funciones el mismo conjunto de permisos en el recurso asociado. Considere la posibilidad de minimizar el número de funciones con acceso a credenciales específicas moviendo las funciones que no las utilizan a una aplicación de funciones independiente. Siempre puede usar técnicas como el [encadenamiento de funciones](/learn/modules/chain-azure-functions-data-using-bindings/) para pasar datos entre funciones de diferentes aplicaciones de funciones.  

#### <a name="managed-identities"></a>Identidades administradas

[!INCLUDE [app-service-managed-identities](../../includes/app-service-managed-identities.md)]

Para obtener más información, vea [Procedimiento para usar identidades administradas para App Service y Azure Functions](../app-service/overview-managed-identity.md?toc=%2fazure%2fazure-functions%2ftoc.json).

#### <a name="restrict-cors-access"></a>Restricción del acceso a CORS

[Uso compartido de recursos entre orígenes (CORS)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) es una manera de permitir que las aplicaciones web que se ejecutan en otro dominio realicen solicitudes a los puntos de conexión del desencadenador HTTP. App Service proporciona compatibilidad integrada para entregar los encabezados de CORS necesarios en las solicitudes HTTP. Las reglas de CORS se definen en el nivel de la aplicación de funciones.  

Aunque resulte tentador usar un carácter comodín que permita a todos los sitios acceder al punto de conexión, esto anula el propósito de CORS, que es ayudar a evitar los ataques de scripts entre sitios. En su lugar, agregue una entrada de CORS independiente para el dominio de cada aplicación web que tenga que acceder al punto de conexión. 

### <a name="managing-secrets"></a>Administración de secretos 

Para poder conectarse a los diversos servicios y recursos necesarios para ejecutar el código, las aplicaciones de funciones deben poder acceder a secretos, como cadenas de conexión y claves de servicio. En esta sección se describe cómo almacenar los secretos requeridos por las funciones.

Nunca almacene los secretos en el código de función. 

#### <a name="application-settings"></a>Configuración de la aplicación

De forma predeterminada, las cadenas de conexión y los secretos que usa la aplicación de funciones y los enlaces se almacenan como configuración de la aplicación. Esto hace que estas credenciales estén disponibles tanto para el código de función como para los distintos enlaces que usa la función. El nombre de la configuración de la aplicación (clave) se usa para recuperar el valor real, que es el secreto. 

Por ejemplo, todas las aplicaciones de funciones requieren una cuenta de almacenamiento asociada, que la usa por el tiempo de ejecución. De forma predeterminada, la conexión a esta cuenta de almacenamiento se almacena en una configuración de aplicación denominada `AzureWebJobsStorage`.

La configuración de la aplicación y las cadenas de conexión se almacenan cifradas en Azure. Solo se descifran antes de insertarlas en la memoria de proceso de la aplicación cuando se inicia la aplicación. Las claves de cifrado rotan con regularidad. Si en su lugar prefiere administrar el almacenamiento seguro de los secretos, la configuración de la aplicación debe ser referencias a Azure Key Vault. 

#### <a name="key-vault-references"></a>Referencias de Key Vault

Aunque la configuración de la aplicación es suficiente para la mayoría de las funciones, es posible que quiera compartir los mismos secretos entre varios servicios. En este caso, el almacenamiento redundante de los secretos genera más vulnerabilidades. Un enfoque más seguro consiste en usar un servicio de almacenamiento secreto central y utilizar referencias a este servicio en lugar de los propios secretos.      

[Azure Key Vault](../key-vault/general/overview.md) es un servicio que proporciona administración centralizada de los secretos, con control total sobre las directivas de acceso y el historial de auditoría. Puede usar una referencia de Key Vault en lugar de una cadena de conexión o clave en la configuración de la aplicación. Para obtener más información, vea [Uso de referencias de Key Vault para App Service y Azure Functions](../app-service/app-service-key-vault-references.md?toc=%2fazure%2fazure-functions%2ftoc.json).

### <a name="set-usage-quotas"></a>Establecimiento de cuotas de uso

Considere la posibilidad de establecer una cuota de uso en las funciones que se ejecutan en un plan de consumo. Cuando se establece un límite diario de GB por segundo en la suma total de la ejecución de funciones en la aplicación de funciones, la ejecución se detiene cuando se alcanza el límite. Esto podría ayudar a mitigar la posibilidad de que código malintencionado ejecute las funciones. Para obtener información sobre cómo calcular el consumo de las funciones, vea [Estimación de los costos según el plan de consumo](functions-consumption-costs.md). 

### <a name="data-validation"></a>Validación de datos

Los desencadenadores y enlaces que usan las funciones no proporcionan ninguna validación de datos adicional. El código debe validar los datos recibidos de un desencadenador o enlace de entrada. Si un servicio ascendente está en peligro, no querrá que las entradas no validadas fluyan a través de las funciones. Por ejemplo, si la función almacena datos de una cola de Azure Storage en una base de datos relacional, debe validar los datos y parametrizar los comandos para evitar ataques por inyección de código SQL. 

No asuma que los datos que entran en la función ya se han validado o saneado. También es recomendable comprobar que los datos que se escriben en los enlaces de salida sean válidos. 

### <a name="handle-errors"></a>errores

Aunque parezca básico, es importante escribir un control de errores adecuado en las funciones. Los errores no controlados se propagan al host y se controlan mediante el tiempo de ejecución. Cada enlace controla el procesamiento de errores de manera distinta. Para obtener más información, vea [Control de errores de Azure Functions](functions-bindings-error-pages.md).

### <a name="disable-remote-debugging"></a>Deshabilitar la depuración remota

Asegúrese de que la depuración remota está deshabilitada, excepto cuando depure las funciones de forma activa. Puede deshabilitar la depuración remota en la pestaña **Configuración general** de la **Configuración** de la aplicación de funciones en el portal. 

### <a name="restrict-cors-access"></a>Restricción del acceso a CORS

[!INCLUDE [functions-cors](../../includes/functions-cors.md)]

No use caracteres comodín en la lista de orígenes permitidos. En su lugar, enumere los dominios específicos de los que espera obtener solicitudes.

### <a name="store-data-encrypted"></a>Almacenamiento de datos cifrados

[!INCLUDE [functions-storage-encryption](../../includes/functions-storage-encryption.md)]

## <a name="secure-deployment"></a>Implementación segura

Las herramientas de Azure Functions para la integración facilitan la publicación local de código de proyectos de función en Azure. Es importante entender cómo funciona la implementación a la hora de considerar la seguridad de una topología de Azure Functions.   

### <a name="deployment-credentials"></a>Credenciales de implementación

Las implementaciones de App Service requieren un conjunto de credenciales de implementación. Estas credenciales de implementación se usan para proteger las implementaciones de aplicaciones de funciones. Las credenciales de implementación se administran mediante la plataforma App Service y se cifran en reposo. 

Hay dos tipos de credenciales de implementación:

[!INCLUDE [app-service-deploy-credentials](../../includes/app-service-deploy-credentials.md)]

En este momento, no se admite Key Vault para las credenciales de implementación. Para obtener más información sobre la administración de credenciales de implementación, vea [Configuración de credenciales de implementación para Azure App Service](../app-service/deploy-configure-credentials.md).

### <a name="disable-ftp"></a>Deshabilitación de FTP

De forma predeterminada, cada aplicación de funciones tiene un punto de conexión FTP habilitado. Para acceder al punto de conexión FTP se usan credenciales de implementación. 

No se recomienda FTP para implementar el código de función. Las implementaciones de FTP son manuales y requieren la sincronización de los desencadenadores. Para obtener más información, vea [Implementación de FTP](functions-deployment-technologies.md#ftp). 

Si no tiene previsto usar FTP, debe deshabilitarlo en el portal. Si decide usar FTP, debe [aplicar FTPS](../app-service/deploy-ftp.md#enforce-ftps).

### <a name="secure-the-scm-endpoint"></a>Protección del punto de conexión scm

Todas las aplicaciones de funciones tienen un punto de conexión de servicio `scm` correspondiente que usa el servicio de herramientas avanzadas (Kudu) para las implementaciones y otras [extensiones de sitio](https://github.com/projectkudu/kudu/wiki/Azure-Site-Extensions) de App Service. El punto de conexión scm para una aplicación de funciones siempre es una dirección URL con el formato `https://<FUNCTION_APP_NAME.scm.azurewebsites.net>`. Cuando se usa el aislamiento de red para proteger las funciones, también debe tener en cuenta este punto de conexión. 

Al tener un punto de conexión scm independiente, puede controlar las implementaciones y otras funcionalidades de herramientas avanzadas para la aplicación de funciones que están aisladas o se ejecutan en una red virtual. El punto de conexión scm admite la autenticación básica (con credenciales de implementación) y el inicio de sesión único con las credenciales de Azure Portal. Para obtener más información, vea [Acceso al servicio Kudu](https://github.com/projectkudu/kudu/wiki/Accessing-the-kudu-service). 

### <a name="continuous-security-validation"></a>Validación continua de la seguridad

Como en todos los pasos del proceso de desarrollo se debe tener en cuenta la seguridad, también tiene sentido implementar validaciones de seguridad en un entorno de implementación continua. Esto se denomina en ocasiones DevSecOps. Al usar Azure DevOps para la canalización de implementación se puede integrar la validación en el proceso de implementación. Para obtener más información, vea [Incorporación de un mecanismo continuo para validar la seguridad en la canalización de CI/CD](/azure/devops/migrate/security-validation-cicd-pipeline).  

## <a name="network-security"></a>Seguridad de las redes

La restricción del acceso de red a la aplicación de funciones permite controlar quién puede acceder a los puntos de conexión de las funciones. Functions aprovecha la infraestructura de App Service para permitir que las funciones accedan a los recursos sin usar direcciones enrutables por Internet o para restringir el acceso a Internet a un punto de conexión de función. Para obtener más información sobre estas opciones de red, vea [Opciones de redes de Azure Functions](functions-networking-options.md).

### <a name="set-access-restrictions"></a>Establecimiento de restricciones de acceso

Las restricciones de acceso permiten definir listas de reglas de permiso y denegación para controlar el tráfico a la aplicación. Las reglas se evalúan en orden de prioridad. Si no se define ninguna regla, la aplicación aceptará el tráfico de cualquier dirección. Para obtener más información, vea [Restricciones de acceso de Azure App Service](../app-service/app-service-ip-restrictions.md?toc=%2fazure%2fazure-functions%2ftoc.json).

### <a name="private-site-access"></a>El acceso privado a sitios

[!INCLUDE [functions-private-site-access](../../includes/functions-private-site-access.md)]

### <a name="deploy-your-function-app-in-isolation"></a>Implementación de la aplicación de funciones en aislamiento

[!INCLUDE [functions-deploy-isolation](../../includes/functions-deploy-isolation.md)]

### <a name="use-a-gateway-service"></a>Uso de un servicio de puerta de enlace

Los servicios de puerta de enlace, como [Azure Application Gateway](../application-gateway/overview.md) y [Azure Front Door](../frontdoor/front-door-overview.md) permiten configurar un firewall de aplicaciones web (WAF). Las reglas de WAF se usan para supervisar o bloquear los ataques detectados, lo que proporciona una capa de protección adicional para las funciones. Para configurar un firewall de aplicaciones web, la aplicación de funciones debe estar en ejecución en un ASE o usar puntos de conexión privados (versión preliminar). Para obtener más información, vea [Uso de puntos de conexión privados](../app-service/networking/private-endpoint.md).    

## <a name="next-steps"></a>Pasos siguientes

+ [Base de referencia de seguridad de Azure para Azure Functions](security-baseline.md)
+ [Diagnósticos de Azure Functions](functions-diagnostics.md)
