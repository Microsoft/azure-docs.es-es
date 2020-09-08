---
title: Patrón de aplicación web multiinquilino | Microsoft Docs
description: Encuentre información general de arquitectura y patrones de diseño que describan cómo implementar una aplicación web multiempresa en Azure.
services: ''
documentationcenter: .net
author: wadepickett
manager: wpickett
editor: ''
ms.assetid: 4f0281d2-1555-42b0-a99d-1222fade0b0f
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 06/05/2015
ms.author: wpickett
ms.custom: devx-track-dotnet
ms.openlocfilehash: ad77278d593e4b2165708e0c9110a9ffff19a255
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89017709"
---
# <a name="multitenant-applications-in-azure"></a>Aplicaciones multiempresa en Azure
Una aplicación multiinquilino es un recurso compartido que permite a los "usuarios en inquilinos independientes" ver la aplicación como si fuera propia. Un escenario típico de una aplicación multiinquilino es aquel en el que todos los usuarios de la aplicación de inquilinos diferentes pueden querer personalizar la experiencia de usuario pero, por otra parte, tienen los mismos requisitos empresariales básicos. Office 365, Outlook.com y visualstudio.com son ejemplos de grandes aplicaciones multiempresa.

Desde la perspectiva del proveedor de una aplicación, los beneficios de la arquitectura multiempresa se relacionan principalmente con la eficacia operativa y de costes. Una versión de la aplicación puede satisfacer las necesidades de muchos inquilinos/clientes, de manera que se permita la consolidación de las tareas de administración del sistema, como la supervisión, el ajuste del rendimiento, el mantenimiento del software y las copias de seguridad de los datos.

A continuación se ofrece una lista de los objetivos y los requisitos más importantes desde la perspectiva de un proveedor.

* **Aprovisionamiento**: debe tener la capacidad de aprovisionar nuevos inquilinos para la aplicación.  En aplicaciones multiempresa con un gran número de inquilinos, suele ser necesario automatizar este proceso mediante la habilitación del aprovisionamiento de autoservicio.
* **Mantenimiento**: debe tener la capacidad de actualizar la aplicación y ejecutar otras tareas de mantenimiento mientras varios inquilinos la utilizan.
* **Supervisión**: debe tener la capacidad de supervisar la aplicación en todo momento para identificar todos los problemas y solucionarlos. Esto comprende supervisar el uso que cada inquilino hace de la aplicación.

Una aplicación multiempresa correctamente implementada ofrece los siguientes beneficios a los usuarios.

* **Aislamiento**: las actividades de los inquilinos a nivel individual no afectan al uso que otros inquilinos hacen de la aplicación. Los inquilinos no pueden tener acceso a los datos de los demás. Ante el inquilino, parecerán tener uso exclusivo de la aplicación.
* **Disponibilidad**: los inquilinos individuales desean que la aplicación esté constantemente disponible, quizá con garantías definidas en un contrato de nivel de servicio. De nuevo, las actividades de otros inquilinos no deben afectar a la disponibilidad de la aplicación.
* **Escalabilidad**: la aplicación se escala para satisfacer la demanda de cada uno de los inquilinos. La presencia y las acciones de otros inquilinos no deben afectar al rendimiento de la aplicación.
* **Costos**: los costos son más bajos en comparación con la ejecución de una aplicación dedicada de un solo inquilino, ya que la arquitectura multiinquilino permite compartir recursos.
* **Personalización**. la capacidad de personalizar la aplicación para un inquilino individual de varias formas, como agregar o eliminar características, cambiar el color y los logotipos o incluso agregar su propio código o script.

En resumen, si bien hay muchas consideraciones que debe tener en cuenta para ofrecer un servicio de alta escalabilidad, también hay una serie de objetivos y requisitos comunes a muchas aplicaciones multiempresa. Algunos pueden no resultar pertinentes en escenarios específicos y la importancia de los objetivos y requisitos individuales variará en cada escenario. Como proveedor de la aplicación multiempresa, también tendrá objetivos y requisitos como satisfacer los requisitos y los objetivos del inquilino, rentabilidad, facturación, varios niveles de servicio, aprovisionamiento, mantenimiento, supervisión y automatización.

Para obtener más información acerca de las consideraciones de diseño adicionales de una aplicación multiempresa, consulte [Hospedaje de una aplicación multiempresa en Azure][Hosting a Multi-Tenant Application on Azure]. Para más información sobre los patrones comunes de la arquitectura de datos de aplicaciones de base de datos de software como servicio (SaaS) multiinquilino, consulte [Modelos de diseño para las aplicaciones SaaS multiinquilino y Azure SQL Database](sql-database/sql-database-design-patterns-multi-tenancy-saas-applications.md). 

Azure ofrece muchas características que le permiten solucionar los principales problemas detectados al diseñar un sistema multiempresa.

**Aislamiento**

* Segmentación de inquilinos de sitios web mediante encabezados host con o sin comunicación de TLS
* Segmentar inquilinos de sitio web según parámetros de consulta
* Servicios web en roles de trabajo
  * Los roles de trabajo que suelen procesar datos en el back-end de una aplicación.
  * Roles web que suelen actuar como el front-end de las aplicaciones

**Storage**

Administración de datos como los servicios de Azure SQL Database o Azure Storage, por ejemplo, Table service, que ofrece servicios para el almacenamiento de grandes volúmenes de datos no estructurados, y Blob service, que ofrece servicios para almacenar grandes volúmenes de texto no estructurado o datos binarios, como vídeo, audio e imágenes.

* Proteger los datos multiinquilino en SQL Database para inicios de sesión de SQL Server por inquilino.
* Con el uso de las Tablas de Azure para recursos de la aplicación mediante la definición de una política de acceso a nivel de contenedor, tiene la posibilidad de ajustar permisos sin tener que emitir la nueva URL para los recursos protegidos con firmas de acceso compartido.
* Las colas de Azure para recursos de la aplicación se suelen usar para dirigir el procesamiento en nombre de los inquilinos, pero también se pueden usar para distribuir el trabajo necesario para el aprovisionamiento o la administración.
* Colas de Service Bus para recursos de aplicaciones que insertan el trabajo en un servicio compartido, donde puede usar una única cola en la que cada remitente inquilino solo tiene permisos (según se deriva de las notificaciones emitidas desde ACS) para insertar en dicha cola, mientras que solo los receptores del servicio tienen permiso para extraer de la cola los datos procedentes de varios inquilinos.

**Servicios de conexión y seguridad**

* Azure Service Bus, una infraestructura de mensajería que se ubica entre las aplicaciones, de manera que les permite intercambiar mensajes sin conexión directa, a fin de ofrecer mayor escalabilidad y resistencia.

**Servicios de red**

Azure ofrece varios servicios de red que admiten la autenticación y mejoran la manejabilidad de las aplicaciones hospedadas. Estos servicios incluyen lo siguiente:

* Azure Virtual Network le permite aprovisionar y administrar redes privadas virtuales (VPN) en Azure, así como vincularlas de forma segura con la infraestructura de TI local.
* Traffic Manager de Virtual Network le permite cargar tráfico entrante equilibrado entre varios servicios de Azure hospedados, independientemente de que se ejecuten en el mismo centro de datos o en diferentes centros de datos en todo el mundo.
* Azure Active Directory (Azure AD) es un servicio moderno basado en REST que ofrece capacidades de administración de identidades y control de acceso para las aplicaciones en la nube. El uso de Azure AD para Recursos de aplicaciones ofrece una forma sencilla de autenticar y autorizar a los usuarios a la hora de tener acceso a las aplicaciones y los servicios web, a la vez que se permite que las características de autenticación y autorización queden excluidas del código.
* Azure Service Bus ofrece funcionalidades de flujo de datos y mensajería segura para aplicaciones distribuidas e híbridas, como la comunicación entre aplicaciones hospedadas de Azure y aplicaciones y servicios locales, sin requerir infraestructuras de seguridad y firewall complejas. El uso de Service Bus Relay para que los recursos de aplicación accedan a los servicios expuestos como puntos de conexión puede recaer en el inquilino (por ejemplo, hospedados fuera del sistema, como en local), o bien puede tratarse de servicios aprovisionados específicamente para el inquilino (porque los datos importantes específicos del inquilino se transfieren a través de ellos).

**Aprovisionamiento de recursos**

Azure ofrece diferentes formas de aprovisionar nuevos inquilinos para la aplicación. En aplicaciones multiempresa con un gran número de inquilinos, suele ser necesario automatizar este proceso mediante la habilitación del aprovisionamiento de autoservicio.

* Los roles de trabajo le permiten aprovisionar y desaprovisionar recursos por inquilino (como cuando un nuevo inquilino inicia sesión o la cancela), recopilar métricas para medir el uso y administrar la escalación conforme a un programa determinado o para responder al cruce de umbrales de indicadores de rendimiento claves. Este mismo rol se puede usar también para realizar actualizaciones de la solución.
* Los blobs de Azure se pueden usar para aprovisionar recursos de almacenamiento informáticos o preinicializados para nuevos inquilinos, a la vez que se ofrecen directivas de acceso de nivel de contenedor para proteger los paquetes de servicios informáticos, las imágenes VHD y otros recursos.
* Las opciones para aprovisionar los recursos de la SQL Database para un inquilino incluyen:
  
  * DDL en scripts o insertados como recursos en ensamblados.
  * Paquetes SQL Server 2008 R2 DAC implementados mediante programación
  * Copiar desde una base de datos de referencia maestra.
  * Usar la importación y exportación de base de datos para aprovisionar bases de datos nuevas desde un archivo

<!--links-->

[Hosting a Multi-Tenant Application on Azure]: https://msdn.microsoft.com/library/hh534480.aspx
[Designing Multitenant Applications on Azure]: https://msdn.microsoft.com/library/windowsazure/hh689716
