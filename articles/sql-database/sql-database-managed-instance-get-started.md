---
title: 'Azure Portal: Creación de una Instancia administrada de SQL | Microsoft Docs'
description: Cree una instancia administrada de SQL, un entorno de red y una máquina virtual cliente para el acceso.
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: ''
ms.devlang: ''
ms.topic: quickstart
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: Carlrab
manager: craigg
ms.date: 02/18/2019
ms.openlocfilehash: 481c82eb74bcf80c3d0546324009ec0bf6495cfb
ms.sourcegitcommit: 7723b13601429fe8ce101395b7e47831043b970b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/21/2019
ms.locfileid: "56587062"
---
# <a name="quickstart-create-an-azure-sql-database-managed-instance"></a>Inicio rápido: Creación de una Instancia administrada de Azure SQL Database

Este inicio rápido le guía por la creación de una [Instancia administrada](sql-database-managed-instance.md) de Azure SQL Database en Azure Portal.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en el [Azure Portal](https://portal.azure.com/).

## <a name="create-a-managed-instance"></a>Creación de una instancia administrada

En los pasos siguientes se muestra cómo crear una Instancia administrada.

1. Elija **Crear un recurso** en la esquina superior izquierda de Azure Portal.
2. Busque **Instancia administrada** y, después, seleccione **Instancia administrada de Azure SQL**.
3. Seleccione **Crear**.

   ![Crear una instancia administrada](./media/sql-database-managed-instance-get-started/managed-instance-create.png)

4. Rellene el formulario **Instancia administrada de SQL** con la información solicitada, para lo que debe usar los datos de la siguiente tabla:

   | Configuración| Valor sugerido | DESCRIPCIÓN |
   | ------ | --------------- | ----------- |
   | **Suscripción** | Su suscripción | Una suscripción en la que tiene permiso para crear recursos. |
   |**Nombre de la instancia administrada**|Cualquier nombre válido|Para conocer cuáles son los nombres válidos, consulte el artículo [Convenciones de nomenclatura para los recursos de Azure](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions).|
   |**Inicio de sesión de administrador de la instancia administrada**|Cualquier nombre de usuario válido|Para conocer cuáles son los nombres válidos, consulte el artículo [Convenciones de nomenclatura para los recursos de Azure](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions). No utilice "serveradmin", ya es un rol de nivel de servidor reservado.|
   |**Contraseña**|Cualquier contraseña válida|La contraseña debe tener al menos 16 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).|
   |**Intercalación**|La intercalación que desea usar para una instancia administrada|Si va a migrar bases de datos desde SQL Server, compruebe la intercalación de origen mediante `SELECT SERVERPROPERTY(N'Collation')` y use ese valor. Para información sobre las intercalaciones, consulte [Intercalaciones de nivel de servidor](https://docs.microsoft.com/sql/relational-databases/collations/set-or-change-the-server-collation).|
   |**Ubicación**|La ubicación en la que quiere crear la Instancia administrada|Para obtener información sobre las regiones, consulte [Regiones de Azure](https://azure.microsoft.com/regions/).|
   |**Red virtual**|Seleccione **Crear una nueva red virtual** o una subred y red virtual válidas.| Si hay una red o subred que aparece deshabilitada, se debe [modificar para cumplir con los requisitos de la red](sql-database-managed-instance-configure-vnet-subnet.md) antes de seleccionarla como destino de la nueva instancia administrada. Para información acerca los requisitos necesarios para configurar el entorno de red de una instancia administrada, consulte [Arquitectura de conectividad de la Instancia administrada de Azure SQL Database](sql-database-managed-instance-connectivity-architecture.md). |
   |**Grupos de recursos**|Un grupo de recursos nuevo o existente.|Para conocer cuáles son los nombres de grupo de recursos válidos, consulte el artículo [Naming conventions](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions) (Convenciones de nomenclatura).|

   ![Formulario de la instancia administrada](./media/sql-database-managed-instance-get-started/managed-instance-create-form.png)

5. Para utilizar la instancia administrada como un grupo de conmutación por error de la instancia secundario, seleccione la restauración y especifique la instancia administrada de DnsAzurePartner. Esta característica está en versión preliminar y no se muestra en la captura de pantalla que lo acompaña.
6. Seleccione **Plan de tarifa** para cambiar el tamaño de los recursos de almacenamiento y de proceso, así como para revisar las opciones del plan de tarifa. El plan de tarifa de uso general con 32 GB de memoria y 16 núcleos de virtuales es el valor predeterminado.
7. Utilice los controles deslizantes o cuadros de texto para especificar la cantidad de almacenamiento y el número de núcleos virtuales.
8. Cuando haya terminado, elija **Aplicar** para guardar la selección.  
9. Seleccione **Crear** para implementar la instancia administrada.
10. Seleccione el icono **Notificaciones** para ver el estado de la implementación.

    ![Progreso de la implementación de la Instancia administrada](./media/sql-database-managed-instance-get-started/deployment-progress.png)

11. Seleccione **Implementación en curso** para abrir la ventana de la instancia administrada y supervisar el progreso de la implementación.

> [!IMPORTANT]
> En la primera instancia de una subred el tiempo de implementación habitualmente es mucho mayor que en las siguientes. No cancele la operación de implementación porque dure más de lo previsto. La creación de la segunda instancia administrada en la subred solo tarda un par de minutos.

## <a name="review-resources-and-retrieve-your-fully-qualified-server-name"></a>Revisión de los recursos y recuperación del nombre del servidor completo

Cuando la implementación finalice correctamente, revise los recursos creados y recupere el nombre completo del servidor para su uso en tutoriales posteriores.

1. Abra el grupo de recursos de la instancia administrada y vea los recursos que se crearon de forma automática en el inicio rápido [Creación de una instancia administrada](#create-a-managed-instance).

   ![Recursos de la instancia administrada](./media/sql-database-managed-instance-get-started/resources.png)

2. Seleccione la tabla de rutas para revisar la ruta definida por el usuario (UDR) que se creó automáticamente.

   ![Tabla de rutas](./media/sql-database-managed-instance-get-started/route-table.png)

3. En la tabla de rutas, revise las entradas para enrutar el tráfico desde la parte interior de la red virtual de instancia administrada. Si va a crear o a configurar manualmente la tabla de rutas, debe asegurarse de crear estas entradas.

   ![Entrada de la subred de Instancia administrada local](./media/sql-database-managed-instance-get-started/udr.png)

4. Vuelva al grupo de recursos y seleccione el grupo de seguridad de red para revisar las reglas de seguridad.

   ![Grupo de seguridad de red](./media/sql-database-managed-instance-get-started/network-security-group.png)

5. Revise las reglas de seguridad entrantes y salientes.

   ![Reglas de seguridad](./media/sql-database-managed-instance-get-started/security-rules.png)

6. Vuelva al grupo de recursos y seleccione su instancia administrada.

   ![Instancia administrada](./media/sql-database-managed-instance-get-started/managed-instance.png)

7. En la pestaña **Información general**, busque la propiedad **Host** y copie la dirección de host completa de la instancia administrada para usarla en el siguiente inicio rápido.

   ![Nombre de host](./media/sql-database-managed-instance-get-started/host-name.png)

   El nombre es similar **nombre_del_equipo.a1b2c3d4e5f6.database.windows.net**.

## <a name="next-steps"></a>Pasos siguientes

- Para información sobre cómo conectarse a una Instancia administrada, consulte:
  - Para información general de las opciones de conexión para las aplicaciones, consulte [Conexión de la aplicación a Instancia administrada de Azure SQL Database](sql-database-managed-instance-connect-app.md).
  - Para obtener un inicio rápido en el que se muestra cómo conectarse a una instancia administrada desde una máquina virtual de Azure, consulte [Guía de inicio rápido: Configurar una máquina virtual de Azure para la conexión a una Instancia administrada de Azure SQL Database](sql-database-managed-instance-configure-vm.md).
  - Para obtener un inicio rápido en el que se muestra cómo conectarse a una instancia administrada desde un equipo cliente local mediante una conexión de punto a sitio, consulte [Inicio rápido: Configuración de una conexión de punto a sitio a una Instancia administrada de Azure SQL Database desde el entorno local](sql-database-managed-instance-configure-p2s.md).
- Para restaurar una base de datos SQL Server existente desde el entorno local en una Instancia administrada, puede usar [Azure Database Migration Service (DMS) para la migración](../dms/tutorial-sql-server-to-managed-instance.md) a fin de restaurar desde un archivo de copia de seguridad de base de datos, o bien el [comando T-SQL RESTORE](sql-database-managed-instance-get-started-restore.md) para restaurar desde un archivo de copia de seguridad de base de datos.
- Para una supervisión avanzada del rendimiento de la base de datos de Instancia administrada con inteligencia de solución de problemas integrada, vea [Supervisión de instancias de Azure SQL Database con Azure SQL Analytics](../azure-monitor/insights/azure-sql.md).
