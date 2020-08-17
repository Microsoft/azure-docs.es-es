---
title: Importe o exporte una base de datos de Azure SQL Database sin permitir que los servicios de Azure accedan al servidor.
description: Importe o exporte una base de datos de Azure SQL Database sin permitir que los servicios de Azure accedan al servidor.
services: sql-database
ms.service: sql-database
ms.subservice: migration
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/08/2020
ms.openlocfilehash: ea5f2d5838c926fa8ee7b92278b0854264346a7b
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543765"
---
# <a name="import-or-export-an-azure-sql-database-without-allowing-azure-services-to-access-the-server"></a>Importación o exportación de una base de datos de Azure SQL Database sin permitir que los servicios de Azure accedan al servidor
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se muestra cómo importar o exportar una base de datos de Azure SQL Database cuando *Permitir servicios de Azure* esté establecido en *DESACTIVADO* en el servidor. El flujo de trabajo usa una máquina virtual de Azure para ejecutar SqlPackage y realizar la operación de importación o exportación.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="create-the-azure-virtual-machine"></a>Creación de la máquina virtual de Azure

Seleccione el botón **Implementar en Azure** para crear una máquina virtual de Azure.

Esta plantilla le permite implementar una máquina virtual de Windows simple mediante diversas opciones de versión de Windows con el parche más reciente. Esto implementará una máquina virtual de tamaño A2 en la ubicación del grupo de recursos y devolverá el nombre de dominio completo de la máquina virtual.
<br><br>

[![Imagen que muestra un botón con la etiqueta "Implementar en Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-vm-simple-windows%2Fazuredeploy.json)

Para obtener más información, consulte [Implementación muy simple de una máquina virtual de Windows](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows).

## <a name="connect-to-the-virtual-machine"></a>Conexión a la máquina virtual

En los pasos siguientes se muestra cómo conectarse a la máquina virtual mediante una conexión de escritorio remoto.

1. Una vez finalizada la implementación, vaya al recurso de máquina virtual.

   ![máquina virtual](./media/database-import-export-azure-services-off/vm.png)  

2. Seleccione **Conectar**.

   Aparece un formulario de archivo de Protocolo de escritorio remoto (archivo .rdp) con la dirección IP pública y el número de puerto de la máquina virtual.

   ![Formulario RDP](./media/database-import-export-azure-services-off/rdp.png)  

3. Seleccione **Descargar archivo RDP**.

   > [!NOTE]
   > También puede usar SSH para conectarse a la máquina virtual.

4. Cierre el formulario **Conexión a la máquina virtual**.
5. Para conectarse a la máquina virtual, abra el archivo RDP descargado.
6. Cuando se le pida, seleccione **Connect** (Conectar). En un equipo Mac, necesita un cliente RDP como este [Cliente de Escritorio remoto](https://apps.apple.com/app/microsoft-remote-desktop-10/id1295203466?mt=12) de Mac App Store.

7. Escriba el nombre de usuario y la contraseña que especificó al crear la máquina virtual y, a continuación, elija **Aceptar**.

8. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Elija **Sí** o en **Continuar** para continuar con la conexión.

## <a name="install-sqlpackage"></a>Instalación de SqlPackage

[Descargue e instale la versión más reciente de SqlPackage](https://docs.microsoft.com/sql/tools/sqlpackage-download).

Para más información, vea [SqlPackage.exe](https://docs.microsoft.com/sql/tools/sqlpackage).

## <a name="create-a-firewall-rule-to-allow-the-vm-access-to-the-database"></a>Creación de una regla de firewall para permitir que la máquina virtual acceda al servidor

Agregue la dirección IP pública de la máquina virtual al firewall del servidor.

En los pasos siguientes se crea una regla de Firewall de IP de nivel de servidor para la dirección IP pública de la máquina virtual y se habilita la conectividad de la máquina virtual.

1. Seleccione **SQL Database** en el menú de la izquierda y seleccione la base de datos en la página **SQL Database**. Se abre la página de información general de la base de datos, que muestra el nombre completo del servidor (por ejemplo, **servername.database.windows.net**) y ofrece otras opciones de configuración.

2. Copie este nombre de servidor completo para conectarse a su servidor y a sus bases de datos.

   ![nombre del servidor](./media/database-import-export-azure-services-off/server-name.png)

3. Seleccione **Establecer el firewall del servidor** en la barra de herramientas. Se abrirá la página **Configuración del firewall** del servidor.

   ![Regla de firewall de IP en el nivel de servidor](./media/database-import-export-azure-services-off/server-firewall-rule.png)

4. Seleccione **Agregar IP de cliente** en la barra de herramientas para agregar la dirección IP pública de la máquina virtual a la nueva regla de firewall de IP en el nivel de servidor. Esta regla de firewall puede abrir el puerto 1433 para una única dirección IP o un intervalo de direcciones IP.

5. Seleccione **Guardar**. Se crea una regla de firewall de IP en el nivel de servidor para el puerto 1433 de la dirección IP pública de la máquina virtual en el servidor.

6. Cierre la página **Configuración de firewall**.

## <a name="export-a-database-using-sqlpackage"></a>Exportación de una base de datos mediante SqlPackage

Para exportar una base de datos de Azure SQL Database mediante la utilidad de línea de comandos [SqlPackage](https://docs.microsoft.com/sql/tools/sqlpackage), consulte [Parámetros y propiedades de la exportación](https://docs.microsoft.com/sql/tools/sqlpackage#export-parameters-and-properties). La utilidad SqlPackage incluye las versiones más recientes de [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) y [SQL Server Data Tools](https://docs.microsoft.com/sql/ssdt/download-sql-server-data-tools-ssdt). También puede descargar la versión más reciente de [SqlPackage](https://docs.microsoft.com/sql/tools/sqlpackage-download).

Se recomienda usar la utilidad SqlPackage para la escala y el rendimiento en la mayoría de los entornos de producción. Consulte cómo [migrar de SQL Server a Azure SQL Database con archivos BACPAC](https://blogs.msdn.microsoft.com/sqlcat/20../../migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/) en el blog de Customer Advisory Team de SQL Server.

Este ejemplo muestra cómo exportar una base de datos con SqlPackage.exe con Autenticación universal de Active Directory. Reemplace los valores por otros específicos de su entorno.

```cmd
SqlPackage.exe /a:Export /tf:testExport.bacpac /scs:"Data Source=<servername>.database.windows.net;Initial Catalog=MyDB;" /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="import-a-database-using-sqlpackage"></a>Importación de una base de datos mediante SqlPackage

Para importar una base de datos de SQL Server mediante la utilidad de línea de comandos [SqlPackage](https://docs.microsoft.com/sql/tools/sqlpackage), consulte [Parámetros y propiedades de la importación](https://docs.microsoft.com/sql/tools/sqlpackage#import-parameters-and-properties). SqlPackage incluye las versiones más recientes de [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) y [SQL Server Data Tools](https://docs.microsoft.com/sql/ssdt/download-sql-server-data-tools-ssdt). También puede descargar la versión más reciente de [SqlPackage](https://docs.microsoft.com/sql/tools/sqlpackage-download).

Para el escalado y rendimiento, se recomienda usar SqlPackage en la mayoría de los entornos de producción en lugar de usar Azure Portal. Consulte cómo [migrar de SQL Server a Azure SQL Database con archivos BACPAC](https://blogs.msdn.microsoft.com/sqlcat/2016/10/20/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/) en el blog de Customer Advisory Team de SQL Server sobre cómo migrar usuarios con archivos `BACPAC`.

El siguiente comando SqlPackage importa la base de datos **AdventureWorks2017** del almacenamiento local a Azure SQL Database. Crea una base de datos llamada **myMigratedDatabase** con un nivel de servicio **Premium** y un objetivo de servicio **P6**. Cambie estos valores según corresponda para su entorno.

```cmd
sqlpackage.exe /a:import /tcs:"Data Source=<serverName>.database.windows.net;Initial Catalog=myMigratedDatabase>;User Id=<userId>;Password=<password>" /sf:AdventureWorks2017.bacpac /p:DatabaseEdition=Premium /p:DatabaseServiceObjective=P6
```

> [!IMPORTANT]
> Para conectarse a Azure SQL Database desde detrás de un firewall corporativo, el firewall debe tener abierto el puerto 1433.

Este ejemplo muestra cómo importar una base de datos con SqlPackage con Autenticación universal de Active Directory.

```cmd
sqlpackage.exe /a:Import /sf:testExport.bacpac /tdn:NewDacFX /tsn:apptestserver.database.windows.net /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="performance-considerations"></a>Consideraciones de rendimiento

Las velocidades de exportación varían debido a muchos factores (por ejemplo, forma de datos), por lo que es imposible predecir qué velocidad se debe esperar. SqlPackage puede tardar un tiempo considerable, especialmente en el caso de las bases de datos de gran tamaño.

Para obtener el mejor rendimiento, puede probar las estrategias siguientes:

1. Asegurarse de que no hay ninguna otra carga de trabajo en ejecución en la base de datos. Crear una copia antes de la exportación puede ser la mejor solución para garantizar que no haya ninguna otra carga de trabajo en ejecución.
2. Aumentar el objetivo de nivel de servicio (SLO) de la base de datos para controlar mejor la carga de trabajo de exportación (principalmente E/S de lectura). Si la base de datos actual es GP_Gen5_4, quizás un nivel de Crítico para la empresa sería útil para la carga de trabajo de lectura.
3. Asegurarse de que hay índices agrupados, especialmente para las tablas grandes.
4. Las máquinas virtuales (VM) deben estar en la misma región que la base de datos para ayudar a evitar restricciones de red.
5. Las máquinas virtuales deben tener discos SSD con el tamaño adecuado para generar artefactos temporales antes de cargarlos en el almacenamiento de blobs.
6. Las máquinas virtuales deben tener una configuración de memoria y de núcleo adecuada para la base de datos específica.

## <a name="store-the-imported-or-exported-bacpac-file"></a>Almacenamiento del archivo .BACPAC importado o exportado

El archivo .BACPAC se puede almacenar en [blobs de Azure](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-overview) o [Azure Files](https://docs.microsoft.com/azure/storage/files/storage-files-introduction).

Para lograr el mejor rendimiento, utilice Azure Files. SqlPackage funciona con el sistema de archivos para que pueda acceder a Azure Files directamente.

Para reducir el costo, use blobs de Azure, que cuestan menos que un recurso compartido de archivos Premium de Azure. Sin embargo, deberá copiar el [archivo .BACPAC](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac) entre el blob y el sistema de archivos local antes de la operación de importación o exportación. Como resultado, el proceso tardará más.

Para cargar o descargar los archivos .BACPAC, consulte [Transferir datos con AzCopy y BLOB Storage](../../storage/common/storage-use-azcopy-blobs.md) y [Transferir datos con AzCopy y almacenamiento de archivos](../../storage/common/storage-use-azcopy-files.md).

En función de su entorno, es posible que necesite [Configurar firewalls y redes virtuales de Azure Storage](../../storage/common/storage-network-security.md).

## <a name="next-steps"></a>Pasos siguientes

- Para información sobre cómo conectarse a una base de datos SQL importada y cómo realizar consultas en ella, lea [Inicio rápido: Azure SQL Database: uso de SQL Server Management Studio para conectarse a los datos y realizar consultas en ellos](connect-query-ssms.md).
- Consulte cómo [migrar de SQL Server a Azure SQL Database con archivos BACPAC](https://techcommunity.microsoft.com/t5/DataCAT/Migrating-from-SQL-Server-to-Azure-SQL-Database-using-Bacpac/ba-p/305407) en el blog de Customer Advisory Team de SQL Server.
- Para información sobre el proceso de migración de bases de datos SQL Server completo, incluidas las recomendaciones relacionadas con el rendimiento, consulte [Migración de una base de datos de SQL Server a Azure SQL Database](migrate-to-database-from-sql-server.md).
- Para obtener información sobre cómo administrar y compartir de forma segura claves de almacenamiento y firmas de acceso compartido, vea la [Guía de seguridad de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-security-guide).
