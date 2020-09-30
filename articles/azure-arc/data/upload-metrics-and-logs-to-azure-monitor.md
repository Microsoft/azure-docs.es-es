---
title: Carga de inventario de recursos, datos de uso, métricas y registros en Azure Monitor
description: Carga de inventario de recursos, datos de uso, métricas y registros en Azure Monitor
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: ac6ffd2b5bf48079db6a0cd261dbe2535e1821ac
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932226"
---
# <a name="upload-resource-inventory-usage-data-metrics-and-logs-to-azure-monitor"></a>Carga de inventario de recursos, datos de uso, métricas y registros en Azure Monitor

Con los servicios de datos de Azure Arc, *opcionalmente* puede cargar métricas y registros en Azure Monitor a fin de agregar y analizar métricas, registros, generar alertas, enviar notificaciones o desencadenar acciones automatizadas. El envío de datos a Azure Monitor también permite almacenar datos de supervisión y registro fuera del sitio y a gran escala, lo que permite el almacenamiento a largo plazo de los datos para realizar análisis avanzados.  Si tiene varios sitios con servicios de datos de Azure Arc, puede usar Azure Monitor como ubicación central para recopilar todos los registros y las métricas en todos los sitios.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="before-you-begin"></a>Antes de empezar

Hay algunos pasos de instalación única que son necesarios para habilitar los escenarios de carga de registros y métricas:

1) Cree una aplicación de entidad de servicio o Azure Active Directory, lo que incluye crear un secreto de acceso de cliente y asignar la entidad de servicio al rol "Publicador de métricas de supervisión" en las suscripciones donde se encuentran los recursos de la instancia de base de datos.
2) Cree un área de trabajo de Log Analytics, obtenga las claves y establezca la información en variables de entorno.

El primer elemento es necesario para cargar las métricas y el segundo para cargar los registros.

Siga estos comandos para crear la entidad de servicio de carga de métricas y asignarla a los roles "Publicador de métricas de supervisión" y "Colaborador" para que la entidad de servicio pueda cargar métricas y realizar operaciones de creación y carga.

## <a name="create-service-principal-and-assign-roles"></a>Creación de una entidad de servicio y asignación de roles

Siga estos comandos para crear la entidad de servicio de carga de métricas y asignarla al rol "Publicador de métricas de supervisión":

Para crear una entidad de servicio, ejecute este comando:

> [!NOTE]
> Para crear una entidad de servicio se necesitan [determinados permisos en Azure](/active-directory/develop/howto-create-service-principal-portal#required-permissions).

```console
az ad sp create-for-rbac --name <a name you choose>

#Example:
#az ad sp create-for-rbac --name azure-arc-metrics
```

Salida de ejemplo:

```console
"appId": "2e72adbf-de57-4c25-b90d-2f73f126e123",
"displayName": "azure-arc-metrics",
"name": "http://azure-arc-metrics",
"password": "5039d676-23f9-416c-9534-3bd6afc78123",
"tenant": "72f988bf-85f1-41af-91ab-2d7cd01ad1234"
```

Guarde los valores de appId y tenant en una variable de entorno para su uso posterior:

```console
#PowerShell

$Env:SPN_CLIENT_ID='<the 'appId' value from the output of the 'az ad sp create-for-rbac' command above>'
$Env:SPN_CLIENT_SECRET='<the 'password' value from the output of the 'az ad sp create-for-rbac' command above>'
$Env:SPN_TENANT_ID='<the 'tenant' value from the output of the 'az ad sp create-for-rbac' command above>'

#Linux/macOS

export SPN_CLIENT_ID='<the 'appId' value from the output of the 'az ad sp create-for-rbac' command above>'
export SPN_CLIENT_SECRET='<the 'password' value from the output of the 'az ad sp create-for-rbac' command above>'
export SPN_TENANT_ID='<the 'tenant' value from the output of the 'az ad sp create-for-rbac' command above>'

#Example (using Linux):
export SPN_CLIENT_ID='2e72adbf-de57-4c25-b90d-2f73f126e123'
export SPN_CLIENT_SECRET='5039d676-23f9-416c-9534-3bd6afc78123'
export SPN_TENANT_ID='72f988bf-85f1-41af-91ab-2d7cd01ad1234'
```

Ejecute este comando para asignar la entidad de servicio al rol "Editor de métricas de supervisión" en la suscripción donde se encuentran los recursos de la instancia de base de datos:

```console
az role assignment create --assignee <appId value from output above> --role 'Monitoring Metrics Publisher' --scope subscriptions/<sub ID>
az role assignment create --assignee <appId value from output above> --role 'Contributor' --scope subscriptions/<sub ID>

#Example:
#az role assignment create --assignee 2e72adbf-de57-4c25-b90d-2f73f126ede5 --role 'Monitoring Metrics Publisher' --scope subscriptions/182c901a-129a-4f5d-56e4-cc6b29459123
#az role assignment create --assignee 2e72adbf-de57-4c25-b90d-2f73f126ede5 --role 'Contributor' --scope subscriptions/182c901a-129a-4f5d-56e4-cc6b29459123
```

Salida de ejemplo:

```console
{
  "canDelegate": null,
  "id": "/subscriptions/182c901a-129a-4f5d-86e4-cc6b29459123/providers/Microsoft.Authorization/roleAssignments/f82b7dc6-17bd-4e78-93a1-3fb733b912d",
  "name": "f82b7dc6-17bd-4e78-93a1-3fb733b9d123",
  "principalId": "5901025f-0353-4e33-aeb1-d814dbc5d123",
  "principalType": "ServicePrincipal",
  "roleDefinitionId": "/subscriptions/182c901a-129a-4f5d-86e4-cc6b29459123/providers/Microsoft.Authorization/roleDefinitions/3913510d-42f4-4e42-8a64-420c39005123",
  "scope": "/subscriptions/182c901a-129a-4f5d-86e4-cc6b29459123",
  "type": "Microsoft.Authorization/roleAssignments"
}
```

## <a name="create-a-log-analytics-workspace"></a>Creación de un área de trabajo de Log Analytics

A continuación, ejecute estos comandos para crear un área de trabajo de Log Analytics y establecer la información de acceso en variables de entorno.

> [!NOTE]
> Si ya tiene un área de trabajo, omita este paso.

```console
az monitor log-analytics workspace create --resource-group <resource group name> --name <some name you choose>

#Example:
#az monitor log-analytics workspace create --resource-group MyResourceGroup --name MyLogsWorkpace
```

Salida de ejemplo:

```console
{
  "customerId": "d6abb435-2626-4df1-b887-445fe44a4123",
  "eTag": null,
  "id": "/subscriptions/182c901a-129a-4f5d-86e4-cc6b29459123/resourcegroups/user-arc-demo/providers/microsoft.operationalinsights/workspaces/user-logworkspace",
  "location": "eastus",
  "name": "user-logworkspace",
  "portalUrl": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "user-arc-demo",
  "retentionInDays": 30,
  "sku": {
    "lastSkuUpdate": "Thu, 30 Jul 2020 22:37:53 GMT",
    "maxCapacityReservationLevel": 3000,
    "name": "pergb2018"
  },
  "source": "Azure",
  "tags": null,
  "type": "Microsoft.OperationalInsights/workspaces"
}
```

## <a name="assign-id-and-shared-key-to-environment-variables"></a>Asignación del identificador y la clave compartida a variables de entorno

Guarde el valor customerId (el id. de área de trabajo) como una variable de entorno para usarla más adelante:

```console
#PowerShell
$Env:WORKSPACE_ID='<the customerId from the 'log-analytics workspace create' command output above>'

#Linux/macOS
export WORKSPACE_ID='<the customerId from the 'log-analytics workspace create' command output above>'

#Example (using Linux)
#export WORKSPACE_ID='d6abb435-2626-4df1-b887-445fe44a4123'
```

Este comando imprimirá las claves de acceso necesarias para conectarse al área de trabajo de Log Analytics:

```console
az monitor log-analytics workspace get-shared-keys --resource-group MyResourceGroup --name MyLogsWorkpace
```

Salida de ejemplo:

```console
{
  "primarySharedKey": "JXzQp1RcGgjXFCDS3v0sXoxPvbgCoGaIv35lf11Km2WbdGFvLXqaydpaj1ByWGvKoCghL8hL4BRoypXxkLr123==",
  "secondarySharedKey": "p2XHSxLJ4o9IAqm2zINcEmx0UWU5Z5EZz8PQC0OHpFjdpuVaI0zsPbTv5VyPFgaCUlCZb2yEbkiR4eTuTSF123=="
}
```

Guarde la clave principal en una variable de entorno para usarla más adelante:

```console
#PowerShell:
$Env:WORKSPACE_SHARED_KEY='<the primarySharedKey value from the 'get-shared-keys' command above'

#Linux/macOS:
export WORKSPACE_SHARED_KEY='<the primarySharedKey value from the 'get-shared-keys' command above'

#Example (using Linux):
export WORKSPACE_SHARED_KEY='JXzQp1RcGgjXFCDS3v0sXoxPvbgCoGaIv35lf11Km2WbdGFvLXqaydpaj1ByWGvKoCghL8hL4BRoypXxkLr123=='

```

## <a name="set-final-environment-variables-and-confirm"></a>Establecimiento de las variables de entorno finales y confirmación

Establezca la dirección URL de la autoridad de SPN en una variable de entorno:

```console
#PowerShell
$Env:SPN_AUTHORITY='https://login.microsoftonline.com'

#Linux/macOS:
export SPN_AUTHORITY='https://login.microsoftonline.com'
```

Si quiere, asegúrese de que se hayan establecido todas las variables de entorno necesarias:

```console
#PowerShell
$Env:WORKSPACE_ID
$Env:WORKSPACE_SHARED_KEY
$Env:SPN_TENANT_ID
$Env:SPN_CLIENT_ID
$Env:SPN_CLIENT_SECRET
$Env:SPN_AUTHORITY

#Linux/macOS
echo $WORKSPACE_ID
echo $WORKSPACE_SHARED_KEY
echo $SPN_TENANT_ID
echo $SPN_CLIENT_ID
echo $SPN_CLIENT_SECRET
echo $SPN_AUTHORITY
```

## <a name="upload-metrics-to-azure-monitor"></a>Carga de métricas a Azure Monitor

Para cargar las métricas de las instancias administradas de Azure SQL y los grupos de servidores Hiperescala de Azure Database for PostgreSQL, ejecute los siguientes comandos de la CLI:

Todas las métricas se exportarán al archivo especificado:

```console
azdata arc dc export -t metrics --path metrics.json
```

Esto cargará métricas a Azure Monitor:

```console
azdata arc dc upload --path metrics.json
```

## <a name="view-the-metrics-in-the-portal"></a>Visualización de las métricas en el portal

Una vez que se carguen las métricas, podrá visualizarlas desde Azure Portal.

Para ver las métricas en el portal, use este vínculo especial para abrirlo: <https://portal.azure.com> Después, busque la instancia de base de datos por nombre en la barra de búsqueda:

Puede ver el uso de la CPU en la página Información general o bien, si quiere métricas más detalladas, puede hacer clic en Métricas en el panel de navegación de la izquierda.

Elija SQL Server como el espacio de nombres de la métrica:

Seleccione la métrica que quiera visualizar (también puede seleccionar varias):

Cambie la frecuencia a los últimos 30 minutos:

> [!NOTE]
> Solo puede cargar las métricas de los últimos 30 minutos. Azure Monitor rechaza las métricas de más de 30 minutos.

## <a name="upload-logs-to-azure-monitor"></a>Carga de registros a Azure Monitor

 Para cargar los registros de las instancias administradas de Azure SQL y los grupos de servidores Hiperescala de Azure Database for PostgreSQL, ejecute los siguientes comandos de la CLI:

Esto exportará todos los archivos al archivo especificado:

```console
azdata arc dc export -t logs --path logs.json
```

Esto cargará los registros a un área de trabajo de Log Analytics en Azure Monitor:

```console
azdata arc dc upload --path logs.json
```

## <a name="view-your-logs-in-azure-portal"></a>Visualización de los registros en Azure Portal

Una vez que se hayan cargado los registros, debería poder consultarlos mediante el explorador de consultas de registros como se indica a continuación:

1. Abra Azure Portal y, después, busque el área de trabajo por su nombre en la barra de búsqueda de la parte superior y selecciónela.
2. Haga clic en Registros en el panel de la izquierda.
3. Haga clic en Comenzar (o bien en los vínculos de la página Comenzar para obtener más información sobre Log Analytics si no está familiarizado con él)
4. Siga el tutorial para obtener más información sobre Log Analytics si es la primera vez
5. Expanda Registros personalizados en la parte inferior de la lista de tablas y verá una tabla denominada "sql_instance_logs_CL".
6. Haga clic en el icono de ojo situado junto al nombre de la tabla
7. Haga clic en el botón "Ver en el editor de consultas"
8. Ahora tendrá una consulta en el editor de consultas que mostrará los 10 eventos más recientes del registro.
9. Desde aquí, puede experimentar con la consulta de los registros mediante el editor de consultas, establecer alertas, etc.

## <a name="automating-metrics-and-logs-uploads-optional"></a>Automatización de las cargas de métricas y registros (opcional)

Si quiere cargar métricas y registros de forma constante, puede crear un script y ejecutarlo en un temporizador cada pocos minutos.  A continuación se muestra un ejemplo de cómo automatizar las cargas mediante un script de shell de Linux.

En el editor de código o texto que prefiera, agregue lo siguiente al contenido del script en el archivo y guárdelo como un archivo ejecutable de script como .sh (Linux o Mac), o bien .cmd, .bat, .ps1.

```console
azdata arc dc export --type metrics --path metrics.json --force
azdata arc dc upload --path metrics.json
```

Conversión del archivo de script en ejecutable

```console
chmod +x myuploadscript.sh
```

Ejecute el script cada dos minutos:

```console
watch -n 120 ./myuploadscript.sh
```

También puede usar un programador de trabajos como cron o el Programador de tareas de Windows, o bien un orquestador como Ansible, Puppet o Chef.
