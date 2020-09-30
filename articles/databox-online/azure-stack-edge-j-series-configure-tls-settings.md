---
title: Configuración de TLS 1.2 en los clientes de Windows que acceden al dispositivo Azure Stack Edge Pro con GPU
description: Se describe cómo se configura TLS 1.2 en los clientes de Windows que acceden al dispositivo Azure Stack Edge Pro con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: 9a6b0910fcfd2a632f2520a2fe683b15592017cf
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90891173"
---
# <a name="configure-tls-12-on-windows-clients-accessing-azure-stack-edge-pro-device"></a>Configuración de TLS 1.2 en los clientes de Windows que acceden a un dispositivo Azure Stack Edge Pro

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

Si usa un cliente de Windows para acceder a un dispositivo Azure Stack Edge Pro, debe configurar TLS 1.2 en el cliente. En este artículo se proporcionan recursos e instrucciones para configurar TLS 1.2 en el cliente de Windows. 

Las instrucciones que se proporcionan aquí se basan en las pruebas realizadas en un cliente que ejecuta Windows Server 2016.

## <a name="configure-tls-12-for-current-powershell-session"></a>Configuración de TLS 1.2 para la sesión actual de PowerShell

Siga estos pasos para configurar TLS 1.2 en el cliente.

1. Ejecute PowerShell como administrador.
2. Para establecer TLS 1.2 para la sesión actual de PowerShell, escriba:
  
    ```azurepowershell
    $TLS12Protocol = [System.Net.SecurityProtocolType] 'Ssl3 , Tls12'
    [System.Net.ServicePointManager]::SecurityProtocol = $TLS12Protocol
    ```
## <a name="configure-tls-12-on-client"></a>Configuración de TLS 1.2 en el cliente

Si desea establecer TLS 1.2 en todo el sistema para su entorno, siga las instrucciones de estos documentos:

- [General: Cómo se habilita TLS 1.2](https://docs.microsoft.com/windows-server/security/tls/tls-registry-settings#tls-12)
- [Procedimientos para habilitar TLS 1.2 en clientes](https://docs.microsoft.com/configmgr/core/plan-design/security/enable-tls-1-2-client)
- [Procedimientos para habilitar TLS 1.2 en los servidores y sistemas de sitios remotos](https://docs.microsoft.com/configmgr/core/plan-design/security/enable-tls-1-2-server)
- [Protocolos de TLS/SSL (Schannel SSP)](https://docs.microsoft.com/windows-server/security/tls/manage-tls#configuring-tls-ecc-curve-order)
- [Conjuntos de cifrado](https://docs.microsoft.com/windows-server/security/tls/tls-registry-settings#tls-12): En concreto, [Configuración del orden del conjunto de cifrado TLS](https://docs.microsoft.com/windows-server/security/tls/manage-tls#configuring-tls-cipher-suite-order). Asegúrese de que enumera los conjuntos de cifrado actuales y antepone los que faltan en la lista siguiente:

    - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
    - TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384

    También puede agregar estos conjuntos de cifrado editando directamente la configuración del registro.

    ```azurepowershell
    New-ItemProperty -Path "$HklmSoftwarePath\Policies\Microsoft\Cryptography\Configuration\SSL\00010002" -Name "Functions"  -PropertyType String -Value ("TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384, TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384, TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384")
    ```

- Establecimiento de curvas elípticas

    Asegúrese de que enumera las curvas elípticas actuales y antepone las que falten en la lista siguiente:

    - P-256 
    - P-384

    También puede agregar estas curvas elípticas editando directamente la configuración del registro.
    
    ```azurepowershell
    New-ItemProperty -Path "$HklmSoftwarePath\Policies\Microsoft\Cryptography\Configuration\SSL\00010002" -Name "EccCurves" -PropertyType MultiString -Value @("NistP256", "NistP384")
    ```
    
    - [Establezca el tamaño mínimo de intercambio de claves RSA en 2048](https://docs.microsoft.com/windows-server/security/tls/tls-registry-settings#keyexchangealgorithm---client-rsa-key-sizes).



## <a name="next-steps"></a>Pasos siguientes

[Conexión a Azure Resource Manager](azure-stack-edge-j-series-connect-resource-manager.md)