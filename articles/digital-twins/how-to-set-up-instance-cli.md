---
title: Configuración de una instancia y de la autenticación (CLI)
titleSuffix: Azure Digital Twins
description: Descubra cómo configurar una instancia del servicio Azure Digital Twins mediante la CLI.
author: baanders
ms.author: baanders
ms.date: 7/23/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: e9a78690128c2406277ab4e8fb6e6e4625d2787f
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89280101"
---
# <a name="set-up-an-azure-digital-twins-instance-and-authentication-cli"></a>Configuración de una instancia de Azure Digital Twins y autenticación (CLI)

[!INCLUDE [digital-twins-setup-selector.md](../../includes/digital-twins-setup-selector.md)]

En este artículo se describen los pasos para **configurar una nueva instancia de Azure Digital Twins**, incluidas la creación de la instancia y la configuración de la autenticación. Después de completar este artículo, tendrá una instancia de Azure Digital Twins lista para empezar a programar.

En esta versión de este artículo se realizan los pasos manualmente, uno por uno, mediante la CLI.
* Para seguir estos pasos manualmente con Azure Portal, consulte la versión del portal de este artículo: [*Procedimiento: Configuración de una instancia y autenticación (Azure Portal)* ](how-to-set-up-instance-portal.md).
* Para ejecutar una configuración automatizada mediante un script de implementación de ejemplo, consulte la versión con scripts de este artículo: [*Procedimiento: Configuración de una instancia y autenticación (con scripts)* ](how-to-set-up-instance-scripted.md).

[!INCLUDE [digital-twins-setup-steps-prereq.md](../../includes/digital-twins-setup-steps-prereq.md)]
[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="set-up-cloud-shell-session"></a>Configuración de una sesión de Cloud Shell
[!INCLUDE [Cloud Shell for Azure Digital Twins](../../includes/digital-twins-cloud-shell.md)]

## <a name="create-the-azure-digital-twins-instance"></a>Creación de una instancia de Azure Digital Twins

En esta sección, **creará una nueva instancia de Azure Digital Twins** mediante el comando de Cloud Shell. Tendrá que proporcionar:
* Un grupo de recursos donde se realizará la implementación. Si todavía no tiene un grupo de recursos existente, puede crear uno ahora con este comando:
    ```azurecli
    az group create --location <region> --name <name-for-your-resource-group>
    ```
* Una región para la implementación. Para ver qué regiones admiten Azure Digital Twins, visite [*Productos de Azure disponibles por región*](https://azure.microsoft.com/global-infrastructure/services/?products=digital-twins).
* Un nombre para la instancia. El nombre de la nueva instancia debe ser único dentro de la región de la suscripción (lo que significa que si la suscripción tiene otra instancia de Azure Digital Twins en la región que ya usa el nombre elegido, se le solicitará que elija otro nombre).

Use estos valores en el comando siguiente para crear la instancia:

```azurecli
az dt create --dt-name <name-for-your-Azure-Digital-Twins-instance> -g <your-resource-group> -l <region>
```

### <a name="verify-success-and-collect-important-values"></a>Comprobación de que la operación es correcta y recopilación de valores importantes

Si la instancia se creó correctamente, el resultado en Cloud Shell tiene un aspecto similar al siguiente, donde se muestra información sobre el recurso que ha creado:

:::image type="content" source="media/how-to-set-up-instance/cloud-shell/create-instance.png" alt-text="Ventana Comandos con la creación correcta del grupo de recursos y la instancia de Azure Digital Twins":::

Anote *hostName*, *name* y *resourceGroup* del resultado de la instancia de Azure Digital Twins. Estos son todos los valores importantes que puede necesitar a medida que sigue trabajando con la instancia de Azure Digital Twins para configurar la autenticación y los recursos de Azure relacionados. Si otros usuarios van a programar en la instancia, debe compartirlos con ellos.

> [!TIP]
> Puede ver estas propiedades, junto con todas las propiedades de la instancia, en cualquier momento si ejecuta `az dt show --dt-name <your-Azure-Digital-Twins-instance>`.

Ahora tiene lista una instancia de Azure Digital Twins. A continuación, debe proporcionar los permisos de usuario de Azure adecuados para administrarla.

## <a name="set-up-user-access-permissions"></a>Configuración de permisos de acceso de usuarios

[!INCLUDE [digital-twins-setup-role-assignment.md](../../includes/digital-twins-setup-role-assignment.md)]

Use el comando siguiente para asignar el rol (debe ser ejecutado por un usuario con [suficientes permisos](#prerequisites-permission-requirements) en la suscripción de Azure). El comando requiere que pase el *nombre principal de usuario* en la cuenta de Azure AD del usuario al que se debe asignar el rol. En la mayoría de los casos, coincidirá con el correo electrónico del usuario en la cuenta de Azure AD.

```azurecli
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<Azure-AD-user-principal-name-of-user-to-assign>" --role "Azure Digital Twins Owner (Preview)"
```

El resultado de este comando es la información de salida acerca de la asignación de roles que se ha creado.

> [!NOTE]
> Si este comando devuelve un error que indica que la CLI **no puede encontrar el usuario ni la entidad de servicio en la base de datos de grafos**:
>
> Asigne el rol mediante el *id. de objeto* del usuario en su lugar. Esto puede ocurrir para los usuarios de [cuentas de Microsoft (MSA)](https://account.microsoft.com/account) personales. 
>
> Use la [página de Azure Portal de usuarios de Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/AllUsers) para seleccionar la cuenta de usuario y abrir los detalles. Copie el *id. de objeto* del usuario:
>
> :::image type="content" source="media/includes/user-id.png" alt-text="Vista de la página de usuario en Azure Portal en la que se resalta el GUID en el campo Id. de objeto" lightbox="media/includes/user-id.png":::
>
> A continuación, repita el comando de lista de asignación de roles con el *id. de objeto* del usuario para el parámetro `assignee` anterior.

### <a name="verify-success"></a>Comprobación de que la operación se ha completado correctamente

[!INCLUDE [digital-twins-setup-verify-role-assignment.md](../../includes/digital-twins-setup-verify-role-assignment.md)]

Ahora tiene lista una instancia de Azure Digital Twins y los permisos asignados para administrarla. A continuación, configurará los permisos para que una aplicación cliente obtenga el acceso a ella.

## <a name="set-up-access-permissions-for-client-applications"></a>Configuración de los permisos de acceso para aplicaciones cliente

[!INCLUDE [digital-twins-setup-app-registration.md](../../includes/digital-twins-setup-app-registration.md)]

Para crear un registro de la aplicación, debe proporcionar los identificadores de recursos de las API de Azure Digital Twins y los permisos de línea de base para la API.

En el directorio de trabajo, cree un archivo nuevo y escriba el siguiente fragmento de código JSON para configurar estos detalles: 

```json
[{
    "resourceAppId": "0b07f429-9f4b-4714-9392-cc5e8e80c8b0",
    "resourceAccess": [
     {
       "id": "4589bd03-58cb-4e6c-b17f-b580e39652f8",
       "type": "Scope"
     }
    ]
}]
``` 

Guarde este archivo como _**manifest.json**_.

> [!NOTE] 
> Hay algunos lugares en los que se puede usar una cadena `https://digitaltwins.azure.net` legible y "descriptiva" para el identificador de la aplicación de recursos de Azure Digital Twins en lugar del GUID `0b07f429-9f4b-4714-9392-cc5e8e80c8b0`. En particular, muchos ejemplos a lo largo de este conjunto de documentación usan la autenticación con la biblioteca MSAL y, para ello, se puede usar la cadena descriptiva. Sin embargo, durante este paso de creación del registro de la aplicación, es necesario el formato de GUID del identificador, tal y como se muestra arriba. 

A continuación, cargará este archivo a Cloud Shell. En la ventana de Cloud Shell, haga clic en el icono "Cargar/Descargar archivos" y elija "Cargar".

:::image type="content" source="media/how-to-set-up-instance/cloud-shell/cloud-shell-upload.png" alt-text="Ventana de Cloud Shell que muestra la selección de la opción Cargar":::
Vaya al archivo *manifest.json* que acaba de crear y seleccione "Abrir".

A continuación, ejecute el siguiente comando para crear un registro de aplicación, con una dirección URL de respuesta *Cliente público/nativo (móvil y escritorio)* de `http://localhost`. Reemplace los marcadores de posición según sea necesario:

```azurecli
az ad app create --display-name <name-for-your-app-registration> --native-app --required-resource-accesses manifest.json --reply-url http://localhost
```

Este es un extracto de la salida de este comando y muestra información sobre el registro que ha creado:

:::image type="content" source="media/how-to-set-up-instance/cloud-shell/new-app-registration.png" alt-text="Salida de Cloud Shell del nuevo registro de aplicaciones de Azure AD":::

### <a name="verify-success"></a>Comprobación de que la operación se ha completado correctamente

[!INCLUDE [digital-twins-setup-verify-app-registration-1.md](../../includes/digital-twins-setup-verify-app-registration-1.md)]

En primer lugar, compruebe que la configuración del archivo *manifest.json* cargado se estableció correctamente en el registro. Para ello, seleccione *Manifiesto* en la barra de menús para ver el código del manifiesto del registro de la aplicación. Desplácese hasta la parte inferior de la ventana de código y busque los campos del archivo *manifest.json* en `requiredResourceAccess`:

[!INCLUDE [digital-twins-setup-verify-app-registration-2.md](../../includes/digital-twins-setup-verify-app-registration-2.md)]

### <a name="collect-important-values"></a>Recopilación de valores importantes

A continuación, seleccione *Información general* en la barra de menús para ver los detalles del registro de la aplicación:

:::image type="content" source="media/how-to-set-up-instance/portal/app-important-values.png" alt-text="Vista del portal de los valores importantes para el registro de la aplicación":::

Tome nota del *Id. de la aplicación (cliente)* y el *Id. de directorio (inquilino)* , como se muestra en **su** página. Estos valores se necesitarán más adelante para [autenticar una aplicación cliente en las API de Azure Digital Twins](how-to-authenticate-client.md). Si usted no es quien va a escribir el código para esas aplicaciones, debe compartir estos valores con la persona que lo hará.

### <a name="other-possible-steps-for-your-organization"></a>Otros posibles pasos para la organización

[!INCLUDE [digital-twins-setup-additional-requirements.md](../../includes/digital-twins-setup-additional-requirements.md)]

## <a name="next-steps"></a>Pasos siguientes

Pruebe las llamadas individuales de la API de REST en su instancia mediante los comandos de la CLI de Azure Digital Twins: 
* [Referencia de az dt](https://docs.microsoft.com/cli/azure/ext/azure-iot/dt?view=azure-cli-latest)
* [*Procedimiento: Uso de la CLI de Azure Digital Twins*](how-to-use-cli.md).

O bien, consulte cómo conectar la aplicación cliente a la instancia mediante la escritura del código de autenticación de la aplicación cliente:
* [*Procedimiento: Escritura de código de autenticación de aplicación*](how-to-authenticate-client.md)
