---
title: ¿Qué es el firewall de aplicaciones web de Azure en Azure Front Door?
description: Obtenga información sobre cómo el firewall de aplicaciones web de Azure en Azure Front Door Service protege sus aplicaciones web frente a ataques malintencionados.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: conceptual
ms.date: 05/22/2020
ms.author: victorh
ms.openlocfilehash: e3b3b7a058fbd1235619c01e0dca24a3de9036db
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89231442"
---
# <a name="azure-web-application-firewall-on-azure-front-door"></a>Firewall de aplicaciones web en Azure en Azure Front Door

Firewall de aplicaciones web (WAF) en Azure Front Door proporciona protección centralizada para las aplicaciones web. WAF defiende los servicios web frente a vulnerabilidades comunes. Mantiene su servicio de alta disponibilidad para los usuarios y le ayuda a cumplir los requisitos de cumplimiento.

WAF en Front Door es una solución global y centralizada. Está implementado en ubicaciones de perímetro de red de Azure de todo el mundo. Las aplicaciones web habilitadas para WAF inspeccionan todas las solicitudes entrantes entregadas por Front Door en el perímetro de red. 

WAF evita los ataques malintencionados cerca de los orígenes de ataques antes de que entren en la red virtual. El usuario obtiene protección a gran escala sin sacrificar el rendimiento. Una directiva WAF se vincula fácilmente a cualquier perfil de Front Door de la suscripción. Las nuevas reglas se implementan en cuestión de minutos, por lo que puede responder rápidamente a los cambios en los patrones de amenazas.

![Firewall de aplicaciones web de Azure](../media/overview/wafoverview.png)

## <a name="waf-policy-and-rules"></a>Reglas y directiva de WAF

Puede configurar una [directiva de WAF](waf-front-door-create-portal.md) y asociarla a uno o varios servidores front-end de Front Door para estar protegido. Una directiva de WAF consta de dos tipos de reglas de seguridad:

- Reglas personalizadas que crea el cliente.

- Conjuntos de reglas administrados que son una colección del conjunto de reglas configurado previamente y administrado por Azure.

Cuando ambos están presentes, las reglas personalizadas se procesan antes de procesar las reglas de un conjunto de reglas administrado. Una regla está formada por una condición de coincidencia, una prioridad y una acción. Los tipos de acción que se admiten son los siguientes: ALLOW, BLOCK, LOG y REDIRECT. Puede crear una directiva totalmente personalizada que cumpla sus requisitos específicos de protección de aplicaciones al combinar reglas personalizadas y administradas.

Las reglas de una directiva se procesan en un orden de prioridad. La prioridad es un entero único que describe el orden de las reglas que se van a procesar. El valor entero más pequeño indica una prioridad más alta y estas reglas se evalúan antes que las reglas con un valor entero más alto. Una vez que una regla coincide, la acción correspondiente que se ha definido en la regla se aplica a la solicitud. Cuando se procesa esa coincidencia, ya no se procesan más reglas con prioridades inferiores.

Una aplicación web entregada por Front Door solo puede tener una directiva de WAF asociada a la vez. En cambio, puede tener una configuración de Front Door sin ninguna directiva de WAF asociada con ella. Si una directiva de WAF está presente, se replica a todas nuestras ubicaciones perimetrales para garantizar que haya coherencia en las directivas de seguridad en todo el mundo.

## <a name="waf-modes"></a>Modos de WAF

La directiva WAF se puede configurar para ejecutarse en los siguientes dos modos:

- **Modo de detección:** cuando se ejecuta en el modo de detección, WAF no realiza ninguna acción que no sea supervisar y registrar la solicitud y sus reglas de WAF coincidentes en los registros de WAF. Puede activar los diagnósticos de registro para Front Door. Vaya a la sección **Diagnostics** del portal.

- **Modo de prevención:** En el modo de prevención, WAF realiza la acción especificada si una solicitud coincide con una regla. Si se encuentra una coincidencia, no se evalúa ninguna otra regla con prioridad más baja. Todas las solicitudes coincidentes también se registran en los registros de WAF.

## <a name="waf-actions"></a>Acciones de WAF

Los clientes de WAF pueden optar por ejecutar desde una de las acciones cuando una solicitud coincide con las condiciones de una regla:

- **Permitir:**  la solicitud pasa por WAF y se reenvía al servidor back-end. Ninguna otra regla de prioridad más baja puede bloquear esta solicitud.
- **Bloquear:** la solicitud está bloqueada y WAF envía una respuesta al cliente sin reenviar la solicitud al servidor back-end.
- **Registrar:**  la solicitud se registra en los registros de WAF y WAF sigue evaluando las reglas de prioridad más baja.
- **Redirigir:** WAF redirige la solicitud al URI especificado. El URI especificado es una configuración de nivel de directiva. Una vez configurado, todas las solicitudes que coinciden con la acción **Redirigir** se enviarán a ese URI.

## <a name="waf-rules"></a>Reglas de WAF

Una directiva de WAF puede constar de dos tipos de reglas de seguridad: reglas personalizadas (creadas por el cliente) y conjuntos de reglas administrados (un conjunto de reglas configurado previamente y administrado por Azure).

### <a name="custom-authored-rules"></a>Reglas creadas personalizadas

Puede configurar reglas personalizadas de WAF de la siguiente forma:

- **Listas de direcciones IP permitidas y denegadas:** puede controlar el acceso a las aplicaciones web en función de una lista de direcciones IP de cliente o intervalos de direcciones IP. Se admiten los tipos de direcciones IPv4 e IPv6. Esta lista puede configurarse para bloquear o permitir esas solicitudes en las que la IP de origen coincide con una IP de la lista.

- **Control de acceso basado en la ubicación geográfica:** puede controlar el acceso a las aplicaciones web en función del código de país asociado con la dirección IP de un cliente.

- **Control de acceso basado en los parámetros HTTP:** puede basar las reglas en coincidencias de cadenas en los parámetros de solicitud HTTP/HTTPS.  Por ejemplo, las cadenas de consulta, los argumentos POST, el URI de solicitud, el encabezado de solicitud y el cuerpo de la solicitud.

- **Control de acceso basado en el método de solicitud:** puede basar las reglas en el método de solicitud HTTP de la solicitud. Por ejemplo, GET, PUT o HEAD.

- **Restricción del tamaño:** puede basar las reglas en las longitudes de determinadas partes de una solicitud, como la cadena de consulta, el URI o el cuerpo de la solicitud.

- **Reglas de limitación de frecuencia:** Las reglas de control de frecuencia están diseñadas para limitar el tráfico anormalmente alto procedente de cualquier dirección IP de cliente. Puede configurar un umbral en función del número de solicitudes web que se permiten de una dirección IP de cliente durante un minuto. Esta regla es distinta de la regla personalizada de permiso o bloqueo basada en la lista de IP que permita o bloquee todas las solicitudes de una IP de cliente. La limitación de velocidad puede combinarse con condiciones de coincidencia adicionales, como la coincidencia de parámetros HTTP(S) para un control granular de la velocidad.

### <a name="azure-managed-rule-sets"></a>Conjuntos de reglas administrados por Azure

Los conjuntos de reglas administrados por Azure proporcionan una forma fácil de implementar la protección frente a un conjunto común de amenazas de seguridad. Dado que Azure administra estos conjuntos de reglas, las reglas se actualizan según sea necesario para protegerse frente a nuevas firmas de ataque. El conjunto de reglas predeterminado administrado por Azure incluye reglas frente a las siguientes categorías de amenaza:

- Scripting entre sitios
- Ataques de Java
- Inclusión de archivos locales
- Ataques por inyección de PHP
- Ejecución de comandos remotos
- Inclusión de archivos remotos
- Fijación de sesión
- Protección contra la inyección de código SQL
- Atacantes de protocolo

El número de versión del conjunto de reglas predeterminado se incrementa cuando se agreguen nuevas firmas de ataque al conjunto de reglas.
El conjunto de reglas predeterminado está habilitado de forma predeterminada en el modo de detección de las directivas de WAF. Puede deshabilitar o habilitar reglas individuales en el conjunto de reglas predeterminado para satisfacer los requisitos de su aplicación. También puede establecer acciones específicas (ALLOW/BLOCK/REDIRECT/LOG) por regla.

A veces, es posible que tenga que omitir determinados atributos de una solicitud de una evaluación del WAF. Un ejemplo común son los tokens insertados de Active Directory que se usan para la autenticación. Puede configurar una lista de exclusión para una regla administrada, un grupo de reglas o para todo el conjunto de reglas.  

La acción predeterminada es bloquear (BLOCK). Además, las reglas personalizadas pueden configurarse en la misma directiva de WAF si quiere omitir cualquiera de las reglas previamente configuradas en el conjunto de reglas predeterminado.

Las reglas personalizadas se aplican siempre antes de que se evalúen las reglas del conjunto de reglas predeterminado. Si una solicitud coincide con una regla personalizada, se aplica la acción de la regla correspondiente. La solicitud se bloquea o se pasa por el back-end. No se procesa ninguna otra regla personalizada ni las reglas del conjunto de reglas predeterminado. También puede quitar el conjunto de reglas predeterminado de las directivas de WAF.

### <a name="bot-protection-rule-set-preview"></a>Conjunto de reglas de protección contra bots (versión preliminar)

Se puede habilitar un conjunto administrado de reglas de protección contra bots realizar acciones personalizadas en las solicitudes de categorías de bots conocidas. 

Se admiten tres categorías de bot: Defectuosos, correctos y desconocidos La plataforma WAF administra y actualiza dinámicamente las firmas de bots.

Los bots defectuosos incluyen bots de direcciones IP malintencionadas y bots que han falsificado sus identidades. Las direcciones IP malintencionadas proceden de la fuente de inteligencia sobre amenazas de Microsoft. [Intelligent Security Graph](https://www.microsoft.com/security/operations/intelligence) impulsa la Inteligencia sobre amenazas de Microsoft y lo utilizan numerosos servicios, incluido Azure Security Center.

Los bots correctos incluyen motores de búsqueda validados. Entre las categorías desconocidas se incluyen los grupos de bot adicionales que se identificaron a sí mismos como bots. Por ejemplo, analizadores de mercado, recopiladores de fuentes y agentes de recopilación de datos. 

Los bots desconocidos se clasifican mediante agentes de usuario publicados sin una validación adicional. Puede establecer acciones personalizadas para bloquear, permitir, registrar o redirigir los distintos tipos de bots.

![Conjunto de reglas de protección contra bots](../media/afds-overview/botprotect2.png)

> [!IMPORTANT]
> El conjunto de reglas de protección contra bots se encuentra en versión preliminar pública y se proporciona con un Acuerdo de nivel de servicio de versión preliminar. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.  Para más información, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Si la protección contra bots está habilitada, las solicitudes entrantes que coincidan con las reglas de bots se registran en el registro FrontdoorWebApplicationFirewallLog. Puede acceder a los registros de WAF desde una cuenta de almacenamiento, centro de eventos o análisis de registros.

## <a name="configuration"></a>Configuración

Puede configurar e implementar todos los tipos de reglas de WAF mediante Azure Portal, las API REST, las plantillas de Azure Resource Manager y Azure PowerShell.

## <a name="monitoring"></a>Supervisión

La supervisión de WAF en Front Door se integra con Azure Monitor para realizar un seguimiento de las alertas y supervisar con facilidad las tendencias del tráfico.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda sobre el [firewall de aplicaciones web en Azure Application Gateway](../ag/ag-overview.md).