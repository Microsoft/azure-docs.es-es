---
title: Instalación de Hybrid Cloud Extension (HCX)
description: Configuración de la solución VMware Hybrid Cloud Extension (HCX) para la nube privada de Azure VMware Solution
ms.topic: how-to
ms.date: 07/15/2020
ms.openlocfilehash: fb8497af33b364c1d2ab475233bd2a83ef1befad
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88752316"
---
# <a name="install-hcx-for-azure-vmware-solution"></a>Instalación de HCX para Azure VMware Solution

En este artículo, se recorren los procedimientos para configurar la solución VMware Hybrid Cloud Extension (HCX) para la nube privada de Azure VMware Solution. HCX permite la migración de las cargas de trabajo de VMware a la nube y a otros sitios conectados mediante diversos tipos de migración integrados compatibles con HCX.

HCX Advanced, la instalación predeterminada, admite hasta tres conexiones de sitio (locales o de nube a nube). Si se necesitan más de tres conexiones de sitio, los clientes tienen la opción de habilitar el complemento HCX Enterprise mediante el soporte técnico, que, de momento, está en versión preliminar. HCX Enterprise conlleva cargos adicionales para los clientes después de la disponibilidad general (GA), pero proporciona [características adicionales](https://cloud.vmware.com/community/2019/08/08/introducing-hcx-enterprise/).


Revise primero detenidamente las secciones [Antes de comenzar](#before-you-begin), [Requisitos de versión de software](#software-version-requirements) y [Requisitos previos](#prerequisites). 

A continuación, se van a examinar todos los procedimientos necesarios para:

> [!div class="checklist"]
> * Implementar OVA de HCX (conector) local
> * Activar y configurar HCX
> * Configurar el vínculo superior de red y la malla de servicio
> * Completar la instalación mediante la comprobación del estado del dispositivo

Después de completar la instalación, puede seguir los siguientes pasos recomendados que se proporcionan al final de este artículo.  

## <a name="before-you-begin"></a>Antes de empezar
    
* Revise la [serie de tutoriales](tutorial-network-checklist.md) básicos de Software Defined Datacenter (SDDC) de Azure VMware Solution.
* Revise y consulte la [documentación de VMware HCX](https://docs.vmware.com/en/VMware-HCX/index.html), incluida la guía de usuario de HCX.
* Revise los documentos de VMware [Migración de máquinas virtuales con VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-D0CD0CC6-3802-42C9-9718-6DA5FEC246C6.html?hWord=N4IghgNiBcIBIGEAaACAtgSwOYCcwBcMB7AOxAF8g).
* También, revise [Consideraciones de implementación de VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/install-checklist/GUID-C0A0E820-D5D0-4A3D-AD8E-EEAA3229F325.html).
* Opcionalmente, revise los materiales relacionados de VMware en HCX, como la [serie de blogs](https://blogs.vmware.com/vsphere/2019/10/cloud-migration-series-part-2.html) de VMware vSphere en HCX. 
* Solicite la activación de HCX Enterprise para Azure VMware Solution a través de los canales de soporte técnico de Azure VMware Solution.

El ajuste de tamaño de las cargas de trabajo con respecto a los recursos de proceso y almacenamiento es un paso esencial del planeamiento al prepararse para usar la solución HCX de la nube privada de Azure VMware Solution. Aborde el paso de ajuste de tamaño como parte del planeamiento inicial del entorno de la nube privada. 

También puede ajustar el tamaño de las cargas de trabajo si completa una evaluación de Azure VMware Solution en el portal de Azure Migrate (https://docs.microsoft.com/azure/migrate/how-to-create-azure-vmware-solution-assessment).

## <a name="software-version-requirements"></a>Requisitos de versión de software

Los componentes de la infraestructura deben ejecutar la versión mínima necesaria. 
                                                         
| Tipo de componente    | Requisitos del entorno de origen    | Requisitos del entorno de destino   |
| --- | --- | --- |
| vCenter Server   | 5,1<br/><br/>Si usa 5.5 U1 o versiones anteriores, emplee la interfaz de usuario de HCX independiente para las operaciones de HCX.  | 6.0 U2 y versiones posteriores   |
| ESXi   | 5.0    | ESXi 6.0 y versiones posteriores   |
| NSX    | Para la extensión de red de HCX de conmutadores lógicos en el origen: NSXv 6.2+ o NSX-T 2.4+   | NSXv 6.2+ o NSX-T 2.4+<br/><br/>Para el enrutamiento de proximidad de HCX: NSXv 6.4 + (el enrutamiento de proximidad no se admite con NSX-T) |
| vCloud Director   | No necesario: sin interoperabilidad con vCloud Director en el sitio de origen | Al integrar el entorno de destino con vCloud Director, la versión mínima es 9.1.0.2.  |

## <a name="prerequisites"></a>Requisitos previos

* Se debe configurar Global Reach de ExpressRoute entre los circuitos de ExpressRoute del SDDC de Azure VMware Solution y del entorno local.

* Todos los puertos necesarios deben estar abiertos entre el SDDC de Azure VMware Solution y el entorno local (consulte la [documentación de VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-E456F078-22BE-494B-8E4B-076EF33A9CF4.html)).

* Una dirección IP para HCX Manager en el entorno local y dos direcciones IP como mínimo para los dispositivos de interconexión (IX) y de extensión de red (NE).

* Los dispositivos locales IX y NE de HCX deben ser capaces de acceder a la infraestructura de vCenter y ESXi.

* Para implementar el dispositivo de interconexión WAN, además del bloque de direcciones de red CIDR /22 que se usa para la implementación del SDDC en Azure Portal, HCX requiere un bloque /29. Asegúrese de factorizar este requisito en el planeamiento de la red.

## <a name="deploy-the-vmware-hcx-ova-on-premises"></a>Implementación de OVA de VMware HCX en el entorno local

1. Inicie sesión en vCenter de SDDC de Azure VMware Solution y seleccione **HCX**.

   :::image type="content" source="media/hybrid-cloud-extension-installation/avs-vsphere-client.png" alt-text="Inicio de sesión en vCenter de SDDC de Azure VMware Solution y selección de HCX.":::

1. En **Administración**, seleccione **Actualizaciones del sistema** y, a continuación, seleccione **vínculo de descarga de solicitud** para descargar el archivo VMware HCX OVA.

   :::image type="content" source="media/hybrid-cloud-extension-installation/administration-updates.png" alt-text="En Administración, seleccione Actualizaciones del sistema y, a continuación, seleccione Solicitar vínculo de descarga para descargar el archivo VMware HCX OVA.":::

1. A continuación, vaya a la instancia local de vCenter y seleccione una plantilla OVF para implementarla en él.  

   :::image type="content" source="media/hybrid-cloud-extension-installation/select-template.png" alt-text="A continuación, vaya a la instancia local de vCenter y seleccione una plantilla OVF para implementarla en él.":::

1. Seleccione un nombre y una ubicación y, luego, elija un recurso o un clúster en el que implementar HCX. A continuación, revise los detalles y los recursos necesarios.  

   :::image type="content" source="media/hybrid-cloud-extension-installation/configure-template.png" alt-text="Seleccione un nombre y una ubicación y, luego, elija un recurso o un clúster en el que implementar HCX. A continuación, revise los detalles y los recursos necesarios.":::

1. Revise los términos de la licencia y, si está de acuerdo, seleccione el almacenamiento y la red necesarios. Luego, seleccione **Siguiente**.

1. En **Customize template** (Personalizar plantilla), escriba toda la información necesaria. 

   :::image type="content" source="media/hybrid-cloud-extension-installation/customize-template.png" alt-text="En Personalizar plantilla, escriba toda la información necesaria.":::

1. Seleccione **Next** (Siguiente), compruebe la configuración y elija **Finish** (Finalizar)para implementar OVA de HCX.

## <a name="activate-hcx"></a>Activación de HCX

Después de la instalación, realice los pasos siguientes.

1. Inicie sesión en HCX Manager local en `https://HCXManagerIP:9443` y conéctese con su nombre de usuario y contraseña. 

   > [!IMPORTANT]
   > Asegúrese de incluir el número de puerto `9443` con la dirección IP de HCX Manager.

1. En **Licensing** (Licencia), escriba **HCX Advanced Key** (Licencia de HCX Advanced).  

   :::image type="content" source="media/hybrid-cloud-extension-installation/hcx-key.png" alt-text="En Licensing (Licencia), escriba HCX Advanced Key (Licencia de HCX Advanced).":::
    
    > [!NOTE]
    > HCX Manager debe tener acceso abierto a Internet o un proxy configurado.

1. En **vCenter**, si es necesario, edite la información de vCenter.

   :::image type="content" source="media/hybrid-cloud-extension-installation/configure-vcenter.png" alt-text="En vCenter, si es necesario, edite la información de vCenter.":::

1. En **Ubicación del centro de datos**, si es necesario, edite la ubicación del centro de datos.

   :::image type="content" source="media/hybrid-cloud-extension-installation/system-location.png" alt-text="En Ubicación del centro de datos, si es necesario, edite la ubicación del centro de datos.":::

## <a name="configure-hcx"></a>Configuración de HCX 

1. Inicie sesión en vCenter local y en **Inicio**, seleccione **HCX**.

   :::image type="content" source="media/hybrid-cloud-extension-installation/hcx-vcenter.png" alt-text="Inicie sesión en vCenter local y en Inicio, seleccione HCX.":::

1. En **Infrastructure** (Infraestructura), seleccione **Site Pairing** > **Add a site pairing** (Emparejamiento de sitio > Agregar un emparejamiento de sitio).

   :::image type="content" source="media/hybrid-cloud-extension-installation/site-pairing.png" alt-text="En Infrastructure (Infraestructura), seleccione Site Pairing > Add a site pairing (Emparejamiento de sitio > Agregar un emparejamiento de sitio).":::

1. Escriba la dirección IP o el URL del HCX remoto, el nombre de usuario y contraseña de cloudadmin de Azure VMware Solution y, a continuación, seleccione **Conectar**.

   El sistema muestra el sitio conectado.

   :::image type="content" source="media/hybrid-cloud-extension-installation/site-connection.png" alt-text="El sistema muestra el sitio conectado.":::

1. En **Infrastructure** (Infraestructura), seleccione **Interconnect** > **Multi-Site Service Mesh** > **Network Profiles** > **Create Network Profile** (Interconectar > Malla de servicios de varios sitios > Perfiles de red > Crear perfil de red).

   :::image type="content" source="media/hybrid-cloud-extension-installation/create-network-profile.png" alt-text="En Infrastructure (Infraestructura), seleccione Interconnect > Multi-Site Service Mesh > Network Profiles > Create Network Profile (Interconectar > Malla de servicios de varios sitios > Perfiles de red > Crear perfil de red).":::

1. Para el nuevo perfil de red, escriba los intervalos de direcciones IP de IX y NE de HCX (se requieren dos direcciones IP como mínimo para los dispositivos IX y NE).

   :::image type="content" source="media/hybrid-cloud-extension-installation/enter-address-ranges.png" alt-text="Para el nuevo perfil de red, escriba los intervalos de direcciones IP de IX y NE de HCX (se requieren dos direcciones IP como mínimo para los dispositivos IX y NE).":::
  
   > [!NOTE]
   > El dispositivo de extensión de red (HCX-NE) tiene una relación uno a uno con un conmutador virtual distribuido (DVS).  

1. Ahora, seleccione **Compute profile** > **Create compute profile** (Perfil de proceso > Crear perfil de proceso).

1. Escriba un nombre de perfil de proceso y seleccione **Continue** (Continuar).  

   :::image type="content" source="media/hybrid-cloud-extension-installation/create-compute-profile.png" alt-text="Escriba un nombre de perfil de proceso y seleccione Continue (Continuar).":::

1. Seleccione los servicios que se van a habilitar, como la migración, la extensión de red o la recuperación ante desastres y, a continuación, seleccione **Continue** (Continuar).

   :::image type="content" source="media/hybrid-cloud-extension-installation/select-services.png" alt-text="Seleccione los servicios que se van a habilitar, como la migración, la extensión de red o la recuperación ante desastres y, a continuación, seleccione Continue (Continuar).":::

1. En **Select Service Resources** (Seleccionar recursos de servicio), seleccione uno o más recursos de servicio para los que se deban habilitar los servicios de HCX seleccionados. Seleccione **Continuar**.

   :::image type="content" source="media/hybrid-cloud-extension-installation/select-service-resources.png" alt-text="En Select Service Resources (Seleccionar recursos de servicio), seleccione uno o más recursos de servicio para los que se deban habilitar los servicios de HCX seleccionados. Seleccione Continue (Continuar).":::
  
   > [!NOTE]
   > Seleccione clústeres específicos en los que las máquinas virtuales de origen tienen como destino la migración mediante HCX.

1. Seleccione **Datastore** (Almacén de datos) y elija **Continue** (Continuar). 
      
   Seleccione cada recurso de proceso y almacenamiento para implementar los dispositivos de interconexión de HCX. Cuando se seleccionan varios recursos, HCX usa el primero seleccionado hasta que se agota su capacidad.  

   :::image type="content" source="media/hybrid-cloud-extension-installation/deployment-resources.png" alt-text="Seleccione cada recurso de proceso y almacenamiento para implementar los dispositivos de interconexión de HCX. Cuando se seleccionan varios recursos, HCX usa el primero seleccionado hasta que se agota su capacidad.":::

1. Seleccione el perfil de red de administración creado en **Network Profiles** (Perfiles de red) y elija **Continue** (Continuar).  
      
   Seleccione el perfil de red a través del que se puede acceder a la interfaz de administración de vCenter y los hosts ESXi. Si aún no ha definido este tipo de perfil de red, puede crearlo aquí.  

   :::image type="content" source="media/hybrid-cloud-extension-installation/management-network-profile.png" alt-text="Seleccione el perfil de red a través del que se puede acceder a la interfaz de administración de vCenter y a los hosts ESXi. Si aún no ha definido este tipo de perfil de red, puede crearlo aquí.":::

1. Seleccione **Network Uplink** (Vínculo superior de red) y elija **Continue** (Continuar).
      
   Seleccione uno o varios perfiles de red, de modo que se cumpla una de las siguientes condiciones:  
   * Se puede acceder a los dispositivos de interconexión del sitio remoto a través de esta red.  
   * Los dispositivos remotos pueden tener acceso a los dispositivos de interconexión locales a través de esta red.  
    
   Si tiene redes punto a punto como Direct Connect que no se comparten entre varios sitios, puede omitir este paso, ya que los perfiles de proceso se comparten con varios sitios. En tales casos, los perfiles de vínculo superior de red se pueden invalidar y especificar durante la creación de la malla de servicio de interconexión.  

   :::image type="content" source="media/hybrid-cloud-extension-installation/uplink-network-profile.png" alt-text="Seleccione Network Uplink (Vínculo superior de red) y elija Continue (Continuar).":::

1. Seleccione **vMotion Network Profile** (Perfil de red de vMotion) y elija **Continue** (Continuar).
      
   Seleccione el perfil de red a través del cual es posible comunicarse con la interfaz vMotion de los hosts ESXi. Si aún no ha definido este tipo de perfil de red, puede crearlo aquí. Si no tiene una red de vMotion, seleccione **Management Network Profile** (Perfil de red de administración).  

   :::image type="content" source="media/hybrid-cloud-extension-installation/vmotion-network-profile.png" alt-text="Seleccione vMotion Network Profile (Perfil de red de vMotion) y elija Continue (Continuar).":::

1. En **Select vSphere Replication Network Profile** (Seleccionar el perfil de red de replicación de vSphere), seleccione un perfil de red que la interfaz de replicación de vSphere de ESXi hospede y, después, seleccione **Continue** (Continuar).
      
   En la mayoría de los casos, este perfil es el mismo que el perfil de red de administración.  

   :::image type="content" source="media/hybrid-cloud-extension-installation/vsphere-replication-network-profile.png" alt-text="En Select vSphere Replication Network Profile (Seleccionar el perfil de red de replicación de vSphere), seleccione un perfil de red que la interfaz de replicación de vSphere de ESXi hospede y, después, seleccione Continue (Continuar).":::

1. En **Select Distributed Switches for Network Extensions** (Seleccionar conmutadores distribuidos para extensiones de red), seleccione el DVS en el que tiene las redes en las que las máquinas virtuales se integrarán y con las que están conectadas.  Seleccione **Continuar**.  

   :::image type="content" source="media/hybrid-cloud-extension-installation/distributed-switches.png" alt-text="En Select Distributed Switches for Network Extensions (Seleccionar conmutadores distribuidos para extensiones de red), seleccione el DVS en el que tiene las redes en las que las máquinas virtuales se integrarán y con las que están conectadas.  Seleccione Continue (Continuar).":::

1. Revise las reglas de conexión y seleccione **Continue** (Continuar).  

   :::image type="content" source="media/hybrid-cloud-extension-installation/complete-compute-profile.png" alt-text="Revise las reglas de conexión y seleccione Continue (Continuar).":::

1.  Seleccione **Finish** (Finalizar) para crear el perfil de proceso.

## <a name="configure-network-uplink"></a>Configuración del vínculo superior de red

Ahora, configure el cambio de perfil de red en el SCCD de Azure VMware Solution para el vínculo superior de red.

1. Inicie sesión en NSX-T del SDDC para crear un nuevo conmutador lógico, o bien use un conmutador lógico existente que pueda usarse para el vínculo superior de red entre el entorno local y el SDDC de Azure VMware Solution.

1. Cree un perfil de red para el vínculo superior de HCX en el SDDC de Azure VMware Solution que pueda usarse para la comunicación entre este y el entorno local.  

   :::image type="content" source="media/hybrid-cloud-extension-installation/network-profile-uplink.png" alt-text="Cree un perfil de red para el vínculo superior de HCX en el SDDC de Azure VMware Solution que pueda usarse para la comunicación entre este y el entorno local.":::

1. Escriba un nombre para el perfil de red y al menos 4 o 5 direcciones IP libres basadas en la extensión de red L2 necesaria.  

   :::image type="content" source="media/hybrid-cloud-extension-installation/configure-uplink-profile.png" alt-text="Escriba un nombre para el perfil de red y al menos 4 o 5 direcciones IP libres basadas en la extensión de red L2 necesaria.":::

1. Seleccione **Create** (Crear) para finalizar la configuración del SDDC de Azure VMware Solution.

## <a name="configure-service-mesh"></a>Configuración de la malla de servicio

Ahora, configure la malla de servicio entre el entorno local y el SDDC de Azure VMware Solution.

1. Inicie sesión en vCenter de SDDC de Azure VMware Solution y seleccione **HCX**.

2. En **Infrastructure** (Infraestructura), seleccione **Interconnect** > **Service Mesh** > **Create Service Mesh** (Interconectar > Malla de servicios > Crear malla de servicios) para configurar la red y procesar los perfiles creados en los pasos anteriores.    

   :::image type="content" source="media/hybrid-cloud-extension-installation/configure-service-mesh.png" alt-text="En Infrastructure (Infraestructura), seleccione Interconnect > Service Mesh > Create Service Mesh (Interconectar > Malla de servicios > Crear malla de servicios) para configurar la red y procesar los perfiles creados en los pasos anteriores.":::

3. Seleccione sitios emparejados para habilitar la capacidad híbrida y seleccione **Continue** (Continuar).   

   :::image type="content" source="media/hybrid-cloud-extension-installation/select-paired-sites.png" alt-text="Seleccione sitios emparejados para habilitar la capacidad híbrida y seleccione Continue (Continuar).":::

4. Seleccione los perfiles de proceso de origen y remoto para permitir los servicios híbridos y seleccione **Continue** (Continuar).
      
   Las selecciones definen los recursos, donde las máquinas virtuales pueden consumir los servicios de HCX.  

   :::image type="content" source="media/hybrid-cloud-extension-installation/enable-hybridity.png" alt-text="Las selecciones definen los recursos, donde las máquinas virtuales pueden consumir los servicios de HCX.":::

5. Seleccione los servicios que quiera habilitar y luego **Continue** (Continuar).  

   :::image type="content" source="media/hybrid-cloud-extension-installation/hcx-services.png" alt-text="Seleccione los servicios que quiera habilitar y luego Continue (Continuar).":::

6. En **Advanced Configuration - Override Uplink Network profiles** (Configuración avanzada: Invalidar los perfiles de vínculo superior de red), seleccione **Continue** (Continuar).  
      
   Los perfiles de vínculo superior de red se usan para conectarse a la red a través de la cual se puede acceder a los dispositivos de interconexión del sitio remoto.  

   :::image type="content" source="media/hybrid-cloud-extension-installation/override-uplink-profiles.png" alt-text="Los perfiles de vínculo superior de red se usan para conectarse a la red a través de la cual se puede acceder a los dispositivos de interconexión del sitio remoto.":::

7. Seleccione **Configure the Network Extension Appliance Scale Out** (Configurar la escalabilidad horizontal de dispositivo de extensión de red). 

   :::image type="content" source="media/hybrid-cloud-extension-installation/network-extension-scale-out.png" alt-text="Seleccione Configure the Network Extension Appliance Scale Out (Configurar la escalabilidad horizontal de dispositivo de extensión de red).":::

8. Escriba el número de dispositivos correspondiente al número de conmutadores de DVS.  

   :::image type="content" source="media/hybrid-cloud-extension-installation/appliance-scale.png" alt-text="Escriba el número de dispositivos correspondiente al número de conmutadores de DVS.":::

9. Seleccione **Continue** (Continuar).  

   :::image type="content" source="media/hybrid-cloud-extension-installation/traffic-engineering.png" alt-text="Seleccione Continue (Continuar) para omitir.":::

10. Revise la vista previa de la topología y seleccione **Continue** (Continuar). 

11. A continuación, escriba un nombre descriptivo para esta malla de servicios y seleccione **Finalizar** para terminar.  

   :::image type="content" source="media/hybrid-cloud-extension-installation/complete-service-mesh.png" alt-text="Finalización de la malla de servicio":::

   La malla de servicio está implementada y configurada.  

   :::image type="content" source="media/hybrid-cloud-extension-installation/deployed-service-mesh.png" alt-text="Malla de servicio implementada":::

## <a name="check-appliance-status"></a>Comprobación del estado del dispositivo
Para comprobar el estado del dispositivo, seleccione **Interconnect** > **Appliances** (Interconexión > Dispositivos). 

:::image type="content" source="media/hybrid-cloud-extension-installation/appliance-status.png" alt-text="Compruebe el estado del dispositivo.":::

## <a name="next-steps"></a>Pasos siguientes

Cuando el valor de **Tunnel Status** (Estado del túnel) es **UP** (ACTIVO) y verde, está listo para migrar y proteger las máquinas virtuales de Azure VMware Solution con HCX. Consulte la [documentación de VMware HCX](https://docs.vmware.com/en/VMware-HCX/index.html) y el artículo [Migración de máquinas virtuales con VMware HCX](https://docs.vmware.com/en/VMware-HCX/services/user-guide/GUID-D0CD0CC6-3802-42C9-9718-6DA5FEC246C6.html?hWord=N4IghgNiBcIBIGEAaACAtgSwOYCcwBcMB7AOxAF8g) en la documentación técnica de VMware.
