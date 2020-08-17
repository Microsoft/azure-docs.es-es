---
title: Roles y requisitos de Azure Data Share
description: Obtenga información sobre los permisos necesarios para compartir y recibir datos con Azure Data Share.
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: conceptual
ms.date: 07/30/2020
ms.openlocfilehash: 84d1ba6ff343b5f3d1f88d7ae5c618601f416e2c
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513772"
---
# <a name="roles-and-requirements-for-azure-data-share"></a>Roles y requisitos de Azure Data Share 

En este artículo se describen los roles y los permisos necesarios para compartir y recibir datos mediante el servicio Azure Data Share. 

## <a name="roles-and-requirements"></a>Roles y requisitos

Con el servicio de Azure Data Share, puede compartir datos sin intercambiar credenciales entre el proveedor de datos y el consumidor. El servicio de Azure Data Share usa identidades administradas (anteriormente conocidas como MSI) para autenticarse en el almacén de datos de Azure. 

Se debe conceder a la identidad administrada del recurso de Azure Data Share acceso al almacén de datos de Azure. Después, el servicio de Azure Data Share usa esta identidad administrada para leer y escribir datos para el uso compartido basado en instantáneas, y para establecer un vínculo simbólico para el uso compartido en contexto. 

Para compartir o recibir datos de un almacén de datos de Azure, el usuario necesita al menos los permisos siguientes. Se requieren permisos adicionales para el uso compartido basado en SQL.

* Permiso para escribir en el almacén de datos de Azure. Generalmente, este permiso existe en el rol **Colaborador**.
* Permiso para crear la asignación de roles en el almacén de datos de Azure. Normalmente, el permiso para crear asignaciones de rol existe en el rol **Propietario**, en el rol Administrador de acceso de usuario o en un rol personalizado con el permiso Microsoft.Authorization/role assignments/write (asignación de roles o escritura) asignado. Este permiso no es necesario si ya se concedió a la identidad administrada del recurso compartido de datos acceso al almacén de datos de Azure. Consulte la tabla siguiente para obtener el rol requerido.

A continuación se muestra un resumen de los roles asignados a la identidad administrada del recurso compartido de datos:

|**Tipo de almacén de datos**|**Almacén de datos de origen del proveedor de datos**|**Almacén de datos de destino del consumidor de datos**|
|---|---|---|
|Azure Blob Storage| Lector de datos de blobs de almacenamiento | Colaborador de datos de blobs de almacenamiento
|Azure Data Lake Gen1 | Propietario | No compatible
|Azure Data Lake Gen2 | Lector de datos de blobs de almacenamiento | Colaborador de datos de blobs de almacenamiento
|Azure SQL Server | Colaborador de Base de datos de SQL | Colaborador de Base de datos de SQL
|Clúster de Azure Data Explorer | Colaborador | Colaborador
|

En el caso del uso compartido basado en SQL, se debe crear un usuario de SQL a partir de un proveedor externo en Azure SQL Database con el mismo nombre que el recurso de Azure Data Share. A continuación se muestra un resumen del permiso requerido por el usuario de SQL.

|**Tipo de base de datos SQL**|**Permiso de usuario de SQL del proveedor de datos**|**Permiso de usuario de SQL del consumidor de datos**|
|---|---|---|
|Azure SQL Database | db_datareader | db_datareader, db_datawriter, db_ddladmin
|Azure Synapse Analytics (anteriormente SQL DW) | db_datareader | db_datareader, db_datawriter, db_ddladmin
|

### <a name="data-provider"></a>Proveedor de datos

Para agregar un conjunto de datos en Azure Data Share, se debe conceder a la identidad administrada del recurso compartido de datos del proveedor acceso al almacén de datos de Azure de origen. Por ejemplo, en el caso de la cuenta de almacenamiento, a la identidad administrada del recurso compartido de datos se le concede el rol Lector de datos de blobs de almacenamiento. 

Esto lo hace automáticamente el servicio de Azure Data Share cuando el usuario agrega un conjunto de datos a través de Azure Portal y tiene el permiso adecuado. Por ejemplo, el usuario es un propietario del almacén de datos de Azure o miembro de un rol personalizado que tiene asignado el permiso Microsoft.Authorization/role assignments/write (asignación de roles o escritura). 

Como alternativa, el usuario puede hacer que el propietario del almacén de datos de Azure agregue manualmente la identidad administrada del recurso compartido de datos al almacén de datos de Azure. Esta acción solo debe realizarse una vez por cada recurso compartido de datos.

Para crear una asignación de roles para la identidad administrada del recurso compartido de datos, siga estos pasos:

1. Vaya al almacén de datos de Azure.
1. Seleccione **Access Control (IAM)** .
1. Seleccione **Agregar una asignación de roles**.
1. En *Rol*, seleccione el rol en la tabla de asignación de roles anterior (por ejemplo, en cuenta de almacenamiento, seleccione *Lector de datos de blobs de almacenamiento*).
1. En *Seleccionar*, escriba el nombre del recurso de Azure Data Share.
1. Haga clic en *Save*(Guardar).

En el caso de orígenes basados en SQL, además de los pasos anteriores, se debe crear un usuario de SQL a partir de un proveedor externo en SQL Database con el mismo nombre que el recurso de Azure Data Share. Se debe conceder a este usuario el permiso *db_datareader*. Puede encontrar un script de ejemplo junto con otros requisitos previos para el uso compartido basado en SQL en el tutorial [Uso compartido de datos](share-your-data.md). 

### <a name="data-consumer"></a>Consumidor de datos
Para recibir datos, se debe conceder a la identidad administrada del recurso compartido de datos del consumidor acceso al almacén de datos de Azure de destino. Por ejemplo, en el caso de la cuenta de almacenamiento, a la identidad administrada del recurso compartido de datos se le concede el rol Colaborador de datos de blobs de almacenamiento. 

Esto lo hace automáticamente el servicio de Azure Data Share si el usuario especifica un almacén de datos de destino a través de Azure Portal y el usuario tiene el permiso adecuado. Por ejemplo, el usuario es el propietario del almacén de datos de Azure o es miembro de un rol personalizado que tiene asignado el permiso Microsoft.Authorization/role assignments/write (asignación de roles o escritura). 

Como alternativa, el usuario puede hacer que el propietario del almacén de datos de Azure agregue manualmente la identidad administrada del recurso compartido de datos al almacén de datos de Azure. Esta acción solo debe realizarse una vez por cada recurso compartido de datos.

Para crear una asignación de roles manualmente para la identidad administrada del recurso compartido de datos, siga estos pasos:

1. Vaya al almacén de datos de Azure.
1. Seleccione **Access Control (IAM)** .
1. Seleccione **Agregar una asignación de roles**.
1. En *Rol*, seleccione el rol en la tabla de asignación de roles anterior (por ejemplo, en cuenta de almacenamiento, seleccione *Lector de datos de blobs de almacenamiento*).
1. En *Seleccionar*, escriba el nombre del recurso de Azure Data Share.
1. Haga clic en *Save*(Guardar).

En el caso de un destino basado en SQL, además de los pasos anteriores, se debe crear un usuario de SQL a partir de un proveedor externo en SQL Database con el mismo nombre que el recurso de Azure Data Share. Se debe conceder a este usuario el permiso*db_datareader, db_datawriter, db_ddladmin*. En el tutorial [Aceptación y recepción de datos con Azure Data Share](subscribe-to-data-share.md), puede encontrar un script de ejemplo junto con otros requisitos previos para el uso compartido basado en SQL. 

Si va a compartir datos con las API REST, debe crear estas asignaciones de roles manualmente. 

Para más información sobre cómo agregar una asignación de roles, consulte [esta documentación](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal#add-a-role-assignment). 

## <a name="resource-provider-registration"></a>Registro del proveedor de recursos 

Es posible que tenga que registrar manualmente el proveedor de recursos Microsoft.DataShare en la suscripción de Azure en los escenarios siguientes: 

* Consulte la invitación de Azure Data Share por primera vez en su inquilino de Azure.
* Opción para compartir datos de un almacén de datos de Azure en una suscripción de Azure diferente del recurso de Azure Data Share
* Recepción de datos en un almacén de datos de Azure en una suscripción de Azure diferente del recurso de Azure Data Share

Siga estos pasos para registrar el proveedor de recursos Microsoft.DataShare en la suscripción de Azure. Necesita acceso de *Colaborador* a la suscripción de Azure para registrar el proveedor de recursos.

1. En Azure Portal, vaya a **Suscripciones**.
1. Seleccione la suscripción que va a usar para Azure Data Share.
1. Haga clic en **Proveedores de recursos**
1. Busque Microsoft.DataShare.
1. Haga clic en **Registrar**.

## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de los roles en Azure - [Descripción de definiciones de roles](../role-based-access-control/role-definitions.md)
