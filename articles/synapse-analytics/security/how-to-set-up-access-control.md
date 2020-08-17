---
title: Protección del área de trabajo de Synapse (versión preliminar)
description: En este artículo se explica cómo usar roles y el control de acceso para controlar las actividades y el acceso a los datos en el área de trabajo de Synapse.
services: synapse-analytics
author: matt1883
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 04/15/2020
ms.author: mahi
ms.reviewer: jrasnick
ms.openlocfilehash: 6b1fe78ac00b57d7627a64970e84ae8b717ae5c3
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501923"
---
# <a name="secure-your-synapse-workspace-preview"></a>Protección del área de trabajo de Synapse (versión preliminar)

En este artículo se explica cómo usar roles y el control de acceso para controlar las actividades y el acceso a los datos. Si sigue estas instrucciones, se simplificará el control de acceso en Azure Synapse Analytics. Solo debe agregar y quitar usuarios de uno de los tres grupos de seguridad.

## <a name="overview"></a>Información general

Para proteger un área de trabajo de Synapse (versión preliminar), debe seguir un patrón de configuración para los siguientes elementos:

- Roles de Azure (como los roles integrados de propietario, colaborador, etc.)
- Roles de Synapse: estos roles son exclusivos de Synapse y no están basados en los roles de Azure. Hay tres roles de este tipo:
  - Administrador del área de trabajo de Synapse
  - Administrador de SQL de Synapse
  - Apache Spark para el administrador de Azure Synapse Analytics
- Control de acceso a los datos de Azure Data Lake Storage Gen 2 (ADLSGEN2).
- Control de acceso a las bases de datos de SQL y Spark de Synapse

## <a name="steps-to-secure-a-synapse-workspace"></a>Pasos para proteger un área de trabajo de Synapse

En este documento se usan nombres estándar para simplificar las instrucciones. Reemplácelos por los nombres de su preferencia.

|Configuración | Valor de ejemplo | Descripción |
| :------ | :-------------- | :---------- |
| **Área de trabajo de Synapse** | WS1 |  Nombre que tendrá el área de trabajo de Synapse. |
| **Cuenta de ADLSGEN2** | STG1 | Cuenta de ADLS que se usará con el área de trabajo. |
| **Contenedor** | CNT1 | Contenedor en STG1 que el área de trabajo usará de forma predeterminada. |
| **Inquilino de Active Directory** | contoso | Nombre del inquilino de Active Directory.|
||||

## <a name="step-1-set-up-security-groups"></a>PASO 1: Configuración de los grupos de seguridad

Cree y rellene tres grupos de seguridad para el área de trabajo:

- **WS1\_WSAdmins**: para los usuarios que necesitan control completo sobre el área de trabajo.
- **WS1\_SparkAdmins**: para aquellos usuarios que necesitan control completo sobre las características de Spark del área de trabajo.
- **WS1\_SQLAdmins**: para aquellos usuarios que necesitan control completo sobre las características de SQL del área de trabajo.
- Agregue **WS1\_WSAdmins** a **WS1\_SQLAdmins**
- Agregue **WS1\_WSAdmins** a **WS1\_SparkAdmins**

## <a name="step-2-prepare-your-data-lake-storage-gen2-account"></a>PASO 2: Preparación de una cuenta de Azure Data Lake Storage Gen2

Identifique esta información sobre su instancia de almacenamiento:

- La cuenta de ADLSGEN2 que usará para el área de trabajo. En este documento, se llama STG1.  STG1 se considera la cuenta de almacenamiento "principal" del área de trabajo.
- El contenedor dentro de WS1 que el área de trabajo de Synapse usará de forma predeterminada. En este documento, se llama CNT1.  Este contenedor se usa para lo siguiente:
  - Almacenar los archivos de datos de copia de seguridad para las tablas de Spark
  - Los registros de ejecución de los trabajos de Spark

- Mediante Azure Portal, asigne los grupos de seguridad a los siguientes roles de CNT1.

  - Asigne el grupo **WS1\_WSAdmins** al rol **Colaborador de datos de Storage Blob**.
  - Asigne el grupo **WS1\_SparkAdmins** al rol **Colaborador de datos de Storage Blob**.
  - Asigne el grupo **WS1\_SQLAdmins** al rol **Colaborador de datos de Storage Blob**.

## <a name="step-3-create-and-configure-your-synapse-workspace"></a>PASO 3: Creación y configuración del área de trabajo de Synapse

En Azure Portal, cree un área de trabajo de Synapse:

- Asigne el nombre WS1 al área de trabajo.
- Elija STG1 para la cuenta de almacenamiento.
- Elija CNT1 para el contenedor que se usará como "sistema de archivos".
- Abra WS1 en Synapse Studio.
- Seleccione **Administrar** > **Control de acceso** y asigne los grupos de seguridad a los siguientes roles de Synapse.
  - Asigne **WS1\_WSAdmins** a los administradores del área de trabajo de Synapse.
  - Asigne **WS1\_SparkAdmins** a los administradores de Spark de Synapse.
  - Asigne **WS1\_SQLAdmins** a los administradores de SQL de Synapse.

## <a name="step-4-configure-data-lake-storage-gen2-for-use-by-synapse-workspace"></a>PASO 4: Configuración de Data Lake Storage Gen2 para que lo use el área de trabajo de Synapse

El área de trabajo de Synapse necesita acceso a STG1 y CNT1 para que ejecutar canalizaciones y realizar tareas del sistema.

- Abra Azure Portal.
- Busque la cuenta STG1.
- Navegue al contenedor CNT1.
- Asegúrese de que el archivo MSI (Managed Service Identity) para WS1 esté asignado al rol **Colaborador de datos de Storage Blob** en CNT1.
  - Si no lo está, asígnelo.
  - El MSI tiene el mismo nombre que el área de trabajo. En este caso, sería &quot;WS1&quot;.

## <a name="step-5-configure-admin-access-for-sql-pools"></a>PASO 5: Configuración del acceso de administrador para los grupos de SQL

- Abra Azure Portal.
- Vaya a al área de trabajo WS1.
- En **Configuración**, haga clic en **SQL Active Directory admin** (Administrador de SQL Active Directory).
- Haga clic en **Establecer administrador** y seleccione WS1\_SQLAdmins.

## <a name="step-6-maintain-access-control"></a>PASO 6: Mantenimiento del control de acceso

La configuración se ha finalizado.

Ahora, para administrar el acceso de los usuarios, puede agregar y quitar usuarios de los tres grupos de seguridad.

Aunque puede asignar los usuarios manualmente a los roles de Synapse, al hacerlo no configurará las cosas de forma coherente. En su lugar, solo tiene que agregar o quitar usuarios de los grupos de seguridad.

## <a name="step-7-verify-access-for-users-in-the-roles"></a>PASO 7: Comprobación del acceso para los usuarios de los roles

Los usuarios de cada rol deben completar los siguientes pasos:

| Number | Paso | Administradores de áreas de trabajo | Administradores de Spark | Administradores de SQL |
| --- | --- | --- | --- | --- |
| 1 | Cargar un archivo Parquet en CNT1 | SÍ | SÍ | SÍ |
| 2 | Leer el archivo Parquet mediante SQL a petición | SÍ | No | SÍ |
| 3 | Crear un grupo de Spark | SÍ [1] | SÍ [1] | No  |
| 4 | Leer el archivo Parquet con una instancia de Notebook | SÍ | SÍ | No |
| 5 | Crear una canalización desde la instancia de Notebook y desencadenar la canalización para ejecutarse ahora | SÍ | No | No |
| 6 | Crear un grupo de SQL y ejecutar un script de SQL como &quot;SELECT 1&quot; | SÍ [1] | No | SÍ [1] |

> [!NOTE]
> [1] Para crear grupos de SQL o Spark, el usuario debe tener como mínimo el rol de Colaborador en el área de trabajo de Synapse.
> [!TIP]
>
> - Algunos de los pasos no estarán permitidos de forma deliberada en función del rol.
> - Tenga en cuenta que pueden producirse errores en algunas tareas si la seguridad no se configuró por completo. Dichas tareas se indican en la tabla.

## <a name="step-8-network-security"></a>PASO 8: Seguridad de redes

Para configurar el firewall del área de trabajo, la red virtual y la instancia de [Private Link](../../azure-sql/database/private-endpoint-overview.md).

## <a name="step-9-completion"></a>PASO 9: Completion

El área de trabajo ahora está completamente configurada y protegida.

## <a name="how-roles-interact-with-synapse-studio"></a>Cómo interactúan los roles con Synapse Studio

Synapse Studio se comportará de forma diferente en función de los roles de usuario. Algunos elementos pueden estar ocultos o deshabilitados si un usuario no tiene asignados los roles que proporcionan el acceso adecuado. En la siguiente tabla se resumen los efectos en Synapse Studio.

| Tarea | Administradores de áreas de trabajo | Administradores de Spark | Administradores de SQL |
| --- | --- | --- | --- |
| Abrir Synapse Studio | SÍ | SÍ | SÍ |
| Ver el centro de Inicio | SÍ | SÍ | SÍ |
| Ver el centro de datos | SÍ | SÍ | SÍ |
| Centro de datos / ver los contenedores y las cuentas de ADLS Gen2 vinculados | SÍ [1] | SÍ [1] | SÍ [1] |
| Centro de datos: ver las bases de datos | SÍ | SÍ | SÍ |
| Centro de datos: ver los objetos en las bases de datos | SÍ | SÍ | SÍ |
| Centro de datos: acceso a datos en las bases de datos del grupo de SQL | SÍ   | No   | SÍ   |
| Centro de datos: acceso a datos en las bases de datos de SQL a petición | SÍ [2]  | No  | SÍ [2]  |
| Centro de datos: acceso a datos en las bases de datos de Spark | SÍ [2] | SÍ [2] | SÍ [2] |
| Usar el centro de desarrollo | SÍ | SÍ | SÍ |
| Centro de desarrollo: crear scripts de SQL | SÍ | No | SÍ |
| Centro de desarrollo: crear definiciones de trabajos de Spark | SÍ | SÍ | No |
| Centro de desarrollo: crear instancias de Notebook | SÍ | SÍ | No |
| Centro de desarrollo: crear flujos de datos | SÍ | No | No |
| Usar el centro de orquestación | SÍ | SÍ | SÍ |
| Centro de orquestación: usar las canalizaciones | SÍ | No | No |
| Usar el centro de administración | SÍ | SÍ | SÍ |
| Centro de administración: grupos de SQL | SÍ | No | SÍ |
| Centro de administración: grupos de Spark | SÍ | SÍ | No |
| Centro de administración: desencadenadores | SÍ | No | No |
| Centro de administración: servicios vinculados | SÍ | SÍ | SÍ |
| Centro de administración: control de acceso (asignar usuarios a los roles del área de trabajo de Synapse) | SÍ | No | No |
| Centro de administración: entornos de ejecución de integración | SÍ | SÍ | SÍ |
| Uso del concentrador de supervisión | SÍ | SÍ | SÍ |
| Concentrador de supervisión/orquestación/ejecuciones de canalizaciones  | SÍ | No | No |
| Concentrador de supervisión/orquestación/ejecuciones de desencadenadores  | SÍ | No | No |
| Concentrador de supervisión/orquestación/tiempo de ejecución de integraciones  | SÍ | SÍ | SÍ |
| Concentrador de supervisión/actividades/aplicaciones de Spark | SÍ | SÍ | No  |
| Concentrador de supervisión/actividades/solicitudes SQL | SÍ | No | SÍ |
| Concentrador de supervisión/actividades/grupos de Spark | SÍ | SÍ | No  |
| Concentrador de supervisión/desencadenadores | SÍ | No | No |
| Centro de administración: servicios vinculados | SÍ | SÍ | SÍ |
| Centro de administración: control de acceso (asignar usuarios a los roles del área de trabajo de Synapse) | SÍ | No | No |
| Centro de administración: entornos de ejecución de integración | SÍ | SÍ | SÍ |


> [!NOTE]
> [1] El acceso a los datos de los contenedores depende del control de acceso en ADLS Gen2. </br>
> [2] Las tablas de SQL a petición y las tablas de Spark almacenan datos en ADLS Gen2, por lo que el acceso requiere los permisos adecuados en ADLS Gen2.

## <a name="next-steps"></a>Pasos siguientes

Creación de un [área de trabajo de Synapse](../quickstart-create-workspace.md)
