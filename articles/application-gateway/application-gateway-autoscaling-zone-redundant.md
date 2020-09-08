---
title: Escalabilidad automática y Application Gateway con redundancia de zona v2
description: En este artículo se presenta la SKU Standard_v2 y WAF_v2 de la aplicación de Azure, que incluye características de escalabilidad automática y redundancia de zona.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: conceptual
ms.date: 06/06/2020
ms.author: victorh
ms.custom: fasttrack-edit, references_regions
ms.openlocfilehash: dc3daf28a4e8dd4ebf1fcedddd1a46986ac80cc4
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400609"
---
# <a name="autoscaling-and-zone-redundant-application-gateway-v2"></a>Escalabilidad automática y Application Gateway con redundancia de zona v2 

Application Gateway está disponible en una SKU Standard_v2. El firewall de aplicaciones web (WAF) está disponible en una SKU WAF_v2. La SKU v2 ofrece mejoras de rendimiento y agrega compatibilidad con nuevas características esenciales, como el escalado automático, la redundancia de zona y las IP virtuales estáticas. La nueva SKU v2 sigue admitiendo las características existentes en la SKU Standard y WAF, salvo algunas excepciones que se indican en la sección de [comparación](#differences-from-v1-sku).

La nueva SKU v2 incluye las siguientes mejoras:

- **Escalabilidad automática**: Las implementaciones de Application Gateway o WAF en la SKU de escalado automática pueden escalarse o reducirse verticalmente en función de los cambiantes patrones de la carga de tráfico. La escalabilidad automática también elimina el requisito de tener elegir un tamaño de implementación o un número de instancias durante el aprovisionamiento. Esta SKU ofrece verdadera elasticidad. En la SKU Standard_v2 y WAF_v2, Application Gateway puede funcionar con una capacidad fija (escalabilidad automática deshabilitada) y en modo de escalabilidad automática habilitada. El modo de capacidad fija es útil para escenarios con cargas de trabajo coherentes y predecibles. El modo de escalabilidad automática es útil en aplicaciones que tendrán variación en el tráfico de las aplicaciones.
- **Redundancia de zona**: una implementación de Application Gateway o WAF puede abarcar varias zonas de disponibilidad, lo que elimina la necesidad de aprovisionar instancias de Application Gateway independientes en cada zona con una instancia de Traffic Manager. Puede elegir una o varias zonas donde se implementarán las instancias de Application Gateway, lo que mejora la resistencia ante errores de zona. El grupo de back-end de las aplicaciones se puede distribuir de manera similar entre las zonas de disponibilidad.

  La redundancia de zona están disponibles solo donde estén disponibles las zonas de Azure. En otras regiones, se admiten todas las demás características. Para obtener más información, consulte [Regiones y zonas de disponibilidad en Azure](../availability-zones/az-overview.md)
- **VIP estática**: la SKU v2 de Application Gateway admite ahora exclusivamente el tipo de IP virtual estática. Con esto se garantiza que la IP virtual asociada a la puerta de enlace de aplicaciones no cambia durante la vigencia de la implementación, ni siquiera tras un reinicio.  No hay una IP virtual estática en la v1, por lo que debe usar la URL de la puerta de enlace de aplicaciones en lugar de la dirección IP para el enrutamiento del nombre de dominio a App Services a través de la puerta de enlace de aplicaciones.
- **Reescritura de encabezados**: Application Gateway permite agregar, quitar o actualizar los encabezados de solicitud y respuesta HTTP con la SKU v2. Para más información, consulte [Rescritura de encabezados HTTP con Application Gateway](rewrite-http-headers.md).
- **Integración de Key Vault**: Application Gateway v2 admite la integración con Key Vault para certificados de servidor adjuntos a clientes de escucha con HTTPS habilitado. Para obtener más información, consulte [Terminación TLS con certificados de Key Vault](key-vault-certs.md).
- **Controlador de entrada de Azure Kubernetes Service (AKS)** : El controlador de entrada de Application Gateway v2 permite que Azure Application Gateway se utilice como entrada para una instancia de Azure Kubernetes Service (AKS) llamada clúster AKS. Para más información, consulte [¿Qué es el controlador de entrada de Application Gateway?](ingress-controller-overview.md)
- **Mejoras de rendimiento**: la SKU v2 ofrece un rendimiento cinco veces superior en la descarga de TLS en comparación con la SKU Standard/WAF.
- **Menores tiempos de implementación y actualización**: la SKU v2 proporciona menores tiempos de implementación y actualización en comparación con la SKU Standard/WAF. Esto también incluye los cambios de configuración en WAF.

![Diagrama de zona de ajuste de escala automático.](./media/application-gateway-autoscaling-zone-redundant/application-gateway-autoscaling-zone-redundant.png)

## <a name="supported-regions"></a>Regiones admitidas

La SKU Standard_v2 y WAF_v2 está disponible en las siguientes regiones: Centro-norte de EE. UU., Centro-sur de EE. UU., Oeste de EE. UU., Oeste de EE. UU. 2, Este de EE. UU., Este de EE. UU. 2, Centro de EE. UU., Norte de Europa, Oeste de Europa, Sudeste de Asia, Centro de Francia, Oeste de Reino Unido, Este de Japón, Oeste de Japón, Este de Australia, Sudeste de Australia, Sur de Brasil, Centro de Canadá, Este de Canadá, Este de Asia, Centro de Corea del Sur, Sur de Corea del Sur, Sur de Reino Unido, Centro de la India, Oeste de la India, Sur de la India.

## <a name="pricing"></a>Precios

Con la SKU v2, el modelo de precios está orientado al consumo y ya no está asociado a recuentos de instancias o tamaños. El modelo de precios de la SKU v2 presenta dos componentes:

- **Precio fijo**: precio por hora (o por fracción de hora) para aprovisionar una instancia de Gateway Standard_v2 o WAF_v2. Tenga en cuenta que 0 instancias mínimas adicionales todavía garantizan una alta disponibilidad del servicio que siempre se incluye con el precio fijo.
- **Precio por unidad de capacidad**: se trata del costo basado en el consumo que se suma al costo fijo. La unidad de capacidad se cobra también por hora o fracciones de hora de proceso. Hay tres dimensiones para la unidad de capacidad: unidad de proceso, conexiones persistentes y rendimiento. La unidad de proceso es una medida de la capacidad de procesador consumida. Los factores que afectan a la unidad de proceso son las conexiones TLS/seg, los cálculos de reescritura de direcciones URL y el procesamiento de reglas de WAF. La conexión persistente es una medida de las conexiones TCP establecidas para la puerta de enlace de aplicaciones en un intervalo de facturación determinado. El rendimiento es el promedio de megabits/seg procesados por el sistema en un intervalo de facturación determinado.  La facturación se realiza en un nivel de unidad de capacidad para cualquier elemento que esté por encima del número de instancias reservadas.

Cada unidad de capacidad se compone a lo sumo de: 1 unidad de proceso, 2500 conexiones persistentes y rendimiento de 2,22 Mbps.

Para obtener más información, consulte [Explicación de los precios](understanding-pricing.md).

## <a name="scaling-application-gateway-and-waf-v2"></a>Escalado de Application Gateway y WAF v2

Application Gateway and WAF pueden configurarse para escalarse de dos maneras:

- **Escalado automático**: con el escalado automático habilitado, las SKU v2 de Application Gateway y WAF se escalan o reducen verticalmente en función de los requisitos del tráfico de la aplicación. Este modo ofrece una mayor elasticidad a su aplicación y elimina la necesidad de adivinar el tamaño de la puerta de enlace de aplicaciones o el número de instancias. Este modo también le permite ahorrar costos al no requerir que se ejecute la puerta de enlace a una capacidad máxima de aprovisionamiento para una carga de tráfico máxima prevista. Debe especificar un número de instancias mínimo y opcionalmente máximo. La capacidad mínima garantiza que Application Gateway y WAF v2 no caigan por debajo del número mínimo de instancias especificado, incluso en ausencia de tráfico. Cada instancia es aproximadamente equivalente a 10 unidades de capacidad reservada adicionales. Cero significa que no hay capacidad reservada y es por naturaleza estrictamente de escalado automático. Opcionalmente, también puede especificar un número máximo de instancias, lo que garantiza que Application Gateway no se amplíe más allá del número de instancias especificado. Solo se le facturará por la cantidad de tráfico que atienda el servicio Gateway. Los números de instancias pueden oscilar entre 0 y 125. El valor predeterminado para el número máximo de instancias es 20 si no se especifica.
- **Manual**: también puede elegir el modo Manual para que la puerta de enlace no se escale automáticamente. En este modo, si hay más tráfico del que puede asumir Application Gateway o WAF, podría perderse tráfico. Con el modo manual, es obligatorio especificar el número de instancias. El número de instancias puede variar de 1 a 125 instancias.

## <a name="autoscaling-and-high-availability"></a>Escalado automático y alta disponibilidad

Las instancias de Azure Application Gateway siempre se implementan para ofrecer alta disponibilidad. El servicio se ha diseñado a partir de varias instancias que se crean según se configuran (si el escalado automático está deshabilitado) o según lo requiere la carga de la aplicación (si el escalado automático está habilitado). Tenga en cuenta que, desde el punto de vista del usuario, no es necesario que tenga visibilidad de las instancias individuales, ya que tendrá suficiente con ver el servicio Application Gateway en su conjunto. Si una determinada instancia tiene un problema y deja de funcionar, Azure Application Gateway creará una nueva de forma transparente.

Tenga en cuenta que, aunque configure el escalado automático sin establecer ninguna instancia mínima, el servicio seguirá siendo de alta disponibilidad, ya que esta capacidad siempre se incluye con el precio fijo.

Sin embargo, la creación de una nueva instancia puede tardar algún tiempo (aproximadamente seis o siete minutos). Por lo tanto, si no quiere tener que abordar este tiempo de inactividad, puede configurar un recuento de instancias mínimo de 2, idealmente con la compatibilidad con la zona de disponibilidad. De este modo, en circunstancias normales tendrá al menos dos instancias en Azure Application Gateway, por lo que, si una de ellas tuviera un problema, la otra intentará gestionar el tráfico durante el tiempo en que se cree una nueva. Tenga en cuenta que una instancia de Azure Application Gateway puede admitir unas 10 unidades de capacidad, por lo que, en función de la cantidad de tráfico que tenga normalmente, es posible que quiera establecer la configuración de escalado automático de instancia mínima en un valor superior a 2.

## <a name="feature-comparison-between-v1-sku-and-v2-sku"></a>Comparación de características entre SKU v1 y SKU v2

En la tabla siguiente se comparan las características disponibles con cada SKU.

| Característica                                           | SKU v1   | SKU v2   |
| ------------------------------------------------- | -------- | -------- |
| Escalado automático                                       |          | &#x2713; |
| Redundancia de zona                                   |          | &#x2713; |
| IP virtual estática                                        |          | &#x2713; |
| Controlador de entrada de Azure Kubernetes Service (AKS) |          | &#x2713; |
| Integración de Azure Key Vault                       |          | &#x2713; |
| Reescritura de encabezados HTTP(S)                           |          | &#x2713; |
| Enrutamiento basado en dirección URL                                 | &#x2713; | &#x2713; |
| Hospedaje de varios sitios                             | &#x2713; | &#x2713; |
| Redireccionamiento de tráfico                               | &#x2713; | &#x2713; |
| Firewall de aplicaciones web (WAF)                    | &#x2713; | &#x2713; |
| Reglas personalizadas del firewall de aplicaciones web                                  |          | &#x2713; |
| Terminación de la Seguridad de la capa de transporte (TLS) o Capa de sockets seguros (SSL)            | &#x2713; | &#x2713; |
| Cifrado TLS de un extremo a otro                         | &#x2713; | &#x2713; |
| Afinidad de sesión                                  | &#x2713; | &#x2713; |
| Páginas de error personalizadas                                | &#x2713; | &#x2713; |
| Compatibilidad con WebSocket                                 | &#x2713; | &#x2713; |
| Compatibilidad con HTTP/2                                    | &#x2713; | &#x2713; |
| Purga de la conexión                               | &#x2713; | &#x2713; |

> [!NOTE]
> La SKU v2 de escalado automático ahora admite [sondeos de estado predeterminados](application-gateway-probe-overview.md#default-health-probe) para supervisar automáticamente el estado de todos los recursos en su grupo de back-end y resaltar aquellos miembros de back-end que se considere que no están en buen estado. El sondeo de estado predeterminado se configura automáticamente para back-ends que no tienen ninguna configuración de sondeo personalizado. Para obtener más información, consulte [Información general sobre la supervisión de estado de la puerta de enlace de aplicaciones](application-gateway-probe-overview.md).

## <a name="differences-from-v1-sku"></a>Diferencias respecto a la SKU v1

En esta sección se describen las características y limitaciones de la SKU v2 que difieren de la SKU v1.

|Diferencia|Detalles|
|--|--|
|Certificado de autenticación|No compatible.<br>Para más información, consulte [Introducción a TLS de un extremo a otro con Application Gateway](ssl-overview.md#end-to-end-tls-with-the-v2-sku).|
|Combinación de Application Gateway Standard y Standard_v2 en la misma subred|No compatible|
|Ruta definida por el usuario (UDR) en la subred de Application Gateway|Se admite (escenarios específicos). En versión preliminar.<br> Para obtener más información sobre los escenarios admitidos, consulte [Introducción a la configuración de Application Gateway](configuration-overview.md#user-defined-routes-supported-on-the-application-gateway-subnet).|
|NSG para el intervalo de puertos de entrada| - 65200 a 65535 para la SKU Standard_v2<br>- 65503 a 65534 para la SKU Standard.<br>Para más información, consulte las [preguntas más frecuentes](application-gateway-faq.md#are-network-security-groups-supported-on-the-application-gateway-subnet).|
|Registros de rendimiento en Azure Diagnostics|No compatible.<br>Se deben usar las métricas de Azure.|
|Facturación|La facturación está programada para que comience el 1 de julio de 2019.|
|Modo FIPS|Actualmente no se admiten.|
|Solo modo ILB|Actualmente no se admite. Se admiten los modos público e ILB juntos.|
|Integración de Net Watcher|No compatible.|
|Integración en Azure Security Center|No disponible todavía.

## <a name="migrate-from-v1-to-v2"></a>Migración de v1 a v2

Hay un script de Azure PowerShell disponible en la galería de PowerShell para ayudarle a migrar desde la v1 de Application Gateway/WAF al SKU de escalado automático v2. Este script le ayuda a copiar la configuración de la puerta de enlace v1. La migración del tráfico sigue siendo su responsabilidad. Para más información, consulte [Migración de Azure Application Gateway de v1 a v2](migrate-v1-v2.md).

## <a name="next-steps"></a>Pasos siguientes

- [Inicio rápido: Dirección del tráfico web con Azure Application Gateway: Azure Portal](quick-create-portal.md)
- [Creación de una puerta de enlace de aplicaciones con redundancia de zona, escalabilidad automática y una dirección IP virtual reservada con Azure PowerShell](tutorial-autoscale-ps.md)
- Más información acerca de [Application Gateway](overview.md).
- Más información acerca de [Azure Firewall](../firewall/overview.md).
