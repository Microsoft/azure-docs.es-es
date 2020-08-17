---
title: Administración de Azure Data Lake Analytics con el portal de Azure
description: En este artículo se describe cómo usar Azure Portal para administrar cuentas, orígenes de datos, usuarios y trabajos de Data Lake Analytics.
services: data-lake-analytics
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.assetid: a0e045f1-73d6-427f-868d-7b55c10f811b
ms.topic: how-to
ms.date: 12/05/2016
ms.openlocfilehash: cbce903bc7463cd917d48e341614afbe18536262
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87531219"
---
# <a name="manage-azure-data-lake-analytics-using-the-azure-portal"></a>Administración de Azure Data Lake Analytics mediante Azure Portal
[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

En este artículo se describe cómo administrar cuentas, orígenes de datos, usuarios y trabajos de Azure Data Lake Analytics mediante Azure Portal.


<!-- ################################ -->
<!-- ################################ -->

## <a name="manage-data-lake-analytics-accounts"></a>Administración de cuentas de Data Lake Analytics

### <a name="create-an-account"></a>Crear una cuenta

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Haga clic en **Crear un recurso** > **Inteligencia y análisis** > **Data Lake Analytics**.
3. Seleccione los valores de los siguientes elementos: 
   1. **Name**: nombre de la cuenta de Data Lake Analytics.
   2. **Suscripción**: suscripción de Azure usada para la cuenta.
   3. **Grupo de recursos**: grupo de recursos de Azure en el que se crea la cuenta. 
   4. **Ubicación**: centro de datos de Azure para la cuenta de Data Lake Analytics. 
   5. **Data Lake Store**: almacén predeterminado que se va a usar para la cuenta de Data Lake Analytics. Las cuentas de Azure Data Lake Store y de Data Lake Analytics deben estar en la misma ubicación.
4. Haga clic en **Crear**. 

### <a name="delete-a-data-lake-analytics-account"></a>Eliminar una cuenta de Data Lake Analytics

Para eliminar una cuenta de Data Lake Analytics, elimine la cuenta de Data Lake Store predeterminada.

1. En Azure Portal, vaya a la cuenta de Data Lake Analytics.
2. Haga clic en **Eliminar**.
3. Escriba el nombre de la cuenta.
4. Haga clic en **Eliminar**.

<!-- ################################ -->
<!-- ################################ -->

## <a name="manage-data-sources"></a>Administración de orígenes de datos

Data Lake Analytics admite los siguientes orígenes de datos:

* Data Lake Store
* Azure Storage

Puede usar el explorador de datos para examinar los orígenes de datos y realizar operaciones básicas de administración de archivos. 

### <a name="add-a-data-source"></a>Agregar un origen de datos

1. En Azure Portal, vaya a la cuenta de Data Lake Analytics.
2. Haga clic en **Orígenes de datos**.
3. Haga clic en **Agregar origen de datos**.
    
   * Para agregar una cuenta de Data Lake Store, necesita el nombre de la cuenta y acceso a ella para poder realizar consultas.
   * Para agregar Azure Blob Storage, necesita la cuenta y la clave de almacenamiento. Para encontrarlas, vaya a la cuenta de almacenamiento en el portal.

## <a name="set-up-firewall-rules"></a>Configurar reglas de firewall

Puede usar Data Lake Analytics para bloquear aún más el acceso a la cuenta de Data Lake Analytics en el nivel de red. Puede habilitar un firewall, especificar una dirección IP o definir un intervalo de direcciones IP para los clientes de confianza. Una vez habilitadas estas medidas, solo los clientes con direcciones IP del intervalo definido pueden conectarse al almacén.

Si otros servicios de Azure, como Azure Data Factory o las máquinas virtuales, se conectan a la cuenta de Data Lake Analytics, asegúrese de que **Permitir los servicios de Azure** esté **Activado**. 

### <a name="set-up-a-firewall-rule"></a>Configurar una regla de firewall

1. En Azure Portal, vaya a la cuenta de Data Lake Analytics.
2. En el menú de la izquierda, haga clic en **Firewall**.

## <a name="add-a-new-user"></a>Agregar un nuevo usuario

Puede usar el **Asistente para agregar usuario** para aprovisionar fácilmente nuevos usuarios de Data Lake.

1. En Azure Portal, vaya a la cuenta de Data Lake Analytics.
2. A la izquierda, en **Introducción**, haga clic en **Asistente para agregar usuario**.
3. Seleccione un usuario y luego haga clic en **Seleccionar**.
4. Seleccione un rol y luego haga clic en **Seleccionar**. Para configurar un nuevo desarrollador que use Azure Data Lake, seleccione el rol **Desarrollador de Data Lake Analytics**.
5. Seleccione las listas de control de acceso (ACL) para las bases de datos de U-SQL. Cuando esté satisfecho con las opciones seleccionadas, haga clic en **Seleccionar**.
6. Seleccione las ACL para los archivos. Para el almacén predeterminado, no cambie las ACL de la carpeta raíz "/" ni de la carpeta /system. Haga clic en **Seleccionar**.
7. Revise todos los cambios seleccionados y luego haga clic en **Ejecutar**.
8. Al finalizar el asistente, haga clic en **Listo**.

## <a name="manage-role-based-access-control"></a>Administrar el control de acceso basado en roles

Al igual que otros servicios de Azure, puede usar el control de acceso basado en roles (RBAC) para controlar cómo interactúan los usuarios con el servicio.

Los roles estándar de Azure tienen las siguientes funcionalidades:
* **Propietario**: puede enviar, supervisar y cancelar trabajos de cualquier usuario y configurar la cuenta.
* **Colaborador**: puede enviar, supervisar y cancelar trabajos de cualquier usuario y configurar la cuenta.
* **Lector**: puede supervisar trabajos.

Emplee el rol de desarrollador de Data Lake Analytics para permitir que los desarrolladores de U-SQL usen el servicio Data Lake Analytics. Puede usar el rol de desarrollador de Data Lake Analytics para:
* Enviar trabajos.
* Supervisar el estado y el progreso de los trabajos enviados por cualquier usuario.
* Ver los scripts de U-SQL de los trabajos enviados por cualquier usuario.
* Cancelar solo sus propios trabajos.

### <a name="add-users-or-security-groups-to-a-data-lake-analytics-account"></a>Agregar usuarios o grupos de seguridad a una cuenta de Data Lake Analytics

1. En Azure Portal, vaya a la cuenta de Data Lake Analytics.
2. Haga clic en **Control de acceso (IAM)**  > **Agregar asignación de roles**.
3. Seleccione un rol.
4. Agregue un usuario.
5. Haga clic en **OK**.

>[!NOTE]
>Si un usuario o un grupo de seguridad necesita enviar trabajos, también necesita permiso en la cuenta de almacenamiento. Para más información, vea [Secure data stored in Data Lake Store (Protección de datos almacenados en Data Lake Store)](../data-lake-store/data-lake-store-secure-data.md).
>

<!-- ################################ -->
<!-- ################################ -->

## <a name="manage-jobs"></a>Trabajos de administración

### <a name="submit-a-job"></a>Enviar un trabajo

1. En Azure Portal, vaya a la cuenta de Data Lake Analytics.

2. Haga clic en **Nuevo trabajo**. Para cada trabajo, configure:

    1. **Nombre del trabajo**: Nombre del trabajo.
    2. **Prioridad**: los números más bajos tienen mayor prioridad. Si hay dos trabajos en cola, se ejecuta primero el que tenga un valor de prioridad más bajo.
    3. **Paralelismo**: número máximo de procesos de cálculo que se van a reservar para este trabajo.

3. Haga clic en **Enviar trabajo**.

### <a name="monitor-jobs"></a>Supervisión de trabajos

1. En Azure Portal, vaya a la cuenta de Data Lake Analytics.
2. Haga clic en **Ver todos los trabajos**. Se muestra una lista de todos los trabajos activos y finalizados recientemente en la cuenta.
3. De forma opcional, haga clic en **Filtrar** para encontrar los trabajos por los valores **Intervalo de tiempo**, **Nombre del trabajo** y **Autor**. 

### <a name="monitoring-pipeline-jobs"></a>Supervisión de trabajos de canalización
Los trabajos que forman parte de un trabajo de canalización actúan juntos, normalmente de forma secuencial, para lograr un escenario concreto. Por ejemplo, puede tener una canalización que limpia, extrae, transforma y agrega el uso para información del cliente. Los trabajos de canalización se identifican mediante la propiedad "Pipeline" al enviar el trabajo. En los trabajos programados con ADF V2 esta propiedad se rellena de forma automática. 

Para ver una lista de trabajos de U-SQL que forman parte de canalizaciones: 

1. En Azure Portal, vaya a las cuentas de Data Lake Analytics.
2. Haga clic en **Información de trabajos**. La pestaña "Todos los trabajos" aparece de forma predeterminada y muestra una lista de trabajos en ejecución, en cola y finalizados.
3. Haga clic en la pestaña **Trabajos de canalización**. Se muestra una lista de trabajos de canalización junto con estadísticas agregadas de cada canalización.

### <a name="monitoring-recurring-jobs"></a>Supervisión de trabajos periódicos
Un trabajo periódico es aquel que tiene la misma lógica de negocios pero usa distintos datos de entrada cada vez que se ejecuta. Lo ideal sería que los trabajos periódicos siempre fueran correctos y tuvieran un tiempo de ejecución relativamente estable; la supervisión de estos comportamientos ayuda a garantizar que el trabajo sea correcto. Los trabajos periódicos se identifican mediante la propiedad "Recurrence". En los trabajos programados con ADF V2 esta propiedad se rellena de forma automática.

Para ver una lista de trabajos de U-SQL que son periódicos: 

1. En Azure Portal, vaya a las cuentas de Data Lake Analytics.
2. Haga clic en **Información de trabajos**. La pestaña "Todos los trabajos" aparece de forma predeterminada y muestra una lista de trabajos en ejecución, en cola y finalizados.
3. Haga clic en la pestaña **Trabajos periódicos**. Se muestra una lista de trabajos periódicos junto con estadísticas agregadas de cada uno de ellos.

## <a name="next-steps"></a>Pasos siguientes

* [Información general de Azure Data Lake Analytics](data-lake-analytics-overview.md)
* [Manage Azure Data Lake Analytics by using Azure PowerShell](data-lake-analytics-manage-use-powershell.md) (Administración de Azure Data Lake Analytics mediante Azure PowerShell)
* [Administración de Azure Data Lake Analytics con directivas](data-lake-analytics-account-policies.md)
