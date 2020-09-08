---
title: Creación o modificación de un emparejamiento directo mediante PowerShell
titleSuffix: Azure
description: Creación o modificación de un emparejamiento directo mediante PowerShell
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: how-to
ms.date: 11/27/2019
ms.author: prmitiki
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: be4bab10d369fcefe626bbe890acfff7fc7004b0
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89079071"
---
# <a name="create-or-modify-a-direct-peering-by-using-powershell"></a>Creación o modificación de un emparejamiento directo mediante PowerShell

En este artículo se describe cómo crear un emparejamiento directo de Microsoft mediante cmdlets de PowerShell y el modelo de implementación de Azure Resource Manager. También se muestra cómo comprobar el estado del recurso, así como la forma de actualizarlo o eliminarlo y desaprovisionarlo.

Si lo prefiere, puede realizar los pasos de esta guía mediante [Azure Portal](howto-direct-portal.md).

## <a name="before-you-begin"></a>Antes de empezar
* Antes de comenzar la configuración, revise los [requisitos previos](prerequisites.md) y el [tutorial del emparejamiento directo](walkthrough-direct-all.md).
* Si ya dispone de conexiones de emparejamiento directo con Microsoft que no convierten en recursos de Azure, consulte [Conversión de un emparejamiento directo heredado en un recurso de Azure mediante PowerShell](howto-legacy-direct-powershell.md).

### <a name="work-with-azure-powershell"></a>Funcionamiento con Azure PowerShell
[!INCLUDE [CloudShell](./includes/cloudshell-powershell-about.md)]

## <a name="create-and-provision-a-direct-peering"></a>Creación y aprovisionamiento de un emparejamiento directo

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Iniciar sesión en la cuenta de Azure y seleccione la suscripción
[!INCLUDE [Account](./includes/account-powershell.md)]

### <a name="get-the-list-of-supported-peering-locations-for-direct-peering"></a><a name=direct-location></a>Obtención de la lista de ubicaciones de emparejamiento compatibles para el emparejamiento directo
[!INCLUDE [direct-location](./includes/direct-powershell-create-location.md)]

### <a name="create-a-direct-peering"></a><a name=create></a>Creación de un emparejamiento directo
[!INCLUDE [direct-peering](./includes/direct-powershell-create-connection.md)]

### <a name="verify-direct-peering"></a><a name=get></a>Verificación de un emparejamiento directo
[!INCLUDE [peering-direct-get](./includes/direct-powershell-get.md)]

## <a name="modify-a-direct-peering"></a><a name="modify"></a>Modificación de un emparejamiento directo
[!INCLUDE [peering-direct-modify](./includes/direct-powershell-modify.md)]

## <a name="deprovision-a-direct-peering"></a><a name="delete"></a>Desaprovisionamiento de un emparejamiento directo
[!INCLUDE [peering-direct-delete](./includes/delete.md)]

## <a name="next-steps"></a>Pasos siguientes

* [Creación o modificación de un emparejamiento de Exchange mediante PowerShell](howto-exchange-powershell.md)
* [Conversión de un emparejamiento de Exchange heredado en un recurso de Azure mediante PowerShell](howto-legacy-exchange-powershell.md)

## <a name="additional-resources"></a>Recursos adicionales
Puede obtener una descripción detallada de todos los parámetros ejecutando el siguiente comando:

```powershell
Get-Help Get-AzPeering -detailed
```

Para más información, consulte las [preguntas frecuentes sobre el emparejamiento de Internet](faqs.md).
