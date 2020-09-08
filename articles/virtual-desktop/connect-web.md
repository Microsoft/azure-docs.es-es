---
title: 'Conexión a Windows Virtual Desktop con el cliente web: Azure'
description: Cómo conectarse a Windows Virtual Desktop mediante el cliente web.
author: Heidilohr
ms.topic: how-to
ms.date: 09/24/2019
ms.author: helohr
manager: lizross
ms.openlocfilehash: c8a4b22966c3d4db268e212bb3f2d1bbb78fee74
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400643"
---
# <a name="connect-to-windows-virtual-desktop-with-the-web-client"></a>Conexión a Windows Virtual Desktop con el cliente web

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop con objetos de Windows Virtual Desktop de Azure Resource Manager. Si usa Windows Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/connect-web-2019.md).

El cliente web le permite acceder a los recursos de Windows Virtual Desktop desde un explorador web sin el largo proceso de instalación.

>[!NOTE]
>El cliente web no tiene compatibilidad con SO móvil actualmente.

## <a name="supported-operating-systems-and-browsers"></a>Exploradores y sistemas operativos compatibles

Aunque debería funcionar cualquier explorador compatible con HTML5, oficialmente se admiten los siguientes sistemas operativos y exploradores.

| Browser           | SO admitido                     | Notas               |
|-------------------|----------------------------------|---------------------|
| Microsoft Edge    | Windows                          |                     |
| Internet Explorer | Windows                          | Versión 11 o posterior |
| Apple Safari      | macOS                            |                     |
| Mozilla Firefox   | Windows, macOS, Linux            | Versión 55 o posterior |
| Google Chrome     | Windows, macOS, Linux, Chrome OS |                     |

## <a name="access-remote-resources-feed"></a>Acceso remoto a la fuente de recursos

En un explorador, vaya a la versión integrada de Windows Virtual Desktop del cliente web de Azure Resource Manager, en <https://rdweb.wvd.microsoft.com/arm/webclient>, e inicie sesión con su cuenta de usuario.

>[!NOTE]
>En cambio, si usa Windows Virtual Desktop (clásico) sin integración con Azure Resource Manager, conecte los recursos a <https://rdweb.wvd.microsoft.com/webclient>.
>
> Si utiliza el portal de US Gov, use <https://rdweb.wvd.azure.us/arm/webclient/index.html>.

>[!NOTE]
>Si ya ha iniciado sesión con una cuenta de Azure Active Directory diferente a la que quiere usar para Windows Virtual Desktop, debe cerrar la sesión o usar una ventana privada del explorador.

Después de iniciar sesión, debería ver una lista de recursos. Puede iniciar los recursos seleccionándolos de la misma forma en que lo haría con una aplicación normal en la pestaña **Todos los recursos**.

## <a name="using-an-input-method-editor"></a>Uso de un Editor de métodos de entrada

El cliente web admite el uso de un Editor de métodos de entrada (IME) en la sesión remota de la versión **1.0.21.16 o posterior**. El paquete de idioma del teclado que quiere usar en la sesión remota debe estar instalado en la máquina virtual del host. Para obtener más información sobre cómo configurar paquetes de idioma en la sesión remota, consulte [Adición de paquetes de idioma a una imagen multisesión de Windows 10](language-packs.md).

Para habilitar la entrada de IME mediante el cliente web:

1. Antes de conectarse a la sesión remota, vaya al panel **Configuración** del cliente web.

2. Ajuste la opción **Enable Input Method Editor** (Habilitar el Editor de métodos de entrada) en **Activado**.

3. En el menú desplegable, seleccione el teclado que va a usar en la sesión remota.

4. Conéctese a la sesión remota.

El cliente web suprimirá la ventana del IME local cuando se active la sesión remota. El cambio de configuración del IME una vez que se haya conectado a la sesión remota no tendrá ningún efecto.

>[!NOTE]
>Si el paquete de idioma no está instalado en la máquina virtual del host, la sesión remota se establecerá de forma predeterminada en el teclado inglés (Estados Unidos).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo usar el cliente web, consulte [Introducción al cliente web](/windows-server/remote/remote-desktop-services/clients/remote-desktop-web-client).
