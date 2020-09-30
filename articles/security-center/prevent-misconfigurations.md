---
title: Evitar errores de configuración con Azure Security Center
description: Aprenda a usar las opciones "Aplicar" y "Denegar" de Security Center en las páginas de detalles de recomendaciones.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 09/08/2020
ms.author: memildin
ms.openlocfilehash: 19e9a33350b6a1a67986dc35a372f737e45ab39a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906388"
---
# <a name="prevent-misconfigurations-with-enforcedeny-recommendations"></a>Evitar errores de configuración con las recomendaciones Aplicar o Denegar

Los errores de configuración de seguridad son una causa importante de los incidentes de seguridad. Security Center ofrece ahora la posibilidad de ayudar a *evitar* errores de configuración de los nuevos recursos con respecto a recomendaciones específicas. 

Esta característica puede ayudar a mantener protegidas las cargas de trabajo y a estabilizar la puntuación segura.

La aplicación de una configuración segura, basada en una recomendación específica, se ofrece de dos modos:

- Con el efecto **Denegar** de Azure Policy, puede impedir que se creen recursos en mal estado.
- Con la opción **Aplicar**, puede sacar partido del efecto **DeployIfNotExist** de Azure Policy y corregir automáticamente los recursos no compatibles tras la creación.

Esta opción se puede encontrar en la parte superior de la página de detalles del recurso de las recomendaciones de seguridad seleccionadas (consulte [Recomendaciones con las opciones denegar o aplicar](#recommendations-with-denyenforce-options)).

## <a name="prevent-resource-creation"></a>Impedir la creación de recursos

1. Abra la recomendación que deben satisfacer los nuevos recursos y seleccione el botón **Denegar** en la parte superior de la página.

    :::image type="content" source="./media/security-center-remediate-recommendations/recommendation-deny-button.png" alt-text="Página de recomendaciones con el botón Denegar resaltado":::

    Se abre el panel de configuración con las opciones de ámbito. 

1. Establezca el ámbito seleccionando la suscripción o el grupo de administración en cuestión.

    > [!TIP]
    > Puede usar los tres puntos que hay al final de la fila para cambiar una única suscripción, o bien usar las casillas para seleccionar varias suscripciones o grupos y, luego, elegir **Change to Deny** (Cambiar a Denegar).

    :::image type="content" source="./media/security-center-remediate-recommendations/recommendation-prevent-resource-creation.png" alt-text="Establecimiento del ámbito de denegación de Azure Policy":::


## <a name="enforce-a-secure-configuration"></a>Aplicación de una configuración segura

1. Abra la recomendación para la que se implementará una plantilla si los nuevos recursos no la satisfacen y seleccione el botón **Aplicar** en la parte superior de la página.

    :::image type="content" source="./media/security-center-remediate-recommendations/recommendation-enforce-button.png" alt-text="Página de recomendaciones con el botón Aplicar resaltado":::

    Se abre el panel de configuración con todas las opciones de configuración de directiva. 

    :::image type="content" source="./media/security-center-remediate-recommendations/recommendation-enforce-config.png" alt-text="Aplicación de opciones de configuración":::

1. Establezca el ámbito, el nombre de la asignación y otras opciones pertinentes.

1. Seleccione **Revisar + crear**.

## <a name="recommendations-with-denyenforce-options"></a>Recomendaciones con las opciones denegar o aplicar

Estas recomendaciones se pueden usar con la opción **Denegar**:

- Se deben migrar las máquinas virtuales a nuevos recursos de Azure Resource Manager
- Se deben migrar las cuentas de almacenamiento a los nuevos recursos de Azure Resource Manager
- Todas las reglas de autorización excepto RootManageSharedAccessKey se deben eliminar del espacio de nombres del centro de eventos
- Todas las reglas de autorización excepto RootManageSharedAccessKey se deben eliminar del espacio de nombres de Service Bus
- Se debe habilitar la transferencia segura a las cuentas de almacenamiento
- Solo se deben habilitar las conexiones seguras a Redis Cache
- Las variables de cuenta de automatización deben cifrarse
- Los clústeres de Service Fabric solo deben usar Azure Active Directory para la autenticación de cliente
- Se debe establecer la propiedad ClusterProtectionLevel en EncryptAndSign en los clústeres de Service Fabric
- Auditar el acceso de red sin restricciones a cuentas de almacenamiento


Estas recomendaciones se pueden usar con la opción **Aplicar**:

- Los registros de diagnóstico de Logic Apps deben estar habilitados
- Los registros de diagnóstico de Data Lake Analytics deben estar habilitados
- Los registros de diagnóstico de IoT Hub deben estar habilitados
- Se deben habilitar los registros de diagnóstico en las cuentas de Batch
- Se deben habilitar los registros de diagnóstico en Azure Stream Analytics
- Los registros de diagnóstico de Service Bus deben estar habilitados
- Los registros de diagnóstico de los servicios de Search deben estar habilitados.
- Los registros de diagnóstico del centro de eventos deben estar habilitados
- Los registros de diagnóstico de Virtual Machine Scale Sets deben estar habilitados
- Los registros de diagnóstico en Key Vault deben estar habilitados
- La auditoría de SQL Server debe estar habilitada
- La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server



