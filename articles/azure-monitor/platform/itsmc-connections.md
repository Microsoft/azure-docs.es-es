---
title: IT Service Management Connector en Azure Monitor
description: En este artículo, se proporciona información sobre cómo conectar los productos y servicios de ITSM al Conector de Administración de servicios de TI (ITSMC) en Azure Monitor para supervisar y administrar de forma centralizada los elementos de trabajo de ITSM.
ms.subservice: logs
ms.topic: conceptual
author: nolavime
ms.author: v-jysur
ms.date: 05/12/2020
ms.openlocfilehash: 9d037fa0faa1419e4cd1b600eea1b3b3eb0a29c0
ms.sourcegitcommit: 94c750edd4d755d6ecee50ac977328098a277479
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90058774"
---
# <a name="connect-itsm-productsservices-with-it-service-management-connector"></a>Conectar productos o servicios de ITSM con el Conector de Administración de servicios de TI
En este artículo se proporciona información sobre cómo configurar la conexión entre los productos y servicios de ITSM y el Conector de Administración de servicios de TI (ITSMC) en Log Analytics para administrar de forma centralizada los elementos de trabajo. Para obtener más información sobre ITSMC, vea [Información general](./itsmc-overview.md).

Se admiten los siguientes productos y servicios de ITSM. Seleccione un producto para ver información detallada sobre cómo conectarlo a ITSMC.

- [System Center Service Manager](#connect-system-center-service-manager-to-it-service-management-connector-in-azure)
- [ServiceNow](#connect-servicenow-to-it-service-management-connector-in-azure)
- [Provance](#connect-provance-to-it-service-management-connector-in-azure)
- [Cherwell](#connect-cherwell-to-it-service-management-connector-in-azure)

> [!NOTE]
> 
> A partir del 1 de octubre de 2020, las integraciones de ITSM de Cherwell y Provance con alertas de Azure ya no se habilitarán para los clientes nuevos. No se admitirán las nuevas conexiones de ITSM. 
> Se admitirán las conexiones de ITSM existentes.

## <a name="connect-system-center-service-manager-to-it-service-management-connector-in-azure"></a>Conectar System Center Service Manager con el Conector de Administración de servicios de TI en Azure

En las secciones siguientes se proporcionan detalles sobre cómo conectar su producto de System Center Service Manager con ITSMC en Azure.

### <a name="prerequisites"></a>Prerrequisitos

Asegúrese de que se cumplen los siguientes requisitos previos:

- ITSMC instalado Más información: [Agregar la solución IT Service Management Connector](./itsmc-overview.md#adding-the-it-service-management-connector-solution).
- Se implementa y se configura la aplicación web de Service Manager (aplicación web). [Aquí](#create-and-deploy-service-manager-web-app-service) se puede obtener información sobre la aplicación web.
- Conexión híbrida creada y configurada Más información: [Configuración de la conexión híbrida](#configure-the-hybrid-connection).
- Versiones admitidas de Service Manager:  2012 R2 o 2016.
- Rol de usuario:  [operador avanzado](/previous-versions/system-center/service-manager-2010-sp1/ff461054(v=technet.10)).

> [!NOTE]
> 
> El conector de ITSM solo se puede conectar a instancias de ServiceNow basadas en la nube. Actualmente no se admiten instancias locales de ServiceNow.

### <a name="connection-procedure"></a>Procedimiento de conexión

Use el siguiente procedimiento para conectar la instancia de System Center Service Manager con ITSMC:

1. En Azure Portal, vaya a **Todos los recursos** y busque **ServiceDesk(YourWorkspaceName)** .

2.  En **ORÍGENES DE DATOS DEL ÁREA DE TRABAJO**, haga clic en **Conexiones de ITSM**.

    ![Nueva conexión](media/itsmc-connections/add-new-itsm-connection.png)

3. En la parte superior del panel derecho, haga clic en **Agregar**.

4. Proporcione la información tal como se describe en la tabla siguiente y haga clic en **Aceptar** para crear la conexión.

> [!NOTE]
> 
> Todos estos parámetros son obligatorios.

| **Campo** | **Descripción** |
| --- | --- |
| **Nombre de la conexión**   | Escriba un nombre para la instancia de System Center Service Manager que quiere conectar con ITSMC.  Usará este nombre más adelante cuando configure los elementos de trabajo en el análisis de registros detallados de la instancia o vista. |
| **Tipo de asociado**   | Seleccione **System Center Service Manager**. |
| **Dirección URL del servidor**   | Escriba la dirección URL de la aplicación web de Service Manager. [Aquí](#create-and-deploy-service-manager-web-app-service) podrá obtener más información sobre la aplicación web de Service Manager.
| **Id. de cliente**   | Escriba el identificador de cliente que ha generado (mediante el script automático) para autenticar la aplicación web. [Aquí](./itsmc-service-manager-script.md) podrá obtener más información sobre el script automatizado.|
| **Secreto de cliente**   | Escriba el secreto de cliente generado para este identificador.   |
| **Sincronización de datos**   | Seleccione los elementos de trabajo de Service Manager que quiere sincronizar mediante ITSMC.  Estos elementos de trabajo se importan en Log Analytics. **Opciones:**  incidentes, solicitudes de cambio.|
| **Ámbito de sincronización de datos** | Escriba el número de días pasados de los que desea los datos. **Límite máximo**: 120 días. |
| **Creación de un elemento de configuración de solución ITSM** | Seleccione esta opción si desea crear los elementos de configuración en el producto ITSM. Al seleccionarla, Log Analytics crea los elementos de configuración afectados como elementos de configuración (si no existen) en el sistema ITSM compatible. **Valor predeterminado**: deshabilitado. |

![Conexión de Service Manager](media/itsmc-connections/service-manager-connection.png)

**Cuando se ha conectado y sincronizado correctamente**:

- Los elementos de trabajo seleccionados de Service Manager se importan en Azure **Log Analytics**. Puede ver el resumen de estos elementos de trabajo en el icono de IT Service Management Connector.

- Puede crear incidentes a partir de alertas de Log Analytics, de entradas de registros o de alertas de Azure en esta instancia de Service Manager.


Más información: [Creación de elementos de trabajo de ITSM a partir de alertas de Azure](./itsmc-overview.md#create-itsm-work-items-from-azure-alerts).

### <a name="create-and-deploy-service-manager-web-app-service"></a>Creación e implementación del servicio de aplicaciones web de Service Manager

Para conectar la instancia de Service Manager local con ITSMC en Azure, Microsoft ha creado una aplicación web de Service Manager en GitHub.

Para configurar la aplicación web de ITSM para Service Manager, haga lo siguiente:

- **Implemente la aplicación web**: implemente la aplicación web, establezca las propiedades y autentíquese con Azure AD. Puede implementar la aplicación web mediante el [script automatizado ](./itsmc-service-manager-script.md) que Microsoft ha proporcionado.
- **Configure la conexión híbrida** - [Configure esta conexión](#configure-the-hybrid-connection), manualmente.

#### <a name="deploy-the-web-app"></a>Implementación de la aplicación web
Use el [script](./itsmc-service-manager-script.md) automatizado para implementar la aplicación web, establecer las propiedades y autenticarse con Azure AD.

Ejecute el script proporcionando los siguientes detalles necesarios:

- Detalles de la suscripción de Azure
- Definición de un nombre de grupo de recursos
- Location
- Detalles del servidor de Service Manager (nombre del servidor, dominio, nombre de usuario y contraseña)
- Prefijo de nombre de sitio de la aplicación web
- Espacio de nombres de ServiceBus.

El script crea la aplicación web con el nombre que especificó (junto con algunas cadenas adicionales para hacerlo único). Genera la **dirección URL de la aplicación web**, el **id. de cliente** y el **secreto de cliente**.

Guarde los valores, ya que se usan al crear una conexión con ITSMC.

**Comprobación de la instalación de la aplicación web**

1. Vaya a **Azure Portal** > **Recursos**.
2. Seleccione la aplicación web y haga clic en **Configuración** > **Configuración de la aplicación**.
3. Confirme la información sobre la instancia de Service Manager que ha proporcionado en el momento de la implementación de la aplicación a través del script.

### <a name="configure-the-hybrid-connection"></a>Configuración de la conexión híbrida

Use el procedimiento siguiente para configurar la conexión híbrida que conecta la instancia de Service Manager con ITSMC en Azure.

1. Busque la aplicación web de Service Manager, **Recursos de Azure**.
2. Haga clic en **Configuración** > **Redes**.
3. En **Conexiones híbridas**, haga clic en **Configure los puntos de conexión de la conexión híbrida**.

    ![Redes de conexión híbrida](media/itsmc-connections/itsmc-hybrid-connection-networking-and-end-points.png)
4. En la hoja **Conexiones híbridas**, haga clic en **Agregar conexión híbrida**.

    ![Adición de conexión híbrida](media/itsmc-connections/itsmc-new-hybrid-connection-add.png)

5. En la hoja **Agregar conexiones híbridas**, haga clic en **Crear conexión híbrida nueva**.

    ![Conexión híbrida nueva](media/itsmc-connections/itsmc-create-new-hybrid-connection.png)

6. Escriba los valores siguientes:

   - **Nombre del punto de conexión**: especifique un nombre para la conexión híbrida nueva.
   - **Host del punto de conexión**: nombre de dominio completo del servidor de administración de Service Manager.
   - **Puerto del punto de conexión**: tipo 5724
   - **Espacio de nombres de Service Bus**: use un espacio de nombres de Service Bus existente o cree uno.
   - **Ubicación**: seleccione la ubicación.
   - **Name**: especifique un nombre para la instancia de Service Bus si lo está creando.

     ![Valores de la conexión híbrida](media/itsmc-connections/itsmc-new-hybrid-connection-values.png)
6. Haga clic en **Aceptar** para cerrar la hoja **Crear conexión híbrida** y empiece a crearla.

    Una vez creada, se muestra debajo de la hoja.

7. Cuando se crea la conexión híbrida, seleccione la conexión y haga clic en **Agregar conexión híbrida seleccionada**.

    ![Conexión híbrida nueva](media/itsmc-connections/itsmc-new-hybrid-connection-added.png)

#### <a name="configure-the-listener-setup"></a>Configuración del programa de instalación del agente de escucha

Utilice el procedimiento siguiente para configurar el programa de instalación del agente de escucha para la conexión híbrida.

1. En la hoja **Conexiones híbridas**, haga clic en **Descargar administrador de conexión** e instálelo en el equipo donde se ejecuta la instancia de System Center Service Manager.

    Una vez finalizada la instalación, la opción **Hybrid Connection Manager UI** (IU de administración de conexión híbrida) está disponible en el menú **Iniciar**.

2. Haga clic en **Hybrid Connection Manager UI** (IU de administración de conexión híbrida), se le pedirán las credenciales de Azure.

3. Inicie sesión con sus credenciales de Azure y seleccione la suscripción en la que se creó la conexión híbrida.

4. Haga clic en **Save**(Guardar).

La conexión híbrida se ha conectado correctamente.

![conexión híbrida correcta](media/itsmc-connections/itsmc-hybrid-connection-listener-set-up-successful.png)
> [!NOTE]
> 
> Después de crear la conexión híbrida, compruebe y pruebe la conexión visitando la aplicación web de la instancia de Service Manager implementado. Asegúrese de que la conexión es correcta antes de intentar conectarse con ITSMC en Azure.

En la siguiente imagen de ejemplo se muestran los detalles de una conexión correcta:

![Prueba de conexión híbrida](media/itsmc-connections/itsmc-hybrid-connection-test.png)

## <a name="connect-servicenow-to-it-service-management-connector-in-azure"></a>Conectar ServiceNow al Conector de Administración de servicios de TI en Azure

En las secciones siguientes se proporcionan detalles sobre cómo conectar su producto de ServiceNow a ITSMC en Azure.

### <a name="prerequisites"></a>Prerrequisitos
Asegúrese de que se cumplen los siguientes requisitos previos:
- ITSMC instalado Más información: [Agregar la solución IT Service Management Connector](./itsmc-overview.md#adding-the-it-service-management-connector-solution).
- Versiones compatibles con ServiceNow: Orlando, Nueva York, Madrid, Londres, Kingston, Yakarta, Estambul, Helsinki, Ginebra.
> [!NOTE]
> ITSMC admite solo la oferta de SaaS oficial de ServiceNow. No se admiten implementaciones privadas de ServiceNow. 

**Los administradores de ServiceNow deben realizar lo siguiente en la instancia de ServiceNow:** :
- Generar el identificador y el secreto del cliente para el producto de ServiceNow. Para más información sobre cómo generar el identificador y el secreto de cliente, consulte la siguiente información según sea necesario:

    - [Configuración de OAuth para Orlando](https://docs.servicenow.com/bundle/orlando-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
    - [Configuración de OAuth para Nueva York](https://docs.servicenow.com/bundle/newyork-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
    - [Configuración de OAuth para Madrid](https://docs.servicenow.com/bundle/madrid-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
    - [Configuración de OAuth para Londres](https://docs.servicenow.com/bundle/london-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
    - [Configuración de OAuth para Kingston](https://docs.servicenow.com/bundle/kingston-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
    - [Configuración de OAuth para Yakarta](https://docs.servicenow.com/bundle/jakarta-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
    - [Configuración de OAuth para Estambul](https://docs.servicenow.com/bundle/istanbul-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
    - [Configuración de OAuth para Helsinki](https://docs.servicenow.com/bundle/helsinki-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
    - [Configuración de OAuth para Ginebra](https://docs.servicenow.com/bundle/geneva-servicenow-platform/page/administer/security/task/t_SettingUpOAuth.html)
> [!NOTE]
> Como parte de la definición de "Configuración de OAuth", se recomienda:
>
> 1) **Actualizar la duración del token de actualización a 90 días (7 776 000 segundos).** Como parte de la [Configuración de OAuth](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fdocs.servicenow.com%2Fbundle%2Fnewyork-platform-administration%2Fpage%2Fadminister%2Fsecurity%2Ftask%2Ft_SettingUpOAuth.html&data=02%7C01%7CNoga.Lavi%40microsoft.com%7C2c6812e429a549e71cdd08d7d1b148d8%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637208431696739125&sdata=Q7mF6Ej8MCupKaEJpabTM56EDZ1T8vFVyihhoM594aA%3D&reserved=0) en la fase 2: [Crear un punto de conexión para que los clientes accedan a la instancia](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fdocs.servicenow.com%2Fbundle%2Fnewyork-platform-administration%2Fpage%2Fadminister%2Fsecurity%2Ftask%2Ft_CreateEndpointforExternalClients.html&data=02%7C01%7CNoga.Lavi%40microsoft.com%7C2c6812e429a549e71cdd08d7d1b148d8%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637208431696749123&sdata=hoAJHJAFgUeszYCX1Q%2FXr4N%2FAKiFcm5WV7mwR2UqeWA%3D&reserved=0) Después de la definición del punto de conexión, en la hoja ServiceNow, busque OAuth del sistema y, a continuación, seleccione Registro de aplicaciones. Seleccione el nombre de OAuth que se definió y actualice el campo de duración del token de actualización a 7 776 000 (90 días expresados en segundos).
> Finalmente, haga clic en Actualizar.
> 2) **Se recomienda establecer un procedimiento interno para asegurarse de que la conexión permanezca activa** según la duración del token de actualización para actualizar el token. Asegúrese de realizar las siguientes operaciones antes de la fecha de expiración esperada del token de actualización (se recomienda un par de días antes de que expire la duración del token de actualización):
>
> 1. [Completar un proceso de sincronización manual para la configuración del conector de ITSM](./itsmc-resync-servicenow.md)
> 2. Revocar el token de actualización anterior, ya que no se recomienda mantener las claves antiguas por motivos de seguridad. En la hoja ServiceNow, busque System OAuth (OAuth del sistema) y seleccione Manage Tokens (Administrar tokens). Elija el token antiguo de la lista según el nombre de OAuth y la fecha de expiración.
> ![Definición de OAuth del sistema de nieve](media/itsmc-connections/snow-system-oauth.png)
> 3. Haga clic en Revoke Access (Revocar acceso) y, a continuación, en Revoke (Revocar).

- Instalar la aplicación de usuario para la integración de Microsoft Log Analytics (aplicación de ServiceNow). [Más información](https://store.servicenow.com/sn_appstore_store.do#!/store/application/ab0265b2dbd53200d36cdc50cf961980/1.0.1 ).
- Crear el rol de usuario de integración para la aplicación de usuario instalada. [Aquí](#create-integration-user-role-in-servicenow-app) puede encontrar más información sobre cómo crear el rol de usuario de integración.

### <a name="connection-procedure"></a>**Procedimiento de conexión**
Use el procedimiento siguiente para crear una nueva conexión a ServiceNow.


1. En Azure Portal, vaya a **Todos los recursos** y busque **ServiceDesk(YourWorkspaceName)** .

2.  En **ORÍGENES DE DATOS DEL ÁREA DE TRABAJO**, haga clic en **Conexiones de ITSM**.
    ![Nueva conexión](media/itsmc-connections/add-new-itsm-connection.png)

3. En la parte superior del panel derecho, haga clic en **Agregar**.

4. Proporcione la información tal como se describe en la tabla siguiente y haga clic en **Aceptar** para crear la conexión.


> [!NOTE]
> Todos estos parámetros son obligatorios.

| **Campo** | **Descripción** |
| --- | --- |
| **Nombre de la conexión**   | Escriba un nombre para la instancia de ServiceNow que quiere conectar con ITSMC.  Usará este nombre más adelante en Log Analytics cuando configure los elementos de trabajo en el análisis de registros detallados de ITSM o vista. |
| **Tipo de asociado**   | Seleccione **ServiceNow**. |
| **Nombre de usuario**   | Escriba el nombre de usuario de integración que ha creado en la aplicación de ServiceNow para que admita la conexión con ITSMC. Más información: [Creación de un rol de usuario de aplicación de ServiceNow](#create-integration-user-role-in-servicenow-app).|
| **Contraseña**   | Escriba la contraseña asociada con este nombre de usuario. **Nota**: El nombre de usuario y la contraseña se utilizan para generar únicamente tokens de autenticación y no se almacenan en ningún lugar dentro del servicio ITSMC.  |
| **Dirección URL del servidor**   | Escriba la dirección URL de la instancia de ServiceNow que quiere conectar con ITSMC. La dirección URL debe apuntar a una versión de SaaS compatible con el sufijo ".servicenow.com".|
| **Id. de cliente**   | Escriba el identificador de cliente que desea utilizar para la autenticación de OAuth2 y que ha generado anteriormente.  Para más información acerca de cómo generar el identificador y el secreto del cliente:   [Configuración de OAuth](https://wiki.servicenow.com/index.php?title=OAuth_Setup). |
| **Secreto de cliente**   | Escriba el secreto de cliente generado para este identificador.   |
| **Ámbito de sincronización de datos**   | Seleccione los elementos de trabajo de ServiceNow que quiere sincronizar con Azure Log Analytics mediante ITSMC.  Los valores seleccionados se importan en Log Analytics.   **Opciones:**  incidentes y solicitudes de cambio.|
| **Sincronización de datos** | Escriba el número de días pasados de los que desea los datos. **Límite máximo**: 120 días. |
| **Creación de un elemento de configuración de solución ITSM** | Seleccione esta opción si desea crear los elementos de configuración en el producto ITSM. Cuando se selecciona, ITMSC crea los elementos de configuración afectados como elementos de configuración (en el caso de que no existan) en el sistema ITSM compatible. **Valor predeterminado**: deshabilitado. |

![Conexión de ServiceNow](media/itsmc-connections/itsm-connection-servicenow-connection-latest.png)

**Cuando se ha conectado y sincronizado correctamente**:

- Los elementos de trabajo seleccionados en la instancia de ServiceNow se importan en Azure **Log Analytics**. Puede ver el resumen de estos elementos de trabajo en el icono de IT Service Management Connector.

- Puede crear incidentes a partir de alertas de Log Analytics, de entradas de registros o de alertas de Azure en esta instancia de ServiceNow.

Más información: [Creación de elementos de trabajo de ITSM a partir de alertas de Azure](./itsmc-overview.md#create-itsm-work-items-from-azure-alerts).


> [!NOTE]
> ServiceNow tiene un límite de frecuencia para las solicitudes por hora. Para configurar dicho límite, defina el valor de "Limitación de velocidad de la API REST de entrada" en la instancia de ServiceNow.

### <a name="create-integration-user-role-in-servicenow-app"></a>Creación de un rol de usuario de integración de aplicación de ServiceNow

Utilice el siguiente procedimiento:

1. Visite la [tienda de ServiceNow](https://store.servicenow.com/sn_appstore_store.do#!/store/application/ab0265b2dbd53200d36cdc50cf961980/1.0.1) e instale la **aplicación de usuario para la integración de OMS de Microsoft y ServiceNow** en la instancia de ServiceNow.
   
   >[!NOTE]
   >Como parte de la transición continuada de Microsoft Operations Management Suite (OMS) a Azure Monitor, OMS ahora se conoce como Log Analytics.     
2. Después de la instalación, visite la barra de navegación izquierda de la instancia de ServiceNow, y busque y seleccione el integrador de OMS de Microsoft.  
3. Haga clic en **Installation Checklist** (Lista de comprobación de instalación).

   El estado se muestra como **Not complete** (Sin completar) si aún no se creó el rol de usuario.

4. En los cuadros de texto, junto a **Create integration user** (Crear usuario de integración), escriba el nombre de usuario para el usuario que puede conectarse a ITSMC en Azure.
5. Escriba la contraseña para este usuario y haga clic en **Aceptar**.  

> [!NOTE]
> 
> Utilice estas credenciales para realizar la conexión de ServiceNow en Azure.

El usuario recién creado se muestra con los roles predeterminados asignados.

**Roles predeterminados**:
- personalize_choices
- import_transformer
-   x_mioms_microsoft.User
-   itil
-   template_editor
-   view_changer

Cuando el usuario se crea correctamente, el estado de **Check Installation Checklist** (Comprobar lista de comprobación de instalación) pasa a Completed (Completado) y se muestran los detalles del rol de usuario creado para la aplicación.

> [!NOTE]
> 
> El Conector ITSM puede enviar incidentes a ServiceNow sin otros módulos instalados en la instancia de ServiceNow. Si usa el módulo EventManagement en su instancia de ServiceNow y quiere crear eventos o alertas en ServiceNow mediante el conector, agregue los siguientes roles para el usuario de integración:
> 
>    - evt_mgmt_integration
>    - evt_mgmt_operator  


## <a name="connect-provance-to-it-service-management-connector-in-azure"></a>Conectar Provance al Conector de Administración de servicios de TI en Azure

En las secciones siguientes se proporcionan detalles sobre cómo conectar su producto de Provance a ITSMC en Azure.

> [!NOTE]
> 
> A partir del 1 de octubre de 2020, la integración de ITSM de Provance con alertas de Azure ya no se habilitará para los clientes nuevos. No se admitirán las nuevas conexiones de ITSM. 
> Se admitirán las conexiones de ITSM existentes.

### <a name="prerequisites"></a>Prerrequisitos

Asegúrese de que se cumplen los siguientes requisitos previos:


- ITSMC instalado Más información: [Agregar la solución IT Service Management Connector](./itsmc-overview.md#adding-the-it-service-management-connector-solution).
- La aplicación Provance debe registrarse con Azure AD y estará disponible el identificador de cliente. Para una información detallada, consulte el tema sobre [cómo configurar la autenticación de Active Directory](../../app-service/configure-authentication-provider-aad.md).

- Rol de usuario:  Administrador.

### <a name="connection-procedure"></a>Procedimiento de conexión

Use el procedimiento siguiente para crear una conexión a Provance:

1. En Azure Portal, vaya a **Todos los recursos** y busque **ServiceDesk(YourWorkspaceName)** .

2.  En **ORÍGENES DE DATOS DEL ÁREA DE TRABAJO**, haga clic en **Conexiones de ITSM**.
    ![Nueva conexión](media/itsmc-connections/add-new-itsm-connection.png)

3. En la parte superior del panel derecho, haga clic en **Agregar**.

4. Proporcione la información tal como se describe en la tabla siguiente y haga clic en **Aceptar** para crear la conexión.

> [!NOTE]
> 
> Todos estos parámetros son obligatorios.

| **Campo** | **Descripción** |
| --- | --- |
| **Nombre de la conexión**   | Escriba un nombre para la instancia de Provance que quiere conectar con ITSMC.  Usará este nombre más adelante cuando configure los elementos de trabajo en el análisis de registros detallados de ITSM o vista. |
| **Tipo de asociado**   | Seleccione **Provance**. |
| **Nombre de usuario**   | Escriba el nombre de usuario que puede conectarse a ITSMC.    |
| **Contraseña**   | Escriba la contraseña asociada con este nombre de usuario. **Nota:** El nombre de usuario y la contraseña se utilizan para generar únicamente tokens de autenticación y no se almacenan en ningún lugar dentro del servicio ITSMC._|
| **Dirección URL del servidor**   | Escriba la dirección URL de la instancia de Provance que quiere conectar con ITSMC. |
| **Id. de cliente**   | Escriba el identificador de cliente para autenticar esta conexión, que genera en la instancia de Provance.  Para más información sobre el identificador de cliente, consulte el tema sobre [cómo configurar la autenticación de Active Directory](../../app-service/configure-authentication-provider-aad.md). |
| **Ámbito de sincronización de datos**   | Seleccione los elementos de trabajo de Provance que quiera sincronizar con Azure Log Analytics mediante ITSMC.  Estos elementos de trabajo se importan en Log Analytics.   **Opciones:**   incidentes, solicitudes de cambio.|
| **Sincronización de datos** | Escriba el número de días pasados de los que desea los datos. **Límite máximo**: 120 días. |
| **Creación de un elemento de configuración de solución ITSM** | Seleccione esta opción si desea crear los elementos de configuración en el producto ITSM. Cuando se selecciona, ITMSC crea los elementos de configuración afectados como elementos de configuración (en el caso de que no existan) en el sistema ITSM compatible. **Valor predeterminado**: deshabilitado.|

![Conexión a Provance](media/itsmc-connections/itsm-connections-provance-latest.png)

**Cuando se ha conectado y sincronizado correctamente**:

- Los elementos de trabajo seleccionados en la instancia de Provance se importan en Azure **Log Analytics**. Puede ver el resumen de estos elementos de trabajo en el icono de IT Service Management Connector.

- Puede crear incidentes a partir de alertas de Log Analytics, de entradas de registros o de alertas de Azure en esta instancia de Provance.

Más información: [Creación de elementos de trabajo de ITSM a partir de alertas de Azure](./itsmc-overview.md#create-itsm-work-items-from-azure-alerts).

## <a name="connect-cherwell-to-it-service-management-connector-in-azure"></a>Conectar Cherwell con el Conector de Administración de servicios de TI en Azure

En las secciones siguientes se proporcionan detalles sobre cómo conectar su producto de Cherwell a ITSMC en Azure.

> [!NOTE]
> 
> A partir del 1 de octubre de 2020, la integración de ITSM de Cherwell con alertas de Azure ya no se habilitará para los clientes nuevos. No se admitirán las nuevas conexiones de ITSM. 
> Se admitirán las conexiones de ITSM existentes.

### <a name="prerequisites"></a>Prerrequisitos

Asegúrese de que se cumplen los siguientes requisitos previos:

- ITSMC instalado Más información: [Agregar la solución IT Service Management Connector](./itsmc-overview.md#adding-the-it-service-management-connector-solution).
- Identificador de cliente generado. Más información: [Generación del identificador de cliente para Cherwell](#generate-client-id-for-cherwell).
- Rol de usuario:  Administrador.

### <a name="connection-procedure"></a>Procedimiento de conexión

Use el procedimiento siguiente para crear una conexión a Cherwell:

1. En Azure Portal, vaya a **Todos los recursos** y busque **ServiceDesk(YourWorkspaceName)** .

2.  En **ORÍGENES DE DATOS DEL ÁREA DE TRABAJO**, haga clic en **Conexiones de ITSM**.
    ![Nueva conexión](media/itsmc-connections/add-new-itsm-connection.png)

3. En la parte superior del panel derecho, haga clic en **Agregar**.

4. Proporcione la información tal como se describe en la tabla siguiente y haga clic en **Aceptar** para crear la conexión.

> [!NOTE]
> 
> Todos estos parámetros son obligatorios.

| **Campo** | **Descripción** |
| --- | --- |
| **Nombre de la conexión**   | Escriba un nombre para la instancia de Cherwell que quiere conectar con ITSMC.  Usará este nombre más adelante cuando configure los elementos de trabajo en el análisis de registros detallados de ITSM o vista. |
| **Tipo de asociado**   | Seleccione **Cherwell.** |
| **Nombre de usuario**   | Escriba el nombre de usuario de Cherwell que puede conectarse a ITSMC. |
| **Contraseña**   | Escriba la contraseña asociada con este nombre de usuario. **Nota:** El nombre de usuario y la contraseña se utilizan para generar únicamente tokens de autenticación y no se almacenan en ningún lugar dentro del servicio ITSMC.|
| **Dirección URL del servidor**   | Escriba la dirección URL de la instancia de Cherwell que quiere conectar con ITSMC. |
| **Id. de cliente**   | Escriba el identificador de cliente para autenticar esta conexión, que genera en la instancia de Cherwell.   |
| **Ámbito de sincronización de datos**   | Seleccione los elementos de trabajo de Cherwell que quiere sincronizar mediante ITSMC.  Estos elementos de trabajo se importan en Log Analytics.   **Opciones:**  incidentes, solicitudes de cambio. |
| **Sincronización de datos** | Escriba el número de días pasados de los que desea los datos. **Límite máximo**: 120 días. |
| **Creación de un elemento de configuración de solución ITSM** | Seleccione esta opción si desea crear los elementos de configuración en el producto ITSM. Cuando se selecciona, ITMSC crea los elementos de configuración afectados como elementos de configuración (en el caso de que no existan) en el sistema ITSM compatible. **Valor predeterminado**: deshabilitado. |


![Conexión a Provance](media/itsmc-connections/itsm-connections-cherwell-latest.png)

**Cuando se ha conectado y sincronizado correctamente**:

- Los elementos de trabajo seleccionados en la instancia de Cherwell se importan en Azure **Log Analytics**. Puede ver el resumen de estos elementos de trabajo en el icono de IT Service Management Connector.

- Puede crear incidentes a partir de alertas de Log Analytics, de entradas de registros o de alertas de Azure en esta instancia de Cherwell.

Más información: [Creación de elementos de trabajo de ITSM a partir de alertas de Azure](./itsmc-overview.md#create-itsm-work-items-from-azure-alerts).

### <a name="generate-client-id-for-cherwell"></a>Generación del identificador de cliente para Cherwell

Para generar el identificador o clave de cliente para Cherwell, utilice el procedimiento siguiente:

1. Inicie sesión en la instancia de Cherwell como administrador.
2. Haga clic en **Security** > **Edit REST API client settings** (Seguridad > Editar configuración de cliente de API de REST).
3. Seleccione **Create new client** > **client secret** (Crear nuevo cliente > Secreto de cliente).

    ![Identificador de usuario de Cherwell](media/itsmc-connections/itsmc-cherwell-client-id.png)


## <a name="next-steps"></a>Pasos siguientes
 - [Creación de elementos de trabajo de ITSM a partir de alertas de Azure](./itsmc-overview.md#create-itsm-work-items-from-azure-alerts)

