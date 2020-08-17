---
title: Extensión Desired State Configuration con plantillas de Azure Resource Manager
description: Obtenga información sobre la definición de la plantilla de Resource Manager para la extensión Desired State Configuration (DSC) en Azure.
services: virtual-machines-windows
author: bobbytreed
manager: carmonm
tags: azure-resource-manager
keywords: dsc
ms.assetid: b5402e5a-1768-4075-8c19-b7f7402687af
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: na
ms.date: 10/05/2018
ms.author: robreed
ms.openlocfilehash: dc73b5b9f05d24de206b25095ea7eaf93f035298
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86511167"
---
# <a name="desired-state-configuration-extension-with-azure-resource-manager-templates"></a>Extensión Desired State Configuration con plantillas de Azure Resource Manager

En este artículo se describe la plantilla de Azure Resource Manager para el [controlador de la extensión Desired State Configuration (DSC)](dsc-overview.md). Muchos de los ejemplos usan **RegistrationURL** (proporcionado como String) y **RegistrationKey** (proporcionado como [PSCredential](/dotnet/api/system.management.automation.pscredential)) para incorporarlos en Azure Automation. Para más información acerca de cómo obtener dichos valores, consulte [Incorporación de máquinas para su administración mediante Azure Automation State Configuration](../../automation/automation-dsc-onboarding.md#enable-machines-securely-using-registration).

> [!NOTE]
> Podría encontrar ejemplos de esquema levemente distintos. El cambio en el esquema ocurrió en la versión de octubre de 2016. Para obtener detalles, vea [Actualización desde un formato anterior](#update-from-a-previous-format).

## <a name="template-example-for-a-windows-vm"></a>Ejemplo de plantilla para una máquina virtual de Windows

El fragmento de código siguiente corresponde a la sección **Resource** de la plantilla.
La extensión DSC hereda las propiedades de extensión predeterminadas.
Para más información, consulte la [clase VirtualMachineExtension](/dotnet/api/microsoft.azure.management.compute.models.virtualmachineextension?view=azure-dotnet).

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "Microsoft.Powershell.DSC",
  "apiVersion": "2018-06-30",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', parameters('VMName'))]"
  ],
  "properties": {
    "publisher": "Microsoft.Powershell",
    "type": "DSC",
    "typeHandlerVersion": "2.77",
    "autoUpgradeMinorVersion": true,
    "protectedSettings": {
      "Items": {
        "registrationKeyPrivate": "[listKeys(resourceId('Microsoft.Automation/automationAccounts/', parameters('automationAccountName')), '2018-06-30').Keys[0].value]"
      }
    },
    "settings": {
      "Properties": [
        {
          "Name": "RegistrationKey",
          "Value": {
            "UserName": "PLACEHOLDER_DONOTUSE",
            "Password": "PrivateSettingsRef:registrationKeyPrivate"
          },
          "TypeName": "System.Management.Automation.PSCredential"
        },
        {
          "Name": "RegistrationUrl",
          "Value": "[reference(concat('Microsoft.Automation/automationAccounts/', parameters('automationAccountName'))).registrationUrl]",
          "TypeName": "System.String"
        },
        {
          "Name": "NodeConfigurationName",
          "Value": "[parameters('nodeConfigurationName')]",
          "TypeName": "System.String"
        }
      ]
    }
  }
}
```

## <a name="template-example-for-windows-virtual-machine-scale-sets"></a>Ejemplo de plantilla para conjuntos de escalado de máquinas virtuales Windows

Un nodo del conjunto de escalado de máquinas virtuales tiene una sección **properties** que tiene un atributo **VirtualMachineProfile, extensionProfile**.
En **extensiones**, agregue los detalles para la extensión DSC.

La extensión DSC hereda las propiedades de extensión predeterminadas.
Para más información, consulte la [clase VirtualMachineScaleSetExtension](/dotnet/api/microsoft.azure.management.compute.models.virtualmachinescalesetextension?view=azure-dotnet).

```json
"extensionProfile": {
    "extensions": [
      {
        "name": "Microsoft.Powershell.DSC",
        "properties": {
          "publisher": "Microsoft.Powershell",
          "type": "DSC",
          "typeHandlerVersion": "2.77",
          "autoUpgradeMinorVersion": true,
          "protectedSettings": {
            "Items": {
              "registrationKeyPrivate": "[listKeys(resourceId('Microsoft.Automation/automationAccounts/', parameters('automationAccountName')), '2018-06-30').Keys[0].value]"
            }
          },
          "settings": {
            "Properties": [
              {
                "Name": "RegistrationKey",
                "Value": {
                  "UserName": "PLACEHOLDER_DONOTUSE",
                  "Password": "PrivateSettingsRef:registrationKeyPrivate"
                },
                "TypeName": "System.Management.Automation.PSCredential"
              },
              {
                "Name": "RegistrationUrl",
                "Value": "[reference(concat('Microsoft.Automation/automationAccounts/', parameters('automationAccountName'))).registrationUrl]",
                "TypeName": "System.String"
              },
              {
                "Name": "NodeConfigurationName",
                "Value": "[parameters('nodeConfigurationName')]",
                "TypeName": "System.String"
              }
            ]
          }
        }
      }
    ]
  }
```

## <a name="detailed-settings-information"></a>Información de configuración detallada

Use el esquema siguiente en la sección **settings** de la extensión DSC de Azure en una plantilla de Resource Manager.

Para una lista de los argumentos disponibles para el script de configuración predeterminada, consulte [Script de configuración predeterminada](#default-configuration-script).

```json
"settings": {
    "wmfVersion": "latest",
    "configuration": {
        "url": "http://validURLToConfigLocation",
        "script": "ConfigurationScript.ps1",
        "function": "ConfigurationFunction"
    },
    "configurationArguments": {
        "argument1": "Value1",
        "argument2": "Value2"
    },
    "configurationData": {
        "url": "https://foo.psd1"
    },
    "privacy": {
        "dataCollection": "enable"
    },
    "advancedOptions": {
        "downloadMappings": {
            "customWmfLocation": "http://myWMFlocation"
        }
    }
},
"protectedSettings": {
    "configurationArguments": {
        "parameterOfTypePSCredential1": {
            "userName": "UsernameValue1",
            "password": "PasswordValue1"
        },
        "parameterOfTypePSCredential2": {
            "userName": "UsernameValue2",
            "password": "PasswordValue2"
        }
    },
    "configurationUrlSasToken": "?g!bber1sht0k3n",
    "configurationDataUrlSasToken": "?dataAcC355T0k3N"
}
```

## <a name="details"></a>Detalles

| Nombre de propiedad | Tipo | Descripción |
| --- | --- | --- |
| settings.wmfVersion |string |Especifica la versión de Windows Management Framework (WMF) que debe instalarse en la máquina virtual. Si se establece esta propiedad en **latest**, se instala la versión más reciente de WMF. Actualmente, los únicos valores posibles para esta propiedad son **4.0**, **5.0**, **5.1** y **más recientes**. Estos valores posibles están sujetos a actualizaciones. El valor predeterminado es **latest**. |
| settings.configuration.url |string |Especifica la ubicación de la dirección URL desde la que descargar el archivo .zip de la configuración de DSC. Si la dirección URL proporcionada requiere un token de SAS para el acceso, establezca la propiedad **protectedSettings.configurationUrlSasToken** en el valor de su token de SAS. Esta propiedad es necesaria si se definen **settings.configuration.script** o **settings.configuration.function**. Si no se especifica ningún valor para estas propiedades, la extensión llamará al script de configuración predeterminada para establecer los metadatos del administrador de configuración de ubicación (LCM) y se deben proporcionar los argumentos. |
| settings.configuration.script |string |Especifica el nombre de archivo del script que contiene la definición de la configuración de DSC. Este script debe estar en la carpeta raíz del archivo .zip descargado de la dirección URL especificada por la propiedad **settings.configuration.url**. Esta propiedad es necesaria si se definen **settings.configuration.url** o **settings.configuration.script**. Si no se especifica ningún valor para estas propiedades, la extensión llama al script de configuración predeterminada para establecer los metadatos de LCM, y deben proporcionarse los argumentos. |
| settings.configuration.function |string |Especifica el nombre de la configuración de DSC. La configuración con nombre se debe incluir en el script que **settings.configuration.script** define. Esta propiedad es necesaria si se definen **settings.configuration.url** o **settings.configuration.function**. Si no se especifica ningún valor para estas propiedades, la extensión llama al script de configuración predeterminada para establecer los metadatos de LCM, y deben proporcionarse los argumentos. |
| settings.configurationArguments |Colección |Define los parámetros que desea pasar a la configuración de DSC. Esta propiedad no está cifrada. |
| settings.configurationData.url |string |Especifica la dirección URL desde la que descargar el archivo de datos de configuración (.psd1) que se usará como entrada para la configuración de DSC. Si la dirección URL proporcionada requiere un token de SAS para el acceso, establezca la propiedad **protectedSettings.configurationDataUrlSasToken** en el valor de su token de SAS. |
| settings.privacy.dataCollection |string |Habilita o deshabilita la recopilación de telemetría. Los únicos valores posibles para esta propiedad son **Enable**, **Disable**, **''** o **$null**. Si se deja esta propiedad en blanco o como null, se habilita la telemetría. El valor predeterminado es **''** . Para más información, consulte [Azure DSC extension data collection](https://devblogs.microsoft.com/powershell/azure-dsc-extension-data-collection-2/) (Colección de datos de la extensión DSC de Azure). |
| settings.advancedOptions.downloadMappings |Colección |Define las ubicaciones alternativas desde las que descargar WMF. Para más información, consulte el artículo sobre la [extensión DSC 2.8 de Azure y cómo asignar las descargas de las dependencias de la extensión a su propia ubicación](https://devblogs.microsoft.com/powershell/azure-dsc-extension-2-8-how-to-map-downloads-of-the-extension-dependencies-to-your-own-location/). |
| protectedSettings.configurationArguments |Colección |Define los parámetros que desea pasar a la configuración de DSC. Esta propiedad no está cifrada. |
| protectedSettings.configurationUrlSasToken |string |Especifica el token de SAS que se usa para acceder a la dirección URL que **settings.configuration.url** define. Esta propiedad no está cifrada. |
| protectedSettings.configurationDataUrlSasToken |string |Especifica el token de SAS que se usa para acceder a la dirección URL que **settings.configurationData.url** define. Esta propiedad no está cifrada. |

## <a name="default-configuration-script"></a>Script de configuración predeterminada

Para más información sobre los valores siguientes, consulte [Configuración básica del administración de configuración local](/powershell/scripting/dsc/managing-nodes/metaConfig#basic-settings).
Puede usar el script de configuración predeterminada de la extensión DSC para configurar solo las propiedades de LCM que aparecen en la tabla siguiente.

| Nombre de propiedad | Tipo | Descripción |
| --- | --- | --- |
| protectedSettings.configurationArguments.RegistrationKey |PSCredential |Propiedad obligatoria. Especifica la clave que se usa para que un nodo se registre en el servicio de Azure Automation, como la contraseña de un objeto de credencial de PowerShell. Este valor se puede detectar automáticamente a través del método **listkeys** con la cuenta de Automation.  Observe el [ejemplo](#example-using-referenced-azure-automation-registration-values). |
| settings.configurationArguments.RegistrationUrl |string |Propiedad obligatoria. Especifique la dirección URL del punto de conexión de Automation donde el nodo intenta registrarse. Este valor se puede detectar automáticamente a través del método **reference** con la cuenta de Automation. |
| settings.configurationArguments.NodeConfigurationName |string |Propiedad obligatoria. Especifica la configuración del nodo en la cuenta de Automation que se asignará al nodo. |
| settings.configurationArguments.ConfigurationMode |string |Especifica el modo de LCM. Entre las opciones válidas se incluyen **ApplyOnly**, **ApplyandMonitor** y **ApplyandAutoCorrect**.  El valor predeterminado es **ApplyandMonitor**. |
| settings.configurationArguments.RefreshFrequencyMins | uint32 | Especifica la frecuencia con que LCM intenta consultar a la cuenta de Automation para comprobar si hay actualizaciones.  El valor predeterminado es **30**.  El valor mínimo es **15**. |
| settings.configurationArguments.ConfigurationModeFrequencyMins | uint32 | Especifica la frecuencia con que el LCM valida la configuración actual. El valor predeterminado es **15**. El valor mínimo es **15**. |
| settings.configurationArguments.RebootNodeIfNeeded | boolean | Especifica si un nodo puede reiniciarse automáticamente si una operación de DSC lo solicita. El valor predeterminado es **false**. |
| settings.configurationArguments.ActionAfterReboot | string | Especifica lo que ocurre tras un reinicio al aplicar una configuración. Las opciones válidas son **ContinueConfiguration** y **StopConfiguration**. El valor predeterminado es **ContinueConfiguration**. |
| settings.configurationArguments.AllowModuleOverwrite | boolean | Especifica si el LCM sobrescribe los módulos existentes en el nodo. El valor predeterminado es **false**. |

## <a name="settings-vs-protectedsettings"></a>settings vs. protectedSettings

Todas las configuraciones se guardan en un archivo de texto de configuración en la máquina virtual.
Las propiedades que aparece en **settings** son propiedades públicas.
Las propiedades públicas no están cifradas en el archivo de texto de configuración.
Las propiedades que aparecen en **protectedSettings** se cifran con un certificado y no aparecen en texto sin formato en el archivo de configuración de la máquina virtual.

Si la configuración necesita credenciales, puede incluirlas en **protectedSettings**:

```json
"protectedSettings": {
    "configurationArguments": {
        "parameterOfTypePSCredential1": {
               "userName": "UsernameValue1",
               "password": "PasswordValue1"
        }
    }
}
```

## <a name="example-configuration-script"></a>Script de configuración de ejemplo

En el ejemplo siguiente se muestra el comportamiento predeterminado de la extensión DSC, que consiste en proporcionar la configuración de los metadatos al LCM y registrarse en el servicio de DSC de Automatización.
Los argumentos de configuración son obligatorios.
Los argumentos de configuración se transfieren al script de configuración predeterminada para establecer los metadatos del LCM.

```json
"settings": {
    "configurationArguments": {
        "RegistrationUrl" : "[parameters('registrationUrl1')]",
        "NodeConfigurationName" : "nodeConfigurationNameValue1"
    }
},
"protectedSettings": {
    "configurationArguments": {
        "RegistrationKey": {
            "userName": "NOT_USED",
            "Password": "registrationKey"
        }
    }
}
```

## <a name="example-using-the-configuration-script-in-azure-storage"></a>Ejemplo con el script de configuración en Azure Storage

El ejemplo siguiente proviene de la [información general del controlador de la extensión DSC](dsc-overview.md).
Este ejemplo usa plantillas de Resource Manager en lugar de cmdlets para implementar la extensión.
Guarde la configuración IisInstall.ps1, colóquela en un archivo .zip (ejemplo: `iisinstall.zip`) y cargue el archivo en una dirección URL accesible.
Este ejemplo usa Azure Blob Storage, pero puede descargar los archivos .zip desde cualquier ubicación arbitraria.

En la plantilla de Resource Manager, el código siguiente indica a la máquina virtual que descargue el archivo correcto y, luego, ejecute la función de PowerShell adecuada:

```json
"settings": {
    "configuration": {
        "url": "https://demo.blob.core.windows.net/iisinstall.zip",
        "script": "IisInstall.ps1",
        "function": "IISInstall"
    }
},
"protectedSettings": {
    "configurationUrlSasToken": "odLPL/U1p9lvcnp..."
}
```

## <a name="example-using-referenced-azure-automation-registration-values"></a>Ejemplo de uso de los valores de registro de Azure Automation a los que se hace referencia

En el ejemplo siguiente se obtienen **RegistrationUrl** y **RegistrationKey**, para lo que se hace referencia a las propiedades de la cuenta de Azure Automation y se utiliza el método **listkeys** para recuperar la clave principal (0).  En este ejemplo, se han proporcionado los parámetros **automationAccountName** y **NodeConfigName** a la plantilla.

```json
"settings": {
    "RegistrationUrl" : "[reference(concat('Microsoft.Automation/automationAccounts/', parameters('automationAccountName'))).registrationUrl]",
    "NodeConfigurationName" : "[parameters('NodeConfigName')]"
},
"protectedSettings": {
    "configurationArguments": {
        "RegistrationKey": {
            "userName": "NOT_USED",
            "Password": "[listKeys(resourceId('Microsoft.Automation/automationAccounts/', parameters('automationAccountName')), '2018-01-15').Keys[0].value]"
        }
    }
}
```

## <a name="update-from-a-previous-format"></a>Actualización desde un formato anterior

Todas las configuraciones que estén en un formato anterior de la extensión (y que tienen las propiedades públicas **ModulesUrl**, **ModuleSource**, **ModuleVersion**, **ConfigurationFunction**, **SasToken** o **Properties**) se adaptan automáticamente al formato actual de la extensión.
Se ejecuta tal como lo hacía antes.

El esquema siguiente muestra el aspecto que tenía el esquema de configuración anterior:

```json
"settings": {
    "WMFVersion": "latest",
    "ModulesUrl": "https://UrlToZipContainingConfigurationScript.ps1.zip",
    "SasToken": "SAS Token if ModulesUrl points to private Azure Blob Storage",
    "ConfigurationFunction": "ConfigurationScript.ps1\\ConfigurationFunction",
    "Properties": {
        "ParameterToConfigurationFunction1": "Value1",
        "ParameterToConfigurationFunction2": "Value2",
        "ParameterOfTypePSCredential1": {
            "UserName": "UsernameValue1",
            "Password": "PrivateSettingsRef:Key1"
        },
        "ParameterOfTypePSCredential2": {
            "UserName": "UsernameValue2",
            "Password": "PrivateSettingsRef:Key2"
        }
    }
},
"protectedSettings": {
    "Items": {
        "Key1": "PasswordValue1",
        "Key2": "PasswordValue2"
    },
    "DataBlobUri": "https://UrlToConfigurationDataWithOptionalSasToken.psd1"
}
```

Así se adapta el formato anterior al actual:

| Nombre de propiedad actual | Equivalente en el esquema anterior |
| --- | --- |
| settings.wmfVersion |settings.wmfVersion |
| settings.configuration.url |settings.ModulesUrl |
| settings.configuration.script |Primera parte de settings.ConfigurationFunction (antes de \\\\) |
| settings.configuration.function |Segunda parte de settings.ConfigurationFunction (después de \\\\) |
| settings.configuration.module.name | settings.ModuleSource |
| settings.configuration.module.version | settings.ModuleVersion |
| settings.configurationArguments |settings.Properties |
| settings.configurationData.url |protectedSettings.DataBlobUri (sin token de SAS) |
| settings.privacy.dataCollection |settings.Privacy.dataCollection |
| settings.advancedOptions.downloadMappings |settings.advancedOptions.downloadMappings |
| protectedSettings.configurationArguments |protectedSettings.Properties |
| protectedSettings.configurationUrlSasToken |settings.SasToken |
| protectedSettings.configurationDataUrlSasToken |Token de SAS de protectedSettings.DataBlobUri |

## <a name="troubleshooting"></a>Solución de problemas

Esto son algunos de los errores que pueden surgir y cómo se pueden corregir.

### <a name="invalid-values"></a>Valores no válidos

"Privacy.dataCollection is '{0}'. ("Privacy.dataCollection es "{0}")
The only possible values are '', 'Enable', and 'Disable'" (Privacy.dataCollection es '{0}'. Los únicos valores posibles son ", "Enable" y "Disable").
"WmfVersion is '{0}'. ("WmfVersion es "{0}")
Los únicos valores posibles son: and "latest" (WmfVersion es '{0}'. Los únicos valores posibles son... y "latest").

**Problema**: se proporcionó un valor no permitido.

**Solución**: cambie el valor no válido por un valor válido.
Para más información, consulte la tabla que aparece en [Detalles](#details).

### <a name="invalid-url"></a>Dirección URL no válida

"ConfigurationData.url is '{0}'. ("ConfigurationData.url es "{0}") This is not a valid URL" "DataBlobUri is '{0}'. (Esta dirección URL no es válida" "DataBlobUri es "{0}") This is not a valid URL" "Configuration.url is '{0}'. (Esta dirección URL no es válida" "Configuration.url es "{0}") This is not a valid URL" This is not a valid URL" (Configuration.url es '{0}'. No es una dirección URL válida)

**Problema**: una URL proporcionada no es válida.

**Solución**: revise todas las direcciones URL proporcionadas.
Asegúrese de que todas las direcciones URL se resuelvan en ubicaciones válidas a las que la extensión pueda acceder en la máquina remota.

### <a name="invalid-registrationkey-type"></a>Tipo RegistrationKey no válido

"Tipo no válido para el parámetro RegistrationKey del tipo PSCredential."

**Problema**: el valor de *RegistrationKey* en protectedSettings.configurationArguments no se puede proporcionar como cualquier tipo que no sea PSCredential.

**Solución**: cambie la entrada de protectedSettings.configurationArguments de RegistrationKey a un tipo PSCredential con el formato siguiente:

```json
"configurationArguments": {
    "RegistrationKey": {
        "userName": "NOT_USED",
        "Password": "RegistrationKey"
    }
}
```

### <a name="invalid-configurationargument-type"></a>Tipo de ConfigurationArgument no válido

"Invalid configurationArguments type {0}" ("Tipo {0} de configurationArguments" no válido)

**Problema**: la propiedad *ConfigurationArguments* no se puede resolver en un objeto **Tabla hash**.

**Solución**: convierta la propiedad *ConfigurationArguments* en una **Tabla hash**.
Siga el formato proporcionado en los ejemplos anteriores. Esté atento a las comillas, comas y llaves.

### <a name="duplicate-configurationarguments"></a>ConfigurationArguments duplicadas

"Found duplicate arguments '{0}' in both public and protected configurationArguments" (Se encontraron argumentos duplicados "{0}" en propiedades configurationArguments públicas y privadas)

**Problema**: la propiedad *ConfigurationArguments* en la configuración pública y la propiedad *ConfigurationArguments* en la configuración protegida contienen propiedades con el mismo nombre.

**Solución**: quite una de las propiedades duplicadas.

### <a name="missing-properties"></a>Propiedades que faltan

"settings.Configuration.function requiere que se especifiquen settings.configuration.url o settings.configuration.module"

"settings.Configuration.url requiere que se especifique settings.configuration.script"

"settings.Configuration.script requiere que se especifique settings.configuration.url"

"settings.Configuration.url requiere que se especifique settings.configuration.function"

"protectedSettings.ConfigurationUrlSasToken requieres que se especifique settings.configuration.url"

"protectedSettings.ConfigurationDataUrlSasToken requiere que se especifique settings.configurationData.url"

**Problema**: una propiedad definida requiere otra propiedad que falta.

**Soluciones**:

- Proporcione la propiedad que falta.
- Quite la propiedad que necesita la propiedad que falta.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre el [uso de los conjuntos de escalado de máquinas virtuales con la extensión DSC de Azure](../../virtual-machine-scale-sets/virtual-machine-scale-sets-dsc.md).
- Encuentre más detalles sobre la [administración segura de credenciales de DSC](dsc-credentials.md).
- Obtenga una [introducción al controlador de la extensión DSC de Azure](dsc-overview.md).
- Para obtener más información sobre DSC de PowerShell, vaya al [centro de documentación de PowerShell](/powershell/scripting/dsc/overview/overview).
