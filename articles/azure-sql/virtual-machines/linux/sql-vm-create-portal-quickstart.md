---
title: 'Inicio rápido: Creación de una VM con SQL Server Linux en Azure'
description: Este tutorial muestra cómo crear una máquina virtual Linux con SQL Server 2017 en Azure Portal.
services: virtual-machines-linux
author: MashaMSFT
ms.date: 10/22/2019
tags: azure-service-management
ms.topic: conceptual
ms.service: virtual-machines-sql
ms.workload: iaas-sql-server
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 5ce3ab92be33b378aeb6694efe181ee19da5d3f4
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87284360"
---
# <a name="provision-a-linux-virtual-machine-running-sql-server-in-the-azure-portal"></a>Aprovisionamiento de una máquina virtual Linux con SQL Server en Azure Portal
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!div class="op_single_selector"]
> * [Linux](sql-vm-create-portal-quickstart.md)
> * [Windows](../windows/sql-vm-create-portal-quickstart.md)

En este tutorial de inicio rápido, se usa Azure Portal para crear una máquina virtual Linux con SQL Server 2017 instalado. Aprenderá lo siguiente: 


* [Crear una máquina virtual Linux que ejecute SQL Server desde la galería](#create)
* [Conectarse a la nueva máquina virtual con SSH](#connect)
* [Cambiar la contraseña de SA](#password)
* [Realizar la configuración para conexiones remotas](#remote)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

## <a name="create-a-linux-vm-with-sql-server-installed"></a><a id="create"></a>Creación de una máquina virtual Linux con SQL Server instalado

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. En el panel izquierdo, seleccione **Crear un recurso**.

1. En el panel **Crear un recurso**, seleccione **Proceso**.

1. Seleccione **See all** (Ver todos) junto al encabezado **Featured** (Destacados).

   ![Ver todas las imágenes de máquina virtual](./media/sql-vm-create-portal-quickstart/azure-compute-blade.png)

1. En el cuadro de búsqueda, escriba **SQL Server 2019** y seleccione **Entrar** para iniciar la búsqueda.

1. Para limitar los resultados de la búsqueda, seleccione **Sistema operativo** > **Redhat**.

    ![Filtro de búsqueda para imágenes de máquina virtual con SQL Server 2019](./media/sql-vm-create-portal-quickstart/searchfilter.png)

1. Seleccione una imagen Linux con SQL Server 2019 de los resultados de búsqueda. En este tutorial se usa **SQL Server 2019 en RHEL74**.

   > [!TIP]
   > La edición Developer permite probar o desarrollar con las características de la edición Enterprise, pero sin los costos de licencia de SQL Server. Solo se paga el costo de ejecutar la máquina virtual Linux.

1. Seleccione **Crear**. 


### <a name="set-up-your-linux-vm"></a>Configuración de la máquina virtual Linux

1. En la pestaña **Fundamentos**, seleccione la **suscripción** y el **grupo de recursos**. 

    ![Ventana Fundamentos](./media/sql-vm-create-portal-quickstart/basics.png)

1. En **Nombre de la máquina virtual**, escriba un nombre para la nueva máquina virtual Linux.
1. Luego, escriba o seleccione los valores siguientes:
   * **Región**: Seleccione la región de Azure adecuada para usted.
   * **Opciones de disponibilidad**: Elija la mejor opción de disponibilidad y redundancia para sus aplicaciones y datos.
   * **Cambiar el tamaño**: Seleccione esta opción para elegir un tamaño de máquina y, cuando termine, elija **Seleccionar**. Para más información acerca de los tamaños de máquina virtual, consulte [Tamaños de las máquinas virtuales](../../../virtual-machines/sizes.md).

     ![Selección del tamaño de la máquina virtual](./media/sql-vm-create-portal-quickstart/vmsizes.png)

   > [!TIP]
   > Para el desarrollo y las pruebas funcionales, use un tamaño de máquina virtual de **DS2** o superior. Para las pruebas de rendimiento, use **DS13**, o un tamaño superior.

   * **Tipo de autenticación**: Seleccione **Clave pública SSH**.

     > [!Note]
     > Para la autenticación, puede una clave pública SSH o una contraseña. La opción de SSH es más segura. Para obtener instrucciones acerca de cómo generar una clave SSH, consulte [Creación y uso de un par de claves SSH pública y privada para máquinas virtuales Linux en Azure](../../../virtual-machines/linux/mac-create-ssh-keys.md).

   * **Nombre de usuario**: Escriba el nombre del administrador de la máquina virtual.
   * **Clave pública SSH**: Escriba la clave pública RSA.
   * **Puertos de entrada públicos**: Elija **Permitir los puertos seleccionados** y elija el puerto **SSH (22)** en la lista **Seleccionar puertos de entrada públicos**. En este inicio rápido, este paso es necesario para conectarse y completar la configuración de SQL Server. Si desea conectarse de forma remota a SQL Server, deberá permitir manualmente el tráfico para el puerto predeterminado (1433) que usa Microsoft SQL Server para las conexiones a través de Internet después de crear la máquina virtual.

     ![Puertos de entrada](./media/sql-vm-create-portal-quickstart/port-settings.png)

1. Haga los cambios que quiera en la configuración en las pestañas adicionales siguientes o mantenga la configuración predeterminada.
    * **Discos**
    * **Redes**
    * **Administración**
    * **Configuración de invitado**
    * **Etiquetas**

1. Seleccione **Revisar + crear**.
1. En el panel **Revisar + crear**, seleccione **Crear**.

## <a name="connect-to-the-linux-vm"></a><a id="connect"></a>Conexión a la máquina virtual Linux

Si ya utiliza un shell de BASH, conéctese a la máquina virtual de Azure mediante el comando **ssh**. En el siguiente comando, reemplace el nombre de usuario y la dirección IP de la máquina virtual para conectarse a su máquina virtual Linux.

```bash
ssh azureadmin@40.55.55.555
```

La dirección IP de cualquier máquina virtual se puede encontrar en Azure Portal.

![Dirección IP en Azure Portal](./media/sql-vm-create-portal-quickstart/vmproperties.png)

Si utiliza Windows y no tiene un shell de BASH, instale un cliente de SSH, como PuTTY.

1. [Descargue e instale PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

1. Ejecute PuTTY.

1. En la pantalla de configuración de PuTTY, escriba la dirección IP pública de la máquina virtual.

1. Seleccione **Abrir** y escriba el nombre de usuario y la contraseña en los cuadros.

Para más información acerca de cómo conectarse a máquinas virtuales Linux, consulte [Creación de máquinas virtuales Linux con Azure Portal](../../../virtual-machines/linux/quick-create-portal.md).

> [!NOTE]
> Si ve una alerta de seguridad de PuTTY que indique que la clave de host del servidor no se almacena en la caché del registro, elija entre las opciones siguientes. Si confía en este host, seleccione **Sí** para agregar la clave a la caché de PuTTY y siga conectándose. Si quiere conectarse solo una vez, sin agregar la clave a la caché, seleccione **No**. Si no confía en este host, seleccione **Cancelar** para abandonar la conexión.

## <a name="change-the-sa-password"></a><a id="password"></a> Cambio de la contraseña de SA

La máquina virtual nueva instala SQL Server con una contraseña de SA aleatoria. Restablezca esta contraseña antes de conectarse a SQL Server con el inicio de sesión de SA.

1. Después de conectarse a su máquina virtual Linux, abra un terminal de comandos nuevo.

1. Cambie la contraseña de SA con los siguientes comandos:

   ```bash
   sudo systemctl stop mssql-server
   sudo /opt/mssql/bin/mssql-conf set-sa-password
   ```

   Escriba una nueva contraseña de SA y la confirmación de contraseña cuando se le solicite.

1. Reinicie el servicio SQL Server.

   ```bash
   sudo systemctl start mssql-server
   ```

## <a name="add-the-tools-to-your-path-optional"></a>Adición de las herramientas a la ruta de acceso (opcional)

De manera predeterminada se instalan varios [paquetes](sql-server-on-linux-vm-what-is-iaas-overview.md#packages) de SQL Server, entre los que se incluye el paquete de herramientas de línea de comandos de SQL Server. El paquete de herramientas contiene las herramientas **sqlcmd** y **bcp**. Para mayor comodidad, puede agregar la ruta de acceso de herramientas, `/opt/mssql-tools/bin/`, a su variable de entorno **PATH**.

1. Ejecute los comandos siguientes para modificar la variable **PATH**, tanto para las sesiones de inicio de sesión como para las sesiones interactivas o sin inicio de sesión:

   ```bash
   echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
   echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
   source ~/.bashrc
   ```

## <a name="configure-for-remote-connections"></a><a id="remote"></a>Configuración para conexiones remotas

Si necesita conectarse remotamente a SQL Server en la máquina virtual de Azure, debe configurar una regla de entrada en el grupo de seguridad de red. La regla permite el tráfico en el puerto en el que SQL Server escucha (valor predeterminado de 1433). Los pasos siguientes muestran cómo usar Azure Portal para este paso.

> [!TIP]
> Si ha seleccionado el puerto de entrada **MS SQL (1433)** en la configuración durante el aprovisionamiento, estos cambios se han realizado automáticamente. Puede pasar a la siguiente sección sobre cómo configurar el firewall.

1. En el portal, seleccione **Máquinas virtuales**y, luego, seleccione su máquina virtual de SQL Server.
1. En el panel de navegación de la izquierda, en **Configuración**, seleccione **Redes**.
1. En la ventana Redes, seleccione **Agregar puerto de entrada** en **Reglas de puerto de entrada**.

   ![Reglas de puerto de entrada](./media/sql-vm-create-portal-quickstart/networking.png)

1. En la lista **Servicio**, seleccione **MS SQL**.

    ![Regla de grupo de seguridad de red de MS SQL](./media/sql-vm-create-portal-quickstart/sqlnsgrule.png)

1. Haga clic en **Aceptar** para guardar la regla correspondiente a su máquina virtual.

### <a name="open-the-firewall-on-rhel"></a>Apertura del firewall en RHEL

Este tutorial le ha indicado que cree una máquina virtual de Red Hat Enterprise Linux (RHEL). Si desea conectarse de forma remota a máquinas virtuales de RHEL, también tiene que abrir el puerto 1433 en el firewall de Linux.

1. [Conéctese](#connect) a su máquina virtual de RHEL.

1. En el shell de BASH, ejecute los siguientes comandos:

   ```bash
   sudo firewall-cmd --zone=public --add-port=1433/tcp --permanent
   sudo firewall-cmd --reload
   ```

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene una máquina virtual con SQL Server 2017 en Azure, puede conectarse localmente con **sqlcmd** para ejecutar consultas Transact-SQL.

Si ha configurado la máquina virtual de Azure para conexiones remotas con SQL Server, debería poder conectarse de forma remota. Para ver un ejemplo de cómo conectarse de forma remota a SQL Server en Linux desde Windows, consulte [Usar SQL Server Management Studio (SSMS) de Windows para conectarse a SQL Server en Linux](https://docs.microsoft.com/sql/linux/sql-server-linux-develop-use-ssms). Para conectar con Visual Studio Code, consulte [Usar Visual Studio Code para crear y ejecutar scripts de Transact-SQL para SQL Server](https://docs.microsoft.com/sql/linux/sql-server-linux-develop-use-vscode)

Para información general sobre SQL Server en Linux, consulte [Información general de SQL Server de 2017 en Linux](https://docs.microsoft.com/sql/linux/sql-server-linux-overview). Para más información acerca de cómo usar máquinas virtuales Linux con SQL Server 2017, consulte [Overview of SQL Server 2017 virtual machines on Azure](sql-server-on-linux-vm-what-is-iaas-overview.md) (Introducción a las máquinas virtuales con SQL Server 2017 en Azure).
