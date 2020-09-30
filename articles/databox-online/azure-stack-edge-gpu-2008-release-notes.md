---
title: Notas de la versión de la versión preliminar de Azure Stack Edge Pro | Microsoft Docs
description: Se describen las incidencias críticas y las soluciones para la versión de disponibilidad general de Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 09/07/2020
ms.author: alkohli
ms.openlocfilehash: d166b0a4c4b69f03d7dba9d997d7d07fbd81ef41
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90893985"
---
# <a name="azure-stack-edge-pro-with-gpu-preview-release-notes"></a>Notas de la versión de la versión preliminar de Azure Stack Edge Pro con GPU

En las notas de la versión siguientes, se identifican las incidencias críticas pendientes y las resueltas de la versión preliminar de 2008 de los dispositivos Azure Stack Edge Pro con GPU.

Las notas de la versión se actualizan continuamente y se van agregando a medida que se descubren problemas críticos que requieren una solución alternativa. Antes de implementar el dispositivo Azure Stack Edge Pro, revise detenidamente la información que encontrará en las notas de la versión.

Este artículo se aplica a la versión siguiente de software: **Azure Stack Edge Pro 2008**. 

<!--- **2.1.1328.1904**-->

## <a name="whats-new"></a>Novedades

Se han agregado las características siguientes en la versión Azure Stack Edge 2008. En función de la versión de software en versión preliminar específica que se esté ejecutando, es posible que vea un subconjunto de estas características. 

- **Clases de almacenamiento**: en la versión anterior, solo podía aprovisionar el almacenamiento de forma estática mediante recursos compartidos SMB o NFS para las aplicaciones con estado implementadas en el clúster de Kubernetes que se ejecuta en el dispositivo Azure Stack Edge Pro. En esta versión, se han agregado clases de almacenamiento que permiten aprovisionar el almacenamiento de forma dinámica. Para obtener más información, vea [Administración del almacenamiento de Kubernetes en el dispositivo Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-kubernetes-storage.md#dynamicprovisioning). 
- **Panel de Kubernetes con servidor de métricas**: en esta versión, se agrega un panel de Kubernetes con un complemento de servidor de métricas. Puede usar el panel para obtener información general sobre las aplicaciones que se ejecutan en el dispositivo Azure Stack Edge Pro, conocer el estado de los recursos de clúster de Kubernetes y consultar los errores que se han producido en el dispositivo. El servidor de métricas agrega el uso de la CPU y la memoria en los recursos de Kubernetes del dispositivo. Para obtener más información, vea [Uso del panel de Kubernetes para supervisar el dispositivo Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md).
- **Azure Arc para Azure Stack Edge Pro**: a partir de esta versión, puede implementar cargas de trabajo de aplicaciones en el dispositivo Azure Stack Edge Pro a través de Azure Arc. Azure Arc es una herramienta de administración híbrida que permite implementar aplicaciones en los clústeres de Kubernetes. Para obtener más información, vea [Implementación de cargas de trabajo mediante Azure Arc en el dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-arc-kubernetes-cluster.md).  

## <a name="known-issues"></a>Problemas conocidos 

En la tabla siguiente se proporciona un resumen de las incidencias conocidas del dispositivo Azure Stack Edge Pro.

| No. | Característica | Problema | Soluciones alternativas o comentarios |
| --- | --- | --- | --- |
| **1.** |Azure Stack Edge Pro y Azure SQL | La creación de una base de datos SQL requiere acceso de administrador.   |Lleve a cabo los pasos siguientes en lugar de los pasos 1 y 2 de [https://docs.microsoft.com/azure/iot-edge/tutorial-store-data-sql-server#create-the-sql-database](https://docs.microsoft.com/azure/iot-edge/tutorial-store-data-sql-server#create-the-sql-database). <ul><li>En la interfaz de usuario local del dispositivo, habilite la interfaz de proceso. Seleccione **Proceso > Número de puerto > Habilitar para el proceso > Aplicar**.</li><li>Descargue `sqlcmd` en la máquina cliente desde https://docs.microsoft.com/sql/tools/sqlcmd-utility. </li><li>Conéctese a la dirección IP de la interfaz de proceso (el puerto que se habilitó), agregando ",1401" al final de la dirección.</li><li>El comando final tendrá el siguiente aspecto: sqlcmd -S {IP de interfaz},1401-U SA -P "Con1raseña!Segura".</li>Tras este cambio, los pasos 3 y 4 de la documentación actual deben ser idénticos. </li></ul> |
| **2.** |Actualizar| No se admiten los cambios incrementales en los blobs restaurados mediante la funcionalidad de **actualización**. |En los puntos de conexión de blob, las actualizaciones parciales de los blobs después de la funcionalidad de actualización pueden dar lugar a que las actualizaciones no se carguen en la nube. Por ejemplo, supongamos una secuencia de acciones como la siguiente:<ul><li>Crear un blob en la nube. O bien, eliminar del dispositivo un blob cargado previamente.</li><li>Actualizar el blob desde la nube en el dispositivo mediante la funcionalidad de actualización.</li><li>Actualizar solo una parte del blob mediante las API REST de Azure SDK.</li></ul>Estas acciones pueden dar lugar a que las secciones actualizadas del blob no se actualicen en la nube. <br>**Solución alternativa**: use herramientas como robocopy o la copia normal de archivos con el Explorador o la línea de comandos para reemplazar los blobs completos.|
|**3.**|Limitaciones|En una limitación, si no se permiten nuevas escrituras en el dispositivo, las escrituras realizadas por el cliente NFS generarán el error "Permiso denegado".| El error se mostrará como se indica a continuación:<br>`hcsuser@ubuntu-vm:~/nfstest$ mkdir test`<br>mkdir: no se puede crear el directorio 'test': Permiso denegado|
|**4.**|Ingesta de Blob Storage|Si usa AzCopy versión 10 para la ingesta de Blob Storage, ejecute AzCopy con el siguiente argumento: `Azcopy <other arguments> --cap-mbps 2000`.| Si no se proporcionan estos límites para AzCopy, se podría enviar un gran número de solicitudes al dispositivo, lo que provocaría problemas con el servicio.|
|**5.**|Cuentas de almacenamiento en capas|Cuando se usan cuentas de almacenamiento en capas, se aplica lo siguiente:<ul><li> Solo se admiten blobs en bloques. No se admiten los blobs en páginas.</li><li>No hay compatibilidad con la API de copia o instantánea.</li><li> No se admite la ingesta de cargas de trabajo de Hadoop mediante `distcp`, ya que utiliza la operación de copia de forma intensiva.</li></ul>||
|**6.**|Conexión de recurso compartido NFS|Si varios procesos copian en el mismo recurso compartido y no se utiliza el atributo `nolock`, es posible que aparezcan errores durante la copia.|Para copiar archivos en el recurso compartido NFS, se debe usar el atributo `nolock` con el comando mount. Por ejemplo: `C:\Users\aseuser mount -o anon \\10.1.1.211\mnt\vms Z:`.|
|**7.**|Clúster de Kubernetes|Cuando se aplica una actualización en el dispositivo que ejecuta un clúster de Kubernetes, las máquinas virtuales de Kubernetes se reinician. En este caso, solo se restauran automáticamente después de una actualización los pods que se implementan con las réplicas especificadas.  |Si ha creado pods individuales fuera de un controlador de replicación sin especificar un conjunto de réplicas, estos pods no se restaurarán automáticamente después de la actualización del dispositivo. Tendrá que restaurarlos de forma manual.<br>Los pods que se eliminan o finalizan por alguna razón, como un error en un nodo o una actualización del nodo que genere problemas, se sustituyen por conjuntos de réplicas. Por este motivo, es conveniente que utilice un conjunto de réplicas aunque la aplicación solamente necesite un único pod.|
|**8.**|Clúster de Kubernetes|Kubernetes solo se admite en Azure Stack Edge Pro con Helm v3 o una versión posterior. Para más información, consulte las [preguntas frecuentes: eliminación de Tiller](https://v3.helm.sh/docs/faq/).|
|**9.**|Azure Arc y Azure Stack Edge Pro|No se admiten implementaciones de Azure Arc si hay un proxy web configurado en el dispositivo Azure Stack Edge Pro.||
|**10.**|Kubernetes |El puerto 31000 está reservado para el panel de Kubernetes. De forma similar, en la configuración predeterminada, las direcciones IP 172.28.0.1 y 172.28.0.10 se reservan para el servicio Kubernetes y el servicio DNS principal, respectivamente.|No use las direcciones IP reservadas.|
|**11.**|Kubernetes |Actualmente, Kubernetes no permite servicios LoadBalancer con varios protocolos. Por ejemplo, un servicio DNS que tuviese que escuchar en TCP y UDP. |Para solucionar esta limitación de Kubernetes con MetalLB, se pueden crear dos servicios, uno para TCP y otro para UDP, en el mismo selector del pod. Estos servicios usan la misma clave de uso compartido y el mismo valor de spec.loadBalancerIP para compartir la misma dirección IP. También puede compartir las direcciones IP si tiene más servicios que direcciones IP disponibles. <br> Para obtener más información, consulte [Uso compartido de las direcciones IP](https://metallb.universe.tf/usage/#ip-address-sharing).|
|**12.**|Clúster de Kubernetes|Los módulos de Azure IoT Edge existentes en Marketplace no se ejecutarán en el clúster de Kubernetes como plataforma de hospedaje para IoT Edge en un dispositivo Azure Stack Edge.|Estos módulos deberán modificarse antes de implementarlos en el dispositivo Azure Stack Edge. Para más información, consulte el tema sobre modificación de los módulos de Azure IoT Edge de Marketplace para que se ejecuten en un dispositivo Azure Stack Edge.<!-- insert link-->|
|**13.**|Kubernetes |En un dispositivo Azure Stack Edge, no se admiten los montajes de enlace basados en archivos con Azure IoT Edge en Kubernetes.|IoT Edge usa una capa de traducción para traducir las opciones `ContainerCreate` a construcciones de Kubernetes. La creación de asignaciones de `Binds` al directorio hostpath o esta creación y los posteriores montajes de los enlaces basados en archivos no se puede enlazar a rutas de acceso en los contenedores de IoT Edge.|
|**14.**|Kubernetes |Si trae sus propios certificados para IoT Edge y los agrega en el dispositivo Azure Stack Edge, los nuevos certificados no se seleccionarán como parte de la actualización de los gráficos de Helm.|Para solucionar este problema, [conéctese a la interfaz de PowerShell del dispositivo](azure-stack-edge-gpu-connect-powershell-interface.md). Reinicie los pods `iotedged` y `edgehub`.|
|**15.**|Certificados |En determinados casos, el estado del certificado en la interfaz de usuario local puede tardar varios segundos en actualizarse. |Los escenarios siguientes en la interfaz de usuario local pueden verse afectados.<ul><li>La columna **Estado** en la página **Certificados**.</li><li>El icono **Seguridad** en la página de **Introducción**.</li><li>El icono **Configuración** en la página de **Información general**.</li></ul>  |

## <a name="next-steps"></a>Pasos siguientes

- [Preparación de la implementación del dispositivo Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-deploy-prep.md)

