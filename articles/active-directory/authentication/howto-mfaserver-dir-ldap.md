---
title: 'Autenticación de LDAP y Servidor Azure MFA: Azure Active Directory'
description: Implementación de Autenticación de LDAP y Servidor Azure Multi-Factor Authentication.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/11/2018
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0f15fcd2d8e42d56deb50ee7eaae7472dae6bf6c
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88949497"
---
# <a name="ldap-authentication-and-azure-multi-factor-authentication-server"></a>Servidor de autenticación LDAP y Azure Multi-Factor Authentication

De forma predeterminada, Servidor Azure Multi-Factor Authentication está configurado para importar o sincronizar usuarios desde Active Directory. Sin embargo, se puede configurar para enlazar a directorios LDAP diferentes, por ejemplo, un directorio ADAM o un controlador de dominio de Active Directory específico. Cuando se conecta a un directorio mediante LDAP, el Servidor Azure Multi-Factor Authentication puede actuar como proxy de LDAP para realizar autenticaciones. También permite el uso del enlace LDAP como destino RADIUS para la autenticación previa de usuarios con autenticación de IIS o la autenticación principal en el portal de usuario de Azure MFA.

Para usar Azure Multi-Factor Authentication como proxy de LDAP, inserte el Servidor Azure Multi-Factor Authentication entre el cliente LDAP (por ejemplo, el dispositivo VPN o la aplicación) y el servidor de directorio LDAP. El Servidor Azure Multi-Factor Authentication se debe configurar para comunicarse con los servidores de cliente y el directorio LDAP. En esta configuración, el servidor de Azure Multi-Factor Authentication acepta las solicitudes LDAP de servidores y aplicaciones cliente y las reenvía al servidor de directorio LDAP de destino para validar las credenciales principales. Si el directorio LDAP valida las credenciales principales, Azure Multi-Factor Authentication realiza una segunda comprobación de identidad y envía una respuesta de vuelta al cliente LDAP. El proceso entero de autenticación se completará correctamente solo si tanto la autenticación del servidor LDAP como el segundo paso de la comprobación se realizan satisfactoriamente.

> [!IMPORTANT]
> A partir del 1 de julio de 2019, Microsoft ya no ofrecerá el servidor MFA para implementaciones nuevas. Los clientes nuevos que quieran exigir la autenticación multifactor (MFA) durante los eventos de inicio de sesión deben usar Azure Multi-Factor Authentication basado en la nube.
>
> Para empezar a trabajar con MFA basado en la nube, consulte [Tutorial: Protección de eventos de inicio de sesión de usuario con Azure Multi-Factor Authentication](tutorial-enable-azure-mfa.md).
>
> Los clientes existentes que hayan activado el Servidor MFA antes del 1 de julio de 2019 podrán descargar la versión más reciente y las actualizaciones futuras, así como generar credenciales de activación como de costumbre.

## <a name="configure-ldap-authentication"></a>Configuración de la autenticación LDAP

Para configurar la autenticación LDAP, instale al servidor Azure Multi-Factor Authentication en un servidor Windows. Utilice el siguiente procedimiento:

### <a name="add-an-ldap-client"></a>Agregue un cliente LDAP.

1. En el Servidor Azure Multi-Factor Authentication, seleccione el icono Autenticación LDAP en el menú izquierdo.
2. Active la casilla **Habilitar autenticación LDAP**.

   ![Autenticación de LDAP en el servidor MFA](./media/howto-mfaserver-dir-ldap/ldap2.png)

3. En la pestaña Clientes, cambie el puerto TCP y SSL (TLS) si el servicio LDAP de Azure Multi-Factor Authentication debe enlazarse con puertos no estándar para escuchar solicitudes LDAP.
4. Si tiene previsto usar LDAPS entre el cliente y el Servidor Azure Multi-Factor Authentication, se debe instalar un certificado TLS/SSL en el mismo servidor que el servidor de MFA. Haga clic en **Examinar** junto al cuadro de certificado SSL (TLS) y seleccione el certificado que se usará para la conexión segura.
5. Haga clic en **Agregar**.
6. En el cuadro de diálogo Agregar cliente LDAP, escriba la dirección IP del dispositivo, servidor o aplicación que se autenticará en el servidor y un nombre de aplicación (opcional). El nombre de la aplicación aparece en los informes de Azure Multi-Factor Authentication y puede mostrarse en los mensajes de autenticación SMS o de aplicación móvil.
7. Active la casilla **Requerir coincidencia de usuario de Azure Multi-Factor Authentication** si todos los usuarios se importaron o importarán al servidor y están sujetos a la verificación en dos pasos. Si aún no se importó al servidor un número significativo de usuarios o se van a excluir de la verificación en dos pasos, deje la casilla desactivada. Consulte el archivo de ayuda del servidor de MFA para obtener información adicional sobre de esta característica.

Repita estos pasos para agregar más clientes LDAP.

### <a name="configure-the-ldap-directory-connection"></a>Configuración de la conexión del directorio LDAP

Cuando se configura la Azure Multi-Factor Authentication para recibir autenticaciones LDAP, debe entregar dichas autenticaciones en el directorio LDAP. Por lo tanto, en la pestaña Destino solo muestra una única opción atenuada para usar un destino LDAP.

> [!NOTE]
> No se garantiza que la integración de directorios funcione con directorios que no sean Active Directory Domain Services.

1. Para configurar la conexión del directorio LDAP, haga clic en el icono de **Integración de directorios**.
2. En la pestaña Configuración, seleccione el botón de radio **Usar la configuración LDAP específica**.
3. Seleccione **Editar...**
4. En el cuadro de diálogo Editar configuración de LDAP, rellene los campos con la información necesaria para conectarse al directorio LDAP. Las descripciones de los campos se incluyen en el archivo de ayuda del Servidor Azure Multi-Factor Authentication.

    ![Configuración de LDAP de integración de directorio](./media/howto-mfaserver-dir-ldap/ldap.png)

5. Pruebe la conexión LDAP haciendo clic en el botón **Probar**.
6. Si la prueba de conexión LDAP se realizó correctamente, haga clic en el botón **Aceptar**.
7. Haga clic en la pestaña **Filtros**. El servidor está preconfigurado para cargar contenedores, grupos de seguridad y usuarios de Active Directory. Si desea enlazar a un directorio LDAP diferente, probablemente necesitará editar los filtros mostrados. Haga clic en el vínculo **Ayuda** para más información acerca de los filtros.
8. Haga clic en la pestaña **Attributes** (Atributos). El servidor está preconfigurado para asignar atributos de Active Directory.
9. Si desea enlazar a un directorio LDAP diferente o cambiar las asignaciones de atributos configuradas previamente, haga clic en **Editar...**
10. En el cuadro de diálogo Editar atributos, modifique las asignaciones de atributos LDAP para el directorio. Los nombres de atributos se pueden escribir o seleccionar haciendo clic en el botón **...** que se encuentra situado junto a cada campo. Haga clic en el vínculo **Ayuda** para más información sobre los atributos.
11. Haga clic en el botón **Aceptar** .
12. Haga clic en el icono **Configuración de la empresa** y seleccione la pestaña **Resolución de nombre de usuario**.
13. Si desea conectarse a Active Directory desde un servidor unido a un dominio, deje el botón de opción **Usar los identificadores de seguridad de Windows (SID) para que coincidan con los nombres de usuario** seleccionado. De lo contrario, seleccione el botón de radio **Usar el atributo del identificador único LDAP para los nombres de usuario coincidentes**.

Cuando el botón de opción **Usar el atributo del identificador único LDAP para los nombres de usuario coincidentes** está seleccionado, el Servidor Azure Multi-factor Authentication trata de resolver cada nombre de usuario en un identificador único en el directorio LDAP. Se realiza una búsqueda LDAP en los atributos de Nombre de usuario definidos en la pestaña Integración de directorios -> Atributos. Cuando un usuario se autentica, el nombre de usuario se resuelve en un identificador único en el directorio LDAP. El identificador único se usa para hacer coincidir el usuario en el archivo de datos de Azure Multi-Factor Authentication. Esto permite comparaciones que no distinguen mayúsculas de minúsculas, así como formatos largo y corto de nombres de usuario.

Después de realizar estos pasos, el servidor de MFA comienza a escuchar en los puertos configurados para las solicitudes de acceso LDAP de los clientes configurados y actúa como proxy para esas solicitudes en el directorio LDAP para la autenticación.

## <a name="configure-ldap-client"></a>Configuración del cliente LDAP

Para configurar el cliente LDAP, use las siguientes directrices:

* Configure el dispositivo, el servidor o la aplicación para autenticarse mediante LDAP en el servidor de Azure Multi-Factor Authentication como si fuese el directorio LDAP. Use la misma configuración que usaría normalmente para conectarse directamente a su directorio LDAP, excepto el nombre del servidor o la dirección IP que será la del Servidor Azure Multi-Factor Authentication.
* Configure el tiempo de espera de LDAP entre 30 y 60 segundos para que haya tiempo para validar las credenciales del usuario con el directorio LDAP, realizar la comprobación en dos pasos, recibir su respuesta y luego responder a la solicitud de acceso LDAP.
* Si usa LDAPS, el dispositivo o el servidor que realiza las consultas LDAP debe confiar en el certificado TLS/SSL instalado en el servidor Azure Multi-Factor Authentication.
