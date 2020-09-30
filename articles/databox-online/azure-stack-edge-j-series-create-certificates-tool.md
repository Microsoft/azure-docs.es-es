---
title: Creación de certificados con la herramienta Microsoft Azure Stack Hub Readiness Checker | Microsoft Docs
description: Describe cómo crear solicitudes de certificado y, después, obtener e instalar certificados en un dispositivo Azure Stack Edge Pro con GPU mediante la herramienta Azure Stack Hub Readiness Checker.
services: Azure Stack Edge Pro
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: 544625fe9fd2dbd87ad7330d7277494cbfbe6eb9
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90891089"
---
# <a name="create-certificates-for-your-azure-stack-edge-pro-using-azure-stack-hub-readiness-checker-tool"></a>Creación de certificados para Azure Stack Edge Pro con la herramienta Azure Stack Hub Readiness Checker 

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

En este artículo se describe cómo crear certificados para Azure Stack Edge Pro mediante la herramienta Azure Stack Hub Readiness Checker. 

## <a name="using-azure-stack-hub-readiness-checker-tool"></a>Uso de la herramienta Azure Stack Hub Readiness Checker

Use la herramienta Azure Stack Hub Readiness Checker para crear solicitudes de firma de certificado (CSR) para la implementación de un dispositivo Azure Stack Edge Pro. Estas solicitudes se pueden crear después de hacer el pedido del dispositivo Azure Stack Edge Pro y esperar a que llegue el dispositivo. 

> [!NOTE]
> Use esta herramienta solo con fines de prueba o desarrollo, y no para dispositivos de producción. 

Puede usar la herramienta Azure Stack Hub Readiness Checker (AzsReadinessChecker) para solicitar los siguientes certificados:

- Certificado de Azure Resource Manager
- Certificado de UI local
- Certificado de nodo
- Certificado de blob
- Certificado de VPN


## <a name="prerequisites"></a>Requisitos previos

Para crear solicitudes de firma de certificado para la implementación de un dispositivo Azure Stack Edge Pro, asegúrese de lo siguiente: 

- Tiene un cliente que ejecuta Windows 10 o Windows Server 2016 o posterior. 
- Ha descargado la herramienta Microsoft Azure Stack Hub Readiness Checker 1.2002.1133.85 [de la Galería de PowerShell](https://aka.ms/AzsReadinessChecker) en este sistema. Es posible que tenga que buscar este paquete. Esta versión de la herramienta es la única que puede crear certificados para dispositivos Azure Stack Edge Pro.
- Tiene la siguiente información para los certificados:
  - Nombre de dispositivo
  - Número de serie del nodo
  - Nombres de dominio completos (FQDN) externos

## <a name="generate-certificate-signing-requests"></a>Generación de solicitudes de firma de certificado

Siga estos pasos para preparar los certificados de dispositivo Azure Stack Edge Pro:

1. Ejecute PowerShell como administrador (5.1 o posterior).
2. Instale la herramienta Azure Stack Hub Readiness Checker. En el símbolo del sistema de PowerShell, escriba: 

    ```azurepowershell
    Install-Module -Name Microsoft.AzureStack.ReadinessChecker -RequiredVersion 1.2002.1133.85 -Force
    ```

    Para comprobar la versión instalada, escriba:  

    ```azurepowershell
    Get-InstalledModule -Name Microsoft.AzureStack.ReadinessChecker  | ft Name, Version 
    ```

3. Cree un directorio para todos los certificados si no existe. Escriba: 
    
    ```azurepowershell
    New-Item "C:\certrequest" -ItemType Directory
    ``` 
    
4. Para crear una solicitud de certificados, indique la siguiente información. Si va a generar un certificado de VPN, no se aplican algunas de estas entradas. 
    
    |Entrada |Descripción  |
    |---------|---------|
    |`OutputRequestPath`|Ruta de acceso del archivo en el cliente local en el que quiere que se creen las solicitudes de certificados.        |
    |`DeviceName`|Nombre del dispositivo en la página **Dispositivos** en la interfaz de usuario web local del dispositivo. <br> Este campo no es obligatorio para un certificado de VPN.         |
    |`NodeSerialNumber`|Número de serie del nodo de dispositivo en la página **Red** en la interfaz de usuario web local del dispositivo. <br> Este campo no es obligatorio para un certificado de VPN.       |
    |`ExternalFQDN`|Para obtener esta dirección URL, vaya a la página **Dispositivos** en la interfaz de usuario web local del dispositivo.         |
    |`RequestType`|El tipo de solicitud puede ser para `MultipleCSR`, distintos certificados para los distintos puntos de conexión, o `SingleCSR`, un único certificado para todos los puntos de conexión. <br> Este campo no es obligatorio para un certificado de VPN.     |

    Para todos los certificados excepto el certificado de VPN, escriba: 
    
    ```azurepowershell
    $edgeCSRparams = @{
        CertificateType = 'AzureStackEdgeDEVICE'
        DeviceName = 'myTEA1'
        NodeSerialNumber = 'VM1500-00025'
        externalFQDN = 'azurestackedge.contoso.com'
        requestType = 'MultipleCSR'
        OutputRequestPath = "C:\certrequest"
    }
    New-AzsCertificateSigningRequest @edgeCSRparams
    ```

    Si va a crear un certificado de VPN, escriba: 

    ```azurepowershell
    $edgeCSRparams = @{
        CertificateType = 'AzureStackEdgeVPN'
        externalFQDN = 'azurestackedge.contoso.com'
        OutputRequestPath = "C:\certrequest"
    }
    New-AzsCertificateSigningRequest @edgeCSRparams
    ```

    
5. Encontrará los archivos de solicitud de certificados en el directorio que especificó en el parámetro OutputRequestPath anterior. Al usar el parámetro `MultipleCSR`, verá cuatro archivos con la extensión `.req`. Los archivos son los siguientes:

    
    |Nombres de archivo  |Tipo de solicitud de certificados  |
    |---------|---------|
    |Comenzando con su `DeviceName`     |Solicitud de certificados de interfaz de usuario web local      |
    |Comenzando con su `NodeSerialNumber`     |Solicitud de certificado de nodo         |
    |A partir de `login`     |Solicitud de certificado de punto de conexión de Azure Resource Manager       |
    |A partir de `wildcard`     |Solicitud de certificado de almacenamiento de blobs; contiene un carácter comodín porque abarca todas las cuentas de almacenamiento que se pueden crear en el dispositivo.          |
    |A partir de `AzureStackEdgeVPNCertificate`     |Solicitud de certificado de cliente VPN.         |

    También verá una carpeta INF. Contiene un archivo de información de <edge-nombrededispositivo> en texto no cifrado que explica los detalles del certificado.  


6. Envíe estos archivos a su entidad de certificación (ya sea interna o pública). Asegúrese de que la entidad de certificación genere certificados con su solicitud generada que cumple los requisitos de certificados de Azure Stack Edge Pro para [certificados de nodos](azure-stack-edge-j-series-manage-certificates.md#node-certificates), [certificados de puntos de conexión](azure-stack-edge-j-series-manage-certificates.md#endpoint-certificates) y [certificados de interfaces de usuario locales](azure-stack-edge-j-series-manage-certificates.md#local-ui-certificates).

## <a name="prepare-certificates-for-deployment"></a>Preparación de los certificados para la implementación

Los archivos de certificado que obtiene de la entidad de certificación deben importarse y exportarse de forma que las propiedades coincidan con los requisitos de certificado del dispositivo Azure Stack Edge Pro. Complete los pasos siguientes en el mismo sistema donde generó las solicitudes de firma de certificados.

- Para importar los certificados, siga los pasos que se indican en [Importación de certificados en los clientes que acceden al dispositivo Azure Stack Edge Pro](azure-stack-edge-j-series-manage-certificates.md#import-certificates-on-the-client-accessing-the-device).

- Para exportar los certificados, siga los pasos que se indican en [Exportación de certificados de los clientes que acceden al dispositivo Azure Stack Edge Pro](azure-stack-edge-j-series-manage-certificates.md#import-certificates-on-the-client-accessing-the-device).


## <a name="validate-certificates"></a>Validación de certificados

En primer lugar, generará una estructura de carpetas adecuada y colocará los certificados en las carpetas correspondientes. Solo entonces se validarán los certificados con la herramienta.

1. Ejecute PowerShell como administrador.

2. Para generar la estructura de carpetas adecuada, en el símbolo del sistema, escriba:

    `New-AzsCertificateFolder -CertificateType AzureStackEdge -OutputPath "$ENV:USERPROFILE\Documents\AzureStackCSR"`

3. Convierta la contraseña PFX en una cadena segura. Escriba:       

    `$pfxPassword = Read-Host -Prompt "Enter PFX Password" -AsSecureString` 

4. A continuación, valide los certificados. Escriba:

    `Invoke-AzsCertificateValidation -CertificateType AzureStackEdge -DeviceName mytea1 -NodeSerialNumber VM1500-00025 -externalFQDN azurestackedge.contoso.com -CertificatePath $ENV:USERPROFILE\Documents\AzureStackCSR\AzureStackEdge -pfxPassword $pfxPassword`

## <a name="next-steps"></a>Pasos siguientes

[Implementación de un dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md)
