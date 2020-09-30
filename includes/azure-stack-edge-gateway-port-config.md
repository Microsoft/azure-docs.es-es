---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 12/16/2019
ms.author: alkohli
ms.openlocfilehash: baf18ae0263215e6ff83570557255d06c3117fd4
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89082474"
---
| N.º de puerto| Dentro o fuera | Ámbito de puerto| Obligatorio|   Notas |   |
|--------|-----|-----|-----------|----------|-----------|
| TCP 80 (HTTP)|Fuera|WAN |No|El puerto de salida se usa para obtener acceso a Internet para así recuperar las actualizaciones. <br>El usuario puede configurar el proxy web de salida. |
| TCP 443 (HTTPS)|Fuera|WAN|Sí|El puerto de salida se usa para tener acceso a los datos en la nube.<br>El usuario puede configurar el proxy web de salida.|
| UDP 123 (NTP)|Fuera|WAN|En algunos casos.<br>Vea las notas|Este puerto es necesario solo si está utilizando un servidor NTP basado en Internet.  |   
| UDP 53 (DNS)|Fuera|WAN|En algunos casos.<br>Vea las notas|Este puerto es necesario solo si está utilizando un servidor DNS basado en Internet.<br>Se recomienda usar un servidor DNS local. |
| TCP 5985 (WinRM)|Salida/Entrada|LAN|En algunos casos.<br>Vea las notas|Este puerto es necesario para conectarse al dispositivo a través de la instancia remota de PowerShell a través de HTTP.  |
| UDP 67 (DHCP)|Fuera|LAN|En algunos casos.<br>Vea las notas|Este puerto solo es necesario si se usa un servidor DHCP local.  |
| TCP 80 (HTTP)|Salida/Entrada|LAN|Sí|Este es el puerto de entrada de la interfaz de usuario local del dispositivo para la administración local. <br>Si obtiene acceso a la interfaz de usuario local mediante HTTP, se le redireccionará automáticamente a HTTPS.  |
| TCP 443 (HTTPS)|Salida/Entrada|LAN|Sí|Este es el puerto de entrada de la interfaz de usuario local del dispositivo para la administración local. Este puerto también se usa para conectar Azure Resource Manager a las API locales del dispositivo, para conectar el almacenamiento de blobs a través de las API REST y al servicio de token de seguridad (STS) para autenticar a través de tokens de acceso y actualización.|
| TCP 445 (SMB)|En|LAN|En algunos casos.<br>Vea las notas|Este puerto solo es necesario si se va a conectar mediante SMB. |
| TCP 2049 (NFS)|En|LAN|En algunos casos.<br>Vea las notas|Este puerto solo es necesario si se va a conectar mediante NFS. |


