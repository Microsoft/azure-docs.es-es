---
title: Recomendaciones de seguridad para imágenes de Azure Marketplace | Microsoft Docs
description: En este artículo se proporcionan recomendaciones para las imágenes incluidas en Marketplace.
services: security
documentationcenter: na
author: terrylanfear
manager: barbkess
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.date: 01/11/2019
ms.author: terrylan
ms.openlocfilehash: 7c317a0b4fea0c981b227bace00c1b8924fd582c
ms.sourcegitcommit: c52e50ea04dfb8d4da0e18735477b80cafccc2cf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/08/2020
ms.locfileid: "89536389"
---
# <a name="security-recommendations-for-azure-marketplace-images"></a>Recomendaciones de seguridad para imágenes de Azure Marketplace

La imagen debe cumplir estas recomendaciones de configuración de seguridad. Esto le ayuda a mantener un alto nivel de seguridad para las imágenes de la solución del asociado de Azure Marketplace.

Ejecute siempre una detección de vulnerabilidades de seguridad en la imagen antes de enviarla. Si detecta una vulnerabilidad de seguridad en su propia imagen publicada, debe informar a los clientes de forma oportuna de la vulnerabilidad y de cómo corregirlo.

## <a name="open-source-based-images"></a>Abrir imágenes basadas en origen

| Category | Comprobar |
| -------- | ----- |
| Seguridad                                                     | Instalar todas las revisiones de seguridad más recientes de la distribución de Linux.                                                                                                                                                                                                              |
| Seguridad                                                     | Seguir las directrices del sector para proteger la imagen de la máquina virtual para la distribución de Linux específica.                                                                                                                                                                                     |
| Seguridad                                                     | Limitar la superficie expuesta a ataques al mantener una superficie mínima con los roles, las características, los servicios y los puertos de red de Windows Server necesarios.                                                                                                                                               |
| Seguridad                                                     | Examinar el código fuente y la imagen de máquina virtual resultante en busca de malware.                                                                                                                                                                                                                                   |
| Seguridad                                                     | La imagen de disco duro virtual solo incluye las cuentas bloqueadas necesarias, que no tienen las contraseñas predeterminadas que permitirían un inicio de sesión interactivo; no hay puertas traseras.                                                                                                                                           |
| Seguridad                                                     | Deshabilitar las reglas de firewall, a menos que la aplicación se base funcionalmente en ellas, como un dispositivo de firewall.                                                                                                                                                                             |
| Seguridad                                                     | Quitar toda la información confidencial de la imagen de disco duro virtual, como las claves SSH de prueba, los archivo de hosts conocidos, los archivos de registro y los certificados innecesarios.                                                                                                                                       |
| Seguridad                                                     | Evitar usar LVM.                                                                                                                                                                                                                                            |
| Seguridad                                                     | Incluir las versiones más recientes de las bibliotecas necesarias: </br> - OpenSSL v1.0 o posterior. </br> - Python 2.5 o posterior (se recomienda Python 2.6+). </br> - Paquete pyans1 de Python, si no está ya instalado. </br> - d.OpenSSL v 1.0 o posterior.                                                                |
| Seguridad                                                     | Borrar las entradas del historial de Bash/Shell.                                                                                                                                                                                                                                             |
| Redes                                                   | Incluir el servidor SSH de forma predeterminada. Establezca la conexión persistente de SSH en la configuración sshd con la siguiente opción: ClientAliveInterval 180.                                                                                                                                                        |
| Redes                                                   | Quitar cualquier configuración de red personalizada de la imagen. Elimine resolv.conf: `rm /etc/resolv.conf`.                                                                                                                                                                                |
| Implementación                                                   | Instalar la versión más reciente del agente Linux de Azure.</br> -Instalar con el paquete RPM o Deb.  </br> -  También puede usar el proceso de instalación manual, pero se recomienda y se prefiere que se usen los paquetes del instalador. </br> - Si instala el agente manualmente desde el repositorio de GitHub, copie primero el archivo `waagent` en `/usr/sbin` y ejecute (como raíz): </br>`# chmod 755 /usr/sbin/waagent` </br>`# /usr/sbin/waagent -install` </br>El archivo de configuración del agente se coloca en `/etc/waagent.conf`. |
| Implementación                                                   | Garantizar que el soporte técnico de Azure puede proporcionar a nuestros asociados la salida de la consola de serie cuando sea necesario y proporcionar el tiempo de espera adecuado para el montaje del disco del sistema operativo desde el almacenamiento en la nube. Agregue los parámetros siguientes a la línea de arranque de kernel de la imagen: `console=ttyS0 earlyprintk=ttyS0 rootdelay=300`. |
| Implementación                                                   | No hay ninguna partición de intercambio en el disco del SO. El agente de Linux puede solicitar el intercambio para la creación en el disco del recurso local.         |
| Implementación                                                   | Crear una sola partición raíz para el disco de SO.      |
| Implementación                                                   | Solo el sistema operativo de 64 bits.                                                                                                                                                                                                                                                          |

## <a name="windows-server-based-images"></a>Imágenes basadas en Windows Server

| Category | Comprobar |
|--------- | ----- |
| Seguridad                                                         | Usar una imagen base de sistema operativo segura. El disco duro virtual que se usa para el origen de cualquier imagen basada en Windows Server debe pertenecer a las imágenes del sistema operativo de Windows Server que se proporcionan a través de Microsoft Azure. |
| Seguridad                                                         | Instalar todas las actualizaciones de seguridad.                                                                                                                                     |
| Seguridad                                                         | Las aplicaciones no deben depender de nombres de usuario con permisos restringidos, como administrador o raíz.                                                                |
| Seguridad                                                         | Habilitar el cifrado de unidad BitLocker tanto para las unidades de disco duro del sistema operativo como para las unidades de disco duro de datos.                                                             |
| Seguridad                                                         | Limitar la superficie expuesta a ataques al mantener una superficie mínima habilitando solo los roles, las características, los servicios y los puertos de red de Windows Server necesarios.                         |
| Seguridad                                                         | Examinar el código fuente y la imagen de máquina virtual resultante en busca de malware.                                                                                                                     |
| Seguridad                                                         | Establecer la actualización de seguridad de las imágenes de Windows Server para que se actualicen automáticamente.                                                                                                                |
| Seguridad                                                         | La imagen de disco duro virtual solo incluye las cuentas bloqueadas necesarias, que no tienen las contraseñas predeterminadas que permitirían un inicio de sesión interactivo; no hay puertas traseras.                             |
| Seguridad                                                         | Deshabilitar las reglas de firewall, a menos que la aplicación se base funcionalmente en ellas, como un dispositivo de firewall.                                                               |
| Seguridad                                                         | Quitar toda la información confidencial de la imagen de disco duro virtual, incluidos los archivos de hosts, los archivos de registro y los certificados innecesarios.                                              |
| Implementación                                                       | Solo el sistema operativo de 64 bits.                            |

Incluso si su organización no tiene imágenes en Azure Marketplace, considere la posibilidad de comprobar las configuraciones de imagen de Windows y Linux con estas recomendaciones.

