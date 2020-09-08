---
title: Integración de Key Vault con la entidad de certificación DigiCert
description: Cómo integrar Key Vault con la entidad de certificación de DigiCert
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: how-to
ms.date: 06/02/2020
ms.author: sebansal
ms.openlocfilehash: 01383acad9f221e376f814ecf99794eb0431d0cd
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88588932"
---
# <a name="integrating-key-vault-with-digicert-certificate-authority"></a>Integración de Key Vault con la entidad de certificación DigiCert

Azure Key Vault permite aprovisionar, administrar e implementar fácilmente certificados digitales para una red y habilitar las comunicaciones seguras para las aplicaciones. Un certificado digital es una credencial electrónica para establecer la prueba de identidad en una transacción electrónica. 

Los usuarios de Azure Key Vault pueden generar certificados DigiCert directamente desde sus instancias de Key Vault. Key Vault garantizará la administración del ciclo de vida de los certificados de un extremo a otro para los certificados emitidos por DigiCert mediante la asociación de confianza de Key Vault con la entidad de certificación de DigiCert.

Para más información general acerca de los certificados, consulte [Certificados de Azure Key Vault](/azure/key-vault/certificates/about-certificates).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía, necesitará los recursos siguientes:
* Un almacén de claves. Puede usar un almacén de claves existente o crear uno nuevo siguiendo los pasos de uno de estos artículos de inicio rápido:
   - [Creación de un almacén de claves con la CLI de Azure](../secrets/quick-create-cli.md)
   - [Creación de un almacén de claves con Azure PowerShell](../secrets/quick-create-powershell.md)
   - [Creación de un almacén de claves con Azure Portal](../secrets/quick-create-portal.md)
*   Tiene que activar la cuenta CertCentral de DigiCert. [Regístrese](https://www.digicert.com/account/signup/) para la cuenta CertCentral.
*   Permisos de nivel de administrador en sus cuentas.


### <a name="before-you-begin"></a>Antes de empezar

Asegúrese de tener a mano la siguiente información de la cuenta CertCentral de DigiCert:
-   Identificador de la cuenta CertCentral
-   Identificador de la organización
-   Clave de API

## <a name="adding-certificate-authority-in-key-vault"></a>Incorporación de una entidad de certificación en Key Vault 
Después de recopilar información sobre la cuenta CertCentral de DigiCert, ahora puede Agregar DigiCert a la lista de entidades de certificación en el almacén de claves.

### <a name="azure-portal"></a>Azure portal

1.  Para agregar la entidad de certificación DigiCert, vaya al almacén de claves en el que desea agregar DigiCert. 
2.  En las páginas de propiedades de Key Vault, seleccione **Certificados**.
3.  Seleccione la pestaña **Entidades de certificación**. ![Propiedades del certificado](../media/certificates/how-to-integrate-certificate-authority/select-certificate-authorities.png)
4.  Seleccione la opción **Agregar**.
 ![Propiedades del certificado](../media/certificates/how-to-integrate-certificate-authority/add-certificate-authority.png)
5.  En la pantalla **Creación de una entidad de certificación**, elija los siguientes valores:
    -   **Name**: Agregue un nombre de emisor identificable. Example DigicertCA
    -   **Proveedor**: Seleccione DigiCert en el menú.
    -   **Id. de cuenta**: Escriba el identificador de la cuenta CertCentral de DigiCert.
    -   **Contraseña de cuenta**: Escriba la clave de API que generó en la cuenta CertCentral de DigiCert.
    -   **Id. de la organización**: Escriba OrgID recopilado de la cuenta CertCentral de DigiCert. 
    -   Haga clic en **Crear**.
   
6.  Verá que DigicertCA se ha agregado ahora a la lista de entidades de certificación.


### <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell se usa para crear y administrar recursos de Azure mediante comandos o scripts. En Azure se hospeda Azure Cloud Shell, un entorno de shell interactivo que puede utilizar mediante Azure Portal en el explorador.

Si decide instalar y usar PowerShell de forma local, en este tutorial necesitará la versión 1.0.0 del módulo de Azure PowerShell, o cualquier versión posterior. Escriba `$PSVersionTable.PSVersion` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Login-AzAccount` para crear una conexión con Azure.

```azurepowershell-interactive
Login-AzAccount
```

1.  Cree un **grupo de recursos**.

Cree un grupo de recursos de Azure con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 

```azurepowershell-interactive
New-AzResourceGroup -Name ContosoResourceGroup -Location EastUS
```

2. Cree un **almacén de claves**.

Tiene que usar un nombre único para el almacén de claves. Aquí "Contoso-Vaultname" es el nombre del almacén de claves a lo largo de esta guía.

- **Nombre del almacén**: Contoso-Vaultname.
- **Nombre del grupo de recursos**: ContosoResourceGroup.
- **Ubicación**: EastUS.

```azurepowershell-interactive
New-AzKeyVault -Name 'Contoso-Vaultname' -ResourceGroupName 'ContosoResourceGroup' -Location 'EastUS'
```

3. Defina las variables para la información recopilada de la cuenta CertCentral de DigiCert.

- Defina la variable **ID. de cuenta**.
- Defina la variable **Id. org.**
- Defina la variable **Clave de API**.
- Defina la variable **Nombre del emisor**.

```azurepowershell-interactive
$accountId = "myDigiCertCertCentralAccountID"
$org = New-AzKeyVaultCertificateOrganizationDetails -Id OrganizationIDfromDigiCertAccount
$secureApiKey = ConvertTo-SecureString DigiCertCertCentralAPIKey -AsPlainText –Force
$issuerName = "DigiCertCA"
```

4. Establezca el **Emisor**. Esto agregará a DigiCert como entidad de certificación en el almacén de claves.
```azurepowershell-interactive
Set-AzureKeyVaultCertificateIssuer -VaultName $vaultName -IssuerName $issuerName -IssuerProvider DigiCert -AccountId $accountId -ApiKey $secureApiKey -OrganizationDetails $org
```

5. **Establecimiento de la directiva para el certificado y emisión del certificado** desde DigiCert directamente dentro de Key Vault.

```azurepowershell-interactive
$Policy = New-AzKeyVaultCertificatePolicy -SecretContentType "application/x-pkcs12" -SubjectName "CN=contoso.com" -IssuerName DigiCertCA -ValidityInMonths 12 -RenewAtNumberOfDaysBeforeExpiry 60
Add-AzKeyVaultCertificate -VaultName "Contoso-Vaultname" -Name "ExampleCertificate" -CertificatePolicy $Policy
```

La entidad de certificación DigiCert ha emitido correctamente el certificado dentro de la instancia de Key Vault especificada mediante esta integración.

## <a name="troubleshoot"></a>Solución de problemas

Si el certificado emitido se encuentra en el estado "deshabilitado" en Azure Portal, continúe para ver la **Operación de certificados** y revisar el mensaje de error de DigiCert para ese certificado.

 ![Propiedades del certificado](../media/certificates/how-to-integrate-certificate-authority/certificate-operation-select.png)

Para más información, consulte las [operaciones con certificados en la referencia de la API REST de Key Vault](/rest/api/keyvault). Para obtener información sobre cómo establecer permisos, vea [Almacenes: crear o actualizar](/rest/api/keyvault/vaults/createorupdate) y [Almacenes: actualizar directiva de acceso](/rest/api/keyvault/vaults/updateaccesspolicy).

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

- ¿Puedo generar un certificado comodín de DigiCert con KeyVault? 
   Sí. Dependería de cómo haya configurado la cuenta de DigiCert.
- Si vamos a crear un certificado EV, ¿cómo lo especificamos? 
   Al crear un certificado, haga clic en Configuración de directiva avanzada y, a continuación, especifique el tipo de certificado. Los valores admitidos son: OV-SSL, EV-SSL
- ¿Hay un retraso de tiempo en la creación de un certificado de DigiCert mediante la integración y la adquisición de un certificado con DigiCert directamente?
   No. Al crear un certificado, el proceso de comprobación es lo que puede tardar en realizarse y esa comprobación depende del proceso DigiCert siguiente.


## <a name="next-steps"></a>Pasos siguientes

- [Autenticación, solicitudes y respuestas](../general/authentication-requests-and-responses.md)
- [Guía del desarrollador de Key Vault](../general/developers-guide.md)
