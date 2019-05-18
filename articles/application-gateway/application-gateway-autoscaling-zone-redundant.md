---
title: El escalado automático y la puerta de enlace de aplicaciones con redundancia de zona en Azure
description: Este artículo presentan los Standard_v2 de aplicación de Azure y SKU WAF_v2, que incluye características de escalado automático y con redundancia de zona.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 5/16/2019
ms.author: victorh
ms.openlocfilehash: 0da5d8a3eec0faa4001ccf229c6748c253f1b6e5
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827434"
---
# <a name="autoscaling-and-zone-redundant-application-gateway"></a>El escalado automático y la puerta de enlace de aplicaciones con redundancia de zona 

Application Gateway y Firewall de aplicaciones Web (WAF) también están disponibles en una SKU de WAF_v2 y Standard_v2. La SKU v2 ofrece mejoras de rendimiento y agrega compatibilidad con nuevas características esenciales, como el escalado automático, redundancia de zona y soporte técnico para VIP estáticas. Siguen siendo compatibles en la nueva SKU v2, con algunas excepciones que se muestran en las características existentes en el estándar y la SKU de WAF [comparación](#differences-with-v1-sku) sección.

La nueva SKU v2 incluye las siguientes mejoras:

- **Escalabilidad automática**: Las implementaciones de Application Gateway o WAF en la SKU de escalado automática pueden escalarse o reducirse verticalmente en función de los cambiantes patrones de la carga de tráfico. La escalabilidad automática también elimina el requisito de tener elegir un tamaño de implementación o un número de instancias durante el aprovisionamiento. Esta SKU ofrece elasticidad es true. En la SKU de WAF_v2 y Standard_v2, Application Gateway puede operar en capacidad fija (el escalado automático deshabilitado) y en modo de escalado automático habilitado. El modo de capacidad fija es útil para escenarios con cargas de trabajo coherentes y predecibles. Modo de escalado automático es útil en aplicaciones que se vea varianza en el tráfico de la aplicación.
- **Redundancia de zona**: Una implementación de WAF o Application Gateway puede abarcar varias zonas de disponibilidad, eliminando la necesidad de aprovisionar instancias independientes de Application Gateway en cada zona con un administrador de tráfico. Puede elegir una sola zona o varias zonas donde se implementan instancias de Application Gateway, que hace sea más resistente a errores de zona. El grupo de back-end de las aplicaciones se puede distribuir de manera similar entre las zonas de disponibilidad.

  Redundancia de zona está disponible solo en las zonas de Azure están disponibles. En otras regiones, se admiten todas las demás características. Para obtener más información, consulte [¿cuáles son las zonas de disponibilidad en Azure?](../availability-zones/az-overview.md#services-support-by-region)
- **VIP estática**: Application gateway admite la SKU v2 la VIP estática de tipo exclusivamente. Esto garantiza que la VIP asociada con application gateway no cambia para el ciclo de vida de la implementación, incluso después del reinicio.
- **Reescritura de encabezado**: Instancia de Application Gateway permite agregar, quitar o actualizar los encabezados de solicitud y respuesta HTTP con SKU v2. Para obtener más información, consulte [encabezados HTTP reescribir con Application Gateway](rewrite-http-headers.md)
- **Integración de Key Vault (versión preliminar)**: V2 de la puerta de enlace de aplicaciones admite la integración con Key Vault (en versión preliminar pública) para los certificados de servidor que se adjuntan a los agentes de escucha HTTPS habilitado. Para obtener más información, consulte [terminación SSL con certificados de Key Vault](key-vault-certs.md).
- **Controlador de entrada de servicio (versión preliminar) de Azure Kubernetes**: El controlador de entrada de v2 Application Gateway permite que la puerta de enlace de aplicaciones de Azure que se usará como la entrada para una Azure Kubernetes Service (AKS) se conoce como clúster de AKS. Para obtener más información, consulte el [página de documentación](https://azure.github.io/application-gateway-kubernetes-ingress/).
- **Mejoras de rendimiento**: La versión 2 SKU ofrece hasta 5 X a mejor SSL la descarga de rendimiento en comparación con la SKU estándar y WAF.
- **Tiempos de implementación y actualización** la SKU v2 proporciona tiempos de implementación y actualización en comparación con la SKU estándar y WAF. Esto también incluye los cambios de configuración de WAF.

![](./media/application-gateway-autoscaling-zone-redundant/application-gateway-autoscaling-zone-redundant.png)

## <a name="supported-regions"></a>Regiones admitidas

La SKU de WAF_v2 y Standard_v2 está disponible en las siguientes regiones: Centro-norte de EE. UU., Centro-sur de EE. UU., Oeste de EE. UU., Oeste de EE. UU. 2, Este de EE. UU., Este de EE. UU. 2, Centro de EE. UU., Europa del Norte, Europa Occidental, Sudeste Asiático, Centro de Francia, Oeste de Reino Unido, Este de Japón, Oeste de Japón. En el futuro se agregará más regiones.

## <a name="pricing"></a>Precios

Con la SKU v2, el modelo de precios está controlado por el consumo y ya no está asociado a recuentos de instancias o tamaños. Los precios de SKU v2 tienen dos componentes:

- **Precio fijo** -esto es cada hora (o fracciones de hora) precio para aprovisionar un Standard_v2 o WAF_v2 puerta de enlace.
- **Precio unitario capacidad** -se trata de un costo basado en consumo que se cobra adicionales al costo fijo. Cargo de unidad de capacidad también se calcula por hora o parcial cada hora. Existen tres dimensiones a la unidad de capacidad, rendimiento, las conexiones persistentes y unidad de proceso. Calcular la unidad es una medida de capacidad de procesador consumida. Factores que afectan a la unidad de proceso son las conexiones TLS/seg., los cálculos de reescritura de direcciones URL y procesamiento de reglas de WAF. Conexión persistente es una medida de las conexiones TCP establecidas para la puerta de enlace de aplicaciones en un intervalo de facturación determinado. El rendimiento es medio Megabits por segundo procesadas por el sistema en un intervalo de facturación determinado.

Cada unidad de capacidad a lo sumo se compone de: 1 unidad, o las conexiones persistentes de 2500 o 2.22 Mbps de rendimiento de proceso.

Orientación de la unidad de proceso:

- **Standard_v2** -cada unidad de proceso es capaz de aproximadamente 50 conexiones por segundo con certificado TLS de la clave RSA 2048 bits.
- **WAF_v2** : cada unidad puede admitir aproximadamente 10 solicitudes simultáneas por segundo de solicitudes de combinación de 70-30% del tráfico con un 70% menos de 2 KB GET/POST de proceso y restante más alto. Rendimiento de WAF no se ve afectado por el tamaño de respuesta en actualmente.

> [!NOTE]
> Cada instancia actualmente puede admitir aproximadamente 10 unidades de capacidad.
> El número de solicitudes que puede controlar una unidad de proceso depende de diversos criterios, como el tamaño de clave de certificado TLS, el algoritmo de intercambio de claves, las operaciones de reescritura de encabezado y en el caso de tamaño de la solicitud entrante de WAF. Se recomienda que realizar pruebas de aplicación para determinar la tasa de solicitud por unidad de proceso. Unidad de capacidad y unidad de proceso estará disponibles como una métrica antes de la facturación se inicia.

**Precio en el este de EE. UU.**:

|              Nombre de SKU                             | Precio fijo ($/ h)  | Precio unitario de capacidad (CU-$/ h)   |
| ------------------------------------------------- | ------------------- | ------------------------------- |
| Standard_v2                                       |    0.20             | 0.0080                          |
| WAF_v2                                            |    0.36             | 0.0144                          |

Para obtener más información sobre precios, consulte el [página de precios](https://azure.microsoft.com/pricing/details/application-gateway/). La facturación está programada para que comience el 1 de julio de 2019.

**Ejemplo 1**

Se aprovisiona un Standard_v2 de puerta de enlace de aplicaciones sin el escalado automático en el modo de escalado manual con capacidad fija de cinco instancias.

Precio fijo = 744(hours) * 0,20 USD = $148.8 <br>
Unidades de capacidad = unidad de capacidad (horas) 10 744 por instancia * cinco instancias * $0.008 por hora de unidad de capacidad = $297.6

Precio total = $148.8 + $297.6 = $446.4

**Ejemplo 2**

Se aprovisiona un standard_v2 Application Gateway para un mes y durante este tiempo recibe 25 nuevas SSL conexiones por segundo, promedio de transferencia de datos de 8.88 Mbps. Suponiendo que las conexiones tienen una duración breve, el precio sería:

Precio fijo = 744(hours) * 0,20 USD = $148.8

Precio unitario de capacidad = 744(hours) * Max (unidad de proceso de 25/50 conexiones por segundo, unidad de capacidad de 8.88/2.22 de rendimiento) * $0.008 = 744 * 4 * 0.008 = $23.81

Precio total = $148. 23.81 8 + = $172.61

**Ejemplo 3**

Se aprovisiona un WAF_v2 de puerta de enlace de aplicación durante un mes. Durante este tiempo, recibe el 25 nuevas SSL conexiones por segundo, promedio de transferencia de datos de 8.88 Mbps y 80 solicitudes por segundo. Suponiendo que las conexiones son de cortas duración, y que el cálculo de unidad de proceso para la aplicación es compatible con 10 solicitudes por segundo por unidad de proceso, el precio sería:

Precio fijo = 744(hours) * $0,36 = $267.84

Precio unitario de capacidad = 744(hours) * Max (proceso unidad Max(25/50 for connections/sec, 80/10 WAF RPS), unidad de capacidad de 8.88/2.22 de rendimiento) * $0.0144 = 744 * 8 * 0.0144 = $85.71

Precio total = $267.84 + $85.71 = $353.55

El [página de precios](https://azure.microsoft.com/pricing/details/application-gateway/) se actualizará para reflejar los precios regionales en 14 de mayo de 2019. La facturación está programada para que comience el 1 de junio de 2019.

## <a name="scaling-application-gateway-and-waf-v2"></a>Escalado v2 Application Gateway y WAF

WAF y Application Gateway pueden configurarse a escala en dos modos:

- **El escalado automático** : con el escalado automático habilitado, la puerta de enlace de aplicaciones y WAF v2 SKU escalar o reducir verticalmente en función de los requisitos de tráfico de aplicación. Este modo ofrece elasticidad mejor a su aplicación y elimina la necesidad de adivinar el recuento de tamaño o la instancia de puerta de enlace de aplicaciones. Este modo también le permite ahorrar costos al no requerir para ejecutar las puertas de enlace en su capacidad máxima aprovisionada para la carga de tráfico máximo previsto. Los clientes deben especificar un recuento de instancias mínimas y máximas opcionalmente. Capacidad mínima que se garantiza que Application Gateway y WAF v2 no se encuentran por debajo del recuento de instancias mínimo especificado, incluso en ausencia de tráfico. Se le facturará esta capacidad mínima incluso en ausencia de todo el tráfico. Opcionalmente, también puede especificar un recuento máximo de instancias, lo que garantiza que la puerta de enlace de aplicaciones no escala más allá del número especificado de instancias. Seguirá facturando para la cantidad de tráfico atendido por la puerta de enlace. Los números de instancias pueden oscilar entre 0 y 125. El valor predeterminado de recuento de instancias máximo es 20 si no se especifica.
- **Manual** -o bien puede elegir que no se escalará automáticamente la puerta de enlace de modo Manual. En este modo, si hay más tráfico que lo que Application Gateway o WAF puede administrar, podría provocar pérdida de tráfico. Con el modo manual, es obligatorio especificar el número de instancia. Recuento de instancias puede variar de 1 a 125 instancias.

## <a name="feature-comparison-between-v1-sku-and-v2-sku"></a>Comparación de características entre SKU v1 y v2 SKU

En la tabla siguiente se compara las características disponibles con cada SKU.

|                                                   | v1 SKU   | v2 SKU   |
| ------------------------------------------------- | -------- | -------- |
| Escalado automático                                       |          | &#x2713; |
| Redundancia de zona                                   |          | &#x2713; |
| VIP estático                                        |          | &#x2713; |
| Azure controlador Ingress de Kubernetes Service (AKS) |          | &#x2713; |
| Integración de Azure Key Vault                       |          | &#x2713; |
| Vuelva a escribir los encabezados HTTP (S)                           |          | &#x2713; |
| Enrutamiento basado en dirección URL                                 | &#x2713; | &#x2713; |
| Hospedaje de varios sitios                             | &#x2713; | &#x2713; |
| Redireccionamiento del tráfico                               | &#x2713; | &#x2713; |
| Firewall de aplicaciones web (WAF)                    | &#x2713; | &#x2713; |
| Terminación de la Capa de sockets seguros (SSL)            | &#x2713; | &#x2713; |
| Cifrado SSL de extremo a otro                         | &#x2713; | &#x2713; |
| Afinidad de sesión                                  | &#x2713; | &#x2713; |
| Páginas de error personalizadas                                | &#x2713; | &#x2713; |
| Compatibilidad con WebSocket                                 | &#x2713; | &#x2713; |
| Compatibilidad con HTTP/2                                    | &#x2713; | &#x2713; |
| Purga de conexión                               | &#x2713; | &#x2713; |

> [!NOTE]
> La versión 2 de escalado automático ahora es compatible con la SKU [sondeos de estado predeterminada](application-gateway-probe-overview.md#default-health-probe) para supervisar el estado de todos los recursos de su grupo de back-end y resaltar los miembros de back-end que se consideran en mal estado automáticamente. El sondeo de estado predeterminada se configura automáticamente para el back-ends que no tienen ninguna configuración de sondeo personalizado. Para obtener más información, consulte [sondeos de estado de application gateway](application-gateway-probe-overview.md).

## <a name="differences-with-v1-sku"></a>Diferencias con SKU v1

|Diferencia|Detalles|
|--|--|
|Certificado de autenticación|No compatible.<br>Para más información, consulte [Introducción a SSL de un extremo a otro con Application Gateway](ssl-overview.md#end-to-end-ssl-with-the-v2-sku).|
|Combinación de Application Gateway Standard y Standard_v2 en la misma subred|No compatible|
|Ruta definida por el usuario (UDR) en la subred de Application Gateway|No compatible|
|NSG para el intervalo de puertos de entrada| - 65200 a 65535 para la SKU Standard_v2<br>- 65503 a 65534 para la SKU Standard.<br>Para más información, consulte las [preguntas más frecuentes](application-gateway-faq.md#are-network-security-groups-supported-on-the-application-gateway-subnet).|
|Registros de rendimiento en Azure Diagnostics|No compatible.<br>Se deben usar las métricas de Azure.|
|Facturación|Facturación programada para iniciarse en el 1 de junio de 2019.|
|Modo FIPS|Actualmente no se admiten.|
|Solo modo ILB|Actualmente no se admite. Se admiten los modos público e ILB juntos.|
|Integración de Netwatcher|No compatible.|
|Integración del centro de soporte técnico de Azure|No disponible todavía.

## <a name="next-steps"></a>Pasos siguientes

- [Inicio rápido: Tráfico de web Direct con Azure Application Gateway mediante Azure portal](quick-create-portal.md)
- [Creación de una puerta de enlace de aplicaciones con redundancia de zona, escalabilidad automática y una dirección IP virtual reservada con Azure PowerShell](tutorial-autoscale-ps.md)
- Más información acerca de [Application Gateway](overview.md).
- Más información acerca de [Azure Firewall](../firewall/overview.md).