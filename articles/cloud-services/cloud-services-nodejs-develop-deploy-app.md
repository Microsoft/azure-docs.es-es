---
title: Guía de introducción a Node.js
description: Aprenda a crear una sencilla aplicación web de Node.js e impleméntela en un servicio en la nube de Azure.
services: cloud-services
documentationcenter: nodejs
author: tgore03
ms.service: cloud-services
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 08/17/2017
ms.author: tagore
ms.custom: devx-track-javascript
ms.openlocfilehash: 489025d9cfdd6f1bdbc950e2bd1e7e4d835a4fff
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88079218"
---
# <a name="build-and-deploy-a-nodejs-application-to-an-azure-cloud-service"></a>Compilación e implementación de una aplicación Node.js en un Servicio en la nube de Azure

En este tutorial se muestra cómo crear una aplicación de Node.js simple con Servicio en la nube de Azure. Cloud Services son los bloques de compilación de aplicaciones en la nube escalables en Azure. Permiten la separación y la administración independiente de los componentes front-end y back-end de su aplicación, así como su escalación horizontal.  Cloud Services proporciona una máquina virtual dedicada y robusta para hospedar cada rol de forma fiable.

Para obtener más información sobre Cloud Services y sobre su comparación con Azure Websites y Azure Virtual Machines, consulte [Comparación entre Azure Websites, Cloud Services y Virtual Machines].

> [!TIP]
> ¿Desea compilar un sitio web sencillo? Si el escenario solo requiere un sencillo front-end para sitios web, considere la posibilidad de [utilizar una aplicación web ligera]. Puede actualizar a un Servicio en la nube más adelante, cuando su aplicación web sea más grande y sus requisitos cambien.

Siguiendo este tutorial, podrá compilar una aplicación web sencilla hospedada en un rol web. Utilizará el emulador de proceso para probar su aplicación localmente y, a continuación, la implementará con las herramientas de línea de comandos de PowerShell.

La aplicación es una aplicación sencilla de "Hola a todos":

![Un explorador web muestra la página web Hello World][A web browser displaying the Hello World web page]

## <a name="prerequisites"></a>Prerequisites
> [!NOTE]
> Este tutorial usa PowerShell de Azure, que requiere Windows.

* Instale y configure [Azure PowerShell].
* Descargar e instalar el [SDK de Azure para .NET 2.7]. En la configuración de la instalación, seleccione:
  * MicrosoftAzureAuthoringTools
  * MicrosoftAzureComputeEmulator

## <a name="create-an-azure-cloud-service-project"></a>Cree un proyecto del servicio de nube de Azure
Realice las siguientes tareas para crear un nuevo proyecto de Servicio en la nube de Azure, junto con scaffolding básico de Node.js:

1. Ejecute **Windows PowerShell** como administrador y, en el **menú Inicio** o en la **pantalla Inicio**, busque **Windows PowerShell**.
2. [Conecte PowerShell] a su suscripción.
3. Escriba el siguiente cmdlet de PowerShell para crear el proyecto:

   ```powershell
   New-AzureServiceProject helloworld
   ```

   ![Resultado del comando New-AzureService helloworld][The result of the New-AzureService helloworld command]

   El cmdlet **New-AzureServiceProject** genera una estructura básica para publicar una aplicación de Node.js en un servicio en la nube. Contiene archivos de configuración necesarios para la publicación en Azure. El cmdlet también cambia su directorio de trabajo al directorio del servicio.

   El cmdlet crea los siguientes archivos:

   * **ServiceConfiguration.Cloud.cscfg**, **ServiceConfiguration.Local.cscfg** y **ServiceDefinition.csdef**: estos archivos específicos de Azure son necesarios para publicar la aplicación. Para obtener más información, consulte [Información general de la creación de un servicio hospedado para Azure].
   * **deploymentSettings.json**: almacena la configuración local que utilizan los cmdlet de implementación de Azure PowerShell.

4. Escriba el siguiente comando para agregar un rol web nuevo:

   ```powershell
   Add-AzureNodeWebRole
   ```

   ![Salida del comando Add-AzureNodeWebRole][The output of the Add-AzureNodeWebRole command]

   El cmdlet **Add-AzureNodeWebRole** crea una aplicación de Node.js básica. También modifica los archivos **.csfg** y **.csdef** para agregar entradas de configuración al nuevo rol.

   > [!NOTE]
   > Si no especifica un nombre de rol, se usa el nombre predeterminado. Puede proporcionar un nombre como primer parámetro de cmdlet: `Add-AzureNodeWebRole MyRole`

La aplicación Node.js se define en el archivo **server.js**, que se encuentra en el directorio del rol web (**WebRole1** de forma predeterminada). Este es el código:

```js
var http = require('http');
var port = process.env.port || 1337;
http.createServer(function (req, res) {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello World\n');
}).listen(port);
```

Este código es básicamente el mismo que el ejemplo "Hola a todos" del sitio web de [nodejs.org] , excepto en que utiliza el número de puerto asignado por el entorno de la nube.

## <a name="deploy-the-application-to-azure"></a>Implementación de la aplicación en Azure

> [!NOTE]
> Para completar este tutorial, deberá tener una cuenta de Azure. Puede [activar sus beneficios de suscriptor a MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) o [registrarse para obtener una evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A85619ABF).

### <a name="download-the-azure-publishing-settings"></a>Descarga de la configuración de publicación de Azure
Para implementar una aplicación en Azure, en primer lugar es preciso descargar la configuración de publicación de la suscripción de Azure.

1. Ejecute el siguiente cmdlet de PowerShell de Azure:

    ```powershell
    Get-AzurePublishSettingsFile
    ```

   Este cmdlet utilizará su explorador para navegar hasta la página de descarga de la configuración de publicación. Es posible que se le solicite iniciar sesión con una cuenta de Microsoft. En ese caso, utilice una cuenta asociada a su suscripción de Azure.

   Guarde el perfil descargado en un ubicación de archivo a la que pueda tener acceso fácilmente.
2. Ejecute el siguiente cmdlet para importar el perfil de publicación que descargó:

    ```powershell
    Import-AzurePublishSettingsFile [path to file]
    ```

    > [!NOTE]
    > Después de importar la configuración de publicación, considere la posibilidad de eliminar el archivo .publishSettings descargado, ya que contiene información que podría permitir que alguien obtuviera acceso a su cuenta.

### <a name="publish-the-application"></a>Publicación de la aplicación
Para publicar, ejecute los siguientes comandos:

```powershell
$ServiceName = "NodeHelloWorld" + $(Get-Date -Format ('ddhhmm'))
Publish-AzureServiceProject -ServiceName $ServiceName  -Location "East US" -Launch
```

* **-ServiceName** especifica el nombre de la implementación. Debe ser un nombre exclusivo. De lo contrario, se producirá un error en el proceso de publicación. El comando **Get-Date** agrega una cadena de fecha y hora que debería hacer que el nombre sea único.
* **-Location** especifica el centro de datos en que se hospedará la aplicación. Para ver una lista de los centros de datos disponibles, utilice el cmdlet **Get-AzureLocation** .
* **-Launch** abre una ventana del explorador y navega hasta el servicio hospedado después de que se haya completado la implementación.

Después de que finalice satisfactoriamente el proceso de publicación, verá una respuesta similar a la siguiente:

![La salida del comando Publish-AzureService][The output of the Publish-AzureService command]

> [!NOTE]
> La implementación de la aplicación puede tardar varios minutos y la aplicación estará disponible cuando se publique.

Una vez finalizada la implementación, se abrirá una ventana del explorador y navegará hasta el servicio en la nube.

![Ventana del explorador que muestra la página hello world; la dirección URL indica que la página está hospedada en Azure.][A browser window displaying the hello world page; the URL indicates the page is hosted on Azure.]

La aplicación ya se está ejecutando en Azure.

El cmdlet **Publish-AzureServiceProject** realiza los siguientes pasos:

1. Crea un paquete de implementación. El paquete contiene todos los archivos en la carpeta de la aplicación.
2. Crea una nueva **cuenta de almacenamiento** si no hay ninguna disponible. La cuenta de almacenamiento de Azure se utiliza para almacenar el paquete de la aplicación durante la implementación. Una vez finalizada la implementación, puede eliminar de forma segura la cuenta de almacenamiento.
3. Crea un nuevo **servicio en la nube** si no hay ninguno disponible. Un **servicio en la nube** es el contenedor en el que se hospeda la aplicación cuando se implementa en Azure. Para obtener más información, consulte [Información general de la creación de un servicio hospedado para Azure].
4. Publica el paquete de implementación en Azure.

## <a name="stopping-and-deleting-your-application"></a>Detención y eliminación de la aplicación
Después de implementar su aplicación, es posible que desee deshabilitarla para evitar costes adicionales. Azure factura las instancias de rol web por hora consumida de tiempo de servidor. El tiempo de servidor se empieza a consumir una vez implementada su aplicación, incluso si las instancias no se están ejecutando y se encuentran detenidas.

1. En la ventana de Windows PowerShell, detenga la implementación del servicio creado en la sección anterior con el siguiente cmdlet:

    ```powershell
    Stop-AzureService
    ```

   La detención del servicio puede durar varios minutos. Una vez detenido el servicio, recibirá un mensaje que le avisará de su detención.

   ![Estado del comando Stop-AzureService][The status of the Stop-AzureService command]
2. Para eliminar el servicio, llame al siguiente cmdlet:

    ```powershell
    Remove-AzureService
    ```

   Cuando se le solicite, escriba **Y** para eliminar el servicio.

   La eliminación del servicio puede durar varios minutos. Una vez eliminado el servicio, recibirá un mensaje que le avisará de su eliminación.

   ![Estado del comando Remove-AzureService][The status of the Remove-AzureService command]

   > [!NOTE]
   > La eliminación del servicio no elimina la cuenta de almacenamiento que se creó al publicar por primera vez el servicio, por lo que se le seguirá facturando por el almacenamiento utilizado. Si nada más está utilizando el almacenamiento, puede eliminarlo.

## <a name="next-steps"></a>Pasos siguientes
Para más información, vea el [Centro para desarrolladores de Node.js].

<!-- URL List -->

[Comparación entre Azure Websites, Cloud Services y Virtual Machines]: /azure/architecture/guide/technology-choices/compute-decision-tree
[utilizar una aplicación web ligera]: ../app-service/quickstart-nodejs.md
[Azure PowerShell]: /powershell/azure/
[SDK de Azure para .NET 2.7]: https://www.microsoft.com/en-us/download/details.aspx?id=48178
[Conecte PowerShell]: /powershell/azure/
[nodejs.org]: https://nodejs.org/
[Información general de la creación de un servicio hospedado para Azure]: https://azure.microsoft.com/documentation/services/cloud-services/
[Centro para desarrolladores de Node.js]: https://azure.microsoft.com/develop/nodejs/

<!-- IMG List -->

[The result of the New-AzureService helloworld command]: ./media/cloud-services-nodejs-develop-deploy-app/node9.png
[The output of the Add-AzureNodeWebRole command]: ./media/cloud-services-nodejs-develop-deploy-app/node11.png
[A web browser displaying the Hello World web page]: ./media/cloud-services-nodejs-develop-deploy-app/node14.png
[The output of the Publish-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node19.png
[A browser window displaying the hello world page; the URL indicates the page is hosted on Azure.]: ./media/cloud-services-nodejs-develop-deploy-app/node21.png
[The status of the Stop-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node48.png
[The status of the Remove-AzureService command]: ./media/cloud-services-nodejs-develop-deploy-app/node49.png



