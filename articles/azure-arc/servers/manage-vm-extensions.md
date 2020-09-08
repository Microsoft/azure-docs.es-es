---
title: Administración de extensiones de máquina virtual con servidores habilitados para Azure Arc (versión preliminar)
description: Los servidores habilitados para Azure Arc (versión preliminar) pueden administrar la implementación de extensiones de máquina virtual que proporcionan tareas de automatización y configuración posteriores a la implementación con máquinas virtuales que no son de Azure.
ms.date: 06/17/2020
ms.topic: conceptual
ms.openlocfilehash: 259a55c0e5cb4074871fd70a61e6c61d5cf3c50a
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018168"
---
# <a name="virtual-machine-extension-management-with-azure-arc-enabled-servers-preview"></a>Administración de extensiones de máquina virtual con servidores habilitados para Azure Arc (versión preliminar)

Las extensiones de máquina virtual (VM) son pequeñas aplicaciones que realizan tareas de automatización y configuración posteriores a la implementación en máquinas virtuales de Azure. Por ejemplo, si una máquina virtual necesita que se instale software, protección antivirus o ejecutar un script en ella, se puede usar una extensión de máquina virtual.

Los servidores habilitados para Azure Arc (versión preliminar) le permite implementar extensiones de máquina virtual de Azure en máquinas virtuales Windows y Linux que no son de Azure, lo que simplifica la administración de su máquina híbrida local, perimetral y otros entornos en la nube durante su ciclo de vida.

## <a name="key-benefits"></a>Ventajas principales

La compatibilidad con la extensión de máquina virtual de servidores habilitados para Azure Arc (versión preliminar) ofrece las siguientes ventajas principales:

* Usar [Azure Automation State Configuration](../../automation/automation-dsc-overview.md) para almacenar de forma centralizada las configuraciones y mantener el estado deseado de las máquinas conectadas híbridas que se hayan habilitado a través de la extensión de máquina virtual de DSC.

* Recopilar datos de registro para su análisis con los [registros de Azure Monitor](../../azure-monitor/platform/data-platform-logs.md) habilitados a través de la extensión de máquina virtual del agente de Log Analytics. Son especialmente útiles para llevar a cabo un análisis complejo de datos procedentes de orígenes diversos.

* Con [Azure Monitor para VM](../../azure-monitor/insights/vminsights-overview.md), se analiza el rendimiento de las VM Windows y Linux, y se supervisa el impacto de sus procesos y dependencias en otros recursos y procesos externos. Esto se logra al habilitar las extensiones de máquina virtual del agente de Log Analytics y Dependency Agent.

* Descargue y ejecute scripts en máquinas conectadas híbridas mediante la extensión de script personalizado. Esta extensión es útil para la configuración posterior a la implementación, la instalación de software o cualquier otra tarea de configuración o administración.

## <a name="availability"></a>Disponibilidad

La funcionalidad de la extensión de máquina virtual solo está disponible en la lista de [regiones admitidas](overview.md#supported-regions). Asegúrese de incorporar el equipo en una de estas regiones.

## <a name="extensions"></a>Extensiones

En esta versión preliminar se admiten las siguientes extensiones de máquina virtual en máquinas Windows y Linux.

|Extensión |SO |Publicador |Información adicional |
|----------|---|----------|-----------------------|
|CustomScriptExtension |Windows |Microsoft.Compute |[Extensión de script personalizado de Windows](../../virtual-machines/extensions/custom-script-windows.md)|
|DSC |Windows |Microsoft.PowerShell|[Extensión DSC de Windows PowerShell](../../virtual-machines/extensions/dsc-windows.md)|
|Agente de Log Analytics |Windows |Microsoft.EnterpriseCloud.Monitoring |[Extensión de máquina virtual de Log Analytics para Windows](../../virtual-machines/extensions/oms-windows.md)|
|Microsoft Dependency Agent | Windows |Microsoft.Compute | [Extensión de máquina virtual de Dependency Agent para Windows](../../virtual-machines/extensions/agent-dependency-windows.md)|
|CustomScript|Linux |Microsoft.Azure.Extension |[Extensión de script personalizado de Linux versión 2](../../virtual-machines/extensions/custom-script-linux.md) |
|DSC |Linux |Microsoft.OSTCExtensions |[Extensión DSC de PowerShell para Linux](../../virtual-machines/extensions/dsc-linux.md) |
|Agente de Log Analytics |Linux |Microsoft.EnterpriseCloud.Monitoring |[Extensión de máquina virtual de Log Analytics para Linux](../../virtual-machines/extensions/oms-linux.md) |
|Microsoft Dependency Agent | Linux |Microsoft.Compute | [Extensión de máquina virtual de Dependency Agent para Linux](../../virtual-machines/extensions/agent-dependency-linux.md) |

Las extensiones de máquina virtual se pueden ejecutar con plantillas de Azure Resource Manager, desde Azure Portal o Azure PowerShell en servidores híbridos administrados por servidores habilitados para Arc (versión preliminar).

Para obtener información sobre el paquete del agente de Azure Connected Machine y los detalles del componente del agente de extensión, consulte [Información general del agente](agent-overview.md#agent-component-details).

## <a name="prerequisites"></a>Requisitos previos

Esta característica depende de los siguientes proveedores de recursos de Azure en su suscripción:

* **Microsoft.HybridCompute**
* **Microsoft.GuestConfiguration**

Si aún no están registrados, siga los pasos descritos en [Registro de proveedores de recursos de Azure](agent-overview.md#register-azure-resource-providers).

La extensión de máquina virtual del agente de Log Analytics para Linux requiere que Python 2.x esté instalado en el equipo de destino.

### <a name="connected-machine-agent"></a>Agente de Connected Machine

Compruebe que el equipo coincida con las [versiones compatibles](agent-overview.md#supported-operating-systems) de los sistemas operativos Windows y Linux para el agente de Azure Connected Machine.

La versión mínima del agente de Connected Machine que es compatible con esta característica es la siguiente:

* Windows: 0.7.x
* Linux: 0.8.x

Para actualizar la máquina a la versión requerida del agente, consulte [Actualización del agente](manage-agent.md#upgrading-agent).

## <a name="enable-extensions-from-the-portal"></a>Habilitación de extensiones desde el portal

Las extensiones de máquina virtual se pueden aplicar a la máquina administrada de Arc para servidores (versión preliminar) mediante Azure Portal.

1. En el explorador, vaya a [Azure Portal](https://aka.ms/arcserver-preview).

2. En el portal, vaya a **Máquinas - Azure Arc** y seleccione la máquina híbrida de la lista.

3. Elija **Extensiones** y, después, seleccione **Agregar**. Elija la extensión que quiera en la lista de extensiones disponibles y siga las instrucciones del asistente. En este ejemplo, se implementará la extensión de máquina virtual de Log Analytics. 

    ![Selección de la extensión de máquina virtual para la máquina seleccionada](./media/manage-vm-extensions/add-vm-extensions.png)

    En el ejemplo siguiente se muestra la instalación de la extensión de máquina virtual de Log Analytics desde Azure Portal:

    ![Instalación de la extensión de máquina virtual de Log Analytics](./media/manage-vm-extensions/mma-extension-config.png)

    Para completar la instalación, debe proporcionar el id. del área de trabajo y la clave principal. Si no está familiarizado con el método para encontrar esta información, consulte [Obtención del id. y la clave del área de trabajo](../../azure-monitor/platform/log-analytics-agent.md#workspace-id-and-key).

4. Después de confirmar la información necesaria que proporcionó, seleccione **Crear**. Se muestra un resumen de la implementación y puede revisar el estado de esta.

>[!NOTE]
>Aunque puede procesar por lotes varias extensiones, estas se instalan en serie. Una vez completada la instalación de la primera extensión, se intenta realizar la instalación de la siguiente.

## <a name="azure-resource-manager-templates"></a>Plantillas del Administrador de recursos de Azure

Las extensiones de máquina virtual se pueden agregar a una plantilla de Azure Resource Manager y ejecutar con la implementación de la plantilla. En el caso de las extensiones de máquina virtual compatibles con servidores habilitados para Arc (versión preliminar), puede implementar la extensión de máquina virtual admitida en máquinas Linux o Windows mediante Azure PowerShell. Cada ejemplo a continuación incluye un archivo de plantilla y un archivo de parámetros con valores de ejemplo para la plantilla.

>[!NOTE]
>Aunque varias extensiones se pueden procesar por lotes, estas se instalan en serie. Una vez completada la instalación de la primera extensión, se intenta la instalación de la siguiente.

### <a name="deploy-the-log-analytics-vm-extension"></a>Implementación de la extensión de máquina virtual de Log Analytics

Para implementar fácilmente el agente de Log Analytics, el siguiente ejemplo puede instalar el agente en Windows o Linux.

#### <a name="template-file-for-linux"></a>Archivo de plantilla para Linux

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "workspaceId": {
            "type": "string"
        },
        "workspaceKey": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[concat(parameters('vmName'),'/OMSAgentForLinux')]",
            "type": "Microsoft.HybridCompute/machines/extensions",
            "location": "[parameters('location')]",
            "apiVersion": "2019-08-02-preview",
            "properties": {
                "publisher": "Microsoft.EnterpriseCloud.Monitoring",
                "type": "OmsAgentForLinux",
                "settings": {
                    "workspaceId": "[parameters('workspaceId')]"
                },
                "protectedSettings": {
                    "workspaceKey": "[parameters('workspaceKey')]"
                }
            }
        }
    ]
}
```

#### <a name="template-file-for-windows"></a>Archivo de plantilla para Windows

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "workspaceId": {
            "type": "string"
        },
        "workspaceKey": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[concat(parameters('vmName'),'/MicrosoftMonitoringAgent')]",
            "type": "Microsoft.HybridCompute/machines/extensions",
            "location": "[parameters('location')]",
            "apiVersion": "2019-08-02-preview",
            "properties": {
                "publisher": "Microsoft.EnterpriseCloud.Monitoring",
                "type": "MicrosoftMonitoringAgent",
                "autoUpgradeMinorVersion": true,
                "settings": {
                    "workspaceId": "[parameters('workspaceId')]"
                },
                "protectedSettings": {
                    "workspaceKey": "[parameters('workspaceKey')]"
                }
            }
        }
    ]
}
```

#### <a name="parameter-file"></a>Archivo de parámetros

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "value": "<vmName>"
        },
        "location": {
            "value": "<region>"
        },
        "workspaceId": {
            "value": "<MyWorkspaceID>"
        },
        "workspaceKey": {
            "value": "<MyWorkspaceKey>"
        }
    }
}
```

Guarde la plantilla y los archivos de parámetros en el disco y edite el archivo de parámetros con los valores adecuados para su implementación. Después, podrá instalar la extensión en todas las máquinas conectadas dentro de un grupo de recursos con el siguiente comando. El comando usa el parámetro *TemplateFile* para especificar la plantilla y el parámetro *TemplateParameterFile* para especificar un archivo que contenga los parámetros y sus valores correspondientes.

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "ContosoEngineering" -TemplateFile "D:\Azure\Templates\LogAnalyticsAgentWin.json" -TemplateParameterFile "D:\Azure\Templates\LogAnalyticsAgentWinParms.json"
```

### <a name="deploy-the-custom-script-extension"></a>Implementación de la extensión de script personalizado

Para usar la extensión de script personalizado, el siguiente ejemplo puede ejecutarse en Windows y Linux. Si no está familiarizado con la extensión de script personalizado, consulte [Extensión de script personalizado para Windows](../../virtual-machines/extensions/custom-script-windows.md) o [Extensión de script personalizado para Linux](../../virtual-machines/extensions/custom-script-linux.md). Hay un par de características distintas que debe comprender al usar esta extensión con máquinas híbridas:

* La lista de sistemas operativos compatibles con la extensión de script personalizado para máquina virtual de Azure no se aplica a servidores habilitados para Azure Arc. Puede encontrar [aquí](agent-overview.md#supported-operating-systems) la lista de sistemas operativos compatibles con servidores habilitados para Arc.

* No se aplican los detalles de configuración relacionados con Azure Virtual Machine Scale Sets o las máquinas virtuales clásicas.

* Si las máquinas necesitan descargar un script de forma externa y solo se pueden comunicar a través de un servidor proxy, debe [configurar el agente de Connected Machine](manage-agent.md#update-or-remove-proxy-settings) para establecer la variable de entorno del servidor proxy.

La configuración de la extensión de script personalizado especifica aspectos como la ubicación del script y el comando que se ejecutará. Esta configuración se especifica en una plantilla de Azure Resource Manager, que se muestra a continuación para las máquinas híbridas Windows y Linux.

#### <a name="template-file-for-linux"></a>Archivo de plantilla para Linux

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string"
    },
    "location": {
      "type": "string"
    },
    "fileUris": {
      "type": "array"
    },
    "commandToExecute": {
      "type": "securestring"
    }
  },
  "resources": [
    {
      "name": "[concat(parameters('vmName'),'/CustomScript')]",
      "type": "Microsoft.HybridCompute/machines/extensions",
      "location": "[parameters('location')]",
      "apiVersion": "2019-08-02-preview",
      "properties": {
        "publisher": "Microsoft.Azure.Extensions",
        "type": "CustomScript",
        "autoUpgradeMinorVersion": true,
        "settings": {},
        "protectedSettings": {
          "commandToExecute": "[parameters('commandToExecute')]",
          "fileUris": "[parameters('fileUris')]"
        }
      }
    }
  ]
}
```

#### <a name="template-file-for-windows"></a>Archivo de plantilla para Windows

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "fileUris": {
            "type": "string"
        },
        "arguments": {
            "type": "securestring",
            "defaultValue": " "
        }
    },
    "variables": {
        "UriFileNamePieces": "[split(parameters('fileUris'), '/')]",
        "firstFileNameString": "[variables('UriFileNamePieces')[sub(length(variables('UriFileNamePieces')), 1)]]",
        "firstFileNameBreakString": "[split(variables('firstFileNameString'), '?')]",
        "firstFileName": "[variables('firstFileNameBreakString')[0]]"
    },
    "resources": [
        {
            "name": "[concat(parameters('vmName'),'/CustomScriptExtension')]",
            "type": "Microsoft.HybridCompute/machines/extensions",
            "location": "[parameters('location')]",
            "apiVersion": "2019-08-02-preview",
            "properties": {
                "publisher": "Microsoft.Compute",
                "type": "CustomScriptExtension",
                "autoUpgradeMinorVersion": true,
                "settings": {
                    "fileUris": "[split(parameters('fileUris'), ' ')]"
                },
                "protectedSettings": {
                    "commandToExecute": "[concat ('powershell -ExecutionPolicy Unrestricted -File ', variables('firstFileName'), ' ', parameters('arguments'))]"
                }
            }
        }
    ]
}
```

#### <a name="parameter-file"></a>Archivo de parámetros

```json
{
  "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
  "handler": "Microsoft.Azure.CreateUIDef",
  "version": "0.1.2-preview",
  "parameters": {
    "basics": [
      {}
    ],
    "steps": [
      {
        "name": "customScriptExt",
        "label": "Add Custom Script Extension",
        "elements": [
          {
            "name": "fileUris",
            "type": "Microsoft.Common.FileUpload",
            "label": "Script files",
            "toolTip": "The script files that will be downloaded to the virtual machine.",
            "constraints": {
              "required": false
            },
            "options": {
              "multiple": true,
              "uploadMode": "url"
            },
            "visible": true
          },
          {
            "name": "commandToExecute",
            "type": "Microsoft.Common.TextBox",
            "label": "Command",
            "defaultValue": "sh script.sh",
            "toolTip": "The command to execute, for example: sh script.sh",
            "constraints": {
              "required": true
            },
            "visible": true
          }
        ]
      }
    ],
    "outputs": {
      "vmName": "[vmName()]",
      "location": "[location()]",
      "fileUris": "[steps('customScriptExt').fileUris]",
      "commandToExecute": "[steps('customScriptExt').commandToExecute]"
    }
  }
}
```

### <a name="deploy-the-powershell-dsc-extension"></a>Implementación de la extensión DSC de PowerShell

Para usar la extensión DSC de PowerShell, el siguiente ejemplo puede ejecutarse en Windows y Linux. Si no está familiarizado con la extensión DSC de PowerShell, consulte [Introducción al controlador de extensiones DSC](../../virtual-machines/extensions/dsc-overview.md). Hay un par de características distintas que debe comprender al usar esta extensión con máquinas híbridas:

* La lista de sistemas operativos compatibles con la extensión DSC de PowerShell para máquina virtual de Azure no se aplica a los servidores habilitados para Azure Arc. Puede encontrar [aquí](agent-overview.md#supported-operating-systems) la lista de sistemas operativos compatibles con servidores habilitados para Arc.

* Si las máquinas necesitan descargar un script de forma externa y solo se pueden comunicar a través de un servidor proxy, debe [configurar el agente de Connected Machine](manage-agent.md#update-or-remove-proxy-settings) para establecer la variable de entorno del servidor proxy.

#### <a name="template-file-for-linux"></a>Archivo de plantilla para Linux

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string",
            "metadata": {
                "description": "Name of the vm, will be used as DNS Name for the Public IP used to access the Virtual Machine."
            }
        },
        "location": {
            "type": "string",
            "metadata": {
                "description": "Location for all resources."
            }
        },
        "mode": {
            "type": "string",
            "defaultValue": "Push",
            "metadata": {
                "description": "The functional mode, push MOF configuration (Push), distribute MOF configuration (Pull), install custom DSC module (Install)"
            },
            "allowedValues": [
                "Push",
                "Pull",
                "Install",
                "Register"
            ]
        },
        "fileUri": {
            "type": "string",
            "defaultValue": "",
            "metadata": {
                "description": "The uri of the MOF file/Meta MOF file/resource ZIP file"
            }
        },
        "registrationUrl": {
            "type": "string",
            "defaultValue": "",
            "metadata": {
                "description": "The URL of the Azure Automation account"
            }
        },
        "registrationKey": {
            "type": "string",
            "defaultValue": "",
            "metadata": {
                "description": "The access key of the Azure Automation account"
            }
        }
    },
    "resources": [
        {
            "name": "[concat(parameters('vmName'),'/DSCForLinux')]",
            "type": "Microsoft.HybridCompute/machines/extensions",
            "location": "[parameters('location')]",
            "apiVersion": "2019-08-02-preview",
            "properties": {
                "publisher": "Microsoft.OSTCExtensions",
                "type": "DSCForLinux",
                "settings": {
                    "Mode": "[parameters('mode')]",
                    "FileUri": "[parameters('fileUri')]"
                },
                "protectedSettings": {
                    "RegistrationUrl": "[parameters('registrationUrl')]",
                    "RegistrationKey": "[parameters('registrationKey')]"
                }
            }
        }
    ]
}
```

#### <a name="template-file-for-windows"></a>Archivo de plantilla para Windows

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "modulesUrl": {
            "type": "string"
        },
        "configurationFunction": {
            "type": "string"
        },
        "properties": {
            "type": "string",
            "defaultValue": ""
        },
        "dataBlobUri": {
            "type": "string",
            "defaultValue": ""
        },
        "wmfVersion": {
            "type": "string",
            "defaultValue": "latest",
            "allowedValues": [
                "4.0",
                "5.0",
                "5.1",
                "latest"
            ]
        },
        "privacy": {
            "type": "string",
            "defaultValue": ""
        },
        "autoUpdate": {
            "type": "bool",
            "defaultValue": false
        }
    },
    "resources": [
        {
            "name": "[concat(parameters('vmName'),'/Microsoft.Powershell.DSC')]",
            "type": "Microsoft.HybridCompute/machines/extensions",
            "location": "[parameters('location')]",
            "apiVersion": "2019-08-02-preview",
            "properties": {
                "publisher": "Microsoft.Powershell",
                "type": "DSC",
                "autoUpgradeMinorVersion": "[parameters('autoUpdate')]",
                "settings": {
                    "ModulesUrl": "[parameters('modulesUrl')]",
                    "ConfigurationFunction": "[parameters('configurationFunction')]",
                    "Properties": "[parameters('properties')]",
                    "WmfVersion": "[parameters('wmfVersion')]",
                    "Privacy": {
                        "DataCollection": "[parameters('privacy')]"
                    }
                },
                "protectedSettings": {
                    "DataBlobUri": "[parameters('dataBlobUri')]"
                }
            }
        }
    ]
}
```

#### <a name="parameter-file"></a>Archivo de parámetros

```json
{
  "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
  "handler": "Microsoft.Azure.CreateUIDef",
  "version": "0.1.2-preview",
  "parameters": {
    "basics": [
      {}
    ],
    "steps": [
      {
        "name": "dscExtension",
        "label": "Add DSC Extension",
        "elements": [
          {
            "name": "Mode",
            "type": "Microsoft.Common.OptionsGroup",
            "label": "Mode",
            "defaultValue": 0,
            "toolTip": "The functional mode, push MOF configuration (Push), distribute MOF configuration (Pull), install custom DSC module (Install)",
            "constraints": {
              "allowedValues": [
                {
                  "label": "Push",
                  "value": "Push"
                },
                {
                  "label": "Pull",
                  "value": "Pull"
                },
                {
                  "label": "Install",
                  "value": "Install"
                },
                {
                  "label": "Register",
                  "value": "Register"
                }
              ]
            },
            "visible": true
          },
          {
            "name": "FileUri",
            "type": "Microsoft.Common.FileUpload",
            "label": "File URI",
            "toolTip": "The uri of the MOF file/Meta MOF file/resource ZIP file",
            "constraints": {
              "required": false,
              "accept": ".psd1"
            },
            "options": {
              "multiple": false,
              "uploadMode": "url",
              "openMode": "binary",
              "encoding": "UTF-8"
            }
          },
          {
            "name": "RegistrationUrl",
            "type": "Microsoft.Common.TextBox",
            "label": "Registration URL",
            "toolTip": "The URL of the Azure Automation account",
            "constraints": {
              "required": false
            }
          },
          {
            "name": "RegistrationKey",
            "type": "Microsoft.Common.TextBox",
            "label": "Registration key",
            "toolTip": "The access key of the Azure Automation account",
            "constraints": {
              "required": false
            }
          }
        ]
      }
    ],
    "outputs": {
      "vmName": "[vmName()]",
      "location": "[location()]",
      "mode": "[steps('dscExtension').Mode]",
      "fileUri": "[steps('dscExtension').FileUri]",
      "registrationUrl": "[steps('dscExtension').RegistrationUrl]",
      "registrationKey": "[steps('dscExtension').RegistrationKey]"
    }
  }
}
```

### <a name="dependency-agent"></a>Dependency Agent

Para usar la extensión de Dependency Agent de Azure Monitor, el siguiente ejemplo puede ejecutarse en Windows y Linux. Si no está familiarizado con Dependency Agent, consulte [Información general sobre los agentes de Azure Monitor](../../azure-monitor/platform/agents-overview.md#dependency-agent).

#### <a name="template-file-for-linux"></a>Archivo de plantilla para Linux

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "metadata": {
        "description": "The name of existing Linux machine."
      }
    }
  },
  "variables": {
    "vmExtensionsApiVersion": "2017-03-30"
  },
  "resources": [
    {
      "type": "Microsoft.HybridCompute/machines/extensions",
      "name": "[concat(parameters('vmName'),'/DAExtension')]",
      "apiVersion": "[variables('vmExtensionsApiVersion')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
      ],
      "properties": {
        "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
        "type": "DependencyAgentLinux",
        "typeHandlerVersion": "9.5",
        "autoUpgradeMinorVersion": true
      }
    }
  ],
    "outputs": {
    }
}
```

#### <a name="template-file-for-windows"></a>Archivo de plantilla para Windows

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "metadata": {
        "description": "The name of existing Windows machine."
      }
    }
  },
  "variables": {
    "vmExtensionsApiVersion": "2017-03-30"
  },
  "resources": [
    {
      "type": "Microsoft.HybridCompute/machines/extensions",
      "name": "[concat(parameters('vmName'),'/DAExtension')]",
      "apiVersion": "[variables('vmExtensionsApiVersion')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
      ],
      "properties": {
        "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
        "type": "DependencyAgentWindows",
        "typeHandlerVersion": "9.5",
        "autoUpgradeMinorVersion": true
      }
    }
  ],
    "outputs": {
    }
}
```

## <a name="troubleshooting"></a>Solución de problemas

Los datos sobre el estado de las implementaciones de extensiones pueden recuperarse desde Azure Portal.

Los pasos de solución de problemas siguientes se aplican a todas las extensiones de máquina virtual.

1. Para comprobar el registro de agente invitado, examine la actividad cuando la extensión se aprovisionó en `%SystemDrive%\ProgramData\GuestConfig\ext_mgr_logs` para Windows y en `/var/lib/GuestConfig/ext_mgr_logs` para Linux.

2. Compruebe los registros de la extensión específica para obtener más detalles en `%SystemDrive%\ProgramData\GuestConfig\extension_logs\<Extension>` para Windows. La salida de la extensión se registra en un archivo para cada extensión instalada en Linux en la ruta `/var/log/GuestConfig/extension_logs`.

3. Revise las secciones de solución de problemas de la documentación específica de la extensión para ver los códigos de errores, los problemas conocidos, etc. Puede encontrar información adicional para solucionar los problemas de cada extensión en la sección **Solución de problemas y soporte técnico** de la información general de la extensión. Esto incluye la descripción de los códigos de error que se escriben en el registro. Puede consultar los vínculos para los artículos sobre la extensión en la [tabla de extensiones](#extensions) que se encuentra anteriormente en este artículo.

4. Examine los registros del sistema. Compruebe si hay otras operaciones que puedan haber interferido con la extensión, como una instalación de ejecución prolongada de otra aplicación que requería acceso exclusivo al administrador de paquetes.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo administrar la máquina con [Azure Policy](../../governance/policy/overview.md) para, por ejemplo, la [configuración de invitado](../../governance/policy/concepts/guest-configuration.md) de VM, la comprobación de que la máquina informa al área de trabajo de Log Analytics esperada, la habilitación de la supervisión con [Azure Monitor con máquinas virtuales](../../azure-monitor/insights/vminsights-enable-policy.md) y mucho más.

- Más información sobre el [agente de Log Analytics](../../azure-monitor/platform/log-analytics-agent.md). El agente de Log Analytics para Windows y Linux es necesario si desea supervisar de forma proactiva el sistema operativo y las cargas de trabajo que se ejecutan en la máquina, administrarlos mediante runbooks de Automation o soluciones como Update Management, o usar otros servicios de Azure como [Azure Security Center](../../security-center/security-center-intro.md).