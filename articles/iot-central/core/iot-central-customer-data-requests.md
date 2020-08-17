---
title: Características de solicitud de datos del cliente de Azure IoT Central | Microsoft Docs
description: En este artículo se describe cómo identificar, eliminar y exportar datos de clientes en la aplicación de Azure IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 08/23/2019
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: timlt
ms.openlocfilehash: dabcadea96f4ced5bdf73a35ef533e6d290595c2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87001883"
---
# <a name="azure-iot-central-customer-data-request-features"></a>Características de solicitud de datos del cliente de Azure IoT Central

Azure IoT Central es una solución de software como servicio de Internet de las cosas (IoT) totalmente administrada que facilita la conexión, la supervisión y la administración de sus recursos de IoT a escala, la extracción de conclusiones detalladas de sus datos de IoT y la realización de acciones informadas.

[!INCLUDE [gdpr-intro-sentence](../../../includes/gdpr-intro-sentence.md)]

## <a name="identifying-customer-data"></a>Identificación de los datos del cliente

Los id. de objeto de Azure Active Directory se utilizan para identificar usuarios y asignar roles. El portal Azure IoT Central muestra direcciones de correo electrónico de usuario para asignaciones de roles, pero solo se almacena el id. de objeto de Azure Active Directory; la dirección de correo electrónico se consulta de forma dinámica desde Azure Active Directory. Los administradores de Azure IoT Central pueden ver, exportar y eliminar usuarios de la aplicación en la sección de administración de usuarios de una aplicación de Azure IoT Central.

Desde la aplicación, se pueden configurar direcciones de correo electrónico para recibir alertas. En este caso, las direcciones de correo electrónico se almacenan en IoT Central y se deben administrar desde la página de administración de la cuenta de la aplicación.

En cuanto a los dispositivos, Microsoft no conserva ninguna información y no tiene acceso a datos que permitan la correlación entre el dispositivo y el usuario. Muchos de los dispositivos que se administran en Azure IoT Central no son dispositivos personales; por ejemplo, una máquina expendedora o una cafetera. Sin embargo, los clientes pueden considerar que algunos dispositivos se pueden identificar a nivel personal y, a su discreción, pueden mantener sus propios sistemas de seguimiento de activos o de inventario que vinculan dispositivos a personas. Azure IoT Central administra y almacena todos los datos asociados con dispositivos como si fueran datos personales.

Cuando se usan servicios para empresas de Microsoft, Microsoft genera cierta información, conocida como registros generados por el sistema. Estos registros constituyen acciones objetivas que se realizan en el servicio, diagnostican datos relacionados con dispositivos individuales y no están relacionados con la actividad del usuario. Los administradores de la aplicación no pueden acceder a los registros generados por el sistema de Azure IoT Central ni exportarlos.

## <a name="deleting-customer-data"></a>Eliminación de datos del cliente

La capacidad de eliminar datos del usuario solo se proporciona a través de la página de administración de IoT Central. Para eliminar el registro, los administradores de la aplicación pueden seleccionar el usuario que quieren eliminar y elegir **Eliminar** en la esquina superior derecha de la aplicación. Los administradores de la aplicación también pueden quitar cuentas individuales que ya no están asociadas con la aplicación en cuestión.

Al eliminar un usuario, este deja de recibir alertas por correo electrónico. Sin embargo, su dirección de correo electrónico debe quitarse individualmente de cada alerta configurada.

## <a name="exporting-customer-data"></a>Exportación de datos del cliente

La capacidad de exportar datos solo se proporciona a través de la página de administración de IoT Central. Un usuario de la aplicación puede seleccionar, copiar y pegar los datos del cliente, incluidos los roles asignados.

## <a name="links-to-additional-documentation"></a>Vínculos a documentación adicional

Para obtener más información acerca de la administración de cuentas, incluidas las definiciones de roles, consulte [How to administer your application](howto-administer.md) (Cómo administrar la aplicación).
