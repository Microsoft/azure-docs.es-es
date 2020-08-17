---
title: Configuración de un laboratorio de SOLIDWORKS para ingeniería con Azure Lab Services | Microsoft Docs
description: Aprenda a configurar un laboratorio para cursos de ingeniería con SOLIDWORKS.
author: nicolela
ms.topic: article
ms.date: 06/26/2020
ms.author: nicolela
ms.openlocfilehash: 5511ad5a517bbd320ce3d66de90a8aec084c7e15
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87290724"
---
# <a name="set-up-a-lab-for-engineering-classes-using-solidworks"></a>Configuración de un laboratorio para clases de ingeniería con SOLIDWORKS

[SOLIDWORKS](https://www.solidworks.com/) proporciona un entorno de diseño asistido por PC (CAD) 3D para el modelado de objetos sólidos y se usa en distintos tipos de campos de ingeniería.  Con SOLIDWORKS, los ingenieros pueden crear, visualizar, simular y documentar sus diseños fácilmente.

Una opción de obtención de licencias usada habitualmente por las universidades es la licencia de red de SOLIDWORKS.   Con esta opción, los usuarios comparten un grupo de licencias administradas por un servidor de licencias.  Este tipo de licencia a veces se denomina licencia "flotante" porque solo es necesario tener suficientes licencias para el número de usuarios simultáneos.  Cuando un usuario termina de usar SOLIDWORKS, su licencia vuelve al grupo de licencias administrado centralmente para que otro usuario pueda reutilizarla.

En este artículo, le mostraremos cómo configurar una clase que usa SOLIDWORKS 2019 y una licencia de red.

## <a name="license-server"></a>Servidor de licencias

La licencia de red de SOLIDWORKS requiere que el administrador de licencias de SOLIDWORKS esté instalado y activado en el servidor de licencias.  Este servidor de licencias se suele encontrar en la red local o en una red privada dentro de Azure.  Para más información sobre cómo configurar el administrador de licencias de SolidNetWork en el servidor, consulte [Instalación y activación de un administrador de licencias](https://help.solidworks.com/2019/English/Installation/install_guide/t_installing_snl_lic_mgr.htm) en la guía de instalación de SOLIDWORKS.  Al configurar esta opción, recuerde el **número de puerto** y el [**número de serie**](https://help.solidworks.com/2019/english/installation/install_guide/r_hid_state_serial_number.htm) que se usan, ya que se necesitarán en pasos posteriores.

Una vez configurado el servidor de licencias, tendrá que emparejar la [red virtual (VNet)](https://docs.microsoft.com/azure/lab-services/classroom-labs/how-to-connect-peer-virtual-network) con la [cuenta de laboratorio](https://docs.microsoft.com/azure/lab-services/classroom-labs/tutorial-setup-lab-account).  El emparejamiento de redes debe realizarse antes de crear el laboratorio para que las máquinas virtuales de este puedan acceder al servidor de licencias y viceversa.

> [!NOTE]
> Debe comprobar que estén abiertos los puertos adecuados en los firewalls para permitir la comunicación entre las máquinas virtuales del laboratorio y el servidor de licencias.  Por ejemplo, consulte las instrucciones sobre la [Modificación de puertos en la computadora del Administrador de licencias para el Firewall de Windows](http://help.solidworks.com/2019/english/installation/install_guide/t_mod_ports_on_lic_mgr_for_firewall.htm) que muestran cómo agregar reglas de entrada y de salida al firewall del servidor de licencias.  Es posible que también tenga que abrir puertos para las máquinas virtuales del laboratorio.  Siga los pasos del artículo sobre la [configuración del firewall para Azure laboratorios](https://docs.microsoft.com/azure/lab-services/classroom-labs/how-to-configure-firewall-settings) para más información, incluido cómo obtener la dirección IP pública del laboratorio.

## <a name="lab-configuration"></a>Configuración del laboratorio

Para configurar este laboratorio, para empezar, necesita una suscripción a Azure y una cuenta de laboratorio. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar. Una vez que obtenga una suscripción a Azure, puede crear una nueva cuenta de laboratorio en Azure Lab Services. Para más información sobre la creación de una nueva cuenta de laboratorio, consulte el tutorial sobre la [configuración de una cuenta de laboratorio](https://docs.microsoft.com/azure/lab-services/classroom-labs/tutorial-setup-lab-account). También puede usar una cuenta de laboratorio existente.

### <a name="lab-account-settings"></a>Configuración de la cuenta de laboratorio

Habilite la configuración que se describe en la tabla siguiente para la cuenta de laboratorio. Para obtener más información sobre cómo habilitar imágenes de marketplace, consulte el artículo sobre la [especificación de las imágenes de Marketplace disponibles para los creadores de laboratorios](https://docs.microsoft.com/azure/lab-services/classroom-labs/specify-marketplace-images).

| Configuración de la cuenta de laboratorio | Instructions |
| ------------------- | ------------ |
|Imagen de Marketplace| Habilite la imagen de Windows 10 Pro para su uso en la cuenta de laboratorio.|

> [!NOTE]
> Además de Windows 10, SOLIDWORKS admite otras versiones de Windows.  Consulte [Requisitos del sistema de SOLIDWORKS](https://www.solidworks.com/sw/support/SystemRequirements.html) para más información.

### <a name="lab-settings"></a>Configuración del laboratorio

Use las opciones de la tabla siguiente al configurar un laboratorio educativo. Para más información sobre cómo crear un laboratorio educativo, consulte el tutorial de configuración de un laboratorio educativo.

| Configuración del laboratorio | Valor/instrucciones |
| ------------ | ------------------ |
|Tamaño de la máquina virtual| **GPU pequeña (visualización)** .  Esta máquina virtual es más adecuada para visualización remota, streaming, juegos y codificación mediante plataformas como OpenGL y DirectX.|  
|Imagen de máquina virtual| Windows 10 Pro|

> [!NOTE]
> El tamaño de máquina virtual de **GPU pequeña (visualización)** está configurado para permitir una experiencia de gráficos de alto rendimiento.  Para obtener más información sobre este tamaño de máquina virtual, consulte el artículo sobre la [configuración de un laboratorio con GPU](./how-to-setup-lab-gpu.md).

> [!WARNING]
> No olvide [emparejar la red virtual](https://www.mathworks.com/support/requirements/matlab-system-requirements.html) de la cuenta de laboratorio a la red virtual del servidor de licencias **antes** de crear el laboratorio.

## <a name="template-virtual-machine-configuration"></a>Configuración de plantilla de máquina virtual

En los pasos de esta sección se muestra cómo configurar la plantilla de máquina virtual descargando los archivos de instalación de SOLIDWORKS e instalando el software cliente:

1. Inicie la plantilla de máquina virtual y conéctese a la máquina con RDP.

1. Descargue los archivos de instalación del software cliente de SOLIDWORKS. Tiene dos opciones para la descarga:
   - Descarga desde el [portal del cliente de SOLIDWORKS](https://login.solidworks.com/nidp/idff/sso?id=cpenglish&sid=1&option=credential&sid=1&target=https%3A%2F%2Fcustomerportal.solidworks.com%2F).
   - Descarga desde un directorio de un servidor.  Si usa esta opción, debe asegurarse de que el servidor sea accesible desde la plantilla de máquina virtual.  Por ejemplo, este servidor puede encontrarse en la misma red virtual que está emparejada con su cuenta de laboratorio.
  
    Para más información, consulte [Instalación en computadoras individuales](http://help.solidworks.com/2019/english/Installation/install_guide/c_installing_on_individual_computers.htm?id=fc149e8a968a422a89e2a943265758d3#Pg0) en la guía de instalación de SOLIDWORKS.

1. Una vez que se hayan descargado los archivos de instalación, instale el software cliente con el administrador de instalación de SOLIDWORKS. Vea los detalles en [Instalación de un cliente de licencia](http://help.solidworks.com/2019/english/installation/install_guide/t_installing_snl_license_client.htm) en la guía de instalación de SOLIDWORKS.

    > [!NOTE]
    > En el cuadro de diálogo **Agregar servidor**, se le pedirá el **número de puerto** usado para el servidor de licencias y el nombre o la dirección IP del servidor de licencias.

## <a name="cost"></a>Coste

Vamos a cubrir una posible estimación de costos para esta clase. Esta estimación no incluye el costo de ejecutar el servidor de licencias. Usaremos una clase de 25 alumnos. Hay 20 horas de tiempo de clase programado. Además, cada alumno obtiene una cuota de 10 horas para deberes o tareas fuera del tiempo de clase programado. El tamaño de la máquina virtual que hemos elegido es el de **GPU pequeña (visualización)** , que tiene 160 unidades de laboratorio.

25 alumnos \* (20 horas programadas + 10 horas de cuota) \* 160 unidades de laboratorio * 0,01 USD por hora = 1200,00 USD

>[!IMPORTANT]
> La estimación de costos solo se utiliza con fines de ejemplo.  Para conocer los detalles actuales sobre los precios, consulte [Precios de Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).  

## <a name="next-steps"></a>Pasos siguientes

Los pasos siguientes son comunes a la configuración de cualquier laboratorio.

- [Creación y administración de plantillas](how-to-create-manage-template.md)
- [Incorporación de usuarios](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Establecimiento de la cuota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Establecimiento de la programación](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
- [Vínculos de registro por correo electrónico para los alumnos](how-to-configure-student-usage.md#send-invitations-to-users)
