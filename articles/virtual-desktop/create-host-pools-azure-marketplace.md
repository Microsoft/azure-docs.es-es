---
title: Grupo de hosts de Windows Virtual Desktop en Azure Portal
description: Creación de un grupo de hosts de Windows Virtual Desktop con Azure Portal.
author: Heidilohr
ms.topic: tutorial
ms.date: 04/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: c9a421e15f3561bb4de7f528ab1c707a0251dfe5
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88002657"
---
# <a name="tutorial-create-a-host-pool-with-the-azure-portal"></a>Tutorial: Creación de un grupo de hosts con Azure Portal

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop con objetos de Windows Virtual Desktop de Azure Resource Manager. Si usa Windows Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/create-host-pools-azure-marketplace-2019.md). Cualquier objeto que cree con Windows Virtual Desktop (clásico) no se puede administrar con Azure Portal.

Los grupos de hosts son una colección de una o varias máquinas virtuales (VM) idénticas en entornos de Windows Virtual Desktop. Cada grupo de hosts puede contener un grupo de aplicaciones con las que los usuarios pueden interactuar igual que harían en un equipo de escritorio físico.

Este artículo le guía en el proceso de configuración para crear un grupo de hosts para un entorno de Windows Virtual Desktop mediante Azure Portal. El método que se ofrece proporciona una interfaz de usuario basada en explorador para crear un grupo de hosts en Windows Virtual Desktop, crear un grupo de recursos con máquinas virtuales en una suscripción de Azure, unir esas máquinas virtuales al dominio de Azure Active Directory (AD) y registrar las máquinas virtuales en Windows Virtual Desktop.

## <a name="prerequisites"></a>Requisitos previos

Deberá especificar los siguientes parámetros para crear un grupo de hosts:

- El nombre de la imagen de máquina virtual
- Configuración de VM
- Las propiedades de proceso y red
- Las propiedades del grupo de hosts de Windows Virtual Desktop

También deberá saber lo siguiente:

- Dónde está el origen de la imagen que quiere usar. ¿Está en Azure Gallery o es una imagen personalizada?
- Sus credenciales de unión un dominio.

Asegúrese también de que ha registrado el proveedor de recursos Microsoft.DesktopVirtualization. Si aún no lo ha hecho, vaya a **Suscripciones**, seleccione el nombre de su suscripción y, a continuación, **Proveedores de recursos de Azure**.

Cuando se crea un grupo de hosts de Windows Virtual Desktop con la plantilla de Azure Resource Manager, se puede crear una máquina virtual desde una imagen de Azure Gallery, una imagen administrada o una imagen no administrada. Para obtener más información sobre cómo crear imágenes de máquina virtual, vea [Preparación de un VHD o un VHDX de Windows para cargar en Azure](../virtual-machines/windows/prepare-for-upload-vhd-image.md) y [Creación de una imagen administrada de una máquina virtual generalizada en Azure](../virtual-machines/windows/capture-image-resource.md).

Si aún no tiene una suscripción a Azure, asegúrese de [crear una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar a seguir estas instrucciones.

## <a name="begin-the-host-pool-setup-process"></a>Inicio del proceso de configuración del grupo de hosts

Para empezar a crear el grupo de hosts:

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com/).

2. Escriba **Windows Virtual Desktop** en la barra de búsqueda y, luego, busque y seleccione **Windows Virtual Desktop** en Servicios.

3. En la página de información general de **Windows Virtual Desktop**, seleccione **Create a host pool** (Crear un grupo de hosts).

4. En la pestaña **Basics** (Aspectos básicos), seleccione la suscripción correcta en los detalles del proyecto.

5. Seleccione **Create new** (Crear nuevo) para crear un grupo de recursos o seleccione un grupo de recursos existente en el menú desplegable.

6. Escriba un nombre único para el grupo de hosts.

7. En el campo "Location" (Ubicación), en el menú desplegable, seleccione la región en la que quiere crear el grupo de hosts.

   La geografía de Azure asociada a las regiones seleccionadas es donde se almacenarán los metadatos de este grupo de hosts y sus objetos relacionados. Asegúrese de elegir las regiones dentro de la geografía en la que quiere que se almacenen los metadatos del servicio.

     > [!div class="mx-imgBorder"]
     > ![Captura de pantalla de Azure Portal que muestra el campo "Location" (Ubicación) con la ubicación East US (Este de EE. UU.) seleccionada. Junto al campo aparece texto que dice que los metadatos se almacenarán en el este de EE. UU.](media/portal-location-field.png)

8. En "Host pool type" (Tipo de grupo de hosts), seleccione si el grupo de hosts será **Personal** o **Pooled** (Agrupado).

    - Si elige **Personal**, seleccione **Automatic** (Automático) o **Direct** (Directo) en el campo Assignment Type (Tipo de asignación).

      > [!div class="mx-imgBorder"]
      > ![Captura de pantalla del menú desplegable del campo "Assignment Type" (Tipo de asignación). El usuario ha seleccionado "Automatic" (Automático).](media/assignment-type-field.png)

9. Si elige **Pooled** (Agrupado), escriba la siguiente información:

     - En **Max session limit** (Límite máximo de sesiones), escriba el número máximo de usuarios entre los que desea que se equilibre la carga en un solo host de sesión.
     - En **Load balancing algorithm** (Algoritmo de equilibrio de carga), elija equilibrio de carga en amplitud o equilibrio de carga en profundidad, según el patrón de uso.

       > [!div class="mx-imgBorder"]
       > ![Captura de pantalla del campo "Assignment Type" (Tipo de asignación) con "Pooled" (Agrupado) seleccionado. El usuario está desplazando el cursor sobre "Breadth-first" (Equilibrio de carga en amplitud) en el menú desplegable "Load Balancing" (Equilibrio de carga).](media/pooled-assignment-type.png)

10. Seleccione **Siguiente: VM details** (Detalles de VM).

11. Si ya ha creado las máquinas virtuales y quiere usarlas con el nuevo grupo de hosts, seleccione **No**. Si quiere crear máquinas virtuales y registrarlas en el nuevo grupo de hosts, seleccione **Yes** (Sí).

Ahora que ha completado la primera parte, vamos a pasar a la siguiente parte del proceso de configuración en el que se crea la máquina virtual.

## <a name="virtual-machine-details"></a>Detalles de máquina virtual

Ahora que hemos recorrido la primera parte, tendrá que configurar la máquina virtual.

Para configurar la máquina virtual en el proceso de configuración del grupo de hosts:

1. En "Resource Group" (Grupo de recursos), elija el grupo de recursos donde quiere crear las máquinas virtuales. Puede ser un grupo de recursos diferente al que usó para el grupo de hosts.

2. Elija la **región de la máquina virtual** donde quiere crear las máquinas virtuales. Puede ser igual o diferente a la región seleccionada para el grupo de hosts.

3. A continuación, elija el tamaño de la máquina virtual que quiere crear. Puede mantener el tamaño predeterminado tal cual o seleccionar **Change Size** (Cambiar tamaño) para cambiar el tamaño. Si selecciona **Change Size** (Cambiar tamaño), en la ventana que aparece, elija el tamaño de la máquina virtual adecuado para su carga de trabajo.

4. En "Number of VMs" (Número de máquinas virtuales), proporcione el número de máquinas virtuales que quiere crear para el grupo de hosts.

    >[!NOTE]
    >En el proceso de configuración se pueden crear hasta 400 máquinas virtuales al configurar el grupo de hosts y cada proceso de configuración de máquina virtual crea cuatro objetos en el grupo de recursos. Dado que en el proceso de creación no se comprueba la cuota de suscripción, asegúrese de que el número de máquinas virtuales que especifique se encuentre dentro de los límites de la máquina virtual y de la API de Azure en el grupo de recursos y la suscripción. Después de terminar de crear el grupo de hosts, puede agregar más máquinas virtuales.

5. A continuación, proporcione un **prefijo de nombre** para asignar un nombre a las máquinas virtuales que crea el proceso de configuración. El sufijo será `-` y los números comienzan a partir de 0.

6. A continuación, elija la imagen que debe usarse para crear la máquina virtual. Puede elegir entre **Gallery** (Galería) o **Storage Blob**.

    - Si elige **Gallery** (Galería), seleccione una de las imágenes recomendadas en el menú desplegable:

      - sesión múltiple de Windows 10 Enterprise, versión 1909 + Aplicaciones de Microsoft 365 para empresa, Gen 1
      - Sesión múltiple de Windows 10 Enterprise, versión 1909 - Gen 1
      - Windows Server 2019 Datacenter - Gen 1

     Si no ve la imagen que quiere, seleccione **Browse all images and disks** (Examinar todas las imágenes y discos), lo que le permite seleccionar otra imagen de la galería o una imagen proporcionada por Microsoft y otros anunciantes.

     > [!div class="mx-imgBorder"]
     > ![Captura de pantalla de Marketplace que muestra una lista de imágenes de Microsoft.](media/marketplace-images.png)

     También puede ir a **My Items** (Mis elementos) y elegir una imagen personalizada que ya haya cargado.

     > [!div class="mx-imgBorder"]
     > ![Captura de pantalla de la pestaña "My Items" (Mis elementos).](media/my-items.png)

    - Si elige **Storage Blob**, puede aprovechar su propia imagen creada mediante Hyper-V o en una máquina virtual de Azure. Todo lo que tiene que hacer es escribir la ubicación de la imagen en el blob de almacenamiento como un URI.

7. Elija qué tipo de discos del sistema operativo quiere que usen las máquinas virtuales: SSD estándar, SSD Premium o HDD estándar.

8. En "Network and security" (Red y seguridad), seleccione la red virtual y la subred en la que quiere colocar las máquinas virtuales creadas. Asegúrese de que la red virtual pueda conectarse al controlador de dominio, ya que tendrá que unir al dominio las máquinas virtuales que se encuentran dentro de la red virtual. Luego, seleccione si quiere o no una dirección IP pública para las máquinas virtuales. Se recomienda seleccionar **No**, ya que una dirección IP privada es más segura.

9. Seleccione el tipo de grupo de seguridad que desee: **Basic** (Básico), **Advanced** (Avanzado) o **None** (Ninguno).

    Si selecciona **Basic** (Básico), tendrá que decidir si quiere abrir algún puerto de entrada. Si selecciona **Yes** (Sí), elija de la lista de puertos estándar a los que permitir conexiones entrantes.

    >[!NOTE]
    >Para mayor seguridad, se recomienda no abrir puertos de entrada públicos.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la página del grupo de seguridad que muestra una lista de los puertos disponibles en un menú desplegable.](media/available-ports.png)

    Si elige **Advanced** (Avanzado), seleccione un grupo de seguridad de red existente que ya haya configurado.

10. Después, seleccione si quiere que las máquinas virtuales se unan a un dominio y a una unidad organizativa específicos. Si elige **Yes** (Yes), especifique el dominio al que quiere unirse. También puede agregar una unidad organizativa determinada en la que quiera que estén las máquinas virtuales.

11. En "Administrator account" (Cuenta de administrador), escriba las credenciales del administrador de dominio de Active Directory de la red virtual que ha seleccionado.

12. Seleccione **Workspace** (Área de trabajo).

Ahora, estamos preparados para iniciar la siguiente fase de configuración del grupo de hosts: registro del grupo de aplicaciones en un área de trabajo.

## <a name="workspace-information"></a>Información del área de trabajo

El proceso de configuración del grupo de hosts crea un grupo de aplicaciones de escritorio de forma predeterminada. Para que el grupo de hosts funcione según lo previsto, debe publicar este grupo de aplicaciones en usuarios o grupos de usuarios, y debe registrar el grupo de aplicaciones en un área de trabajo.

Para registrar el grupo de aplicaciones de escritorio en un área de trabajo:

1. Seleccione **Sí**.

   Si selecciona **No**, puede registrar el grupo de aplicaciones más adelante, pero se recomienda realizar el registro del área de trabajo tan pronto como pueda para que el grupo de hosts funcione correctamente.

2. A continuación, elija si quiere crear un área de trabajo o seleccionar entre las áreas de trabajo existentes. Solo las áreas de trabajo creadas en la misma ubicación que el grupo de hosts se pueden registrar en el grupo de aplicaciones.

3. Opcionalmente, puede seleccionar **Tags** (Etiquetas).

    Aquí puede agregar etiquetas para poder agrupar los objetos con metadatos y así facilitar las tareas para los administradores.

4. Seleccione **Revisar y crear** cuando haya terminado.

     >[!NOTE]
     >El proceso de validación "Review + create" (Revisar y crear) no comprueba si la contraseña cumple los estándares de seguridad o si la arquitectura es correcta, por lo que deberá confirmar si hay algún problema con cualquiera de estos aspectos.

5. Revise la información sobre la implementación para asegurarse de que todo es correcto. Seleccione **Crear** cuando haya terminado. Esta acción inicia el proceso de implementación, que crea los objetos siguientes:

     - El nuevo grupo de hosts.
     - Un grupo de aplicaciones de escritorio.
     - Un área de trabajo, si decidió crearla.
     - Si ha decidido registrar el grupo de aplicaciones de escritorio, el registro se completará.
     - Máquinas virtuales, si ha decidido crearlas, que se unen al dominio y se registran en el nuevo grupo de hosts.
     - Un vínculo de descarga de una plantilla de administración de recursos de Azure basada en la configuración.

Y ya está.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado un grupo de hosts, puede rellenarlo con programas de RemoteApp. Para más información sobre cómo administrar aplicaciones en Windows Virtual Desktop, avance al siguiente tutorial:

> [!div class="nextstepaction"]
> [Tutorial: Administración de grupos de aplicaciones](./manage-app-groups.md)
