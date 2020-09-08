---
title: 'Conceptos: control de acceso basado en roles (RBAC)'
description: Más información sobre las funcionalidades clave del control de acceso basado en rol para Azure VMware Solution
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 6b9e83f0442953f59d02c42514a8550301ea947b
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88752254"
---
# <a name="role-based-access-control-rbac-for-azure-vmware-solution"></a>Control de acceso basado en rol (RBAC) para Azure VMware Solution

En una implementación local de vCenter y ESXi, el administrador tiene acceso a la cuenta de administrator@vsphere.local vCenter y puede tener asignados usuarios o grupos de Active Directory (AD) adicionales. Sin embargo, en una implementación de Azure VMware Solution, el administrador no tiene acceso a la cuenta de usuario de administrador, pero puede asignar usuarios y grupos de AD al rol CloudAdmin en vCenter.  Además, el usuario de la nube privada de Azure VMware Solution no tiene permiso para acceder o configurar componentes de administración específicos que Microsoft admite y administra, como clústeres, hosts, almacenes de datos y conmutadores virtuales distribuidos.


En Azure VMware Solution, vCenter tiene un usuario local integrado llamado cloudadmin que está asignado al rol integrado CloudAdmin. El usuario cloudadmin local se usa para configurar usuarios adicionales en AD. El rol CloudAdmin, en general, tiene el privilegio para crear y administrar cargas de trabajo en la nube privada (máquinas virtuales, grupos de recursos, almacenes de datos y redes). El rol CloudAdmin en Azure VMware Solution tiene un conjunto específico de privilegios de vCenter que son distintos de otras soluciones en la nube de VMware.   

> [!NOTE]
> Actualmente, la Azure VMware Solution no ofrece roles personalizados en vCenter ni en el portal de Azure VMware Solution. 

## <a name="azure-vmware-solution-cloudadmin-role-on-vcenter"></a>Rol CloudAdmin de Azure VMware Solution en vCenter

Puede ver los privilegios concedidos al rol CloudAdmin de Azure VMware Solution en su instancia de vCenter de la nube privada de Azure VMware Solution.

1. Inicie sesión en el cliente de SDDC vSphere y vaya a **Menú** > **Administración**.
1. Mire en **Control de acceso**, seleccione **Roles**.
1. En la lista de roles, seleccione **CloudAdmin** y luego, seleccione **Privilegios**. 

   :::image type="content" source="media/rbac-cloudadmin-role-privileges.png" alt-text="Cómo ver los privilegios de rol de CloudAdmin en el cliente de vSphere":::

El rol CloudAdmin de Azure VMware Solution tiene los siguientes privilegios en vCenter. Consulte la [documentación del producto VMware](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-ED56F3C4-77D0-49E3-88B6-B99B8B437B62.html) para obtener una explicación detallada de cada privilegio.

| Privilegio | Descripción |
| --------- | ----------- |
| **Alarmas** | Confirmar alarma<br />Crear alarma<br />Deshabilitar acción de alarma<br />Modificar alarma<br />Eliminar alarma<br />Establecer estado de la alarma |
| **Permisos** | Modificar permisos |
| **Biblioteca de contenido** | Agregar elemento de biblioteca<br />Crear una suscripción para una biblioteca publicada<br />Crear biblioteca local<br />Crear biblioteca suscrita<br />Eliminar elemento de biblioteca<br />Eliminar biblioteca local<br />Eliminar biblioteca suscrita<br />Eliminar la suscripción de una biblioteca publicada<br />Descarga de archivos<br />Expulsar elementos de biblioteca<br />Desalojar biblioteca suscrita<br />Importar almacenamiento<br />Sondear información de suscripción<br />Publicar un elemento de biblioteca en sus suscriptores<br />Publicar una biblioteca en sus suscriptores<br />Leer en el almacenamiento<br />Sincronizar elemento de biblioteca<br />Sincronizar biblioteca suscrita<br />Escribir introspección<br />Actualizar valores de configuración<br />Actualizar archivos<br />Actualizar biblioteca<br />Actualizar elemento de biblioteca<br />Actualizar biblioteca local<br />Actualizar biblioteca suscrita<br />Actualizar la suscripción de una biblioteca publicada<br />Ver valores de configuración |
| **Operaciones criptográficas** | Acceso directo |
| **Almacén de datos** | Asignar espacio<br />Examen de un almacén de datos<br />Configurar almacén de datos<br />Operaciones de archivo de bajo nivel<br />Quitar archivos<br />Actualizar metadatos de máquina virtual |
| **Carpeta** | Crear carpeta<br />Eliminar carpeta<br />Mover carpeta<br />Cambiar nombre de la carpeta |
| **Global** | Cancelar tarea<br />Etiqueta global<br />Health<br />Registrar eventos<br />Administrar atributos personalizados<br />Administradores de servicio<br />Establecer atributo personalizado<br />Etiqueta del sistema |
| **Host** | Replica de vSphere<br />&#160;&#160;&#160;&#160;Administrar la replicación |
| **Etiquetas de vSphere** | Asignar y anular la asignación de la etiqueta vSphere<br />Crear etiqueta vSphere<br />Crear categoría de etiqueta vSphere<br />Eliminar etiqueta vSphere<br />Eliminar categoría de etiqueta vSphere<br />Editar etiqueta vSphere<br />Editar categoría de etiqueta vSphere<br />Modificar el campo UsedBy de la categoría<br />Modificar el campo UsedBy de la etiqueta |
| **Network** | Asignar red |
| **Recurso** | Aplicar recomendaciones<br />Asignar vApp a un grupo de recursos<br />Asignar máquina virtual a grupo de recursos<br />Crear grupo de recursos<br />Migrar máquina virtual apagada<br />Migrar máquina virtual encendida<br />Modificar grupo de recursos<br />Mover grupo de recursos<br />Consultar vMotion<br />Eliminar grupo de recursos<br />Cambiar nombre del grupo de recursos |
| **Tarea programada** | Crear la tarea<br />Modificar tarea<br />Quitar tarea<br />Ejecutar tarea |
| **Sesiones** | Message<br />Validar sesión |
| **Perfil** | Vista de almacenamiento controlado por perfiles |
| **Vista de almacenamiento** | Ver |
| **vApp** | Agregar máquina virtual<br />Asignar grupo de recursos<br />Asignar vApp<br />Clonar<br />Crear<br />Eliminar<br />Exportación<br />Importar<br />Move<br />Apagado<br />Encendido<br />Cambiar nombre<br />Suspender<br />Unregister<br />Ver el entorno de OVF<br />Configuración de aplicaciones de vApp<br />Configuración de instancias de vApp<br />Configuración de managedBy de vApp<br />Configuración de recursos de vApp |
| **Máquina virtual** | Cambiar configuración<br />&#160;&#160;&#160;&#160;Adquirir concesión de disco<br />&#160;&#160;&#160;&#160;Agregar disco existente<br />&#160;&#160;&#160;&#160;Agregar nuevo disco<br />&#160;&#160;&#160;&#160;Agregar o quitar dispositivo<br />&#160;&#160;&#160;&#160;Configuración avanzada<br />&#160;&#160;&#160;&#160;Cambiar cantidad de CPU<br />&#160;&#160;&#160;&#160;Cambiar memoria<br />&#160;&#160;&#160;&#160;Cambiar configuración<br />&#160;&#160;&#160;&#160;Cambiar la ubicación del archivo de intercambio<br />&#160;&#160;&#160;&#160;Cambiar recurso<br />&#160;&#160;&#160;&#160;Configurar dispositivo USB de host<br />&#160;&#160;&#160;&#160;Configurar dispositivo sin formato<br />&#160;&#160;&#160;&#160;Configurar managedBy<br />&#160;&#160;&#160;&#160;Mostrar la configuración de la conexión<br />&#160;&#160;&#160;&#160;Extender disco virtual<br />&#160;&#160;&#160;&#160;Modificar la configuración del dispositivo<br />&#160;&#160;&#160;&#160;Compatibilidad de tolerancia a errores de consulta<br />&#160;&#160;&#160;&#160;Consultar archivos sin propietario<br />&#160;&#160;&#160;&#160;Recarga desde rutas de acceso<br />&#160;&#160;&#160;&#160;Quitar disco<br />&#160;&#160;&#160;&#160;Cambiar el nombre<br />&#160;&#160;&#160;&#160;Restablecer información de invitado<br />&#160;&#160;&#160;&#160;Establecer anotación<br />&#160;&#160;&#160;&#160;Alternancia del seguimiento de cambios de disco<br />&#160;&#160;&#160;&#160;Alternar la bifurcación primaria<br />&#160;&#160;&#160;&#160;Actualización de la compatibilidad de máquinas virtuales<br />Editar inventario<br />&#160;&#160;&#160;&#160;Crear a partir de existente<br />&#160;&#160;&#160;&#160;Crear nuevo<br />&#160;&#160;&#160;&#160;Mover<br />&#160;&#160;&#160;&#160;Registrar<br />&#160;&#160;&#160;&#160;Quitar<br />&#160;&#160;&#160;&#160;Anular el registro<br />Operaciones de invitado<br />&#160;&#160;&#160;&#160;Modificación de alias de operación de invitado<br />&#160;&#160;&#160;&#160;Consulta de alias de operación de invitado<br />&#160;&#160;&#160;&#160;Modificaciones de operaciones de invitado<br />&#160;&#160;&#160;&#160;Ejecución del programa de operaciones de invitado<br />&#160;&#160;&#160;&#160;Consultas de operaciones de invitado<br />Interacción<br />&#160;&#160;&#160;&#160;Responder pregunta<br />&#160;&#160;&#160;&#160;Operación de copia de seguridad en la máquina virtual<br />&#160;&#160;&#160;&#160;Configurar elementos multimedia de CD<br />&#160;&#160;&#160;&#160;Configurar soporte de disquete<br />&#160;&#160;&#160;&#160;Conexión de dispositivos<br />&#160;&#160;&#160;&#160;Interacción de la consola<br />&#160;&#160;&#160;&#160;Crear captura de pantalla<br />&#160;&#160;&#160;&#160;Desfragmentar todos los discos<br />&#160;&#160;&#160;&#160;Arrastrar y colocar<br />&#160;&#160;&#160;&#160;Administración de sistemas operativos invitados por VIX API<br />&#160;&#160;&#160;&#160;Inyectar códigos de digitalización de USB HID<br />&#160;&#160;&#160;&#160;Instalación de herramientas de VMware<br />&#160;&#160;&#160;&#160;Pausar o desactivar la pausa<br />&#160;&#160;&#160;&#160;Realizar operaciones de borrado o reducción<br />&#160;&#160;&#160;&#160;Apagar<br />&#160;&#160;&#160;&#160;Encender<br />&#160;&#160;&#160;&#160;Registrar sesión en la máquina virtual<br />&#160;&#160;&#160;&#160;Sesión de reproducción en máquina virtual<br />&#160;&#160;&#160;&#160;Suspender<br />&#160;&#160;&#160;&#160;Suspender tolerancia a errores<br />&#160;&#160;&#160;&#160;Probar la conmutación por error<br />&#160;&#160;&#160;&#160;Reinicio de prueba de máquina virtual secundaria<br />&#160;&#160;&#160;&#160;Desactivar la tolerancia a errores<br />&#160;&#160;&#160;&#160;Activar la tolerancia a errores<br />Aprovisionamiento<br />&#160;&#160;&#160;&#160;Permitir acceso al disco<br />&#160;&#160;&#160;&#160;Permitir el acceso a archivos<br />&#160;&#160;&#160;&#160;Permitir acceso a disco de solo lectura<br />&#160;&#160;&#160;&#160;Permitir descarga de máquina virtual<br />&#160;&#160;&#160;&#160;Clonar plantilla<br />&#160;&#160;&#160;&#160;Clonar máquina virtual<br />&#160;&#160;&#160;&#160;Crear plantilla a partir de la máquina virtual<br />&#160;&#160;&#160;&#160;Personalizar invitado<br />&#160;&#160;&#160;&#160;Implementar plantilla<br />&#160;&#160;&#160;&#160;Marcar como plantilla<br />&#160;&#160;&#160;&#160;Modificar especificación de personalización<br />&#160;&#160;&#160;&#160;Promover discos<br />&#160;&#160;&#160;&#160;Leer especificaciones de personalización<br />Configuración del servicio<br />&#160;&#160;&#160;&#160;Permitir notificaciones<br />&#160;&#160;&#160;&#160;Permitir sondeo de notificaciones de eventos globales<br />&#160;&#160;&#160;&#160;Administrar la configuración del servicio<br />&#160;&#160;&#160;&#160;Modificar la configuración del servicio<br />&#160;&#160;&#160;&#160;Configuraciones del servicio de consulta<br />&#160;&#160;&#160;&#160;Leer configuración de servicio<br />Administración de instantáneas<br />&#160;&#160;&#160;&#160;Crear instantánea<br />&#160;&#160;&#160;&#160;Quitar instantánea<br />&#160;&#160;&#160;&#160;Cambiar nombre de instantánea<br />&#160;&#160;&#160;&#160;Revertir instantánea<br />Replica de vSphere<br />&#160;&#160;&#160;&#160;Configurar la replicación<br />&#160;&#160;&#160;&#160;Administrar la replicación<br />&#160;&#160;&#160;&#160;Supervisión de la replicación |
| **vService** | Crear dependencia<br />Destruir dependencia<br />Volver a configurar la configuración de dependencia<br />Actualizar dependencia |



## <a name="next-steps"></a>Pasos siguientes

Consulte la [documentación del producto VMware](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-ED56F3C4-77D0-49E3-88B6-B99B8B437B62.html) para obtener una explicación detallada de cada privilegio.

<!-- LINKS - internal -->

<!-- LINKS - external-->
[VMware product documentation]: https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-ED56F3C4-77D0-49E3-88B6-B99B8B437B62.html

