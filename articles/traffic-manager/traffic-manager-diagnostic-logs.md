---
title: Habilitación del registro de recursos en Azure Traffic Manager
description: Aprenda a habilitar el registro de recursos para su perfil de Traffic Manager y acceda a los archivos de registro que se crean como resultado.
services: traffic-manager
author: rohinkoul
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/25/2019
ms.author: rohink
ms.openlocfilehash: 09c04d727e42f88bb376a59d66a23688e16abb13
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089065"
---
# <a name="enable-resource-logging-in-azure-traffic-manager"></a>Habilitación del registro de recursos en Azure Traffic Manager

En este artículo se describe cómo habilitar la recopilación de registros de recursos y acceder a los datos de registro de un perfil de Traffic Manager.

Los registros de recursos de Azure Traffic Manager pueden proporcionar conclusiones sobre el comportamiento del recurso de perfil de Traffic Manager. Por ejemplo, puede usar los datos de registro del perfil para determinar porqué determinados sondeos han agotado el tiempo de espera en un punto de conexión.

## <a name="enable-resource-logging"></a>Habilitación de registro de recurso

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Puede ejecutar los comandos siguientes en [Azure Cloud Shell](https://shell.azure.com/powershell), o mediante la ejecución de PowerShell en el equipo. Azure Cloud Shell es un shell interactivo gratuito. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Si ejecuta PowerShell desde el equipo, necesita el módulo Azure PowerShell, versión 1.0.0 o posterior. Puede ejecutar `Get-Module -ListAvailable Az` para encontrar la versión instalada. Si necesita instalarla o actualizarla, consulte el artículo sobre [cómo instalar el módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si ejecuta PowerShell localmente, también debe ejecutar `Login-AzAccount` para iniciar sesión en Azure.

1. **Recupere el perfil de Traffic Manager:**

    Para habilitar el registro de recursos, necesita el identificador de un perfil de Traffic Manager. Recupere el perfil de Traffic Manager para el que quiere habilitar el registro de recursos con [Get-AzTrafficManagerProfile](/powershell/module/az.TrafficManager/Get-azTrafficManagerProfile). La salida incluye la información del identificador del perfil de Traffic Manager.

    ```azurepowershell-interactive
    Get-AzTrafficManagerProfile -Name <TrafficManagerprofilename> -ResourceGroupName <resourcegroupname>
    ```

2. **Habilitación del registro de recursos para el perfil de Traffic Manager:**

    Habilite el registro de recursos para el perfil de Traffic Manager con el identificador obtenido en el paso anterior con [Set-AzDiagnosticSetting](https://docs.microsoft.com/powershell/module/az.monitor/set-azdiagnosticsetting?view=latest). El comando siguiente almacena los registros detallados del perfil de Traffic Manager en una cuenta de Azure Storage especificada. 

      ```azurepowershell-interactive
    Set-AzDiagnosticSetting -ResourceId <TrafficManagerprofileResourceId> -StorageAccountId <storageAccountId> -Enabled $true
      ``` 
3. **Compruebe la configuración de diagnóstico:**

      Compruebe la configuración de diagnóstico del perfil de Traffic Manager mediante [Get-AzDiagnosticSetting](https://docs.microsoft.com/powershell/module/az.monitor/get-azdiagnosticsetting?view=latest). El siguiente comando muestra las categorías que se registran para un recurso.

     ```azurepowershell-interactive
     Get-AzDiagnosticSetting -ResourceId <TrafficManagerprofileResourceId>
     ```  
      Asegúrese de que todas las categorías de registro asociadas con el recurso del perfil de Traffic Manager se muestran como habilitadas. También, compruebe que la cuenta de almacenamiento está configurada correctamente.

## <a name="access-log-files"></a>Acceso a los archivos de registro
1. Inicie sesión en [Azure Portal](https://portal.azure.com). 
1. Vaya a la cuenta de Azure Storage en el portal.
2. En la página **Información general** de su cuenta de Azure Storage, en **Servicios**, seleccione **Blobs**.
3. En **Contenedores**, seleccione **insights-logs-probehealthstatusevents** y vaya al archivo PT1H.json y haga clic en **Descargar** para descargar y guardar una copia de este archivo de registro.

    ![Acceso a los archivos de registro del perfil de Traffic Manager desde un almacenamiento de blobs](./media/traffic-manager-logs/traffic-manager-logs.png)


## <a name="traffic-manager-log-schema"></a>Esquema de registro de Traffic Manager

Todos los registros de recursos disponibles a través de Azure Monitor comparten un esquema común de nivel superior, con flexibilidad para que cada servicio emita propiedades únicas para sus propios eventos. Para ver el esquema de registros de recursos de nivel superior, consulte [Esquemas, categorías y servicios admitidos para registros de recursos de Azure](../azure-monitor/platform/tutorial-dashboards.md).

En la tabla siguiente se incluye el esquema de registros específico del recurso del perfil de Azure Traffic Manager.

|Nombre del campo|Tipo de campo|Definición|Ejemplo|
|----|----|---|---|
|EndpointName|String|El nombre del punto de conexión de Traffic Manager cuyo mantenimiento se está registrando.|*myPrimaryEndpoint*|
|Status|String|El estado de mantenimiento del punto de conexión de Traffic Manager que se sondeó. El estado puede ser **Up** (Arriba) o **Down** (Abajo).|**Up** (Arriba)|
|||||

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre la [supervisión de Traffic Manager](traffic-manager-monitoring.md)

