---
title: 'Sistemas operativos y motores de contenedor compatibles: Azure IoT Edge'
description: Obtenga información sobre qué sistemas operativos pueden ejecutar el demonio y el entorno de ejecución de Azure IoT Edge y los motores de contenedor admitidos para los dispositivos de producción
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/17/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: ccfd85c5976642d30b3f21d2d2afeb868ba49bf9
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87924621"
---
# <a name="azure-iot-edge-supported-systems"></a>Sistemas compatibles con Azure IoT Edge

En este artículo se proporcionan detalles sobre qué sistemas y componentes son compatibles con IoT Edge, ya sea oficialmente o en versión preliminar.

Si tiene problemas al usar el servicio Azure IoT Edge, hay varias maneras de buscar soporte técnico. Pruebe uno de los siguientes canales para obtener soporte técnico:

**Notificación de errores**: la mayor parte del desarrollo que lleva el producto de Azure IoT Edge se produce en el proyecto de código abierto de IoT Edge. Los errores se pueden notificar en la [página de problemas](https://github.com/azure/iotedge/issues) del proyecto. Las correcciones pasan pronto del proyecto a las actualizaciones de producto.

**Equipo de soporte técnico al cliente de Microsoft**: los usuarios que dispongan de un [plan de soporte técnico](https://azure.microsoft.com/support/plans/) pueden ponerse en contacto con este equipo creando una incidencia de soporte técnico directamente en [Azure Portal](https://ms.portal.azure.com/signin/index/?feature.settingsportalinstance=mpac).

**Solicitudes de características**: el producto Azure IoT Edge realiza un seguimiento de las solicitudes de características a través de la [página Voz del usuario](https://feedback.azure.com/forums/907045-azure-iot-edge).

## <a name="container-engines"></a>Motores de contenedor

Los módulos de Azure IoT Edge se implementan como contenedores, por lo que IoT Edge necesita un motor de contenedores para iniciarlos. Microsoft ofrece un motor de contenedor, moby-engine, para satisfacer este requisito. Este motor de contenedores se basa en el proyecto de código abierto de Moby. Docker CE y Docker EE son otros motores de contenedores conocidos. También se basan en el proyecto de código abierto de Moby y son compatibles con Azure IoT Edge. Microsoft proporciona el mejor soporte técnico posible para los sistemas que usan esos motores de contenedores; sin embargo, Microsoft no puede proporcionar soluciones para los problemas de ellos. Por esta razón, Microsoft recomienda el uso de moby-engine en sistemas de producción.

<br>
<center>

![Motor de Moby como entorno de ejecución de contenedor](./media/support/only-moby-for-production.png)
</center>

## <a name="operating-systems"></a>Sistemas operativos

Azure IoT Edge se ejecuta en la mayoría de los sistemas operativos que pueden ejecutar contenedores. No obstante, la compatibilidad no es igual en todos. Los sistemas operativos se agrupan en niveles que representan el nivel de compatibilidad que los usuarios pueden esperar.

* Se admiten los sistemas de nivel 1. Para los sistemas de nivel 1, Microsoft:
  * tiene este sistema operativo en pruebas automatizadas
  * proporciona paquetes de instalación para ellos
* Los sistemas de nivel 2 son compatibles con Azure IoT Edge y pueden usarse con relativa facilidad. Para los sistemas de nivel 2:
  * Microsoft ha realizado pruebas informales en las plataformas o conoce a un asociado que ha ejecutado correctamente Azure IoT Edge en la plataforma
  * Los paquetes de información de otras plataformas pueden funcionar en estas plataformas

La familia del sistema operativo host siempre debe coincidir con la familia del sistema operativo invitado que se utilice en el contenedor de un módulo. En otras palabras, solo se pueden usar contenedores de Linux en Linux y contenedores de Windows en Windows. En el caso de Windows, solo se pueden usar contenedores aislados de los procesos; no se admiten los contenedores aislados de Hyper-V.  

<br>
<center>

![El sistema operativo host coincide con el sistema operativo invitado](./media/support/edge-on-device.png)
</center>

### <a name="tier-1"></a>Nivel 1

Los sistemas que se enumeran en la tabla siguiente son compatibles con Microsoft, tanto si están disponibles con carácter general como en versión preliminar pública, y se prueban con cada versión nueva. 

| Sistema operativo | AMD64 | ARM32v7 | ARM64 |
| ---------------- | ----- | ------- | ----- |
| Raspbian Stretch |  | ![Raspbian Stretch + ARM32v7](./media/tutorial-c-module/green-check.png) |  |
| [Ubuntu Server 16.04](https://wiki.ubuntu.com/XenialXerus/ReleaseNotes) | ![Ubuntu Server 16.04 + AMD64](./media/tutorial-c-module/green-check.png) |  | Versión preliminar pública  |
| [Ubuntu Server 18.04](https://wiki.ubuntu.com/BionicBeaver/ReleaseNotes) | ![Ubuntu Server 18.04 + AMD64](./media/tutorial-c-module/green-check.png) |  | Versión preliminar pública |
| [Windows 10 IoT Core](https://docs.microsoft.com/windows/iot-core/windows-iot-core), compilación 17763 | ![Windows IoT Core + AMD64](./media/tutorial-c-module/green-check.png) |  |  |
| [Windows 10 IoT Enterprise](https://docs.microsoft.com/windows/iot-core/windows-iot-enterprise), compilación 17763 | ![Windows 10 IoT Enterprise + AMD64](./media/tutorial-c-module/green-check.png) |  |  |
| [Windows Server 2019](https://docs.microsoft.com/windows-server/get-started-19/rel-notes-19), compilación 17763 | ![Windows Server 2019 + AMD64](./media/tutorial-c-module/green-check.png) |  |  |
| [Windows Server IoT 2019](https://docs.microsoft.com/windows/iot-core/windows-server), compilación 17763 | ![Windows Server IoT 2019 + AMD64](./media/tutorial-c-module/green-check.png) |  |  |

Los sistemas operativos de Windows anteriores son obligatorios en los dispositivos que ejecutan contenedores de Windows en Windows, que es la única configuración compatible para producción. Los paquetes de instalación de Azure IoT Edge para Windows permiten usar contenedores de Linux en Windows, aunque solo en los entornos de desarrollo y pruebas. Para obtener más información, consulte [Uso de IoT Edge en Windows para ejecutar contenedores de Linux](how-to-install-iot-edge-windows-with-linux.md).

### <a name="tier-2"></a>Nivel 2

Los sistemas que se enumeran en la tabla siguiente se consideran compatibles con Azure IoT Edge, pero Microsoft no los prueba ni mantiene activamente.

| Sistema operativo | AMD64 | ARM32v7 | ARM64 |
| ---------------- | ----- | ------- | ----- |
| [CentOS 7.5](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7.1804) | ![CentOS + AMD64](./media/tutorial-c-module/green-check.png) | ![CentOS + ARM32v7](./media/tutorial-c-module/green-check.png) | ![CentOS + ARM64](./media/tutorial-c-module/green-check.png) |
| [Debian 8](https://www.debian.org/releases/jessie/) | ![Debian 8 + AMD64](./media/tutorial-c-module/green-check.png) | ![Debian 8 + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Debian 8 + ARM64](./media/tutorial-c-module/green-check.png) |
| [Debian 9](https://www.debian.org/releases/stretch/) | ![Debian 9 + AMD64](./media/tutorial-c-module/green-check.png) | ![Debian 9 + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Debian 9 + ARM64](./media/tutorial-c-module/green-check.png) |
| [Debian 10](https://www.debian.org/releases/buster/) <sup>1</sup> | ![Debian 10 + AMD64](./media/tutorial-c-module/green-check.png) | ![Debian 10 + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Debian 10 + ARM64](./media/tutorial-c-module/green-check.png) |
| [Mentor Embedded Linux Flex OS](https://www.mentor.com/embedded-software/linux/mel-flex-os/) | ![Mentor Embedded Linux Flex OS + AMD64](./media/tutorial-c-module/green-check.png) | ![Mentor Embedded Linux Flex OS + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Mentor Embedded Linux Flex OS + ARM64](./media/tutorial-c-module/green-check.png) |
| [Mentor Embedded Linux Omni OS](https://www.mentor.com/embedded-software/linux/mel-omni-os/) | ![Mentor Embedded Linux Omni OS + AMD64](./media/tutorial-c-module/green-check.png) |  | ![Mentor Embedded Linux Omni OS + ARM64](./media/tutorial-c-module/green-check.png) |
| [RHEL 7.5](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/7.5_release_notes/index) | ![RHEL 7.5 + AMD64](./media/tutorial-c-module/green-check.png) | ![RHEL 7.5 + ARM32v7](./media/tutorial-c-module/green-check.png) | ![RHEL 7.5 + ARM64](./media/tutorial-c-module/green-check.png) |
| [Ubuntu 16.04](https://wiki.ubuntu.com/XenialXerus/ReleaseNotes) | ![Ubuntu 16.04 + AMD64](./media/tutorial-c-module/green-check.png) | ![Ubuntu 16.04 + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Ubuntu 16.04 + ARM64](./media/tutorial-c-module/green-check.png) |
| [Ubuntu 18.04](https://wiki.ubuntu.com/BionicBeaver/ReleaseNotes) | ![Ubuntu 18.04 + AMD64](./media/tutorial-c-module/green-check.png) | ![Ubuntu 18.04 + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Ubuntu 18.04 + ARM64](./media/tutorial-c-module/green-check.png) |
| [Wind River 8](https://docs.windriver.com/category/os-wind_river_linux) | ![Wind River 8 + AMD64](./media/tutorial-c-module/green-check.png) |  |  |
| [Yocto](https://www.yoctoproject.org/) | ![Yocto + AMD64](./media/tutorial-c-module/green-check.png) | ![Yocto + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Yocto + ARM64](./media/tutorial-c-module/green-check.png) |
| Raspbian Buster <sup>1</sup> |  | ![Raspbian Buster + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Raspbian Buster + ARM64](./media/tutorial-c-module/green-check.png) |
| [Ubuntu 20.04 <sup>2</sup>](https://wiki.ubuntu.com/FocalFossa/ReleaseNotes) | ![Ubuntu 20.04 + AMD64](./media/tutorial-c-module/green-check.png) | ![Ubuntu 20.04 + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Ubuntu 20.04 + ARM64](./media/tutorial-c-module/green-check.png) |

<sup>1</sup> Los sistemas Debian 10, incluido Raspian Buster, usan una versión de OpenSSL que IoT Edge no admite. Use el siguiente comando para instalar una versión anterior antes de instalar IoT Edge:

```bash
sudo apt-get install libssl1.0.2
```

<sup>2</sup> Los paquetes de Debian 9 del [repositorio de versiones de Azure IoT Edge](https://github.com/Azure/azure-iotedge/releases) deberían funcionar de serie con Ubuntu 20.04.

## <a name="releases"></a>Lanzamientos

Los recursos de versión y las notas de la versión de IoT Edge están disponibles en la página de [versiones de azure-iotedge](https://github.com/Azure/azure-iotedge/releases). En esta sección se refleja la información de las notas de la versión para ayudarle a visualizar los componentes de cada versión más fácilmente.

Los componentes de IoT Edge se pueden instalar o actualizar individualmente y son compatibles con componentes de versiones anteriores. En la tabla siguiente se enumeran los componentes incluidos en cada versión:

| Release | Demonio de seguridad | Centro de Microsoft Edge<br>Agente de Edge | Libiothsm | Moby |
|--|--|--|--|--|
| **1.0.9** | 1.0.9.4<br>1.0.9.3<br>1.0.9.2<br>1.0.9.1<br>1.0.9 | 1.0.9.4<br>1.0.9.3<br>1.0.9.2<br>1.0.9.1<br>1.0.9 | 1.0.9.4<br>1.0.9.3<br>1.0.9.2<br>1.0.9.1<br>1.0.9 |  |
| **1.0.8** | 1.0.8 | 1.0.8.5<br>1.0.8.4<br>1.0.8.3<br>1.0.8.2<br>1.0.8.1<br>1.0.8 | 1.0.8 | 3.0.6 |
| **1.0.7** | 1.0.7.1<br>1.0.7 | 1.0.7.1<br>1.0.7 | 1.0.7.1<br>1.0.7 | 3.0.5<br>3.0.4 (ARMv7hl, CentOS) |
| **1.0.6** | 1.0.6.1<br>1.0.6 | 1.0.6.1<br>1.0.6 | 1.0.6.1<br>1.0.6 |  |
| **1.0.5** | 1.0.5 | 1.0.5 | 1.0.5 | 3.0.2 |

IoT Edge usa el SDK de Microsoft.Azure.Device.Client. Para más información, consulte el [repositorio de GitHub de SDK de para Azure IoT en C#](https://github.com/Azure/azure-iot-sdk-csharp) o el [contenido de referencia del SDK de Azure para .NET](https://docs.microsoft.com/dotnet/api/overview/azure/iot/client?view=azure-dotnet). En la siguiente lista se muestra la versión del SDK de cliente con el que se prueba cada versión:

* **IoT Edge 1.0.9**: SDK de cliente 1.21.1
* **IoT Edge 1.0.8**: SDK de cliente 1.20.3
* **IoT Edge 1.0.7**: SDK de cliente 1.20.1
* **IoT Edge 1.0.6**: SDK de cliente 1.17.1
* **IoT Edge 1.0.5**: SDK de cliente 1.17.1

## <a name="virtual-machines"></a>Virtual Machines

Azure IoT Edge puede ejecutarse en máquinas virtuales. El uso de una máquina virtual como dispositivo de IoT Edge es una práctica habitual cuando los clientes desean mejorar la infraestructura existente con inteligencia perimetral. La familia del sistema operativo de la máquina virtual host debe coincidir con la familia del sistema operativo invitado que se utiliza en el contenedor de un módulo. Este requisito también se aplica cuando Azure IoT Edge se ejecuta directamente en un dispositivo. Azure IoT Edge es independiente de la tecnología de virtualización subyacente y funciona en máquinas virtuales que incorporan plataformas como Hyper-V y vSphere.

<br>
<center>

![Azure IoT Edge en una máquina virtual](./media/support/edge-on-vm.png)
</center>

## <a name="minimum-system-requirements"></a>Requisitos mínimos del sistema

Azure IoT Edge funciona perfectamente en dispositivos tan pequeños como Raspberry Pi3 y en hardware de nivel de servidor. La elección del hardware adecuado para el escenario dependerá de las cargas de trabajo que se deseen ejecutar. Tomar la decisión final sobre el dispositivo que se va a utilizar puede resultar complicado; sin embargo, puede empezar creando un prototipo de una solución en portátiles o equipos de escritorio tradicionales.

La experiencia que adquiera mientras crea ese prototipo le ayudará a realizar la selección final del dispositivo. Debe considerar las siguientes cuestiones:

* ¿Cuántos módulos hay en la carga de trabajo?
* ¿Cuántas capas comparten los contenedores de los módulos?
* ¿En qué idioma se escriben los módulos?
* ¿Cuántos datos procesarán los módulos?
* ¿Necesitan los módulos algún hardware especializado para acelerar sus cargas de trabajo?
* ¿Cuáles son las características del rendimiento deseado de la solución?
* ¿Cuál es su presupuesto para hardware?
