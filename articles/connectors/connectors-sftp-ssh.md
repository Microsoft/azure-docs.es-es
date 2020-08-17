---
title: Conexión al servidor SFTP con SSH
description: Automatice las tareas que supervisan, crean, administran, envían y reciben archivos en un servidor SFTP mediante SSH y Azure Logic Apps.
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 07/20/2020
tags: connectors
ms.openlocfilehash: f3de582ff69dbd57aa4692fd5c3901602569cf9e
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87286621"
---
# <a name="monitor-create-and-manage-sftp-files-by-using-ssh-and-azure-logic-apps"></a>Supervisión, creación y administración de archivos SFTP mediante SSH y Azure Logic Apps

Para automatizar las tareas que supervisan, crean, envían y reciben archivos en un servidor [Secure File Transfer Protocol (SFTP)](https://www.ssh.com/ssh/sftp/) mediante el protocolo [Secure Shell (SSH)](https://www.ssh.com/ssh/protocol/), puede crear y automatizar los flujos de trabajo de integración mediante Azure Logic Apps y el conector SFTP-SSH. SFTP es un protocolo de red que proporciona acceso a archivos, transferencia de archivos y administración de archivos a través de cualquier flujo de datos confiable.

> [!NOTE]
> El conector SFTP-SSH no es compatible actualmente con estos servidores SFTP:
> 
> * IBM DataPower
> * MessageWay
> * OpenText Secure MFT
> * OpenText GXS

Estas son algunas tareas de ejemplo que se pueden automatizar:

* Supervisar cuándo se agregan o cambian archivos
* Obtener, crear, copiar, cambiar el nombre, actualizar, enumerar y eliminar archivos.
* Crear carpetas.
* Obtener contenido de archivos y metadatos
* Extraer archivos en carpetas

Puede usar desencadenadores que supervisen eventos en el servidor SFTP y permitir que la salida esté disponible para otras acciones. Puede usar acciones que realicen diversas tareas en el servidor SFTP. También puede hacer que otras acciones de la aplicación lógica usen la salida de las acciones SFTP. Por ejemplo, si recupera regularmente archivos del servidor SFTP, puede enviar por correo electrónico alertas sobre esos archivos y su contenido mediante el conector Office 365 Outlook o el conector Outlook.com. Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)

Para conocer las diferencias entre el conector SFTP-SSH y el conector SFTP, revise [Comparación de SFTP-SSH y SFTP](#comparison) más adelante en este tema.

## <a name="limits"></a>Límites

* Las acciones de SFTP-SSH que admiten la [fragmentación](../logic-apps/logic-apps-handle-large-messages.md) pueden administrar archivos de hasta 1 GB, mientras que las acciones de SFTP-SSH que no admiten la fragmentación pueden administrar archivos de hasta 50 MB. Aunque el tamaño de fragmento predeterminado es de 15 MB, este tamaño puede cambiar dinámicamente, empezando por 5 MB y aumentando gradualmente hasta 50 MB, como máximo, en función de factores como la latencia de red, el tiempo de respuesta del servidor, etc.

  > [!NOTE]
  > En el caso de las aplicaciones lógicas de un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la versión con la etiqueta ISE necesita fragmentarse para usar los [límites de mensajes de ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits).

  Puede invalidar este comportamiento adaptable en caso de [especificar un tamaño de fragmento constante](#change-chunk-size) para usar en su lugar. Este tamaño puede oscilar entre 5 MB y 50 MB. Por ejemplo, supongamos que tiene un archivo de 45 MB y una red que admite ese tamaño de archivo sin latencia. La fragmentación adaptable da como resultado varias llamadas, en lugar de una llamada. Para reducir el número de llamadas, puede intentar establecer un tamaño de fragmento de 50 MB. En un escenario diferente, si se agota el tiempo de espera de la aplicación lógica, por ejemplo, al usar fragmentos de 15 MB, puede intentar reducir el tamaño a 5 MB.

  El tamaño del fragmento está asociado a una conexión, lo que significa que puede usar la misma conexión para las acciones que admiten la fragmentación y, a continuación, para las acciones que no la admiten. En este caso, el tamaño del fragmento para las acciones que no admiten fragmentación abarca de 5 MB a 50 MB. En esta tabla se muestra qué acciones de SFTP-SSH admiten fragmentación:

  | Acción | Compatibilidad con la fragmentación | Invalidación de la compatibilidad con el tamaño de fragmento |
  |--------|------------------|-----------------------------|
  | **Copiar archivo** | No | No es aplicable |
  | **Crear archivo** | Sí | Sí |
  | **Crear carpeta** | No aplicable | No aplicable |
  | **Eliminar archivo** | No aplicable | No aplicable |
  | **Extraer archivo en la carpeta** | No aplicable | No aplicable |
  | **Obtener contenido de archivo** | Sí | Sí |
  | **Obtener contenido de archivo mediante la ruta de acceso** | Sí | Sí |
  | **Obtener metadatos de archivo** | No aplicable | No aplicable |
  | **Obtener metadatos de archivo mediante la ruta de acceso** | No aplicable | No aplicable |
  | **Enumerar archivos de la carpeta** | No aplicable | No aplicable |
  | **Cambiar nombre de archivo** | No aplicable | No aplicable |
  | **Actualizar archivo** | No | No aplicable |
  ||||

* Los desencadenadores SFTP-SSH no admiten la fragmentación de mensajes. Cuando se solicita el contenido del archivo, los desencadenadores seleccionan solo los archivos que tienen un tamaño de 15 MB o menos. Para obtener archivos de más de 15 MB, siga este patrón en su lugar:

  1. Use un desencadenador SFTP-SSH que devuelva solo las propiedades de archivo, como **Cuando se agrega o modifica un archivo (solo propiedades)** .

  1. Siga el desencadenador con la acción **Obtener contenido de archivo** SFTP-SSH, que lee el archivo completo y utiliza implícitamente la fragmentación de mensajes.

<a name="comparison"></a>

## <a name="compare-sftp-ssh-versus-sftp"></a>Comparación de SFTP-SSH y SFTP

Estas son otras diferencias importantes entre el conector SFTP-SSH y el conector SFTP donde el conector SFTP-SSH tiene estas funcionalidades:

* Usa la biblioteca [SSH.NET](https://github.com/sshnet/SSH.NET), que es una biblioteca de Secure Shell (SSH) de código abierto que admite .NET.

* Proporciona la acción **Crear carpeta**, que crea una carpeta en la ruta de acceso especificada en el servidor SFTP.

* Proporciona la acción **Cambiar nombre de archivo**, que cambia el nombre de un archivo en el servidor SFTP.

* Almacena en caché la conexión al servidor SFTP *durante hasta 1 hora*, lo que mejora el rendimiento y reduce el número de intentos de conexión al servidor. Para establecer la duración de este comportamiento de almacenamiento en caché, modifique la propiedad [**ClientAliveInterval**](https://man.openbsd.org/sshd_config#ClientAliveInterval) en la configuración de SSH del servidor SFTP.

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* La dirección del servidor SFTP y las credenciales de cuenta, que permiten que la aplicación lógica acceda a la cuenta de SFTP. También necesita acceso a una clave privada SSH y a la contraseña de clave privada SSH. Para usar la fragmentación al cargar archivos de gran tamaño, necesita permisos de lectura y escritura para la carpeta raíz del servidor SFTP. De lo contrario, recibirá un error "401 no autorizado".

  > [!IMPORTANT]
  >
  > El conector SFTP-SSH *solo* admite estas claves privadas, formatos, algoritmos y huellas digitales:
  >
  > * **Formatos de clave privada**: claves RSA (Rivest Shamir Adleman) y DSA (Digital Signature Algorithm) en formatos OpenSSH y ssh.com. Si la clave privada tiene el formato de archivo PuTTy (.ppk), en primer lugar [convierta la clave al formato de archivo OpenSSH (.pem)](#convert-to-openssh).
  >
  > * **Algoritmos de cifrado**: DES-EDE3-CBC, DES-EDE3-CFB, DES-CBC, AES-128-CBC, AES-192-CBC y AES-256-CBC
  >
  > * **Huella digital**: MD5
  >
  > Después de agregar el desencadenador o la acción SFTP-SSH que quiera a la aplicación lógica, debe proporcionar información de conexión para el servidor SFTP. Cuando proporcione su clave privada SSH para esta conexión, ***no escriba ni edite manualmente la clave***, ya que podría provocar un error en la conexión. En su lugar, asegúrese de ***copiar la clave*** del archivo de clave privada SSH y ***pegar*** esa clave en los detalles de la conexión. 
  > Para obtener más información, consulte la sección [Conectarse a SFTP con SSH](#connect) que se detalla más adelante en este artículo.

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* La aplicación lógica desde donde quiere acceder a la cuenta de SFTP. Para comenzar con un desencadenador SFTP-SSH, [cree una aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para usar una acción SFTP-SSH, inicie la aplicación lógica con otro desencadenador, por ejemplo, el desencadenador **Recurrence**.

## <a name="how-sftp-ssh-triggers-work"></a>Cómo funcionan los desencadenadores SFTP-SSH

El método de funcionamiento de los desencadenadores SFTP-SSH es sondear el sistema de archivos SFTP y buscar cualquier archivo que se haya cambiado desde el último sondeo. Algunas herramientas le permiten conservar la marca de tiempo cuando cambian los archivos. En estos casos, tiene que deshabilitar esta característica para que el desencadenador funcione. Estas son algunas opciones de configuración habituales:

| Cliente SFTP | Acción |
|-------------|--------|
| WinSCP | Vaya a **Options** > **Preferences** > **Transfer** > **Edit** > **Preserve timestamp** > **Disable** (Opciones > Preferencias > Transferir > Editar > Conservar marca de tiempo >Deshabilitar). |
| FileZilla | Vaya a **Transfer** > **Preserve timestamps of transferred files** > **Disable** (Transferir > Conservar marca de tiempo de archivos transferidos > Deshabilitar). |
|||

Cuando un desencadenador encuentra un nuevo archivo, el desencadenador comprueba que el nuevo archivo se ha escrito por completo y no parcialmente. Por ejemplo, un archivo podría tener los cambios en curso cuando el desencadenador comprueba el servidor de archivos. Para evitar devolver un archivo parcialmente escrito, el desencadenador anota la marca de tiempo del archivo que tiene cambios recientes, pero no devuelve inmediatamente dicho archivo. El desencadenador devuelve el archivo solo al volver a sondear el servidor. En ocasiones, este comportamiento puede provocar un retraso de hasta dos veces el intervalo de sondeo del desencadenador.

<a name="convert-to-openssh"></a>

## <a name="convert-putty-based-key-to-openssh"></a>Convertir la clave basada en PuTTy en OpenSSH

Si la clave privada está en formato PuTTy, que usa la extensión de nombre de archivo. ppk (clave privada de PuTTy), primero debe convertir la clave al formato OpenSSH, que usa la extensión de nombre de archivo. PEM (correo de privacidad mejorada).

### <a name="unix-based-os"></a>Sistema operativo basado en UNIX

1. Si las herramientas de PuTTy no están ya instaladas en el sistema, puede instalarlas ahora; por ejemplo:

   `sudo apt-get install -y putty`

1. Ejecute este comando, que crea un archivo que puede usar con el conector SFTP-SSH:

   `puttygen <path-to-private-key-file-in-PuTTY-format> -O private-openssh -o <path-to-private-key-file-in-OpenSSH-format>`

   Por ejemplo:

   `puttygen /tmp/sftp/my-private-key-putty.ppk -O private-openssh -o /tmp/sftp/my-private-key-openssh.pem`

### <a name="windows-os"></a>SO Windows

1. Si aún no lo ha hecho, [descargue la última herramienta de generación de PuTTY (puttygen.exe)](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) y, a continuación, inicie la herramienta.

1. En esta pantalla, seleccione **Load** (Cargar).

   ![Seleccione "Load" (Cargar).](./media/connectors-sftp-ssh/puttygen-load.png)

1. Busque el archivo de clave privada en formato PuTTY y seleccione **Open** (Abrir).

1. En el menú **Conversions** (Conversiones), seleccione **Export OpenSSH key** (Exportar clave OpenSSH).

   ![Seleccione "Export OpenSSH key" (Exportar clave OpenSSH)](./media/connectors-sftp-ssh/export-openssh-key.png)

1. Guarde el archivo de clave privada con la extensión de nombre de archivo `.pem`.

## <a name="considerations"></a>Consideraciones

En esta sección se describen las consideraciones para revisar las acciones y los desencadenadores de este conector.

<a name="create-file"></a>

### <a name="create-file"></a>Crear archivo

Para crear un archivo en el servidor SFTP, puede usar la acción **Crear archivo** de SFTP-SSH. Cuando esta acción crea el archivo, el servicio Logic Apps también llama automáticamente al servidor SFTP para obtener los metadatos del archivo. Sin embargo, si mueve el archivo recién creado antes de que el servicio Logic Apps pueda realizar la llamada para obtener los metadatos, recibirá un mensaje de error `404`, `'A reference was made to a file or folder which does not exist'`. Para omitir la lectura de los metadatos del archivo tras la creación de este, siga los pasos para [agregar y establecer la propiedad **Obtener todos los metadatos del archivo** en **No**](#file-does-not-exist).

<a name="connect"></a>

## <a name="connect-to-sftp-with-ssh"></a>Conexión a SFTP con SSH

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra la aplicación lógica en el diseñador de aplicaciones lógicas, si aún no lo ha hecho.

1. Para las aplicaciones lógicas en blanco, en el cuadro de búsqueda, escriba `sftp ssh` como filtro. En la lista de desencadenadores, seleccione el que desee.

   O bien

   Para las aplicaciones lógicas existentes, en el último paso donde desea agregar una acción, seleccione **Nuevo paso**. En el cuadro de búsqueda, escriba `sftp ssh` como filtro. En la lista de acciones, seleccione la que desee.

   Para agregar una acción entre un paso y otro, mueva el puntero sobre la flecha entre ellos. Seleccione el signo más ( **+** ) que aparece y, luego, seleccione **Agregar una acción**.

1. Proporcione la información necesaria para la conexión.

   > [!IMPORTANT]
   >
   > Al escribir la clave privada SSH en la propiedad **SSH private key**, siga estos pasos adicionales para asegurarse de proporcionar el valor completo y correcto para esta propiedad. Una clave no válida provoca un error de conexión.

   Aunque puede usar cualquier editor de texto, estos son los pasos de ejemplo que muestran cómo copiar y pegar correctamente la clave mediante Notepad.exe.

   1. Abra el archivo de clave privada SSH en un editor de texto. Estos pasos usan Bloc de notas como ejemplo.

   1. En el menú **Edición** del Bloc de notas, elija **Seleccionar todo**.

   1. Seleccione **Editar** > **Copiar**.

   1. En el desencadenador o la acción SFTP-SSH que ha agregado, pegue la clave *completa* que copió en la propiedad **SSH private key**, que admite varias líneas.  ***Asegúrese de pegar*** la clave. ***No especifique ni edite la clave manualmente***.

1. Cuando haya terminado de especificar los detalles de conexión, seleccione **Crear**.

1. Ahora, proporcione los detalles necesarios para el desencadenador o la acción seleccionados y continúe con la compilación del flujo de trabajo de la aplicación lógica.

<a name="change-chunk-size"></a>

## <a name="override-chunk-size"></a>Invalidación del tamaño de fragmento

Para invalidar el comportamiento adaptable predeterminado que usa la fragmentación, puede especificar un tamaño de fragmento constante de 5 MB a 50 MB.

1. En la esquina superior derecha de la acción, seleccione el botón de puntos suspensivos ( **...** ) y, a continuación, seleccione **Configuración**.

   ![Apertura de la configuración de SFTP-SSH](./media/connectors-sftp-ssh/sftp-ssh-connector-setttings.png)

1. En **Transferencia de contenido**, en la propiedad **Tamaño de fragmento**, escriba un valor entero de `5` a `50`, por ejemplo: 

   ![Especificación del tamaño de fragmento que se usará en su lugar](./media/connectors-sftp-ssh/specify-chunk-size-override-default.png)

1. Cuando haya finalizado, seleccione **Listo**.

## <a name="examples"></a>Ejemplos

<a name="file-added-modified"></a>

### <a name="sftp---ssh-trigger-when-a-file-is-added-or-modified"></a>SFTP - desencadenador SSH: When a file is added or modified (Cuando se agrega o modifica un archivo)

Este desencadenador inicia un flujo de trabajo de aplicación lógica cuando se agrega o se modifica un archivo en un servidor SFTP. Por ejemplo, puede agregar una condición que compruebe el contenido del archivo y obtenga el contenido en función de si cumple una condición especificada. Luego, puede agregar una acción que obtenga el contenido del archivo y lo coloque en una carpeta en el servidor SFTP.

**Ejemplo Enterprise**: Puede usar este desencadenador para supervisar nuevos archivos en una carpeta de SFTP que representan los pedidos de los clientes. Seguidamente, puede usar una acción SFTP, como **Get file content** para obtener el contenido del pedido para su posterior procesamiento y almacenar ese pedido en una base de datos de pedidos.

<a name="get-content"></a>

### <a name="sftp---ssh-action-get-file-content-using-path"></a>SFTP - acción SSH: Obtener contenido de archivo mediante la ruta de acceso

Esta acción obtiene el contenido de un archivo en un servidor SFTP especificando la ruta de acceso del archivo. Por ejemplo, puede agregar el desencadenador del ejemplo anterior y una condición que debe cumplir el contenido del archivo. Si la condición es verdadera, se puede ejecutar la acción que obtiene el contenido.

<a name="troubleshooting-errors"></a>

## <a name="troubleshoot-errors"></a>Solución de errores

En esta sección se describen posibles soluciones a errores o problemas comunes.

<a name="file-does-not-exist"></a>

### <a name="404-error-a-reference-was-made-to-a-file-or-folder-which-does-not-exist"></a>Error 404: "Se ha hecho referencia a un archivo o carpeta que no existe"

Este error puede producirse cuando la aplicación lógica crea un nuevo archivo en el servidor SFTP a través de la acción **Crear archivo** de SFTP-SSH, pero el archivo recién creado se mueve inmediatamente antes de que el servicio Logic Apps pueda obtener los metadatos del archivo. Cuando la aplicación lógica ejecuta la acción **Crear archivo**, el servicio Logic Apps también llama automáticamente al servidor SFTP para obtener los metadatos del archivo. Sin embargo, si se mueve el archivo, el servicio Logic Apps ya no podrá encontrarlo, por lo que obtendrá el mensaje de error `404`.

Si no puede evitar o retrasar el traslado del archivo, puede omitir la lectura de los metadatos del mismo después de la creación del archivo en su lugar siguiendo estos pasos:

1. En la acción **Crear archivo**, abra la lista **Agregar nuevo parámetro**, seleccione la propiedad **Obtener todos los metadatos del archivo** y establezca el valor en **No**.

1. Si necesita estos metadatos del archivo más adelante, puede usar la acción **Obtener metadatos de archivo**.

## <a name="connector-reference"></a>Referencia de conectores

Si necesita más detalles técnicos sobre este conector, como los desencadenadores, las acciones y los límites que se describen en el archivo de Swagger del conector, vea la [página de referencia del conector](/connectors/sftpwithssh/).

> [!NOTE]
> En el caso de las aplicaciones lógicas de un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la versión con la etiqueta ISE necesita fragmentarse para usar los [límites de mensajes de ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)

