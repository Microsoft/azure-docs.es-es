---
title: Creación de un runbook de Python en Azure Automation
description: Este artículo le enseña a crear, probar y publicar un sencillo runbook de Python.
services: automation
ms.subservice: process-automation
ms.date: 04/19/2020
ms.topic: tutorial
ms.custom: has-adal-ref, devx-track-python
ms.openlocfilehash: e12327651165606e6a9b571d410f547a09a8ec8e
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87847931"
---
# <a name="tutorial-create-a-python-runbook"></a>Tutorial: Creación de un runbook de Python

Este tutorial le guiará por la creación de un [runbook de Python](../automation-runbook-types.md#python-runbooks) en Azure Automation. Runbooks de Python compilados en Python 2. Puede modificar directamente el código del runbook con el editor de texto en Azure Portal.

> [!div class="checklist"]
> * Creación de un runbook de Python sencillo
> * Prueba y publicación del runbook
> * Ejecución y seguimiento del estado del trabajo de runbook
> * Actualización del runbook para iniciar una máquina virtual de Azure con parámetros de runbook

> [!NOTE]
> No se admite el uso de un webhook para iniciar un runbook de Python.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesitará lo siguiente:

- Suscripción de Azure. Si aún no tiene ninguna, puede [activar las ventajas de la suscripción a MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o suscribirse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Cuenta de Automation](../index.yml) para contener el Runbook y autenticarse en recursos de Azure. Esta cuenta debe tener permiso para iniciar y detener la máquina virtual.
- Una máquina virtual de Azure. Puede detener e iniciar esta máquina, por lo que no debería ser una máquina virtual de producción.

## <a name="create-a-new-runbook"></a>Crear un runbook

Empieza creando un runbook simple cuya salida sea el texto *Hola mundo*.

1. En Azure Portal, abra su cuenta de Automation.

    La página de la cuenta de Automation proporciona una vista rápida de los recursos que hay en esa cuenta. Ya debería tener algunos recursos. Muchas de esos recursos son los módulos que se incluyen automáticamente en una cuenta nueva de Automation. También debe tener el recurso de credencial que se menciona en los [requisitos previos](#prerequisites).

2. En **Automatización de procesos**, haga clic en **Runbooks** para abrir la lista de runbooks.

3. Seleccione **Agregar un Runbook** para crear un runbook.

4. Asigne al runbook el nombre **MyFirstRunbook-Python**.

5. Seleccione **Python 2** para **Tipo de Runbook**.

6. Haga clic en **Crear** para crear el runbook y abra el editor de texto.

## <a name="add-code-to-the-runbook"></a>Agregar código al runbook

Ahora, agregue un comando simple para imprimir el texto `Hello World`.

```python
print("Hello World!")
```

Haga clic en **Guardar** para guardar el runbook.

## <a name="test-the-runbook"></a>Probar el runbook

Antes de publicar el runbook para que esté disponible en producción, puede que quiera probarlo para asegurarse de que funciona correctamente. Cuando se prueba un runbook, se ejecuta su versión Borrador y se visualiza su salida de forma interactiva.

1. Haga clic en **Panel Prueba** para abrir el panel de prueba.

2. Haga clic en **Iniciar** para iniciar la prueba. Esta debe ser la única opción habilitada.

3. Se crea un [trabajo de runbook](../automation-runbook-execution.md) y se muestra su estado.
   El estado del trabajo se inicia como En cola, lo que indica que está esperando que haya disponible un trabajo de runbook en la nube. Su estado cambia a Iniciando cuando un trabajo de runbook solicita el trabajo. Cuando el runbook comienza a ejecutarse realmente, el estado es En ejecución.

4. Cuando se complete el trabajo del runbook, se mostrará su resultado. En este caso, debería ver `Hello World`.

5. Cierre el panel Prueba para volver al lienzo.

## <a name="publish-and-start-the-runbook"></a>Publicar e iniciar el runbook

El runbook que acaba de crear aún está en modo de borrador. Tiene que publicarlo antes de que pueda ejecutarlo en producción. Al publicar un runbook, se sobrescribe la versión publicada existente con la versión de borrador. En este caso, no tiene una versión publicada aún porque acaba de crear el runbook.

1. Haga clic en **Publicar** para publicar el runbook y en **Sí** cuando se le solicite.

2. Si se desplaza a la izquierda para ver el runbook en la página Runbooks, observará que indica **Publicado** en **Estado de creación**.

3. Desplácese de nuevo a la derecha para ver el panel de **MyFirstRunbook-Python**.

   Las opciones en la parte superior le permiten iniciar el runbook, verlo o programarlo para que se inicie en algún momento en el futuro.

4. Haga clic en **Iniciar** y, después, en **Aceptar** cuando se abra la hoja Iniciar Runbook.

5. Se abre un panel Trabajo para el trabajo de runbook que acaba de crear. Puede cerrar este panel, pero déjelo abierto por ahora para que pueda ver el progreso del trabajo.

6. El estado del trabajo se muestra en **Resumen del trabajo** y coincide con los estados que vio cuando probó el runbook.

7. Cuando el estado del runbook aparezca como Completado, haga clic en **Salida**. Se abre la página Salida, donde puede ver `Hello World`.

8. Cierre el panel Salida.

9. Haga clic en **Todos los registros** para abrir el panel Transmisiones para el trabajo de Runbook. Solo debería ver `Hello World` en el flujo Salida. No obstante, este panel puede mostrar otras transmisiones para un trabajo de runbook, como Error y Modo detallado, si el runbook escribe en ellas.

10. Cierre el panel Transmisiones y el panel Trabajos para volver al panel de MyFirstRunbook-Python.

11. Haga clic en **Trabajos** para abrir el panel Trabajos de este runbook. Esta página enumera todos los trabajos creados por este runbook. Solo debería ver un trabajo en la lista ya que solo ejecutó el trabajo una vez.

12. Puede hacer clic en este trabajo para abrir el mismo panel Trabajo que vio cuando se inició el runbook. Esta panel permite volver atrás en el tiempo y ver los detalles de cualquier trabajo que se creó para un runbook determinado.

## <a name="add-authentication-to-manage-azure-resources"></a>Agregar autenticación para administrar recursos de Azure

Ha probado y publicado su runbook, pero hasta ahora no hace nada útil. Quiere que administre recursos de Azure.
Para ello, el script debe autenticarse mediante las credenciales de su cuenta de Automation. Para ayudarle, puede usar el [paquete de la utilidad de Azure Automation](https://github.com/azureautomation/azure_automation_utility) para facilitar la autenticación e interacción con recursos de Azure.

> [!NOTE]
> La cuenta de Automation se debe haber creado con la característica de entidad de servicio para que haya un certificado de ejecución.
> Si la cuenta de Automation no se ha creado con la entidad de servicio, se puede autenticar tal como se describe en [Autenticación con las bibliotecas de administración de Azure para Python](/azure/python/python-sdk-azure-authenticate).

1. Abra el editor de texto haciendo clic en **Editar** en el panel MyFirstRunbook-Python.

2. Agregue el siguiente código para autenticarse en Azure:

   ```python
   import os
   from azure.mgmt.compute import ComputeManagementClient
   import azure.mgmt.resource
   import automationassets

   def get_automation_runas_credential(runas_connection):
       from OpenSSL import crypto
       import binascii
       from msrestazure import azure_active_directory
       import adal

       # Get the Azure Automation RunAs service principal certificate
       cert = automationassets.get_automation_certificate("AzureRunAsCertificate")
       pks12_cert = crypto.load_pkcs12(cert)
       pem_pkey = crypto.dump_privatekey(crypto.FILETYPE_PEM,pks12_cert.get_privatekey())

       # Get run as connection information for the Azure Automation service principal
       application_id = runas_connection["ApplicationId"]
       thumbprint = runas_connection["CertificateThumbprint"]
       tenant_id = runas_connection["TenantId"]

       # Authenticate with service principal certificate
       resource ="https://management.core.windows.net/"
       authority_url = ("https://login.microsoftonline.com/"+tenant_id)
       context = adal.AuthenticationContext(authority_url)
       return azure_active_directory.AdalAuthentication(
       lambda: context.acquire_token_with_client_certificate(
               resource,
               application_id,
               pem_pkey,
               thumbprint)
       )

   # Authenticate to Azure using the Azure Automation RunAs service principal
   runas_connection = automationassets.get_automation_connection("AzureRunAsConnection")
   azure_credential = get_automation_runas_credential(runas_connection)
   ```

## <a name="add-code-to-create-python-compute-client-and-start-the-vm"></a>Agregar código para crear un cliente de Compute de Python e iniciar la máquina virtual

Para trabajar con máquinas virtuales de Azure, cree una instancia del [cliente de Azure Compute para Python](/python/api/azure-mgmt-compute/azure.mgmt.compute.computemanagementclient).

Use el cliente de Compute para iniciar la máquina virtual. Agregue el siguiente código al runbook:

```python
# Initialize the compute management client with the RunAs credential and specify the subscription to work against.
compute_client = ComputeManagementClient(
    azure_credential,
    str(runas_connection["SubscriptionId"])
)


print('\nStart VM')
async_vm_start = compute_client.virtual_machines.start(
    "MyResourceGroup", "TestVM")
async_vm_start.wait()
```

Donde `MyResourceGroup` es el nombre del grupo de recursos que contiene la máquina virtual y `TestVM` es el nombre de la máquina virtual que quiere iniciar.

Pruebe y ejecute el runbook de nuevo para constatar que inicia la máquina virtual.

## <a name="use-input-parameters"></a>Usar parámetros de entrada

Actualmente, el runbook usa valores codificados de forma rígida en los nombres del grupo de recursos y de la máquina virtual. Ahora vamos a agregar código para obtener estos valores de parámetros de entrada.

Use la variable `sys.argv` para obtener los valores de parámetro. Agregue el siguiente código al runbook, inmediatamente después de las demás instrucciones `import`:

```python
import sys

resource_group_name = str(sys.argv[1])
vm_name = str(sys.argv[2])
```

Con esta operación se importa el módulo `sys` y se crean dos variables para contener los nombres del grupo de recursos y de la máquina virtual. Observe que el elemento de la lista de argumentos, `sys.argv[0]`, es el nombre del script y no lo especifica el usuario.

Ahora, puede modificar las dos últimas líneas del runbook para usar los valores de parámetro de entrada en lugar de los valores codificados de forma rígida:

```python
async_vm_start = compute_client.virtual_machines.start(
    resource_group_name, vm_name)
async_vm_start.wait()
```

Cuando se inicia un runbook de Python (ya sea en la página Prueba o como un runbook publicado), se pueden especificar los valores de los parámetros en la página Iniciar Runbook, en **Parámetros**.

Cuando empiece a escribir un valor en el primer cuadro, aparecerá un segundo cuadro (y así sucesivamente) para que pueda especificar varios valores de parámetro si es necesario.

Los valores estarán disponibles para el script en la matriz `sys.argv` como en el código que se acaba de agregar.

Escriba el nombre del grupo de recursos como valor del primer parámetro y el nombre de la máquina virtual que se va a iniciar como valor del segundo parámetro.

![Escribir valores de parámetro](../media/automation-tutorial-runbook-textual-python/runbook-python-params.png)

Haga clic en **Aceptar** para iniciar el runbook. El runbook se ejecuta e inicia la máquina virtual que se ha especificado.

## <a name="error-handling-in-python"></a>Control de errores en Python

También puede usar las siguientes convenciones para recuperar varios flujos de los runbooks de Python, como los flujos WARNING, ERROR y DEBUG.

```python
print("Hello World output")
print("ERROR: - Hello world error")
print("WARNING: - Hello world warning")
print("DEBUG: - Hello world debug")
print("VERBOSE: - Hello world verbose")
```

En el ejemplo siguiente se muestra esta convención utilizada en un bloque `try...except`.

```python
try:
    raise Exception('one', 'two')
except Exception as detail:
    print 'ERROR: Handling run-time error:', detail
```

> [!NOTE]
> Azure Automation no admite `sys.stderr`.

## <a name="next-steps"></a>Pasos siguientes

- Para empezar a trabajar con Runbooks de PowerShell, consulte [Creación de un runbook de PowerShell](automation-tutorial-runbook-textual-powershell.md).
- Para empezar a trabajar con runbooks gráficos, consulte [Creación de un runbook gráfico](automation-tutorial-runbook-graphical.md).
- Para empezar a trabajar con runbooks de flujo de trabajo de PowerShell, consulte [Creación de un runbook de flujo de trabajo de PowerShell](automation-tutorial-runbook-textual.md).
- Para más información sobre los tipos de runbook, sus ventajas y sus limitaciones, consulte [Tipos de runbooks de Azure Automation](../automation-runbook-types.md).
- Para más información sobre cómo desarrollar para Azure con Python, vea [Azure para desarrolladores de Python](/azure/python/).
- Para ver runbooks de ejemplo 2 de Python, consulte el [GitHub de Azure Automation](https://github.com/azureautomation/runbooks/tree/master/Utility/Python).
