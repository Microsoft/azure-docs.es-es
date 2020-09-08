---
title: Creación o modificación de un emparejamiento de Exchange mediante PowerShell
titleSuffix: Azure
description: Creación o modificación de un emparejamiento de Exchange mediante PowerShell
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: how-to
ms.date: 11/27/2019
ms.author: prmitiki
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 59d098f1ff0f78e37601544619c603b776e0cf22
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89071761"
---
# <a name="create-or-modify-an-exchange-peering-by-using-powershell"></a>Creación o modificación de un emparejamiento de Exchange mediante PowerShell

En este artículo se describe cómo crear un emparejamiento de Microsoft Exchange con cmdlets de PowerShell y el modelo de implementación de Resource Manager. También se muestra cómo comprobar el estado del recurso, así como la forma de actualizarlo o eliminarlo y desaprovisionarlo.

Si lo prefiere, puede realizar los pasos de esta guía mediante [Azure Portal](howto-exchange-portal.md).

## <a name="before-you-begin"></a>Antes de empezar
* Revise los [Requisitos previos](prerequisites.md) y el [Tutorial del emparejamiento de Exchange](walkthrough-exchange-all.md) antes de comenzar la configuración.
* Si ya dispone de emparejamientos de Exchange con Microsoft que no se convertirán en recursos de Azure, consulte el artículo [Conversión de un emparejamiento de Exchange heredado en un recurso de Azure mediante PowerShell](howto-legacy-exchange-powershell.md).

### <a name="work-with-azure-powershell"></a>Funcionamiento con Azure PowerShell
[!INCLUDE [CloudShell](./includes/cloudshell-powershell-about.md)]

## <a name="create-and-provision-an-exchange-peering"></a>Creación y aprovisionamiento de un emparejamiento de Exchange

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Iniciar sesión en la cuenta de Azure y seleccione la suscripción
[!INCLUDE [Account](./includes/account-powershell.md)]

### <a name="get-the-list-of-supported-peering-locations-for-exchange-peering"></a><a name=exchange-location></a>Obtención de la lista de ubicaciones de emparejamiento compatibles para el emparejamiento de Exchange
[!INCLUDE [exchange-location](./includes/exchange-powershell-create-location.md)]

### <a name="create-an-exchange-peering"></a><a name=create></a>Creación de un emparejamiento de Exchange
[!INCLUDE [exchange-peering](./includes/exchange-powershell-create-connection.md)]

### <a name="get-exchange-peering"></a><a name=get></a>Obtención de un emparejamiento de Exchange
[!INCLUDE [peering-exchange-get](./includes/exchange-powershell-get.md)]

## <a name="modify-an-exchange-peering"></a><a name="modify"></a>Modificación de un emparejamiento de Exchange
[!INCLUDE [peering-exchange-modify](./includes/exchange-powershell-modify.md)]

## <a name="deprovision-an-exchange-peering"></a><a name=delete></a>Desaprovisionamiento de un emparejamiento de Exchange

[!INCLUDE [peering-exchange-delete](./includes/delete.md)]

## <a name="next-steps"></a>Pasos siguientes

* [Creación o modificación de un emparejamiento directo mediante PowerShell](howto-direct-powershell.md)
* [Conversión de un emparejamiento directo heredado en un recurso de Azure mediante PowerShell](howto-legacy-direct-powershell.md)

## <a name="additional-resources"></a>Recursos adicionales
Puede obtener una descripción detallada de todos los parámetros ejecutando el siguiente comando:

```powershell
Get-Help Get-AzPeering -detailed
```

Para obtener más información, consulte las [preguntas más frecuentes sobre el emparejamiento de Internet](faqs.md).
