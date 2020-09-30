---
title: ¿Qué es Azure Data Share?
description: Aprenda a compartir datos de forma sencilla y segura con varios clientes y asociados mediante Azure Data Share.
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: overview
ms.date: 07/30/2020
ms.openlocfilehash: e80ea37409f7b5beb48ba7b8e96aeeda9a5836e5
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89433439"
---
# <a name="what-is-azure-data-share"></a>¿Qué es Azure Data Share?

En la actualidad, los datos se ven como un recurso estratégico clave que muchas organizaciones necesitan compartir de forma sencilla y segura con sus clientes y asociados. Hoy en día, los clientes tiene a sus disposición varias formas de hacerlo, entre las que se incluyen el FTP, el correo electrónico y las API, entre otras. Por eso, es fácil que las organizaciones pierdan el rastro de las personas y entidades con quienes han compartido sus datos. El uso compartido de datos a través de FTP o creando una infraestructura de API propia a menudo es caro de aprovisionar y administrar. El uso de estos métodos para compartir datos a gran escala lleva asociada una sobrecarga de administración. 

Muchas organizaciones necesitan ser capaces de asumir la responsabilidad de los datos que han compartido. Además de dicha responsabilidad, muchas organizaciones desean poder controlar, administrar y supervisar todos el uso compartido de sus datos de una manera sencilla. En el mundo actual, donde se espera que los datos sigan creciendo a un ritmo exponencial, las organizaciones necesitan una forma sencilla de compartir los macrodatos. Los clientes exigen que los datos estén totalmente actualizados para estar seguros de que son capaces de sacar conclusiones oportunas.

Azure Data Share permite a las organizaciones compartir datos de forma simple y segura con varios clientes y asociados. Con unos pocos clics, puede aprovisionar una nueva cuenta de Azure Data Share, agregar conjuntos de datos e invitar a sus clientes y asociados a Data Share. Los proveedores de datos siempre tienen el control de los datos que han compartido. Azure Data Share simplifica la administración y supervisión de los datos que se comparten, cuándo y por quién. 

Los proveedores de datos pueden seguir manteniendo el control de sus datos. Para ello, especifican las condiciones de uso de su instancia de Azure Data Share. Quien consuma estos datos debe aceptar dichas condiciones para poder recibirlos. Los proveedores de datos pueden especificar la frecuencia con la que los consumidores de sus datos reciben actualizaciones. El proveedor de datos puede revocar el acceso a las nuevas actualizaciones en cualquier momento. 

Azure Data Share le ayuda a mejorar las conclusiones, ya que facilita la combinación de datos de terceros, con el fin de enriquecer los escenarios de inteligencia artificial y análisis. Use fácilmente las eficaces herramientas de análisis de Azure para preparar, procesar y analizar los datos compartidos mediante Azure Data Share. 

Tanto el proveedor de datos como el consumidor de datos tienen que tener una suscripción de Azure para compartir y recibir datos. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/).

## <a name="scenarios-for-azure-data-share"></a>Escenarios de Azure Data Share

Azure Data Share puede usarse en varios sectores diferentes. Por ejemplo, un minorista puede compartir datos recientes de un punto de ventas con sus proveedores. Mediante Azure Data Share, un minorista puede configure un recurso compartido de datos que contenga los datos de un punto de ventas para todos sus proveedores y compartir las ventas por hora o diariamente. 

Azure Data Share también se puede usar para establecer un mercado de datos para un sector concreto. Por ejemplo, una institución gubernamental o de investigación que comparte regularmente datos anónimos acerca del crecimiento de la población con terceros. 

Otro caso de uso de Azure Data Share consiste en establecer un consorcio de datos. Por ejemplo, varias instituciones de investigación diferentes puede compartir datos con un único cuerpo de confianza. Los datos se analizan, agregan o procesan mediante las herramientas de análisis de Azure y, después, se comparten los datos con las partes interesadas. 

## <a name="how-it-works"></a>Funcionamiento

Azure Data Share ofrece actualmente un uso compartido basado en instantáneas y un uso compartido in situ. 

En el uso compartido basado en instantáneas, los datos parten de la suscripción de Azure del proveedor de datos y llegan a la suscripción de Azure del consumidor de datos. Los proveedores de datos aprovisionan recursos compartidos de datos e invitan a ellos a los destinatarios. Los consumidores de datos reciben una invitación a su recurso compartido de datos a través de correo electrónico. Una vez que un consumidor de datos acepta la invitación, puede desencadenar una instantánea completa de los datos que ha compartido con él. Estos datos se reciben en la cuenta de almacenamiento de los consumidores de los datos. Los consumidores de los datos pueden recibir actualizaciones regulares e incrementales de los datos compartidos con ellos, con el fin de que siempre tengan la versión más reciente de los datos. 

Los proveedores de datos pueden ofrecer a sus consumidores de datos actualizaciones incrementales de los datos compartidos con ellos mediante de una programación de instantáneas. Las programaciones de instantáneas se ofrecen cada hora o diariamente. Cuando un consumidor de datos acepta y configura su recurso compartido de datos, puede suscribirse a una programación de instantáneas. Esto es útil en aquellos escenarios en los que los datos compartidos se actualizan de forma periódica y el consumidor de datos necesita los datos más actualizados. 

![flujo de recurso compartido de datos](media/data-share-flow.png)

Cuando un consumidor de datos acepta un recurso compartido de datos, puede recibir los datos en el almacén de datos que prefiera. Por ejemplo, si el proveedor de datos comparte datos mediante Azure Blob Storage, el consumidor de datos puede recibirlos en Azure Data Lake Store. Del mismo modo, si el proveedor de datos comparte datos desde una instancia de Azure Synapse Analytics (anteriormente SQL Data Warehouse), el consumidor de datos puede elegir si desea recibirlos en una instancia de Azure Data Lake Store, una de Azure SQL Database o una de Azure Synapse Analytics. En el caso de uso compartido desde orígenes basados en SQL, el consumidor de datos también puede elegir si recibirá los datos en formato Parquet o csv. 

Con el uso compartido in situ, los proveedores de datos pueden compartir datos donde estos residen sin copiar los datos. Después de establecer la relación de uso compartido a través del flujo de invitación, se crea un vínculo simbólico entre el almacén de datos de origen del proveedor de datos y el almacén de datos de destino del consumidor de datos. El consumidor de datos puede leer los datos y realizar consultas en ellos en tiempo real mediante su propio almacén de datos. Los cambios que se realicen en el almacén de datos de origen están disponibles de inmediato para el consumidor de datos. El uso compartido in situ se encuentra actualmente en versión preliminar para Azure Data Explorer.

## <a name="key-capabilities"></a>Principales capacidades

Azure Data Share permite a los proveedores de datos:

* Compartir datos desde la lista de [almacenes de datos admitidos](supported-data-stores.md) con clientes y asociados de fuera de la organización

* Realizar un seguimiento de los clientes y asociados con los que se han compartido datos

* Elección del uso compartido basado en instantáneas o el uso compartido in situ

* Saber con qué frecuencia los consumidores de datos reciben actualizaciones de sus datos

* Permitir que los clientes extraigan la versión más reciente de los datos cuando la necesiten o que puedan recibir automáticamente cambios incrementales de los datos a intervalos que usted defina

Azure Data Share permite a los consumidores de datos: 

* Ver una descripción del tipo de datos que se comparte

* Ver las condiciones de uso de los datos

* Aceptar o rechazar una invitación de Azure Data Share

* Aceptar los datos compartidos con usted en un [almacén de datos admitido](supported-data-stores.md).

* Desencadenar una instantánea completa o incremental de una instancia de Azure Data Share que una organización ha compartido con usted

* Suscribirse a una instancia de Azure Data Share para recibir la copia más reciente de los datos a través de una instantánea incremental

Todas las funcionalidades claves mencionadas anteriormente se admiten a través de Azure Portal o de las API REST. Para obtener más detalles acerca del uso de Azure Data Share a través de las API REST, consulte la documentación de referencia. 

## <a name="supported-regions"></a>Regiones admitidas

Para obtener una lista de regiones en las que Azure Data Share está disponible, consulte la página de [productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=data-share) y busque Azure Data Share. 

Azure Data Share no almacena una copia de los datos en sí. Los datos se almacenan en el almacén de datos subyacente que se comparte. Por ejemplo, si un productor de datos almacena sus datos en una cuenta de Azure Data Lake Store que se encuentra la región Oeste de EE. UU., ahí es donde se almacenan los datos. Si comparte los datos con una cuenta de Azure Storage ubicada en la región Oeste de Europa mediante una instantánea, lo habitual es que los datos se transfieran directamente a la cuenta de Azure Storage que se encuentra en Oeste de Europa.

No es preciso que el servicio Azure Data Share esté disponible en su región para aprovechar el servicio. Por ejemplo, si tiene datos almacenados en una cuenta de Azure Storage ubicada en una región en la que Azure Data Share aún no esté disponible, puede aprovechar el servicio para compartir los datos. 

## <a name="next-steps"></a>Pasos siguientes

Para obtener información acerca de cómo empezar a compartir datos, vaya al tutorial que cubre cómo [compartir sus datos](share-your-data.md).
