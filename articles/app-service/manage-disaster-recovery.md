---
title: Recuperación de un error en toda la región
description: Obtenga información sobre cómo Azure App Service le ayuda a mantener las funcionalidades de continuidad empresarial y recuperación ante desastres (BCDR). Recupere la aplicación de un error en toda una región de Azure.
ms.topic: how-to
ms.date: 06/09/2020
ms.custom: subject-moving-resources
ms.openlocfilehash: 1388dc11254324f74efcbaa55c97cac2ccd0c026
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073738"
---
# <a name="move-an-app-service-app-to-another-region"></a>Traslado de una aplicación de App Service a otra región

En este artículo se describe cómo volver a poner en línea los recursos de App Service en una región de Azure diferente durante un desastre que afecte a toda una región de Azure. Cuando un desastre deja sin conexión a toda una región de Azure, todas las aplicaciones de App Service hospedadas en dicha región se colocan en modo de recuperación ante desastres. Hay características disponibles para ayudarle a restaurar la aplicación en otra región o recuperar archivos de la aplicación afectada.

Los recursos de App Service son específicos de una región y no pueden moverse entre regiones. Debe restaurar la aplicación en una nueva aplicación en una región distinta y luego crear configuraciones o recursos de reflejo para la nueva aplicación.

## <a name="prerequisites"></a>Requisitos previos

- Ninguno. La [restauración a partir de una instantánea](app-service-web-restore-snapshots.md) normalmente requiere un nivel **Premium** pero, en modo de recuperación ante desastres, se habilita automáticamente para la aplicación afectada, independientemente del nivel en el que se encuentra dicha aplicación.

## <a name="prepare"></a>Preparación

Identifique todos los recursos de App Service que la aplicación afectada utiliza actualmente. Por ejemplo:

- Aplicaciones de App Service
- [Planes de App Service](overview-hosting-plans.md)
- [Ranuras de implementación](deploy-staging-slots.md)
- [Dominios personalizados adquiridos en Azure](manage-custom-dns-buy-domain.md)
- [Certificados SSL](configure-ssl-certificate.md)
- [Integración de Azure Virtual Network](web-sites-integrate-with-vnet.md)
- [Conexiones híbridas](app-service-hybrid-connections.md)
- [Identidades administradas](overview-managed-identity.md)
- [Configuración de copia de seguridad](manage-backup.md)

Ciertos recursos, como los certificados importados o las conexiones híbridas, tienen elementos que están integrados con otros servicios de Azure. Para más información acerca de cómo migrar esos recursos entre regiones, consulte la documentación de los servicios correspondientes.

## <a name="restore-app-to-a-different-region"></a>Restauración de una aplicación a una región diferente

1. Cree una nueva aplicación de App Service en una región de Azure *diferente* a la aplicación afectada. Esta es la aplicación de destino en el escenario de recuperación ante desastres.

1. En [Azure Portal](https://portal.azure.com), vaya a la página de administración de la aplicación afectada. En una región de Azure con errores, la aplicación afectada muestra un texto de advertencia. Haga clic en el texto de advertencia.

    ![Captura de pantalla de la página de la aplicación afectada. Se ve una notificación de advertencia que describe la situación y proporciona un vínculo para restaurar la aplicación.](media/manage-disaster-recovery/restore-start.png)

1. En la página **Restaurar copia de seguridad**, configure la operación de restauración de acuerdo con la tabla siguiente. Cuando termine, haga clic en **Aceptar**.

   | Configuración | Value | Descripción |
   |-|-|-|
   | **Instantánea (versión preliminar)** | Seleccione una instantánea. | Están disponibles las dos instantáneas más recientes. |
   | **Destino de restauración** | **Aplicación existente** | Haga clic en la nota siguiente que indica **Haga clic aquí para cambiar la aplicación de destino de la restauración** y seleccione la aplicación de destino. En un escenario de desastre, solo puede restaurar la instantánea en una aplicación en una región de Azure diferente. |
   | **Restaurar la configuración del sitio** | **Sí** | |

    ![Captura de pantalla de la página Restaurar copia de seguridad. Una instantánea específica, las opciones que se muestran en la tabla anterior y el botón Aceptar están resaltados.](media/manage-disaster-recovery/restore-configure.png)

3. Configure el [resto del contenido](#prepare) en la aplicación de destino para reflejar la aplicación afectada y compruebe la configuración.

4. Cuando tenga todo preparado para que el dominio personalizado apunte a la aplicación de destino, [reasigne el nombre de dominio](manage-custom-dns-migrate-domain.md#remap-the-active-dns-name).

## <a name="recover-app-content-only"></a>Recuperación solamente del contenido de la aplicación

Si solo desea recuperar los archivos de la aplicación afectada sin restaurarlos, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), vaya a la página de administración de la aplicación afectada y haga clic en **Obtener perfil de publicación**.

    ![Captura de pantalla de la página de la aplicación afectada. Se ve una notificación de advertencia, pero no está resaltada. En su lugar, el elemento Obtener perfil de publicación está resaltado.](media/manage-disaster-recovery/get-publish-profile.png)

1. Abra el archivo descargado y busque el perfil de publicación que contiene `ReadOnly - FTP` en su nombre. Se trata del perfil de recuperación ante desastres. Por ejemplo:

    ```xml
    <publishProfile profileName="%app-name% - ReadOnly - FTP" publishMethod="FTP" publishUrl="ftp://%ftp-site%/site/wwwroot" ftpPassiveMode="True" userName="%app-name%\$%app-name%" userPWD="" destinationAppUrl="http://%app-name%.azurewebsites.net" SQLServerDBConnectionString="" mySQLDBConnectionString="" hostingProviderForumLink="" controlPanelLink="http://windows.azure.com" webSystem="WebSites">
        <databases />
    </publishProfile>
    ```
    
    Copie tres valores de atributo: 
        
    - `publishUrl`: nombre de host de FTP
    - `userName` y `userPWD`: las credenciales de FTP

1. Use el cliente FTP que prefiera y conéctese al host de FTP de la aplicación afectada mediante el nombre de host y las credenciales.

1. Una vez conectado, descargue toda la carpeta */site/wwwroot*. En la captura de pantalla siguiente se muestra cómo descargar en [FileZilla](https://filezilla-project.org/).

    ![Captura de pantalla de una jerarquía de archivos FileZilla. La carpeta wwwroot está resaltada y se ve el menú contextual. En ese menú, Descargar está resaltado.](media/manage-disaster-recovery/download-content.png)

## <a name="next-steps"></a>Pasos siguientes
[Restauración de una aplicación en Azure desde una instantánea](app-service-web-restore-snapshots.md)
