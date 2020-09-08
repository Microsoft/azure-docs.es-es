---
title: 'Tutorial: Transferencia de datos a la cuenta de almacenamiento de Azure Stack Edge con GPU | Microsoft Docs'
description: Obtenga información sobre cómo agregar y conectarse a cuentas de almacenamiento de Edge y local en un dispositivo de Azure Stack Edge con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to add and connect to storage accounts on Azure Stack Edge so I can use it to transfer data to Azure.
ms.openlocfilehash: 66c09b7fa0eb8fbf709441da5996fb2237b3e284
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89256403"
---
# <a name="tutorial-transfer-data-via-storage-accounts-with-azure-stack-edge-gpu"></a>Tutorial: Transferencia de datos a través de cuentas de almacenamiento de Azure Stack Edge con GPU 

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

En este tutorial se describe cómo agregar cuentas de almacenamiento a un dispositivo de Azure Stack Edge y conectarse a ellas. Después de agregar las cuentas de almacenamiento, Azure Stack Edge puede transferir datos a Azure.

Este procedimiento tarda aproximadamente 30 minutos en completarse.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Agregar una cuenta de almacenamiento
> * Conexión con la cuenta de almacenamiento

 
## <a name="prerequisites"></a>Requisitos previos

Antes de agregar cuentas de almacenamiento a Azure Stack Edge, compruebe lo siguiente:

- Ha instalado el dispositivo físico tal como se describe en [Instalación de Azure Stack Edge](azure-stack-edge-gpu-deploy-install.md).

- Ha activado el dispositivo físico tal como se describe en [Activación de Azure Stack Edge](azure-stack-edge-gpu-deploy-activate.md).


## <a name="add-an-edge-storage-account"></a>Adición de una cuenta de almacenamiento de Edge

Para crear una cuenta de almacenamiento de Edge, realice el procedimiento siguiente:

[!INCLUDE [Add an Edge storage account](../../includes/azure-stack-edge-gateway-add-storage-account.md)]


## <a name="connect-to-the-edge-storage-account"></a>Conexión a la cuenta de almacenamiento de Edge

Ahora puede conectarse a las API de REST de almacenamiento de Edge a través de *http* o *https*.

- *Https* es la forma segura y recomendada.
- *Http* se utiliza para conectarse en redes de confianza.

## <a name="connect-via-http"></a>Conexión mediante http

Estos son los pasos para conectarse a las API de REST de almacenamiento de Edge mediante http:

- Adición de la dirección VIP de servicio coherente de Azure y el punto de conexión de Blob service al host remoto
- Comprobación de la conexión 

En las secciones siguientes se describen todos estos pasos.

### <a name="add-device-ip-address-and-blob-service-endpoint-to-the-remote-client"></a>Adición de la dirección IP del dispositivo y el punto de conexión de Blob service al cliente remoto

[!INCLUDE [Add device IP and blob service endpoint to host file](../../includes/azure-stack-edge-gateway-add-device-ip-address-blob-service-endpoint.md)]


### <a name="verify-connection"></a>Comprobación de la conexión

Para comprobar la conexión, normalmente necesitará la siguiente información (puede variar) que recopiló en el paso anterior:

- Nombre de la cuenta de almacenamiento.
- Clave de acceso de la cuenta de almacenamiento.
- Punto de conexión de Blob service.

Ya tiene el nombre de la cuenta de almacenamiento y el punto de conexión de Blob service. Puede obtener la clave de acceso de la cuenta de almacenamiento conectándose al dispositivo a través de Azure Resource Manager mediante un cliente de Azure PowerShell.

Siga los pasos descritos en [Conexión al dispositivo a través de Azure Resource Manager](azure-stack-edge-j-series-connect-resource-manager.md). Una vez que haya iniciado sesión en las API de dispositivo local a través de Azure Resource Manager, obtenga la lista de cuentas de almacenamiento en el dispositivo. Ejecute el siguiente cmdlet:

`Get-AzureRMStorageAccount`

En la lista de cuentas de almacenamiento del dispositivo, identifique la cuenta de almacenamiento para la que necesita la clave de acceso. Compruebe la cuenta de almacenamiento y el grupo de recursos.

A continuación se muestra una salida de ejemplo:

```azurepowershell
PS C:\windows\system32> Get-AzureRmStorageAccount

StorageAccountName ResourceGroupName Location SkuName     Kind    AccessTier CreationTime          ProvisioningState EnableHttpsTrafficOnly
------------------ ----------------- -------- -------     ----    ---------- ------------          ----------------- ----------------------
myasetiered1       myasetiered1      DBELocal StandardLRS Storage            11/27/2019 7:10:12 PM Succeeded         False
```

Para obtener la clave de acceso, ejecute el siguiente cmdlet:

`Get-AzureRmStorageAccountAccessKey`

A continuación se muestra una salida de ejemplo:

```azurepowershell
PS C:\windows\system32> Get-AzureRmStorageAccountKey

cmdlet Get-AzureRmStorageAccountKey at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
ResourceGroupName: myasetiered1
Name: myasetiered1

KeyName Value    Permissions                                                                                
------- -----    -----------                                                                                
key1    Jb2brrNjRNmArFcDWvL4ufspJjlo+Nie1uh8Mp4YUOVQNbirA1uxEdHeV8Z0dXbsG7emejFWI9hxyR1T93ZncA==        Full
key2    6VANuHzHcJV04EFeyPiWRsFWnHPkgmX1+a3bt5qOQ2qIzohyskIF/2gfNMqp9rlNC/w+mBqQ2mI42QgoJSmavg==        Full
```

Cópiela y guárdela. Usará esta clave para comprobar la conexión mediante el Explorador de Azure Storage.

Para comprobar que la conexión se establece correctamente, use el Explorador de Storage para conectarse a una cuenta de almacenamiento externa. [Descargue el Explorador de Storage](https://go.microsoft.com/fwlink/?LinkId=708343&clcid=0x409), si no lo tiene.

[!INCLUDE [Verify connection using Storage Explorer](../../includes/azure-stack-edge-gateway-verify-connection-storage-explorer.md)]


## <a name="connect-via-https"></a>Conexión mediante https

Estos son los pasos para conectarse a las API REST de Azure Blob Storage mediante https:

- Obtención del certificado de punto de conexión de blob
- Importación del certificado en el cliente o el host remoto
- Adición de la dirección IP del dispositivo y el punto de conexión de Blob service al cliente o al host remoto
- Configuración y comprobación de la conexión

En las secciones siguientes se describen todos estos pasos.

### <a name="get-certificate"></a>Obtención del certificado

Para acceder al almacenamiento de blobs a través de HTTPS, el dispositivo necesita un certificado SSL. También cargará este certificado en el dispositivo de Azure Stack Edge como archivo *.pfx* con una clave privada asociada. Para obtener más información sobre cómo crear (solo con fines de prueba y desarrollo) y cargar estos certificados en el dispositivo de Azure Stack Edge, vaya a:

- [Creación del certificado de punto de conexión de blob](azure-stack-edge-j-series-manage-certificates.md#create-certificates-optional).
- [Carga del certificado de punto de conexión de blob](azure-stack-edge-j-series-manage-certificates.md#upload-certificates).
- [Importación de certificados en el cliente que accede al dispositivo](azure-stack-edge-j-series-manage-certificates.md#import-certificates-on-the-client-accessing-the-device).

### <a name="import-certificate"></a>Importación de certificado

Si usa el Explorador de Azure Storage para conectarse a las cuentas de almacenamiento del dispositivo, también debe importar el certificado al Explorador de Storage en formato PEM. En el entorno de Windows, *.cer* codificado en base 64 es igual que el formato PEM.

Siga estos pasos para importar los certificados al Explorador de Azure Storage:

1. Asegúrese de que el destino del Explorador de Azure Storage sea las API de Azure Stack. Vaya a **Editar > API de Azure Stack de destino**. Cuando se le pida, actívela y reinicie el Explorador de Storage para que el cambio surta efecto.

2. Para importar certificados SSL, vaya a **Editar > Certificados SSL > Importar certificados**.

  
   ![Importación del certificado al Explorador de Storage](./media/azure-stack-edge-j-series-deploy-add-storage-accounts/import-cert-storage-explorer-1.png) 

3. Navegue y proporcione la cadena de firma y los certificados de blob. Tanto la cadena de firma como el certificado de blob deben estar en formato PEM, que es igual que el formato codificado en Base64 del sistema Windows. Se le notificará que los certificados se han importado correctamente.


### <a name="add-device-ip-address-and-blob-service-endpoint"></a>Adición de la dirección IP del dispositivo y el punto de conexión de Blob service

Siga los mismos pasos para [agregar la dirección IP del dispositivo y el punto de conexión de Blob service cuando se conecta mediante *http*](#add-device-ip-address-and-blob-service-endpoint-to-the-remote-client).

### <a name="configure-and-verify-connection"></a>Configuración y comprobación de la conexión

Siga los pasos para la [Configuración y comprobación de la conexión que usó durante la conexión mediante *http*](#verify-connection). La única diferencia es que debe dejar la opción *Usar http* desactivada.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha adquirido conocimientos sobre los siguientes temas de Azure Stack Edge:

> [!div class="checklist"]
> * Agregar una cuenta de almacenamiento
> * Conexión a una cuenta de almacenamiento

Para aprender a transformar los datos mediante Azure Stack Edge, pase al siguiente tutorial:

> [!div class="nextstepaction"]
> [Transformación de datos con Azure Stack Edge](./azure-stack-edge-j-series-deploy-configure-compute.md)


