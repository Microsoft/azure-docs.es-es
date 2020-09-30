---
title: Incorporación y administración de certificados TLS/SSL
description: Cree un certificado gratuito, importe un certificado de App Service, importe un certificado de Key Vault o compre un certificado de App Service en Azure App Service.
tags: buy-ssl-certificates
ms.topic: tutorial
ms.date: 10/25/2019
ms.reviewer: yutlin
ms.custom: seodec18
ms.openlocfilehash: c8ede3c4a186b4b24d56651deb8172fdcde8e5ed
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89420887"
---
# <a name="add-a-tlsssl-certificate-in-azure-app-service"></a>Incorporación de un certificado TLS/SSL en Azure App Service

[Azure App Service](overview.md) proporciona un servicio de hospedaje web muy escalable y con aplicación de revisiones de un modo automático. En este artículo se muestra cómo crear, cargar o importar un certificado privado o un certificado público en App Service. 

Una vez que el certificado se agrega a la aplicación de App Service o la [aplicación de funciones](../azure-functions/index.yml), se puede [proteger un nombre DNS personalizado con él](configure-ssl-bindings.md) o [usarlo en el código de la aplicación](configure-ssl-certificate-in-code.md).

En la tabla siguiente se enumeran las opciones que tiene para agregar certificados en App Service:

|Opción|Descripción|
|-|-|
| Creación de un certificado administrado de App Service gratuito (versión preliminar) | Es un certificado privado fácil de usar si solo necesita proteger el [dominio personalizado](app-service-web-tutorial-custom-domain.md) `www` o cualquier dominio no desnudo en App Service. |
| Compra de un certificado de App Service | Es un certificado privado administrado por Azure. Combina la simplicidad de la administración automatizada de certificados con la flexibilidad de las opciones de renovación y exportación. |
| Importación de un certificado de Key Vault | Resulta útil si usa [Azure Key Vault](../key-vault/index.yml) para administrar los [certificados PKCS12](https://wikipedia.org/wiki/PKCS_12). Consulte [Requisitos de certificados privados](#private-certificate-requirements). |
| Carga de un certificado privado | Si ya tiene un certificado privado de un proveedor de terceros, puede cargarlo. Consulte [Requisitos de certificados privados](#private-certificate-requirements). |
| Carga de un certificado público | Los certificados públicos no se usan para proteger los dominios personalizados, pero se pueden cargar en el código si se necesitan para acceder a recursos remotos. |

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía paso a paso, debe:

- [Cree una aplicación de App Service](./index.yml).
- Solo certificado gratuito: asigne un subdominio (por ejemplo, `www.contoso.com`) a App Service con un [registro CNAME](app-service-web-tutorial-custom-domain.md#map-a-cname-record).

## <a name="private-certificate-requirements"></a>Requisitos de certificados privados

> [!NOTE]
> Azure Web Apps **no** admite AES256 y todos los archivos pfx deben cifrarse con TripleDES.

El [certificado administrado de App Service gratuito](#create-a-free-certificate-preview) o el [certificado de App Service](#import-an-app-service-certificate) ya cumplen los requisitos de App Service. Si opta por cargar o importar un certificado privado en App Service, este certificado debe cumplir los siguientes requisitos:

* Haberse exportado como [archivo PFX protegido por contraseña](https://en.wikipedia.org/w/index.php?title=X.509&section=4#Certificate_filename_extensions)
* Contener una clave privada con una longitud de al menos 2048 bits
* Contener todos los certificados intermedios de la cadena de certificados

Para proteger un dominio personalizado en un enlace TLS, el certificado debe cumplir otros requisitos:

* Contener un [uso mejorado de clave](https://en.wikipedia.org/w/index.php?title=X.509&section=4#Extensions_informing_a_specific_usage_of_a_certificate) para la autenticación de servidor (OID = 1.3.6.1.5.5.7.3.1)
* Estar firmado por una entidad de certificación de confianza

> [!NOTE]
> Los **certificados de criptografía de curva elíptica (ECC)** pueden funcionar con App Service, pero están fuera del ámbito de este artículo. Trabaje con la entidad de certificación sobre los pasos exactos para crear certificados ECC.

[!INCLUDE [Prepare your web app](../../includes/app-service-ssl-prepare-app.md)]

## <a name="create-a-free-certificate-preview"></a>Creación de un certificado gratuito (versión preliminar)

El certificado administrado de App Service gratuito es una solución inmediata para proteger el nombre DNS personalizado en App Service. Se trata de un certificado TLS/SSL totalmente funcional administrado por App Service que se renueva automáticamente. El certificado gratuito presenta las siguientes limitaciones:

- No admite certificados comodín.
- No admite dominios desnudos.
- No se puede exportar.
- No se admite en App Service Environment (ASE).
- No admite registros A. Por ejemplo, la renovación automática no funciona con registros A.

> [!NOTE]
> El certificado gratuito lo emite DigiCert. En algunos dominios de nivel superior debe permitir explícitamente DigiCert como emisor de certificados mediante la creación de un [registro de dominio de CAA](https://wikipedia.org/wiki/DNS_Certification_Authority_Authorization) con el valor `0 issue digicert.com`.
> 

Para crear un certificado administrado de App Service gratuito:

En <a href="https://portal.azure.com" target="_blank">Azure Portal</a>, en el menú de la izquierda, seleccione **App Services** >  **\<app-name>** .

En el panel de navegación izquierdo de la aplicación, seleccione **Configuración de TLS/SSL** > **Certificados de clave privada (.pfx)**  > **Crear certificado administrado de App Service**.

![Creación de un certificado gratuito en App Service](./media/configure-ssl-certificate/create-free-cert.png)

En el cuadro de diálogo se muestran los dominios no desnudos correctamente asignados a la aplicación con un registro CNAME. Seleccione el dominio personalizado para el que desea crear un certificado gratuito y seleccione **Crear**. Solo puede crear un certificado para cada dominio personalizado admitido.

Cuando se complete la operación, verá el certificado en la lista **Certificados de clave privada**.

![Creación del certificado gratuito finalizada](./media/configure-ssl-certificate/create-free-cert-finished.png)

> [!IMPORTANT] 
> Para proteger un dominio personalizado con este certificado, todavía debe crear un enlace de certificado. Siga los pasos descritos en [Creación del enlace](configure-ssl-bindings.md#create-binding).
>

## <a name="import-an-app-service-certificate"></a>Importación de un certificado de App Service

Si adquiere un certificado de App Service de Azure, Azure administra las siguientes tareas:

- Se ocupa del proceso de compra en GoDaddy.
- Realiza la comprobación de dominio del certificado.
- Mantiene el certificado en [Azure Key Vault](../key-vault/general/overview.md).
- Administra la renovación del certificado (consulte [Renovar un certificado](#renew-certificate)).
- Sincroniza el certificado automáticamente con las copias importadas en las aplicaciones de App Service.

Para adquirir un certificado de App Service, vaya a [Inicio del pedido de certificado](#start-certificate-order).

Si ya tiene un certificado de App Service en funcionamiento, puede realizar lo siguiente:

- [Importar el certificado en App Service](#import-certificate-into-app-service).
- [Administrar el certificado](#manage-app-service-certificates); por ejemplo, renovarlo, volver a especificar la clave y exportarlo.

### <a name="start-certificate-order"></a>Inicio del pedido de certificado

Inicie el pedido de un certificado de App Service en la <a href="https://portal.azure.com/#create/Microsoft.SSL" target="_blank">página de creación del certificado de App Service</a>.

![Inicio de la compra del certificado de App Service](./media/configure-ssl-certificate/purchase-app-service-cert.png)

Use la tabla siguiente para obtener ayuda para configurar el certificado. Cuando haya terminado, haga clic en **Crear**.

| Configuración | Descripción |
|-|-|
| Nombre | Nombre descriptivo para el certificado de App Service. |
| Nombre de host de dominio desnudo | Especifique aquí el dominio raíz. El certificado emitido protege *al mismo tiempo* el dominio raíz y el subdominio `www`. En el certificado emitido, el campo Nombre común contiene el dominio raíz, mientras que el campo Nombre alternativo del firmante contiene el dominio `www`. Para proteger cualquier subdominio solamente, especifique el nombre de dominio completo del subdominio aquí (por ejemplo, `mysubdomain.contoso.com`).|
| Suscripción | La suscripción que contendrá el certificado. |
| Resource group | El grupo de recursos que contendrá el certificado. Puede usar un nuevo grupo de recursos o seleccionar el mismo grupo de recursos que la aplicación de App Service, por ejemplo. |
| SKU de certificado | Determine el tipo de certificado a crear, ya sea un certificado estándar o un [certificado comodín](https://wikipedia.org/wiki/Wildcard_certificate). |
| Términos legales | Haga clic para confirmar que está de acuerdo con los términos legales. Los certificados se obtienen de GoDaddy. |

> [!NOTE]
> Los certificados de App Service adquiridos de Azure los emite GoDaddy. En algunos dominios de nivel superior debe permitir explícitamente GoDaddy como emisor de certificados mediante la creación de un [registro de dominio de CAA](https://wikipedia.org/wiki/DNS_Certification_Authority_Authorization) con el valor `0 issue godaddy.com`.
> 

### <a name="store-in-azure-key-vault"></a>Almacenamiento en Azure Key Vault

Una vez completado el proceso de compra del certificado, hay algunos pasos más que debe completar antes de poder empezar a usarlo. 

Seleccione el certificado en la página [certificados de App Service](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders) y, a continuación, haga clic en **Configuración del certificado** > **Paso 1: Almacenamiento**.

![Configuración del almacenamiento en Key Vault del certificado de App Service](./media/configure-ssl-certificate/configure-key-vault.png)

[Key Vault](../key-vault/general/overview.md) es un servicio de Azure que ayuda a proteger claves criptográficas y secretos que emplean servicios y aplicaciones en la nube. Es el almacenamiento preferido para certificados de App Service.

En la página **Estado de Key Vault**, haga clic en **Repositorio de Key Vault** para crear un nuevo almacén o elegir uno existente. Si decide crear un nuevo almacén, use la tabla siguiente para ayudarle a configurarlo y haga clic en Crear para Cree la nueva instancia de Key Vault dentro de la misma suscripción y el mismo grupo de recursos que la aplicación de App Service.

| Configuración | Descripción |
|-|-|
| Nombre | Un nombre único que consta de caracteres alfanuméricos y guiones. |
| Resource group | Como recomendación, seleccione el mismo grupo de recursos que tiene el certificado de App Service. |
| Location | Seleccione la misma ubicación que tiene la aplicación de App Service. |
| Plan de tarifa | Para obtener información, consulte [Detalles de precios de Azure Key Vault](https://azure.microsoft.com/pricing/details/key-vault/). |
| Directivas de acceso| Define las aplicaciones y el acceso permitido a los recursos del almacén. Puede configurarlo más adelante si sigue los pasos descritos en [Asignación de una directiva de acceso de Key Vault](/azure/key-vault/general/assign-access-policy-portal). |
| Acceso de redes virtuales | Restringe el acceso de almacén a determinadas redes virtuales de Azure. Puede configurarlo más adelante si sigue los pasos descritos en [Configurar firewalls y redes virtuales de Azure Key Vault](/azure/key-vault/general/network-security) |

Una vez que haya seleccionado el almacén, cierre la página del **repositorio de Key Vault**. La opción **Paso 1: Almacenamiento** debería mostrar una marca de verificación verde si se completó correctamente. Mantenga la página abierta para el siguiente paso.

### <a name="verify-domain-ownership"></a>Comprobar la propiedad del dominio

En la página **Configuración del certificado** que usó en el paso 2, haga clic en **Paso 2: Comprobación**.

![Comprobación del dominio del certificado de App Service](./media/configure-ssl-certificate/verify-domain.png)

Seleccione **Comprobación de App Service**. Puesto que ya ha asignado el dominio a la aplicación web (consulte los [requisitos previos](#prerequisites)), este ya se comprobó. Haga clic en **Comprobar** para finalizar este paso. Haga clic en el botón **Actualizar** hasta que aparezca el mensaje **El certificado tiene el dominio comprobado**.

> [!NOTE]
> Se admiten cuatro tipos de métodos de comprobación de dominio: 
> 
> - **App Service**: es la opción más conveniente cuando el dominio ya está asignado a una aplicación App Service de la misma suscripción. Se aprovecha de que la aplicación App Service ya ha comprobado la propiedad del dominio.
> - **Dominio**: se comprueba un [dominio de App Service que haya adquirido a través de Azure](manage-custom-dns-buy-domain.md). Azure agrega automáticamente el registro TXT de comprobación en su lugar y completa el proceso.
> - **Correo electrónico**: se comprueba el dominio mediante el envío de un correo electrónico al administrador de dominio. Cuando selecciona la opción, se proporcionan instrucciones.
> - **Manual**: se comprueba el dominio ya sea con una página HTML (solo los certificados **estándar**) o un registro TXT de DNS. Cuando selecciona la opción, se proporcionan instrucciones.

### <a name="import-certificate-into-app-service"></a>Importación del certificado en App Service

En <a href="https://portal.azure.com" target="_blank">Azure Portal</a>, en el menú de la izquierda, seleccione **App Services** >  **\<app-name>** .

En el panel izquierdo de la aplicación, seleccione **Configuración de TLS/SSL** > **Certificados de clave privada (.pfx)**  > **Importar certificado de App Service**.

![Importación del certificado de App Service en App Service](./media/configure-ssl-certificate/import-app-service-cert.png)

Seleccione el certificado que acaba de comprar y, después, seleccione **Aceptar**.

Cuando se complete la operación, verá el certificado en la lista **Certificados de clave privada**.

![Importación del certificado de App Service finalizada](./media/configure-ssl-certificate/import-app-service-cert-finished.png)

> [!IMPORTANT] 
> Para proteger un dominio personalizado con este certificado, todavía debe crear un enlace de certificado. Siga los pasos descritos en [Creación del enlace](configure-ssl-bindings.md#create-binding).
>

## <a name="import-a-certificate-from-key-vault"></a>Importación de un certificado de Key Vault

Si usa Azure Key Vault para administrar los certificados, puede importar un certificado PKCS12 de Key Vault en App Service, siempre que [cumpla los requisitos](#private-certificate-requirements).

### <a name="authorize-app-service-to-read-from-the-vault"></a>Autorización a App Service para leer desde el almacén
De forma predeterminada, el proveedor de recursos de App Service no tiene acceso al almacén de claves. Para usar un almacén de claves para la implementación de un certificado, debe [autorizar al proveedor de recursos acceso de lectura al almacén de claves](../key-vault/general/group-permissions-for-apps.md#grant-access-to-your-key-vault). 

`abfa0a7c-a6b6-4736-8310-5855508787cd` es el nombre de la entidad de seguridad de servicio del proveedor de recursos para App Service y es el mismo para todas las suscripciones de Azure. Para un entorno en la nube de Azure Government, use `6a02c803-dafd-4136-b4c3-5a6f318b4714` en lugar del nombre de la entidad de seguridad de servicio del proveedor de recursos.

### <a name="import-a-certificate-from-your-vault-to-your-app"></a>Importación de un certificado desde el almacén a la aplicación

En <a href="https://portal.azure.com" target="_blank">Azure Portal</a>, en el menú de la izquierda, seleccione **App Services** >  **\<app-name>** .

En el panel izquierdo de la aplicación, seleccione **Configuración de TLS/SSL** > **Certificados de clave privada (.pfx)**  > **Importar certificado de Key Vault**.

![Importación de un certificado de Key Vault en App Service](./media/configure-ssl-certificate/import-key-vault-cert.png)

Use la tabla siguiente como ayuda para seleccionar el certificado.

| Configuración | Descripción |
|-|-|
| Subscription | Suscripción a la que pertenece la instancia de Key Vault. |
| Key Vault | Almacén que incluye el certificado que desea importar. |
| Certificado | Seleccione en la lista de certificados PKCS12 del almacén. Se enumeran todos los certificados PKCS12 del almacén con sus huellas digitales, pero no todos se admiten en App Service. |

Cuando se complete la operación, verá el certificado en la lista **Certificados de clave privada**. Si se produce un error en la importación, indica que el certificado no cumple los [requisitos para App Service](#private-certificate-requirements).

![Importación del certificado de Key Vault finalizada](./media/configure-ssl-certificate/import-app-service-cert-finished.png)

> [!NOTE]
> Si actualiza el certificado en Key Vault con un nuevo certificado, App Service sincronizará automáticamente el certificado en un plazo de 48 horas.

> [!IMPORTANT] 
> Para proteger un dominio personalizado con este certificado, todavía debe crear un enlace de certificado. Siga los pasos descritos en [Creación del enlace](configure-ssl-bindings.md#create-binding).
>

## <a name="upload-a-private-certificate"></a>Carga de un certificado privado

Una vez que obtenga un certificado de su proveedor de certificados, siga los pasos que se describen en esta sección para prepararlo para App Service.

### <a name="merge-intermediate-certificates"></a>Combinación de certificados intermedios

Si la entidad emisora de certificados ofrece varios certificados en la cadena de certificados, debe combinar los certificados en orden.

Para ello, abra cada certificado que ha recibido en un editor de texto.

Cree un archivo para el certificado combinado, denominado _mergedcertificate.crt_. En un editor de texto, copie el contenido de cada certificado en este archivo. Los certificados deben seguir el orden de la cadena de certificados, comenzando por el certificado y terminando por el certificado raíz. Debe ser similar al ejemplo siguiente:

```
-----BEGIN CERTIFICATE-----
<your entire Base64 encoded SSL certificate>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded intermediate certificate 1>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded intermediate certificate 2>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded root certificate>
-----END CERTIFICATE-----
```

### <a name="export-certificate-to-pfx"></a>Exportar el certificado a PFX

Exporte el certificado TLS/SSL personalizado con la clave privada que utilizó para generar la solicitud de certificado.

Si la solicitud de certificado se genera con OpenSSL, se crea un archivo de clave privada. Para exportar el certificado a PFX, ejecute el comando siguiente: Reemplace los marcadores de posición _&lt;private-key-file>_ y _&lt;merged-certificate-file>_ por la ruta a la clave privada y al archivo de certificado combinado.

```bash
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>  
```

Cuando se le pida, defina una contraseña de exportación. Esta contraseña deberá usarla cuando posteriormente cargue el certificado TLS/SSL en App Service.

Si usó IIS o _Certreq.exe_ para generar la solicitud de certificado, instale el certificado en la máquina local y luego [exporte el certificado a PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).

### <a name="upload-certificate-to-app-service"></a>Carga del certificado en App Service

Ya está listo para cargar el certificado en App Service.

En <a href="https://portal.azure.com" target="_blank">Azure Portal</a>, en el menú de la izquierda, seleccione **App Services** >  **\<app-name>** .

En el panel izquierdo de la aplicación, seleccione **Configuración de TLS/SSL** > **Certificados de clave privada (.pfx)**  > **Cargar certificado**.

![Carga del certificado privado en App Service](./media/configure-ssl-certificate/upload-private-cert.png)

En **Archivo de certificado PFX**, seleccione el archivo PFX. En **Contraseña del certificado**, escriba la contraseña que creó al exportar el archivo PFX. Cuando termine, haga clic en **Cargar**. 

Cuando se complete la operación, verá el certificado en la lista **Certificados de clave privada**.

![Carga del certificado finalizada](./media/configure-ssl-certificate/create-free-cert-finished.png)

> [!IMPORTANT] 
> Para proteger un dominio personalizado con este certificado, todavía debe crear un enlace de certificado. Siga los pasos descritos en [Creación del enlace](configure-ssl-bindings.md#create-binding).
>

## <a name="upload-a-public-certificate"></a>Carga de un certificado público

Se admiten certificados públicos en el formato *.cer*. 

En <a href="https://portal.azure.com" target="_blank">Azure Portal</a>, en el menú de la izquierda, seleccione **App Services** >  **\<app-name>** .

En el panel de navegación izquierdo de la aplicación, haga clic en **Configuración de TLS/SSL** > **Certificados públicos (.cer)**  > **Cargar certificado de clave pública**.

En **Nombre**, escriba un nombre para el certificado. En **Archivo de certificado CER**, seleccione el archivo CER.

Haga clic en **Cargar**.

![Carga de un certificado público en App Service](./media/configure-ssl-certificate/upload-public-cert.png)

Una vez que el certificado se cargue, copie la huella digital del certificado y consulte [Que el certificado sea accesible](configure-ssl-certificate-in-code.md#make-the-certificate-accessible).

## <a name="manage-app-service-certificates"></a>Administración de certificados de App Service

En esta sección se muestra cómo administrar un certificado de App Service adquirido en [Importación de un certificado de App Service](#import-an-app-service-certificate).

- [Volver a especificar la clave del certificado](#rekey-certificate)
- [Renovar un certificado](#renew-certificate)
- [Exportar un certificado](#export-certificate)
- [Eliminar un certificado](#delete-certificate)

### <a name="rekey-certificate"></a>Volver a especificar la clave del certificado

Si piensa que la clave privada del certificado está en peligro, puede volver a especificar la clave del certificado. Seleccione el certificado en la página [Certificados de App Service](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders) y, después, seleccione **Regenerar claves y sincronizar** desde el panel de navegación izquierdo.

Haga clic en el botón **Regenerar clave** para iniciar el proceso. Este proceso puede tardar de 1 a 10 minutos en completarse.

![Nueva especificación de la clave de un certificado de App Service](./media/configure-ssl-certificate/rekey-app-service-cert.png)

La regeneración de claves del certificado renovará el certificado con un nuevo certificado emitido desde la entidad de certificación.

Una vez que se complete la operación de regeneración de claves, haga clic en **Sincronizar**. La operación de sincronización actualiza automáticamente los enlaces de nombre de host para el certificado en App Service sin tiempo de inactividad para las aplicaciones.

> [!NOTE]
> Si no hace clic en **Sincronizar**, App Service sincroniza automáticamente el certificado dentro de 48 horas.

### <a name="renew-certificate"></a>Renovar un certificado

Para activar la renovación automática del certificado en cualquier momento, seleccione el certificado en la página [Certificados de App Service](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders) y, después, haga clic en **Configuración de renovación automática** en el panel de navegación izquierdo. De forma predeterminada, los certificados de App Service tienen un período de validez de un año.

Seleccione **Activar** y luego haga clic en **Guardar**. Los certificados pueden comenzar a renovarse automáticamente 60 días antes del vencimiento si tiene activada la renovación automática.

![Renovación automática del certificado de App Service](./media/configure-ssl-certificate/auto-renew-app-service-cert.png)

En cambio, para renovar manualmente el certificado, haga clic en **Renovación manual**. Puede solicitar renovar manualmente el certificado de 60 días antes de que expire.

Una vez que se completa la operación de renovación, haga clic en **Sincronizar**. La operación de sincronización actualiza automáticamente los enlaces de nombre de host para el certificado en App Service sin tiempo de inactividad para las aplicaciones.

> [!NOTE]
> Si no hace clic en **Sincronizar**, App Service sincroniza automáticamente el certificado dentro de 48 horas.

### <a name="export-certificate"></a>Exportación de certificado

Dado que un certificado de App Service es un [secreto de Key Vault](../key-vault/general/about-keys-secrets-certificates.md), puede exportar una copia PFX y usarla para otros servicios de Azure o fuera de Azure.

Para exportar el certificado de App Service como un archivo PFX, ejecute los siguientes comandos en [Cloud Shell](https://shell.azure.com). También puede ejecutarlo localmente si [instaló la CLI de Azure](/cli/azure/install-azure-cli). Reemplace los marcadores de posición por los nombres que usó cuando [creó el certificado de App Service](#start-certificate-order).

```azurecli-interactive
secretname=$(az resource show \
    --resource-group <group-name> \
    --resource-type "Microsoft.CertificateRegistration/certificateOrders" \
    --name <app-service-cert-name> \
    --query "properties.certificates.<app-service-cert-name>.keyVaultSecretName" \
    --output tsv)

az keyvault secret download \
    --file appservicecertificate.pfx \
    --vault-name <key-vault-name> \
    --name $secretname \
    --encoding base64
```

El archivo *appservicecertificate.pfx* descargado es un archivo PKCS12 sin procesar que contiene los certificados tanto público como privado. En cada solicitud, use una cadena vacía para la contraseña de importación y la frase de contraseña PEM.

### <a name="delete-certificate"></a>Eliminar certificado 

La eliminación de un certificado de App Service es final e irreversible. La eliminación de un recurso de App Service Certificate da como resultado la revocación del certificado. Cualquier enlace en App Service con este certificado dejará de ser válido. Para evitar la eliminación accidental, Azure coloca un bloqueo en el certificado. Para eliminar un certificado de App Service, antes debe quitar su bloqueo de eliminación.

Seleccione el certificado en la página [Certificados de App Service](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders) y, después, seleccione **Bloqueos** en el panel de navegación izquierdo.

Busque el bloqueo en el certificado con el tipo de bloqueo **Eliminar**. A la derecha de él, seleccione **Eliminar**.

![Eliminación del bloqueo del certificado de App Service](./media/configure-ssl-certificate/delete-lock-app-service-cert.png)

Ahora puede eliminar el certificado de App Service. En el panel de navegación izquierdo, seleccione **Información general** > **Eliminar**. En el cuadro de diálogo de confirmación, escriba el nombre del certificado y seleccione **Aceptar**.

## <a name="automate-with-scripts"></a>Automatizar con scripts

### <a name="azure-cli"></a>Azure CLI

[!code-azurecli[main](../../cli_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.sh?highlight=3-5 "Bind a custom TLS/SSL certificate to a web app")] 

### <a name="powershell"></a>PowerShell

[!code-powershell[main](../../powershell_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.ps1?highlight=1-3 "Bind a custom TLS/SSL certificate to a web app")]

## <a name="more-resources"></a>Más recursos

* [Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service](configure-ssl-bindings.md)
* [Aplicación de HTTPS](configure-ssl-bindings.md#enforce-https)
* [Aplicación de TLS 1.1 y 1.2](configure-ssl-bindings.md#enforce-tls-versions)
* [Uso de un certificado TLS/SSL en el código de Azure App Service](configure-ssl-certificate-in-code.md)
* [Preguntas más frecuentes: Certificados de App Service](./faq-configuration-and-management.md)
