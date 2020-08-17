---
title: Configuración de la recuperación ante desastres para una aplicación web de IIS mediante Azure Site Recovery
description: Aprenda cómo replicar las máquinas virtuales de una granja de servidores web de IIS con Azure Site Recovery.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 11/27/2018
ms.author: mayg
ms.openlocfilehash: 7a4408b54b663b2cd8abc22772ac1b799ea50de0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083776"
---
# <a name="set-up-disaster-recovery-for-a-multi-tier-iis-based-web-application"></a>Configuración de la recuperación ante desastres para una aplicación web basada en IIS de niveles múltiples

El software de las aplicaciones es el motor que mantiene la productividad del negocio. En una organización, puede haber diversas aplicaciones web con diferentes propósitos. Algunas aplicaciones, como las que se usan para el procesamiento de nóminas, aplicaciones financieras y sitios web orientado al cliente, pueden resultar fundamentales para una organización. Para evitar la pérdida de productividad, es importante que la organización tenga estas aplicaciones continuamente en funcionamiento. Más importante aún, tener estas aplicaciones disponibles sistemáticamente puede ayudar a impedir que se perjudique la marca o la imagen de la organización.

Las aplicaciones web que resultan esenciales suelen configurarse como aplicaciones de capas múltiples: web, base de datos y aplicación. Además de abarcar diferentes capas, las aplicaciones también pueden usar diferentes servidores en cada nivel para equilibrar la carga del tráfico. Por otra parte, las asignaciones entre las diferentes capas y en el servidor web podrían basarse en direcciones IP estáticas. Si se produce una conmutación por error, algunas de estas asignaciones deben actualizarse, especialmente, si hay varios sitios web configurados en el servidor web. Si las aplicaciones web usan TLS, debe actualizar los enlaces de certificado.

Con los métodos de recuperación tradicionales que no se basan en la replicación, es necesario realizar la copia de seguridad de varios archivos de configuración, valores del Registro, enlaces, componentes personalizados (COM o .NET), contenido y certificados. Los archivos se recuperan mediante un conjunto de pasos manuales. Los métodos tradicionales de copia de seguridad y recuperación manual de los archivos son complicados, propensos a errores y no se pueden escalar. Por ejemplo, es fácil olvidarse de realizar una copia de seguridad de los certificados. Después de la conmutación por error, no le queda más remedio que comprar nuevos certificados para el servidor.

Una buena solución de recuperación ante desastres admite el modelado de planes para arquitecturas de aplicación complejas. También debe poder agregar pasos personalizados al plan de recuperación para controlar las asignaciones de aplicación entre capas. Si se produce un desastre, las asignaciones de aplicación proporcionan una solución infalible con un solo clic que conducen a un RTO inferior.

En este artículo se describe cómo proteger una aplicación web basada en Internet Information Services (IIS) mediante [Azure Site Recovery](site-recovery-overview.md). También se describen los procedimientos recomendados para replicar en Azure una aplicación web basada en IIS de tres capas, cómo llevar a cabo un simulacro de recuperación ante desastres y cómo conmutar por error la aplicación a Azure.

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, asegúrese de que sabe cómo realizar las tareas siguientes:

* [Replicar una máquina virtual en Azure](vmware-azure-tutorial.md)
* [Diseñar una red de recuperación](./concepts-on-premises-to-azure-networking.md)
* [Realizar una conmutación por error de prueba en Azure](site-recovery-test-failover-to-azure.md)
* [Realizar una conmutación por error en Azure](site-recovery-failover.md)
* [Replicar un controlador de dominio](site-recovery-active-directory.md)
* [Replicación de SQL Server](site-recovery-sql.md)

## <a name="deployment-patterns"></a>Modelos de implementación
Por lo general, las aplicaciones web basadas en IIS se ajustan a uno de los siguientes patrones de implementación:

**Patrón de implementación 1**

Una granja de servidores web basados en IIS con Enrutamiento de solicitud de aplicaciones (ARR), un servidor IIS y SQL Server.

![Diagrama de una granja de servidores web basados en IIS que consta de tres capas](./media/site-recovery-iis/deployment-pattern1.png)

**Patrón de implementación 2**

Una granja de servidores web basados en IIS con ARR, un servidor IIS, un servidor de aplicaciones y SQL Server.

![Diagrama de una granja de servidores web basados en IIS que tiene cuatro capas](./media/site-recovery-iis/deployment-pattern2.png)

## <a name="site-recovery-support"></a>Compatibilidad de Site Recovery

En los ejemplos de este artículo, usamos máquinas virtuales de VMware con IIS 7.5 en Windows Server 2012 R2 Enterprise. Dado que la replicación de Site Recovery no es específica de la aplicación, se espera que las recomendaciones de este artículo se apliquen en los escenarios indicados en la siguiente tabla y para distintas versiones de IIS.

### <a name="source-and-target"></a>Origen y destino

Escenario | En un sitio secundario | En Azure
--- | --- | ---
Hyper-V | Sí | Sí
VMware | Sí | Sí
Servidor físico | No | Sí
Azure|N/D|Sí

## <a name="replicate-virtual-machines"></a>Replicación de máquinas virtuales

Para iniciar la replicación de todas las máquinas virtuales de la granja de servidores web IIS en Azure, siga las instrucciones de [Conmutación por error de prueba a Azure en Site Recovery](site-recovery-test-failover-to-azure.md).

Si usa una dirección IP estática, puede especificar la dirección IP que quiere que se utilice con la máquina virtual. Para establecer la dirección IP, vaya a la **configuración de proceso y red** > **IP DE DESTINO**.

![Captura de pantalla que muestra cómo establecer la dirección IP de destino en el panel de red y proceso de Site Recovery](./media/site-recovery-active-directory/dns-target-ip.png)

## <a name="create-a-recovery-plan"></a>Creación de un plan de recuperación
Un plan de recuperación admite la secuenciación de distintas capas en una aplicación de varios niveles durante una conmutación por error. La secuenciación ayuda a mantener la coherencia de la aplicación. Cuando cree un plan de recuperación para una aplicación web de varios niveles, complete los pasos descritos en [Creación de un plan de recuperación mediante Site Recovery](site-recovery-create-recovery-plans.md).

### <a name="add-virtual-machines-to-failover-groups"></a>Adición de máquinas virtuales a grupos de conmutación por error
Por lo general, una aplicación web de IIS de varias capas consta de los siguientes componentes:
* Una capa de base de datos que tiene máquinas virtuales de SQL.
* La capa web, que consta de un servidor IIS y una capa de aplicación. 

Agregue máquinas virtuales a diferentes grupos según la capa:

1. Cree un plan de recuperación. Agregue las máquinas virtuales de la capa de base de datos en el grupo 1. De esta forma, se garantiza que las máquinas virtuales de la capa de base de datos se apagan las últimas y se activan primero.
1. Agregue las máquinas virtuales de la capa de aplicación en el grupo 2. De esta forma, se garantiza que las máquinas virtuales de la capa de aplicación se activan una vez que lo han hecho las de la capa de base de datos.
1. Agregue las máquinas virtuales de la capa web en el grupo 3. De esta forma, se garantiza que las máquinas virtuales de la capa web se activan una vez que lo han hecho las de la capa de aplicación.
1. Agregue las máquinas virtuales de equilibrio de carga en el grupo 4. De esta forma, se garantiza que las máquinas virtuales de equilibrio de carga se activan una vez que lo han hecho las de la capa web.

Para más información, consulte cómo [personalizar el plan de recuperación](site-recovery-runbook-automation.md#customize-the-recovery-plan).


### <a name="add-a-script-to-the-recovery-plan"></a>Adición de un script al plan de recuperación
Es posible que tenga que realizar algunas operaciones en las máquinas virtuales de Azure tras la conmutación por error o durante una conmutación por error de prueba. Algunas de las operaciones posteriores a la conmutación por error se pueden automatizar. Por ejemplo, puede actualizar la entrada DNS, cambiar un enlace de sitio o cambiar una cadena de conexión mediante la adición de los scripts correspondientes al plan de recuperación. En [Adición de scripts de VMM a los planes de recuperación](./hyper-v-vmm-recovery-script.md) se describe cómo configurar tareas automatizadas mediante un script.

#### <a name="dns-update"></a>Actualización de DNS
Si el DNS está configurado para que se actualice de forma dinámica, por lo general, las máquinas virtuales lo actualizarán con la nueva dirección IP al iniciarse. Si quiere incorporar un paso explícito para actualizar DNS con las nuevas direcciones IP de las máquinas virtuales, agregue un [script para actualizar direcciones IP en DNS](https://aka.ms/asr-dns-update) como una acción posterior a la conmutación por error en los grupos de planes de recuperación.  

#### <a name="connection-string-in-an-applications-webconfig"></a>Cadena de conexión del archivo web.config de una aplicación
La cadena de conexión especifica la base de datos con la que se comunica el sitio web. Si la cadena de conexión contiene el nombre de la máquina virtual de la base de datos, no es necesario realizar ningún paso adicional después de realizar la conmutación por error. La aplicación puede comunicarse automáticamente con la base de datos. Asimismo, si la dirección IP de la máquina virtual de la base de datos no cambia, no será necesario actualizar la cadena de conexión. 

Si la cadena de conexión hace referencia a la máquina virtual de la base de datos mediante una dirección IP, es necesario actualizarla después de realizar la conmutación por error. Por ejemplo, la siguiente cadena de conexión apunta a la base de datos con la dirección 127.0.1.2:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
<connectionStrings>
<add name="ConnStringDb1" connectionString="Data Source= 127.0.1.2\SqlExpress; Initial Catalog=TestDB1;Integrated Security=False;" />
</connectionStrings>
</configuration>
```

Para actualizar la cadena de conexión en la capa web, agregue un [script de actualización de la conexión de IIS](https://gallery.technet.microsoft.com/Update-IIS-connection-2579aadc) después del grupo 3 en el plan de recuperación.

#### <a name="site-bindings-for-the-application"></a>Enlaces del sitio en la aplicación
Todos los sitios se componen de información de enlace. La información de enlace incluye el tipo de enlace, la dirección IP en la que el servidor IIS escucha las solicitudes del sitio, el número de puerto y los nombres de host del sitio. Durante la conmutación por error, puede ser necesario actualizar estos enlaces si hay un cambio en la dirección IP que tienen asociada.

> [!NOTE]
>
> Si establece el enlace del sitio en **Todas sin asignar**, no es necesario actualizar este enlace después de la conmutación por error. Además, si la dirección IP asociada con un sitio no cambia después de la conmutación por error, no es necesario actualizar el enlace del sitio. (La retención de la dirección IP depende de la arquitectura de red y de las subredes asignadas a los sitios principal y de recuperación. Puede que actualizarlos no sea factible en su organización).

![Captura de pantalla que muestra la configuración del enlace de TLS/SSL](./media/site-recovery-iis/sslbinding.png)

Si la dirección IP está asociada con un sitio, actualice todos los enlaces de sitio con la nueva dirección IP. Para cambiar los enlaces de sitio, agregue un [script de actualización de capa web de IIS](https://aka.ms/asr-web-tier-update-runbook-classic) después del grupo 3 en el plan de recuperación.

#### <a name="update-the-load-balancer-ip-address"></a>Actualización de la dirección IP del equilibrador de carga
Si tiene una máquina virtual de ARR, para actualizar la dirección IP, agregue un [script de conmutación por error de ARR para IIS](https://aka.ms/asr-iis-arrtier-failover-script-classic) después al grupo 4.

#### <a name="tlsssl-certificate-binding-for-an-https-connection"></a>Enlace de certificado TLS/SSL para una conexión HTTPS
Un sitio web puede tener un certificado TLS/SSL asociado que ayude a garantizar una comunicación segura entre el servidor web y el explorador del usuario. Si el sitio web tiene una conexión HTTPS y también tiene un enlace de sitio HTTPS asociado a la dirección IP del servidor IIS con un enlace de certificado TLS/SSL, debe agregar un nuevo enlace de sitio para el certificado con la dirección IP de la máquina virtual de IIS después de la conmutación por error.

El certificado TLS/SSL se puede emitir para estos componentes:

* El nombre de dominio completo del sitio web.
* El nombre del servidor.
* Un certificado comodín para el nombre de dominio.  
* Una dirección IP. Si se emite el certificado TLS/SSL para la dirección IP del servidor IIS, se debe emitir otro certificado TLS/SSL para la dirección IP del servidor IIS en el sitio de Azure. Será necesario crear un enlace TLS adicional para este certificado. Por este motivo, se recomienda no usar un certificado TLS/SSL emitido para la dirección IP. Esta opción se usa mucho menos y dejará de utilizarse pronto con arreglo a los nuevos cambios del foro de entidad de certificación/explorador.

#### <a name="update-the-dependency-between-the-web-tier-and-the-application-tier"></a>Actualización de la dependencia entre la capa web y la capa de aplicación
Si tiene una dependencia específica de la aplicación basada en la dirección IP de las máquinas virtuales, debe actualizarla tras la conmutación por error.

## <a name="run-a-test-failover"></a>Ejecución de una conmutación por error de prueba

1. En Azure Portal, seleccione el almacén de Recovery Services.
2. Seleccione el plan de recuperación que creó para la granja de servidores web de IIS.
3. Seleccione **Conmutación por error de prueba**.
4. Para iniciar el proceso de conmutación por error de prueba, seleccione el punto de recuperación y la red virtual de Azure.
5. Cuando el entorno secundario esté activo, podrá realizar validaciones.
6. Cuando las validaciones hayan finalizado y quiera limpiar el entorno de conmutación por error de prueba, seleccione **Validations complete** (Validaciones finalizadas).

Para más información, consulte [Conmutación por error de prueba a Azure en Site Recovery](site-recovery-test-failover-to-azure.md).

## <a name="run-a-failover"></a>Ejecución de la conmutación por error

1. En Azure Portal, seleccione el almacén de Recovery Services.
1. Seleccione el plan de recuperación que creó para la granja de servidores web de IIS.
1. Seleccione **Failover** (Conmutación por error).
1. Para iniciar el proceso de conmutación por error, seleccione el punto de recuperación.

Para más información, consulte [Conmutación por error en Site Recovery](site-recovery-failover.md).

## <a name="next-steps"></a>Pasos siguientes
* Aprenda más sobre la [replicación de otras aplicaciones](site-recovery-workload.md) mediante Site Recovery.
