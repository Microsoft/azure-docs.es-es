---
title: Novedades de Azure Application Gateway
description: Obtenga información acerca de las novedades de Azure Application Gateway, como, por ejemplo, las notas de la versión más recientes, los problemas conocidos, las correcciones de errores, las funcionalidades en desuso y los próximos cambios.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: overview
ms.date: 06/10/2020
ms.author: victorh
ms.openlocfilehash: 6ea0b28f5e0f23e0f8bb30e8fe2b0f792bce0709
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287450"
---
# <a name="whats-new-in-azure-application-gateway"></a>Novedades de Azure Application Gateway

Azure Application Gateway se actualiza de forma continuada. Para mantenerse al día con los avances más recientes, este artículo proporciona información acerca de los elementos siguientes:

- Versiones más recientes
- Problemas conocidos
- Corrección de errores
- Funciones obsoletas

## <a name="new-features"></a>Nuevas características

|Característica  |Descripción  |Fecha de adición  |
|---------|---------|---------|
| Nombres de host con el carácter comodín en los clientes de escucha (versión preliminar) | Ahora puede definir nombres de host con el carácter comodín en un cliente de escucha de varios sitios y hasta 5 nombres de host por cliente de escucha. Para obtener más información, consulte los [nombres de host comodín en el cliente de escucha (versión preliminar)](multiple-site-overview.md#wildcard-host-names-in-listener-preview). | Julio de 2020 |
| Reescritura de direcciones URL (versión preliminar) | Ahora puede volver a escribir la ruta de la dirección URL y los parámetros de la cadena de consulta mediante la reescritura de direcciones URL. Consulte [Reescritura de encabezados HTTP y de direcciones URL](rewrite-http-headers-url.md) para más información y consulte [Reescritura de direcciones URL en Application Gateway con Azure Portal](rewrite-url-portal.md) para una guía paso a paso. | Julio de 2020 |
| Puerto personalizado para sondeos de estado | El SKU de Application Gateway v2 ofrece ahora la posibilidad de proporcionar un puerto personalizado en la configuración de sondeo de estado. Para más información, consulte la [introducción a los sondeos de estado](application-gateway-probe-overview.md). | Julio de 2020 |
| Complemento de AKS del controlador de entrada de Application Gateway (AGIC) (versión preliminar) |El controlador de entrada de Application Gateway ahora se puede implementar como un complemento de AKS nativo en una línea mediante la CLI de Azure. El hecho de ser un complemento de AKS permite que AGIC se convierta en un servicio totalmente administrado, aunque se siga ejecutando en el clúster de AKS del cliente. Para más información, consulte [Diferencias del complemento de AGIC](ingress-controller-overview.md#difference-between-helm-deployment-and-aks-add-on). |Junio de 2020 |
| Rutas definidas por el usuario (UDR) en v2 (versión preliminar) |Ahora se admiten las rutas definidas por el usuario en algunos escenarios en las SKU de Application Gateway V2. Para más información, consulte [Introducción a la configuración de Application Gateway](configuration-overview.md#user-defined-routes-supported-on-the-application-gateway-subnet). |Marzo de 2020 |
|Cambios en la cookie de afinidad |Cuando está habilitada la afinidad basada en cookies, Application Gateway inserta otra cookie idéntica llamada *ApplicationGatewayAffinityCORS* además de la cookie ApplicationGatewayAffinity existente. *ApplicationGatewayAffinityCORS* tiene dos atributos más agregados (*SameSite = None; Secure*) para que se mantengan las sesiones persistentes, incluso para las solicitudes entre orígenes. Para más información consulte la sección sobre [afinidad basada en cookies de Application Gateway ](configuration-overview.md#cookie-based-affinity). |Febrero de 2020 |
|Mejoras de sondeo |Con las mejoras de sondeo personalizadas de la SKU de Application Gateway v2, hemos simplificado la [configuración del sondeo](https://docs.microsoft.com/azure/application-gateway/application-gateway-create-probe-portal#create-probe-for-application-gateway-v2-sku), facilitado las [pruebas de mantenimiento de back-end a petición](https://docs.microsoft.com/azure/application-gateway/application-gateway-create-probe-portal#test-backend-health-with-the-probe) y agregado [más información de diagnóstico](https://docs.microsoft.com/azure/application-gateway/application-gateway-backend-health-troubleshooting#error-messages) para ayudarle a solucionar los problemas de mantenimiento de back-end.  |Octubre de 2019 |
|Más métricas |Hemos agregado las siguientes métricas nuevas para ayudarle a supervisar la SKU de Application Gateway v2: [Métricas relacionadas con el control de tiempo](https://docs.microsoft.com/azure/application-gateway/application-gateway-metrics#timing-metrics), estado de respuesta de back-end, bytes recibidos, bytes enviados, protocolo TLS de cliente y unidades de proceso actuales. Consulte las [Métricas compatibles con la SKU de Application Gateway V2](https://docs.microsoft.com/azure/application-gateway/application-gateway-metrics#metrics-supported-by-application-gateway-v2-sku). |Agosto de 2019 |
|Reglas personalizadas del firewall de aplicaciones web |Application Gateway WAF_v2 ya admite la creación de reglas personalizadas. Consulte [Reglas personalizadas de Application Gateway](custom-waf-rules-overview.md). |Junio de 2019 |
|Escalado automático, redundancia de zona, disponibilidad general del soporte técnico de VIP estática |Disponibilidad general de SKU v2, que admite el escalabilidad automática, la redundancia de zona, la mejora del rendimiento, las VIP estáticas, Key Vault y la reescritura de encabezados. Vea la [documentación sobre el escalado automático de Application Gateway](application-gateway-autoscaling-zone-redundant.md). |Abril de 2019 |
|Integración de Key Vault |Application Gateway ahora admite la integración con Key Vault (en versión preliminar pública) para certificados de servidor que se adjuntan a los clientes de escucha con HTTPS habilitado. Vea [Terminación TLS con certificados de Key Vault](key-vault-certs.md). |Abril de 2019 |
|CRUD y reescrituras de encabezado     |Ya puede reescribir encabezados HTTP. Consulte [Tutorial: Crear una instancia de Application Gateway y reescribir los encabezados HTTP](tutorial-http-header-rewrite-powershell.md) para más información.|Diciembre de 2018|
|Configuración de WAF y lista de exclusión     |Hemos agregado más opciones para ayudarle a configurar el WAF y reducir los falsos positivos. Para más información consulte [Listas de exclusión y límites de tamaño de solicitud del firewall de aplicaciones web](application-gateway-waf-configuration.md).|Diciembre de 2018|
|Escalado automático, redundancia de zona y disponibilidad general del soporte técnico de VIP estática      |En la versión v2 de SKU hay muchas mejoras, como el escalado automático, un rendimiento mejorado, etc. Consulte [¿Qué es Azure Application Gateway?](overview.md) para más información.|Septiembre de 2018|
|Purga de la conexión     |Purga de conexión le permite eliminar correctamente los miembros de los grupos de back-end. Para más información, consulte: [Purga de conexión](features.md#connection-draining).|Septiembre de 2018|
|Páginas de error personalizadas     |Con páginas de errores personalizadas, puede crear una página de errores en el mismo formato que el resto de los sitios web. Para ello, consulte [Create Application Gateway custom error pages](custom-error.md) (Creación de páginas de errores personalizadas de Application Gateway).|Septiembre de 2018|
|Mejoras en las métricas     |Puede obtener una mejor visión del estado de Application Gateway con métricas mejoradas. Para habilitar las métricas en Application Gateway, consulte [Mantenimiento del back-end, registros de diagnóstico y métricas de Application Gateway](application-gateway-diagnostics.md).|Junio de 2018|

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Application Gateway, consulte [¿Qué es Azure Application Gateway?](overview.md)
