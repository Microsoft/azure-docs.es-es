---
title: 'Generación y transferencia de claves protegidas con HSM para Azure Key Vault: Azure Key Vault | Microsoft Docs'
description: Use este artículo para ayudarle a planear, generar y, a continuación, transfiera sus propias claves protegidas con HSM para utilizar con Azure Key Vault. También se denomina BYOK o "traiga su propia clave".
services: key-vault
author: barclayn
manager: barbkess
tags: azure-resource-manager
ms.service: key-vault
ms.topic: conceptual
ms.date: 02/12/2019
ms.author: barclayn
ms.openlocfilehash: 1ae94718aa41c58f4d5e397942492ad8ed643ae3
ms.sourcegitcommit: 9e8dfa1169a55c3c8af93a6c5f4e0dace4de48b2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65556197"
---
# <a name="how-to-generate-and-transfer-hsm-protected-keys-for-azure-key-vault"></a>Generación y transferencia de claves protegidas con HSM para Azure Key Vault

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para obtener una mayor seguridad, cuando utilice Azure Key Vault, puede importar o generar claves en módulos de seguridad de hardware (HSM) que no se salen nunca del límite de los HSM. Con frecuencia este escenario se conoce también como *Aportar tu propia clave*, o BYOK. Los HSM tienen la validación FIPS 140-2 de nivel 2. Azure Key Vault usa la familia de nCipher nShield de HSM para proteger sus claves.

La información de este tema le ayudará a planear, generar y transferir sus propias claves protegidas con HSM para utilizarlas en Azure Key Vault.

Esta funcionalidad no está disponible para Azure China.

> [!NOTE]
> Para obtener más información sobre Azure Key Vault, consulte [¿Qué es Azure Key Vault?](key-vault-whatis.md)  
> Para ver un tutorial introductorio que incluya la creación de un almacén de claves para claves protegidas con HSM, consulte [¿Qué es Azure Key Vault?](key-vault-overview.md).

Obtenga más información acerca de cómo generar y transferir una clave protegida con HSM a través de Internet:

* Genere la clave desde una estación de trabajo sin conexión, lo que reduce la superficie de ataque.
* La clave se cifra con una clave de intercambio de claves (KEK), que permanece cifrada hasta que se transfiere a los HSM de Azure Key Vault. Solo la versión cifrada de la clave deja la estación de trabajo original.
* El conjunto de herramientas establece las propiedades en su clave de inquilino que enlaza la clave con el espacio de seguridad de Azure Key Vault. Por consiguiente, después de que los HSM de Azure Key Vault reciban y descifren la clave, dichos HSM son los únicos los que puedan usarla. La clave no se puede exportar. Este enlace lo exigen lo HSM nCipher.
* La clave de intercambio de claves (KEK) que se utiliza para cifrar la clave se genera dentro de los HSM de Azure Key Vault y no es exportable. Los HSM exigen que no pueda haber una versión sin cifrar de la KEK fuera de los HSM. Además, el conjunto de herramientas incluye la atestación desde nCipher que la KEK no es exportable y se generó dentro de un HSM genuino que fabricó nCipher.
* El conjunto de herramientas incluye la atestación desde nCipher que también se generó el mundo de seguridad de Azure Key Vault en un HSM genuino fabricado por nCipher. Esta certificación demuestra que Microsoft usa hardware original.
* Microsoft usa KEK independientes y separa los espacios de seguridad de las distintas regiones geográficas. Esta separación garantiza que la clave puede utilizarse únicamente en centros de datos de la región en la que se ha cifrado. Por ejemplo, una clave de un cliente europeo no se puede utilizar en centros de datos de América del Norte o Asia.

## <a name="more-information-about-ncipher-hsms-and-microsoft-services"></a>Más información sobre los HSM nCipher y servicios de Microsoft

nCipher seguridad es un proveedor líder mundial de cifrado de datos y soluciones de seguridad cibernética al servicios financieros, alta tecnología, fabricación, gobierno y sectores tecnológicos. Con una información de gobierno y 40 años trayectoria de protección corporativa, soluciones de cifrado de seguridad nCipher usan cuatro de las cinco mayores compañías de sectores energético y aeroespacial. Sus soluciones también se usan en OTAN 22 países o regiones y proteger más del 80 por ciento de las transacciones de pago en todo el mundo.

Microsoft ha colaborado con nCipher seguridad para mejorar el estado del arte de HSM. Estas mejoras te permiten obtener los beneficios típicos de los servicios hospedados sin tener que renunciar al control de las claves. En concreto, estas mejoras permiten que Microsoft administre los HSM para que no lo tengan que hacer sus usuarios. Al ser un servicio en la nube, Azure Key Vault se escala verticalmente en muy poco tiempo para cubrir los picos de uso de cualquier organización. Al mismo tiempo, la clave está protegida dentro de los HSM de Microsoft: El usuario conserva el control sobre el ciclo de vida de la clave ya que es el que genera la clave y la transfiere a los HSM de Microsoft.

## <a name="implementing-bring-your-own-key-byok-for-azure-key-vault"></a>Implementación del método Aportar tu propia clave (BYOK) en Azure Key Vault

Utilice la siguiente información y procedimientos si va a generar su propia clave protegida con HSM y, a continuación, va a transferirla a Azure Key Vault, el escenario de Aportar tu propia clave (BYOK).

## <a name="prerequisites-for-byok"></a>Requisitos previos de BYOK

En la tabla siguiente puede ver una lista de los requisitos previos del método Aportar tu propia clave (BYOK) para Azure Key Vault.

| Requisito | Más información |
| --- | --- |
| Una suscripción a Azure |Para crear una instancia de Azure Key Vault, se necesita una suscripción a Azure: [Suscríbase ahora para disfrutar de una prueba gratis](https://azure.microsoft.com/pricing/free-trial/). |
| Nivel de servicio Premium de Azure Key Vault, que admita claves protegidas con HSM |Para obtener más información sobre los niveles de servicio y las funcionalidades de Azure Key Vault, consulte el sitio web [Precios de Key Vault](https://azure.microsoft.com/pricing/details/key-vault/) . |
| nCipher nShield HSM, tarjetas inteligentes y software de soporte técnico |Debe tener acceso a un módulo de seguridad de Hardware nCipher y conocimiento operacional básico de nCipher nShield HSM. Consulte [nCipher nShield Hardware Security Module](https://www.ncipher.com/products/key-management/cloud-microsoft-azure/how-to-buy) para ver la lista de los modelos compatibles o comprar un HSM si no tiene uno. |
| El siguiente hardware y software:<ol><li>Un x64 sin conexión estación de trabajo con un sistema operativo de Windows mínimo de Windows 7 y nCipher nShield software que tiene al menos la versión 11.50 o superior.<br/><br/>Si esta estación de trabajo ejecuta Windows 7, debe [instalar Microsoft .NET Framework 4.5](https://download.microsoft.com/download/b/a/4/ba4a7e71-2906-4b2d-a0e1-80cf16844f5f/dotnetfx45_full_x86_x64.exe).</li><li>Una estación de trabajo conectada a Internet y con un sistema operativo Windows 7 como mínimo y [Azure PowerShell](/powershell/azure/overview?view=azps-1.2.0) **con al menos la versión 1.1.0** instalada.</li><li>Una unidad USB u otro dispositivo de almacenamiento portátil con al menos 16 MB de espacio libre.</li></ol> |Por seguridad, se recomienda que la primera estación de trabajo no esté conectada a una red. Sin embargo, esta recomendación no es de obligado cumplimiento.<br/><br/>En las instrucciones siguientes, esta estación de trabajo se conoce como la desconectada.</p></blockquote><br/>Además, si la clave de inquilino es para una red de producción, se recomienda usar una segunda estación de trabajo independiente para descargar el conjunto de herramientas y cargar la clave de inquilino. Sin embargo, para la prueba puede usar la misma estación de trabajo que la primera.<br/><br/>En las instrucciones siguientes, la segunda estación de trabajo se conoce como la que está conectada a Internet.</p></blockquote><br/> |

## <a name="generate-and-transfer-your-key-to-azure-key-vault-hsm"></a>Generación y transferencia de una clave a un HSM de Azure Key Vault

Los cinco pasos siguientes sirven para generar y transferir la clave a un HSM de Azure Key Vault:

* [Paso 1: Preparación de la estación de trabajo conectada a Internet](#step-1-prepare-your-internet-connected-workstation)
* [Paso 2: Preparación de la estación de trabajo desconectada](#step-2-prepare-your-disconnected-workstation)
* [Paso 3: Generación de la clave](#step-3-generate-your-key)
* [Paso 4: Preparación de la clave para la transferencia](#step-4-prepare-your-key-for-transfer)
* [Paso 5: Transferencia de la clave a Azure Key Vault](#step-5-transfer-your-key-to-azure-key-vault)

## <a name="step-1-prepare-your-internet-connected-workstation"></a>Paso 1: Preparación de la estación de trabajo conectada a Internet

En este primer paso, realice los siguientes procedimientos en la estación de trabajo conectada a Internet.

### <a name="step-11-install-azure-powershell"></a>Paso 1.1: Instalar Azure PowerShell

Desde la estación de trabajo conectada a Internet, descargue e instale el módulo de Azure PowerShell que incluye los cmdlets para administrar Azure Key Vault. Para ver las instrucciones de instalación, consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/overview).

### <a name="step-12-get-your-azure-subscription-id"></a>Paso 1.2: Obtención del identificador de la suscripción a Azure

Inicie una sesión de Azure PowerShell e inicie sesión en su cuenta de Azure con el siguiente comando:

```Powershell
   Connect-AzAccount
```
En la ventana emergente del explorador, escriba el nombre de usuario y la contraseña de su cuenta de Azure. A continuación, use el comando [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription):

```powershell
   Get-AzSubscription
```
En la salida, busque el identificador de la suscripción que va a utilizar para Azure Key Vault. Lo necesitará más adelante.

No cierre la ventana de Azure PowerShell.

### <a name="step-13-download-the-byok-toolset-for-azure-key-vault"></a>Paso 1.3: Descarga del conjunto de herramientas BYOK para Azure Key Vault

Vaya al Centro de descarga de Microsoft y [descargue el conjunto de herramientas de BYOK para Azure Key Vault](https://www.microsoft.com/download/details.aspx?id=45345) de su región geográfica o instancia de Azure. Utilice la siguiente información para identificar el nombre del paquete para descargar y su hash de paquete SHA-256 correspondiente:

- - -
**Estados Unidos:**

KeyVault-BYOK-Tools-UnitedStates.zip

2E8C00320400430106366A4E8C67B79015524E4EC24A2D3A6DC513CA1823B0D4

- - -
**Europa:**

KeyVault-BYOK-Tools-Europe.zip

9AAA63E2E7F20CF9BB62485868754203721D2F88D300910634A32DFA1FB19E4A

- - -
**Asia:**

KeyVault-BYOK-Tools-AsiaPacific.zip

4BC14059BF0FEC562CA927AF621DF665328F8A13616F44C977388EC7121EF6B5

- - -
**América Latina:**

KeyVault-BYOK-Tools-LatinAmerica.zip

E7DFAFF579AFE1B9732C30D6FD80C4D03756642F25A538922DD1B01A4FACB619

- - -
**Japón:**

KeyVault-BYOK-Tools-Japan.zip

3933C13CC6DC06651295ADC482B027AF923A76F1F6BF98B4D4B8E94632DEC7DF

- - -
**Corea:**

KeyVault-BYOK-Tools-Korea.zip

71AB6BCFE06950097C8C18D532A9184BEF52A74BB944B8610DDDA05344ED136F

- - -
**Sudáfrica:**

KeyVault-BYOK-Tools-SouthAfrica.zip

C41060C5C0170AAAAD896DA732E31433D14CB9FC83AC3C67766F46D98620784A

- - -
**UAE:**

KeyVault-BYOK-Tools-UAE.zip

FADE80210B06962AA0913EA411DAB977929248C65F365FD953BB9F241D5FC0D3

- - -
**Australia:**

KeyVault-BYOK-Tools-Australia.zip

CD0FB7365053DEF8C35116D7C92D203C64A3D3EE2452A025223EEB166901C40A

- - -
[**Azure Government:**](https://azure.microsoft.com/features/gov/)

KeyVault-BYOK-Tools-USGovCloud.zip

F8DB2FC914A7360650922391D9AA79FF030FD3048B5795EC83ADC59DB018621A

- - -
**DoD del Gobierno de Estados Unidos:**

KeyVault-BYOK-Tools-USGovernmentDoD.zip

A79DD8C6DFFF1B00B91D1812280207A205442B3DDF861B79B8B991BB55C35263

- - -
**Canadá:**

KeyVault-BYOK-Tools-Canada.zip

61BE1A1F80AC79912A42DEBBCC42CF87C88C2CE249E271934630885799717C7B

- - -
**Alemania:**

KeyVault-BYOK-Tools-Germany.zip

5385E615880AAFC02AFD9841F7BADD025D7EE819894AA29ED3C71C3F844C45D6

- - -
**India:**

KeyVault-BYOK-Tools-India.zip

49EDCEB3091CF1DF7B156D5B495A4ADE1CFBA77641134F61B0E0940121C436C8

- - -
**Francia:**

KeyVault-BYOK-Tools-France.zip

5C9D1F3E4125B0C09E9F60897C9AE3A8B4CB0E7D13A14F3EDBD280128F8FE7DF

- - -
**Reino Unido:**

KeyVault-BYOK-Tools-UnitedKingdom.zip

432746BD0D3176B708672CCFF19D6144FCAA9E5EB29BB056489D3782B3B80849

- - -

Para validar la integridad del conjunto de herramientas BYOK que descargó, use el cmdlet [Get-FileHash](https://technet.microsoft.com/library/dn520872.aspx) en la sesión de Azure PowerShell.

   ```powershell
   Get-FileHash KeyVault-BYOK-Tools-*.zip
   ```

El conjunto de herramientas incluye:

* Un paquete de clave de intercambio de claves (KEK) cuyo nombre comienza por **BYOK-KEK-pkg-.**
* Un paquete de espacio de seguridad cuyo nombre comienza por **BYOK-SecurityWorld-pkg-.**
* Un script Python denominado **verifykeypackage.py.**
* Un archivo ejecutable de línea de comandos denominado **KeyTransferRemote.exe** y asociado a las DLL.
* Un paquete redistribuible de Visual C++ denominado **vcredist_x64.exe**.

Copie el paquete en una unidad USB u otro dispositivo de almacenamiento portátil.

## <a name="step-2-prepare-your-disconnected-workstation"></a>Paso 2: Preparación de la estación de trabajo desconectada

En este segundo paso, realice los siguientes procedimientos en la estación de trabajo que no está conectada a una red (Internet o la red interna).

### <a name="step-21-prepare-the-disconnected-workstation-with-ncipher-nshield-hsm"></a>Paso 2.1: Preparar la estación de trabajo desconectada con HSM del nCipher nShield

Instalar el software de soporte nCipher en un equipo de Windows y, a continuación, adjunte un HSM de nCipher nShield a ese equipo.

Asegúrese de que la ruta de acceso de las herramientas nCipher (**%nfast_home%\bin**). Por ejemplo, escriba lo siguiente:

  ```cmd
  set PATH=%PATH%;"%nfast_home%\bin"
  ```

Para obtener más información, consulte la Guía de usuario incluida con el nShield HSM.

### <a name="step-22-install-the-byok-toolset-on-the-disconnected-workstation"></a>Paso 2.2: Instalación del conjunto de herramientas BYOK en la estación de trabajo desconectada

Copie el paquete del conjunto de herramientas BYOK de la unidad USB, u otro dispositivo de almacenamiento portátil, y, a continuación, haga lo siguiente:

1. Extraiga los archivos del paquete descargado en cualquier carpeta.
2. En esa carpeta, ejecute vcredist_x64.exe.
3. Siga las instrucciones para instalar los componentes de tiempo de ejecución de Visual C++ para Visual Studio 2013.

## <a name="step-3-generate-your-key"></a>Paso 3: Generación de la clave

En el tercer paso, realice los siguientes procedimientos en la estación de trabajo desconectada. Para completar este paso, HSM debe estar en modo de inicialización. 

### <a name="step-31-change-the-hsm-mode-to-i"></a>Paso 3.1: Cambio del modo HSM a "I"

Si usas nCipher nShield Edge, para cambiar el modo: 1. Use el botón de modo para resaltar el modo requerido. 2. Después de un momento, mantenga presionado el botón Borrar durante un par de segundos. Si el modo cambia, el LED del nuevo modo deja de parpadear y permanece encendido. El LED del estado puede parpadear irregularmente durante algunos segundos y, luego, parpadea regularmente cuando el dispositivo está listo. En caso contrario, el dispositivo permanece en el modo actual, con el modo correspondiente de LED encendido.

### <a name="step-32-create-a-security-world"></a>Paso 3.2: Creación de un espacio de seguridad

Inicie un símbolo del sistema y ejecute el programa de nuevo mundo nCipher.

   ```cmd
    new-world.exe --initialize --cipher-suite=DLf3072s256mRijndael --module=1 --acs-quorum=2/3
   ```

Este programa crea un archivo de **espacio de seguridad** en %NFAST_KMDATA%\local\world, que corresponde a la carpeta C:\ProgramData\nCipher\Key Management Data\local. Puede usar valores diferentes para el quórum, pero en nuestro ejemplo, se le pedirá que escriba tres tarjetas en blanco y PIN para cada una de ellos. A continuación, dos tarjetas cualesquiera le proporcionarán acceso total al espacio de seguridad. Estas tarjetas pasan a ser el **Conjunto de tarjetas de administrador** del nuevo espacio de seguridad.

> [!NOTE]
> Si su dispositivo HSM no admite el conjunto de cifrado más reciente DLf3072s256mRijndael, puede reemplazar --cipher-suite= DLf3072s256mRijndael por --cipher-suite=DLf1024s160mRijndael.

A continuación, haga lo siguiente:

* Realice una copia de seguridad del archivo del espacio. Proteja dicho archivo, las tarjetas de administrador y sus PIN, y asegúrese de que nadie tiene acceso a más de una tarjeta.

### <a name="step-33-change-the-hsm-mode-to-o"></a>Paso 3.3: Cambio del modo HSM a "O"

Si usas nCipher nShield Edge, para cambiar el modo: 1. Use el botón de modo para resaltar el modo requerido. 2. Después de un momento, mantenga presionado el botón Borrar durante un par de segundos. Si el modo cambia, el LED del nuevo modo deja de parpadear y permanece encendido. El LED del estado puede parpadear irregularmente durante algunos segundos y, luego, parpadea regularmente cuando el dispositivo está listo. En caso contrario, el dispositivo permanece en el modo actual, con el modo correspondiente de LED encendido.

### <a name="step-34-validate-the-downloaded-package"></a>Paso 3.4: Validación del paquete descargado

Este paso es opcional, pero se recomienda realizarlo para poder validar estos elementos:

* Se ha generado la clave de intercambio que se incluye en el conjunto de herramientas desde un HSM de original nCipher nShield.
* El hash del mundo de seguridad que se incluye en el conjunto de herramientas se ha generado en un HSM de original nCipher nShield.
* La clave de intercambio de claves no es exportable.

> [!NOTE]
> Para validar el paquete descargado, el HSM debe estar conectado, encendido y debe tener un espacio de seguridad (como el que acaba de crear).

Para validar el paquete descargado:

1. Ejecute el script verifykeypackage.py, para lo que, en función de su región geográfica o instancia de Azure, debe escribir lo siguiente:

   * Norteamérica:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-NA-1 -w BYOK-SecurityWorld-pkg-NA-1
   * Europa:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-EU-1 -w BYOK-SecurityWorld-pkg-EU-1
   * Asia:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-AP-1 -w BYOK-SecurityWorld-pkg-AP-1
   * América Latina:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-LATAM-1 -w BYOK-SecurityWorld-pkg-LATAM-1
   * Japón:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-JPN-1 -w BYOK-SecurityWorld-pkg-JPN-1
   * Para Corea:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-KOREA-1 -w BYOK-SecurityWorld-pkg-KOREA-1
   * Para Sudáfrica:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-SA-1 -w BYOK-SecurityWorld-pkg-SA-1
   * Para eau:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-UAE-1 -w BYOK-SecurityWorld-pkg-UAE-1
   * Para Australia:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-AUS-1 -w BYOK-SecurityWorld-pkg-AUS-1
   * Para [Azure Government](https://azure.microsoft.com/features/gov/), que usa la instancia del gobierno de Estados Unidos de Azure:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-USGOV-1 -w BYOK-SecurityWorld-pkg-USGOV-1
   * Para DoD del Gobierno de Estados Unidos:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-USDOD-1 -w BYOK-SecurityWorld-pkg-USDOD-1
   * Para Canadá:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-CANADA-1 -w BYOK-SecurityWorld-pkg-CANADA-1
   * Para Alemania:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-GERMANY-1 -w BYOK-SecurityWorld-pkg-GERMANY-1
   * Para India:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-INDIA-1 -w BYOK-SecurityWorld-pkg-INDIA-1
   * Para Francia:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-FRANCE-1 -w BYOK-SecurityWorld-pkg-FRANCE-1
   * Para Reino Unido:

         "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-UK-1 -w BYOK-SecurityWorld-pkg-UK-1

     > [!TIP]
     > El software de nCipher nShield incluye python en %NFAST_HOME%\python\bin
     >
     >
2. Confirme que ve lo siguiente, que indica que el resultado de la validación ha sido satisfactorio: **Result: SUCCESS**

Este script valida la cadena del firmante hasta la clave de raíz nShield. El hash de esta clave raíz está insertado en el script y su valor debe ser **59178a47 de508c3f 291277ee 184f46c4 f1d9c639**. También puede confirmar este valor por separado, visite la [sitio Web nCipher](https://www.ncipher.com/products/key-management/cloud-microsoft-azure/validation).

Ya está listo para crear una nueva clave.

### <a name="step-35-create-a-new-key"></a>Paso 3.5: Crear una nueva clave

Genere una clave mediante el nShield nCipher **generatekey** programa.

Ejecute el comando siguiente para generar la clave:

    generatekey --generate simple type=RSA size=2048 protect=module ident=contosokey plainname=contosokey nvram=no pubexp=

Cuando ejecute este comando, siga estas instrucciones:

* El parámetro *protect* debe establecerse en el valor **module**, como se muestra. Esto crea una clave protegida por el módulo. El conjunto de herramientas BYOK no es compatible con claves protegidas con OCS.
* Reemplace el valor de *contosokey* en **ident** y **plainname** por cualquier valor de cadena. Para minimizar gastos administrativos y reducir el riesgo de errores, se recomienda utilizar el mismo valor en ambos. El valor **ident** debe contener solo números, guiones y minúsculas.
* En este ejemplo, pubexp se deja en blanco (valor predeterminado), pero puede especificar valores concretos. Para obtener más información, consulte el [nCipher documentación.](https://www.ncipher.com/resources/solution-briefs/protect-sensitive-data-rest-and-use-across-premises-and-azure-based)

Este comando crea un archivo de clave con tokens en la carpeta %NFAST_KMDATA%\local cuyo nombre comienza por **key_simple_**, seguido del valor **ident** que se especificó en el comando. Por ejemplo: **key_simple_contosokey**. Este archivo contiene una clave cifrada.

Realice una copia del archivo tokenizado en una ubicación segura.

> [!IMPORTANT]
> Cuando posteriormente transfiera la clave a Azure Key Vault, Microsoft no podrá exportar esta clave, por lo que resulta extremadamente importante que realice una copia de seguridad de la misma y del espacio de seguridad. Póngase en contacto con [nCipher](https://www.ncipher.com/about-us/contact-us) para obtener instrucciones y procedimientos recomendados para la copia de seguridad de la clave.
>


Ya está listo para transferir la clave a Azure Key Vault.

## <a name="step-4-prepare-your-key-for-transfer"></a>Paso 4: Preparación de la clave para la transferencia

En este paso, realice los siguientes procedimientos en la estación de trabajo desconectada.

### <a name="step-41-create-a-copy-of-your-key-with-reduced-permissions"></a>Paso 4.1: Creación de una copia de la clave con permisos reducidos

Abra un nuevo símbolo del sistema y cambie el directorio actual a la ubicación en la que descomprimió el archivo zip de BYOK. Para reducir los permisos en su clave, desde un símbolo del sistema, ejecute uno de los siguientes comandos, en función de su región geográfica o instancia de Azure:

* Norteamérica:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-NA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-NA-1
* Europa:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-EU-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-EU-1
* Asia:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-AP-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-AP-1
* América Latina:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-LATAM-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-LATAM-1
* Japón:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-JPN-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-JPN-1
* Para Corea:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-KOREA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-KOREA-1
* Para Sudáfrica:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-SA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-SA-1
* Para eau:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-UAE-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-UAE-1
* Para Australia:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-AUS-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-AUS-1
* Para [Azure Government](https://azure.microsoft.com/features/gov/), que usa la instancia del gobierno de Estados Unidos de Azure:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-USGOV-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-USGOV-1
* Para DoD del Gobierno de Estados Unidos:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-USDOD-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-USDOD-1
* Para Canadá:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-CANADA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-CANADA-1
* Para Alemania:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-GERMANY-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-GERMANY-1
* Para India:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-INDIA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-INDIA-1
* Para Francia:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-FRANCE-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-FRANCE-1
* Para Reino Unido:

        KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-UK-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-UK-1

Cuando ejecute este comando, reemplace *contosokey* por el valor que especificó en el **Paso 3.5: Creación de una nueva clave** del paso [Generación de la clave](#step-3-generate-your-key).

Se le pedirá que conecte sus tarjetas de administrador del espacio de seguridad.

Cuando se complete el comando, verá **Result: ÉXITO** y la copia de la clave con permisos reducidos se encuentran en el archivo llamado key_xferacId_\<contosokey >.

Puede inspeccionar ACLS mediante los comandos siguientes utilizando las utilidades de nCipher nShield:

* aclprint.py:

        "%nfast_home%\bin\preload.exe" -m 1 -A xferacld -K contosokey "%nfast_home%\python\bin\python" "%nfast_home%\python\examples\aclprint.py"
* kmfile-dump.exe:

        "%nfast_home%\bin\kmfile-dump.exe" "%NFAST_KMDATA%\local\key_xferacld_contosokey"
  Cuando ejecute estos comandos, reemplace contosokey por el mismo valor que especificó en el **Paso 3.5: Creación de una nueva clave** del paso [Generación de la clave](#step-3-generate-your-key).

### <a name="step-42-encrypt-your-key-by-using-microsofts-key-exchange-key"></a>Paso 4.2: Cifrado de la clave mediante la clave de intercambio de claves de Microsoft

Ejecute uno de los comandos siguientes, dependiendo de su región geográfica o la instancia de Azure:

* Norteamérica:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-NA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-NA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* Europa:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-EU-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-EU-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* Asia:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-AP-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-AP-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* América Latina:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-LATAM-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-LATAM-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* Japón:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-JPN-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-JPN-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* Para Corea:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-KOREA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-KOREA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* Para Sudáfrica:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-SA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-SA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* Para eau:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-UAE-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-UAE-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* Para Australia:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-AUS-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-AUS-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* Para [Azure Government](https://azure.microsoft.com/features/gov/), que usa la instancia del gobierno de Estados Unidos de Azure:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-USGOV-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-USGOV-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* Para DoD del Gobierno de Estados Unidos:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-USDOD-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-USDOD-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* Para Canadá:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-CANADA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-CANADA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* Para Alemania:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-GERMANY-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-GERMANY-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* Para India:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-INDIA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-INDIA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* Para Francia:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-France-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-France-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
* Para Reino Unido:

        KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-UK-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-UK-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey

Cuando ejecute este comando, siga estas instrucciones:

* Reemplace *contosokey* por el identificador que usó para generar la clave en el **Paso 3.5: Creación de una nueva clave** del paso [Generación de la clave](#step-3-generate-your-key).
* Reemplace *SubscriptionID* por el identificador de la suscripción de Azure con Key Vault. Este valor lo recuperó anteriormente, en el **Paso 1.2: Obtención del identificador de la suscripción de Azure** del paso [Preparación de la estación de trabajo conectada a Internet](#step-1-prepare-your-internet-connected-workstation).
* Reemplace *ContosoFirstHSMKey* por una etiqueta que se usará para el nombre del archivo de salida.

Cuando se complete correctamente, muestra **Result: SUCCESS** y hay un nuevo archivo en la carpeta actual que tiene el siguiente nombre: KeyTransferPackage-*ContosoFirstHSMkey*.byok

### <a name="step-43-copy-your-key-transfer-package-to-the-internet-connected-workstation"></a>Paso 4.3: Copia del paquete de transferencia de claves a la estación de trabajo conectada a Internet

Utilice una unidad USB u otro dispositivo de almacenamiento portátil para copiar el archivo de salida del paso anterior (KeyTransferPackage-ContosoFirstHSMkey.byok) a la estación de trabajo conectada a Internet.

## <a name="step-5-transfer-your-key-to-azure-key-vault"></a>Paso 5: Transferencia de la clave a Azure Key Vault

En este paso final, en la estación de trabajo conectada a Internet, use el cmdlet [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey) para cargar el paquete de transferencia de claves que copió de la estación de trabajo desconectada en el dispositivo HSM de Azure Key Vault:

   ```powershell
        Add-AzKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMkey' -KeyFilePath 'c:\KeyTransferPackage-ContosoFirstHSMkey.byok' -Destination 'HSM'
   ```

Si la carga se realiza correctamente, verá que se muestran las propiedades de la clave que acaba de agregar.

## <a name="next-steps"></a>Pasos siguientes

Ahora puede usar esta clave protegida con HSM en el almacén de claves. Para obtener más información, vea este precio y características [comparación](https://azure.microsoft.com/pricing/details/key-vault/).
