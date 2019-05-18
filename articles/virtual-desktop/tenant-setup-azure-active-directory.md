---
title: 'Creación de un inquilino en Windows Virtual Desktop (versión preliminar): Azure'
description: Se describe cómo configurar los inquilinos de la versión preliminar de Windows Virtual Desktop en Azure Active Directory.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: tutorial
ms.date: 03/21/2019
ms.author: helohr
ms.openlocfilehash: 88b3ffa38eb42eef42c98920b2c3193661b1c0f5
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236048"
---
# <a name="tutorial-create-a-tenant-in-windows-virtual-desktop-preview"></a>Tutorial: Creación de un inquilino en Windows Virtual Desktop (versión preliminar)

Crear un inquilino en la versión preliminar de Windows Virtual Desktop es el primer paso hacia la compilación de su primera solución de virtualización de escritorio. Un inquilino es un grupo de uno o varios grupos host. Cada grupo host se compone de varios hosts de sesión, que se ejecutan como máquinas virtuales en Azure y que están registrados en el servicio Windows Virtual Desktop. Cada grupo host consta también de uno o varios grupos de aplicaciones que se usan para publicar recursos del escritorio remoto y de la aplicación remota para los usuarios. Con un inquilino, puede crear grupos host, crear grupos de aplicaciones, asignar usuarios y realizar conexiones a través del servicio.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Conceder permisos de Azure Active Directory en el servicio Windows Virtual Desktop.
> * Asignar el rol de aplicación TenantCreator a un usuario en el inquilino de Azure Active Directory.
> * Crear un inquilino de Windows Virtual Desktop.

Esto es lo que necesita para configurar el inquilino de Windows Virtual Desktop:

* El identificador de inquilino de [Azure Active Directory](https://azure.microsoft.com/services/active-directory/) para los usuarios de Windows Virtual Desktop.
* Una cuenta de administrador global en el inquilino de Azure Active Directory.
   * Esto también es aplicable a las organizaciones de proveedores de soluciones en la nube (CSP) que crean un inquilino de Windows Virtual Desktop para sus clientes. Si este es su caso, debe poder iniciar sesión como administrador global de la instancia de Azure Active Directory del cliente.
   * La cuenta de administrador se debe originar en el inquilino de Azure Active Directory en el que está intentando crear el inquilino de Windows Virtual Desktop. Este proceso no es compatible con las cuentas de Azure Active Directory B2B (invitado).
   * La cuenta de administrador debe ser una cuenta profesional o educativa.
* Una suscripción de Azure

## <a name="grant-azure-active-directory-permissions-to-the-windows-virtual-desktop-preview-service"></a>Conceder permisos de Azure Active Directory en el servicio Windows Virtual Desktop (versión preliminar)

Si ya ha concedido permisos a Windows Virtual Desktop para esta instancia de Azure Active Directory, omita esta sección.

Conceder permisos al servicio Windows Virtual Desktop le permite consultar Azure Active Directory para buscar tareas administrativas y de usuario final.

Para conceder los permisos de servicio:

1. Abra un explorador y conéctese a la [página de consentimiento de Windows Virtual Desktop](https://rdweb.wvd.microsoft.com).
2. En **Consent Option** (Opción de consentimiento) > **Server App**, escriba el nombre del inquilino de Azure Active Directory o el identificador de directorio y, después, seleccione **Submit** (Enviar).
        Para clientes de proveedores de soluciones en la nube, el identificador es el identificador de Microsoft del cliente de Partner Portal. Para los clientes empresariales, el identificador se encuentra en **Azure Active Directory** > **Propiedades** > **Id. de directorio**.
3. Inicie sesión en la página de consentimiento de Windows Virtual Desktop con una cuenta de administrador global. Por ejemplo, si pertenecía a la organización Contoso, su cuenta podría ser admin@contoso.com o admin@contoso.onmicrosoft.com.  
4. Seleccione **Aceptar**.
5. Espere un minuto.
6. Vuelva a la [página de consentimiento de Windows Virtual Desktop](https://rdweb.wvd.microsoft.com).
7. Vaya a **Consent Option** (Opción de consentimiento) > **Client App** (Aplicación cliente) y escriba el mismo nombre de inquilino de Azure Active Directory o identificador de directorio y, después, seleccione **Submit** (Enviar).
8. Inicie sesión en la página de consentimiento de Windows Virtual Desktop como administrador global como hizo en el paso 3 anterior.
9. Seleccione **Aceptar**.

## <a name="assign-the-tenantcreator-application-role-to-a-user-in-your-azure-active-directory-tenant"></a>Asignar el rol de aplicación TenantCreator a un usuario en el inquilino de Azure Active Directory

Asignar a un usuario de Azure Active Directory el rol de aplicación TenantCreator permite a ese usuario crear un inquilino de Windows Virtual Desktop asociado con la instancia de Azure Active Directory. Deberá usar su cuenta de administrador global para asignar el rol TenantCreator.

Para asignar el rol de aplicación TenantCreator con su cuenta de administrador global:

1. Abra un explorador y conéctese a [Azure Portal](https://portal.azure.com) con su cuenta de administrador global.
   - Si está trabajando con varios inquilinos de Azure Active Directory, es recomendable abrir una sesión privada del explorador y copiar y pegar las direcciones URL en la barra de direcciones.
2. En la barra de búsqueda de Azure Portal, busque **Aplicaciones empresariales** y seleccione la entrada que aparece bajo la categoría **Servicios**.
3. En **Aplicaciones empresariales**, busque **Windows Virtual Desktop**. Verá las dos aplicaciones para las que ha dado su consentimiento en la sección anterior. De estas dos aplicaciones, seleccione **Windows Virtual Desktop**.
        ![Captura de pantalla de los resultados de búsqueda al buscar "Windows Virtual Desktop" en "Aplicaciones empresariales". Se resalta la aplicación denominada "Windows Virtual Desktop".](media/tenant-enterprise-app.png)
4. Seleccione **Usuarios y grupos**. Es posible que vea que el administrador que concedió consentimiento a la aplicación ya aparece con el rol **Acceso predeterminado** asignado. Esto no es suficiente para crear un inquilino de Windows Virtual Desktop. Siga con estas instrucciones para agregar el rol **TenantCreator** a un usuario.
        ![Captura de pantalla de los usuarios y grupos asignados para administrar la aplicación empresarial "Windows Virtual Desktop". La captura de pantalla muestra solo una asignación que es para el "Acceso predeterminado".](media/tenant-default-access.png)
5. Seleccione **Agregar usuario** y luego **Usuarios y grupos** en la hoja **Agregar asignación**.
6. Busque una cuenta de usuario que cree el inquilino de Windows Virtual Desktop. Para mayor sencillez, esta puede ser la cuenta de administrador global.

    ![Captura de pantalla de la selección de un usuario para agregar como "TenantCreator".](media/tenant-assign-user.png)

   > [!NOTE]
   > Debe seleccionar un usuario (o un grupo que contiene un usuario) que tenga su origen en esta instancia de Azure Active Directory. No puede elegir un usuario invitado (B2B) o una entidad de servicio.

7. Seleccione la cuenta de usuario, luego el botón **Seleccionar** y, finalmente, **Asignar**.
8. En la página **Windows Virtual Desktop - Usuarios y grupos**, compruebe que ve una nueva entrada con el rol **TenantCreator** asignado al usuario que va a crear el inquilino de Windows Virtual Desktop.
        ![Captura de pantalla de los usuarios y grupos asignados para administrar la aplicación empresarial "Windows Virtual Desktop". La captura de pantalla ahora incluye una segunda entrada de un usuario asignado al rol "TenantCreator".](media/tenant-tenant-creator-added.png)

Antes de continuar para crear el inquilino de Windows Virtual Desktop, necesita dos datos más:
- Su identificador de inquilino de Azure Active Directory (o **identificador de directorio**)
- El identificador de suscripción a Azure

Para buscar su identificador de inquilino de Azure Active Directory (o **identificador de directorio**):
1. En la sesión de Azure Portal, busque **Azure Active Directory** en la barra de búsqueda y seleccione la entrada que aparece bajo la categoría **Servicios**.
        ![Captura de pantalla de los resultados de búsqueda para "Azure Active Directory" en Azure Portal. El resultado de búsqueda en "Servicios" aparece resaltado.](media/tenant-search-azure-active-directory.png)
2. Desplácese hacia abajo hasta que encuentre **Propiedades** y, a continuación, selecciónelo.
3. Busque el **identificador de directorio** y seleccione luego el icono del Portapapeles. Péguelo en un una ubicación práctica para que pueda usarlo más adelante como **AadTenantId**.
        ![Captura de pantalla de las propiedades de Azure Active Directory. El puntero se mantiene sobre el icono del Portapapeles para copiar y pegar el identificador de directorio.](media/tenant-directory-id.png)

Para buscar el identificador de suscripción de Azure:
1. En la misma sesión de Azure Portal, busque **Suscripciones** en la barra de búsqueda y seleccione la entrada que aparece bajo la categoría **Servicios**.
        ![Captura de pantalla de los resultados de búsqueda para "Azure Active Directory" en Azure Portal. El resultado de búsqueda en "Servicios" aparece resaltado.](media/tenant-search-subscription.png)
2. Seleccione la suscripción de Azure que le gustaría usar para recibir las notificaciones del servicio Windows Virtual Desktop.
3. Busque el **identificador de suscripción** y luego mantenga el puntero sobre el valor hasta que aparezca el icono del Portapapeles. Seleccione el icono del Portapapeles y péguelo en una ubicación práctica para que pueda usarlo más adelante como el **AzureSubscriptionId**.
        ![Captura de pantalla de las propiedades de la suscripción de Azure. El puntero se mantiene sobre el icono del Portapapeles para copiar y pegar el identificador de suscripción.](media/tenant-subscription-id.png)

## <a name="create-a-windows-virtual-desktop-preview-tenant"></a>Creación de un inquilino en Windows Virtual Desktop (versión preliminar)

Ahora que ha concedido los permisos al servicio Windows Virtual Desktop para consultar Azure Active Directory y le ha asignado el rol TenantCreator a una cuenta de usuario, puede crear un inquilino de Windows Virtual Desktop.

En primer lugar y, si aún no lo ha hecho, [descargue e importe el módulo de Windows Virtual Desktop](https://docs.microsoft.com/powershell/windows-virtual-desktop/overview) que se usará en la sesión de PowerShell.

Inicie sesión en Windows Virtual Desktop mediante la cuenta de usuario de TenantCreator con este cmdlet:

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

Después de eso, cree un nuevo inquilino de Windows Virtual Desktop asociado al inquilino de Azure Active Directory:

```powershell
New-RdsTenant -Name <TenantName> -AadTenantId <DirectoryID> -AzureSubscriptionId <SubscriptionID>
```

Los valores entre corchetes deben reemplazarse por los valores pertinentes para la organización y el inquilino. Por ejemplo, supongamos que usted tiene el rol TenantCreator en Windows Virtual Desktop para la organización Contoso. El cmdlet que ejecutaría tendría este aspecto:

```powershell
New-RdsTenant -Name Contoso -AadTenantId 00000000-1111-2222-3333-444444444444 -AzureSubscriptionId 55555555-6666-7777-8888-999999999999
```

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya creado a su inquilino, tendrá que crear una entidad de servicio en Azure Active Directory y asignarle un rol dentro de Windows Virtual Desktop. La entidad de servicio le permitirá implementar correctamente la oferta de Azure Marketplace de Windows Virtual Desktop para crear un grupo host. Para más información acerca de los grupos host, continúe con el tutorial de creación de un grupo host en Windows Virtual Desktop.

> [!div class="nextstepaction"]
> [Creación de entidades de servicio y asignaciones de roles con PowerShell](./create-service-principal-role-powershell.md)
