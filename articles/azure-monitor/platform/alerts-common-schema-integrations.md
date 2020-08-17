---
title: Integración del esquema común de alertas con Logic Apps
description: Obtenga información sobre cómo crear una aplicación lógica que aprovecha el esquema común de alertas para controlar todas las alertas.
ms.topic: conceptual
ms.subservice: alerts
ms.date: 05/27/2019
ms.openlocfilehash: a7893f41e3e4cce737853fc168c1931f3bf7b532
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322098"
---
# <a name="how-to-integrate-the-common-alert-schema-with-logic-apps"></a>Integración del esquema común de alertas con Logic Apps

En este artículo se muestra cómo crear una aplicación lógica que aprovecha el esquema común de alertas para controlar todas las alertas.

## <a name="overview"></a>Información general

El [esquema común de alerta](https://aka.ms/commonAlertSchemaDocs) proporciona un esquema JSON estandarizado y extensible a través de todos los diferentes tipos de alerta. El esquema común de alerta es especialmente útil cuando se aprovecha mediante programación: con webhooks, runbooks y aplicaciones lógicas. En este artículo, se muestra cómo se puede crear una única aplicación lógica para controlar todas las alertas. Los mismos principios se pueden aplicar a otros métodos de programación. La aplicación lógica descrita en este artículo crea variables bien definidas para los [campos "esenciales"](alerts-common-schema-definitions.md#essentials)y también se describe cómo puede controlar la lógica específica del [tipos de alerta](alerts-common-schema-definitions.md#alert-context).


## <a name="prerequisites"></a>Prerequisites 

En este artículo se supone que el lector está familiarizado con los conceptos siguientes: 
* Configuración de reglas de alerta ([métrica](./alerts-metric.md), [registro](./alerts-log.md), [registro de actividad](./alerts-activity-log.md))
* Configuración de [grupos de acciones](./action-groups.md)
* Habilitación del [esquema común de alerta](./alerts-common-schema.md#how-do-i-enable-the-common-alert-schema) desde dentro de grupos de acciones

## <a name="create-a-logic-app-leveraging-the-common-alert-schema"></a>Creación de una aplicación lógica, aprovechando el esquema común de alerta

1. Siga los [pasos para crear la aplicación lógica](./action-groups-logic-app.md). 

1.  Seleccione el desencadenador **Cuando se recibe una solicitud HTTP**.

    ![Desencadenadores de aplicación lógica](media/action-groups-logic-app/logic-app-triggers.png "Desencadenadores de aplicación lógica")

1.  Seleccione **Editar** para cambiar el desencadenador de solicitud HTTP.

    ![Desencadenadores de solicitudes HTTP](media/action-groups-logic-app/http-request-trigger-shape.png "Desencadenadores de solicitudes HTTP")


1.  Copie y pegue el siguiente esquema:

    ```json
        {
            "type": "object",
            "properties": {
                "schemaId": {
                    "type": "string"
                },
                "data": {
                    "type": "object",
                    "properties": {
                        "essentials": {
                            "type": "object",
                            "properties": {
                                "alertId": {
                                    "type": "string"
                                },
                                "alertRule": {
                                    "type": "string"
                                },
                                "severity": {
                                    "type": "string"
                                },
                                "signalType": {
                                    "type": "string"
                                },
                                "monitorCondition": {
                                    "type": "string"
                                },
                                "monitoringService": {
                                    "type": "string"
                                },
                                "alertTargetIDs": {
                                    "type": "array",
                                    "items": {
                                        "type": "string"
                                    }
                                },
                                "originAlertId": {
                                    "type": "string"
                                },
                                "firedDateTime": {
                                    "type": "string"
                                },
                                "resolvedDateTime": {
                                    "type": "string"
                                },
                                "description": {
                                    "type": "string"
                                },
                                "essentialsVersion": {
                                    "type": "string"
                                },
                                "alertContextVersion": {
                                    "type": "string"
                                }
                            }
                        },
                        "alertContext": {
                            "type": "object",
                            "properties": {}
                        }
                    }
                }
            }
        }
    ```

1. Seleccione **+** **Nuevo paso** y luego elija **Agregar una acción**.

    ![Agregar una acción](media/action-groups-logic-app/add-action.png "Agregar una acción")

1. En esta fase, puede agregar diversos conectores (Microsoft Teams, Slack, Salesforce, etc.) en función de sus requisitos empresariales específicos. Puede usar los "campos esenciales" proporcionados. 

    ![Campos esenciales](media/alerts-common-schema-integrations/logic-app-essential-fields.png "Campos esenciales")
    
    Como alternativa, puede crear una lógica condicional propia según el tipo de alerta mediante la opción "Expression".

    ![Expresión de aplicación lógica](media/alerts-common-schema-integrations/logic-app-expressions.png "Expresión de aplicación lógica")
    
     El [campo "monitoringService"](alerts-common-schema-definitions.md#alert-context) permite identificar de forma única el tipo de alerta en función del cual puede crear la lógica condicional.

    
    Por ejemplo, el siguiente fragmento de código comprueba si la alerta es una alerta de registro basada en Application Insights y, en caso afirmativo, se imprimen los resultados de búsqueda. En caso contrario, imprime "NA" (No disponible).

    ```text
      if(equals(triggerBody()?['data']?['essentials']?['monitoringService'],'Application Insights'),triggerBody()?['data']?['alertContext']?['SearchResults'],'NA')
    ```
    
     Obtenga más información sobre cómo [escribir expresiones de la lógica de aplicación](../../logic-apps/workflow-definition-language-functions-reference.md#logical-comparison-functions).

    


## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre los grupos de acciones](./action-groups.md)
* [Más información sobre el esquema de alertas comunes](https://aka.ms/commonAlertSchemaDocs).

