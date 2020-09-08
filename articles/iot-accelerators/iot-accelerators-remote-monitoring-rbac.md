---
title: 'Control de acceso de supervisión remota: Azure | Microsoft Docs'
description: En este artículo se proporciona información sobre cómo puede configurar el control de acceso basado en rol (RBAC) en el acelerador de soluciones de supervisión remota
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 03/08/2019
ms.topic: conceptual
ms.custom: has-adal-ref, devx-track-csharp
ms.openlocfilehash: b32e0a60e099b5ad184f8919be3f93646c3a68b3
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018151"
---
# <a name="configure-role-based-access-control-in-the-remote-monitoring-solution-accelerator"></a>Configuración del control de acceso basado en rol en el acelerador de soluciones de supervisión remota

En este artículo se proporciona información sobre cómo configurar el control de acceso basado en rol en el acelerador de soluciones de supervisión remota. El control de acceso basado en rol permite restringir el acceso a determinados usuarios a características específicas de la solución.

## <a name="default-settings"></a>Configuración predeterminada

Al implementar la solución de supervisión remota por primera vez, hay dos roles: **Administrador** y **Solo lectura**.

Cualquier usuario en el rol **Administrador** tiene acceso completo a la solución, incluidos los permisos siguientes. Un usuario en el rol **Solo lectura** solo tendrá acceso a la vista de la solución.

| Permiso            | Administración | Solo lectura |
|----------------       |-------|-----------|
| Vista de la solución         | Sí   | Sí       |
| Actualizar alarmas         | Sí   | No        |
| Eliminar alarmas         | Sí   | No        |
| Crear dispositivos        | Sí   | No        |
| Actualizar dispositivos        | Sí   | No        |
| Eliminar dispositivos        | Sí   | No        |
| Crear grupos de dispositivos  | Sí   | No        |
| Actualizar grupos de dispositivos  | Sí   | No        |
| Eliminar grupos de dispositivos  | Sí   | No        |
| Creación de reglas          | Sí   | No        |
| Actualizar reglas          | Sí   | No        |
| Eliminar reglas          | Sí   | No        |
| Creación de trabajos           | Sí   | No        |
| Actualizar la administración de SIM | Sí   | No        |

De manera predeterminada, al usuario que implementa la solución se le asigna automáticamente el rol **Administrador** y es propietario de la aplicación de Azure Active Directory. Como propietario de la aplicación, puede asignar roles a otros usuarios en Azure Portal. Si quiere que otros usuarios asignen roles en la solución, también se deben establecer como propietarios de la aplicación en Azure Portal.

> [!NOTE]
> El usuario que implementó la solución es la **única persona** que puede verla inmediatamente una vez creada. Para conceder a otros usuarios acceso para ver la aplicación con los roles de Solo lectura, Administrador o Personalizado, vea las instrucciones siguientes acerca de cómo agregar o eliminar usuarios.

## <a name="add-or-remove-users"></a>Incorporación o eliminación de usuarios

Como propietario de la aplicación de Azure Active Directory, puede usar Azure Portal para agregar un usuario a un rol o quitarlo desde la solución de supervisión remota. En los pasos siguientes se usa la [aplicación empresarial de Azure Active Directory](../active-directory/manage-apps/view-applications-portal.md) que se creó cuando implementó la solución de supervisión remota.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Compruebe que el [usuario está en el directorio](../active-directory/fundamentals/add-users-azure-active-directory.md) que usa. El directorio se elige al iniciar sesión en el sitio de [aceleradores de soluciones de Microsoft Azure IoT](https://www.azureiotsolutions.com/Accelerators). El nombre del directorio se puede ver en la esquina superior derecha de la [página](https://www.azureiotsolutions.com/Accelerators).

1. Busque la **aplicación empresarial** para su solución en Azure Portal. Una vez allí, filtre la lista mediante el establecimiento de **Tipo de aplicación** en **Todas las aplicaciones**. Busque la aplicación por su nombre. El nombre de la aplicación es el nombre de la solución de supervisión remota. En la siguiente captura de pantalla, los nombres para mostrar de la solución y de la aplicación son **contoso-rm4**.

    ![Aplicación empresarial](media/iot-accelerators-remote-monitoring-rbac/appregistration.png)

1. Compruebe que es propietario de la aplicación; para ello, haga clic en la aplicación y, luego, en **Propietarios**. En la siguiente captura de pantalla, **Contoso admin** es propietario de la aplicación **contoso-rm4**:

    ![Propietarios](media/iot-accelerators-remote-monitoring-rbac/owners.png)

    Si no es un propietario, debe pedir a un propietario existente que lo agregue a la lista. Solo los propietarios pueden asignar roles de aplicación como **Administrador** o **Solo lectura** a otros usuarios.

1. Para ver la lista de usuarios asignados a roles en la aplicación, haga clic en **Usuarios y grupos**.

1. Para agregar un usuario, haga clic en **+ Agregar usuario** y, a continuación, haga clic en **Usuarios y grupos > Ninguno seleccionado** para seleccionar un usuario del directorio.

1. Para asignar un rol al usuario, haga clic en **Seleccionar rol > Ninguno seleccionado** y elija el rol **Administrador** o **Solo lectura** para el usuario. Haga clic en **Seleccionar** y, a continuación, en **Asignar**.

    ![Seleccionar rol](media/iot-accelerators-remote-monitoring-rbac/selectrole.png)

1. Ahora el usuario puede acceder a la solución de supervisión remota con los permisos definidos por el rol.

1. Puede eliminar usuarios de la aplicación en la página **Usuarios y grupos** del portal.

## <a name="create-a-custom-role"></a>Crear un rol personalizado

La solución de supervisión remota incluye los roles **Administrador** y **Solo lectura** cuando se implementa por primera vez. Puede agregar roles personalizados con diferentes conjuntos de permisos. Para definir un rol personalizado, debe:

- Agregar un nuevo rol a la aplicación en Azure Portal
- Definir una directiva para el nuevo rol en el microservicio Autenticación y autorización
- Actualizar la interfaz de usuario web de la solución

### <a name="define-a-custom-role-in-the-azure-portal"></a>Definir un rol personalizado en Azure Portal

En los pasos siguientes se describe cómo agregar un rol a una aplicación en Azure Active Directory. En este ejemplo, creará un nuevo rol que permite a los miembros crear, actualizar y eliminar dispositivos en la solución de supervisión remota.

1. Busque su solución en el **registro de aplicaciones** de Azure Portal. El nombre de la aplicación es el nombre de la solución de supervisión remota. En la siguiente captura de pantalla, los nombres para mostrar de la solución y de la aplicación son **contoso-rm4**.

    ![Registro de aplicación](media/iot-accelerators-remote-monitoring-rbac/app-registration-2.png)

1. Seleccione la aplicación y, a continuación, haga clic en **Manifiesto**. Puede ver los dos [roles de aplicación](https://docs.microsoft.com/azure/architecture/multitenant-identity/app-roles) existentes definidos para la aplicación:

    ![Visualización del manifiesto](media/iot-accelerators-remote-monitoring-rbac/view-manifest.png)

1. Edite el manifiesto para agregar un rol denominado **ManageDevices** como se muestra en el siguiente fragmento de código. Necesita una cadena única como un GUID para el nuevo identificador de rol. Puede generar un nuevo GUID con un servicio como [Online GUID Generator](https://www.guidgenerator.com/):

    ```json
    "appRoles": [
      {
        "allowedMemberTypes": [
          "User"
        ],
        "displayName": "Admin",
        "id": "a400a00b-f67c-42b7-ba9a-f73d8c67e433",
        "isEnabled": true,
        "description": "Administrator access to the application",
        "value": "Admin"
      },
      {
        "allowedMemberTypes": [
          "User"
        ],
        "displayName": "Read Only",
        "id": "e5bbd0f5-128e-4362-9dd1-8f253c6082d7",
        "isEnabled": true,
        "description": "Read only access to device information",
        "value": "ReadOnly"
      },
      {
        "allowedMemberTypes": [
          "User"
        ],
        "displayName": "ManageDevices",
        "id": "ADD A NEW GUID HERE",
        "isEnabled": true,
        "description": "A new role for the application.",
        "value": "ManageDevices"
      }
    ],
    ```

    Guarde los cambios.

### <a name="define-a-policy-for-the-new-role"></a>Definición de una directiva para el nuevo rol

Después de agregar el rol a la aplicación en Azure Portal, debe definir una directiva en [roles.json](https://github.com/Azure/remote-monitoring-services-dotnet/blob/master/auth/Services/data/policies/roles.json) para el rol que asigne los permisos necesarios para administrar dispositivos.

1. Clone el repositorio de [microservicios de supervisión remota](https://github.com/Azure/remote-monitoring-services-dotnet) de GitHub en el equipo local.

1. Edite el archivo **auth/Services/data/policies/roles.json** para agregar la directiva del rol **ManageDevices**, tal y como se muestra en el siguiente fragmento de código. Los valores **Id.** y **Rol** deben coincidir con la definición de rol del manifiesto de aplicación de la sección anterior. La lista de acciones permitidas permite que alguien con el rol **ManageDevices** pueda crear, actualizar y eliminar dispositivos conectados a la solución:

    ```json
    {
    "Items": [
      {
        "Id": "a400a00b-f67c-42b7-ba9a-f73d8c67e433",
        "Role": "admin",
        "AllowedActions": [
          "UpdateAlarms",
          "DeleteAlarms",
          "CreateDevices",
          "UpdateDevices",
          "DeleteDevices",
          "CreateDeviceGroups",
          "UpdateDeviceGroups",
          "DeleteDeviceGroups",
          "CreateRules",
          "UpdateRules",
          "DeleteRules",
          "CreateJobs",
          "UpdateSimManagement"
        ]
      },
      {
        "Id": "e5bbd0f5-128e-4362-9dd1-8f253c6082d7",
        "Role": "readOnly",
        "AllowedActions": []
      },
      {
        "Id": "GUID FROM APP MANIFEST",
        "Role": "ManageDevices",
        "AllowedActions": [
          "CreateDevices",
          "UpdateDevices",
          "DeleteDevices"
        ]
      }
    ]
    }
    ```

1. Cuando haya terminado de editar el archivo **Services/data/policies/roles.json**, vuelva a compilar e implementar el microservicio Autenticación y autorización en el acelerador de soluciones.

### <a name="how-the-web-ui-enforces-permissions"></a>Cómo la interfaz de usuario web exige permisos

La interfaz de usuario web usa el [microservicio Autenticación y autorización](https://github.com/Azure/remote-monitoring-services-dotnet/tree/master/auth) para determinar qué acciones se permiten a un usuario y qué controles están visibles en la interfaz de usuario. Por ejemplo, si la solución se llama **contoso-rm4**, la interfaz de usuario web recopila una lista de acciones permitidas para el usuario actual mediante el envío de la solicitud siguiente:

```http
http://contoso-rm4.azurewebsites.net/v1/users/current
headers:
X-Source: true
Authorization: Bearer <JWT Token from ADAL>
```

Para un usuario llamado **Administrador de dispositivos** con el rol **ManageDevices**, la respuesta incluye el siguiente código JSON en el cuerpo:

```json
{
  "Id": "userIdExample",
  "Email": "devicemanager@contoso.com",
  "Name": "Device Manager",
  "AllowedActions": [
    "CreateDevices",
    "UpdateDevices",
    "DeleteDevices"
  ]
}
```

El siguiente fragmento de [deviceDelete.js](https://github.com/Azure/pcs-remote-monitoring-webui/blob/master/src/components/pages/devices/flyouts/deviceDelete/deviceDelete.js) en la [interfaz de usuario web](https://github.com/Azure/pcs-remote-monitoring-webui/) muestra cómo se exigen los permisos mediante declaración:

```json
<FlyoutContent>
  <Protected permission={permissions.deleteDevices}>
    <form className="device-delete-container" onSubmit={this.deleteDevices}>
      ...
    </form>
  </Protected>
</FlyoutContent>
```

Para más información, consulte [Protected Components](https://github.com/Azure/pcs-remote-monitoring-webui/blob/master/src/components/shared/protected/README.md) (Componentes protegidos). Puede definir permisos adicionales en el archivo [authModel.js](https://github.com/Azure/pcs-remote-monitoring-webui/blob/master/src/services/models/authModels.js).

### <a name="how-the-microservices-enforce-permissions"></a>Cómo los microservicios exigen permisos

Los microservicios también comprueban los permisos para protegerse frente a solicitudes de API no autorizadas. Cuando un microservicio recibe una solicitud de API, descodifica y valida el token JWT para obtener el identificador de usuario y los permisos asociados con el rol del usuario.

El siguiente fragmento de código del archivo [DevicesController.cs](https://github.com/Azure/remote-monitoring-services-dotnet/blob/master/iothub-manager/WebService/v1/Controllers/DevicesController.cs) en el [microservicio Administrador de IoTHub](https://github.com/Azure/remote-monitoring-services-dotnet/tree/master/iothub-manager), muestra cómo se exigen los permisos:

```csharp
[HttpDelete("{id}")]
[Authorize("DeleteDevices")]
public async Task DeleteAsync(string id)
{
    await this.devices.DeleteAsync(id);
}
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo ha aprendido cómo se implementa el control de acceso basado en rol en el acelerador de soluciones de supervisión remota.

Para obtener información acerca de cómo administrar el acceso al explorador de Time Series Insights en el acelerador de la solución de supervisión remota, consulte [Configure access controls for the Time Series Insights explorer](iot-accelerators-remote-monitoring-rbac-tsi.md) (Configuración de los controles de acceso del explorador de Time Series Insights).

Para información más conceptual sobre el acelerador de la solución de supervisión remota, vea [Arquitectura de supervisión remota](iot-accelerators-remote-monitoring-sample-walkthrough.md).

Para más información sobre cómo personalizar la solución de supervisión remota, consulte [Personalización y nueva implementación de un microservicio](iot-accelerators-microservices-example.md).
<!-- Next tutorials in the sequence -->