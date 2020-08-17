---
title: Traslado de una aplicación a otra región
description: Aprenda a trasladar los recursos de App Service de una región a otra.
ms.topic: how-to
ms.date: 02/27/2020
ms.custom: subject-moving-resources
ms.openlocfilehash: 87a2da31802a8b6858e875c23ef1dbd2d6d006bc
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86524861"
---
# <a name="move-an-app-service-resource-to-another-region"></a>Traslado de un recurso de App Service a otra región

En este artículo, se explica cómo pueden trasladarse los recursos de App Service a otra región de Azure. Puede mover sus recursos a otra región por varios motivos. Por ejemplo, para aprovechar una nueva región de Azure, para implementar características o servicios que solo están disponibles en regiones concretas, para cumplir los requisitos de gobernanza y las directivas internas o en respuesta a los requisitos de planeamiento de capacidad.

Los recursos de App Service son específicos de una región y no pueden moverse entre regiones. Debe crear una copia de los recursos de App Service existentes en la región de destino y, a continuación, mover el contenido a la nueva aplicación. Si la aplicación de origen usa un dominio personalizado, puede [migrarlo a la nueva aplicación de la región de destino](manage-custom-dns-migrate-domain.md) cuando haya terminado.

Para que resulte más fácil copiar una aplicación, puede [clonar una aplicación específica de App Service](app-service-web-app-cloning.md) en un plan de App Service de otra región, aunque esto tiene algunas [limitaciones](app-service-web-app-cloning.md#current-restrictions), en particular, que este mecanismo no puede utilizarse con aplicaciones de Linux.

## <a name="prerequisites"></a>Requisitos previos

- Asegúrese de que la aplicación de App Service esté en la región de Azure desde la que va a trasladarla.
- Compruebe que la región de destino admite App Service y los servicios relacionados, cuyos recursos desea trasladar.
<!-- - Domain bindings, certificates, and managed identities can't replicated using the **Export template** method. You must create them manually. -->

## <a name="prepare"></a>Preparación

Identifique todos los recursos de App Service que utiliza actualmente. Por ejemplo:

- Aplicaciones de App Service
- [Planes de App Service](overview-hosting-plans.md)
- [Ranuras de implementación](deploy-staging-slots.md)
- [Dominios personalizados adquiridos en Azure](manage-custom-dns-buy-domain.md)
- [Certificados SSL](configure-ssl-certificate.md)
- [Integración de Azure Virtual Network](web-sites-integrate-with-vnet.md)
- [Conexiones híbridas](app-service-hybrid-connections.md)
- [Identidades administradas](overview-managed-identity.md)
- [Configuración de copia de seguridad](manage-backup.md)

Ciertos recursos, como los certificados importados o las conexiones híbridas, tienen elementos que están integrados con otros servicios de Azure. Para más información acerca de cómo migrar esos recursos entre regiones, consulte la documentación de los servicios correspondientes.

## <a name="move"></a>Move

1. [Cree una copia de seguridad de la aplicación de origen](manage-backup.md).
1. [Cree una aplicación en un nuevo plan de App Service de la región de destino](app-service-plan-manage.md#create-an-app-service-plan).
2. [Restaure la copia de seguridad en la aplicación de destino](web-sites-restore.md).
2. Si usa un dominio personalizado, [vincúlelo preferentemente a la aplicación de destino](manage-custom-dns-migrate-domain.md#bind-the-domain-name-preemptively) con `awverify.` y [habilite el dominio en la aplicación de destino](manage-custom-dns-migrate-domain.md#enable-the-domain-for-your-app).
3. Configure el resto del contenido de la aplicación de destino para que esté igual que en la aplicación de origen y compruebe la configuración.
4. Cuando tenga todo preparado para que el dominio personalizado apunte a la aplicación de destino, [reasigne el nombre de dominio](manage-custom-dns-migrate-domain.md#remap-the-active-dns-name).

<!-- 1. Login to the [Azure portal](https://portal.azure.com) > **Resource Groups**.
2. Locate the Resource Group that contains the source App Service resources and click on it.
3. Select > **Settings** > **Export template**.
4. Choose **Deploy** in the **Export template** blade.
5. Click **TEMPLATE** > **Edit template** to open the template in the online editor.
6. Click inside the online editor and type Ctrl+F (or ⌘+F on a Mac) and type `"identity": {` to find any managed identity definition. The following is an example if you have a user-assigned managed identity.
    ```json
    "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
            "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/<group-name>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<identity-name>": {
                "principalId": "00000000-0000-0000-0000-000000000000",
                "clientId": "00000000-0000-0000-0000-000000000000"
            }
        }
    },
    ```
6. Click inside the online editor and type Ctrl+F (or ⌘+F on a Mac) and type `"Microsoft.Web/sites/hostNameBindings` to find all hostname bindings. The following is an example if you have a user-assigned managed identity.
    ```json
    {
        "type": "Microsoft.Web/sites/hostNameBindings",
        "apiVersion": "2018-11-01",
        "name": "[concat(parameters('sites_webapp_name'), '/', parameters('sites_webapp_name'), '.azurewebsites.net')]",
        "location": "West Europe",
        "dependsOn": [
            "[resourceId('Microsoft.Web/sites', parameters('sites_webapp_name'))]"
        ],
        "properties": {
            "siteName": "<app-name>",
            "hostNameType": "Verified"
        }
    },
    ```
6. Click inside the online editor and type Ctrl+F (or ⌘+F on a Mac) and type `"Microsoft.Web/certificates` to find all hostname bindings. The following is an example if you have a user-assigned managed identity.
    ```json
    {
        "type": "Microsoft.Web/certificates",
        "apiVersion": "2018-11-01",
        "name": "[parameters('certificates_test2_cephaslin_com_name')]",
        "location": "West Europe",
        "properties": {
            "hostNames": [
                "[parameters('certificates_test2_cephaslin_com_name')]"
            ],
            "password": "[parameters('certificates_test2_cephaslin_com_password')]"
        }
    },
    ```
7. Delete the entire JSON block. Click **Save** in the online editor.
8. Click **BASICS** > **Create new** to create a new resource group. Type the group name and click **OK**.
9. In **BASICS** > **Location**, select the region you want.   -->

## <a name="clean-up-source-resources"></a>Limpieza de los recursos de origen

Elimine la aplicación y el plan de App Service de origen. [Los planes de App Service de los niveles que no son gratuitos generan cargos aunque no se ejecute ninguna aplicación en ellos.](app-service-plan-manage.md#delete-an-app-service-plan)

## <a name="next-steps"></a>Pasos siguientes

[Clonación de aplicaciones de Azure App Service mediante PowerShell](app-service-web-app-cloning.md)