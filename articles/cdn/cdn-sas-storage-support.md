---
title: Uso de la red Azure CDN con SAS | Microsoft Docs
description: Azure CDN admite el uso de firma de acceso compartido (SAS) para conceder acceso limitado a los contenedores de almacenamiento privado.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 06/21/2018
ms.author: allensu
ms.openlocfilehash: d716b026159311c12341c30a8c32d5a9ecc6fa3f
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432743"
---
# <a name="using-azure-cdn-with-sas"></a>Uso de la red Azure CDN con SAS

Al configurar una cuenta de almacenamiento para la Red de entrega de contenido (CDN) de Azure para usar al contenido de la memoria caché, cualquier usuario que conozca las direcciones URL de los contenedores de almacenamiento tiene acceso de manera predeterminada a los archivos que se hayan cargado. Si quiere proteger los archivos de la cuenta de almacenamiento, puede cambiar el acceso de los contenedores de almacenamiento de público a privado. Pero si lo hace, nadie podrá tener acceso a los archivos. 

Si quiere conceder acceso limitado a los contenedores de almacenamiento privado, puede usar la característica de firma de acceso compartido (SAS) de la cuenta de Azure Storage. Una SAS es un identificador URI que concede derechos de acceso restringido a los recursos de Azure Storage sin exponer la clave de cuenta. Puede proporcionar una SAS a los clientes que no son de confianza para entregarles la clave de la cuenta de almacenamiento, pero a los que sí desea delegar acceso a ciertos recursos de la cuenta de almacenamiento. Mediante la distribución de un identificador URI de firma de acceso compartido a estos clientes, les concede acceso a un recurso durante un período de tiempo especificado.
 
Con una SAS, puede definir diversos parámetros de acceso a un blob, como horas de inicio y expiración, permisos (lectura o escritura) e intervalos IP. En este artículo se describe cómo usar SAS junto con la red Azure CDN. Para más información sobre la característica SAS, incluido cómo crearla y sus opciones de parámetros, consulte [Uso de firmas de acceso compartido (SAS)](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1).

## <a name="setting-up-azure-cdn-to-work-with-storage-sas"></a>Configuración de Azure CDN para trabajar con la característica SAS de almacenamiento
Se recomiendan las tres opciones siguientes para usar SAS con la red Azure CDN. En todas las opciones se presupone que ya creó una SAS en funcionamiento (consulte los requisitos previos). 
 
### <a name="prerequisites"></a>Requisitos previos
Para comenzar, cree una cuenta de almacenamiento y, luego, genere una SAS para el recurso. Puede generar dos tipos de firmas de acceso compartido: una SAS de servicio o una SAS de cuenta. Para más información, consulte [Tipos de firmas de acceso compartido](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1#types-of-shared-access-signatures).

Cuando haya generado un token de SAS, tiene acceso a su archivo de Blob Storage mediante la anexión de `?sv=<SAS token>` a la dirección URL. Esta dirección URL tiene el formato siguiente: 

`https://<account name>.blob.core.windows.net/<container>/<file>?sv=<SAS token>`
 
Por ejemplo:
 ```
https://democdnstorage1.blob.core.windows.net/container1/demo.jpg?sv=2017-07-29&ss=b&srt=co&sp=r&se=2038-01-02T21:30:49Z&st=2018-01-02T13:30:49Z&spr=https&sig=QehoetQFWUEd1lhU5iOMGrHBmE727xYAbKJl5ohSiWI%3D
```

Para más información sobre cómo establecer los parámetros, consulte [Consideraciones sobre los parámetros de SAS](#sas-parameter-considerations) y [Parámetros de la firma de acceso compartido](https://docs.microsoft.com/azure/storage/common/storage-sas-overview#how-a-shared-access-signature-works).

![Configuración de SAS de la red CDN](./media/cdn-sas-storage-support/cdn-sas-settings.png)

### <a name="option-1-using-sas-with-pass-through-to-blob-storage-from-azure-cdn"></a>Opción 1: Uso de SAS con paso a través a Blob Storage desde Azure CDN

Esta opción es la más simple y solo usa un único token de SAS, que se pasa desde Azure CDN al servidor de origen.
 
1. Elija un punto de conexión, seleccione **Reglas de caché** y luego, en la lista **Almacenamiento en caché de cadenas de consulta**, elija **Almacenar en caché todas las URL únicas**.

    ![Reglas de caché de la red CDN](./media/cdn-sas-storage-support/cdn-caching-rules.png)

2. Después de configurar SAS en la cuenta de almacenamiento, debe usar el token de SAS con las direcciones URL del punto de conexión de CDN y del servidor de origen para acceder al archivo. 
   
   La dirección URL del punto de conexión de CDN resultante tiene el formato siguiente: `https://<endpoint hostname>.azureedge.net/<container>/<file>?sv=<SAS token>`

   Por ejemplo:   
   ```
   https://demoendpoint.azureedge.net/container1/demo.jpg/?sv=2017-07-29&ss=b&srt=c&sp=r&se=2027-12-19T17:35:58Z&st=2017-12-19T09:35:58Z&spr=https&sig=kquaXsAuCLXomN7R00b8CYM13UpDbAHcsRfGOW3Du1M%3D
   ```
   
3. Ajuste la duración de la caché mediante las reglas de caché o al agregar los encabezados `Cache-Control` en el servidor de origen. Debido a que Azure CDN trata el token de SAS como cadena de consulta sin formato, el procedimiento recomendado que debe seguirse es configurar una duración de la caché con la misma hora de expiración que la SAS o una anterior. De lo contrario, si un archivo se almacena en caché más tiempo que el que está activa la SAS, es posible acceder al archivo desde el servidor de origen de Azure CDN una vez que haya transcurrido la hora de expiración de la SAS. Si esto ocurre y quiere que el archivo en caché no sea accesible, debe realizar una operación de purga en el archivo para eliminarlo de la memoria caché. Para obtener información sobre cómo establecer la duración de la caché en Azure CDN, vea [Control del comportamiento del almacenamiento en caché de Azure CDN con reglas de caché](cdn-caching-rules.md).

### <a name="option-2-hidden-cdn-sas-token-using-a-rewrite-rule"></a>Opción 2: Token de SAS de CDN oculto mediante una regla de reescritura
 
Esta opción solo está disponible para los perfiles de **Azure CDN Premium de Verizon**. Con esta opción, puede proteger el almacenamiento de blobs en el servidor de origen. Puede que quiera usar esta opción si no necesita restricciones de acceso específicas para el archivo, pero quiere evitar que los usuarios tengan acceso directo al origen de almacenamiento para mejorar los tiempos de descarga de Azure CDN. Cualquiera que acceda a los archivos en el contenedor especificado del servidor de origen requiere el token de SAS, que es desconocido para el usuario. Sin embargo, debido a la regla de reescritura de direcciones URL, el token de SAS no es necesario en el punto de conexión de CDN.
 
1. Use el [motor de reglas](cdn-rules-engine.md) para crear una regla de reescritura de direcciones URL. Las nuevas reglas tardan hasta cuatro horas en propagarse.

   ![Botón de administración de CDN](./media/cdn-sas-storage-support/cdn-manage-btn.png)

   ![Botón del motor de reglas de CDN](./media/cdn-sas-storage-support/cdn-rules-engine-btn.png)

   La siguiente regla de reescritura de direcciones URL de ejemplo usa un patrón de expresión regular con un grupo de captura y un punto de conexión denominado *sasstoragedemo*:
   
   Origen:   
   `(container1\/.*)`


   Destino:   
   ```
   $1?sv=2017-07-29&ss=b&srt=c&sp=r&se=2027-12-19T17:35:58Z&st=2017-12-19T09:35:58Z&spr=https&sig=kquaXsAuCLXomN7R00b8CYM13UpDbAHcsRfGOW3Du1M%3D
   ```
   ![Regla de reescritura de direcciones URL de CDN - izquierda](./media/cdn-sas-storage-support/cdn-url-rewrite-rule.png)
   ![Regla de reescritura de direcciones URL de CDN - derecha](./media/cdn-sas-storage-support/cdn-url-rewrite-rule-option-4.png)

2. Una vez que la nueva regla esté activa, cualquiera puede acceder a los archivos del contenedor especificado en el punto de conexión de CDN, con independencia de si usan un token de SAS en la dirección URL. Este es el formato: `https://<endpoint hostname>.azureedge.net/<container>/<file>`
 
   Por ejemplo:   
   `https://sasstoragedemo.azureedge.net/container1/demo.jpg`
       

3. Ajuste la duración de la caché mediante las reglas de caché o al agregar los encabezados `Cache-Control` en el servidor de origen. Debido a que Azure CDN trata el token de SAS como cadena de consulta sin formato, el procedimiento recomendado que debe seguirse es configurar una duración de la caché con la misma hora de expiración que la SAS o una anterior. De lo contrario, si un archivo se almacena en caché más tiempo que el que está activa la SAS, se puede acceder a él desde el punto de conexión de Azure CDN una vez superado el tiempo de validez de la SAS. Si esto ocurre y quiere que el archivo en caché no sea accesible, debe realizar una operación de purga en el archivo para eliminarlo de la memoria caché. Para obtener información sobre cómo establecer la duración de la caché en Azure CDN, vea [Control del comportamiento del almacenamiento en caché de Azure CDN con reglas de caché](cdn-caching-rules.md).

### <a name="option-3-using-cdn-security-token-authentication-with-a-rewrite-rule"></a>Opción 3: Uso de la autenticación de token de seguridad de red CDN con una regla de reescritura

Para usar la autenticación de token de seguridad de Azure CDN, debe tener un perfil de **Azure CDN premium de Verizon**. Esta opción es la más segura y personalizable. El acceso de cliente se basa en los parámetros de seguridad que se establezcan en el token de seguridad. Una vez que ha creado y configurado el token de seguridad, se requerirá en todas las direcciones URL de punto de conexión de CDN. Sin embargo, debido a la regla de reescritura de direcciones URL, el token de SAS no es necesario en el punto de conexión de CDN. Si posteriormente el token de SAS deja de ser válido, Azure CDN ya no podrá revalidar el contenido del servidor de origen.

1. [Cree un token de seguridad de Azure CDN](https://docs.microsoft.com/azure/cdn/cdn-token-auth#setting-up-token-authentication) y actívelo mediante el motor de reglas para el punto de conexión de red CDN y la ruta de acceso donde los usuarios pueden acceder al archivo.

   Una dirección URL de punto de conexión de token de seguridad tiene el formato siguiente:   
   `https://<endpoint hostname>.azureedge.net/<container>/<file>?<security_token>`
 
   Por ejemplo:   
   ```
   https://sasstoragedemo.azureedge.net/container1/demo.jpg?a4fbc3710fd3449a7c99986bkquaXsAuCLXomN7R00b8CYM13UpDbAHcsRfGOW3Du1M%3D
   ```
       
   Las opciones de parámetro de una autenticación de token de seguridad son distintas a las opciones de parámetro de un token de SAS. Si al crear un token de seguridad decide usar una hora de expiración, establézcala en el mismo valor que la hora de expiración del token de SAS. De este modo se garantiza que la hora de expiración sea predecible. 
 
2. Use el [motor de reglas](cdn-rules-engine.md) para crear una regla de reescritura de direcciones URL a fin de habilitar el acceso del token de SAS a todos los blobs del contenedor. Las nuevas reglas tardan hasta cuatro horas en propagarse.

   La siguiente regla de reescritura de direcciones URL de ejemplo usa un patrón de expresión regular con un grupo de captura y un punto de conexión denominado *sasstoragedemo*:
   
   Origen:   
   `(container1\/.*)`
   
   Destino:   
   ```
   $1&sv=2017-07-29&ss=b&srt=c&sp=r&se=2027-12-19T17:35:58Z&st=2017-12-19T09:35:58Z&spr=https&sig=kquaXsAuCLXomN7R00b8CYM13UpDbAHcsRfGOW3Du1M%3D
   ```
   ![Regla de reescritura de direcciones URL de CDN - izquierda](./media/cdn-sas-storage-support/cdn-url-rewrite-rule.png)
   ![Regla de reescritura de direcciones URL de CDN - derecha](./media/cdn-sas-storage-support/cdn-url-rewrite-rule-option-4.png)

3. Si renueva la SAS, actualice la regla de reescritura de direcciones URL con el nuevo token de SAS. 

## <a name="sas-parameter-considerations"></a>Consideraciones sobre los parámetros de SAS

Como los parámetros de SAS no son visibles para Azure CDN, esta no puede cambiar su comportamiento de entrega en función de dichos parámetros. Las restricciones de parámetro definidas solo se aplican en las solicitudes que Azure CDN hace en el servidor de origen y no en las solicitudes del cliente a Azure CDN. Resulta importante tener en cuenta esta distinción cuando se establecen parámetros de SAS. Si se requieren estas funcionalidades avanzadas y se usa la [opción 3](#option-3-using-cdn-security-token-authentication-with-a-rewrite-rule), establezca las restricciones adecuadas en el token de seguridad de Azure CDN.

| Nombre de parámetro de SAS | Descripción |
| --- | --- |
| Start | Hora en que Azure CDN puede comenzar a acceder al archivo de blob. Debido al desplazamiento del reloj (es decir, cuando la señal de un reloj llega a distintas horas para componentes diferentes), elija una hora 15 minutos antes si quiere que el recurso esté disponible de inmediato. |
| End | Hora tras la cual Azure CDN ya no tiene acceso al archivo de blob. Los archivos almacenados previamente en caché en Azure CDN siguen siendo accesibles. Para controlar la hora de expiración de los archivos, establezca la hora de expiración correspondiente en el token de seguridad de Azure CDN o purgue el recurso. |
| Direcciones IP permitidas | Opcional. Si usa **Azure CDN de Verizon**, puede establecer este parámetro en los intervalos que aparecen definidos en [Azure CDN de intervalos IP del servidor perimetral de Verison](/azure/cdn/cdn-pop-list-api). Si usa **Azure CDN de Akamai**, no puede establecer el parámetro de intervalos IP porque las direcciones IP no son estáticas.|
| Protocolos admitidos | Protocolos que se permiten para una solicitud realizada con la SAS de cuenta. Se recomienda la configuración HTTPS.|

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de SAS, vea los siguientes artículos:
- [Uso de Firmas de acceso compartido (SAS)](https://docs.microsoft.com/azure/storage/common/storage-dotnet-shared-access-signature-part-1)
- [Firmas de acceso compartido, Parte 2: Creación y uso de una SAS con Almacenamiento de blobs](https://docs.microsoft.com/azure/storage/blobs/storage-dotnet-shared-access-signature-part-2)

Para más información acerca de cómo configurar la autenticación por tokens, consulte [Protección de recursos de Azure Content Delivery Network con la autenticación por tokens](https://docs.microsoft.com/azure/cdn/cdn-token-auth).
