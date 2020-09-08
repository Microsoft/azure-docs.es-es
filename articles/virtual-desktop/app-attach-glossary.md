---
title: 'Glosario de conexión de aplicaciones en formato MSIX para Windows Virtual Desktop: Azure'
description: Un glosario de la conexión de aplicaciones en formato MSIX con términos y conceptos.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 08/17/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: f3cc8495f673c8b428aa9e6ace2747a70c5b0847
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88556175"
---
# <a name="msix-app-attach-glossary"></a>Glosario sobre la conexión de aplicaciones en formato MSIX

En este artículo se muestra una lista de definiciones de términos y conceptos clave relacionados con la conexión de aplicaciones en formato MSIX.

## <a name="msix-container"></a>Contenedor MSIX

Un contenedor MSIX es donde se ejecutan las aplicaciones de MSIX. Para obtener más información, consulte [contenedores de MSIX](/windows/msix/msix-container).

## <a name="msix-application"></a>Aplicación MSIX 

Una aplicación almacenada en un archivo MSIX.

## <a name="msix-package"></a>Paquete MSIX 

Un paquete de MSIX es un archivo o aplicación de MSIX.

## <a name="msix-share"></a>Recurso compartido MSIX

Un recurso compartido MSIX es un recurso compartido de red que contiene paquetes de MSIX expandidos. Los recursos compartidos de MSIX admiten SMB 3 o posterior. Las aplicaciones se almacenan provisionalmente desde este recurso compartido de MSIX sin tener que trasladar los archivos de aplicación a la unidad del sistema.

## <a name="repackage"></a>Reempaquetar

El reempaquetado toma una aplicación que no es de MSIX y la convierte en MSIX mediante la herramienta de empaquetado de MSIX (MPT). Para más información, consulte [introducción a la herramienta de empaquetado de MSIX](/windows/msix/packaging-tool/tool-overview).

## <a name="expand"></a>Expanda

La expansión del paquete MSIX es un proceso de varios pasos. Toma el archivo MSIX y coloca su contenido en un archivo VHD (x) o CIM. 

Para expandir un paquete de MSIX:

1. Obtenga un paquete MSIX (archivo MSIX).
2. Cambie el nombre del archivo MSIX a un archivo .zip.
3. Descomprima el archivo. zip resultante en una carpeta.
4. Cree un VHD que tenga el mismo tamaño que la carpeta.
5. Monte el VHD.
6. Inicialice un disco.
7. Cree una partición.
8. Procese la partición.
9. Copie el contenido descomprimido en el VHD.
10. Use la herramienta MSIXMGR para aplicar las ACL en el contenido del VHD.
11. Desmonte el VHD(x) o [CIM](#cim).

## <a name="upload-an-msix-package"></a>Cargue un paquete MSIX 

Para cargar un paquete de MSIX, es necesario cargar el VHD(x) o [CIM](#cim) que contenga un paquete MSIX expandido en el recurso compartido MSIX.

En el Windows Virtual Desktop, las cargas se producen una vez por cada recurso compartido de MSIX. Una vez que se carga un paquete, todos los grupos host de la misma suscripción pueden hacer referencia a él.

## <a name="publish-an-msix-package"></a>Publique un paquete MSIX

En Windows Virtual Desktop, al publicar un paquete de MSIX, se vincula a una aplicación remota o a un escritorio.

## <a name="assign-an-msix-package"></a>Asignar un paquete MSIX 

En el Windows Virtual Desktop, se debe asignar un paquete MSIX publicado a un usuario o grupo de usuarios de Dominio de Active Directory (AD DS) o Azure Active Directory (Azure AD).

## <a name="staging"></a>Ensayo

El almacenamiento provisional implica dos cosas:

- Montaje del VHD (x) o [CIM](#cim) a la máquina virtual.
- Notificar al sistema operativo que el paquete MSIX está disponible para el registro.

## <a name="registration"></a>Registro

El registro hace que un paquete de MSIX provisional esté disponible para los usuarios. El registro se basa en cada usuario. Si no ha registrado explícitamente una aplicación para ese usuario específico, no podrá ejecutar la aplicación.

Hay dos tipos de registro: normal y retrasado.

### <a name="regular-registration"></a>Registro normal

En el registro normal, cada aplicación asignada a un usuario se registra por completo. El registro se produce durante el tiempo que el usuario inicia sesión en la sesión, lo que puede afectar al tiempo que tardan en comenzar a usar Windows Virtual Desktop.

### <a name="delayed-registration"></a>Registro con retraso

En el registro retrasado, cada aplicación asignada a un usuario solo se registra parcialmente. El registro parcial significa que el icono del menú Inicio y las conexiones de archivo de doble clic están registradas. El registro se produce durante el tiempo que el usuario inicia sesión, lo que puede afectar al tiempo que tardan en comenzar a usar Windows Virtual Desktop. El registro se completa solo cuando el usuario ejecuta la aplicación en el paquete MSIX.

El registro retrasado es actualmente la configuración predeterminada para la conexión de aplicaciones en formato MSIX.

## <a name="deregistration"></a>Anulación

Al anular el registro se quita un paquete de MSIX registrado pero no en ejecución para un usuario. La anulación del registro se produce mientras el usuario cierra la sesión. Durante la anulación del registro, la conexión de la aplicación en MSIX envía los datos de aplicación específicos del usuario al perfil de usuario local.

## <a name="destage"></a>Quitar del "stage"

El desalmacenamiento notifica al sistema operativo que un paquete MSIX o una aplicación que actualmente no se está ejecutando y no está almacenada provisionalmente para cualquier usuario puede desmontarse. Esto quita todas las referencias a él en el sistema operativo.

## <a name="cim"></a>CIM

.CIM es una nueva extensión de archivo asociada con el sistema de archivos de imagen compuesta (CimFS). El montaje y desmontaje de archivos CIM es más rápido que los archivos VHD. CIM también consume menos CPU y memoria que VHD.

La tabla siguiente es una comparación de rendimiento entre VHD y CimFS. Estos números fueron el resultado de una serie de pruebas con archivos de 300 MB en cada formato ejecutado en un equipo DSv4.

|  Especificaciones                          | VHD                    | CimFS   |
|---------------------------------|--------------------------|-----------|
| Tiempo medio de montaje     | 356 ms                     | 255 ms      |
| Tiempo medio de desmontado   | 1615 ms                    | 36 ms       |
| Consumo de memoria | 6 % (de 8 GB)                      | 2 % (de 8 GB)       |
| CPU (pico de recuento)          | Supere el tiempo de espera varias veces | Ningún impacto |

## <a name="next-steps"></a>Pasos siguientes

Si quiere obtener más información sobre la conexión de aplicaciones en formato MSIX, consulte nuestra [introducción](what-is-app-attach.md) y [Preguntas frecuentes](app-attach-faq.md). De lo contrario, empiece a trabajar con [Configuración de la conexión de aplicaciones](app-attach.md).

