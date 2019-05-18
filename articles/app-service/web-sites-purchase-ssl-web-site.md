---
title: Comprar y configurar un certificado SSL de Azure - App Service | Microsoft Docs
description: Aprenda a comprar un certificado de App Service y a enlazarlo a esta aplicación
services: app-service
documentationcenter: .net
author: cephalin
manager: jpconnoc
tags: buy-ssl-certificates
ms.assetid: cdb9719a-c8eb-47e5-817f-e15eaea1f5f8
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/16/2018
ms.author: apurvajo;cephalin
ms.custom: seodec18
ms.openlocfilehash: 15441f70052950748f07b96ff234557462f268cd
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65786320"
---
# <a name="buy-and-configure-an-ssl-certificate-for-azure-app-service"></a>Compra y configuración de un certificado SSL para Azure App Service

En este tutorial se muestra cómo proteger la [aplicación de App Service](https://docs.microsoft.com/azure/app-service/) o la [aplicación de función](https://docs.microsoft.com/azure/azure-functions/) mediante la creación (compra) de un certificado de App Service en [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-whatis) y su vinculación a una aplicación de App Service.

> [!TIP]
> Las instancias de App Service Certificate pueden usarse para cualquier servicio de Azure o que no sea de Azure, y no se limitan a App Services. Para ello, tiene que crear una copia PFX local de un certificado de App Service que pueda usar donde quiera. Para más información, visite nuestra entrada de blog [Creating a local PFX copy of an App Service Certificate](https://blogs.msdn.microsoft.com/benjaminperkins/2017/04/12/export-an-azure-app-service-certificate-pfx-powershell/) (Creación de una copia PFX local de una instancia de App Service Certificate).
>

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía paso a paso, debe:

- [Crear una aplicación de App Service](/azure/app-service/)
- [Asignar un nombre de dominio a su aplicación](app-service-web-tutorial-custom-domain.md) o [comprarla y configurarla en Azure](manage-custom-dns-buy-domain.md)

[!INCLUDE [Prepare your web app](../../includes/app-service-ssl-prepare-app.md)]

## <a name="start-certificate-order"></a>Inicio del pedido de certificado

Inicie el pedido de un certificado de App Service en la <a href="https://portal.azure.com/#create/Microsoft.SSL" target="_blank">página de creación del certificado de App Service</a>.

![Creación de certificados](./media/app-service-web-purchase-ssl-web-site/createssl.png)

Use la tabla siguiente para obtener ayuda para configurar el certificado. Cuando haya terminado, haga clic en **Crear**.

| Configuración | DESCRIPCIÓN |
|-|-|
| NOMBRE | Nombre descriptivo para el certificado de App Service. |
| Nombre de host de dominio desnudo | Si especifica el dominio raíz aquí, obtenga un certificado que proteja *tanto*  al dominio raíz como al subdominio `www`. Para proteger cualquier subdominio solamente, especifique el nombre de dominio completo del subdominio aquí (por ejemplo, `mysubdomain.contoso.com`). |
| Subscription | El centro de datos donde se hospeda la aplicación web. |
| Grupos de recursos | El grupo de recursos que contiene el certificado. Puede usar un nuevo grupo de recursos o seleccionar el mismo grupo de recursos que la aplicación de App Service, por ejemplo. |
| SKU de certificado | Determine el tipo de certificado a crear, ya sea un certificado estándar o un [certificado comodín](https://wikipedia.org/wiki/Wildcard_certificate). |
| Términos legales | Haga clic para confirmar que está de acuerdo con los términos legales. Los certificados se obtienen de GoDaddy. |

## <a name="store-in-azure-key-vault"></a>Almacenamiento en Azure Key Vault

Una vez completado el proceso de compra del certificado, hay algunos pasos más que debe completar antes de poder empezar a usarlo. 

Seleccione el certificado en la página [certificados de App Service](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders) y, a continuación, haga clic en **Configuración del certificado** > **Paso 1: Almacenamiento**.

![insertar imagen de listo para almacenar en KV](./media/app-service-web-purchase-ssl-web-site/ReadyKV.png)

[Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-whatis) es un servicio de Azure que ayuda a proteger claves criptográficas y secretos que emplean servicios y aplicaciones en la nube. Es el almacenamiento preferido para certificados de App Service.

En la página **Estado de Key Vault**, haga clic en **Repositorio de Key Vault** para crear un nuevo almacén o elegir uno existente. Si decide crear un nuevo almacén, use la tabla siguiente para ayudarle a configurarlo y haga clic en Crear para crear un almacén de claves en la misma suscripción y grupo de recursos.

| Configuración | DESCRIPCIÓN |
|-|-|
| NOMBRE | Un nombre único que consta de caracteres alfanuméricos y guiones. |
| Grupos de recursos | Como recomendación, seleccione el mismo grupo de recursos que tiene el certificado de App Service. |
| Ubicación | Seleccione la misma ubicación que tiene la aplicación de App Service. |
| Plan de tarifa | Para obtener información, consulte [Detalles de precios de Azure Key Vault](https://azure.microsoft.com/pricing/details/key-vault/). |
| Directivas de acceso| Define las aplicaciones y el acceso permitido a los recursos del almacén. Puede configurarlo más adelante si sigue los pasos descritos en [Concesión de permisos para que muchas aplicaciones tengan acceso al almacén de claves](../key-vault/key-vault-group-permissions-for-apps.md). |
| Acceso de redes virtuales | Restringe el acceso de almacén a determinadas redes virtuales de Azure. Puede configurarlo más adelante si sigue los pasos descritos en [Configurar firewalls y redes virtuales de Azure Key Vault](../key-vault/key-vault-network-security.md) |

Una vez que haya seleccionado el almacén, cierre la página del **repositorio de Key Vault**. La opción **Almacenamiento** debería mostrar una marca de verificación verde si se completó correctamente. Mantenga la página abierta para el siguiente paso.

## <a name="verify-domain-ownership"></a>Comprobar la propiedad del dominio

En la página **Configuración del certificado** que usó en el paso 2, haga clic en **Paso 2: Comprobación**.

![](./media/app-service-web-purchase-ssl-web-site/verify-domain.png)

Seleccione **Comprobación de App Service**. Puesto que ya ha asignado el dominio a la aplicación web (consulte los [requisitos previos](#prerequisites)), este ya se comprobó. Haga clic en **Comprobar** para finalizar este paso. Haga clic en el botón **Actualizar** hasta que aparezca el mensaje **El certificado tiene el dominio comprobado**.

> [!NOTE]
> Se admiten cuatro tipos de métodos de comprobación de dominio: 
> 
> - **App Service**: es la opción más conveniente cuando el dominio ya está asignado a una aplicación App Service de la misma suscripción. Se aprovecha de que la aplicación App Service ya ha comprobado la propiedad del dominio.
> - **Dominio**: se comprueba un [dominio de App Service que haya adquirido a través de Azure](manage-custom-dns-buy-domain.md). Azure agrega automáticamente el registro TXT de comprobación en su lugar y completa el proceso.
> - **Correo electrónico**: se comprueba el dominio mediante el envío de un correo electrónico al administrador de dominio. Cuando selecciona la opción, se proporcionan instrucciones.
> - **Manual**: se comprueba el dominio ya sea con una página HTML (solo los certificados **estándar**) o un registro TXT de DNS. Cuando selecciona la opción, se proporcionan instrucciones.

## <a name="bind-certificate-to-app"></a>Enlace del certificado a la aplicación

En **[Azure Portal](https://portal.azure.com/)**, en el menú de la izquierda, seleccione **App Services** > **\<su_aplicacion>**.

En el panel izquierdo de la aplicación, seleccione **Configuración SSL** > **Certificados privados (.pfx)** > **Importar certificado de App Service**.

![insertar imagen de Importar certificado](./media/app-service-web-purchase-ssl-web-site/ImportCertificate.png)

Seleccione el certificado que recién compró.

Ahora que se ha importado el certificado, debe enlazarlo a un nombre de dominio asignado en la aplicación. Seleccione **Enlaces** > **Agregar enlace SSL**. 

![insertar imagen de Importar certificado](./media/app-service-web-purchase-ssl-web-site/AddBinding.png)

Utilice la siguiente tabla para ayudarle a configurar el enlace en el cuadro de diálogo **Enlaces SSL** y, a continuación, haga clic en **Agregar enlace**.

| Configuración | DESCRIPCIÓN |
|-|-|
| Nombre de host. | El nombre de dominio al que agregará el enlace SSL. |
| Huella digital de certificado privado | El certificado a enlazar. |
| Tipo de SSL | <ul><li>**SSL basada en SNI**: pueden agregarse varios enlaces SSL basados en SNI. Esta opción permite que varios certificados SSL protejan varios dominios en una misma dirección IP. Los exploradores más modernos (como Internet Explorer, Chrome, Firefox y Opera) admiten SNI (encontrará información de compatibilidad con exploradores más completa en [Indicación de nombre de servidor](https://wikipedia.org/wiki/Server_Name_Indication)).</li><li>**SSL basada en IP**: solo pueden agregarse enlaces SSL basados en IP. Esta opción solo permite que un único certificado SSL proteja una dirección IP dedicada. Tras configurar el enlace, siga los pasos descritos en [Reasignar un registro D en SSL de IP](app-service-web-tutorial-custom-ssl.md#remap-a-record-for-ip-ssl). </li></ul> |

## <a name="verify-https-access"></a>Comprobación del acceso HTTPS

Visite su aplicación con `HTTPS://<domain_name>` en lugar de `HTTP://<domain_name>`, para verificar que el certificado se ha configurado correctamente.

## <a name="rekey-certificate"></a>Regenerar claves del certificado

Si piensa que privado del certificado del clave está en peligro, puede cambiar la clave del certificado. Seleccione el certificado en el [certificados de App Service](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders) página y, después, seleccione **regenerar claves y sincronizar** desde el panel de navegación izquierdo.

Haga clic en **regenerar** para iniciar el proceso. Este proceso puede tardar de 1 a 10 minutos en completarse.

![insertar imagen de SSL de regeneración de claves](./media/app-service-web-purchase-ssl-web-site/Rekey.png)

La regeneración de claves del certificado renovará el certificado con un nuevo certificado emitido desde la entidad de certificación.

Una vez completada la operación de regeneración de claves, haga clic en **sincronización**. La operación de sincronización actualiza automáticamente los enlaces de nombre de host para el certificado en App Service sin provocar ningún tiempo de inactividad para las aplicaciones.

> [!NOTE]
> Si no elige **sincronización**, App Service se sincroniza automáticamente el certificado en 48 horas.

## <a name="renew-certificate"></a>Renovar certificado

Para activar la renovación automática del certificado en cualquier momento, seleccione el certificado en la página [Certificados de App Service](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders) y, después, haga clic en **Configuración de renovación automática** en el panel de navegación izquierdo.

Seleccione **Activar** y luego haga clic en **Guardar**. Los certificados pueden comenzar a renovarse automáticamente 60 días antes del vencimiento si tiene activada la renovación automática.

![Renovar certificados automáticamente](./media/app-service-web-purchase-ssl-web-site/auto-renew.png)

En cambio, para renovar manualmente el certificado, haga clic en **Renovación manual**. Puede solicitar renovar manualmente el certificado de 60 días antes de que expire.

Una vez completada la operación de renovación, haga clic en **sincronización**. La operación de sincronización actualiza automáticamente los enlaces de nombre de host para el certificado en App Service sin provocar ningún tiempo de inactividad para las aplicaciones.

> [!NOTE]
> Si no elige **sincronización**, App Service se sincroniza automáticamente el certificado en 48 horas.

## <a name="automate-with-scripts"></a>Automatizar con scripts

### <a name="azure-cli"></a>Azure CLI

[!code-azurecli[main](../../cli_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.sh?highlight=3-5 "Bind a custom SSL certificate to a web app")] 

### <a name="powershell"></a>PowerShell

[!code-powershell[main](../../powershell_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.ps1?highlight=1-3 "Bind a custom SSL certificate to a web app")]

## <a name="more-resources"></a>Más recursos

* [Aplicación de HTTPS](app-service-web-tutorial-custom-ssl.md#enforce-https)
* [Aplicación de TLS 1.1 y 1.2](app-service-web-tutorial-custom-ssl.md#enforce-tls-versions)
* [Uso de un certificado SSL en el código de aplicación de Azure App Service](app-service-web-ssl-cert-load.md)
* [Preguntas más frecuentes: Certificados de App Service](https://docs.microsoft.com/en-us/azure/app-service/faq-configuration-and-management/)
