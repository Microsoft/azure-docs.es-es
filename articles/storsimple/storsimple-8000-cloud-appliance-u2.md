---
title: StorSimple Cloud Appliance Update 3 | Microsoft Docs
description: Aprenda a crear, implementar y administrar una instancia de StorSimple Cloud Appliance en una red virtual de Microsoft Azure. (Se aplica a StorSimple Update 3 y versiones posteriores).
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/08/2017
ms.author: alkohli
ms.openlocfilehash: 219e2b77a0f6f30307c43f006fcdd3828d3c8fbf
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87021382"
---
# <a name="deploy-and-manage-a-storsimple-cloud-appliance-in-azure-update-3-and-later"></a>Implementación y administración de una instancia de StorSimple Cloud Appliance en Azure (Update 3 y versiones posteriores)

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]

## <a name="overview"></a>Información general

El dispositivo de nube de StorSimple de la serie 8000 es una funcionalidad adicional que se incluye con la solución de Microsoft Azure StorSimple. StorSimple Cloud Appliance se ejecuta en una máquina virtual de una red virtual de Microsoft Azure y puede usarlo para realizar una copia de seguridad de los datos y clonarlos desde sus hosts.

En este artículo se describe el proceso paso a paso de implementación y administración de una instancia de StorSimple Cloud Appliance en Azure. Después de leer este artículo, habrá aprendido lo siguiente:

* Las diferencias entre el dispositivo de nube y el dispositivo físico.
* La creación y configuración del dispositivo de nube.
* La conexión al dispositivo de nube.
* El uso del dispositivo de nube.

Este tutorial se aplica a todas las instancias de StorSimple Cloud Appliance que ejecutan Update 3 y posterior.

#### <a name="cloud-appliance-model-comparison"></a>Comparación de los modelos de dispositivo de nube

StorSimple Cloud Appliance está disponible en dos modelos, el 8010 estándar (antes conocido como 1100) y el premium 8020 (introducido en Update 2). En la tabla siguiente se presenta una comparación de los dos modelos.

| Modelo de dispositivo | 8010<sup>1</sup> | 8020 |
| --- | --- | --- |
| **Capacidad máxima** |30 TB |64 TB |
| **MV de Azure** |Standard_A3 (4 núcleos, 7 GB de memoria)| Standard_DS3 (4 núcleos, 14 GB de memoria)|
| **Disponibilidad en regiones** |Todas las regiones de Azure |Regiones de Azure que admiten Premium Storage y máquinas virtuales de Azure DS3<br></br>Use [esta lista](https://azure.microsoft.com/regions/services/) para ver si las dos opciones **Virtual Machines > Serie DS** y **Storage > Almacenamiento en disco** están disponibles en su región. |
| **Tipo de almacenamiento** |Azure Standard Storage para discos locales<br></br> Infórmese de cómo [crear una cuenta de Standard Storage](../storage/common/storage-create-storage-account.md) |Usa Azure Premium Storage para discos locales<sup>2</sup> <br></br> |
| **Guía de la carga de trabajo** |Recuperación a nivel de elemento de archivos de copias de seguridad |Escenarios de desarrollo y pruebas de la nube <br></br>Baja latencia y cargas de trabajo de rendimiento más elevado<br></br>Dispositivo secundario para recuperación ante desastres |

<sup>1</sup> *Anteriormente conocido como 1100*.

<sup>2</sup> *Tanto 8010 como 8020 usan Azure Standard Storage para el nivel de nube. La diferencia solo existe en el nivel local del dispositivo*.

## <a name="how-the-cloud-appliance-differs-from-the-physical-device"></a>Diferencias entre el dispositivo de nube y el dispositivo físico

StorSimple Cloud Appliance es una versión solo de software de StorSimple que se ejecuta en un único nodo en una máquina virtual de Microsoft Azure. El dispositivo en la nube es compatible con escenarios de recuperación ante desastres en el que el dispositivo físico no está disponible. El dispositivo en la nube es adecuado para su uso en la recuperación de elementos de copias de seguridad, en la recuperación ante desastres local y en escenarios de desarrollo y pruebas en la nube.

#### <a name="differences-from-the-physical-device"></a>Diferencias del dispositivo físico

En la tabla siguiente se muestran algunas diferencias clave entre StorSimple Cloud Appliance y el dispositivo físico StorSimple:

|  | Dispositivo físico | Dispositivo de nube |
| --- | --- | --- |
| **Ubicación** |Se encuentra en el centro de datos. |Se ejecuta en Azure. |
| **Interfaces de red** |Tiene seis interfaces de red: DATA 0 a DATA 5. |Solo tiene una interfaz de red: DATA 0. |
| **Registro** |Se registra durante el paso de configuración inicial. |Registrado a través de una tarea independiente. |
| **Clave de cifrado de datos de servicio** |Se regenera en el dispositivo físico y luego actualiza el dispositivo de nube con la nueva clave. |No se puede regenerar desde el dispositivo de nube. |
| **Tipos de volúmenes admitidos** |Admite volúmenes conectados localmente y por niveles. |Admite solo volúmenes por niveles. |

## <a name="prerequisites-for-the-cloud-appliance"></a>Requisitos previos del dispositivo de nube

En las siguientes secciones se explican los requisitos previos de configuración de StorSimple Cloud Appliance. Antes de implementar un dispositivo de nube, revise las consideraciones de seguridad relacionadas sobre su uso.

[!INCLUDE [StorSimple Cloud Appliance security](../../includes/storsimple-8000-cloud-appliance-security.md)]

#### <a name="azure-requirements"></a>Requisitos de Azure

Antes de aprovisionar el dispositivo de nube, debe realizar los siguientes preparativos en el entorno de Azure:

* Asegúrese de que tiene un dispositivo físico StorSimple 8000 series (modelo 8100 o 8600) implementado y en ejecución en el centro de datos. Registre este dispositivo con el mismo servicio de StorSimple Device Manager para el que va a crear un StorSimple Cloud Appliance.
* Para el dispositivo de nube, [configure una red virtual en Azure](../virtual-network/manage-virtual-network.md#create-a-virtual-network). Si usa Premium Storage, tiene que crear una red virtual en una región de Azure que admita dicho almacenamiento. Las regiones Premium Storage son las que corresponden a la fila de almacenamiento en disco de la [lista de servicios de Azure por región](https://azure.microsoft.com/regions/services/).
* Se recomienda usar el servidor DNS predeterminado proporcionado por Azure en lugar de especificar su propio nombre de servidor DNS. Si el nombre del servidor DNS no es válido o si el servidor DNS no es capaz de resolver direcciones IP correctamente, la creación del dispositivo de nube dará error.
* Punto a sitio y sitio a sitio son opcionales, pero no obligatorios. Si lo desea, puede configurar estas opciones para escenarios más avanzados.
* Puede crear [Azure Virtual Machines](../virtual-machines/windows/quick-create-portal.md) (servidores host) en la red virtual que pueden usar los volúmenes expuestos por el dispositivo de nube. Estos servidores deben cumplir los siguientes requisitos:

  * Estar en máquinas virtuales de Windows o Linux con el software iSCSI Initiator instalado.
  * Estar ejecutándose en la misma red virtual que el dispositivo de nube.
  * Ser capaces de conectarse al destino iSCSI del dispositivo de nube a través de su dirección IP interna.
  * Asegúrese de que haya habilitado la compatibilidad para el tráfico iSCSI y en la nube en la misma red virtual.

#### <a name="storsimple-requirements"></a>Requisitos de StorSimple

Realice las siguientes actualizaciones de su servicio StorSimple Device Manager antes de crear un dispositivo de nube:

* Agregue [registros de control de acceso](storsimple-8000-manage-acrs.md) para las máquinas virtuales que vayan a ser servidores host para el dispositivo de nube.
* Use una [cuenta de almacenamiento](storsimple-8000-manage-storage-accounts.md#add-a-storage-account) de la misma región que el dispositivo de nube. Las cuentas de almacenamiento en regiones diferentes pueden causar un bajo rendimiento. Puede usar una cuenta de Standard Storage o Premium Storage con el dispositivo de nube. Más información sobre cómo crear una [cuenta de almacenamiento estándar](../storage/common/storage-create-storage-account.md).
* Use una cuenta de almacenamiento diferente para la creación del dispositivo de nube a partir de la que ha usado para sus datos. Utilizar la misma cuenta de almacenamiento puede causar un bajo rendimiento.

Asegúrese de que tiene la siguiente información antes de empezar:

* Su cuenta del portal de Azure con credenciales de acceso.
* Una copia de la clave de cifrado de datos del servicio del dispositivo físico registrado en el servicio StorSimple Device Manager.

## <a name="create-and-configure-the-cloud-appliance"></a>Creación y configuración del dispositivo de nube

Antes de realizar estos procedimientos, asegúrese de que cumple los [requisitos previos del dispositivo de nube](#prerequisites-for-the-cloud-appliance).

Para crear una instancia de StorSimple Cloud Appliance, siga estos pasos:

### <a name="step-1-create-a-cloud-appliance"></a>Paso 1: Creación de un dispositivo en la nube

Para crear la instancia de StorSimple Cloud Appliance, realice los siguientes pasos:

[!INCLUDE [Create a cloud appliance](../../includes/storsimple-8000-create-cloud-appliance-u2.md)]

Si se produce un error en este paso durante la creación del dispositivo de nube, quizás no tenga conexión a Internet. Para más información, vaya a la [solución de problemas de conexión a Internet](#troubleshoot-internet-connectivity-errors) al crear un dispositivo de nube.

### <a name="step-2-configure-and-register-the-cloud-appliance"></a>Paso 2: Configuración y registro del dispositivo en la nube

Antes de comenzar este procedimiento, asegúrese de que tiene una copia de la clave de cifrado de datos del servicio. Esta clave se crea cuando se registra el primer dispositivo físico de StorSimple con el servicio StorSimple Device Manager. Se le pidió que la guardara en una ubicación segura. Si no tiene una copia de la clave de cifrado de datos del servicio, debe ponerse en contacto con Microsoft Support para obtener ayuda.

Realice los pasos siguientes para configurar y registrar su instancia de StorSimple Cloud Appliance.

[!INCLUDE [Configure and register a cloud appliance](../../includes/storsimple-8000-configure-register-cloud-appliance.md)]

### <a name="step-3-optional-modify-the-device-configuration-settings"></a>Paso 3: (Opcional) Modificación de la configuración del dispositivo

En la siguiente sección se describen las opciones de configuración de dispositivo necesarias para StorSimple Cloud Appliance si desea usar CHAP, StorSimple Snapshot Manager o cambiar la contraseña del administrador de dispositivos.

#### <a name="configure-the-chap-initiator"></a>Configuración del iniciador CHAP

Este parámetro contiene las credenciales que espera el dispositivo de nube (destino) de los iniciadores (servidores) que intentan obtener acceso a los volúmenes. Los iniciadores proporcionarán un nombre de usuario y una contraseña CHAP para identificarse en el dispositivo durante esta autenticación. Para obtener los pasos detallados necesarios, vaya a [Configurar CHAP para el dispositivo StorSimple](storsimple-8000-configure-chap.md#unidirectional-or-one-way-authentication).

#### <a name="configure-the-chap-target"></a>Configuración del destino CHAP

Este parámetro contiene las credenciales que el dispositivo de nube usa cuando un iniciador habilitado para CHAP solicita autenticación mutua o bidireccional. El dispositivo de nube usa un nombre de usuario de CHAP inverso y una contraseña de CHAP inversa para identificarse en el iniciador durante este proceso de autenticación.

> [!NOTE]
> Los valores de configuración de destino de CHAP son valores globales. Cuando se aplica esta configuración, todos los volúmenes conectados al dispositivo de nube usan la autenticación CHAP.

Para obtener los pasos detallados necesarios, vaya a [Configurar CHAP para el dispositivo StorSimple](storsimple-8000-configure-chap.md#bidirectional-or-mutual-authentication).

#### <a name="configure-the-storsimple-snapshot-manager-password"></a>Configuración de la contraseña de StorSimple Snapshot Manager

El software StorSimple Snapshot Manager reside en el host de Windows y permite a los administradores administrar copias de seguridad del dispositivo StorSimple en forma de instantáneas locales y en la nube.

> [!NOTE]
> Para el dispositivo de nube, el host de Windows es una máquina virtual de Azure.

Al configurar un dispositivo en StorSimple Snapshot Manager, se le pide que proporcione la dirección IP del dispositivo StorSimple y la contraseña para autenticar el dispositivo de almacenamiento. Para obtener los pasos detallados necesarios, vaya a [Cambio de la contraseña de StorSimple Snapshot Manager](storsimple-8000-change-passwords.md#set-the-storsimple-snapshot-manager-password)

#### <a name="change-the-device-administrator-password"></a>Cambio de la contraseña del administrador de dispositivos

Cuando se usa la interfaz de Windows PowerShell para acceder al dispositivo de nube, se le pide que escriba una contraseña de administrador de dispositivos. Por la seguridad de los datos, debe cambiar esta contraseña para poder usar el dispositivo de nube. Para obtener los pasos detallados necesarios, vaya a [Cambio de la contraseña del administrador de dispositivos](../storsimple/storsimple-8000-change-passwords.md#change-the-device-administrator-password).

## <a name="connect-remotely-to-the-cloud-appliance"></a>Conexión remota al dispositivo de nube

El acceso remoto al dispositivo de nube mediante la interfaz de Windows PowerShell no está habilitado de forma predeterminada. Primero debe habilitar la administración remota en el dispositivo de nube y luego en el cliente usado para acceder a él.

El siguiente procedimiento de dos pasos describe cómo conectarse de forma remota a su dispositivo de nube.

### <a name="step-1-configure-remote-management"></a>Paso 1: Configuración de la administración remota

Para configurar la administración remota para su instancia de StorSimple Cloud Appliance, realice los pasos siguientes:

[!INCLUDE [Configure remote management via HTTP for cloud appliance](../../includes/storsimple-8000-configure-remote-management-http-device.md)]

### <a name="step-2-remotely-access-the-cloud-appliance"></a>Paso 2: Acceso remoto al dispositivo en la nube

Después de habilitar la administración remota en el dispositivo de nube, use la comunicación remota de Windows PowerShell para conectarse al dispositivo desde otra máquina virtual dentro de la misma red virtual. Por ejemplo, puede conectarse desde la máquina virtual host que configuró y usó para la conexión iSCSI. En la mayoría de las implementaciones, abrirá un punto de conexión público para acceder a su máquina virtual host que se puede usar para acceder al dispositivo de nube.

> [!WARNING]
> **Para mayor seguridad, se recomienda utilizar HTTPS al conectarse a los extremos y, a continuación, eliminar los extremos después de haber completado la sesión remota de PowerShell.**

Para configurar la comunicación remota de su dispositivo de nube, debe seguir los procedimientos que se describen en [Conexión de forma remota al dispositivo StorSimple serie 8000](storsimple-8000-remote-connect.md).

## <a name="connect-directly-to-the-cloud-appliance"></a>Conexión directa al dispositivo de nube

También puede conectarse directamente al dispositivo de nube. Para conectarse directamente al dispositivo de nube desde otro equipo fuera de la red virtual o del entorno de Microsoft Azure, debe crear puntos de conexión adicionales.

Realice los pasos siguientes para crear un punto de conexión público en el dispositivo de nube.

[!INCLUDE [Create public endpoints on a cloud appliance](../../includes/storsimple-8000-create-public-endpoints-cloud-appliance.md)]

Se recomienda conectarse desde otra máquina virtual dentro de la misma red virtual, ya que esta práctica minimiza el número de puntos de conexión públicos de la red virtual. En este caso, conéctese a la máquina virtual mediante una sesión de escritorio remoto y, a continuación, configure esa máquina virtual para su uso como haría con cualquier otro cliente de Windows en una red local. No es necesario anexar el número de puerto público porque ya se conoce el puerto.

## <a name="get-private-ip-for-the-cloud-appliance"></a>Obtención de una IP privada para el dispositivo en la nube

Para que el dispositivo en la nube se conecte conectarse al servidor de host de la misma red virtual, se necesita la dirección IP interna o la dirección IP privada del dispositivo en la nube. Realice los pasos siguientes para obtener la dirección IP privada de la aplicación en la nube

1. Vaya a la máquina virtual subyacente del dispositivo en la nube. La máquina virtual tiene el mismo nombre que el dispositivo en la nube. Vaya a **Todos los recursos**, especifique el nombre del dispositivo en la nube y la suscripción, y seleccione el tipo como máquinas virtuales. En la lista de máquinas virtuales que se presentan, seleccione la máquina virtual correspondiente al dispositivo en la nube y haga clic en ella.

     ![Seleccione la máquina virtual del dispositivo en la nube](./media/storsimple-8000-cloud-appliance-u2/sca-vm.png)

2. Vaya a **Configuración > Redes**. En el panel derecho, verá la dirección IP privada del dispositivo en la nube. Anótela.

    ![Obtenga la dirección IP privada del dispositivo en la nube](./media/storsimple-8000-cloud-appliance-u2/sca-private-ip-vm-networking.png)

## <a name="work-with-the-storsimple-cloud-appliance"></a>Uso de StorSimple Cloud Appliance

Ahora que ha creado y configurado StorSimple Cloud Appliance, está listo para comenzar a trabajar con este servicio. Puede trabajar con contenedores de volúmenes, volúmenes y directivas de copia de seguridad en un dispositivo de nube tal y como haría en un dispositivo físico de StorSimple. La única diferencia es que debe asegurarse de seleccionar el dispositivo de nube de la lista de dispositivos. Consulte el artículo sobre [cómo usar el servicio StorSimple Device Manager para administrar un dispositivo de nube](storsimple-8000-manager-service-administration.md). En él encontrará procedimientos paso a paso de las diversas tareas de administración del dispositivo de nube.

En las siguientes secciones se describen algunas de las diferencias encontradas al trabajar con el dispositivo de nube.

### <a name="maintain-a-storsimple-cloud-appliance"></a>Mantenimiento de StorSimple Cloud Appliance

Dado que es un dispositivo solo de software, el mantenimiento del dispositivo de nube es mínimo en comparación con el mantenimiento del dispositivo físico.

No se puede actualizar un dispositivo en la nube. Use la versión más reciente del software para crear un nuevo dispositivo de nube.


### <a name="storage-accounts-for-a-cloud-appliance"></a>Cuentas de almacenamiento para un dispositivo de nube

Las cuentas de almacenamiento que se usan las crean el servicio StorSimple Device Manager, el dispositivo de nube y el dispositivo físico. Al crear las cuentas de almacenamiento, se recomienda usar un identificador regional en el nombre descriptivo. Esto ayuda a garantizar que la región es coherente en todos los componentes del sistema. En el caso de un dispositivo de nube, es importante que todos los componentes estén en la misma región para evitar problemas de rendimiento.

Para conocer el procedimiento paso a paso, vaya a [Agregar una cuenta de almacenamiento](storsimple-8000-manage-storage-accounts.md#add-a-storage-account).

### <a name="deactivate-a-storsimple-cloud-appliance"></a>Desactivación de StorSimple Cloud Appliance

Al desactivar un dispositivo de nube, la acción elimina la máquina virtual y los recursos creados cuando se aprovisionó. Después de desactivar el dispositivo de nube, no se puede restaurar a su estado anterior. Antes de desactivarlo, asegúrese de detener o eliminar clientes y hosts que dependen de él.

La desactivación de un dispositivo de nube da lugar a las siguientes acciones:

* Se quita el dispositivo de nube.
* Se quitan los discos de datos y del sistema operativo creados para el dispositivo de nube.
* Se mantienen el servicio hospedado y la red virtual creados durante el aprovisionamiento. Si no los utiliza, debe eliminarlos manualmente.
* Se conservan las instantáneas de nube creadas para el dispositivo de nube.

Para obtener un procedimiento paso a paso, consulte [Desactivación y eliminación de un dispositivo de StorSimple](storsimple-8000-deactivate-and-delete-device.md).

Tan pronto como se muestre el dispositivo en la nube como desactivado en la hoja del servicio StorSimple Device Manager, puede eliminarlo de la lista de la hoja **Dispositivos**.

### <a name="start-stop-and-restart-a-cloud-appliance"></a>Inicio, detención y reinicio de un dispositivo de nube
A diferencia del dispositivo físico de StorSimple, en un dispositivo de nube de StorSimple no hay ningún botón de encendido o apagado que presionar. Sin embargo, puede haber ocasiones en que deba detener y reiniciar el dispositivo de nube.

La manera más fácil de iniciar, detener y reiniciar un dispositivo en la nube es a través de la hoja del servicio Virtual Machines. Vaya el servicio Virtual Machines. En la lista de máquinas virtuales, identifique la máquina virtual correspondiente a su dispositivo de nube (mismo nombre) y haga clic en su nombre. Si examina la hoja de la máquina virtual, verá que el estado del dispositivo es **En ejecución** porque de forma predeterminada se inicia una vez que se crea. Puede iniciar, detener y reiniciar una máquina virtual en cualquier momento.

[!INCLUDE [Stop and restart cloud appliance](../../includes/storsimple-8000-stop-restart-cloud-appliance.md)]

### <a name="reset-to-factory-defaults"></a>Restablecimiento de los valores predeterminados de fábrica
Si decide que desea empezar de nuevo con su dispositivo de nube, desactívelo y elimínelo y, a continuación, cree uno nuevo.

## <a name="fail-over-to-the-cloud-appliance"></a>Conmutación por error al dispositivo de nube
La recuperación ante desastres (DR) es uno de los escenarios clave para los que se diseñó StorSimple Cloud Appliance. En este escenario, es posible que el dispositivo físico de StorSimple o todo el centro de datos no estén disponible. Afortunadamente, puede usar un dispositivo de nube para las operaciones de restauración en una ubicación alternativa. Durante la recuperación ante desastres, los contenedores de volúmenes del dispositivo de origen cambian la propiedad y se transfieren al dispositivo de nube.

Los requisitos previos de la recuperación ante desastres son:

* El dispositivo de nube se ha creado y configurado.
* Todos los volúmenes del contenedor de volúmenes están sin conexión.
* El contenedor de volúmenes que conmuta por error tiene asociada una instantánea de nube.

> [!NOTE]
> * Al usar un dispositivo de nube como dispositivo secundario para la recuperación ante desastres, tenga en cuenta que el 8010 tiene 30 TB de almacenamiento estándar y 8020 tiene 64 TB de Premium Storage. El dispositivo de nube de capacidad más alta, 8020, puede ser más adecuado para un escenario de conmutación por error.

Para obtener un procedimiento paso a paso, consulte [Conmutación por error al dispositivo de nube](storsimple-8000-device-failover-cloud-appliance.md).

## <a name="delete-the-cloud-appliance"></a>Eliminación del dispositivo de nube
Si anteriormente configuró y usó StorSimple Cloud Appliance pero ahora desea dejar de acumular cargos por su uso, debe detener el dispositivo de nube. Al detener el dispositivo de nube se desasigna la máquina virtual. De esta forma dejan de acumularse cargos en su suscripción. Sin embargo, los gastos de almacenamiento por los discos de datos y del sistema operativo continuarán.

Para detener todos los cargos, debe eliminar el dispositivo de nube. Para eliminar las copias de seguridad creadas por el dispositivo de nube, puede desactivar o eliminar el dispositivo. Para más información, consulte [Desactivación y eliminación de un dispositivo de StorSimple](storsimple-8000-deactivate-and-delete-device.md).

[!INCLUDE [Delete a cloud appliance](../../includes/storsimple-8000-delete-cloud-appliance.md)]

## <a name="troubleshoot-internet-connectivity-errors"></a>Solucionar errores de conexión a Internet
Durante la creación del dispositivo de nube, si no hay conexión a Internet, el paso de creación dará error. Para solucionar problemas debidos a la conexión a Internet, realice los pasos siguientes en Azure Portal:

1. [Cree una máquina virtual Windows en Azure Portal](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal). Esta máquina virtual debe usar la misma cuenta de almacenamiento, red virtual y subred que usa el dispositivo de nube. Si ya tiene un host de Windows Server en Azure con la misma cuenta de almacenamiento, red virtual y subred, también puede usarlo para solucionar los problemas de conexión a Internet.
2. Inicie sesión de forma remota en la máquina virtual creada en el paso anterior.
3. Abra una ventana de comandos en la máquina virtual (Win + R y, después, escriba `cmd`).
4. En el símbolo del sistema, ejecute el siguiente comando.

    `nslookup windows.net`
5. Si `nslookup` produce un error, quiere decir que un error de conexión a Internet impide que el dispositivo de nube se registre en el servicio StorSimple Device Manager.
6. Realice los cambios necesarios en la red virtual para garantizar que el dispositivo de nube pueda acceder a sitios de Azure como _windows.net_.

## <a name="next-steps"></a>Pasos siguientes
* Aprenda a [usar el servicio StorSimple Device Manager para administrar un dispositivo de nube](storsimple-8000-manager-service-administration.md).
* Sepa cómo [restaurar un volumen de StorSimple de un conjunto de copias de seguridad](storsimple-8000-restore-from-backup-set-u2.md).
