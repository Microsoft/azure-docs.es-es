---
title: Configuración de una instancia y autenticación (con scripts)
titleSuffix: Azure Digital Twins
description: Vea cómo configurar una instancia del servicio de Azure Digital Twins mediante la ejecución de un script de implementación automatizada.
author: baanders
ms.author: baanders
ms.date: 7/23/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 605df0f26600f962bda7a0a0def800a91d74b022
ms.sourcegitcommit: 6e1124fc25c3ddb3053b482b0ed33900f46464b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90563002"
---
# <a name="set-up-an-azure-digital-twins-instance-and-authentication-scripted"></a>Configuración de una instancia de Azure Digital Twins y autenticación (con scripts)

[!INCLUDE [digital-twins-setup-selector.md](../../includes/digital-twins-setup-selector.md)]

En este artículo se describen los pasos para **configurar una nueva instancia de Azure Digital Twins**, incluidas la creación de la instancia y la configuración de la autenticación. Después de completar este artículo, tendrá una instancia de Azure Digital Twins lista para empezar a programar.

En esta versión de este artículo se completan estos pasos mediante la ejecución de un [**script de implementación automatizada** de ejemplo](https://docs.microsoft.com/samples/azure-samples/digital-twins-samples/digital-twins-samples/) que simplifica el proceso. 
* Para ver los pasos manuales de la CLI que el script se ejecuta en segundo plano, consulte la versión de la CLI de este artículo: [*Procedimiento: Configuración de una instancia y autenticación (CLI)* ](how-to-set-up-instance-cli.md).
* Para ver los pasos manuales según Azure Portal, consulte la versión del portal de este artículo: [*Procedimiento: Configuración de una instancia y autenticación (portal)* ](how-to-set-up-instance-portal.md).

[!INCLUDE [digital-twins-setup-steps-prereq.md](../../includes/digital-twins-setup-steps-prereq.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="run-the-deployment-script"></a>Ejecutar el script de implementación

En este artículo se usa un ejemplo de código de Azure Digital Twins para implementar una instancia de Azure Digital Twins y la autenticación necesaria de forma semiautomática. También se puede usar como punto de partida para escribir sus propias interacciones con scripts.

El script de ejemplo se escribe en PowerShell. Forma parte de los [ejemplos de Azure Digital Twins](https://docs.microsoft.com/samples/azure-samples/digital-twins-samples/digital-twins-samples/), que puede descargar en la máquina. Para ello, vaya a ese vínculo de ejemplo y seleccione el botón *Descargar ZIP* situado debajo del título.

En la carpeta de ejemplo descargada, el script de implementación se encuentra en _Azure_Digital_Twins_samples.zip > scripts > **deploy.ps1**_.

Estos son los pasos para ejecutar el script de implementación en Cloud Shell.
1. Vaya a una ventana de [Azure Cloud Shell](https://shell.azure.com/) en el explorador. Inicie sesión con este comando:
    ```azurecli
    az login
    ```
    Si la CLI puede abrir el explorador predeterminado, lo hará y cargará una página de inicio de sesión de Azure. En caso contrario, abra una página del explorador en *https://aka.ms/devicelogin* y escriba el código de autorización que se muestra en el terminal.
 
2. Después de iniciar sesión, fíjese en la barra de iconos de la ventana de Cloud Shell. Seleccione el icono "Cargar/Descargar archivos" y elija "Cargar".

    :::image type="content" source="media/how-to-set-up-instance/cloud-shell/cloud-shell-upload.png" alt-text="Ventana de Cloud Shell que muestra la selección de la opción Cargar":::

    Vaya al archivo _**deploy.ps1**_ en la máquina y pulse "Abrir". Con esta acción se cargará el archivo en Cloud Shell para que pueda ejecutarlo en la ventana de Cloud Shell.

3. Ejecute el script mediante el envío del comando `./deploy.ps1` en la ventana de Cloud Shell. Cuando el script se ejecuta a través de los pasos de configuración automatizada, se le solicitará que pase los valores siguientes:
    * Por ejemplo, el *identificador de suscripción* de la suscripción de Azure.
    * Por ejemplo, una *ubicación* en la que quiera implementar la instancia. Para ver qué regiones admiten Azure Digital Twins, visite [*Productos de Azure disponibles por región*](https://azure.microsoft.com/global-infrastructure/services/?products=digital-twins).
    * Por ejemplo, el nombre de un *grupo de recursos*. Puede usar un grupo de recursos existente o escribir un nuevo nombre para crear uno.
    * Por ejemplo, un *nombre* para la instancia de Azure Digital Twins. El nombre de la nueva instancia debe ser único dentro de la región de la suscripción (lo que significa que si la suscripción tiene otra instancia de Azure Digital Twins en la región que ya usa el nombre elegido, se le solicitará que elija otro nombre).
    * Para el registro de la aplicación, un *nombre para mostrar de una aplicación de Azure AD* para asociar al registro. Este registro de la aplicación es donde se configuran los permisos de acceso a las [API de Azure Digital Twins](how-to-use-apis-sdks.md). Más tarde, la aplicación cliente se autentica en el registro de la aplicación y, como resultado, se le conceden los permisos de acceso configurados a las API.
    * Para el registro de la aplicación, un *URL de respuesta de aplicación de Azure AD* para la aplicación de Azure AD. Use `http://localhost`. El script configurará un *URI de cliente público/nativo (para escritorio y aplicaciones móviles*).

El script creará una instancia de Azure Digital Twins, asignará al usuario de Azure el rol de *Propietario de Azure Digital Twins (versión preliminar)* en la instancia y configurará un registro de aplicación de Azure AD para que lo use la aplicación cliente.

>[!NOTE]
>Actualmente hay un **problema conocido** con la instalación con scripts, por el que algunos usuarios (especialmente los usuarios de las [cuentas de Microsoft personales o MSA](https://account.microsoft.com/account)) pueden encontrar que no se creó la **asignación de roles para el _Propietario de Azure Digital Twins (versión preliminar)_**.
>
>Puede comprobar la asignación de roles mediante la sección [*Comprobación de la asignación de roles de usuario*](#verify-user-role-assignment) que aparece más adelante en este artículo y, si es necesario, configurar la asignación de roles manualmente mediante [Azure Portal](how-to-set-up-instance-portal.md#set-up-user-access-permissions) o la [CLI](how-to-set-up-instance-cli.md#set-up-user-access-permissions).
>
>Para más información, consulte [*Solución de problemas: Problemas conocidos en Azure Digital Twins*](troubleshoot-known-issues.md#missing-role-assignment-after-scripted-setup).

A continuación se muestra un extracto del registro de salida del script:

:::image type="content" source="media/how-to-set-up-instance/cloud-shell/deployment-script-output.png" alt-text="Ventana de Cloud Shell que muestra el registro de entrada y salida a través de la ejecución del script de implementación" lightbox="media/how-to-set-up-instance/cloud-shell/deployment-script-output.png":::

Si el script se completa correctamente, la copia impresa final indicará `Deployment completed successfully`. De lo contrario, solucione el mensaje de error y vuelva a ejecutar el script. Omitirá los pasos que ya ha completado y volverá a solicitar la entrada en el punto en el que se quedó.

Una vez finalizado el script, tendrá una instancia de Azure Digital Twins lista con permisos para administrarla, así como los permisos configurados para que una aplicación cliente tenga acceso a ella.

> [!NOTE]
> Actualmente, el script asigna el rol de administración necesario en Azure Digital Twins (*Propietario de Azure Digital Twins (versión preliminar)* ) al mismo usuario que ejecuta el script desde Cloud Shell. Si tiene que asignar este rol a otra persona que vaya a administrar la instancia, puede hacerlo ahora a través de Azure Portal ([instrucciones](how-to-set-up-instance-portal.md#set-up-user-access-permissions)) o la CLI ([instrucciones](how-to-set-up-instance-cli.md#set-up-user-access-permissions)).

## <a name="collect-important-values"></a>Recopilación de valores importantes

Hay varios valores importantes de los recursos configurados por el script que puede necesitar mientras sigue trabajando con la instancia de Azure Digital Twins. En esta sección, usará [Azure Portal](https://portal.azure.com) para recopilar estos valores. Debe guardarlos en un lugar seguro o volver a esta sección para encontrarlos más adelante cuando los necesite.

Si otros usuarios van a programar en la instancia, también debe compartirlos con ellos.

### <a name="collect-instance-values"></a>Recopilación de valores de la instancia

En [Azure Portal](https://portal.azure.com), para buscar la instancia de Azure Digital Twins, escriba su nombre en la barra de búsqueda del portal.

Al seleccionarlo, se abrirá la página *Información general* de la instancia. Anote su *Nombre*, *Grupo de recursos* y *Nombre de host*. Puede necesitarlos más adelante para identificar la instancia y conectarse a ella.

:::image type="content" source="media/how-to-set-up-instance/portal/instance-important-values.png" alt-text="Resaltado de los valores importantes en la página de información general de la instancia":::

### <a name="collect-app-registration-values"></a>Recopilación de valores de registro de aplicaciones 

Hay dos valores importantes del registro de la aplicación que se necesitarán más adelante para [escribir el código de autenticación de la aplicación cliente para las API de Azure Digital Twins](how-to-authenticate-client.md). 

Para buscarlos, siga [este vínculo](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps) para ir a la página de información general del registro de la aplicación de Azure AD en Azure Portal. En esta página se muestran todos los registros de aplicaciones que se han creado en la suscripción.

Debería ver aparecer el registro de aplicación recién creado en esta lista. Selecciónelo para abrir sus detalles:

:::image type="content" source="media/how-to-set-up-instance/portal/app-important-values.png" alt-text="Vista del portal de los valores importantes para el registro de la aplicación":::

Tome nota del *Id. de la aplicación (cliente)* y el *Id. de directorio (inquilino)* , como se muestra en **su** página. Si usted no es quién va a escribir el código para las aplicaciones cliente, debe compartir estos valores con la persona que lo hará.

## <a name="verify-success"></a>Comprobación de que la operación se ha completado correctamente

Si quiere comprobar la creación de los recursos y los permisos configurados por el script, puede verlos en [Azure Portal](https://portal.azure.com).

### <a name="verify-instance"></a>Comprobación de la instancia

Para comprobar que se ha creado la instancia, vaya a la [página de Azure Digital Twins](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.DigitalTwins%2FdigitalTwinsInstances) en Azure Portal. En esta página se muestran todas las instancias de Azure Digital Twins. Busque el nombre de la instancia recién creada en la lista.

### <a name="verify-user-role-assignment"></a>Comprobación de la asignación de roles de usuario

[!INCLUDE [digital-twins-setup-verify-role-assignment.md](../../includes/digital-twins-setup-verify-role-assignment.md)]

> [!NOTE]
> Recuerde que actualmente el script asigna este rol necesario al mismo usuario que ejecuta el script desde Cloud Shell. Si tiene que asignar este rol a otra persona que vaya a administrar la instancia, puede hacerlo ahora a través de Azure Portal ([instrucciones](how-to-set-up-instance-portal.md#set-up-user-access-permissions)) o la CLI ([instrucciones](how-to-set-up-instance-cli.md#set-up-user-access-permissions)).
>
> También puede usar el portal o la CLI para rehacer su propia asignación de roles si se produjeron problemas con la configuración con scripts.

### <a name="verify-app-registration"></a>Comprobación del registro de la aplicación

[!INCLUDE [digital-twins-setup-verify-app-registration-1.md](../../includes/digital-twins-setup-verify-app-registration-1.md)]

En primer lugar, compruebe que la configuración de permisos de Azure Digital Twins se estableció correctamente en el registro. Para ello, seleccione *Manifiesto* en la barra de menús para ver el código del manifiesto del registro de la aplicación. Desplácese hasta la parte inferior de la ventana de código y busque estos campos en `requiredResourceAccess`. Los valores deben coincidir con los de la siguiente captura de pantalla:

[!INCLUDE [digital-twins-setup-verify-app-registration-2.md](../../includes/digital-twins-setup-verify-app-registration-2.md)]

## <a name="other-possible-steps-for-your-organization"></a>Otros posibles pasos para la organización

[!INCLUDE [digital-twins-setup-additional-requirements.md](../../includes/digital-twins-setup-additional-requirements.md)]

## <a name="next-steps"></a>Pasos siguientes

Pruebe las llamadas individuales de la API REST en su instancia mediante los comandos de la CLI de Azure Digital Twins: 
* [az dt reference](https://docs.microsoft.com/cli/azure/ext/azure-iot/dt?view=azure-cli-latest)
* [*Procedimiento: Uso de la CLI de Azure Digital Twins*](how-to-use-cli.md).

O bien, consulte cómo conectar la aplicación cliente a la instancia mediante la escritura del código de autenticación de la aplicación cliente:
* [*Procedimiento: Escritura de código de autenticación de aplicación*](how-to-authenticate-client.md)
