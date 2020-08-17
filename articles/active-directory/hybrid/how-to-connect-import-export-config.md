---
title: Cómo importar y exportar opciones de configuración de Azure AD Connect
description: En este artículo se describen las preguntas frecuentes sobre el aprovisionamiento en la nube.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/13/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: da80af9fe598186fa25d59601c9fa4faccb4286a
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87447050"
---
# <a name="import-and-export-azure-ad-connect-configuration-settings-public-preview"></a>Importar y exportar opciones de configuración de Azure AD Connect (versión preliminar pública)

Las implementaciones de Azure Active Directory (Azure AD) Connect varían de una instalación en modo Rápido del bosque único a implementaciones complejas que se sincronizan entre varios bosques mediante reglas de sincronización personalizadas. Debido al gran número de opciones y mecanismos de configuración, es esencial comprender qué opciones están en vigor para así poder implementar rápidamente un servidor con una configuración idéntica. Esta característica presenta la capacidad de catalogar la configuración de un servidor de sincronización determinado e importar la configuración en una nueva implementación. Se pueden comparar distintas instantáneas de configuración de sincronización para visualizar fácilmente las diferencias entre dos servidores, o del mismo servidor a lo largo del tiempo.

Cada vez que se cambia la configuración desde el asistente de Azure AD Connect, un nuevo archivo de configuración JSON con marca de tiempo se exporta automáticamente a  **%ProgramData%\AADConnect**. El nombre del archivo de configuración tiene el formato **Applied-SynchronizationPolicy-*.JSON**, donde la última parte del nombre de archivo es una marca de tiempo.

> [!IMPORTANT]
> Solo se exportan automáticamente los cambios realizados por Azure AD Connect. Todo cambio realizado mediante PowerShell, Synchronization Service Manager o el Editor de reglas de sincronización debe exportarse a petición según sea necesario para mantener una copia actualizada. La exportación a petición también se puede usar para colocar una copia de la configuración en una ubicación segura para fines de recuperación ante desastres.

## <a name="export-azure-ad-connect-settings"></a>Exportar la configuración de Azure AD Connect 

Para ver un resumen de los valores de configuración, abra la herramienta Azure AD Connect y seleccione la tarea adicional denominada **Ver o exportar la configuración actual**. Se muestra un resumen rápido de la configuración, junto con la capacidad de exportar la configuración completa del servidor.

De forma predeterminada, la configuración se exporta a **%ProgramData%\AADConnect**. También puede guardar la configuración en una ubicación protegida para garantizar la disponibilidad si se produce un desastre. La configuración se exporta con el formato de archivo JSON y no debe crearse ni editarse manualmente para garantizar la coherencia lógica. La importación de un archivo creado o editado manualmente no se admite y puede provocar resultados inesperados.

## <a name="import-azure-ad-connect-settings"></a>Importar la configuración de Azure AD Connect

Para importar una configuración anteriormente exportada:
 
1. Instale **Azure AD Connect** en un nuevo servidor.
1. Seleccione la opción **Personalizar** después de la página de **bienvenida**.
1. Seleccione **Importar configuración de sincronización**. Busque el archivo de configuración JSON anteriormente exportado.
1. Seleccione **Instalar**.

   ![Captura de pantalla que muestra la pantalla para instalar componentes necesarios](media/how-to-connect-import-export-config/import1.png)

> [!NOTE]
> Invalide la configuración de esta página, como el uso de SQL Server en lugar de LocalDB o el uso de una cuenta de servicio existente en lugar de la VSA predeterminada. Esta configuración no se importa desde el archivo de configuración. Está allí para fines de comparación e información.

### <a name="import-installation-experience"></a>Experiencia de instalación de la importación 

La experiencia de instalación de la importación sigue siendo sencilla, ya que cuenta con entradas mínimas del usuario para facilitar la reproducibilidad de un servidor existente.

Estos son los únicos cambios que se pueden realizar durante la experiencia de instalación. Todos los demás cambios se pueden realizar después de la instalación desde el asistente de Azure AD Connect:
- **Credenciales de Azure Active Directory**: de forma predeterminada, se sugiere el nombre de cuenta del administrador global de Azure que se usa para configurar el servidor original. Este *debe* cambiarse si quiere sincronizar la información con un nuevo directorio.
- **Inicio de sesión de usuario**: las opciones de inicio de sesión configuradas para el servidor original están seleccionadas de forma predeterminada y solicitarán automáticamente las credenciales u otra información necesaria durante la configuración. En casos excepcionales, puede ser necesario configurar un servidor con otras opciones para evitar cambiar el comportamiento del servidor activo. De lo contrario, seleccione **Siguiente** para usar la misma configuración.
- **Credenciales del directorio local**: para cada directorio local incluido en la configuración de sincronización, debe proporcionar credenciales para crear una cuenta de sincronización o suministrar una cuenta de sincronización personalizada creada previamente. Este procedimiento es idéntico a la experiencia de instalación limpia, con la excepción de que no se pueden agregar ni quitar directorios.
- **Opciones de configuración**: al igual que con una instalación limpia, puede optar por configurar las opciones iniciales de si quiere iniciar la sincronización automática o habilitar el modo de almacenamiento provisional. La principal diferencia es que el modo de almacenamiento provisional está habilitado intencionadamente de forma predeterminada para permitir la comparación de los resultados de la configuración y la sincronización antes de exportar activamente los resultados a Azure.

![Captura de pantalla que muestra la pantalla para conectar los directorios](media/how-to-connect-import-export-config/import2.png)

> [!NOTE]
> Solo un servidor de sincronización puede estar en el rol principal y exportar activamente los cambios de configuración a Azure. Todos los demás servidores deben colocarse en Modo provisional.

## <a name="migrate-settings-from-an-existing-server"></a>Migración de la configuración desde un servidor existente 

Si un servidor existente no admite la administración de la configuración, puede optar por actualizar el servidor en contexto o migrar la configuración para su uso en un nuevo servidor de almacenamiento provisional.

La migración requiere la ejecución de un script de PowerShell que extraiga la configuración existente para su uso en una nueva instalación. Use este método para catalogar la configuración del servidor existente y, a continuación, aplicarla a un servidor de almacenamiento provisional recién instalado. Comparar los valores del servidor original con el servidor recién creado le permitirá visualizar rápidamente los cambios entre los servidores. Como siempre, siga el proceso de certificación de su organización para asegurarse de que no se requiera ninguna configuración adicional.

### <a name="migration-process"></a>Proceso de migración 
Migración de la configuración:

1. Inicie **AzureADConnect.msi** en el nuevo servidor de almacenamiento provisional y deténgase en la página **principal** de Azure AD Connect.

1. Copie **MigrateSettings.ps1** del directorio Microsoft Azure AD Connect\Tools en una ubicación del servidor existente. Un ejemplo es C:\setup, donde "setup" es un directorio creado en el servidor existente.

   ![Captura de pantalla que muestra los directorios de Azure AD Connect.](media/how-to-connect-import-export-config/migrate1.png)

1. Ejecute el script tal como se muestra a continuación y guarde todo el directorio de configuración del servidor de nivel inferior. Copie este directorio en el nuevo servidor de almacenamiento provisional. Debe copiar toda la carpeta **Exported-ServerConfiguration-** * al nuevo servidor.

   ![Captura de pantalla que muestra el script en Windows PowerShell.](media/how-to-connect-import-export-config/migrate2.png)
   ![Captura de pantalla que muestra la copia de la carpeta Exported-ServerConfiguration-*.](media/how-to-connect-import-export-config/migrate3.png)

1. Haga doble clic en el icono de **Azure AD Connect** en el escritorio para iniciarlo. Acepte los términos de licencia del software de Microsoft y seleccione **Personalizar** en la siguiente página.
1. Seleccione la casilla **Importar configuración de sincronización**. Seleccione **Examinar** para examinar la carpeta Exported-ServerConfiguration-* que se ha copiado. Seleccione MigratedPolicy.json para importar la configuración migrada.

   ![Captura de pantalla que muestra la opción para importar la configuración de la sincronización.](media/how-to-connect-import-export-config/migrate4.png)

## <a name="post-installation-verification"></a>Comprobación posterior a la instalación 

Comparar el archivo de configuración importado originalmente con el archivo de configuración exportado del servidor recién implementado, es un paso esencial para comprender las diferencias entre la implementación prevista y la implementación resultante. Usar su aplicación favorita para la comparación de texto en paralelo permite obtener una visualización instantánea que resalta rápidamente cualquier cambio deseado o accidental.

Si bien ahora se han eliminado muchos pasos de configuración que antes eran manuales, debe seguir el proceso de certificación de su organización para asegurarse de que no se requiere ninguna configuración adicional. Esta configuración puede producirse si usa la configuración avanzada, ya que no se captura actualmente en la versión preliminar pública de la administración de configuración.

Estas son las limitaciones conocidas:
- **Reglas de sincronización**: la prioridad de una regla personalizada debe estar en el rango reservado de 0 a 99 para evitar conflictos con las reglas estándar de Microsoft. La colocación de una regla personalizada fuera del rango reservado puede dar lugar a que la regla personalizada se cambie a medida que se agreguen reglas estándar a la configuración. Se producirá un problema similar si la configuración contiene reglas estándar modificadas. Se recomienda no modificar una regla estándar; además, es probable que la ubicación de la regla sea incorrecta.
- **Escritura diferida de dispositivo**: estos valores están catalogados. Actualmente estos valores no se aplican durante la configuración. Si se ha habilitado la escritura diferida de dispositivo para el servidor original, debe configurar manualmente la característica en el servidor implementado recientemente.
- **Tipos de objeto sincronizados**: aunque es posible restringir la lista de tipos de objeto sincronizados (usuarios, contactos, grupos, etc.) mediante Synchronization Service Manager, esta característica no se admite actualmente a través de la configuración de sincronización. Después de completar la instalación, debe volver a aplicar manualmente la configuración avanzada.
- **Perfiles de ejecución personalizados**: aunque es posible modificar el conjunto predeterminado de perfiles de ejecución mediante Synchronization Service Manager, esta característica no se admite actualmente a través de la configuración de sincronización. Después de completar la instalación, debe volver a aplicar manualmente la configuración avanzada.
- **Configurar la jerarquía de aprovisionamiento**: esta característica avanzada de Synchronization Service Manager no se admite a través de la configuración de sincronización. Se debe volver a configurar manualmente después de finalizar la implementación inicial.
- **Autenticación de los Servicios de federación de Active Directory (AD FS) y PingFederate**: los métodos de inicio de sesión asociados a estas características de autenticación se preseleccionan automáticamente. Debe proporcionar de forma interactiva todos los demás parámetros de configuración necesarios.
- **Una regla de sincronización personalizada deshabilitada se importará como "habilitada"** : una regla de sincronización personalizada deshabilitada se importará como "habilitada". Asegúrese de deshabilitarla también en el nuevo servidor.

 ## <a name="next-steps"></a>Pasos siguientes

- [Hardware y requisitos previos](how-to-connect-install-prerequisites.md) 
- [Configuración rápida](how-to-connect-install-express.md)
- [Configuración personalizada](how-to-connect-install-custom.md)
- [Instalación de agentes de Azure AD Connect Health](how-to-connect-health-agent-install.md) 
