---
title: Guía del desarrollador de Azure Key Vault
description: Los desarrolladores pueden usar Azure Key Vault para administrar las claves criptográficas en el entorno de Microsoft Azure.
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 03/11/2020
ms.author: mbaldwin
ms.openlocfilehash: e6ee8ce065361ac27bba0e80349eb5e1d1877526
ms.sourcegitcommit: 03662d76a816e98cfc85462cbe9705f6890ed638
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90532297"
---
# <a name="azure-key-vault-developers-guide"></a>Guía del desarrollador de Azure Key Vault

Key Vault le permite acceder de forma segura a información confidencial desde sus aplicaciones:

- Las claves y los secretos están protegidos sin tener que escribir el código manualmente, y puede usarlos fácilmente en sus aplicaciones.
- Sus clientes pueden poseer y administrar sus propias claves para que pueda centrarse en proporcionar las características de software fundamentales. De este modo, las aplicaciones no serán responsables de las claves y secretos del inquilino de sus clientes.
- La aplicación puede usar claves para firma y cifrado y aun así que la administración de claves sea externa a la aplicación, lo que le permite ser adecuada como aplicación distribuida geográficamente.
- Administre certificados de Key Vault. Para obtener más información, consulte [Certificados](../certificates/about-certificates.md).

Para obtener más información sobre Azure Key Vault, consulte [¿Qué es Key Vault?](overview.md).

## <a name="public-previews"></a>Versiones preliminares públicas

Periódicamente se publica una versión preliminar pública de una nueva característica de Key Vault. Pruébelas y díganos su opinión a través de la dirección de correo electrónico de comentarios azurekeyvault@microsoft.com.

## <a name="creating-and-managing-key-vaults"></a>Creación y administración de almacenes de claves

Azure Key Vault proporciona una manera de almacenar de forma segura las credenciales y otras claves y secretos, pero el código tiene que autenticarse en Key Vault para recuperarlos. Las identidades administradas par recursos de Azure facilita la resolución de este problema, al ofrecer a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory (Azure AD). Puede usar esta identidad para autenticar cualquier servicio que admita la autenticación de Azure AD, incluido Key Vault, sin necesidad de tener credenciales en el código. 

Para más información sobre las identidades administradas para recursos de Azure, consulte la [introducción a las identidades administradas](../../active-directory/managed-identities-azure-resources/overview.md). Para obtener más información sobre cómo trabajar con Azure AD, consulte [Integración de aplicaciones con Azure Active Directory](../../active-directory/develop/active-directory-integrating-applications.md).

Antes de trabajar con claves, secretos o certificados en el almacén de claves, podrá crear y administrar su almacén de claves a través de la CLI, PowerShell, plantillas de Resource Manager o REST, tal y como se describe en los siguientes artículos:

- [Creación y administración de almacenes claves con la CLI](quick-create-cli.md)
- [Creación y administración de almacenes claves con PowerShell](quick-create-powershell.md)
- [Creación y administración de almacenes claves con Azure Portal](quick-create-portal.md)
- [Creación y administración de almacenes de claves con REST](/rest/api/keyvault/vaults/createorupdate)

### <a name="set-and-retrieve-secrets"></a>Establecimiento y recuperación de secretos

- [Establecimiento y recuperación de un secreto con la CLI](../secrets/quick-create-cli.md)
- [Establecimiento y recuperación de un secreto con PowerShell](../secrets/quick-create-powershell.md)
- [Establecimiento y recuperación de un secreto con Azure Portal](../secrets/quick-create-portal.md)
- [Operaciones secretas con REST](/rest/api/keyvault/#secret-operations)
- [Establecimiento y recuperación de un secreto con Python](../secrets/quick-create-python.md)
- [Establecimiento y recuperación de un secreto con Java](../secrets/quick-create-java.md)
- [Establecimiento y recuperación de un secreto con Node.js](../secrets/quick-create-node.md)
- [Establecimiento y recuperación de un secreto con .NET (SDK v4)](../secrets/quick-create-net.md)
- [Creación de un almacén de claves e incorporación de un secreto mediante una plantilla de Azure Resource Manager](../secrets/quick-create-template.md)

### <a name="set-and-retrieve-keys"></a>Establecimiento y recuperación de claves

- [Establecimiento y recuperación de una clave con la CLI](../keys/quick-create-cli.md)
- [Establecimiento y recuperación de una clave con PowerShell](../keys/quick-create-powershell.md)
- [Establecimiento y recuperación de una clave con Azure Portal](../keys/quick-create-portal.md)
- [Operaciones con claves con REST](/rest/api/keyvault/#key-operations)
- [Establecimiento y recuperación de una clave con Python](../secrets/quick-create-python.md)

### <a name="set-and-retrieve-certificates"></a>Establecimiento y recuperación de certificados
- [Establecimiento y recuperación de un certificado con la CLI](../certificates/quick-create-cli.md)
- [Establecimiento y recuperación de un certificado con PowerShell](../certificates/quick-create-powershell.md)
- [Establecimiento y recuperación de un certificado con Azure Portal](../certificates/quick-create-portal.md)
- [Certificación de operaciones con REST](/rest/api/keyvault/#certificate-operations)
- [Establecimiento y recuperación de un certificado con Python](../certificates/quick-create-python.md)

## <a name="coding-with-key-vault"></a>Codificación con Key Vault

El sistema de administración de Key Vault para los programadores está compuesto por varias interfaces. Esta sección contiene vínculos a todos los lenguajes, así como algunos ejemplos de código. 

### <a name="supported-programming-and-scripting-languages"></a>Lenguajes de programación y de scripting admitidos

#### <a name="rest"></a>REST

Mediante la interfaz REST se puede acceder a todos los recursos de Key Vault: almacenes, claves, secretos, etc. 

[Referencia de la API de REST de Key Vault](/rest/api/keyvault/).

#### <a name="net"></a>.NET

[Referencia de API de .NET para Key Vault](/dotnet/api/overview/azure/key-vault?view=azure-dotnet).

#### <a name="java"></a>Java

[SDK de Java para Key Vault](/java/api/overview/azure/keyvault)

#### <a name="nodejs"></a>Node.js

En Node.js, la API de administración y la API de objetos de Key Vault son independientes. El siguiente artículo de introducción proporciona acceso a ambos. 

[Módulos de Azure Key Vault para Node.js](https://docs.microsoft.com/javascript/api/overview/azure/key-vault-index?view=azure-node-latest)

#### <a name="python"></a>Python

[Bibliotecas de Azure Key Vault para Python](https://docs.microsoft.com/python/api/overview/azure/key-vault-index?view=azure-python)

#### <a name="azure-cli"></a>Azure CLI

[CLI de Azure para Key Vault](/cli/azure/keyvault?view=azure-cli-latest)

#### <a name="azure-powershell"></a>Azure PowerShell 

[Azure PowerShell para Key Vault](/powershell/module/az.keyvault/?view=azps-3.6.1#key_vault)

### <a name="code-examples"></a>Ejemplos de código

Para obtener ejemplos completos de cómo usar Key Vault con las aplicaciones, vea:

- [Ejemplos de código de Azure Key Vault](https://azure.microsoft.com/resources/samples/?service=key-vault). Ejemplos de código para Azure Key Vault. 

## <a name="how-tos"></a>Procedimientos

Los artículos y escenarios siguientes proporcionan instrucciones específicas de tarea sobre cómo trabajar con Azure Key Vault:

- [Cambio del identificador de inquilino de Key Vault después de mover la suscripción](move-subscription.md). Al mover su suscripción de Azure del inquilino A al inquilino B, los almacenes de claves existentes se vuelven inaccesibles para los principales (usuarios y aplicaciones) en el inquilino B. Para corregir este problema, utilice esta guía.
- [Acceso a Key Vault detrás de un firewall](access-behind-firewall.md). Para acceder a un almacén de claves, es preciso que la aplicación cliente de Key Vault pueda acceder a varios puntos de conexión para diversas funcionalidades.
- [Generación y transferencia de claves protegidas con HSM para Azure Key Vault](../keys/hsm-protected-keys.md). Esto le ayudará a planear, generar y transferir sus propias claves protegidas con HSM para usarlas con Azure Key Vault.
- [Paso de valores seguros (como contraseñas) durante la implementación](../../azure-resource-manager/templates/key-vault-parameter.md). Si necesita pasar un valor seguro (como una contraseña) como un parámetro durante la implementación, puede almacenar ese valor como un secreto en Azure Key Vault y hacer referencia al valor en otras plantillas de Resource Manager.
- [Uso de Key Vault para la administración extensible de claves con SQL Server](https://msdn.microsoft.com/library/dn198405.aspx). El Conector de SQL Server para Azure Key Vault permite que SQL Server y SQL en una VM use el servicio Azure Key Vault como un proveedor de administración extensible de claves (EKM) con el fin de proteger sus claves de cifrado para el vínculo de aplicaciones; cifrado de datos transparente, cifrado de copia de seguridad y cifrado de nivel de columna.
- [Implementación de certificados en máquinas virtuales desde Key Vault](https://blogs.technet.microsoft.com/kv/2015/07/14/deploy-certificates-to-vms-from-customer-managed-key-vault/): una aplicación de nube que se ejecute en una máquina virtual de Azure necesita un certificado. Sepa cómo añadirlo a la máquina virtual hoy mismo.
- [Deploying Azure Web App Certificate through Key Vault]( https://blogs.msdn.microsoft.com/appserviceteam/2016/05/24/deploying-azure-web-app-certificate-through-key-vault/) (Implementación de certificado Azure Web App a través de Key Vault) proporciona instrucciones paso a paso para implementar certificados almacenados en Key Vault como parte de la oferta de [App Service Certificate](https://azure.microsoft.com/blog/internals-of-app-service-certificate/).
- Asignación de una directiva de acceso ([CLI](assign-access-policy-cli.md) | [PowerShell](assign-access-policy-powershell.md) | [Azure Portal](assign-access-policy-portal.md)). Key Vault admite hasta 1024 entradas de directivas de acceso. Para permanecer dentro de este límite con los usuarios, cree grupos de seguridad de Azure Active Directory, agregue todas las entidades de servicio asociadas al grupo concreto y, después, conceda al grupo acceso a Key Vault.
- Para obtener más instrucciones específicas sobre las tareas de integración y el uso de los almacenes de claves con Azure, consulte los [ejemplos de plantillas de Azure Resource Manager de Ryan Jones para Key Vault](https://github.com/rjmax/ArmExamples/tree/master/keyvaultexamples).
- [Uso de la eliminación temporal de Key Vault con la CLI](soft-delete-cli.md) le guiará por el uso y ciclo de vida de un almacén de claves y de varios objetos de almacén de claves con la eliminación temporal habilitada.
- [Uso de la eliminación temporal de Key Vault con PowerShell](soft-delete-powershell.md) le guiará por el uso y ciclo de vida de un almacén de claves y de varios objetos de almacén de claves con la eliminación temporal habilitada.

## <a name="integrated-with-key-vault"></a>Integración con Key Vault

En estos artículos se describen otros escenarios y servicios que usan Key Vault o se integran en este servicio.

- [Azure Disk Encryption](../../security/fundamentals/encryption-overview.md) aprovecha la característica [BitLocker](https://technet.microsoft.com/library/cc732774.aspx) estándar del sector de Windows y la característica [DM-Crypt](https://en.wikipedia.org/wiki/Dm-crypt) de Linux para ofrecer cifrado de volumen para el sistema operativo y los discos de datos. La solución se integra con Azure Key Vault para ayudarle a controlar y administrar las claves y secretos de cifrado de disco en su suscripción del almacén de claves, al mismo tiempo que garantiza que todos los datos de los discos de las máquinas virtuales se cifran en reposo en el almacenamiento de Azure.
- [Azure Data Lake Store](../../data-lake-store/data-lake-store-get-started-portal.md) proporciona la opción para el cifrado de datos que se almacenan en la cuenta. Para la administración de claves, Data Lake Store ofrece dos modos de administrar las claves de cifrado maestras (MEK) que son necesarias para descifrar los datos que se almacenan en Data Lake Store. Se puede dejar que Data Lake Store administre las MEK en su lugar o decidir conservar la propiedad de estas mediante su cuenta de Azure Key Vault. Puede especificar el modo de administración de claves durante la creación de una cuenta de Data Lake Store.
- [Azure Information Protection](/azure/information-protection/plan-implement-tenant-key) le permite administrar su propia clave de inquilino. Por ejemplo, en lugar de que Microsoft administre su clave de inquilino (la opción predeterminada), puede administrar su propia clave de inquilino para cumplir con las normas específicas que se aplican a su organización. La administración de su propia clave de inquilino también se conoce como "aportar su propia clave", o BYOK.

## <a name="key-vault-overviews-and-concepts"></a>Conceptos y datos globales de Key Vault

- [Comportamiento de eliminación temporal de Key Vault](soft-delete-overview.md) describe una característica que permite la recuperación de objetos eliminados tanto si dicha eliminación ha sido accidental como intencionada.
- [Limitación del cliente de Key Vault](overview-throttling.md) proporciona orientación sobre los conceptos básicos de la limitación y ofrece un enfoque para la aplicación.
- [Espacios de seguridad de Key Vault](overview-security-worlds.md) describe las relaciones entre regiones y zonas de seguridad.

## <a name="social"></a>Redes sociales

- [Blog de Key Vault](https://aka.ms/kvblog)
- [Foro sobre Key Vault](https://aka.ms/kvforum)

## <a name="supporting-libraries"></a>Bibliotecas compatibles

- [Microsoft Azure Key Vault Core Library](https://www.nuget.org/packages/Microsoft.Azure.KeyVault.Core) (biblioteca principal Microsoft Azure Key Vault) proporciona las interfaces **IKey** e **IKeyResolver** para localizar las claves de los identificadores y realizar operaciones con ellas.
- [Las extensiones de Microsoft Azure Key Vault](https://www.nuget.org/packages/Microsoft.Azure.KeyVault.Extensions) proporcionan capacidades ampliadas para Azure Key Vault.
