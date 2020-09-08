---
title: Implementación de aplicaciones con plantillas
description: Encuentre una guía para crear plantillas de Azure Resource Manager para aprovisionar e implementar aplicaciones de App Service.
author: tfitzmac
ms.topic: article
ms.date: 01/03/2019
ms.author: tomfitz
ms.custom: seodec18
ms.openlocfilehash: 1146b5979d81b91c6c6894aa54b2e0ca50c896c1
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961625"
---
# <a name="guidance-on-deploying-web-apps-by-using-azure-resource-manager-templates"></a>Guía de implementación de aplicaciones web mediante plantillas de Azure Resource Manager

En este artículo se ofrecen recomendaciones para la creación de plantillas de Azure Resource Manager para implementar soluciones de Azure App Service. Estas recomendaciones pueden ayudarle a evitar problemas comunes.

## <a name="define-dependencies"></a>Definición de las dependencias

La definición de las dependencias de Web Apps requiere entender la interacción de los recursos dentro de una instancia de Web Apps. Si especifica las dependencias en un orden incorrecto, puede provocar errores de implementación o crear una condición de carrera que detenga la implementación.

> [!WARNING]
> Si incluye una extensión de sitio de MSDeploy en la plantilla, debe establecer los recursos de configuración de manera que dependan del recurso de MSDeploy. Los cambios de configuración provocan el reinicio asincrónico del sitio. Al hacer que los recursos de configuración dependan de MSDeploy, se garantiza que MSDeploy finaliza antes de que se reinicie el sitio. Sin estas dependencias, el sitio podría reiniciarse durante el proceso de implementación de MSDeploy. Para ver una plantilla de ejemplo, consulte la [plantilla de Wordpress con dependencia de Web Deploy](https://github.com/davidebbo/AzureWebsitesSamples/blob/master/ARMTemplates/WordpressTemplateWebDeployDependency.json).

En la siguiente imagen se muestra el orden de dependencia de distintos recursos de App Service:

![Dependencias de la aplicación web](media/web-sites-rm-template-guidance/web-dependencies.png)

Los recursos se implementan en el orden siguiente:

**Nivel 1**
* Plan de App Service.
* Otros recursos relacionados, como bases de datos o cuentas de almacenamiento.

**Nivel 2**
* Aplicación web: depende de plan de App Service.
* Instancia de Azure Application Insights destinada a la granja de servidores: depende del plan de App Service.

**Nivel 3**
* Control de código fuente: depende de la aplicación web.
* Extensión del sitio MSDeploy: depende de la aplicación web.
* Instancia de Azure Application Insights destinada a la aplicación web: depende de la aplicación web.

**Nivel 4**
* Certificado de App Service: depende del control de código o de MSDeploy (si alguno está presente). De lo contrario, depende de la aplicación web.
* Configuración (cadenas de conexión, valores de configuración web, configuración de la aplicación): depende del control de código fuente o de MSDeploy (si alguno está presente). De lo contrario, depende de la aplicación web.

**Nivel 5**
* Enlaces de nombre de host: depende del certificado, si existe. De lo contrario, depende de un recurso de nivel superior.
* Extensiones de sitio: depende de los valores de configuración, si existen. De lo contrario, depende de un recurso de nivel superior.

Normalmente, la solución incluye solo algunos de estos recursos y niveles. Para los niveles que faltan, asigne los recursos inferiores al siguiente nivel superior.

En el ejemplo siguiente se muestra parte de una plantilla. El valor de configuración de la cadena de conexión depende de la extensión MSDeploy. La extensión de MSDeploy depende de la aplicación web y la base de datos. 

```json
{
    "name": "[parameters('appName')]",
    "type": "Microsoft.Web/Sites",
    ...
    "resources": [
      {
          "name": "MSDeploy",
          "type": "Extensions",
          "dependsOn": [
            "[concat('Microsoft.Web/Sites/', parameters('appName'))]",
            "[concat('Microsoft.Sql/servers/', parameters('dbServerName'), '/databases/', parameters('dbName'))]",
          ],
          ...
      },
      {
          "name": "connectionstrings",
          "type": "config",
          "dependsOn": [
            "[concat('Microsoft.Web/Sites/', parameters('appName'), '/Extensions/MSDeploy')]"
          ],
          ...
      }
    ]
}
```

Para obtener un ejemplo listo para ejecutarse que use el código anterior, consulte [Template: Build a simple Umbraco Web App](https://github.com/Azure/azure-quickstart-templates/tree/master/umbraco-webapp-simple) (Plantilla: creación de una aplicación web de Umbraco sencilla).

## <a name="find-information-about-msdeploy-errors"></a>Búsqueda de información sobre los errores de MSDeploy

Si la plantilla de Resource Manager usa MSDeploy, los mensajes de error de implementación pueden ser difíciles de comprender. Para más información después de un error de implementación, pruebe los pasos siguientes:

1. Vaya a la [consola Kudu](https://github.com/projectkudu/kudu/wiki/Kudu-console) del sitio.
2. Vaya a la capeta en D:\home\LogFiles\SiteExtensions\MSDeploy.
3. Busque los archivos appManagerStatus.xml y appManagerLog.xml. El primer archivo registra el estado. El segundo, información sobre el error. Si el error no está claro, puede incluirlo al pedir ayuda en el [foro](/answers/topics/azure-webapps.html).

## <a name="choose-a-unique-web-app-name"></a>Elija un nombre único para la aplicación web.

El nombre de la aplicación web debe ser globalmente único. Puede usar una convención de nomenclatura que es probable que sea única o la [función uniqueString](../azure-resource-manager/templates/template-functions-string.md#uniquestring), que le ayudará a generar un nombre único.

```json
{
  "apiVersion": "2016-08-01",
  "name": "[concat(parameters('siteNamePrefix'), uniqueString(resourceGroup().id))]",
  "type": "Microsoft.Web/sites",
  ...
}
```

## <a name="deploy-web-app-certificate-from-key-vault"></a>Implementar el certificado de aplicación web desde Key Vault

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Si la plantilla incluye un recurso [Microsoft.Web/certificates](/azure/templates/microsoft.web/certificates) para el enlace TLS/SSL, y el certificado se almacena en Key Vault, debe asegurarse de que la identidad de App Service pueda acceder al certificado.

En Azure global, la entidad de servicio de App Service tiene el identificador de **abfa0a7c-a6b6-4736-8310-5855508787cd**. Para conceder acceso a Key Vault para la entidad de servicio de App Service, use lo siguiente:

```azurepowershell-interactive
Set-AzKeyVaultAccessPolicy `
  -VaultName KEY_VAULT_NAME `
  -ServicePrincipalName abfa0a7c-a6b6-4736-8310-5855508787cd `
  -PermissionsToSecrets get `
  -PermissionsToCertificates get
```

En Azure Government, la entidad de servicio de App Service tiene el Id. de **6a02c803-dafd-4136-b4c3-5a6f318b4714**. Use ese identificador en el ejemplo anterior.

En la instancia de Key Vault, seleccione **Certificados** y **Generar o importar** para cargar el certificado.

![Importación de certificado](media/web-sites-rm-template-guidance/import-certificate.png)

En la plantilla, proporcione el nombre del certificado para `keyVaultSecretName`.

Para obtener una plantilla de ejemplo, vea [Deploy a Web App certificate from Key Vault secret and use it for creating SSL binding](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-certificate-from-key-vault) (Implementar un certificado de aplicación web de secreto de Key Vault y usarlo para crear el enlace SSL).

## <a name="next-steps"></a>Pasos siguientes

* Para un tutorial sobre la implementación de aplicaciones web con una plantilla, consulte [Aprovisionamiento e implementación predecibles de microservicios en Azure](deploy-complex-application-predictably.md).
* Para información sobre la sintaxis de JSON y las propiedades de los tipos de recursos de las plantillas, consulte [Azure Resource Manager template reference](/azure/templates/) (Referencia de las plantillas de Azure Resource Manager).