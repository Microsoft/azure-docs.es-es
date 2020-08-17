---
title: Implementación de Azure Blockchain Workbench (versión preliminar)
description: Cómo implementar Azure Blockchain Workbench (versión preliminar)
ms.date: 07/16/2020
ms.topic: how-to
ms.reviewer: ravastra
ms.custom: references_regions
ms.openlocfilehash: b46a35b45a51d0cc76942c4ca142c4c7792a28b4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87077017"
---
# <a name="deploy-azure-blockchain-workbench-preview"></a>Implementación de Azure Blockchain Workbench (versión preliminar)

Azure Blockchain Workbench (versión preliminar) se implementa mediante una plantilla de solución de Azure Marketplace. La plantilla simplifica la implementación de los componentes necesarios para crear aplicaciones de cadena de bloques. Una vez implementado, Blockchain Workbench proporciona acceso a las aplicaciones cliente para crear y administrar usuarios y aplicaciones de cadena de bloques.

Para más información acerca de los componentes de Blockchain Workbench, consulte [Arquitectura de Azure Blockchain Workbench](architecture.md).

[!INCLUDE [Preview note](./includes/preview.md)]

## <a name="prepare-for-deployment"></a>Preparar la implementación

Blockchain Workbench le permite implementar un libro de contabilidad de cadena de bloques junto con un conjunto de los servicios de Azure correspondientes que se usan más frecuentemente para compilar una aplicación basada en cadena de bloques. La implementación de Blockchain Workbench hace que los siguientes servicios de Azure se aprovisionen en un grupo de recursos de la suscripción de Azure.

* Plan de App Service (Estándar)
* Application Insights
* Event Grid
* Azure Key Vault
* Azure Service Bus
* SQL Database (Estándar S0)
* Cuenta de Azure Storage (Estándar LRS)
* Conjunto de escalado de máquinas virtuales con una capacidad de 1
* Grupo de recursos de Virtual Network (con equilibrador de carga, grupo de seguridad de red, dirección IP pública y red virtual)
* Azure Blockchain Service. Si usa una implementación anterior de Blockchain Workbench, existe la posibilidad de volver a implementar Azure Blockchain Workbench para utilizar Azure Blockchain Service.

La siguiente es una implementación de ejemplo creada en el grupo de recursos **myblockchain**.

![Implementación de ejemplo](media/deploy/example-deployment.png)

El costo de Blockchain Workbench se agrega al costo de los servicios de Azure subyacentes. La información de precios de los servicios de Azure se pueden calcular mediante la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/).

## <a name="prerequisites"></a>Requisitos previos

Azure Blockchain Workbench requiere de registros de aplicación y configuraciones de Azure AD. Puede elegir entre [configurar manualmente](#azure-ad-configuration) Azure AD antes de la implementación o ejecutar un script posterior a la implementación. Si está implementando nuevamente Blockchain Workbench, consulte la [configuración de Azure AD](#azure-ad-configuration) para verificar la configuración de Azure AD.

> [!IMPORTANT]
> El área de trabajo no tiene que implementarse en el mismo inquilino que el que se usa para registrar una aplicación de Azure AD. Se debe implementar en un inquilino donde tenga permisos suficientes para implementar recursos. Para más información sobre los inquilinos de Azure AD, consulte [Obtención de un inquilino de Azure Active Directory](../../active-directory/develop/quickstart-create-new-tenant.md) e [Integración de aplicaciones con Azure Active Directory](../../active-directory/develop/quickstart-register-app.md).

## <a name="deploy-blockchain-workbench"></a>Implementación de Blockchain Workbench

Una vez que se han completado los pasos descritos en los requisitos previos, estará listo para implementar Blockchain Workbench. En las secciones siguientes se describe cómo implementar la plataforma.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione su cuenta en la esquina superior derecha y cambie al inquilino de Azure AD en el que desee implementar Azure Blockchain Workbench.
1. Haga clic en **Crear un recurso** en la esquina superior izquierda de Azure Portal.
1. Seleccione **Cadena de bloques** > **Azure Blockchain Workbench (versión preliminar)** .

    ![Creación de Azure Blockchain Workbench](media/deploy/blockchain-workbench-settings-basic.png)

    | Configuración | Descripción  |
    |---------|--------------|
    | Prefijo de recurso. | Identificador único corto para la implementación. Este valor se utiliza como base para asignar nombres a los recursos. |
    | Nombre de usuario de máquina virtual | El nombre de usuario se utiliza como administrador de todas las máquinas virtuales (VM). |
    | Tipo de autenticación | Seleccione si desea utilizar una contraseña o clave para conectarse a las máquinas virtuales. |
    | Contraseña | La contraseña se usa para conectarse a las máquinas virtuales. |
    | SSH | Use una clave pública RSA en formato de una sola línea con **ssh-rsa** o utilice el formato PEM de varias líneas. Puede generar claves SSH mediante `ssh-keygen` en Linux y OS X o PuTTYGen en Windows. Para más información sobre las claves SSH, consulte [Uso de claves SSH con Windows en Azure](../../virtual-machines/linux/ssh-from-windows.md). |
    | Contraseña de la base de datos y Blockchain | Especifique la contraseña que se utilizará para acceder a la base de datos creada como parte de la implementación. La contraseña debe cumplir tres de los cuatro requisitos siguientes: debe tener una longitud de entre 12 y 72 caracteres, un carácter en minúscula, un carácter en mayúscula, un número y un carácter especial que no sea el signo de número (#), porcentaje (%), coma (,), asterisco (*), comilla inversa (\`), comilla doble ("), comilla simple ('), guion (-) y punto y coma (;). |
    | Región de la implementación | Especifique dónde se van a implementar los recursos de Blockchain Workbench. Para una mejor disponibilidad, el valor debe ser el mismo que el de la ubicación **Región**. No todas las regiones están disponibles durante la versión preliminar. Es posible que las características no estén disponibles en algunas regiones. Azure Blockchain Data Manager está disponible en las siguientes regiones de Azure: Este de EE. UU. y Oeste de Europa.|
    | Suscripción | Especifique la suscripción de Azure que desea usar para la implementación. |
    | Grupos de recursos | Cree un nuevo grupo de recursos seleccionando **Crear nuevo** y especifique un nombre de grupo de recursos único. |
    | Location | Especifique la región en la que desea implementar la plataforma. |

1. Seleccione **Aceptar** para finalizar la sección de configuración básica.

1. En **Configuración avanzada**, elija si desea crear una nueva red de cadena de bloques o usar una red de cadena de bloques de prueba de autoridad existente.

    Para **Crear nueva**:

    La opción *Crear nueva* implementa un libro de contabilidad de cuórum de Azure Blockchain Service con la SKU básica predeterminada.

    ![Configuración avanzada para la nueva red de la cadena de bloques](media/deploy/advanced-blockchain-settings-new.png)

    | Configuración | Descripción  |
    |---------|--------------|
    | Plan de tarifa de Azure Blockchain Service | Elija el nivel de servicio **Básico** o **Estándar** de Azure Blockchain Service que se usa para Blockchain Workbench. |
    | Configuración de Azure Active Directory | Seleccione **Agregar más adelante**.</br>Nota: Si decide [configurar previamente Azure AD](#azure-ad-configuration) o volver a implementar, elija *Agregar ahora*. |
    | Selección de máquina virtual | Seleccione el rendimiento de almacenamiento y el tamaño de máquina virtual preferidos para la red de cadena de bloques. Elija un tamaño más pequeño de máquina virtual, como *Estándar DS1 v2*, si tiene una suscripción con límites de servicio bajos, como el nivel Gratis de Azure. |

    Para **Usar existente**:

    La opción *usar existente* opción le permite especificar una red de cadena de bloques de prueba de autoridad (PoA) de Ethereum. Los puntos de conexión tienen los siguientes requisitos.

   * El punto de conexión debe ser una red de cadena de bloques de prueba de autoridad (PoA) de Ethereum.
   * El punto de conexión debe ser públicamente accesible a través de la red.
   * La red de cadena de bloques PoA debe configurarse para que el precio del gas se establezca en cero.

     > [!NOTE]
     > Las cuentas de Blockchain Workbench no tienen fondos. Si se requieren fondos, habrá un error de transacción.

     ![Configuración avanzada para la red existente de la cadena de bloques](media/deploy/advanced-blockchain-settings-existing.png)

     | Configuración | Descripción  |
     |---------|--------------|
     | Punto de conexión RPC de Ethereum | Proporcione el punto de conexión RPC de una red de cadena de bloques PoA existente. El punto de conexión comienza con https:// o http:// y termina con un número de puerto. Por ejemplo: `http<s>://<network-url>:<port>` |
     | Configuración de Azure Active Directory | Seleccione **Agregar más adelante**.</br>Nota: Si decide [configurar previamente Azure AD](#azure-ad-configuration) o volver a implementar, elija *Agregar ahora*. |
     | Selección de máquina virtual | Seleccione el rendimiento de almacenamiento y el tamaño de máquina virtual preferidos para la red de cadena de bloques. Elija un tamaño más pequeño de máquina virtual, como *Estándar DS1 v2*, si tiene una suscripción con límites de servicio bajos, como el nivel Gratis de Azure. |

1. Seleccione **Revisar y crear** para finalizar con la configuración avanzada.

1. Revise el resumen para comprobar que los parámetros son precisos.

    ![Resumen](media/deploy/blockchain-workbench-summary.png)

1. Seleccione **Crear** para aceptar los términos e implementar Azure Blockchain Workbench.

La implementación puede tardar hasta 90 minutos. Puede usar Azure Portal para supervisar el progreso. En el grupo de recursos recién creado, seleccione **Implementaciones > Información general** para ver el estado de los artefactos implementados.

> [!IMPORTANT]
> Después de la implementación debe completar la configuración de Active Directory. Si eligió **Agregar más adelante**, deberá ejecutar el [script de configuración de Azure AD](#azure-ad-configuration-script).  Si eligió **Agregar ahora**, deberá [configurar la dirección URL de respuesta](#configuring-the-reply-url).

## <a name="blockchain-workbench-web-url"></a>Dirección URL web de Blockchain Workbench

Cuando se haya completado la implementación de Blockchain Workbench, un nuevo grupo de recursos contendrá los recursos de este. Se puede acceder a los servicios de Blockchain Workbench a través de una dirección URL web. Los pasos siguientes muestran cómo recuperar la dirección URL web de la plataforma implementada.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En el panel de navegación de la izquierda, seleccione **Grupos de recursos**.
1. Elija el nombre del grupo de recursos que especificó al implementar Blockchain Workbench.
1. Seleccione el encabezado de columna **TIPO** para ordenar la lista alfabéticamente por tipo.
1. Hay dos recursos del tipo **App Service**. Seleccione el recurso del tipo **App Service** *sin* el sufijo "-api".

    ![Lista de App Service](media/deploy/resource-group-list.png)

1. En **Información general** de App Service, copie el valor **URL** que representa la dirección URL web a la instancia de Blockchain Workbench implementada.

    ![Información esencial de App Service](media/deploy/app-service.png)

Para asociar un nombre de dominio personalizado a Blockchain Workbench, consulte [Configuración de un nombre de dominio personalizado para una aplicación web en Azure App Service mediante Traffic Manager](../../app-service/configure-domain-traffic-manager.md).

## <a name="azure-ad-configuration-script"></a>Script de configuración de Azure AD

Azure AD debe configurarse para completar la implementación de Blockchain Workbench. Usará un script de PowerShell para realizar la configuración.

1. En un explorador, vaya a la [URL web de Blockchain Workbench](#blockchain-workbench-web-url).
1. Podrá ver instrucciones para configurar Azure AD con Cloud Shell. Copie el comando e inicie Cloud Shell.

    ![Inicio del script de AAD](media/deploy/launch-aad-script.png)

1. Elija al inquilino de Azure AD en el que se ha implementado Blockchain Workbench.
1. En el entorno PowerShell de Cloud Shell, pegue y ejecute el comando.
1. Cuando se le solicite, escriba al inquilino de Azure AD que desea usar para Blockchain Workbench. Este será el inquilino que contiene los usuarios de Blockchain Workbench.

    > [!IMPORTANT]
    > El usuario autenticado requiere permisos para crear registros de aplicación de Azure AD y conceder permisos de aplicación delegados en el inquilino. Tendrá que solicitar al administrador del inquilino que ejecute el script de configuración de Azure AD o cree un nuevo inquilino.

    ![Ingreso al inquilino de Azure AD](media/deploy/choose-tenant.png)

1. Se le pedirá que se autentique en el inquilino de Azure AD mediante un explorador. Abra la dirección URL web en un explorador, escriba el código y autentíquese.

    ![Autenticación con código](media/deploy/authenticate.png)

1. El script genera varios mensajes de estado. Obtendrá un mensaje de estado **CORRECTO** si el inquilino se ha aprovisionado correctamente.
1. Navegue a la URL de Blockchain Workbench. Deberá dar su consentimiento para conceder permisos de lectura en el directorio. Esto permite que la aplicación web de Blockchain Workbench tenga acceso a los usuarios en el inquilino. Si es el administrador de inquilinos, puede dar su consentimiento por toda la organización. Esta opción acepta el consentimiento para todos los usuarios en el inquilino. En caso contrario, a cada usuario se le pedirá su consentimiento la primera vez que use la aplicación web de Blockchain Workbench.
1. Seleccione **Aceptar** para dar su consentimiento.

     ![Consentimiento para leer los perfiles de usuarios](media/deploy/graph-permission-consent.png)

1. Después de dar su consentimiento, puede usar la aplicación web de Blockchain Workbench.

Ha completado la implementación de Azure Blockchain Workbench. Vea [Pasos siguientes](#next-steps) para obtener sugerencias sobre cómo empezar a usar la implementación.

## <a name="azure-ad-configuration"></a>Configuración de Azure AD

Si decide configurar o comprobar manualmente la configuración de Azure AD antes de la implementación, complete todos los pasos de esta sección. Si prefiere establecer automáticamente la configuración de Azure AD, utilice el [script de configuración de Azure AD](#azure-ad-configuration-script) después de implementar Blockchain Workbench.

### <a name="blockchain-workbench-api-app-registration"></a>Registro de aplicación de API de Blockchain Workbench

La implementación de Blockchain Workbench requiere el registro de una aplicación de Azure AD. Necesita un inquilino de Azure Active Directory (Azure AD) para registrar la aplicación. Puede usar un inquilino existente o crear uno nuevo. Si va a usar un inquilino de Azure AD ya existente, necesitará suficientes permisos para registrar aplicaciones, otorgar permisos de Graph API y permitir el acceso a los invitados dentro de un inquilino de Azure AD. Si no tiene permisos suficientes en un inquilino existente de Azure AD, cree un inquilino.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione su cuenta en la esquina superior derecha y cambie al inquilino de Azure AD que desee. El inquilino debe ser el del administrador de la suscripción en la que está implementado Azure Blockchain Workbench y en la que tenga permisos suficientes para registrar aplicaciones.
1. En el panel de navegación izquierdo, seleccione el servicio **Azure Active Directory**. Seleccione **Registros de aplicaciones** > **Nuevo registro**.

    ![Registro de aplicación](media/deploy/app-registration.png)

1. Proporcione un **Nombre** para mostrar y elija **Solo las cuentas de este directorio organizativo**.

    ![Crear el registro de aplicaciones](media/deploy/app-registration-create.png)

1. Seleccione **Registrar** para registrar la aplicación de Azure AD.

### <a name="modify-manifest"></a>Modificación del manifiesto

A continuación, debe modificar el manifiesto para que use los roles de aplicación en Azure AD con el fin de especificar los administradores de Blockchain Workbench.  Para más información acerca de los manifiestos de aplicación, consulte [Manifiesto de aplicación de Azure Active Directory](../../active-directory/develop/reference-app-manifest.md).

1. Se requiere un GUID para el manifiesto. Se puede generar un GUID mediante el comando de PowerShell `[guid]::NewGuid()` o el cmdlet `New-GUID`. Otra opción es usar un sitio web generador de GUID.
1. Para la aplicación que se ha registrado, seleccione **Manifiesto** en la sección **Administrar**.
1. Después, actualice la sección **appRoles** del manifiesto. Reemplace `"appRoles": []` por el JSON proporcionado. Asegúrese de reemplazar el valor del campo `id` por el identificador único global que generó.
    ![Editar manifiesto](media/deploy/edit-manifest.png)

    ``` json
    "appRoles": [
         {
           "allowedMemberTypes": [
             "User",
             "Application"
           ],
           "displayName": "Administrator",
           "id": "<A unique GUID>",
           "isEnabled": true,
           "description": "Blockchain Workbench administrator role allows creation of applications, user to role assignments, etc.",
           "value": "Administrator"
         }
       ],
    ```

    > [!IMPORTANT]
    > El valor **Administrador** es necesario para identificar los administradores de Blockchain Workbench.

1. En el manifiesto, cambie también el valor de **Oauth2AllowImplicitFlow** a **true**.

    ``` json
    "oauth2AllowImplicitFlow": true,
    ```

1. Seleccione **Guardar** para guardar los cambios en el manifiesto.

### <a name="add-graph-api-required-permissions"></a>Incorporación de los permisos necesarios de Graph API

La aplicación de API necesita solicitar permiso del usuario para acceder al directorio. Establezca el siguiente permiso necesario para la aplicación de API:

1. En el registro de aplicación de la *API Blockchain*, seleccione **Permisos de la API**. De forma predeterminada, se agrega el permiso de Graph API **User.Read**.
1. La aplicación Workbench necesita acceso de lectura a la información de perfil básica de los usuarios. En *Permisos configurados*, seleccione **Agregar un permiso**. En **API de Microsoft**, seleccione **Microsoft Graph**.
1. Dado que la aplicación Workbench usa las credenciales del usuario autenticado, seleccione **Permisos delegados**.
1. En la categoría *Usuario*, seleccione el permiso **User.ReadBasic.All**.

    ![Configuración de registro de aplicación de Azure AD que muestra la incorporación del permiso delegado User.ReadBasic.All de Microsoft Graph](media/deploy/add-graph-user-permission.png)

    Seleccione **Agregar permisos**.

1. En *Permisos configurados*, seleccione **Conceder consentimiento del administrador** para el dominio y luego **Sí** en el mensaje de comprobación.

   ![Concesión de permisos](media/deploy/client-app-grant-permissions.png)

   La concesión de permisos permite a Blockchain Workbench acceder a los usuarios del directorio. Se requiere el permiso de lectura para buscar y agregar miembros a Blockchain Workbench.

### <a name="get-application-id"></a>Obtención del identificador de la aplicación

Se necesita el identificador de la aplicación y la información del inquilino para la implementación. Recopile y almacene la información para su uso durante la implementación.

1. Para la aplicación que se ha registrado, seleccione **Información general**.
1. Copie y almacene el valor de **Id. de aplicación** para su uso posterior durante la implementación.

    ![Propiedades de la aplicación de API](media/deploy/app-properties.png)

    | Configuración para almacenar  | Uso en la implementación |
    |------------------|-------------------|
    | Id. de aplicación (cliente) | Configuración de Azure Active Directory > Identificador de aplicación |

### <a name="get-tenant-domain-name"></a>Obtención del nombre de dominio del inquilino

Recopile y almacene el nombre de dominio del inquilino de Active Directory donde se registran las aplicaciones.

En el panel de navegación izquierdo, seleccione el servicio **Azure Active Directory**. Seleccione **Nombres de dominio personalizados**. Copie y almacene el nombre de dominio.

![Nombre de dominio](media/deploy/domain-name.png)

### <a name="guest-user-settings"></a>Configuración de usuario invitado

Si tiene usuarios invitados en su inquilino de Azure AD, siga los pasos adicionales para asegurarse de que la asignación y administración de usuarios de Blockchain Workbench funciona correctamente.

1. Cambie a su inquilino de Azure AD y seleccione **Azure Active Directory > Configuración de usuario > Administrar la configuración de colaboración externa**.
1. Establezca la opción **Guest user permissions are limited** (Los permisos de los usuarios invitados están limitados) en **No**.
    ![Configuración de colaboración externa](media/deploy/user-collaboration-settings.png)

## <a name="configuring-the-reply-url"></a>Configuración de la dirección URL de respuesta

Una vez que se ha implementado Azure Blockchain Workbench, debe configurar la **dirección URL de respuesta** de la aplicación cliente de Azure Active Directory (Azure AD) correspondiente a la dirección URL web de la instancia de Blockchain Workbench implementada.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Compruebe que se encuentra en el inquilino en el que se ha registrado la aplicación cliente de Azure AD.
1. En el panel de navegación izquierdo, seleccione el servicio **Azure Active Directory**. Seleccione **App registrations** (Registros de aplicaciones).
1. Seleccione la aplicación cliente de Azure AD que registró en la sección de requisitos previos.
1. Seleccione **Autenticación**.
1. Especifique la dirección URL web principal de la implementación de Azure Blockchain Workbench que se ha recuperado en la sección [Dirección URL web de Azure Blockchain Workbench](#blockchain-workbench-web-url). La dirección URL de respuesta tiene como prefijo `https://`. Por ejemplo: `https://myblockchain2-7v75.azurewebsites.net`

    ![URL de respuesta de autenticación](media/deploy/configure-reply-url.png)

1. En la sección **Configuración avanzada**, active **Tokens de acceso** y **Tokens de id.** .

    ![Configuración avanzada de autenticación](media/deploy/authentication-advanced-settings.png)

1. Seleccione **Guardar** para actualizar el registro del cliente.

## <a name="remove-a-deployment"></a>Eliminación de una implementación

Cuando ya no se necesita una implementación, puede quitarla eliminando el grupo de recursos de Blockchain Workbench.

1. En Azure Portal, vaya a **Grupo de recursos** en el panel de navegación izquierdo y seleccione el grupo de recursos que desea eliminar.
1. Seleccione **Eliminar grupo de recursos**. Compruebe la eliminación escribiendo el nombre del grupo de recursos y seleccionando **Eliminar**.

    ![Eliminación de un grupo de recursos](media/deploy/delete-resource-group.png)

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha implementado Azure Blockchain Workbench. Para aprender a crear una aplicación de cadena de bloques, continúe con el siguiente artículo de procedimientos.

> [!div class="nextstepaction"]
> [Creación de una aplicación de cadena de bloques en Azure Blockchain Workbench](create-app.md)
