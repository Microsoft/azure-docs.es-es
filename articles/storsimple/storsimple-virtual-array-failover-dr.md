---
title: Conmutación por error y recuperación ante desastres para la matriz virtual de StorSimple
description: Obtenga información sobre la recuperación ante desastres para su instancia de Microsoft Azure StorSimple Virtual Array, incluidos los pasos detallados para conmutar por error a otra matriz virtual.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 3c1f9c62-af57-4634-a0d8-435522d969aa
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 02/27/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 24eb03069689d6dc89d8e237e0e65c71af6c6173
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88184727"
---
# <a name="disaster-recovery-and-device-failover-for-your-storsimple-virtual-array-via-azure-portal"></a>Recuperación ante desastres y conmutación por error de dispositivos para la matriz virtual de StorSimple mediante Azure Portal

## <a name="overview"></a>Información general
En este artículo se describe la recuperación ante desastres para su instancia de Microsoft Azure StorSimple Virtual Array, incluidos los pasos detallados para conmutar por error a otra matriz virtual. Una conmutación por error permite mover los datos de un dispositivo de *origen* en el centro de datos a un dispositivo de *destino*. El dispositivo de destino puede estar en la misma ubicación geográfica o en otra diferente. La conmutación por error del dispositivo es para todo el dispositivo. Durante la conmutación por error, los datos de la nube para el dispositivo de origen cambian la propiedad a la del dispositivo de destino.

Este artículo se aplica únicamente a matrices virtuales de StorSimple. Para conmutar por error un dispositivo de la serie 8000, vaya a [Conmutación por error y recuperación ante desastres para el dispositivo StorSimple](storsimple-device-failover-disaster-recovery.md).

## <a name="what-is-disaster-recovery-and-device-failover"></a>Descripción de la recuperación ante desastres y la conmutación por error de dispositivos

En un escenario de recuperación ante desastres, el dispositivo principal deja de funcionar. En este escenario, puede mover los datos en la nube asociados al dispositivo con error a otro dispositivo. Puede usar el dispositivo principal como *origen* y especificar otro dispositivo como *destino*. Este proceso se conoce como *conmutación por error*. Durante la conmutación por error, todos los volúmenes o los recursos compartidos del dispositivo de origen cambian la propiedad y se transfieren al dispositivo de destino. No se permite ningún filtrado de los datos.

La recuperación ante desastres se modela como una restauración total del dispositivo mediante la organización en capas basada en mapas térmico y el seguimiento. Un mapa térmico se define mediante la asignación de un valor término a los datos según los patrones de lectura y escritura. A continuación, este mapa térmico asigna los niveles de los fragmentos de datos de calor más bajo a la nube primero y mantiene los fragmentos de datos de mayor calor (más usados) en el nivel local. Durante una recuperación ante desastres, StorSimple utiliza el mapa térmico para restaurar y rehidratar los datos desde la nube. El dispositivo captura todos los volúmenes o recursos compartidos de la copia de seguridad más reciente (según se determine internamente) y realiza una restauración a partir de la copia de seguridad. La matriz virtual orquesta todo el proceso de recuperación ante desastres.

> [!IMPORTANT]
> El dispositivo de origen se elimina al final de la conmutación por error del dispositivo y, por tanto, no se admite dicha conmutación.
> 
> 

La recuperación ante desastres se orquesta a través de la característica de recuperación ante desastres del dispositivo y se inicia desde la hoja **Dispositivos**. Esta hoja recoge en formato de tabla todos los dispositivos de StorSimple conectados al servicio StorSimple Device Manager. Para cada dispositivo, puede ver el nombre descriptivo, el estado, la capacidad aprovisionada y máxima, el tipo y el modelo.

## <a name="prerequisites-for-device-failover"></a>Requisitos previos para la conmutación por error de un dispositivo

### <a name="prerequisites"></a>Prerequisites

Para una conmutación por error de un dispositivo, asegúrese de que se cumplen los requisitos previos siguientes:

* El dispositivo de origen debe estar en un estado **Desactivado** .
* El dispositivo de destino debe aparecer como **Listo para configurar** en Azure Portal. Aprovisione una matriz virtual de destino una capacidad igual o mayor. Utilice la interfaz de usuario web local para configurar y registrar correctamente la matriz virtual de destino.
  
  > [!IMPORTANT]
  > No intente configurar el dispositivo virtual registrado a través del servicio. No debe realizarse ninguna configuración del dispositivo a través del servicio.
  > 
  > 
* El dispositivo de destino no puede tener el mismo nombre que el dispositivo de origen.
* El dispositivo de origen y de destino deben ser del mismo tipo. Solo puede conmutar por error una matriz virtual configurada como servidor de archivos a otro servidor de archivos. Lo mismo es cierto para un servidor iSCSI.
* Para la recuperación ante desastres de un servidor de archivos, recomendamos unir el dispositivo de destino al mismo dominio que el origen. Esta configuración garantiza que los permisos del recurso compartido se resuelvan automáticamente. Se admite solo la conmutación por error en un dispositivo de destino del mismo dominio.
* Los dispositivos de destino disponibles para la recuperación ante desastres son dispositivos que tienen una capacidad igual o superior en comparación con el dispositivo de origen. Los dispositivos que están conectados al servicio, pero que no cumplen los criterios de espacio suficiente, no están disponibles como dispositivos de destino.

### <a name="other-considerations"></a>Otras consideraciones

* Si es una conmutación por error planeada:
  
  * Se recomienda que deje sin conexión todos los volúmenes o recursos compartidos en el dispositivo de origen.
  * Se recomienda realizar una copia de seguridad del dispositivo y, a continuación, continuar con la conmutación por error para minimizar la pérdida de datos.
* Si es una conmutación por error no planeada, el dispositivo utiliza la copia de seguridad más reciente para restaurar los datos.

### <a name="device-failover-prechecks"></a>Comprobaciones previas a la conmutación por error de dispositivos

Antes de comenzar la recuperación ante desastres, el dispositivo realiza unas comprobaciones previas. Estas comprobaciones ayudan a garantizar que no se producirán errores cuando comience la recuperación ante desastres. Las comprobaciones previas incluyen:

* Validar la cuenta de almacenamiento.
* Comprobar la conectividad en la nube en Azure.
* Comprobar el espacio disponible en el dispositivo de destino.
* Comprobar si un volumen del dispositivo de origen del servidor iSCSI tiene:
  
  * Nombres de ACR válidos.
  * IQN válido (que no supere 220 caracteres).
  * Contraseñas CHAP válidas (entre 12 y 16 caracteres).

Si se produce un error en cualquiera de las comprobaciones previas anteriores, no puede continuar con la recuperación ante desastres. Resuelva esos problemas y vuelva a intentar la recuperación ante desastres.

Una vez que haya completado correctamente la recuperación ante desastres, la propiedad de los datos de la nube en el dispositivo de origen se transfiere al dispositivo de destino. A continuación, el dispositivo de origen ya no estará disponible en el portal. El acceso a todos los volúmenes o recursos compartidos en el dispositivo de origen se bloquea y se activa el dispositivo de destino.

> [!IMPORTANT]
> Aunque el dispositivo ya no está disponible, la máquina virtual que se ha aprovisionado en el sistema host sigue consumiendo recursos. Una vez que la recuperación ante desastres se haya completado correctamente, podrá eliminar esta máquina virtual del sistema host.
> 
> 

## <a name="fail-over-to-a-virtual-array"></a>Conmutación por error a una matriz virtual

Se recomienda aprovisionar, configurar y registrar otra instancia de StorSimple Virtual Array con el servicio StorSimple Device Manager antes de ejecutar este procedimiento.

> [!IMPORTANT]
> 
> * No puede realizar la conmutación por error de un dispositivo de StorSimple de la serie 8000 a un dispositivo virtual 1200.
> * Puede realizar la conmutación por error de un dispositivo virtual habilitado para FIPS (Estándar federal de procesamiento de información) a otro dispositivo habilitado para FIPS o a un dispositivo no habilitado para FIPS en el portal de Government.


Siga estos pasos para restaurar el dispositivo a un dispositivo virtual de StorSimple de destino.

1. Aprovisione y configure un dispositivo de destino que cumpla los [requisitos previos para la conmutación por error de dispositivos](#prerequisites). Complete la configuración del dispositivo mediante la interfaz de usuario web local y regístrese en el servicio StorSimple Device Manager. Si está creando un servidor de archivos, vaya al paso 1 de [configuración como servidor de archivos](storsimple-virtual-array-deploy3-fs-setup.md#step-1-complete-the-local-web-ui-setup-and-register-your-device). Si está creando un servidor iSCSI, vaya al paso 1 de [configuración como servidor iSCSI](storsimple-virtual-array-deploy3-iscsi-setup.md#step-1-complete-the-local-web-ui-setup-and-register-your-device).

2. Desconecte los volúmenes o recursos compartidos en el host. Para desconectar los volúmenes o recursos compartidos, consulte las instrucciones específicas del sistema operativo para el host. Si aún no está sin conexión, necesita desconectar todos los volúmenes o recursos compartidos en el dispositivo mediante los pasos siguientes:
   
    1. Vaya a la hoja **Dispositivos** y seleccione el dispositivo.
   
    2. Vaya a **Configuración > Administrar > Recursos compartidos** (o **Configuración > Administrar > Volúmenes**). 
   
    3. Seleccione un recurso compartido o un volumen, haga clic con el botón derecho y seleccione **Desconectar**. 
   
    4. Cuando se le pida confirmación, active **Entiendo el impacto de desconectar este recurso compartido.** 
   
    5. Haga clic en **Desconectar**.

3. En el servicio StorSimple Device Manager, vaya a **Administración > Dispositivos**. En la hoja **Dispositivos**, haga clic en el dispositivo de origen.

4. En la hoja **Panel del dispositivo**, haga clic en **Desactivar**.

5. En la hoja **Desactivar**, se le solicitará confirmación. La desactivación de un dispositivo es un proceso *permanente* que no se puede deshacer. También se le recordará que desconecte los recursos compartidos o volúmenes en el host. Escriba el nombre del dispositivo para confirmar la eliminación y haga clic en **Desactivar**.
   
    ![Captura de pantalla de la hoja Desactivar. Se rellena el cuadro del nombre del dispositivo y se resalta el botón Desactivar.](./media/storsimple-virtual-array-failover-dr/failover1.png)
6. La desactivación se inicia. Recibirá una notificación cuando la desactivación se haya completado correctamente.
   
    ![Captura de pantalla de una barra de progreso que indica que el dispositivo se está desactivando.](./media/storsimple-virtual-array-failover-dr/failover2.png)
7. En la página Dispositivos, el estado del dispositivo cambiará a **Desactivado**.
    ![Captura de pantalla de la página Dispositivos. Se muestran las propiedades del dispositivo desactivado, entre las que se incluye el estado, que aparece como Desactivado.](./media/storsimple-virtual-array-failover-dr/failover3.png)
8. En la hoja **Dispositivos**, haga clic en el dispositivo de origen desactivado para conmutación por error. 
9. En la hoja **Panel del dispositivo**, haga clic en **Conmutación por error**. 
10. En la hoja **Dispositivo de conmutación por error**, haga lo siguiente:
    
    1. El campo de dispositivo de origen se rellena automáticamente. Tenga en cuenta el tamaño total de los datos para el dispositivo de origen. El tamaño de los datos debe ser inferior a la capacidad disponible en el dispositivo de destino. Revise los detalles asociados con el dispositivo de origen, como el nombre de dispositivo, la capacidad total y los nombres de los recursos compartidos que se conmutan por error.

    2. En la lista desplegable de dispositivos disponibles, elija un **Dispositivo de destino**. En la lista desplegable solo se muestran los dispositivos con capacidad suficiente.

    3. Seleccione **Comprendo que esta operación conmutará los datos por un error al dispositivo de destino**. 

    4. Haga clic en **Conmutación por error**.
    
        ![Captura de pantalla de la hoja Dispositivo de conmutación por error, con los dispositivos de origen y de destino rellenados, la opción seleccionada y el botón Conmutación por error resaltado.](./media/storsimple-virtual-array-failover-dr/failover4.png)
11. Se inicia un trabajo de conmutación por error y recibirá una notificación. Vaya a **Dispositivos > Trabajos** para supervisar la conmutación por error.
    
     ![Captura de pantalla de una barra de progreso que indica que se está produciendo una conmutación por error.](./media/storsimple-virtual-array-failover-dr/failover5.png)
12. En la hoja **Trabajos**, verá un trabajo de conmutación por error creado para el dispositivo de origen. Este trabajo realiza las comprobaciones previas de recuperación ante desastres.
    
    ![Captura de pantalla que muestra que un trabajo de conmutación por error se ha iniciado correctamente.](./media/storsimple-virtual-array-failover-dr/failover6.png)
    
     Después de que las comprobaciones previas de recuperación ante desastres se realizan correctamente, el trabajo de conmutación por error generará los trabajos de restauración para cada recurso compartido o volumen que exista en el dispositivo de origen.
    
    ![Captura de pantalla que muestra los detalles del trabajo de conmutación por error, como el estado, el dispositivo y la duración.](./media/storsimple-virtual-array-failover-dr/failover7.png)
13. Una vez completada la conmutación por error, vaya a la hoja **Dispositivos**.
    
    1. Haga clic en el dispositivo de StorSimple que se usó como dispositivo de destino para el proceso de conmutación por error.
    2. Vaya a **Configuración > administración > Recursos compartidos** (o **Volúmenes** si se trata de un servidor iSCSI). En la hoja **Recursos compartidos**, puede ver todos los recursos compartidos (o volúmenes) del dispositivo antiguo.
        ![Captura de pantalla de la hoja Dispositivos. El dispositivo de destino se muestra con el estado En línea.](./media/storsimple-virtual-array-failover-dr/failover9.png)
14. Tendrá que [crear un alias DNS](https://support.microsoft.com/kb/168322) para que todas las aplicaciones que están intentando conectarse se puedan redirigir al nuevo dispositivo.

## <a name="errors-during-dr"></a>Errores durante la recuperación ante desastres

**Interrupción de la conectividad de la nube durante la recuperación ante desastres**

Si se interrumpe la conectividad de la nube después de haberse iniciado la recuperación ante desastres y antes de completar la restauración del dispositivo, se producirá un error en la recuperación. Recibirá una notificación de conmutación por error. El dispositivo de destino para la recuperación ante desastres está marcado como *No usable*. No puede utilizar el mismo dispositivo de destino para recuperaciones ante desastres futuras.

**No hay dispositivos de destino compatibles**

Si los dispositivos de destino disponibles no tienen espacio suficiente, verá un error que le indicará que no hay ningún dispositivo de destino compatible.

**Errores de las comprobaciones previas**

Si no se cumple una de las comprobaciones previas, verá errores de las comprobaciones previas.

## <a name="business-continuity-disaster-recovery-bcdr"></a>Recuperación ante desastres y continuidad empresarial (BCDR)

Un escenario de recuperación ante desastres y continuidad empresarial (BCDR) se produce cuando todo el centro de datos de Azure deja de funcionar. Esto puede afectar al servicio StorSimple Device Manager y a los dispositivos de StorSimple asociados.

Si hay dispositivos StorSimple que se registraron justo antes de que ocurriera un desastre, es posible que estos dispositivos StorSimple deban eliminarse. Después del desastre, puede volver a crear y configurar esos dispositivos.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo [administrar la matriz virtual de StorSimple mediante la interfaz de usuario web local](storsimple-ova-web-ui-admin.md).

