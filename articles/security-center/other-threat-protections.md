---
title: Protección adicional contra amenazas en Azure Security Center
description: Más información sobre la protección contra amenazas disponible en Azure Security Center además de Azure Defender
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 33c45447-3181-4b75-aa8e-c517e76cd50d
ms.service: security-center
ms.topic: conceptual
ms.date: 09/15/2020
ms.author: memildin
ms.openlocfilehash: 0f4a849af2be9f02187dc3cda526c9c4727cab1b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932136"
---
# <a name="additional-threat-protections-in-azure-security-center"></a>Protecciones adicionales contra amenazas en Azure Security Center
Además de las [protecciones de Azure Defender](azure-defender.md) integradas, Azure Security Center también ofrece las siguientes funcionalidades de protección contra amenazas.

> [!TIP]
> Para habilitar las funcionalidades de protección contra amenazas de Security Center, debe habilitar Azure Defender en la suscripción que contenga las cargas de trabajo aplicables.
>
> Solo puede habilitar la protección contra amenazas para **Azure Database for MariaDB/MySQL/PostgreSQL** en el nivel de recursos.


## <a name="threat-protection-for-azure-network-layer"></a>Protección contra amenazas para la capa de red de Azure <a name="network-layer"></a>
El análisis de la capa de red de Security Center se basa en [datos IPFIX](https://en.wikipedia.org/wiki/IP_Flow_Information_Export) de ejemplo, que son encabezados de paquete recopilados por los enrutadores principales de Azure. En función de esta fuente de distribución de datos, Security Center utiliza modelos de Machine Learning para identificar y marcar actividades de tráfico malintencionado. Security Center también utiliza la base de datos de Microsoft Threat Intelligence para enriquecer las direcciones IP.

Algunas configuraciones de red pueden restringir la Security Center de la generación de alertas en una actividad de red sospechosa. Para que Security Center genere alertas de red, asegúrese de que:
- La máquina virtual tenga una dirección IP pública (o se encuentre en un equilibrador de carga con una dirección IP pública).
- El tráfico de salida de red de la máquina virtual no esté bloqueado por una solución de IDS externa.
- A la máquina virtual se le haya asignado la misma dirección IP durante toda la hora en la que se produjo la comunicación sospechosa. Esto también se aplica a las máquinas virtuales creadas como parte de un servicio administrado (por ejemplo, AKS, Databricks).

Para obtener una lista de las alertas de nivel de red de Azure, consulte la [tabla de referencia de alertas](alerts-reference.md#alerts-azurenetlayer).


## <a name="threat-protection-for-azure-resource-manager-preview"></a>Protección contra amenazas para Azure Resource Manager (versión preliminar)<a name ="management-layer"></a>
La capa de protección de Security Center basada en Azure Resource Manager está actualmente en versión preliminar.

Security Center ofrece una capa adicional de protección al usar eventos de Azure Resource Manager, lo que se considera el plano de control de Azure. Al analizar los registros de Azure Resource Manager, Security Center detecta operaciones inusuales o potencialmente peligrosas en el entorno de suscripción de Azure.

Para obtener una lista de las alertas de Azure Resource Manager (versión preliminar), consulte la [Tabla de referencia de alertas](alerts-reference.md#alerts-azureresourceman).


>[!NOTE]
> Varios de los análisis anteriores se realizan mediante Microsoft Cloud App Security. Para beneficiarse de estos análisis, debe activar una licencia de Cloud App Security. Si tiene una licencia de Cloud App Security, estas alertas están habilitadas de forma predeterminada. Para deshabilitar las alertas:
>
> 1. En el menú de Security Center, seleccione **Precios y configuración**.
> 1. Seleccione la suscripción que desea cambiar.
> 1. Seleccione **Detección de amenazas**.
> 1. Desactive la opción **Permita que Microsoft Cloud App Security acceda a sus datos** y seleccione **Guardar**.


>[!NOTE]
>Security Center almacena datos de clientes relacionados con la seguridad en la misma zona geográfica que su recurso. Si Microsoft aún no ha implementado Security Center en la zona geográfica del recurso, almacenará los datos en Estados Unidos. Cuando Cloud App Security esté habilitado, esta información se almacenará con arreglo a las reglas de ubicación geográfica de Cloud App Security. Para obtener más información, consulte [Almacenamiento de datos para servicios no regionales](https://azuredatacentermap.azurewebsites.net/).

1. Establezca el área de trabajo en la que va a instalar el agente. Asegúrese de que el área de trabajo está en la misma suscripción que se usa en Security Center y que tiene permisos de lectura/escritura en el área de trabajo.

1. Habilite **Azure Defender** y seleccione **Guardar**.


## <a name="threat-protection-for-azure-cosmos-db-preview"></a>Protección contra amenazas para Azure Cosmos DB (versión preliminar)<a name="cosmos-db"></a>

Las alertas de Azure Cosmos DB las generan los intentos inusuales y potencialmente dañinos de acceso o aprovechamiento de cuentas de Azure Cosmos DB.

Para más información, consulte:

* [Advanced Threat Protection para Azure Cosmos DB (versión preliminar)](../cosmos-db/cosmos-db-advanced-threat-protection.md)
* [La lista de alertas de protección contra amenazas para Azure Cosmos DB (versión preliminar)](alerts-reference.md#alerts-azurecosmos)



## <a name="threat-protection-for-other-microsoft-services"></a>Protección contra amenazas para otros servicios de Microsoft <a name="alerts-other"></a>

### <a name="threat-protection-for-azure-waf"></a>Protección contra amenazas para el firewall de aplicaciones web de Azure <a name="azure-waf"></a>

Firewall de aplicaciones web (WAF) es una característica de Azure Application Gateway que proporciona a las aplicaciones web una protección centralizada contra vulnerabilidades de seguridad comunes.

Las aplicaciones web son cada vez más el objetivo de ataques malintencionados que aprovechan vulnerabilidades habitualmente conocidas. El firewall de aplicaciones web de Application Gateway se basa en el conjunto de reglas básicas 3.0 o 2.2.9 de Open Web Application Security Project. El firewall de aplicaciones web se actualiza automáticamente para incluir protección frente a nuevas vulnerabilidades. 

Si tiene una licencia para el firewall de aplicaciones web de Azure, las alertas de este se transmitirán a Security Center sin necesidad de configuración adicional. Para más información sobre las alertas generadas por WAF, consulte [Reglas y grupos de reglas de CRS del firewall de aplicaciones web](../web-application-firewall/ag/application-gateway-crs-rulegroups-rules.md?tabs=owasp31#crs911-31).


### <a name="threat-protection-for-azure-ddos-protection"></a>Protección contra amenazas para Azure DDoS Protection <a name="azure-ddos"></a>

Los ataques de denegación de servicio distribuido (DDoS) son conocidos por lo fáciles que son de ejecutar. Se han convertido en un problema de seguridad muy importante, especialmente si va a trasladar sus aplicaciones a la nube. 

Un ataque DDoS intenta agotar los recursos de una aplicación haciendo que esta no esté disponible para los usuarios legítimos. Los ataques DDoS pueden dirigirse a cualquier punto de conexión accesible a través de Internet.

Para defenderse contra ataques DDoS, adquiera una licencia de Azure DDoS Protection y asegúrese de que está siguiendo los procedimientos recomendados de diseño de aplicaciones. DDoS Protection proporciona distintos niveles de servicio. Para más información, consulte [Introducción a Azure DDoS Protection](https://docs.microsoft.com/azure/virtual-network/ddos-protection-overview).

Para obtener una lista de las alertas de Azure DDoS Protection, consulte la [Tabla de referencia de alertas](alerts-reference.md#alerts-azureddos).


## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de las alertas de seguridad de estas características de protección contra amenazas, consulte los siguientes artículos:

* [Tabla de referencia para todas las alertas de Azure Security Center](alerts-reference.md)
* [Alertas de seguridad en el Centro de seguridad de Azure](security-center-alerts-overview.md)
* [Administración y respuesta a alertas de seguridad en Azure Security Center](security-center-managing-and-responding-alerts.md)
* [Exportar alertas y recomendaciones de seguridad (versión preliminar)](continuous-export.md)