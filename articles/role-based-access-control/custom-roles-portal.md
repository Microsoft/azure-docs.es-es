---
title: 'Creación o actualización de roles personalizados de Azure mediante Azure Portal: RBAC de Azure'
description: Aprenda a crear roles personalizados de Azure mediante Azure Portal y el control de acceso basado en roles de Azure (RBAC de Azure). Esto incluye cómo enumerar, crear, actualizar y eliminar roles personalizados.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/13/2020
ms.author: rolyon
ms.openlocfilehash: 91d2605dddd6107d09e635969f5e5d98c2a02d60
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86511728"
---
# <a name="create-or-update-azure-custom-roles-using-the-azure-portal"></a>Creación o actualización de roles personalizados de Azure mediante Azure Portal

Si los [roles integrados de Azure](built-in-roles.md) no satisfacen las necesidades específicas de la organización, puede crear sus propios roles personalizados de Azure. Al igual que en el caso de los roles integrados, puede asignar los roles personalizados a usuarios, grupos y entidades de servicio en los ámbitos de suscripción y grupo de recursos. Los roles personalizados se almacenan en un directorio de Azure Active Directory (Azure AD) y se pueden compartir entre suscripciones. Cada directorio puede tener hasta 5000 roles personalizados. Se pueden crear roles personalizados con Azure Portal, Azure PowerShell, la CLI de Azure o la API REST. En este artículo se describe cómo crear roles personalizados mediante Azure Portal.

## <a name="prerequisites"></a>Prerrequisitos

Para crear roles personalizados, necesita:

- Permisos para crear roles personalizados, como [Propietario](built-in-roles.md#owner) o [Administrador de acceso de usuarios](built-in-roles.md#user-access-administrator)

## <a name="step-1-determine-the-permissions-you-need"></a>Paso 1: Determinar los permisos que necesita

Azure tiene miles de permisos que se pueden incluir en el rol personalizado. A continuación se muestran cuatro maneras de determinar los permisos que se agregarán al rol personalizado:

| Método | Descripción |
| --- | --- |
| Examinar roles existentes | Puede consultar los roles existentes para ver qué permisos se están usando. Para más información, consulte [Roles integrados en Azure](built-in-roles.md). |
| Buscar permisos por palabra clave | Al crear un rol personalizado mediante Azure Portal, puede buscar permisos por palabra clave. Por ejemplo, puede buscar permisos de *máquina virtual* o de *facturación*. Esta funcionalidad de búsqueda se describe más adelante en el [Paso 4: Permisos](#step-4-permissions). |
| Descargar todos los permisos | Al crear un rol personalizado mediante Azure Portal, puede descargar todos los permisos como un archivo CSV y, a continuación, buscar en este archivo. En el panel **Agregar permisos**, haga clic en el botón **Download all permissions** (Descargar todos los permisos) para descargar todos los permisos. Para más información sobre el panel Agregar permisos, consulte el [Paso 4: Permisos](#step-4-permissions). |
| Ver los permisos en los documentos | Puede ver los permisos disponibles en [Operaciones del proveedor de recursos de Azure Resource Manager](resource-provider-operations.md). |

## <a name="step-2-choose-how-to-start"></a>Paso 2: Decidir cómo comenzar

La creación de un rol personalizado puede comenzar de tres maneras. Se puede clonar un rol existente, empezar desde cero o empezar con un archivo JSON. La manera más fácil es encontrar un rol existente que tenga la mayoría de los permisos necesarios y, a continuación, clonarlo y modificarlo para el escenario concreto. 

### <a name="clone-a-role"></a>Clonación de un rol

Si un rol existente no tiene exactamente los permisos necesarios, puede clonarlo y, después, modificar los permisos. Siga estos pasos para iniciar la clonación de un rol.

1. En Azure Portal, abra una suscripción o un grupo de recursos donde desee que se pueda asignar el rol personalizado y, a continuación, abra **Control de acceso (IAM)** .

    En la siguiente captura de pantalla se muestra la página Control de acceso (IAM) abierta para una suscripción.

    ![Página Control de acceso (IAM) para una suscripción](./media/custom-roles-portal/access-control-subscription.png)

1. Haga clic en la pestaña **Roles** para ver una lista de todos los roles integrados y personalizados.

1. Busque un rol que quiera clonar, como el rol Lector de facturación.

1. Al final de la fila, haga clic en los puntos suspensivos ( **...** ) y, después, haga clic en **Clonar**.

    ![Menú contextual de clonación](./media/custom-roles-portal/clone-menu.png)

    Se abrirá el editor de roles personalizados con la opción **Clonar un rol** seleccionada.

1. Continúe en el [Paso 3: Aspectos básicos](#step-3-basics).

### <a name="start-from-scratch"></a>Comienzo desde cero

Si lo prefiere, puede seguir estos pasos para comenzar un rol personalizado desde cero.

1. En Azure Portal, abra una suscripción o un grupo de recursos donde desee que se pueda asignar el rol personalizado y, a continuación, abra **Control de acceso (IAM)** .

1. Haga clic en **Agregar** y, a continuación, haga clic en **Agregar rol personalizado**.

    ![Menú de incorporación de rol personalizado](./media/custom-roles-portal/add-custom-role-menu.png)

    Se abrirá el editor de roles personalizados con la opción **Empezar de cero** seleccionada.

1. Continúe en el [Paso 3: Aspectos básicos](#step-3-basics).

### <a name="start-from-json"></a>Inicio desde un archivo JSON

Si lo prefiere, puede especificar la mayoría de los valores del rol personalizado en un archivo JSON. Puede abrir el archivo en el editor de roles personalizados, realizar cambios adicionales y, a continuación, crear el rol personalizado. Siga estos pasos para empezar con un archivo JSON.

1. Cree un archivo JSON con el formato siguiente:

    ```json
    {
        "properties": {
            "roleName": "",
            "description": "",
            "assignableScopes": [],
            "permissions": [
                {
                    "actions": [],
                    "notActions": [],
                    "dataActions": [],
                    "notDataActions": []
                }
            ]
        }
    }
    ```

1. En el archivo JSON, especifique los valores para las distintas propiedades. A continuación se muestra un ejemplo con algunos valores agregados. Para obtener información sobre las distintas propiedades, consulte [Descripciones de definiciones de roles para los recursos de Azure](role-definitions.md).

    ```json
    {
        "properties": {
            "roleName": "Billing Reader Plus",
            "description": "Read billing data and download invoices",
            "assignableScopes": [
                "/subscriptions/11111111-1111-1111-1111-111111111111"
            ],
            "permissions": [
                {
                    "actions": [
                        "Microsoft.Authorization/*/read",
                        "Microsoft.Billing/*/read",
                        "Microsoft.Commerce/*/read",
                        "Microsoft.Consumption/*/read",
                        "Microsoft.Management/managementGroups/read",
                        "Microsoft.CostManagement/*/read",
                        "Microsoft.Support/*"
                    ],
                    "notActions": [],
                    "dataActions": [],
                    "notDataActions": []
                }
            ]
        }
    }
    ```
    
1. En Azure Portal, abra la página**Control de acceso (IAM)** .

1. Haga clic en **Agregar** y, a continuación, haga clic en **Agregar rol personalizado**.

    ![Menú de incorporación de rol personalizado](./media/custom-roles-portal/add-custom-role-menu.png)

    Se abrirá el editor de roles personalizados.

1. En la pestaña Aspectos básicos, en **Permisos de línea base**, seleccione **Start from JSON** (Empezar desde JSON).

1. Junto al cuadro Seleccionar un archivo, haga clic en el botón de carpeta para acceder el cuadro de diálogo Abrir.

1. Seleccione el archivo JSON y haga clic en **Abrir**.

1. Continúe en el [Paso 3: Aspectos básicos](#step-3-basics).

## <a name="step-3-basics"></a>Paso 3: Aspectos básicos

En la pestaña **Aspectos básicos**, especifique el nombre, la descripción y los permisos de línea base para el rol personalizado.

1. En el cuadro **Nombre de rol personalizado**, especifique un nombre para el rol personalizado. El nombre debe ser único en el directorio de Azure AD. Puede incluir letras, números, espacios y caracteres especiales.

1. En el cuadro **Descripción**, especifique una descripción opcional para el rol personalizado. Se convertirá en la información sobre herramientas para él.

    La opción **Permisos de línea base** debe estar ya establecida en función del paso anterior, pero puede cambiarla.

    ![Pestaña Aspectos básicos con los valores especificados](./media/custom-roles-portal/basics-values.png)

## <a name="step-4-permissions"></a>Paso 4: Permisos

En la pestaña **Permisos**, especifique los permisos para el rol personalizado. En función de si ha clonado un rol o si ha empezado con JSON, es posible que en la pestaña Permisos ya se muestren algunos permisos.

![Pestaña Permisos en la creación de un rol personalizado](./media/custom-roles-portal/permissions.png)

### <a name="add-or-remove-permissions"></a>Incorporación o eliminación de permisos

Siga los pasos que se indican a continuación para agregar o quitar permisos para el rol personalizado.

1. Si desea agregar permisos, haga clic en **Agregar permisos** para abrir el panel Agregar permisos.

    En este panel se muestran todos los permisos disponibles, agrupados en categorías diferentes con formato de tarjeta. Cada categoría representa un *proveedor de recursos*, que es un servicio que proporciona recursos de Azure.

1. En el cuadro **Buscar un permiso**, escriba una cadena para buscar permisos. Por ejemplo, busque *factura* para buscar los permisos relacionados con la factura.

    Se mostrará una lista de tarjetas de proveedores de recursos en función de la cadena de búsqueda. Para obtener una lista de cómo se asignan los proveedores de recursos a los servicios de Azure, consulte [Proveedores de recursos para servicios de Azure](../azure-resource-manager/management/azure-services-resource-providers.md).

    ![Panel Agregar permisos con proveedor de recursos](./media/custom-roles-portal/add-permissions-provider.png)

1. Haga clic en una tarjeta del proveedor de recursos que pueda tener los permisos que desea agregar al rol personalizado, como **Facturación de Microsoft**.

    Se muestra una lista de los permisos de administración de ese proveedor de recursos en función de la cadena de búsqueda.

    ![Lista de incorporación de permisos](./media/custom-roles-portal/add-permissions-list.png)

1. Si busca permisos que se apliquen al plano de datos, haga clic en **Acciones de datos**. De lo contrario, deje la opción de alternancia establecida en **Acciones**, para que se muestren los permisos que se aplican al plano de administración. Para más información acerca de las diferencias entre el plano de administración y el plano de datos, consulte [Operaciones de administración y datos](role-definitions.md#management-and-data-operations).

1. Si es necesario, actualice la cadena de búsqueda para refinar aún más la búsqueda.

1. Una vez que localice uno o más permisos que desee agregar al rol personalizado, agregue una marca de verificación junto a ellos. Por ejemplo, agregue una marca de verificación junto a **Other: Download Invoice** (Otro: Descargar factura) con el fin de agregar el permiso para descargar facturas.

1. Haga clic en **Agregar** para agregar el permiso a la lista de permisos.

    El permiso se agrega como un elemento `Actions` o `DataActions`.

    ![Permiso agregado](./media/custom-roles-portal/permissions-list-add.png)

1. Para quitar permisos, haga clic en el icono de eliminación al final de la fila. En este ejemplo, dado que un usuario no necesita la capacidad de crear incidencias de soporte técnico, se puede eliminar el permiso `Microsoft.Support/*`.

### <a name="add-wildcard-permissions"></a>Incorporación de permisos con caracteres comodín

En función de cómo haya decidido comenzar, es posible que tenga permisos con caracteres comodín (`*`) en la lista de permisos. Los carácteres comodín (`*`) amplían los permisos a todo aquello que coincida con la cadena de acción que se especifique. Por ejemplo, la siguiente cadena comodín agrega todos los permisos relacionados con Azure Cost Management y las exportaciones. De esta forma también se incluirían los futuros permisos de exportación que se pudieran agregar.

```
Microsoft.CostManagement/exports/*
```

Si desea agregar un nuevo permiso con caracteres comodín, no puede hacerlo mediante el panel **Agregar permisos**. Para agregar un permiso con caracteres comodín, debe hacerlo manualmente mediante la pestaña **JSON**. Para más información, consulte [Paso 6: JSON](#step-6-json).

### <a name="exclude-permissions"></a>Exclusión de permisos

Si el rol tiene un permiso con caracteres comodín (`*`) y desea excluir o restar permisos específicos de él, puede excluirlos. Por ejemplo, supongamos que tiene el siguiente permiso con caracteres comodín:

```
Microsoft.CostManagement/exports/*
```

Si no desea permitir que se elimine una exportación, puede excluir el siguiente permiso de eliminación:

```
Microsoft.CostManagement/exports/delete
```

Cuando se excluye un permiso, se agrega como un elemento `NotActions` o `NotDataActions`. Los permisos de administración efectivos se calculan agregando todos los elementos `Actions` y restando todos los elementos `NotActions`. Los permisos de datos efectivos se calculan agregando todos los elementos `DataActions` y restando todos los elementos `NotDataActions`.

> [!NOTE]
> La exclusión de un permiso no equivale a una denegación. La exclusión de permisos es simplemente una manera cómoda de restar permisos de un permiso con caracteres comodín.

1. Para excluir o restar un permiso de un permiso con caracteres comodín permitido, haga clic en **Excluir permisos** para abrir el panel Excluir permisos.

    En este panel, se especifican los permisos de datos o de administración que se excluyen o se restan.

1. Una vez que localice uno o más permisos que desee excluir, agregue una marca de verificación junto a los permisos y, a continuación, haga clic en el botón **Agregar**.

    ![Panel Excluir permisos: permiso seleccionado](./media/custom-roles-portal/exclude-permissions-select.png)

    El permiso se agrega como elemento `NotActions` o `NotDataActions`.

    ![Permiso excluido](./media/custom-roles-portal/exclude-permissions-list-add.png)

## <a name="step-5-assignable-scopes"></a>Paso 5: Ámbitos asignables

En la pestaña **Ámbitos asignables** se especifica dónde está disponible el rol personalizado para su asignación, como en una suscripción o un grupo de recursos. En función de cómo haya decidido comenzar, esta pestaña podría mostrar el ámbito en el que abrió la página Control de acceso (IAM). No se admite establecer el ámbito asignable en el ámbito raíz ("/"). Actualmente, no se puede agregar un grupo de administración como ámbito asignable.

1. Haga clic en **Agregar ámbitos asignables** para abrir el panel Agregar ámbitos asignables.

    ![Pestaña de ámbitos asignables](./media/custom-roles-portal/assignable-scopes.png)

1. Haga clic en uno o más ámbitos que desee usar, por lo general su suscripción.

    ![Incorporación de ámbitos asignables](./media/custom-roles-portal/add-assignable-scopes.png)

1. Haga clic en el botón **Agregar** para agregar el ámbito asignable.

## <a name="step-6-json"></a>Paso 6: JSON

En la pestaña **JSON**, verá el rol personalizado con formato JSON. Si lo desea, puede editar directamente el archivo JSON. Si desea agregar un permiso con caracteres comodín (`*`), debe usar esta pestaña.

1. Para editar el archivo JSON, haga clic en **Editar**.

    ![Pestaña JSON que muestra el rol personalizado](./media/custom-roles-portal/json.png)

1. Realice cambios en el archivo JSON.

    Si el archivo JSON no tiene el formato correcto, verá una línea dentada roja y un indicador en el medianil vertical.

1. Cuando termine de editar, haga clic en **Guardar**.

## <a name="step-7-review--create"></a>Paso 7: Revisar y crear

En la pestaña **Revisar y crear**, puede revisar la configuración del rol personalizado.

1. Revise la configuración del rol personalizado.

    ![Pestaña Revisar y crear](./media/custom-roles-portal/review-create.png)

1. Haga clic en **Crear** para crear el rol personalizado.

    Transcurridos unos instantes, aparecerá un cuadro de mensaje donde se indica que el rol personalizado se ha creado correctamente.

    ![Mensaje de creación de rol personalizado](./media/custom-roles-portal/custom-role-success.png)

    Si se detecta algún error, se mostrará un mensaje.

    ![Error de revisión y creación](./media/custom-roles-portal/review-create-error.png)

1. Vea el nuevo rol personalizado en la lista **Roles**. Si no ve el rol personalizado, haga clic en **Actualizar**.

     El rol personalizado puede tardar unos minutos en aparecer en todas partes.

## <a name="list-custom-roles"></a>Lista de roles personalizados

Siga estos pasos para ver los roles personalizados.

1. Abra una suscripción o un grupo de recursos y, a continuación, abra **Control de acceso (IAM)** .

1. Haga clic en la pestaña **Roles** para ver una lista de todos los roles integrados y personalizados.

1. En la lista **Tipo**, seleccione **CustomRole** para ver solo los roles personalizados.

    Si acaba de crear el rol personalizado y no lo ve en la lista, haga clic en **Actualizar**.

    ![Lista de roles personalizados](./media/custom-roles-portal/custom-role-list.png)

## <a name="update-a-custom-role"></a>Actualización de un rol personalizado

1. Abra la lista de roles personalizados tal como se describió anteriormente en este artículo.

1. Haga clic en los puntos suspensivos ( **...** ) del rol personalizado que desee actualizar y, a continuación, haga clic en **Editar**. Tenga en cuenta que no puede actualizar los roles integrados.

    El rol personalizado se abre en el editor.

    ![Menú del rol personalizado](./media/custom-roles-portal/edit-menu.png)

1. Use las distintas pestañas para actualizar el rol personalizado.

1. Cuando haya terminado de realizar los cambios, haga clic en la pestaña **Revisar y crear** para revisar los cambios.

1. Haga clic en el botón **Actualizar** para actualizar el rol personalizado.

## <a name="delete-a-custom-role"></a>Eliminación de un rol personalizado

1. Abra la lista de roles personalizados tal como se describió anteriormente en este artículo.

1. Quite todas las asignaciones de roles que utilicen el rol personalizado.

1. Haga clic en los puntos suspensivos ( **...** ) del rol personalizado que desee eliminar y, a continuación, haga clic en **Eliminar**.

    ![Menú del rol personalizado](./media/custom-roles-portal/delete-menu.png)

    El rol personalizado puede tardar unos minutos en eliminarse por completo.

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Creación de un rol personalizado de Azure mediante Azure PowerShell](tutorial-custom-role-powershell.md)
- [Roles personalizados en los recursos de Azure](custom-roles.md)
- [Operaciones del proveedor de recursos de Azure Resource Manager](resource-provider-operations.md)
