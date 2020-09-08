---
title: 'Azure VPN Gateway: generación y exportación de certificados para la conexión de punto a sitio: MakeCert'
description: Cree un certificado raíz autofirmado, exporte la clave pública y genere los certificados de cliente mediante MakeCert.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: 926de9f3fd357cd9d9ca067e4f7beff7d03eec95
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89394199"
---
# <a name="generate-and-export-certificates-for-point-to-site-connections-using-makecert"></a>Generación y exportación de certificados para conexiones de punto a sitio con MakeCert

Las conexiones de punto a sitio utilizan certificados para realizar la autenticación. En este artículo, se muestra cómo crear un certificado raíz autofirmado y generar certificados de cliente con MakeCert. Si busca otras instrucciones de certificado, vea [Certificates - PowerShell](vpn-gateway-certificates-point-to-site.md) (Certificados: PowerShell) o [Certificates - Linux](vpn-gateway-certificates-point-to-site-linux.md) (Certificados: Linux).

Aunque se recomienda utilizar los [pasos de Windows 10 PowerShell](vpn-gateway-certificates-point-to-site.md) para crear los certificados, proporcionamos estas instrucciones de MakeCert como un método opcional. Los certificados que genera mediante cualquiera de estos métodos pueden instalarse en [cualquier sistema operativo cliente compatible](vpn-gateway-howto-point-to-site-resource-manager-portal.md#faq). Sin embargo, MakeCert tiene la siguiente limitación:

* Makecert está en desuso. Esto significa que se puede quitar esta herramienta en cualquier momento. Los certificados ya generados con MakeCert no se verán afectados si MakeCert ya no está disponible. MakeCert solo se utiliza para generar los certificados, no como un mecanismo de validación.

## <a name="create-a-self-signed-root-certificate"></a><a name="rootcert"></a>Creación de un certificado raíz autofirmado

Los siguientes pasos le mostrarán cómo crear un certificado autofirmado mediante MakeCert. Estos pasos no son específicos del modelo de implementación. Son válidos para el Administrador de recursos y la versión clásica.

1. Descargue e instale [MakeCert](https://msdn.microsoft.com/library/windows/desktop/aa386968(v=vs.85).aspx).
2. Después de la instalación, la utilidad makecert.exe se encuentra normalmente en esta ruta de acceso: "C:\Archivos de programa (x86)\Windows Kits\10\bin\<arch>". Sin embargo, es posible que se haya instalado en otra ubicación. Abra un símbolo del sistema como administrador y navegue hasta la ubicación de la utilidad MakeCert. Puede usar el siguiente ejemplo y ajustar la ubicación adecuada:

   ```cmd
   cd C:\Program Files (x86)\Windows Kits\10\bin\x64
   ```
3. Después, cree e instale un certificado en el almacén de certificados personal de su equipo. En el ejemplo siguiente se crea un archivo *.cer* correspondiente que se carga en Azure al configurar P2S. Reemplace P2SRootCert y P2SRootCert.cer por el nombre que quiera usar para el certificado. El certificado se encuentra en los certificados, en "Usuario actual\Personal\Certificados".

   ```cmd
   makecert -sky exchange -r -n "CN=P2SRootCert" -pe -a sha256 -len 2048 -ss My
   ```

## <a name="export-the-public-key-cer"></a><a name="cer"></a>Exportación de la clave pública (.cer)

[!INCLUDE [Export public key](../../includes/vpn-gateway-certificates-export-public-key-include.md)]

El archivo exported.cer debe cargarse en Azure. Para ver las instrucciones, consulte [Configuración de una conexión de punto a sitio](vpn-gateway-howto-point-to-site-resource-manager-portal.md#uploadfile). Para agregar un certificado raíz de confianza adicional, vea [esta sección](vpn-gateway-howto-point-to-site-resource-manager-portal.md#add) del artículo.

### <a name="export-the-self-signed-certificate-and-private-key-to-store-it-optional"></a>Exportación del certificado autofirmado y clave privada para almacenarlo (opcional)

Puede que desee exportar el certificado autofirmado y almacenarlo de manera segura. Si es necesario, más adelante puede instalarlo en otro equipo y generar más certificados de cliente o exportar otro archivo .cer. Para exportar el certificado raíz autofirmado como archivo .pfx, seleccione el certificado raíz y use los mismos pasos descritos en [Exportación de un certificado de cliente](#clientexport).

## <a name="create-and-install-client-certificates"></a>Creación e instalación de certificados de cliente

No instale el certificado autofirmado directamente en el equipo cliente. Debe generar un certificado de cliente a partir del certificado autofirmado. Luego, el certificado de cliente se exporta e instala en el equipo cliente. Los siguientes pasos no son específicos del modelo de implementación. Son válidos para el Administrador de recursos y la versión clásica.

### <a name="generate-a-client-certificate"></a><a name="clientcert"></a>Generación de un certificado de cliente

Cada equipo cliente que se conecta a una red virtual con una conexión de punto a sitio debe tener instalado un certificado de cliente. Puede generarlo desde un certificado raíz autofirmado y, luego, exportar e instalar el certificado de cliente. Si no está instalado el certificado de cliente, se produce un error de autenticación. 

Los pasos siguientes lo llevan por el proceso de generación de un certificado de cliente a partir de un certificado autofirmado. Puede generar varios certificados de cliente desde el mismo certificado raíz. Al generar certificados de cliente mediante los pasos siguientes, el certificado de cliente se instala automáticamente en el equipo que se usó para generar el certificado. Si desea instalar un certificado de cliente en otro equipo cliente, puede exportar el certificado.
 
1. En el mismo equipo que usó para crear el certificado autofirmado, abra un símbolo del sistema como administrador.
2. Modifique y ejecute el ejemplo para generar un certificado de cliente.
   * Cambie *"P2SRootCert"* por el nombre del certificado raíz autofirmado desde el que genera el certificado de cliente. Asegúrese de que esté usando el nombre del certificado raíz, que será el valor de CN= que especificó cuando creó el certificado raíz autofirmado.
   * Cambie *P2SChildCert* por el nombre con el cual desea generar un certificado de cliente.

   Si ejecuta el mismo ejemplo sin modificarlo, el resultado es un certificado de cliente denominado P2SChildcert en el almacén de certificados personal que se generó a partir del certificado raíz P2SRootCert.

   ```cmd
   makecert.exe -n "CN=P2SChildCert" -pe -sky exchange -m 96 -ss My -in "P2SRootCert" -is my -a sha256
   ```

### <a name="export-a-client-certificate"></a><a name="clientexport"></a>Exportación de un certificado de cliente

[!INCLUDE [Export client certificate](../../includes/vpn-gateway-certificates-export-client-cert-include.md)]

### <a name="install-an-exported-client-certificate"></a><a name="install"></a>Instalación de un certificado de cliente exportado

Para instalar un certificado de cliente, consulte [Instalación de un certificado de cliente](point-to-site-how-to-vpn-client-install-azure-cert.md).

## <a name="next-steps"></a>Pasos siguientes

Continúe con la configuración de punto a sitio. 

* Para conocer los pasos del modelo de implementación de **Resource Manager**, consulte cómo [configurar una conexión de punto a sitio mediante la autenticación de certificados de Azure nativa](vpn-gateway-howto-point-to-site-resource-manager-portal.md).
* Para ver los pasos del modelo de implementación **clásica**, consulte el artículo [Configuración de una conexión VPN de punto a sitio a una red virtual mediante el Portal clásico](vpn-gateway-howto-point-to-site-classic-azure-portal.md).

Para obtener información sobre solución de problemas de P2S, vea [Solución de problemas: conexión de punto a sitio de Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).