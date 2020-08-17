---
title: Concesión de permisos para la identidad administrada en el área de trabajo de Synapse
description: En este artículo se explica cómo configurar los permisos de la identidad administrada en el área de trabajo de Azure Synapse.
author: RonyMSFT
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 04/15/2020
ms.author: ronytho
ms.reviewer: jrasnick
ms.openlocfilehash: 5c07943d517f893e67eed5103660a953b508cc87
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87535049"
---
# <a name="grant-permissions-to-workspace-managed-identity-preview"></a>Concesión de permisos a una identidad administrada de área de trabajo (versión preliminar)

En este artículo se muestra cómo conceder permisos a la identidad administrada en el área de trabajo de Azure Synapse. Los permisos, a su vez, permiten acceder a los grupos de SQL en el área de trabajo y la cuenta de almacenamiento de ADLS Gen2 a través de Azure Portal.

>[!NOTE]
>En el resto de este documento, se hará referencia a la identidad administrada del área de trabajo como “identidad administrada”.

## <a name="grant-managed-identity-permissions-to-the-sql-pool"></a>Concesión de permisos de identidad administrada en el grupo de SQL

La identidad administrada concede permisos a los grupos de SQL en el área de trabajo. Con los permisos concedidos, puede orquestar canalizaciones que realicen las actividades relacionadas con el grupo de SQL. Al crear un área de trabajo de Azure Synapse mediante Azure Portal, puede conceder los permisos de CONTROL de identidad administrada en los grupos de SQL.

Seleccione **Seguridad y redes** al crear el área de trabajo de Azure Synapse. Después, seleccione **Grant CONTROL to the workspace's managed identity on SQL pools** (Conceder el permiso CONTROL a la identidad administrada del área de trabajo en los grupos de SQL).

![Permiso CONTROL en los grupos de SQL](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-16.png)

## <a name="grant-the-managed-identity-permissions-to-adls-gen2-storage-account"></a>Concesión de permisos de la cuenta de almacenamiento de ADLS Gen2 a la identidad administrada

Para crear un área de trabajo de Azure Synapse se requiere una cuenta de almacenamiento de ADLS Gen2. Para iniciar correctamente grupos de Spark en el área de trabajo de Azure Synapse, la identidad administrada de Azure Synapse necesita el rol de *Colaborador de datos de Storage Blob* en esta cuenta de almacenamiento. La orquestación de canalizaciones en Azure Synapse también se beneficia de este rol.

### <a name="grant-permissions-to-managed-identity-during-workspace-creation"></a>Concesión de permisos a una identidad administrada durante la creación del área de trabajo

Azure Synapse intentará conceder el rol de colaborador de datos de Blob Storage a la identidad administrada después de crear el área de trabajo de Azure Synapse mediante Azure Portal. Proporcione los detalles de la cuenta de almacenamiento de ADLS Gen2 en la pestaña **Aspectos básicos**.

![Pestaña Aspectos básicos en el flujo de creación de áreas de trabajo](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-1.png)

Elija la cuenta de almacenamiento de ADLS Gen2 y el sistema de archivos en **Nombre de cuenta** y **Nombre del sistema de archivos**, respectivamente.

![Suministro de los detalles de una cuenta de almacenamiento de ADLS Gen2](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-2.png)

Si el creador del área de trabajo también es **Propietario** de la cuenta de almacenamiento de ADLS Gen2, Azure Synapse asignará el rol *Colaborador de datos de Storage Blob* a la identidad administrada. Verá el siguiente mensaje debajo de los detalles de la cuenta de almacenamiento que especificó.

![Asignación correcta del colaborador de datos de Storage Blob](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-3.png)

Si el creador del área de trabajo no es propietario de la cuenta de almacenamiento de ADLS Gen2, Azure Synapse no asignará el rol *Colaborador de datos de Storage Blob* a la identidad administrada. El mensaje que aparece debajo de los detalles de la cuenta de almacenamiento indica al creador del área de trabajo que no tiene permisos suficientes para conceder el rol de *Colaborador de datos de Storage Blob* a la identidad administrada.

![Error al asignar al colaborador de datos de Storage Blob](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-4.png)

Como indica el mensaje, no puede crear grupos de Spark a menos que el *colaborador de datos de Storage Blob* esté asignado a la identidad administrada.

### <a name="grant-permissions-to-managed-identity-after-workspace-creation"></a>Concesión de permisos a una identidad administrada después de la creación del área de trabajo

Durante la creación del área de trabajo, si no asigna el rol *Colaborador de datos de Storage Blob* a la identidad administrada, el **Propietario** de la cuenta de almacenamiento de ADLS Gen2 lo asigna manualmente. Los pasos siguientes le ayudarán a realizar la asignación manual.

#### <a name="step-1-navigate-to-the-adls-gen2-storage-account-in-azure-portal"></a>Paso 1: Navegación a la cuenta de almacenamiento de ADLS Gen2 en Azure Portal

En Azure Portal, abra la cuenta de almacenamiento de ADLS Gen2 y seleccione **Información general** en el panel de navegación izquierdo. Solo deberá asignar el rol de *Colaborador de datos de Storage Blob* en el nivel de contenedor o sistema de archivos. Seleccione **Contenedores**.  
![Información general de la cuenta de almacenamiento de ADLS Gen2](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-5.png)

#### <a name="step-2-select-the-container"></a>Paso 2: Selección del contenedor

La identidad administrada debe tener acceso a los datos del contenedor (sistema de archivos) que se especificó al crear el área de trabajo. Puede encontrar este contenedor o sistema de archivos en Azure Portal. Abra el área de trabajo de Azure Synapse en Azure Portal y seleccione la pestaña **Información general** en el panel de navegación izquierdo.
![Contenedor de la cuenta de almacenamiento de ADLS Gen2](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-7.png)


Seleccione el mismo contenedor o sistema de archivos para conceder el rol de *Colaborador de datos de Storage Blob* a la identidad administrada.
![Selección del contenedor de la cuenta de almacenamiento de ADLS Gen2](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-6.png)

#### <a name="step-3-navigate-to-access-control"></a>Paso 3: Navegación hasta el Control de acceso

Seleccione **Access Control (IAM)** .

![Control de acceso (IAM)](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-8.png)

#### <a name="step-4-add-a-new-role-assignment"></a>Paso 4: Adición de una asignación de roles

Seleccione **+Agregar**.

![Adición de una asignación de roles](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-9.png)

#### <a name="step-5-select-the-azure-role"></a>Paso 5: Seleccione el rol de Azure

Seleccione el rol **Colaborador de datos de Storage Blob**.

![Seleccione el rol de Azure](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-10.png)

#### <a name="step-6-select-the-azure-ad-security-principal"></a>Paso 6: Selección de la entidad de seguridad de Azure AD

En la lista desplegable **Asignar acceso a**, seleccione **Usuario, grupo o entidad de servicio de Azure AD**.

![Selección de la entidad de seguridad de AAD](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-11.png)

#### <a name="step-7-search-for-the-managed-identity"></a>Paso 7: Búsqueda de la identidad administrada

El nombre de la identidad administrada también es el nombre del área de trabajo. Para buscar su identidad administrada, escriba el nombre del área de trabajo de Azure Synapse en **Seleccionar**. Debería aparecer la identidad administrada.

![Búsqueda de la identidad administrada](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-12.png)

#### <a name="step-8-select-the-managed-identity"></a>Paso 8: Selección de la identidad administrada

Seleccione la identidad administrada para los **Miembros seleccionados**. Seleccione **Guardar** para agregar la asignación de roles.

![Selección de la identidad administrada](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-13.png)

#### <a name="step-9-verify-that-the-storage-blob-data-contributor-role-is-assigned-to-the-managed-identity"></a>Paso 9: Comprobación de que el rol de colaborador de datos de Storage Blob está asignado a la identidad administrada

Seleccione **Control de acceso (IAM)** y después **Asignaciones de roles**.

![Comprobación de la asignación de roles](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-14.png)

La identidad administrada debería mostrarse en la sección **Colaborador de datos de Storage Blob** con el rol *Colaborador de datos de Storage Blob* asignado. 
![Selección del contenedor de la cuenta de almacenamiento de ADLS Gen2](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-15.png)

## <a name="next-steps"></a>Pasos siguientes

Más información acerca de la [Identidad administrada del área de trabajo](./synapse-workspace-managed-identity.md)
