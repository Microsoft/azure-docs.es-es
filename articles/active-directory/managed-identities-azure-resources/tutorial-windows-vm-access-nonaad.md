---
title: 'Tutorial`:` Uso de identidades administradas para acceder a Azure Key Vault (Windows): Azure AD'
description: Este tutorial contiene directrices acerca de cómo utilizar una identidad administrada asignada por el sistema de una máquina virtual Windows para acceder a Azure Key Vault.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: daveba
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/10/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2890eb2211ac0a105363742a0e900e52a577ed27
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89255832"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-key-vault"></a>Tutorial: Uso de las identidades administradas asignadas por el sistema de una máquina virtual Windows para acceder a Azure Key Vault 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

En este tutorial se muestra cómo usar una identidad administrada asignada por el sistema en una máquina virtual Windows para acceder a Azure Key Vault. Actuando como un arranque, Key Vault hace posible que la aplicación de cliente use el secreto para tener acceso a recursos que no están protegidos por Azure Active Directory (AD). Las identidades de MSI son administradas automáticamente por Azure y le permiten autenticar los servicios que admiten la autenticación de Azure AD sin necesidad de insertar credenciales en el código. 

Aprenderá a:


> [!div class="checklist"]
> * Concesión de acceso a la máquina virtual a un secreto almacenado en un almacén de claves 
> * Obtener un token de acceso mediante la identidad de máquina virtual y usarlo para recuperar el secreto de Key Vault 

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]


## <a name="enable"></a>Habilitar

[!INCLUDE [msi-tut-enable](../../../includes/active-directory-msi-tut-enable.md)]



## <a name="grant-access"></a>Conceder acceso  
 
En esta sección se muestra cómo conceder acceso a la máquina virtual a un secreto almacenado en un almacén de claves. Mediante Managed Identities for Azure Resources, el código puede obtener tokens de acceso para autenticarse en aquellos recursos que admitan la autenticación de Azure AD.  Sin embargo, no todos los servicios de Azure admiten la autenticación de Azure AD. Para usar Managed Identities for Azure Resources con esos servicios, almacene las credenciales del servicio en Azure Key Vault y use una identidad administrada de la máquina virtual a fin de acceder a Key Vault y recuperar las credenciales. 

En primer lugar, es necesario crear un almacén de claves y conceder a la identidad administrada asignada por el sistema de la máquina virtual acceso al mismo.   

1. En la parte superior de la barra de navegación izquierda, seleccione **Crear un recurso** > **Seguridad e identidad** > **Key Vault**.  
2. Proporcione un **nombre** para el nuevo almacén de claves. 
3. Busque el almacén de claves en la misma suscripción y el mismo grupo de recursos que la máquina virtual que creó anteriormente. 
4. Seleccione **Directivas de acceso** y haga clic en **Agregar nueva**. 
5. En Configurar a partir de plantilla, seleccione **Administración de secretos**. 
6. Elija **Seleccionar la entidad de seguridad** y, en el campo de búsqueda, escriba el nombre de la máquina virtual que creó anteriormente.  Seleccione la máquina virtual de la lista de resultados y haga clic en **Seleccionar** . 
7. Haga clic en **Aceptar** para terminar de agregar la nueva directiva de acceso, y en **Aceptar** para finalizar la selección de la directiva de acceso. 
8. Haga clic en **Crear** para terminar de crear el almacén de claves. 

    ![Texto alternativo de imagen](./media/msi-tutorial-windows-vm-access-nonaad/msi-blade.png)


A continuación, agregue un secreto al almacén de claves, de forma que más adelante pueda recuperarlo mediante código que se ejecuta en la máquina virtual: 

1. Seleccione **Todos los recursos** y busque y seleccione el almacén de claves que acaba de crear. 
2. Seleccione **Secretos** y haga clic en **Agregar**. 
3. En **Opciones de carga**, seleccione **Manual**. 
4. Escriba un nombre y un valor para el secreto.  El valor puede ser cualquiera de su elección. 
5. Deje la fecha de activación y la fecha de expiración y deje la opción **Habilitado** en **Sí**. 
6. Haga clic en **Crear** para crear el secreto. 
 
## <a name="access-data"></a>Acceso a los datos  

En esta sección se muestra cómo obtener un token de acceso mediante la identidad de máquina virtual y usarlo para recuperar el secreto del almacén de claves. Si no tiene PowerShell 4.3.1 o superior instalado, necesitará [descargar e instalar la versión más reciente](/powershell/azure/).

En primer lugar, vamos a usar la identidad administrada asignada por el sistema de la máquina virtual a fin de obtener un token de acceso para autenticarse en Key Vault:
 
1. En el portal, vaya a **Virtual Machines** y diríjase a la máquina virtual Windows. A continuación, en **Introducción**, haga clic en **Conectar**.
2. Escriba su **nombre de usuario** y **contraseña** que agregó cuando creó la **máquina virtual Windows**.  
3. Ahora que ha creado una **conexión a Escritorio remoto** con la máquina virtual, abra PowerShell en la sesión remota.  
4. En PowerShell, invoque la solicitud web en el inquilino para obtener el token del host local en el puerto específico de la máquina virtual.  

    La solicitud de PowerShell:
    
    ```powershell
    $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net' -Method GET -Headers @{Metadata="true"} 
    ```
    
    A continuación, extraiga la respuesta completa, que se almacena como una cadena con formato de notación de objetos JavaScript (JSON) en el objeto $response.  
    
    ```powershell
    $content = $response.Content | ConvertFrom-Json 
    ```
    
    Luego, extraiga el token de acceso de la respuesta.  
    
    ```powershell
    $KeyVaultToken = $content.access_token 
    ```
    
    Finalmente, use el comando Invoke-WebRequest de PowerShell para recuperar el secreto que creó anteriormente en el almacén de claves y pasar el token de acceso en el encabezado de autenticación.  Necesitará la dirección URL de su almacén de claves, que se encuentra en la sección de **Información esencial** de la página **Introducción** del almacén de claves.  
    
    ```powershell
    (Invoke-WebRequest -Uri https://<your-key-vault-URL>/secrets/<secret-name>?api-version=2016-10-01 -Method GET -Headers @{Authorization="Bearer $KeyVaultToken"}).content 
    ```
    
    La respuesta tendrá un aspecto similar al siguiente: 
    
    ```powershell
    {"value":"p@ssw0rd!","id":"https://mytestkeyvault.vault.azure.net/secrets/MyTestSecret/7c2204c6093c4d859bc5b9eff8f29050","attributes":{"enabled":true,"created":1505088747,"updated":1505088747,"recoveryLevel":"Purgeable"}} 
    ```
    
Cuando haya recuperado el secreto del almacén de claves, podrá usarlo para autenticarse en un servicio que requiere un nombre y una contraseña. 


## <a name="disable"></a>Disable

[!INCLUDE [msi-tut-disable](../../../includes/active-directory-msi-tut-disable.md)]



## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar una identidad administrada asignada por el sistema de una máquina virtual Windows para acceder a Azure Key Vault.  Para obtener más información sobre Azure Key Vault, vea:

> [!div class="nextstepaction"]
>[Azure Key Vault](../../key-vault/general/overview.md)