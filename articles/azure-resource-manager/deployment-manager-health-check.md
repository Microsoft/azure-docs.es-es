---
title: Introducir el lanzamiento de la integración de estado a Azure Deployment Manager
description: Describe cómo implementar un servicio en varias regiones con Azure Deployment Manager. Muestra las prácticas de implementación segura para comprobar la estabilidad de la implementación antes de proceder en todas las regiones.
services: azure-resource-manager
documentationcenter: na
author: mumian
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/08/19
ms.author: jgao
ms.openlocfilehash: 41b16498fb79166b2c77c77a517ee5c443ebec75
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796251"
---
# <a name="introduce-health-integration-rollout-to-azure-deployment-manager-public-preview"></a>Introducir el lanzamiento de la integración de estado a Azure Deployment Manager (versión preliminar)

[Azure Deployment Manager](./deployment-manager-overview.md) le permite realizar implementaciones por fases de los recursos de Azure Resource Manager. Los recursos se implementan región por región de forma ordenada. La comprobación de estado integrada de Azure Deployment Manager puede supervisar las implementaciones y automáticamente stop lanzamientos problemático, para que pueda solucionar problemas y reducir la escala del impacto. Esta característica puede reducir la no disponibilidad del servicio causado por las regresiones en las actualizaciones.

## <a name="health-monitoring-providers"></a>Los proveedores de seguimiento de estado

Con el fin de que la integración de mantenimiento más sencillo posible, Microsoft ha trabajado con algunas de las compañías para proporcionar una solución sencilla de copiar y pegar para integrar las comprobaciones de estado con las implementaciones de supervisión de estado de servicio superior. Si ya no usa a un monitor de estado, estas son soluciones excelentes para comenzar:

| ![implementación de Azure manager health monitor proveedor datadog](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-datadog.svg) | ![implementación de Azure manager health monitor proveedor site24x7](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-site24x7.svg) | ![implementación de Azure manager health monitor proveedor wavefront](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-wavefront.svg) |
|-----|------|------|
|Datadog, la supervisión líder y plataforma de análisis para entornos de nube modernas. Consulte [cómo Datadog se integra con Azure Deployment Manager](https://www.datadoghq.com/azure-deployment-manager/).|Site24x7, solución de supervisión de servicios de la nube privada y pública de all-in-one. Consulte [cómo Site24x7 se integra con Azure Deployment Manager](https://www.site24x7.com/azure/adm.html).| Wavefront, la plataforma de supervisión y análisis para entornos de aplicaciones de varias nubes. Consulte [cómo Wavefront se integra con Azure Deployment Manager](https://go.wavefront.com/wavefront-adm/).|

## <a name="how-service-health-is-determined"></a>Cómo se determina el estado del servicio

[Los proveedores de seguimiento de estado](#health-monitoring-providers) ofrecen varios mecanismos para servicios de supervisión y alertas de los problemas de mantenimiento de servicio. [Azure Monitor](../azure-monitor/overview.md) es un ejemplo de una oferta de este tipo. Azure Monitor puede utilizarse para crear alertas cuando se superan determinados umbrales. Por ejemplo, la utilización de CPU y memoria atiendan más allá de los niveles esperados al implementar una nueva actualización en el servicio. Cuando se le notifique, pueda tomar acciones correctivas.

Estos proveedores de estado normalmente ofrecen las API de REST para que se puede examinar el estado de monitores del servicio mediante programación. Las API de REST o bien puede consultar de nuevo con una simple señal correcto/incorrecto (determinado por el código de respuesta HTTP) o con información detallada acerca de las señales que recibe.

El nuevo *healthCheck* paso en el Administrador de implementación de Azure le permite declarar códigos HTTP que indican que un servicio en buen estado, o bien, para obtener resultados más complejos de REST, incluso puede especificar expresiones regulares que, si coinciden, indican un correcto respuesta.

El flujo de instalación de las comprobaciones de estado del Administrador de implementaciones de Azure:

1. Crear a los monitores de mantenimiento a través de un proveedor de servicios de salud de su elección.
1. Crear uno o varios pasos de comprobación del estado como parte de la implementación de Azure Deployment Manager. Rellene los pasos de comprobación de estado con la siguiente información:

    1. El URI para la API de REST para los monitores de mantenimiento (según lo definido por el proveedor de servicios de mantenimiento).
    1. Información de autenticación. Actualmente se admite solo la autenticación de clave de API estilo.
    1. [Códigos de estado HTTP](https://www.wikipedia.org/wiki/List_of_HTTP_status_codes) o expresiones regulares que definen una respuesta correcta. Tenga en cuenta que puede proporcionar expresiones regulares, que todas deben coincidencia para la respuesta que se consideran en buen estado, o puede proporcionar expresiones de los cuales cualquiera deben coincidir para que la respuesta se considera correcta. Se admiten ambos métodos.

    El siguiente Json es un ejemplo:

    ```json
    {
      "type": "Microsoft.DeploymentManager/steps",
      "apiVersion": "2018-09-01-preview",
      "name": "healthCheckStep",
      "location": "[parameters('azureResourceLocation')]",
      "properties": {
        "stepType": "healthCheck",
        "attributes": {
          "waitDuration": "PT0M",
          "maxElasticDuration": "PT0M",
          "healthyStateDuration": "PT1M",
          "type": "REST",
          "properties": {
            "healthChecks": [
              {
                "name": "appHealth",
                "request": {
                  "method": "GET",
                  "uri": "[parameters('healthCheckUrl')]",
                  "authentication": {
                    "type": "ApiKey",
                    "name": "code",
                    "in": "Query",
                    "value": "[parameters('healthCheckAuthAPIKey')]"
                  }
                },
                "response": {
                  "successStatusCodes": [
                    "200"
                  ],
                  "regex": {
                    "matches": [
                      "Status: healthy",
                      "Status: warning"
                    ],
                    "matchQuantifier": "Any"
                  }
                }
              }
            ]
          }
        }
      }
    },
    ```

1. Invocar los pasos de comprobación del estado en el momento adecuado en la implementación de Azure Deployment Manager. En el ejemplo siguiente, se invoca un paso de comprobación de mantenimiento en **postDeploymentSteps** de **stepGroup2**.

    ```json
    "stepGroups": [
        {
            "name": "stepGroup1",
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceWUS.name,  variables('serviceTopology').serviceWUS.serviceUnit2.name)]",
            "postDeploymentSteps": []
        },
        {
            "name": "stepGroup2",
            "dependsOnStepGroups": ["stepGroup1"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceWUS.name,  variables('serviceTopology').serviceWUS.serviceUnit1.name)]",
            "postDeploymentSteps": [
                {
                    "stepId": "[resourceId('Microsoft.DeploymentManager/steps/', 'healthCheckStep')]"
                }
            ]
        },
        {
            "name": "stepGroup3",
            "dependsOnStepGroups": ["stepGroup2"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceEUS.name,  variables('serviceTopology').serviceEUS.serviceUnit2.name)]",
            "postDeploymentSteps": []
        },
        {
            "name": "stepGroup4",
            "dependsOnStepGroups": ["stepGroup3"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceEUS.name,  variables('serviceTopology').serviceEUS.serviceUnit1.name)]",
            "postDeploymentSteps": []
        }
    ]
    ```

Para recorrer un ejemplo, vea [Tutorial: Usar comprobación de estado en Azure Deployment Manager](./deployment-manager-health-check.md).

## <a name="phases-of-a-health-check"></a>Fases de una comprobación de estado

En este momento Azure Deployment Manager sabe cómo consultar el estado del servicio y en qué fases en la implementación para hacerlo. Sin embargo, Azure Deployment Manager también se permite para la configuración de profundidad de la temporización de estas comprobaciones. Se ejecuta un paso de comprobación del estado en 3 fases secuenciales, todos ellos tengan duraciones configurables: 

1. Espera

    1. Una vez completada una operación de implementación, se pueden reiniciar las máquinas virtuales, volver a configurar en función de los nuevos datos o incluso que se inició por primera vez. También puede tardar servicios para empezar a emitir señales de estado al que se agregue con el estado de proveedor de supervisión en algo útil. Durante este proceso difícil, no puede que tenga sentido comprobar el estado de servicio dado que la actualización aún no ha alcanzado un estado estable. De hecho, el servicio puede ser oscilantes entre Estados positiva y negativo como liquidación los recursos. 
    1. Durante la fase de espera, no se supervisa el estado del servicio. Esto se utiliza para permitir que los recursos implementados el tiempo de platos preparados antes de comenzar el proceso de comprobación de mantenimiento. 
1. Elástica

    1. Puesto que es imposible saber en todos los casos de cuánto recursos llevará a platos preparados antes de que sean estables, permite que la fase elástica durante un período de tiempo flexibles entre cuando los recursos son potencialmente inestables y cuando se necesitan para mantener a un equilibrio correcto estado.
    1. Cuando se inicia la fase elástica, Azure Deployment Manager comienza sondeando periódicamente el punto de conexión REST proporcionado para el estado del servicio. El intervalo de sondeo es configurable. 
    1. Si vuelve el monitor de estado con las señales que indica que el servicio está en mal estado, se omiten estas señales, continúa la fase elástica y sondeo continúa. 
    1. Tan pronto como vuelve el monitor de estado con las señales que indica que el servicio está en buen estado, elástica comience la fase finaliza y la fase HealthyState. 
    1. Por lo tanto, la duración especificada para la fase elástica es la cantidad máxima de tiempo que se puede gastar antes de una respuesta correcta se considera obligatoria de sondeo de estado del servicio. 
1. HealthyState

    1. Durante la fase HealthyState, estado del servicio se sondea continuamente en el mismo intervalo que la fase elástico. 
    1. El servicio se espera que mantenga las señales correcto desde el proveedor de supervisión de estado para toda la duración especificada. 
    1. Si en cualquier momento, se detecta una respuesta en mal estado, Azure Deployment Manager detendrá la implementación completa y devolver la respuesta REST que lleva las señales de servicio en mal estado.
    1. Una vez que ha finalizado la duración HealthyState, completada la comprobación del estado y la implementación continúa con el paso siguiente.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido acerca de cómo integrar la supervisión del estado en el Administrador de implementación de Azure. Pase al siguiente artículo, donde aprenderá a realizar implementaciones con Deployment Manager.

> [!div class="nextstepaction"]
> [Tutorial: integración de comprobación de estado en el Administrador de implementación de Azure](./deployment-manager-tutorial-health-check.md)