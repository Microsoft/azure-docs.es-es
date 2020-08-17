---
title: Protección de mensajes B2B con certificados
description: Agregue certificados para proteger los mensajes B2B de Azure Logic Apps con Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 08/17/2018
ms.openlocfilehash: 7d57536eee62c415058dd454f3da6ee5f3d04898
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090338"
---
# <a name="improve-security-for-b2b-messages-by-using-certificates"></a>Mejora de la seguridad de los mensajes B2B mediante certificados

Si necesita mantener la confidencialidad de la comunicación B2B, puede aumentar su seguridad en las aplicaciones de integración empresarial, en particular, las aplicaciones lógicas, si agrega certificados a la cuenta de integración. Los certificados son documentos digitales que comprueban la identidad de los participantes de las comunicaciones electrónicas y ayudan a proteger su comunicación de las siguientes formas:

* Cifrado del contenido de los mensajes.
* Firma digital de los mensajes.

Puede usar los siguientes certificados en las aplicaciones de integración empresarial:

* Los [Certificados públicos](https://en.wikipedia.org/wiki/Public_key_certificate), que debe adquirir desde la [entidad de certificación (CA)](https://en.wikipedia.org/wiki/Certificate_authority) de una red pública de internet, pero no requieren ninguna clave. 

* Los certificados privados o [*certificados autofirmados*](https://en.wikipedia.org/wiki/Self-signed_certificate), que crea y emite usted mismo, pero que además requieren claves privadas. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="upload-a-public-certificate"></a>Carga de un certificado público

Para usar un *certificado público* en las aplicaciones lógicas con funcionalidades B2B, antes hay que cargarlo en la cuenta de integración. Después de definir las propiedades en los [contratos](logic-apps-enterprise-integration-agreements.md) que cree, el certificado estará disponible para que pueda proteger los mensajes B2B.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). En el menú principal de Azure, seleccione **Todos los recursos**. En el cuadro de búsqueda, escriba el nombre de la cuenta de integración y, a continuación, seleccione la cuenta de integración que desee.

   ![Busque y seleccione su cuenta de integración](media/logic-apps-enterprise-integration-certificates/select-integration-account.png)  

2. En **Componentes**, seleccione el mosaico **Certificados**.

   ![Elección de "Certificados"](media/logic-apps-enterprise-integration-certificates/add-certificates.png)

3. En **Certificados**, elija **Agregar**. En **Agregar certificado**, proporcione los detalles de su certificado. Cuando termine, elija **Aceptar**.

   | Propiedad | Value | Descripción | 
   |----------|-------|-------------|
   | **Nombre** | <*nombre-certificado*> | El nombre del certificado, que es "publicCert" en este ejemplo | 
   | **Tipo de certificado** | Público | Su tipo de certificado |
   | **Certificate** | <*nombre-archivo-del-certificado*> | Para buscar y seleccionar el archivo de certificado que desea cargar, elija el icono de carpeta junto al cuadro **Certificado**. |
   ||||

   ![Elija "Añadir" y proporcione los detalles del certificado](media/logic-apps-enterprise-integration-certificates/public-certificate-details.png)

   Azure valida el certificado que eligió y después lo carga.

   ![Azure muestra el nuevo certificado](media/logic-apps-enterprise-integration-certificates/new-public-certificate.png) 

## <a name="upload-a-private-certificate"></a>Carga de un certificado privado

Para usar un *certificado privado* en las aplicaciones lógicas con funcionalidades B2B, antes hay que cargarlo en la cuenta de integración. También debe tener una clave privada que añadirá primero a [Azure Key Vault](../key-vault/general/overview.md). 

Después de definir las propiedades en los [contratos](logic-apps-enterprise-integration-agreements.md) que cree, el certificado estará disponible para que pueda proteger los mensajes B2B.

> [!NOTE]
> En el caso de los certificados privados, asegúrese de agregar un certificado público correspondiente que aparezca en la opción [Enviar y recibir](logic-apps-enterprise-integration-as2.md) del **contrato de AS2** para la firma y el cifrado de mensajes.

1. [Añada la clave privada a Azure Key Vault](../key-vault/certificates/certificate-scenarios.md#import-a-certificate) y especifique un **nombre de clave**.
   
2. Autorice a Azure Logic Apps para que pueda realizar operaciones en Azure Key Vault. Para conceder acceso a la entidad de seguridad de servicio de Logic Apps, use el comando de PowerShell [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy), como en el ejemplo siguiente:

   `Set-AzKeyVaultAccessPolicy -VaultName 'TestcertKeyVault' -ServicePrincipalName 
   '7cd684f4-8a78-49b0-91ec-6a35d38739ba' -PermissionsToKeys decrypt, sign, get, list`
 
3. Inicie sesión en [Azure Portal](https://portal.azure.com). En el menú principal de Azure, seleccione **Todos los recursos**. En el cuadro de búsqueda, escriba el nombre de la cuenta de integración y, a continuación, seleccione la cuenta de integración que desee.

   ![Búsqueda de la cuenta de integración](media/logic-apps-enterprise-integration-certificates/select-integration-account.png) 

4. En **Componentes**, seleccione el mosaico **Certificados**.  

   ![Selección del icono Certificados](media/logic-apps-enterprise-integration-certificates/add-certificates.png)

5. En **Certificados**, elija **Agregar**. En **Agregar certificado**, proporcione los detalles de su certificado. Cuando termine, elija **Aceptar**.

   | Propiedad | Value | Descripción | 
   |----------|-------|-------------|
   | **Nombre** | <*nombre-certificado*> | El nombre del certificado, que es "publicCert" en este ejemplo | 
   | **Tipo de certificado** | Privada | Su tipo de certificado |
   | **Certificate** | <*nombre-archivo-del-certificado*> | Para buscar y seleccionar el archivo de certificado que desea cargar, elija el icono de carpeta junto al cuadro **Certificado**. Cuando se usa un almacén de claves para la clave privada, el archivo cargado será el certificado público. | 
   | **Grupo de recursos** | <*grupo-recursos-de-cuenta-integración*> | Grupo de recursos de su cuenta de integración, que es "MyResourceGroup" en este ejemplo | 
   | **Key Vault** | <*nombre-almacén-claves*> | Nombre de su almacén de claves de Azure |
   | **Key name** | <*key-name*> | Nombre de la clave |
   ||||

   ![Elija "Añadir" y proporcione los detalles del certificado](media/logic-apps-enterprise-integration-certificates/private-certificate-details.png)

   Azure valida el certificado que eligió y después lo carga.

   ![Azure muestra el nuevo certificado](media/logic-apps-enterprise-integration-certificates/new-private-certificate.png) 

## <a name="next-steps"></a>Pasos siguientes

* [Creación de un contrato B2B](logic-apps-enterprise-integration-agreements.md)
