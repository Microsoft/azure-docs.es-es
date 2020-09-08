---
title: 'Introducción a la conexión de aplicaciones en formato MSIX para Windows Virtual Desktop: Azure'
description: ¿Qué es la asociación de aplicaciones en formato .MSIX? Obtenga más información en este artículo.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 08/17/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: c6bf296b5173a662b1e9dd7b025648e3f16d23c8
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88556171"
---
# <a name="what-is-msix-app-attach"></a>¿Qué es la asociación de aplicaciones en formato .MSIX?

MSIX es un nuevo formato de empaquetado que ofrece muchas características destinadas a mejorar la experiencia de empaquetado para todas las aplicaciones de Windows. Para obtener más información sobre MSIX, consulte la información general sobre la [introducción a MSIX](/windows/msix/overview).

La conexión de aplicaciones en formato MSIX es una manera de proporcionar aplicaciones en MSIX a máquinas virtuales y físicas. Sin embargo, la conexión de aplicaciones MSIX es diferente de MSIX normal porque se creó especialmente para Windows Virtual Desktop. En este artículo se describe lo que es la conexión de aplicaciones MSIX y lo que puede hacer por usted.

## <a name="application-delivery-options-in-windows-virtual-desktop"></a>Opciones de entrega de aplicaciones en el Windows Virtual Desktop

Puede proporcionar aplicaciones en Windows Virtual Desktop a través de uno de los métodos siguientes:

- Poner aplicaciones en una imagen maestra
- Usar herramientas como SCCM o Intune para la administración central
- Aprovisionamiento de aplicaciones dinámicas (AppV, VMWare AppVolumes o Citrix AppLayering)
- Crear scripts o herramientas personalizadas con Microsoft y una herramienta de terceros

## <a name="what-does-msix-app-attach-do"></a>¿Qué hace la conexión de aplicaciones MSIX?

En una implementación de Windows Virtual Desktop, la conexión de aplicaciones MSIX puede:

- Crear una separación entre los datos de usuario, el sistema operativo y las aplicaciones mediante [contenedores de MSIX](/windows/msix/msix-container).
- Eliminar la necesidad de volver a empaquetar al entregar aplicaciones de forma dinámica.
- Reducir el tiempo que tarda un usuario en iniciar sesión.
- Reducir el coste y los requisitos de infraestructura.

La asociación de aplicaciones MSIX debe ser aplicable fuera de VDI o SBC.

## <a name="traditional-app-layering-compared-to-msix-app-attach"></a>Las capas tradicionales de la aplicación en comparación con la conexión de aplicaciones MSIX

En la tabla siguiente se compara la característica clave de la conexión de aplicaciones MSIX con las capas tradicionales de la aplicación.

| Característica | Capas tradicionales de la aplicación  | Asociación de aplicaciones en formato .MSIX  |
|-----|-----------------------------|--------------------|
| Formato               | Diferentes tecnologías de niveles de aplicación requieren diferentes formatos de propietario. | Funciona con el formato de empaquetado de MSIX nativo.        |
| Reempaquetado de sobrecarga | Los formatos de propiedad requieren secuenciación y reempaquetado por actualización.         | No es necesario volver a empaquetar las aplicaciones publicadas como MSIX. Sin embargo, si el paquete MSIX no está disponible, se seguirá aplicando la sobrecarga de empaquetado. |
| Ecosistema            | N/A (por ejemplo, los proveedores no envían App-V)  | MSIX es la tecnología estándar de Microsoft que están adoptando los asociados clave de ISV y aplicaciones internas como Office. Puede usar MSIX en los escritorios virtuales y equipos físicos de Windows. |
| Infraestructura       | Se requiere una infraestructura adicional (servidores, clientes, etc.) | Solo almacenamiento   |
| Administración       | Requiere mantenimiento y actualización   | Simplifica las actualizaciones de la aplicación |
| Experiencia del usuario      | Afecta al tiempo de inicio de sesión del usuario. El límite existe entre el estado del sistema operativo, el estado de la aplicación y los datos de usuario.  | No se pueden distinguir las aplicaciones entregadas de las aplicaciones instaladas localmente. |

## <a name="next-steps"></a>Pasos siguientes

Si quiere obtener más información sobre la asociación de aplicaciones en formato MSIX, consulte nuestro [glosario](app-attach-glossary.md) y [Preguntas frecuentes](app-attach-faq.md). De lo contrario, empiece a trabajar con [Configuración de la asociación de aplicaciones](app-attach.md).
