---
title: Errores de cuota
description: Describe cómo resolver errores de cuota de recursos al implementar recursos con Azure Resource Manager.
ms.topic: troubleshooting
ms.date: 03/09/2018
ms.openlocfilehash: 75e8abf31d035a1e3a106bc0c6561624762db5d5
ms.sourcegitcommit: 03662d76a816e98cfc85462cbe9705f6890ed638
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90530427"
---
# <a name="resolve-errors-for-resource-quotas"></a>Resolución de errores de cuotas de recursos

En este artículo se describen errores de cuota que pueden producirse al implementar recursos.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="symptom"></a>Síntoma

Si implementa una plantilla que crea recursos que exceden las cuotas de Azure, obtendrá un error de implementación similar al siguiente:

```
Code=OperationNotAllowed
Message=Operation results in exceeding quota limits of Core.
Maximum allowed: 4, Current in use: 4, Additional requested: 2.
```

O bien, es posible que vea:

```
Code=ResourceQuotaExceeded
Message=Creating the resource of type <resource-type> would exceed the quota of <number>
resources of type <resource-type> per resource group. The current resource count is <number>,
please delete some resources of this type before creating a new one.
```

## <a name="cause"></a>Causa

Las cuotas se aplican por grupo de recursos, suscripciones, cuentas y otros ámbitos. Por ejemplo, la suscripción puede configurarse para limitar el número de núcleos para una región. Si trata de implementar una máquina virtual con más núcleos que los permitidos, recibirá un error que indica que se ha superado la cuota.
Para obtener información completa de las cuotas, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md).

## <a name="troubleshooting"></a>Solución de problemas

### <a name="azure-cli"></a>Azure CLI

Para la CLI de Azure, use el comando `az vm list-usage` para encontrar las cuotas de máquina virtual.

```azurecli
az vm list-usage --location "South Central US"
```

Que devuelve:

```output
[
  {
    "currentValue": 0,
    "limit": 2000,
    "name": {
      "localizedValue": "Availability Sets",
      "value": "availabilitySets"
    }
  },
  ...
]
```

### <a name="powershell"></a>PowerShell

Para PowerShell, use el comando **Get-AzVMUsage** para encontrar las cuotas de máquina virtual.

```powershell
Get-AzVMUsage -Location "South Central US"
```

Que devuelve:

```output
Name                             Current Value Limit  Unit
----                             ------------- -----  ----
Availability Sets                            0  2000 Count
Total Regional Cores                         0   100 Count
Virtual Machines                             0 10000 Count
```

## <a name="solution"></a>Solución

Para solicitar un aumento de cuota, vaya al portal y abra una incidencia de soporte técnico. En la incidencia de soporte técnico, solicite un aumento de la cuota para la región en la que desea realizar la implementación.

> [!NOTE]
> Recuerde que para los grupos de recursos, la cuota para cada región individual, no para toda la suscripción. Si necesita implementar 30 núcleos en el Oeste de EE. UU., debe pedir 30 núcleos de administrador de recursos en el Oeste de EE. UU. Si necesita implementar 30 núcleos en cualquiera de las regiones para las que tiene acceso, debe pedir 30 núcleos de Resource Manager en todas las regiones.
>
>

1. Seleccione **Suscripciones**.

   ![Captura de pantalla que muestra el menú de Azure Portal con la opción Suscripciones seleccionada.](./media/error-resource-quota/subscriptions.png)

2. Seleccione la suscripción que necesita una cuota mayor.

   ![Seleccionar suscripción](./media/error-resource-quota/select-subscription.png)

3. Seleccione **Uso y cuotas**.

   ![Selección de uso y cuotas](./media/error-resource-quota/select-usage-quotas.png)

4. En la esquina superior derecha, seleccione **Solicitar aumento**.

   ![Solicitud de aumento](./media/error-resource-quota/request-increase.png)

5. Rellene los formularios para el tipo de cuota que necesita aumentar.

   ![Rellenado del formulario](./media/error-resource-quota/forms.png)