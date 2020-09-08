---
title: Publicación de un anuncio en un laboratorio de Azure DevTest Labs | Microsoft Docs
description: Obtenga información sobre cómo publicar un anuncio personalizado en un laboratorio existente para notificar a los usuarios acerca de cambios recientes o adiciones en el laboratorio en Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 72c6ed7ab691ece094673ff5a158ffa769a9f7a1
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88270774"
---
# <a name="post-an-announcement-to-a-lab-in-azure-devtest-labs"></a>Publicación de un anuncio en un laboratorio de Azure DevTest Labs

Como administrador del laboratorio, puede publicar un anuncio personalizado en un laboratorio existente para notificar a los usuarios acerca de cambios recientes o adiciones en el laboratorio. Por ejemplo, puede que desee informar a los usuarios acerca de:

- Nuevos tamaños de máquinas virtuales que están disponibles
- Imágenes que están actualmente inutilizables
- Actualizaciones de directivas del laboratorio

Una vez publicado, el anuncio se muestra en la página de información general del laboratorio y el usuario puede seleccionarlo para obtener más detalles.

La característica de anuncios está pensada para usarla para notificaciones temporales.  Puede deshabilitar fácilmente un anuncio cuando este deja de ser necesario.

## <a name="steps-to-post-an-announcement-in-an-existing-lab"></a>Pasos para publicar un anuncio en un laboratorio existente

1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Si es necesario, seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista. (Puede que su laboratorio ya aparezca en el panel, en **Todos los recursos**).
1. En la lista de laboratorios, seleccione aquel en el que desea publicar un anuncio.
1. En el área **Introducción** del laboratorio, seleccione **Configuración y directivas**.

    ![Botón Configuración y directivas](./media/devtest-lab-announcements/devtestlab-config-and-policies.png)

1. A la izquierda en **CONFIGURACIÓN**, seleccione **Lab announcement** (Anuncio del laboratorio).

    ![Botón de anuncio del laboratorio](./media/devtest-lab-announcements/devtestlab-announcements.png)

1. Para crear un mensaje para los usuarios de este laboratorio, establezca **Habilitado** en **Sí**.

1. Puede escribir una **fecha de expiración** para especificar una fecha y hora después de la cual ya no se mostrará el anuncio a los usuarios. Si no escribe una fecha de expiración, el anuncio permanece hasta que lo deshabilita.

   > [!NOTE]
   > Una vez que el anuncio expira, ya no se muestra a los usuarios pero sigue existiendo en el panel **Anuncio de laboratorio**. Puede modificarlo y volver a habilitarlo para que esté activo nuevamente.
   >
   >

1. Escriba un **título del anuncio** y el **texto del anuncio**.

   El título puede contener hasta 100 caracteres y se muestra al usuario en la página de información general del laboratorio. Si el usuario selecciona el título, se muestra el texto del anuncio.

   El texto del anuncio acepta marcado. A medida que escribe el texto del anuncio, puede ver el mensaje en el área de vista previa en la parte inferior de la pantalla.

    ![Pantalla de anuncio del laboratorio para crear el mensaje.](./media/devtest-lab-announcements/devtestlab-post-announcement.png)


1. Seleccione **Guardar** cuando el anuncio esté listo para publicarse.

Cuando ya no quiera mostrar este anuncio a los usuarios del laboratorio, vuelva a la página **Lab announcement** (Anuncio del laboratorio) y establezca **Habilitado** en **No**. Si especificó una fecha de expiración, el anuncio se deshabilita automáticamente en esa fecha y hora.

## <a name="steps-for-users-to-view-an-announcement"></a>Pasos para que los usuarios vean un anuncio

1. En [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040), seleccione un laboratorio.

1. Si el laboratorio tiene un anuncio publicado en él, se muestra un aviso de información en la parte superior de la página de información general del laboratorio. Este aviso de información es el título del anuncio que se especificó cuando se creó este.

    ![Página de información general del anuncio del laboratorio](./media/devtest-lab-announcements/devtestlab-user-announcement.png)

1. El usuario puede seleccionar el mensaje para ver el anuncio completo.

    ![Más información sobre el anuncio del laboratorio](./media/devtest-lab-announcements/devtestlab-user-announcement-text.png)

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="azure-resource-manager-template"></a>Plantilla del Administrador de recursos de Azure
Puede especificar un anuncio como parte de una plantilla de Azure Resource Manager, como se muestra en el ejemplo siguiente:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "string",
            "defaultValue": "devtestlabfromarm"
        },
        "regionId": {
            "type": "string",
            "defaultValue": "eastus"
        }
    },
    "resources": [
        {
            "apiVersion": "2017-04-26-preview",
            "name": "[parameters('name')]",
            "type": "Microsoft.DevTestLab/labs",
            "location": "[parameters('regionId')]",
            "tags": {},
            "properties": {
                "labStorageType": "Premium",
                "announcement":
                {
                    "title": "Important! Three images are currently inaccessible. Click for more information.",
                    "markdown":"# Images are inaccessible\n\nThe following 3 images are currently not available for use: \n\n- image1\n- image2\n- image3\n\nI am working to fix the problem ASAP.",
                    "enabled": "Enabled",
                    "expirationDate":"2018-12-31T06:00:00+00:00",
                    "expired": "false"
                },
                "support": {
                    "markdown": "",
                    "enabled": "Enabled"
                }
            },
            "resources": [
                {
                    "apiVersion": "2017-04-26-preview",
                    "name": "LabVmsShutdown",
                    "location": "[parameters('regionId')]",
                    "type": "schedules",
                    "dependsOn": [
                        "[resourceId('Microsoft.DevTestLab/labs', parameters('name'))]"
                    ],
                    "properties": {
                        "status": "Enabled",
                        "timeZoneId": "Eastern Standard Time",
                        "dailyRecurrence": {
                            "time": "1900"
                        },
                        "taskType": "LabVmsShutdownTask",
                        "notificationSettings": {
                            "status": "Disabled",
                            "timeInMinutes": 30
                        }
                    }
                },
                {
                    "apiVersion": "2017-04-26-preview",
                    "name": "[concat('Dtl', parameters('name'))]",
                    "type": "virtualNetworks",
                    "location": "[parameters('regionId')]",
                    "dependsOn": [
                        "[resourceId('Microsoft.DevTestLab/labs', parameters('name'))]"
                    ]
                }
            ]
        }
    ]
}
```

Puede implementar una plantilla de Azure Resource Manager de una de las maneras siguientes:

- [Azure Portal](../azure-resource-manager/templates/deploy-portal.md)
- [Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md)
- [CLI de Azure](../azure-resource-manager/templates/deploy-cli.md)
- [REST API](../azure-resource-manager/templates/deploy-rest.md)

## <a name="next-steps"></a>Pasos siguientes
* Si cambia o establece una directiva de laboratorio, puede que quiera publicar un anuncio para informar a los usuarios. [Configuración de directivas y programaciones](devtest-lab-set-lab-policy.md) proporciona información acerca de cómo aplicar restricciones y convenciones en la suscripción mediante el uso de directivas personalizadas.
* Explore la [galería de plantillas de inicio rápido de Azure Resource Manager de DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates).
