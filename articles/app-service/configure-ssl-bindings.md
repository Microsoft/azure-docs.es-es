---
title: Protección de un DNS personalizado con un enlace TLS/SSL
description: Proteja el acceso HTTPS al dominio personalizado mediante la creación de un enlace de TLS/SSL con un certificado. Mejore la seguridad de su sitio web mediante la aplicación de HTTPS o TLS 1.2.
tags: buy-ssl-certificates
ms.topic: tutorial
ms.date: 04/30/2020
ms.reviewer: yutlin
ms.custom: seodec18
ms.openlocfilehash: fb62d4d2ca22b6043e63645006c2d60cf0b7859b
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88078638"
---
# <a name="secure-a-custom-dns-name-with-a-tlsssl-binding-in-azure-app-service"></a>Protección de un nombre DNS personalizado con un enlace TLS/SSL en Azure App Service

En este artículo se muestra cómo proteger el [dominio personalizado](app-service-web-tutorial-custom-domain.md) de la [aplicación de App Service](https://docs.microsoft.com/azure/app-service/) o de la [aplicación de funciones](https://docs.microsoft.com/azure/azure-functions/) mediante la creación de un enlace de certificado. Cuando haya terminado, podrá acceder a la aplicación de App Service en el punto de conexión `https://` del nombre DNS personalizado (por ejemplo, `https://www.contoso.com`). 

![Aplicación web con certificado TLS/SSL personalizado](./media/configure-ssl-bindings/app-with-custom-ssl.png)

La protección de un [dominio personalizado](app-service-web-tutorial-custom-domain.md) con un certificado implica dos pasos:

- [Agregar un certificado privado a App Service](configure-ssl-certificate.md) que cumpla todos los [requisitos de los certificados privados](configure-ssl-certificate.md#private-certificate-requirements).
-  Crear un enlace TLS al dominio personalizado correspondiente. En este artículo se trata este segundo paso.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Actualizar el plan de tarifa de la aplicación
> * Proteger un dominio personalizado con un certificado
> * Aplicación de HTTPS
> * Aplicación de TLS 1.1 y 1.2
> * Automatización de la administración de TLS con scripts

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta guía paso a paso, debe:

- [Crear una aplicación de App Service](/azure/app-service/)
- [Asignar un nombre de dominio a su aplicación](app-service-web-tutorial-custom-domain.md) o [comprarla y configurarla en Azure](manage-custom-dns-buy-domain.md)
- [Agregar un certificado privado a la aplicación](configure-ssl-certificate.md)

> [!NOTE]
> La forma más fácil de agregar un certificado privado es [crear un certificado administrado de App Service gratuito](configure-ssl-certificate.md#create-a-free-certificate-preview) (versión preliminar).

[!INCLUDE [Prepare your web app](../../includes/app-service-ssl-prepare-app.md)]

<a name="upload"></a>

## <a name="secure-a-custom-domain"></a>Protección de un dominio personalizado

Siga estos pasos:

En <a href="https://portal.azure.com" target="_blank">Azure Portal</a>, en el menú de la izquierda, seleccione **App Services** >  **\<app-name>** .

En el panel de navegación izquierdo de la aplicación, inicie el cuadro de diálogo **Enlace TLS/SSL** de la siguiente manera:

- Seleccione **Dominios personalizados** > **Agregar enlace**
- Seleccione **Configuración de TLS/SSL** > **Agregar enlace TLS/SSL**

![Agregar enlace a un dominio](./media/configure-ssl-bindings/secure-domain-launch.png)

En **Dominio personalizado**, seleccione el dominio personalizado para el que desea agregar un enlace.

Si la aplicación ya tiene un certificado para el dominio personalizado seleccionado, vaya directamente a [Crear enlace](#create-binding). De lo contrario, continúe.

### <a name="add-a-certificate-for-custom-domain"></a>Adición de un certificado para el dominio personalizado

Si la aplicación no tiene ningún certificado para el dominio personalizado seleccionado, tiene dos opciones:

- **Cargar certificado PFX**: siga el flujo de trabajo de [Carga de un certificado privado](configure-ssl-certificate.md#upload-a-private-certificate) y, a continuación, seleccione esta opción aquí.
- **Importar certificado de App Service**: siga el flujo de trabajo de [Importación de un certificado de App Service](configure-ssl-certificate.md#import-an-app-service-certificate) y, a continuación, seleccione esta opción aquí.

> [!NOTE]
> También puede [crear un certificado gratuito](configure-ssl-certificate.md#create-a-free-certificate-preview) (versión preliminar) o [importar un certificado de Key Vault](configure-ssl-certificate.md#import-a-certificate-from-key-vault), pero debe hacerlo por separado y después volver al cuadro de diálogo **Enlace TLS/SSL**.

### <a name="create-binding"></a>Creación del enlace

Utilice la siguiente tabla como ayuda para configurar el enlace TLS en el cuadro de diálogo **Enlace TLS/SSL** y haga clic en **Agregar enlace**.

| Configuración | Descripción |
|-|-|
| Dominio personalizado | Nombre de dominio para el que se agrega el enlace TLS/SSL. |
| Huella digital de certificado privado | El certificado a enlazar. |
| Tipo de TLS/SSL | <ul><li>**[SSL SNI](https://en.wikipedia.org/wiki/Server_Name_Indication)** : se pueden agregar varios enlaces SSL SNI. Esta opción permite que varios certificados TLS/SSL protejan diferentes dominios de una misma dirección IP. La mayoría de los exploradores modernos (incluidos Internet Explorer, Chrome, Firefox y Opera) admiten SNI (para más información, consulte [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication) [Indicación del nombre del servidor]).</li><li>**SSL de IP**: solo se puede agregar un enlace SSL de IP. Esta opción solo permite que un único certificado TLS/SSL proteja una dirección IP pública dedicada. Tras configurar el enlace, siga los pasos descritos en [Reasignación de registros para IP SSL](#remap-records-for-ip-ssl).<br/>SSL de IP solo se admite en el nivel **Estándar** o superior. </li></ul> |

Una vez completada la operación, el estado TLS/SSL del dominio personalizado cambia a **Seguro**.

![Enlace TLS/SSL correcto](./media/configure-ssl-bindings/secure-domain-finished.png)

> [!NOTE]
> Un estado **Seguro** en **Dominios personalizados** significa que está protegido con un certificado, pero App Service no comprueba si el certificado es autofirmado o ha expirado, por ejemplo, lo que también puede provocar que los exploradores muestren un error o una advertencia.

## <a name="remap-records-for-ip-ssl"></a>Reasignación de registros para SSL de IP

Si no usa SSL de IP en la aplicación, vaya directamente a [Probar HTTPS para el dominio personalizado](#test-https).

Puede realizar dos cambios:

- De manera predeterminada, la aplicación usa una dirección IP pública compartida. Cuando se enlaza un certificado con SSL de IP, App Service crea una dirección IP dedicada nueva para la aplicación. Si asignó un registro D a la aplicación, actualice el registro de dominio con esta nueva dirección IP dedicada.

    La página **Dominio personalizado** de la aplicación se actualiza con la nueva dirección IP dedicada. [Copie esta dirección IP](app-service-web-tutorial-custom-domain.md#info) y luego [reasigne el registro D](app-service-web-tutorial-custom-domain.md#map-an-a-record) a esta nueva dirección IP.

- Si tiene un enlace SNI SSL para `<app-name>.azurewebsites.net`, [reasigne la asignación CNAME](app-service-web-tutorial-custom-domain.md#map-a-cname-record) para que apunte a `sni.<app-name>.azurewebsites.net` en su lugar (agregue el prefijo `sni`).

## <a name="test-https"></a>Probar HTTPS

En varios exploradores, vaya a `https://<your.custom.domain>` para comprobar que da servicio a la aplicación.

![Navegación en el portal a la aplicación de Azure](./media/configure-ssl-bindings/app-with-custom-ssl.png)

El código de aplicación puede inspeccionar el protocolo mediante el encabezado "x-appservice-proto". El encabezado tendrá un valor de `http` o `https`. 

> [!NOTE]
> Si la aplicación genera errores de validación del certificado, probablemente esté utilizando un certificado autofirmado.
>
> Si no es así, puede que haya olvidado certificados intermedios cuando exportó el certificado al archivo PFX.

## <a name="prevent-ip-changes"></a>Prevención de cambios de IP

La dirección IP de entrada puede cambiar al eliminar un enlace, incluso si este es un enlace SSL de IP. Esto es especialmente importante al renovar un certificado que ya está en un enlace SSL de IP. Para evitar un cambio en la dirección IP de su aplicación, siga estos pasos en orden:

1. Carga el nuevo certificado.
2. Enlaza el nuevo certificado al dominio personalizado que desee sin eliminar el antiguo. Esta acción reemplaza el enlace en lugar de quitar el antiguo.
3. Elimine el antiguo certificado. 

## <a name="enforce-https"></a>Aplicación de HTTPS

De forma predeterminada, cualquier usuario puede acceder todavía a su aplicación mediante HTTP. Puede redirigir todas las solicitudes HTTP al puerto HTTPS.

En la página de la aplicación, en el panel de navegación izquierdo, seleccione **Configuración de SSL**. A continuación, en **Solo HTTPS**, seleccione **On**.

![Aplicación de HTTPS](./media/configure-ssl-bindings/enforce-https.png)

Una vez completada la operación, vaya a cualquiera de las direcciones URL HTTP que apuntan a la aplicación. Por ejemplo:

- `http://<app_name>.azurewebsites.net`
- `http://contoso.com`
- `http://www.contoso.com`

## <a name="enforce-tls-versions"></a>Exigencia de las versiones TLS

La aplicación permite [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.2 de forma predeterminada, que es el nivel TLS recomendado por los estándares del sector, como [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard). Para exigir versiones diferentes de TLS, siga estos pasos:

En la página de la aplicación, en el panel de navegación izquierdo, seleccione **Configuración de SSL**. A continuación, en **TLS version** (Versión de TLS), seleccione la versión mínima de TLS que desee. Esta configuración controla solo las llamadas entrantes. 

![Exigir aplicación de TLS 1.1 o 1.2](./media/configure-ssl-bindings/enforce-tls1-2.png)

Una vez completada la operación, la aplicación rechaza todas las conexiones con versiones inferiores de TLS.

## <a name="handle-tls-termination"></a>Administración de terminaciones TLS

En App Service, la [terminación TLS](https://wikipedia.org/wiki/TLS_termination_proxy) tiene lugar en los equilibradores de carga de red, por lo que todas las solicitudes HTTPS llegan a la aplicación en forma de solicitudes HTTP sin cifrar. Si su aplicación lógica necesita comprobar si las solicitudes de usuario están cifradas, inspeccione el encabezado `X-Forwarded-Proto`.

Guías de configuración específicas del lenguaje, como la guía [Configuración de Node.js en Linux](configure-language-nodejs.md#detect-https-session), que muestra cómo detectar una sesión HTTPS en el código de la aplicación.

## <a name="automate-with-scripts"></a>Automatizar con scripts

### <a name="azure-cli"></a>Azure CLI

[!code-azurecli[main](../../cli_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.sh?highlight=3-5 "Bind a custom TLS/SSL certificate to a web app")] 

### <a name="powershell"></a>PowerShell

[!code-powershell[main](../../powershell_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.ps1?highlight=1-3 "Bind a custom TLS/SSL certificate to a web app")]

## <a name="more-resources"></a>Más recursos

* [Uso de un certificado TLS/SSL en el código de Azure App Service](configure-ssl-certificate-in-code.md)
* [Preguntas más frecuentes: Certificados de App Service](https://docs.microsoft.com/azure/app-service/faq-configuration-and-management/)
