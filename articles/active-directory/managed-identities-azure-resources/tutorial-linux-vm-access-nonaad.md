---
title: 'Tutorial`:` Uso de identidades administradas para acceder a Azure Key Vault (Linux): Azure AD'
description: Este tutorial contiene directrices para utilizar una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Resource Manager.
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
ms.date: 11/20/2017
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6756d66f176314ad5abd0c94f7bdf96c42a460a4
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89255316"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-managed-identity-to-access-azure-key-vault"></a>Tutorial: Uso de identidades administradas asignadas por el sistema de una máquina virtual Linux para acceder a Azure Key Vault 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

En este tutorial se muestra cómo usar una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Key Vault. Actuando como un arranque, Key Vault hace posible que la aplicación de cliente use el secreto para tener acceso a recursos que no están protegidos por Azure Active Directory (AD). Azure administra automáticamente la característica Managed Identities for Azure Resources, lo que le permite autenticar los servicios que admiten la autenticación de Azure AD sin necesidad de insertar credenciales en el código. 

Aprenderá a:

> [!div class="checklist"]
> * Concesión de acceso a la máquina virtual a un secreto almacenado en un almacén de claves 
> * Obtención de un token de acceso mediante la identidad de máquina virtual y usarlo para recuperar el secreto de Key Vault 
 
## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

## <a name="grant-your-vm-access-to-a-secret-stored-in-a-key-vault"></a>Concesión de acceso a la máquina virtual a un secreto almacenado en un almacén de claves  

Mediante el uso de identidades de servicio administradas para los recursos de Azure, el código puede obtener tokens de acceso para autenticarse en aquellos recursos que admitan la autenticación de Azure Active Directory. Sin embargo, no todos los servicios de Azure admiten la autenticación de Azure AD. Para usar identidades administradas para los recursos de Azure con esos servicios, almacene las credenciales del servicio en Azure Key Vault y use la característica Managed Identities for Azure Resources a fin de acceder a Key Vault y recuperar las credenciales. 

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
 
## <a name="get-an-access-token-using-the-vms-identity-and-use-it-to-retrieve-the-secret-from-the-key-vault"></a>Obtención de un token de acceso mediante la identidad de máquina virtual y usarlo para recuperar el secreto de Key Vault  

Para completar estos pasos, necesitará un cliente SSH.  Si usa Windows, puede usar el cliente SSH en el [Subsistema de Windows para Linux](/windows/wsl/about). Si necesita ayuda para configurar las claves del cliente de SSH, consulte [Uso de SSH con Windows en Azure](../../virtual-machines/linux/ssh-from-windows.md) o [Creación y uso de un par de claves SSH pública y privada para máquinas virtuales Linux en Azure](../../virtual-machines/linux/mac-create-ssh-keys.md).
 
1. En el portal, vaya a la máquina virtual Linux y, en **Información general**, haga clic en **Conectar**. 
2. **Conéctese** a la máquina virtual con el cliente SSH que elija. 
3. En la ventana del terminal, use CURL para realizar una solicitud al punto de conexión local de identidades administradas para recursos de Azure y obtener un token de acceso para Azure Key Vault.  
 
    La solicitud CURL para el token de acceso está debajo.  
    
    ```bash
    curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net' -H Metadata:true  
    ```
    La respuesta incluye el token de acceso que necesita para acceder a Resource Manager. 
    
    Respuesta:  
    
    ```bash
    {"access_token":"eyJ0eXAi...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1504130527",
    "not_before":"1504126627",
    "resource":"https://vault.azure.net",
    "token_type":"Bearer"} 
    ```
    
    Puede usar este token de acceso para autenticarse en Azure Key Vault.  La siguiente solicitud CURL muestra cómo leer un secreto desde el almacén de claves usando CURL y la API de REST de Key Vault.  Necesitará la dirección URL de su almacén de claves, que se encuentra en la sección de **Información esencial** de la página **Introducción** del almacén de claves.  También necesitará el token de acceso que obtuvo en la llamada anterior. 
        
    ```bash
    curl https://<YOUR-KEY-VAULT-URL>/secrets/<secret-name>?api-version=2016-10-01 -H "Authorization: Bearer <ACCESS TOKEN>" 
    ```
    
    La respuesta tendrá un aspecto similar al siguiente: 
    
    ```bash
    {"value":"p@ssw0rd!","id":"https://mytestkeyvault.vault.azure.net/secrets/MyTestSecret/7c2204c6093c4d859bc5b9eff8f29050","attributes":{"enabled":true,"created":1505088747,"updated":1505088747,"recoveryLevel":"Purgeable"}} 
    ```
    
Cuando haya recuperado el secreto del almacén de claves, podrá usarlo para autenticarse en un servicio que requiere un nombre y una contraseña.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a utilizar una identidad administrada asignada por el sistema de una máquina virtual Linux para acceder a Azure Key Vault.  Para obtener más información sobre Azure Key Vault, vea:

> [!div class="nextstepaction"]
>[Azure Key Vault](../../key-vault/general/overview.md)