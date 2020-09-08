---
title: Exportación de la configuración de aprovisionamiento y reversión a un estado correcto conocido en la recuperación ante desastres
description: Aprenda a exportar la configuración de aprovisionamiento y a revertir a un estado correcto conocido en la recuperación ante desastres.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: how-to
ms.workload: identity
ms.date: 03/19/2020
ms.author: kenwith
ms.openlocfilehash: e34656d6ce515cabe955c101f7b52ac0f2ade8db
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88235824"
---
# <a name="how-to-export-provisioning-configuration-and-roll-back-to-a-known-good-state"></a>Procedimiento: Exportación de la configuración de aprovisionamiento y reversión a un estado correcto conocido

En este artículo, aprenderá a:

- Exportación e importación de la configuración de aprovisionamiento desde Azure Portal
- Exportación e importación de la configuración de aprovisionamiento mediante Microsoft Graph API

## <a name="export-and-import-your-provisioning-configuration-from-the-azure-portal"></a>Exportación e importación de la configuración de aprovisionamiento desde Azure Portal

### <a name="export-your-provisioning-configuration"></a>Exportación de la configuración de aprovisionamiento

Para exportar la configuración:

1. En [Azure Portal](https://portal.azure.com/), en el panel de navegación izquierdo, seleccione **Azure Active Directory**.
1. En el panel de **Azure Active Directory**, seleccione **Aplicaciones empresariales** y elija su aplicación.
1. En el panel de navegación izquierdo, seleccione **aprovisionamiento**. En la página de configuración de aprovisionamiento, haga clic en **Asignaciones de atributos**, en **Mostrar opciones avanzadas** y, por último, en **Revise el esquema**. Esto le llevará al editor de esquemas.
1. Para descargar el esquema, haga clic en el botón de descarga de la barra de comandos situada en la parte superior de la página.

### <a name="disaster-recovery---roll-back-to-a-known-good-state"></a>Recuperación ante desastres: reversión a un estado correcto conocido

Si exporta y guarda la configuración, podrá volver a una versión anterior de la configuración. Se recomienda exportar la configuración de aprovisionamiento y guardarla para más adelante siempre que realice un cambio en las asignaciones de atributos o en los filtros de ámbito. Lo único que debe hacer es abrir el archivo JSON que descargó en los pasos anteriores, copiar todo el contenido del archivo JSON, reemplazar todo el contenido de la carga útil de JSON en el editor de esquemas y, después, guardarlo. Si hay un ciclo de aprovisionamiento activo, se completará y, en el ciclo siguiente, se usará el esquema actualizado. El ciclo siguiente también será un ciclo inicial en el que se volverá a evaluar cada usuario y grupo con arreglo a la nueva configuración. Al revertir a una configuración anterior, tenga en cuenta lo siguiente:

- Los usuarios se evaluarán de nuevo para determinar si deben estar en el ámbito. Si los filtros de ámbito han cambiado, los usuarios que ya no estén en el ámbito se deshabilitarán. Aunque este es el comportamiento deseado en la mayoría de los casos, hay ocasiones en las que tal vez quiera evitarlo, por lo que puede usar la funcionalidad [omisión de las eliminaciones de ámbito](./skip-out-of-scope-deletions.md). 
- Si cambia la configuración de aprovisionamiento, se reiniciará el servicio y se desencadenará un [ciclo inicial](./how-provisioning-works.md#provisioning-cycles-initial-and-incremental).

## <a name="export-and-import-your-provisioning-configuration-by-using-the-microsoft-graph-api"></a>Exportación e importación de la configuración de aprovisionamiento mediante Microsoft Graph API

Puede usar Microsoft Graph API y el Probador de Microsoft Graph para exportar el esquema y las asignaciones de atributos de aprovisionamiento de usuarios en un archivo JSON e importarlos de nuevo a Azure AD. También puede usar los pasos que se indican aquí para crear una copia de seguridad de la configuración de aprovisionamiento.

### <a name="step-1-retrieve-your-provisioning-app-service-principal-id-object-id"></a>Paso 1: Recuperar el identificador de la entidad de servicio de la aplicación de aprovisionamiento (identificador de objeto)

1. Inicie [Azure Portal](https://portal.azure.com) y vaya a la sección Propiedades de la aplicación de aprovisionamiento. Por ejemplo, si desea exportar la asignación de la *aplicación de aprovisionamiento de usuarios de Workday a AD*, vaya a la sección Propiedades de esa aplicación.
1. En la sección Propiedades de la aplicación de aprovisionamiento, copie el valor GUID asociado con el campo *Id. de objeto*. Este valor también se conoce como el elemento **ServicePrincipalId** de la aplicación y se usará en las operaciones del Probador de Microsoft Graph.

   ![Identificador de la entidad de servicio de la aplicación de Workday](./media/export-import-provisioning-configuration/wd_export_01.png)

### <a name="step-2-sign-into-microsoft-graph-explorer"></a>Paso 2: Iniciar sesión en el Probador de Graph de Microsoft

1. Iniciar el [Probador de Graph de Microsoft](https://developer.microsoft.com/graph/graph-explorer)
1. Haga clic en el botón "Iniciar sesión con Microsoft" e inicie sesión con las credenciales de administrador de la aplicación o de administrador global de Azure AD.

    ![Inicio de sesión de Microsoft Graph](./media/export-import-provisioning-configuration/wd_export_02.png)

1. Una vez haya iniciado sesión correctamente, verá los detalles de la cuenta de usuario en el panel izquierdo.

### <a name="step-3-retrieve-the-provisioning-job-id-of-the-provisioning-app"></a>Paso 3: Recuperar el identificador del trabajo de aprovisionamiento de la aplicación de aprovisionamiento

En el Probador de Graph de Microsoft, ejecute la siguiente consulta GET; para ello, reemplace [servicePrincipalId] con la propiedad **ServicePrincipalId** extraído del [paso 1](#step-1-retrieve-your-provisioning-app-service-principal-id-object-id).

```http
   GET https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs
```

Obtendrá una respuesta como la que se muestra a continuación. Copie el "atributo de identificador" que aparece en la respuesta. Este valor se corresponde con la propiedad **ProvisioningJobId** y se utilizará para recuperar los metadatos del esquema subyacente.

   [![Id. del trabajo de aprovisionamiento](./media/export-import-provisioning-configuration/wd_export_03.png)](./media/export-import-provisioning-configuration/wd_export_03.png#lightbox)

### <a name="step-4-download-the-provisioning-schema"></a>Paso 4: Descargar el esquema de aprovisionamiento

En el Probador de Graph de Microsoft, ejecute la siguiente consulta GET, reemplace [servicePrincipalId] y [ProvisioningJobId] con las propiedades ServicePrincipalId y ProvisioningJobId recuperadas en los pasos anteriores.

```http
   GET https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs/[ProvisioningJobId]/schema
```

Copie el objeto JSON de la respuesta y guárdelo en un archivo para crear una copia de seguridad del esquema.

### <a name="step-5-import-the-provisioning-schema"></a>Paso 5: Importar el esquema de aprovisionamiento

> [!CAUTION]
> Realice este paso solo si necesita modificar el esquema de configuración y no puede hacerlo en Azure Portal o si necesita restaurar la configuración desde un archivo de copia de seguridad anterior con un esquema válido que funcione.

En el Probador de Graph de Microsoft, configure la siguiente consulta PUT, reemplace [servicePrincipalId] y [ProvisioningJobId] con las propiedades ServicePrincipalId y ProvisioningJobId recuperadas en los pasos anteriores.

```http
    PUT https://graph.microsoft.com/beta/servicePrincipals/[servicePrincipalId]/synchronization/jobs/[ProvisioningJobId]/schema
```

En la pestaña "Cuerpo de la solicitud", copie el contenido del archivo del esquema JSON.

   [![Cuerpo de la solicitud](./media/export-import-provisioning-configuration/wd_export_04.png)](./media/export-import-provisioning-configuration/wd_export_04.png#lightbox)

En la pestaña "Encabezados de solicitud", agregue el atributo de encabezado Content-Type con el valor “application/json”.

   [![Encabezados de solicitud](./media/export-import-provisioning-configuration/wd_export_05.png)](./media/export-import-provisioning-configuration/wd_export_05.png#lightbox)

Seleccione **Ejecutar consulta** para importar el nuevo esquema.