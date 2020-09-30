---
title: Notas de la versión de Azure Stack Edge Pro GA | Microsoft Docs
description: Se describen las incidencias más importantes y las soluciones para la versión de disponibilidad general de Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 09/10/2020
ms.author: alkohli
ms.openlocfilehash: 6f1199b29e3bdaa2200aeb7fe4f76c4c9d0d8e19
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932202"
---
# <a name="azure-stack-edge-pro-with-gpu-general-availability-ga-release-notes"></a>Notas de la versión de Azure Stack Edge Pro con disponibilidad general (GA) de GPU

En las notas de la versión siguientes, se identifican las incidencias más importantes pendientes y las resueltas de la versión de disponibilidad general (GA) de los dispositivos Azure Stack Edge Pro con GPU.

Las notas de la versión se actualizan continuamente y se van agregando a medida que se descubren problemas críticos que requieren una solución alternativa. Antes de implementar el dispositivo Azure Stack Edge Pro, revise detenidamente la información que encontrará en las notas de la versión.

Este artículo se aplica al software **Azure Stack Edge Pro 2009**. 

<!--- This software release corresponds to software version number **2.1.1328.1904**-->

## <a name="whats-new"></a>Novedades

Se han agregado las siguientes características a Azure Stack Edge 2009. 

- **Clases de almacenamiento**: en esta versión se han agregado clases de almacenamiento que permiten aprovisionar el almacenamiento de forma dinámica. Para obtener más información, vea [Administración del almacenamiento de Kubernetes en el dispositivo Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-kubernetes-storage.md#dynamicprovisioning). 
- **Panel de Kubernetes con servidor de métricas**: en esta versión, se agrega un panel de Kubernetes con un complemento de servidor de métricas. Puede usar el panel para obtener información general sobre las aplicaciones que se ejecutan en el dispositivo Azure Stack Edge Pro, conocer el estado de los recursos de clúster de Kubernetes y consultar los errores que se han producido en el dispositivo. El servidor de métricas agrega el uso de la CPU y la memoria en los recursos de Kubernetes del dispositivo. Para obtener más información, vea [Uso del panel de Kubernetes para supervisar el dispositivo Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md).
- **Kubernetes en Azure Stack Edge Pro habilitado para Azure Arc**: a partir de esta versión, puede implementar cargas de trabajo de aplicaciones en el dispositivo Azure Stack Edge Pro a través de Kubernetes habilitado para Azure Arc. Azure Arc es una herramienta de administración híbrida que permite implementar aplicaciones en los clústeres de Kubernetes. Para obtener más información, vea [Implementación de cargas de trabajo mediante Azure Arc en el dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-arc-kubernetes-cluster.md).  

## <a name="known-issues"></a>Problemas conocidos 

En la siguiente tabla se proporciona un resumen de las incidencias conocidas del dispositivo Azure Stack Edge Pro.

| No. | Característica | Problema | Soluciones alternativas o comentarios |
| --- | --- | --- | --- |
|**1.**|Características en vista previa |Para esta versión de disponibilidad general, las siguientes características: Azure Resource Manager local, máquinas virtuales, Kubernetes, Kubernetes habilitado para Azure Arc, servicio multiproceso (MPS) para GPU: todos están disponibles en versión preliminar para el dispositivo Azure Stack Edge Pro.  |Estas características estarán disponibles con carácter general en una versión posterior. |
| **2.** |Azure Stack Edge Pro y Azure SQL | La creación de una base de datos SQL requiere acceso de administrador.   |Lleve a cabo los pasos siguientes en lugar de los pasos 1 y 2 de [https://docs.microsoft.com/azure/iot-edge/tutorial-store-data-sql-server#create-the-sql-database](https://docs.microsoft.com/azure/iot-edge/tutorial-store-data-sql-server#create-the-sql-database). <ul><li>En la interfaz de usuario local del dispositivo, habilite la interfaz de proceso. Seleccione **Proceso > Número de puerto > Habilitar para el proceso > Aplicar**.</li><li>Descargue `sqlcmd` en la máquina cliente desde https://docs.microsoft.com/sql/tools/sqlcmd-utility. </li><li>Conéctese a la dirección IP de la interfaz de proceso (el puerto que se habilitó), agregando ",1401" al final de la dirección.</li><li>El comando final tendrá el siguiente aspecto: sqlcmd -S {IP de interfaz},1401-U SA -P "Con1raseña!Segura".</li>Tras este cambio, los pasos 3 y 4 de la documentación actual deben ser idénticos. </li></ul> |
| **3.** |Actualizar| No se admiten los cambios incrementales en los blobs restaurados mediante la funcionalidad de **actualización**. |En los puntos de conexión de blob, las actualizaciones parciales de los blobs después de la funcionalidad de actualización pueden dar lugar a que las actualizaciones no se carguen en la nube. Por ejemplo, supongamos una secuencia de acciones como la siguiente:<ul><li>Crear un blob en la nube. O bien, eliminar del dispositivo un blob cargado previamente.</li><li>Actualizar el blob desde la nube en el dispositivo mediante la funcionalidad de actualización.</li><li>Actualizar solo una parte del blob mediante las API REST de Azure SDK.</li></ul>Estas acciones pueden dar lugar a que las secciones actualizadas del blob no se actualicen en la nube. <br>**Solución alternativa**: use herramientas como robocopy o la copia normal de archivos con el Explorador o la línea de comandos para reemplazar los blobs completos.|
|**4.**|Limitaciones|En una limitación, si no se permiten nuevas escrituras en el dispositivo, las escrituras realizadas por el cliente NFS generarán el error "Permiso denegado".| El error se mostrará como se indica a continuación:<br>`hcsuser@ubuntu-vm:~/nfstest$ mkdir test`<br>mkdir: no se puede crear el directorio 'test': Permiso denegado|
|**5.**|Ingesta de Blob Storage|Si usa AzCopy versión 10 para la ingesta de Blob Storage, ejecute AzCopy con el siguiente argumento: `Azcopy <other arguments> --cap-mbps 2000`.| Si no se proporcionan estos límites para AzCopy, se podría enviar un gran número de solicitudes al dispositivo, lo que provocaría problemas con el servicio.|
|**6.**|Cuentas de almacenamiento en capas|Cuando se usan cuentas de almacenamiento en capas, se aplica lo siguiente:<ul><li> Solo se admiten blobs en bloques. No se admiten los blobs en páginas.</li><li>No hay compatibilidad con la API de copia o instantánea.</li><li> No se admite la ingesta de cargas de trabajo de Hadoop mediante `distcp`, ya que utiliza la operación de copia de forma intensiva.</li></ul>||
|**7.**|Conexión de recurso compartido NFS|Si varios procesos copian en el mismo recurso compartido y no se utiliza el atributo `nolock`, es posible que aparezcan errores durante la copia.|Para copiar archivos en el recurso compartido NFS, se debe usar el atributo `nolock` con el comando mount. Por ejemplo: `C:\Users\aseuser mount -o anon \\10.1.1.211\mnt\vms Z:`.|
|**8.**|Clúster de Kubernetes|Cuando se aplica una actualización en el dispositivo que ejecuta un clúster de Kubernetes, las máquinas virtuales de Kubernetes se reinician. En este caso, solo se restauran automáticamente después de una actualización los pods que se implementan con las réplicas especificadas.  |Si ha creado pods individuales fuera de un controlador de replicación sin especificar un conjunto de réplicas, estos pods no se restaurarán automáticamente después de la actualización del dispositivo. Tendrá que restaurarlos de forma manual.<br>Los pods que se eliminan o finalizan por alguna razón, como un error en un nodo o una actualización del nodo que genere problemas, se sustituyen por conjuntos de réplicas. Por este motivo, es conveniente que utilice un conjunto de réplicas aunque la aplicación solamente necesite un único pod.|
|**9.**|Clúster de Kubernetes|Kubernetes en Azure Stack Edge Pro solo es compatible con Helm v3 o posterior. Para más información, consulte las [preguntas frecuentes: eliminación de Tiller](https://v3.helm.sh/docs/faq/).|
|**10.**|Kubernetes habilitado para Azure Arc |En el caso de la versión GA, se ha actualizado Kubernetes habilitado para Azure Arc de la versión 0.1.18 a 0.2.9. Dado que la actualización de Kubernetes habilitado para Azure Arc no es compatible con el dispositivo Azure Stack Edge, tendrá que volver a implementar Kubernetes habilitado para Azure Arc.|Siga estos pasos:<ol><li>[Aplique las actualizaciones del software del dispositivo y Kubernetes](azure-stack-edge-gpu-install-update.md).</li><li>Conéctese a la [interfaz de PowerShell del dispositivo](azure-stack-edge-gpu-connect-powershell-interface.md).</li><li>Quite el agente de Azure Arc existente. Escriba: `Remove-HcsKubernetesAzureArcAgent`.</li><li>Implemente [Azure Arc en un nuevo recurso](azure-stack-edge-gpu-deploy-arc-kubernetes-cluster.md). No use un recurso de Azure Arc existente.</li></ol>|
|**11.**|Kubernetes habilitado para Azure Arc|No se admiten implementaciones de Azure Arc si hay un proxy web configurado en el dispositivo Azure Stack Edge Pro.||
|**12.**|Kubernetes |El puerto 31000 está reservado para el panel de Kubernetes. De forma similar, en la configuración predeterminada, las direcciones IP 172.28.0.1 y 172.28.0.10 se reservan para el servicio Kubernetes y el servicio DNS principal, respectivamente.|No use las direcciones IP reservadas.|
|**13.**|Kubernetes |Actualmente, Kubernetes no permite servicios LoadBalancer con varios protocolos. Por ejemplo, un servicio DNS que tuviese que escuchar en TCP y UDP. |Para solucionar esta limitación de Kubernetes con MetalLB, se pueden crear dos servicios, uno para TCP y otro para UDP, en el mismo selector del pod. Estos servicios usan la misma clave de uso compartido y el mismo valor de spec.loadBalancerIP para compartir la misma dirección IP. También puede compartir las direcciones IP si tiene más servicios que direcciones IP disponibles. <br> Para obtener más información, consulte [Uso compartido de las direcciones IP](https://metallb.universe.tf/usage/#ip-address-sharing).|
|**14.**|Clúster de Kubernetes|Es posible que los módulos de Azure IoT Edge Marketplace existentes necesiten modificaciones para que se ejecuten en IoT Edge en el dispositivo Azure Stack Edge.|Para más información, consulte el tema sobre modificación de los módulos de Azure IoT Edge de Marketplace para que se ejecuten en un dispositivo Azure Stack Edge.<!-- insert link-->|
|**15.**|Kubernetes |En un dispositivo Azure Stack Edge, no se admiten los montajes de enlace basados en archivos con Azure IoT Edge en Kubernetes.|IoT Edge usa una capa de traducción para traducir las opciones `ContainerCreate` a construcciones de Kubernetes. La creación de asignaciones de `Binds` al directorio hostpath y los posteriores montajes de los enlaces basados en archivos no se puede enlazar a rutas de acceso en los contenedores de IoT Edge. Si es posible, asigne el directorio principal.|
|**16.**|Kubernetes |Si trae sus propios certificados para IoT Edge y los agrega en el dispositivo Azure Stack Edge después de que el proceso se haya configurado en el dispositivo, los nuevos certificados no se seleccionarán.|Para solucionar este problema, debe cargar los certificados antes de configurar el proceso en el dispositivo. Si el proceso ya está configurado, [Conéctese a la interfaz de PowerShell del dispositivo y ejecute los comandos de IoT Edge](azure-stack-edge-gpu-connect-powershell-interface.md#use-iotedge-commands). Reinicie los pods `iotedged` y `edgehub`.|
|**17.**|Certificados |En determinados casos, el estado del certificado en la interfaz de usuario local puede tardar varios segundos en actualizarse. |Los escenarios siguientes en la interfaz de usuario local pueden verse afectados.<ul><li>La columna **Estado** en la página **Certificados**.</li><li>El mosaico **Seguridad** en la página de **Introducción**.</li><li>El mosaico **Configuración** en la página de **Información general**.</li></ul>  |
|**17.**|IoT Edge |Los módulos implementados a través de IoT Edge no pueden usar la red de host. | |
|**18.**|Compute + Kubernetes |Compute/Kubernetes no admite el proxy web NTLM. ||
|**19.**|Compute + proxy web + actualización |Si ha configurado Compute con el proxy web, puede producirse un error en la actualización de Compute. |Se recomienda que deshabilite Compute antes de la actualización. |

<!--|**18.**|Azure Private Edge Zone (Preview) |There is a known issue with Virtual Network Function VM if the VM was created on Azure Stack Edge device running earlier preview builds such as 2006/2007b and then the device was updated to 2009 GA release. The issue is that the VNF information can't be retrieved or any new VNFs can't be created unless the VNF VMs are deleted before the device is updated.  |Before you update Azure Stack Edge device to 2009 release, use the PowerShell command `get-mecvnf` followed by `remove-mecvnf <VNF guid>` to remove all Virtual Network Function VMs one at a time. After the upgrade, you will need to redeploy the same VNFs.|-->




## <a name="next-steps"></a>Pasos siguientes

- [Preparación de la implementación del dispositivo Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-deploy-prep.md)

