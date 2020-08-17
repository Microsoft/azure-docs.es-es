---
title: 'Preguntas más frecuentes: importación de certificados de Azure Key Vault'
description: 'Preguntas más frecuentes: importación de certificados de Azure Key Vault'
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: conceptual
ms.date: 07/20/2020
ms.author: sebansal
ms.openlocfilehash: 493c77a8f875018627bfe3167e66addeaf65d089
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87445790"
---
# <a name="frequently-asked-questions---azure-key-vault-certificate-import"></a>Preguntas más frecuentes: importación de certificados de Azure Key Vault

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="how-can-i-import-a-certificate-in-azure-key-vault"></a>¿Cómo importo un certificado en Azure Key Vault?

Importación de certificados: para la operación de importación, Azure Key Vault acepta dos formatos de certificado, PEM y PFX. Aunque hay archivos PEM que contienen solo la parte pública, Azure Key Vault requiere y acepta únicamente un archivo PEM o PFX en la carpeta de archivos y con una clave privada. Siga el [tutorial para importar certificados](https://docs.microsoft.com/azure/key-vault/certificates/tutorial-import-certificate#import-a-certificate-to-key-vault).

### <a name="after-importing-password-protected-certificate-into-the-key-vault-and-then-downloading-it-i-am-not-able-to-see-the-password-associated-with-the-certificate"></a>Después de importar el certificado protegido por contraseña en un almacén de claves y de descargarlo, no pude ver la contraseña asociada al certificado.
    
El certificado protegido que se cargó no guarda la contraseña asociada a él después de almacenarse en el almacén de claves. Solo se requiere una vez durante la operación de importación. Aunque se trata de una decisión de diseño, siempre puede obtener el certificado como secreto y convertirlo de Base64 a PFX y agregar la contraseña anterior a través de [Azure PowerShell](https://social.technet.microsoft.com/wiki/contents/articles/37431.exporting-azure-app-service-certificates.aspx).

### <a name="how-can-i-resolve-bad-parameter-error-what-are-the-supported-certificate-formats-for-importing-in-key-vault"></a>¿Cómo puedo resolver un "Error de parámetro incorrecto"? ¿Cuáles son los formatos de certificado admitidos para la importación en Key Vault?

Al importar el certificado, debe asegurarse de que la clave se incluye en el propio archivo. Si tiene la clave privada por separado en un formato diferente, tendrá que combinar la clave con el certificado. Algunas entidades de certificación proporcionan certificados en formatos diferentes. Por lo tanto, antes de importar el certificado, asegúrese de que está en formato .pem o. pfx y que la clave usada sea RSA o ECC. Consulte esta información para revisar los [requisitos del certificado](https://docs.microsoft.com/azure/key-vault/certificates/certificate-scenarios#formats-of-import-we-support) y los [requisitos de clave de certificado](https://docs.microsoft.com/azure/key-vault/keys/about-keys#cryptographic-protection).

### <a name="error-when-importing-certificate-via-portal-something-went-wrong-how-can-i-investigate-further"></a>Error al importar el certificado mediante portal: "Algo salió mal". ¿Cómo puedo investigar más?
    
Para leer un error más descriptivo, importe el archivo de certificado mediante la [CLI de Azure](https://docs.microsoft.com/cli/azure/keyvault/certificate?view=azure-cli-latest#az-keyvault-certificate-import) o [PowerShell](https://docs.microsoft.com/powershell/module/azurerm.keyvault/import-azurekeyvaultcertificate?view=azurermps-6.13.0).

### <a name="how-can-i-resolve-error-type-access-denied-or-user-is-unauthorized-to-import-certificate"></a>¿Cómo se resuelve el "Tipo de error: Acceso denegado o usuario no autorizado para importar el certificado"?
    
Esta operación requiere el permiso certificados/importar. Vaya a la ubicación en la que se encuentra la instancia de Key Vault, deberá conceder los permisos adecuados al usuario en las directivas de acceso. Vaya a Key Vault > Directivas de acceso > Agregar directiva de acceso > Seleccionar los permisos de certificado (o como quiera los permisos) > Entidad de seguridad > Buscar y, después, agregue el correo electrónico del usuario. [Más información sobre las directivas de acceso relacionadas con certificados](https://docs.microsoft.com/azure/key-vault/certificates/about-certificates#certificate-access-control)


### <a name="how-can-i-resolve-error-type-conflict-when-creating-a-certificate"></a>¿Cómo se resuelve el "Tipo de error: Conflicto al crear un certificado"?
    
El nombre del certificado debe ser único. Es posible que el certificado con el mismo nombre esté en estado de eliminación temporal; además, según la [composición de un certificado](https://docs.microsoft.com/azure/key-vault/certificates/about-certificates#composition-of-a-certificate) en Azure Key Vault, si hay otra clave o secreto en la instancia de Key Vault con el mismo nombre que está intentando especificar para el certificado, se producirá un error y deberá quitar aquella clave o secreto, o usar un nombre diferente para el certificado. [Ver el certificado eliminado](https://docs.microsoft.com/rest/api/keyvault/getdeletedcertificate/getdeletedcertificate).

### <a name="why-am-i-getting-the-error-type-char-length-is-too-long"></a>¿Por qué se muestra el mensaje "Tipo de error: longitud de char demasiado larga"?
    
* La longitud del nombre del sujeto del certificado tiene un límite de caracteres de 200 char.
* La longitud de la contraseña del certificado tiene un límite de caracteres de 200 char.

### <a name="can-i-import-an-expired-certificate-in-azure-key-vault"></a>¿Puedo importar un certificado expirado en Azure Key Vault?
    
No, los certificados PFX expirados no se importarán a Azure Key Vault.

### <a name="how-can-i-convert-my-certificate-to-proper-format"></a>¿Cómo puedo convertir mi certificado al formato adecuado?

Puede pedir a la entidad de certificación que proporcione el certificado en el formato necesario. También hay herramientas de terceros que pueden ayudarle a convertirlo al formato correcto; sin embargo, Microsoft no podrá darle más información sobre cómo pasar el certificado al formato deseado.

### <a name="can-i-import-certificates-from-non-partner-cas"></a>¿Puedo importar certificados de entidades de certificación no asociadas?
Sí, puede importar certificados de cualquier CA, pero Key Vault no podrá renovarlos automáticamente. Podrá configurar notificaciones por correo electrónico para recibir notificaciones sobre la expiración de los certificados.

### <a name="if-i-import-a-certificate-from-a-partner-ca-will-the-auto-renew-feature-still-work"></a>Si se importa un certificado de una CA asociada, ¿seguirá funcionando la característica de renovación automática?
Sí, debe asegurarse de especificar la rotación automática en la directiva de emisión del certificado una vez lo haya cargado. Además, los cambios se verán reflejados hasta el siguiente ciclo o versión de certificado.

### <a name="unable-to-see-the-app-service-certificate-imported-to-key-vault"></a>¿No puede ver la instancia de App Service Certificate importada a Key Vault? 
Si el certificado se importó correctamente, consulte en la hoja de secretos.


## <a name="next-steps"></a>Pasos siguientes

- [Certificados de Azure Key Vault](/azure/key-vault/certificates/about-certificates)
