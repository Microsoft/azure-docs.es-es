---
title: Asociación de un ASN del mismo nivel a una suscripción de Azure mediante PowerShell
titleSuffix: Azure
description: Asociación de un ASN del mismo nivel a una suscripción de Azure mediante PowerShell
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: how-to
ms.date: 11/27/2019
ms.author: prmitiki
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: aed079c467139ac5819951c5895ba753ee38ae2d
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89067746"
---
# <a name="associate-peer-asn-to-azure-subscription-using-powershell"></a>Asociación de un ASN del mismo nivel a una suscripción de Azure mediante PowerShell

Antes de enviar una solicitud de emparejamiento, primero debe asociar su ASN con una suscripción a Azure mediante los pasos siguientes.

Si lo prefiere, puede completar esta guía mediante el [portal](howto-subscription-association-portal.md).

### <a name="working-with-azure-powershell"></a>Trabajo con Azure PowerShell
[!INCLUDE [CloudShell](./includes/cloudshell-powershell-about.md)]

## <a name="create-peerasn-to-associate-your-asn-with-azure-subscription"></a>Creación de un PeerASN (ASN del mismo nivel) para asociar su ASN con la suscripción de Azure

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Iniciar sesión en la cuenta de Azure y seleccione la suscripción
[!INCLUDE [Account](./includes/account-powershell.md)]

### <a name="register-for-peering-resource-provider"></a>Registrarse como proveedor de recursos de emparejamiento
Regístrese como proveedor de recursos de emparejamiento en su suscripción con este comando. Si no hace esto, no se podrá acceder a los recursos de Azure necesarios para configurar el emparejamiento.

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.Peering
```

Puede comprobar el estado del registro con los comandos que se muestran aquí:
```powershell
Get-AzResourceProvider -ProviderNamespace Microsoft.Peering
```

> [!IMPORTANT]
> Espere a que el estado de *RegistrationState* cambie a "Registrado" antes de continuar. Esta acción puede tardar entre 5 y 30 minutos en completarse tras ejecutar el comando.

### <a name="update-the-peer-information-associated-with-this-subscription"></a>Actualización de la información del mismo nivel asociada a la suscripción

Aquí se muestra un ejemplo de actualización de información del mismo nivel.

```powershell
New-AzPeerAsn `
    -Name "Contoso_1234" `
    -PeerName "Contoso" `
    -PeerAsn 1234 `
    -Email noc@contoso.com, support@contoso.com `
    -Phone "+1 (555) 555-5555"
```

> [!NOTE]
> "-Name" corresponde al nombre del recurso y puede ser el que prefiera. Pero "-peerName" corresponde al nombre de su empresa y debe asemejarse lo máximo posible a su perfil de PeeringDB. Tenga en cuenta que el valor de "-peerName" solo admite los caracteres a-z, A-Z y espacio.

Una suscripción puede tener varios ASN. Actualice la información de emparejamiento de cada ASN y asegúrese de que cada uno tenga un nombre único.

se espera que los elementos del mismo nivel tengan un perfil completo y actualizado en [PeeringDB](https://www.peeringdb.com). Usamos esta información durante el registro para validar los detalles del ASN del mismo nivel, como su información de NOC, sus datos de contacto técnico, su presencia en las instalaciones de emparejamiento, etc.

Tenga en cuenta que, en lugar de **{subscriptionId}** en la salida anterior, se mostrará el identificador de la suscripción actual.

## <a name="view-status-of-a-peerasn"></a>Comprobación del estado de un PeerASN

Compruebe el estado de validación del ASN con el siguiente comando:

```powershell
Get-AzPeerAsn
```

Este es un ejemplo de respuesta:
```powershell
PeerContactInfo : Microsoft.Azure.PowerShell.Cmdlets.Peering.Models.PSContactInfo
PeerName        : Contoso
ValidationState : Approved
PeerAsnProperty : 1234
Name            : Contoso_1234
Id              : /subscriptions/{subscriptionId}/providers/Microsoft.Peering/peerAsns/Contoso_1234
Type            : Microsoft.Peering/peerAsns
```

> [!IMPORTANT]
> Espere a que el estado de ValidationState cambie a "Approved" (aprobado) antes de enviar una solicitud de emparejamiento. La aprobación puede tardar hasta 12 horas.

## <a name="modify-peerasn"></a>Modificación de un PeerAsn
Puede modificar la información de contacto del NOC en cualquier momento.

Aquí tiene un ejemplo:

```powershell
Set-PeerAsn -Name Contoso_1234 -Email "newemail@test.com" -Phone "1800-000-0000"
```

## <a name="delete-peerasn"></a>Eliminación de un PeerAsn
Actualmente no se admite la eliminación de PeerASN. Si necesita eliminar un PeerASN, póngase en contacto con el equipo de [emparejamiento de Microsoft](mailto:peering@microsoft.com).

## <a name="next-steps"></a>Pasos siguientes

* [Creación o modificación de un emparejamiento directo](howto-direct-powershell.md)
* [Conversión de un emparejamiento directo heredado en un recurso de Azure](howto-legacy-direct-powershell.md)
* [Creación o modificación de un emparejamiento de Exchange](howto-exchange-powershell.md)
* [Conversión de un emparejamiento de Exchange heredado en un recurso de Azure](howto-legacy-exchange-powershell.md)

## <a name="additional-resources"></a>Recursos adicionales

Para obtener más información, consulte las [preguntas más frecuentes sobre el emparejamiento de Internet](faqs.md).
