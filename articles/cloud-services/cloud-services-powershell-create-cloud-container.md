---
title: Creación de un contenedor de servicios en la nube con PowerShell | Microsoft Docs
description: En este artículo se explica cómo crear un contenedor de servicios en la nube con PowerShell. El contenedor hospeda roles web y roles de trabajo.
services: cloud-services
documentationcenter: .net
author: cawaMS
ms.service: cloud-services
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: powershell
ms.workload: na
ms.date: 11/18/2016
ms.author: cawa
ms.openlocfilehash: d40a5b64cc8018f45bf08158ce808b2baae27962
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87049094"
---
# <a name="use-an-azure-powershell-command-to-create-an-empty-cloud-service-container"></a>Uso de un comando de Azure PowerShell para crear un contenedor vacío de servicios en la nube

En este artículo se explica cómo crear rápidamente un contenedor de Cloud Services mediante cmdlets de Azure PowerShell. Siga estos pasos:

1. Instale el cmdlet de Microsoft Azure PowerShell desde la página de [descargas de Azure PowerShell](https://aka.ms/webpi-azps) .
2. Abra un símbolo del sistema de PowerShell.
3. Use [Add-AzureAccount](/powershell/module/servicemanagement/azure.service/add-azureaccount?view=azuresmps-4.0.0) para iniciar sesión.

   > [!NOTE]
   > Para más instrucciones acerca de cómo instalar el cmdlet de Azure PowerShell y conectarse a la suscripción de Azure, consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/).
   >
   >
4. Use el cmdlet **New-AzureService** para crear un contenedor de servicios en la nube de Azure vacío.

   ```
   New-AzureService [-ServiceName] <String> [-AffinityGroup] <String> [[-Label] <String>] [[-Description] <String>] [[-ReverseDnsFqdn] <String>] [<CommonParameters>]
   New-AzureService [-ServiceName] <String> [-Location] <String> [[-Label] <String>] [[-Description] <String>] [[-ReverseDnsFqdn] <String>] [<CommonParameters>]
   ```

5. Para invocar el cmdlet, siga este ejemplo:

   ```powershell
   New-AzureService -ServiceName "mytestcloudservice" -Location "Central US" -Label "mytestcloudservice"
   ```

Para obtener más información sobre cómo crear el servicio en la nube de Azure, ejecute:

```powershell
Get-help New-AzureService
```

### <a name="next-steps"></a>Pasos siguientes

* Para administrar la implementación de servicios en la nube, consulte los comandos [Get-AzureService](/powershell/module/servicemanagement/azure.service/Get-AzureService?view=azuresmps-4.0.0), [Remove-AzureService](/powershell/module/servicemanagement/azure.service/Remove-AzureService?view=azuresmps-4.0.0) y [Set-AzureService](/powershell/module/servicemanagement/azure.service/set-azureservice?view=azuresmps-4.0.0). Para más información, también puede consultar [Configuración de servicios en la nube](cloud-services-how-to-configure-portal.md) .
* Para publicar el proyecto de servicio en la nube en Azure, consulte el código de ejemplo de **PublishCloudService.ps1** del [repositorio de servicios en la nube archivado](https://github.com/MicrosoftDocs/azure-cloud-services-files/tree/master/Scripts/cloud-services-continuous-delivery).