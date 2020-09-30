---
title: Creación de directivas de seguridad personalizadas en Azure Security Center | Microsoft Docs
description: Definiciones de directivas personalizadas de Azure supervisadas en Azure Security Center.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 04/22/2020
ms.author: memildin
ms.openlocfilehash: 065c6661ffc9f618a5dfd32a9f45d26569a56003
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89278180"
---
# <a name="using-custom-security-policies"></a>Uso de directivas de seguridad personalizadas

Para ayudarle a proteger los sistemas y el entorno, Azure Security Center genera recomendaciones de seguridad. Estas recomendaciones se basan en los procedimientos recomendados del sector, que se incorporan a la directiva de seguridad predeterminada genérica que se proporciona a todos los clientes. Estas recomendaciones también pueden provenir de los conocimientos de Security Center acerca del sector y de los estándares normativos.

Gracias a esta característica, puede agregar sus propias iniciativas *personalizadas*. A continuación, recibirá recomendaciones si el entorno no sigue las directivas que creó. Cualquier iniciativa personalizada que cree aparecerá junto a las iniciativas integradas en el panel de cumplimiento normativo, según se describe en el tutorial [Mejora del cumplimiento normativo](security-center-compliance-dashboard.md).

Como se explicó en la [documentación de Azure Policy](https://docs.microsoft.com/azure/governance/policy/concepts/definition-structure#definition-location), cuando se especifica una ubicación para la iniciativa personalizada, debe ser un grupo de administración o una suscripción. 

## <a name="to-add-a-custom-initiative-to-your-subscription"></a>Agregar una iniciativa personalizada a la suscripción 

1. En la barra lateral de Security Center, abra la página **Directiva de seguridad**.

1. Seleccione una suscripción o un grupo de administración al que quiera agregar una iniciativa personalizada.

    [![Seleccione la suscripción para la que se creará la directiva personalizada](media/custom-security-policies/custom-policy-selecting-a-subscription.png)](media/custom-security-policies/custom-policy-selecting-a-subscription.png#lightbox)

    > [!NOTE]
    > Debe agregar estándares personalizados en el nivel de suscripción (o superior) para que se evalúen y se muestren en Security Center. 
    >
    > Cuando se agrega un estándar personalizado, este asigna una *iniciativa* a ese ámbito. Por lo tanto, se recomienda seleccionar el ámbito más amplio necesario para esa asignación.

1. En la página Directiva de seguridad, en Sus iniciativas personalizadas, haga clic en **Agregar una iniciativa personalizada**.

    [![Haga clic en Agregar una iniciativa personalizada](media/custom-security-policies/custom-policy-add-initiative.png)](media/custom-security-policies/custom-policy-add-initiative.png#lightbox)

    Aparece la siguiente página:

    ![Crear o agregar una directiva](media/custom-security-policies/create-or-add-custom-policy.png)

1. En la página para agregar iniciativas personalizadas, revise la lista de directivas personalizadas ya creadas en la organización. Si ve una que quiera asignar a la suscripción, haga clic en **Agregar**. Si no hay una iniciativa en la lista que satisfaga sus necesidades, omita este paso.

1. Para crear una nueva iniciativa personalizada:

    1. Haga clic en **Crear nueva**.
    1. Escriba la ubicación y el nombre de la definición.
    1. Seleccione las directivas que quiera incluir y haga clic en **Agregar**.
    1. Escriba los parámetros que quiera.
    1. Haga clic en **Save**(Guardar).
    1. En la página Adición de iniciativas personalizadas, haga clic en Actualizar. La nueva iniciativa se mostrará como disponible.
    1. Haga clic en **Agregar** y asígnela a su suscripción.

    > [!NOTE]
    > Recuerde que la creación de nuevas iniciativas requiere credenciales de propietario de la suscripción. Para obtener más información sobre los roles de Azure, consulte los [ permisos en Azure Security Center](security-center-permissions.md).

    Su nueva iniciativa se aplica y puede ver el impacto de las dos formas siguientes:

    * En la barra lateral de Security Center, en Directiva y cumplimiento, seleccione **Cumplimiento normativo**. El panel de cumplimiento se abre para mostrar su nueva iniciativa personalizada junto a las iniciativas integradas.
    
    * Empezará a recibir recomendaciones si el entorno no sigue las directivas que ha definido.

1. Para ver las recomendaciones resultantes de la directiva, haga clic en **Recomendaciones** de la barra lateral para abrir la página recomendaciones. Las recomendaciones aparecerán con una etiqueta "Personalizada" y estarán disponibles en aproximadamente una hora.

    [![Recomendaciones personalizadas](media/custom-security-policies/custom-policy-recommendations.png)](media/custom-security-policies/custom-policy-recommendations-in-context.png#lightbox)

## <a name="enhance-your-custom-recommendations-with-detailed-information"></a>Mejora de las recomendaciones personalizadas con información detallada

Las recomendaciones integradas que se suministran con Azure Security Center incluyen detalles como los niveles de gravedad y las instrucciones de corrección. Si desea agregar este tipo de información a las recomendaciones personalizadas para que aparezca en Azure Portal o donde acceda a las recomendaciones, debe usar la API de REST. 

Los dos tipos de información que puede agregar son:

- **RemediationDescription**, cadena
- **Severity**, enumeración [Low, Medium, High]

Los metadatos deben agregarse a la definición de la directiva para una directiva que forme parte de la iniciativa personalizada. Debe estar en la propiedad "securityCenter", como se muestra a continuación:

```json
 "metadata": {
    "securityCenter": {
        "RemediationDescription": "Custom description goes here",
        "Severity": "High",
    },
```

A continuación se muestra un ejemplo de una directiva personalizada que incluye la propiedad metadata/securityCenter:

  ```json
  {
"properties": {
    "displayName": "Security - ERvNet - AuditRGLock",
    "policyType": "Custom",
    "mode": "All",
    "description": "Audit required resource groups lock",
    "metadata": {
        "securityCenter": {
            "remediationDescription": "Resource Group locks can be set via Azure Portal -> Resource Group -> Locks",
            "severity": "High"
        }
    },
    "parameters": {
        "expressRouteLockLevel": {
            "type": "String",
            "metadata": {
                "displayName": "Lock level",
                "description": "Required lock level for ExpressRoute resource groups."
            },
            "allowedValues": [
                "CanNotDelete",
                "ReadOnly"
            ]
        }
    },
    "policyRule": {
        "if": {
            "field": "type",
            "equals": "Microsoft.Resources/subscriptions/resourceGroups"
        },
        "then": {
            "effect": "auditIfNotExists",
            "details": {
                "type": "Microsoft.Authorization/locks",
                "existenceCondition": {
                    "field": "Microsoft.Authorization/locks/level",
                    "equals": "[parameters('expressRouteLockLevel')]"
                }
            }
        }
    }
}
}
  ```

Para ver otro ejemplo del uso de la propiedad de securityCenter, consulte [esta sección de la documentación de la API de REST](https://docs.microsoft.com/rest/api/securitycenter/assessmentsmetadata/createinsubscription#examples).


## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a crear directivas de seguridad personalizadas. 

Para obtener material relacionado, consulte los siguientes artículos: 

- [Introducción a las directivas de seguridad](tutorial-security-policy.md)
- [Lista de directivas de seguridad integradas](security-center-policy-definitions.md)