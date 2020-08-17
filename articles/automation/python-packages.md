---
title: Administrar paquetes de Python 2 en Azure Automation
description: En este artículo se describe cómo administrar paquetes de Python 2 en Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 02/25/2019
ms.topic: conceptual
ms.custom: devx-track-python
ms.openlocfilehash: 0b0fab40eb95fabd4b74e809c9e6cd6f6573f513
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852912"
---
# <a name="manage-python-2-packages-in-azure-automation"></a>Administrar paquetes de Python 2 en Azure Automation

Azure Automation le permite ejecutar runbooks de Python 2 en Azure y en Hybrid Runbook Worker de Linux. Para simplificar los runbooks, puede usar los paquetes de Python para importar los módulos que necesite. En este artículo se describe cómo administrar y usar paquetes de Python en Azure Automation.

## <a name="import-packages"></a>Importación de paquetes

En la cuenta de Automation, seleccione **Paquetes de Python 2** en **Recursos compartidos**. Haga clic en **+ Add a Python 2 package** (+ Agregar un paquete de Python 2).

![Agregar un paquete de Python](media/python-packages/add-python-package.png)

En la página Agregar un paquete de Python 2, seleccione un paquete local para cargar. El paquete puede ser un archivo **.whl** o **.tar.gz**. Cuando el paquete esté seleccionado, haga clic en **Aceptar** para cargarlo.

![Agregar un paquete de Python](media/python-packages/upload-package.png)

Una vez que se haya importado un paquete, este se muestra en la página de Paquetes de Python 2 en su cuenta de Automation. Si necesita quitar un paquete, selecciónelo y haga clic en **Eliminar**.

![Lista de paquetes](media/python-packages/package-list.png)

## <a name="import-packages-with-dependencies"></a>Importación de paquetes con dependencias

Azure Automation no resuelve las dependencias de los paquetes de Python durante el proceso de importación. Hay dos formas de importar un paquete con todas sus dependencias. Solo es necesario realizar uno de los pasos siguientes para importar los paquetes en su cuenta de Automation.

### <a name="manually-download"></a>Descarga manual

En una máquina Windows de 64 bits con [Python 2.7](https://www.python.org/downloads/release/latest/python2) y [pip](https://pip.pypa.io/en/stable/) instalados, ejecute el siguiente comando para descargar un paquete y todas sus dependencias:

```cmd
C:\Python27\Scripts\pip2.7.exe download -d <output dir> <package name>
```

Una vez que se descargan los paquetes, puede importarlos en su cuenta de Automation.

### <a name="runbook"></a>Runbook

 Para obtener un runbook de Python, [importe paquetes de Python 2 desde pypi a la cuenta de Azure Automation](https://gallery.technet.microsoft.com/scriptcenter/Import-Python-2-packages-57f7d509) desde la galería hasta la cuenta de Automation. Asegúrese de que los parámetros de ejecución están establecidos en **Azure** e inicie el runbook con los parámetros. El runbook requiere una cuenta de ejecución para que la cuenta de Automation funcione. Asegúrese de iniciar cada parámetro con el modificador, como se ve en la lista e imagen siguientes:

* -s \<subscriptionId\>
* -g \<resourceGroup\>
* -a \<automationAccount\>
* -m \<modulePackage\>

![Lista de paquetes](media/python-packages/import-python-runbook.png)

El runbook permite especificar el paquete que se va a descargar. Por ejemplo, el uso del parámetro `Azure` descarga todos los módulos de Azure y todas las dependencias (aproximadamente 105).

Una vez completado el runbook, puede consultar la página **Paquetes de Python 2** en **Recursos compartidos** en su cuenta de Automation para comprobar que el paquete se importó correctamente.

## <a name="use-a-package-in-a-runbook"></a>Usar un paquete en un runbook

Si tiene un paquete importado, puede usarlo en un runbook. En el ejemplo siguiente se usa el [ paquete de utilidades de Azure Automation](https://github.com/azureautomation/azure_automation_utility). Este paquete facilita el uso de Python con Azure Automation. Para usar el paquete, siga las instrucciones del repositorio de GitHub y agréguelo al runbook. Por ejemplo, puede usar `from azure_automation_utility import get_automation_runas_credential` para importar la función para recuperar la cuenta de ejecución.

```python
import azure.mgmt.resource
import automationassets
from azure_automation_utility import get_automation_runas_credential

# Authenticate to Azure using the Azure Automation RunAs service principal
runas_connection = automationassets.get_automation_connection("AzureRunAsConnection")
azure_credential = get_automation_runas_credential()

# Intialize the resource management client with the RunAs credential and subscription
resource_client = azure.mgmt.resource.ResourceManagementClient(
    azure_credential,
    str(runas_connection["SubscriptionId"]))

# Get list of resource groups and print them out
groups = resource_client.resource_groups.list()
for group in groups:
    print group.name
```

## <a name="develop-and-test-runbooks-offline"></a>Desarrollar y probar runbooks sin conexión

Para desarrollar y probar los runbooks de Python 2 desconectado, puede usar el módulo [Recursos emulados de Python de Azure Automation](https://github.com/azureautomation/python_emulated_assets) en GitHub. Este módulo le permite hacer referencia a recursos compartidos como credenciales, variables, conexiones y certificados.

## <a name="next-steps"></a>Pasos siguientes

Para preparar un runbook de Python, consulte [Creación de un runbook de Python](learn/automation-tutorial-runbook-textual-python2.md).
