---
title: Certificados necesarios para permitir servidores back-end
titleSuffix: Azure Application Gateway
description: En este artículo se proporcionan ejemplos de cómo un certificado TLS/SSL se puede convertir en un certificado de autenticación y en un certificado raíz de confianza, necesarios para que se permitan instancias de back-end en Azure Application Gateway.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 06/17/2020
ms.author: absha
ms.openlocfilehash: 64f2abd2a42fb15b994803a48b97679ee8927233
ms.sourcegitcommit: 1b320bc7863707a07e98644fbaed9faa0108da97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89594431"
---
# <a name="create-certificates-to-allow-the-backend-with-azure-application-gateway"></a>Creación de certificados para permitir el back-end con Azure Application Gateway

Para establecer TLS de extremo a extremo, Application Gateway requiere que las instancias de back-end se autoricen mediante la carga de los certificados raíz de confianza o de autenticación. En el caso de la SKU v1, se requieren certificados de autenticación, pero para la SKU v2, se requieren certificados raíz de confianza para permitir los certificados.

En este artículo aprenderá a:


- Exportar el certificado de autenticación desde un certificado de back-end (para SKU v1)
- Exportar el certificado raíz de confianza desde un certificado de back-end (para SKU v2)

## <a name="prerequisites"></a>Requisitos previos

Se requiere un certificado de back-end existente para generar los certificados de autenticación o los certificados raíz de confianza necesarios para permitir las instancias de back-end con Application Gateway. El certificado de back-end puede ser igual que el certificado TLS/SSL o distinto para una mayor seguridad. Application Gateway no proporciona ningún mecanismo para crear o adquirir un certificado TLS/SSL. Con fines de prueba, puede crear un certificado autofirmado, pero no debe usarlo en cargas de trabajo de producción. 

## <a name="export-authentication-certificate-for-v1-sku"></a>Exportación del certificado de autenticación (para SKU v1)

Se requiere un certificado de autenticación para permitir las instancias de back-end de la SKU v1 de Application Gateway. El certificado de autenticación es la clave pública de los certificados de servidor de back-end en formato X.509 codificado en Base 64 (.CER). En este ejemplo, se usará un certificado TLS/SSL como certificado de back-end y se exportará su clave pública para usarla como certificado de autenticación. Además, se usará la herramienta Administrador de certificados de Windows para exportar los certificados necesarios. Puede optar por usar cualquier otra herramienta que le resulte cómoda.

Desde el certificado TLS/SSL, exporte el archivo .cer de clave pública (no la clave privada). Los siguientes pasos ayudan exportar el archivo .cer en formato X.509 codificado en Base 64 (.CER) para el certificado:

1. Para obtener un archivo .cer del certificado, abra **Administrar certificados de usuario**. Busque el certificado; normalmente se encuentra en "Certificados - Usuario actual\Personal\Certificados" y haga clic con el botón derecho. Haga clic en **Todas las tareas** y, luego, en **Exportar**. Se abre el **Asistente para exportar certificados**. Si no encuentra el certificado en Usuario actual\Personal\Certificados, puede que haya abierto de forma accidental "Certificados – equipo Local", en lugar de "Certificados - Usuario actual"). Si desea abrir el Administrador de certificados en el ámbito del usuario actual mediante PowerShell, escriba *certmgr* en la ventana de la consola.

   ![Exportación](./media/certificates-for-backend-authentication/export.png)

2. En el asistente, haga clic en **Siguiente**.

   ![Exportación de certificado](./media/certificates-for-backend-authentication/exportwizard.png)

3. Seleccione **No exportar la clave privada** y, después, haga clic en **Siguiente**.

   ![No exportar la clave privada](./media/certificates-for-backend-authentication/notprivatekey.png)

4. En la página **Formato de archivo de exportación**, seleccione **X.509 codificado base 64 (.CER)** y, luego, haga clic en **Siguiente**.

   ![Codificado con Base 64](./media/certificates-for-backend-authentication/base64.png)

5. En **Archivo que se va a exportar**, haga clic en **Examinar** para ir a la ubicación a la que desea exportar el certificado. En **Nombre de archivo**, asígnele un nombre al archivo de certificado. A continuación, haga clic en **Siguiente**.

   ![Examinar](./media/certificates-for-backend-authentication/browse.png)

6. Haga clic en **Finalizar** para exportar el certificado.

   ![Finalizar](./media/certificates-for-backend-authentication/finish.png)

7. El certificado se exportó correctamente.

   ![Correcto](./media/certificates-for-backend-authentication/success.png)

   El certificado exportado tiene un aspecto similar al siguiente:

   ![Exportado](./media/certificates-for-backend-authentication/exported.png)

8. Si abre el certificado exportado mediante el Bloc de notas, verá algo parecido a este ejemplo. La sección en azul contiene la información que se carga en Application Gateway. Si abre el certificado con el Bloc de notas y no se parece a este, habitualmente eso significa que no lo exportó mediante el formato X.509 codificado base 64 (. CER). Además, si desea utilizar un editor de texto diferente, debe comprender que algunos editores pueden introducir formatos no deseados en segundo plano. Esto puede crear problemas al cargar el texto de este certificado en Azure.

   ![Abrir con el Bloc de notas](./media/certificates-for-backend-authentication/format.png)

## <a name="export-trusted-root-certificate-for-v2-sku"></a>Exportación del certificado raíz de confianza (para SKU v2)

Se requiere un certificado raíz de confianza para permitir las instancias de back-end de la SKU v2 de la puerta de enlace de aplicaciones. El certificado raíz es un certificado raíz en formato X.509 codificado en base 64 (.CER) de los certificados de servidor de back-end. En este ejemplo, se usará un certificado TLS/SSL como certificado de back-end, se exportará su clave pública y, luego, se exportará el certificado raíz de la entidad de certificación de confianza de la clave pública en formato codificado en Base 64 para obtener el certificado raíz de confianza. Los certificados intermedios se deben agrupar con un certificado de servidor e instalar en el servidor de back-end.

Los pasos siguientes le ayudan a exportar el archivo .cer para el certificado:

1. Use los pasos del 1 al 8 mencionados en la sección anterior [Exportación del certificado de autenticación (para SKU v1)](#export-authentication-certificate-for-v1-sku) para exportar la clave pública desde el certificado de back-end.

2. Cuando se haya exportado la clave pública, abra el archivo.

   ![Abrir el certificado de autorización](./media/certificates-for-backend-authentication/openAuthcert.png)

   ![acerca de certificado](./media/certificates-for-backend-authentication/general.png)

3. Mueva la vista de la ruta de certificación para ver la entidad de certificación.

   ![información del certificado](./media/certificates-for-backend-authentication/certdetails.png)

4. Seleccione el certificado raíz y haga clic en **Ver certificado**.

   ![ruta de acceso del certificado](./media/certificates-for-backend-authentication/rootcert.png)

   Debería ver los detalles del certificado raíz.

   ![información de certificado](./media/certificates-for-backend-authentication/rootcertdetails.png)

5. Vaya a la vista **Detalles** y haga clic en **Copiar a archivo...**

   ![copia del certificado raíz](./media/certificates-for-backend-authentication/rootcertcopytofile.png)

6. En este punto, ha extraído los detalles del certificado raíz del certificado de back-end. Verá el **Asistente para exportación de certificados**. Ahora use los pasos del 2 al 9 mencionados en la sección anterior **Exportar el certificado de autenticación desde un certificado de back-end (para SKU v1)** para exportar el certificado raíz de confianza en formato X.509 codificado en Base 64 (.CER).

## <a name="next-steps"></a>Pasos siguientes

Ahora tiene el certificado de autenticación y el certificado raíz de confianza en formato X.509 codificado en Base 64 (.CER). Puede agregarlos a la puerta de enlace de aplicaciones para permitir los servidores back-end para el cifrado TLS de un extremo a otro. Consulte [Configuración de TLS de un extremo a otro con Application Gateway mediante PowerShell](https://docs.microsoft.com/azure/application-gateway/application-gateway-end-to-end-ssl-powershell).

