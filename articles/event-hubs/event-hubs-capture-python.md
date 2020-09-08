---
title: Lectura de datos capturados de Azure Event Hubs desde una aplicación de Python (más reciente)
description: En este artículo se muestra cómo escribir código de Python para capturar los datos que se envían a un centro de eventos y cómo leer los datos de eventos capturados de una cuenta de Azure Storage.
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: cb7165565516136a8425c4c77748c2e13715edb7
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88927874"
---
# <a name="capture-event-hubs-data-in-azure-storage-and-read-it-by-using-python-azure-eventhub-version-5"></a>Capture datos de Event Hubs en Azure Storage y léalos mediante Python (azure-eventhub versión 5)

Puede configurar un centro de eventos para que los datos que se envíen al mismo se capturen en una cuenta de Azure Storage o en Azure Data Lake Storage Gen 1 o Gen 2. En este artículo se muestra cómo escribir código de Python para enviar eventos a un centro de eventos y cómo leer los datos capturados de **Azure Blob Storage**. Para más información sobre esta característica, consulte la [introducción a la característica de captura de Event Hubs](event-hubs-capture-overview.md).

Este inicio rápido usa el [SDK de Azure Python](https://azure.microsoft.com/develop/python/) para mostrar la característica de captura. La aplicación *sender.py* envía datos telemétricos del entorno simulados a Event Hubs en formato JSON. El centro de eventos está configurado para usar la característica Capture para escribir estos datos en Blob Storage en lotes. La aplicación *capturereader.py* lee estos blobs y crea un archivo de anexos para cada dispositivo. Luego, la aplicación escribe los datos en archivos .CSV.

En esta guía de inicio rápido: 

> [!div class="checklist"]
> * Crear una cuenta de Azure Blob Storage y un contenedor en Azure Portal.
> * Crear un espacio de nombres de Event Hubs mediante Azure Portal.
> * Cree un centro de eventos con la característica de Captura habilitada y conéctelo a su cuenta de almacenamiento.
> * Enviar datos al centro de eventos con un script de Python.
> * Leer y procesar los archivos de Capture de Event Hubs con otro script de Python.

## <a name="prerequisites"></a>Requisitos previos

- Python 2.7 y 3.5 o posterior, con PIP instalado y actualizado.  
- Suscripción a Azure. Si no tiene una, [cree una cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.  
- Un espacio de nombres de Event Hubs y un centro de eventos activos.
[Crear un espacio de nombres de Event Hubs y un centro de eventos en el espacio de nombres](event-hubs-create.md). Registre el nombre del espacio de nombres de Event Hubs, el nombre del centro de eventos y la clave de acceso principal del espacio de nombres. Para obtener la clave de acceso, consulte [Obtención de una cadena de conexión de Event Hubs](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). El nombre de la clave predeterminada es *RootManageSharedAccessKey*. Para este inicio rápido solo necesita la clave principal. No necesita la cadena de conexión.  
- Una cuenta de Azure Storage, un contenedor de blobs en la cuenta de almacenamiento y una cadena de conexión de la cuenta de almacenamiento. Si no tiene estos elementos, realice estas operaciones:  
    1. [Creación de una cuenta de Azure Storage](../storage/common/storage-account-create.md?tabs=azure-portal)  
    1. [Creación de un contenedor de blobs en la cuenta de almacenamiento](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container)  
    1. [Obtención de la cadena de conexión para una cuenta de almacenamiento](../storage/common/storage-configure-connection-string.md)

    Asegúrese de registrar la cadena de conexión y el nombre del contenedor para usarlo posteriormente en este inicio rápido.  
- Habilite la característica de captura para el centro de eventos. Para ello, siga las instrucciones de [Habilitación de la característica de captura de Event Hubs desde Azure Portal](event-hubs-capture-enable-through-portal.md). Seleccione la cuenta de almacenamiento y el contenedor de blobs que creó en el paso anterior. También puede habilitar la característica al crear un centro de eventos.  

## <a name="create-a-python-script-to-send-events-to-your-event-hub"></a>Creación de un script de Python para enviar eventos a un centro de eventos
En esta sección, creará un script de Python que envía 200 eventos (10 dispositivos x 20 eventos) a un centro de eventos. Estos eventos son una lectura del entorno de ejemplo que se envía en formato JSON. 

1. Abra el editor de Python que prefiera, como [Visual Studio Code][Visual Studio Code].
2. Crear un script denominado *sender.py*. 
3. Pegue el código siguiente en *sender.py*. 
   
    ```python
    import time
    import os
    import uuid
    import datetime
    import random
    import json
    
    from azure.eventhub import EventHubProducerClient, EventData
    
    # This script simulates the production of events for 10 devices.
    devices = []
    for x in range(0, 10):
        devices.append(str(uuid.uuid4()))
    
    # Create a producer client to produce and publish events to the event hub.
    producer = EventHubProducerClient.from_connection_string(conn_str="EVENT HUBS NAMESAPCE CONNECTION STRING", eventhub_name="EVENT HUB NAME")
    
    for y in range(0,20):    # For each device, produce 20 events. 
        event_data_batch = producer.create_batch() # Create a batch. You will add events to the batch later. 
        for dev in devices:
            # Create a dummy reading.
            reading = {'id': dev, 'timestamp': str(datetime.datetime.utcnow()), 'uv': random.random(), 'temperature': random.randint(70, 100), 'humidity': random.randint(70, 100)}
            s = json.dumps(reading) # Convert the reading into a JSON string.
            event_data_batch.add(EventData(s)) # Add event data to the batch.
        producer.send_batch(event_data_batch) # Send the batch of events to the event hub.
    
    # Close the producer.    
    producer.close()
    ```
4. Reemplace los valores siguientes en los scripts:  
    * Reemplace `EVENT HUBS NAMESPACE CONNECTION STRING` por la cadena de conexión para el espacio de nombres de Event Hubs.  
    * Reemplace `EVENT HUB NAME` por el nombre del centro de eventos.  
5. Ejecute el script para enviar eventos al centro de eventos.  
6. En Azure Portal, puede comprobar que el centro de eventos ha recibido los mensajes. Cambie a la vista **Mensajes** en la sección **Métricas**. Actualice la página para actualizar el gráfico. Pueden pasar unos segundos hasta que aparezca en la página que los mensajes se han recibido. 

    [![Comprobación de que el centro de eventos ha recibido los mensajes](./media/get-started-capture-python-v2/messages-portal.png)](./media/get-started-capture-python-v2/messages-portal.png#lightbox)

## <a name="create-a-python-script-to-read-your-capture-files"></a>Creación de un script de Python que lea archivos de Capture
En este ejemplo, los datos capturados se almacenan en Azure Blob Storage. El script de esta sección lee los archivos de datos capturados de la cuenta de Azure Storage y genera archivos CSV que podrá abrir y ver fácilmente. Verá 10 archivos en el directorio de trabajo actual de la aplicación. Estos archivos contendrán las lecturas de entorno para los 10 dispositivos. 

1. En el editor de Python, cree un script denominado *capturereader.py*. Este script lee los archivos capturados y crea un archivo para cada dispositivo, con el fin de escribir los datos solo de dicho dispositivo.
2. Pegue el código siguiente en *capturereader.py*. 
   
    ```python
    import os
    import string
    import json
    import uuid
    import avro.schema
    
    from azure.storage.blob import ContainerClient, BlobClient
    from avro.datafile import DataFileReader, DataFileWriter
    from avro.io import DatumReader, DatumWriter
    
    
    def processBlob2(filename):
        reader = DataFileReader(open(filename, 'rb'), DatumReader())
        dict = {}
        for reading in reader:
            parsed_json = json.loads(reading["Body"])
            if not 'id' in parsed_json:
                return
            if not parsed_json['id'] in dict:
                list = []
                dict[parsed_json['id']] = list
            else:
                list = dict[parsed_json['id']]
                list.append(parsed_json)
        reader.close()
        for device in dict.keys():
            filename = os.getcwd() + '\\' + str(device) + '.csv'
            deviceFile = open(filename, "a")
            for r in dict[device]:
                deviceFile.write(", ".join([str(r[x]) for x in r.keys()])+'\n')
    
    def startProcessing():
        print('Processor started using path: ' + os.getcwd())
        # Create a blob container client.
        container = ContainerClient.from_connection_string("AZURE STORAGE CONNECTION STRING", container_name="BLOB CONTAINER NAME")
        blob_list = container.list_blobs() # List all the blobs in the container.
        for blob in blob_list:
            # Content_length == 508 is an empty file, so process only content_length > 508 (skip empty files).        
            if blob.size > 508:
                print('Downloaded a non empty blob: ' + blob.name)
                # Create a blob client for the blob.
                blob_client = ContainerClient.get_blob_client(container, blob=blob.name)
                # Construct a file name based on the blob name.
                cleanName = str.replace(blob.name, '/', '_')
                cleanName = os.getcwd() + '\\' + cleanName 
                with open(cleanName, "wb+") as my_file: # Open the file to write. Create it if it doesn't exist. 
                    my_file.write(blob_client.download_blob().readall()) # Write blob contents into the file.
                processBlob2(cleanName) # Convert the file into a CSV file.
                os.remove(cleanName) # Remove the original downloaded file.
                # Delete the blob from the container after it's read.
                container.delete_blob(blob.name)
    
    startProcessing()    
    ```
3. Reemplace `AZURE STORAGE CONNECTION STRING` por la cadena de conexión de su cuenta de Azure Storage. El nombre del contenedor que creó en este inicio rápido es *capture*. Si ha usado otro nombre para el contenedor, reemplace *capture* por el nombre del contenedor en la cuenta de almacenamiento. 

## <a name="run-the-scripts"></a>Ejecución de los scripts
1. Abra un símbolo del sistema que tiene Python en su ruta de acceso y, después, ejecute dichos comandos para instalar los paquetes de requisitos previos de Python:
   
   ```
   pip install azure-storage-blob
   pip install azure-eventhub
   pip install avro-python3
   ```
2. Vaya al directorio en que guardó *sender.py* y *capturereader.py* y ejecute este comando:
   
   ```
   python sender.py
   ```
   
   Este comando inicia un nuevo proceso de Python para ejecutar el remitente.
3. Espere unos minutos a que se ejecute la captura y, después, escriba el siguiente comando en la ventana de comandos original:
   
   ```
   python capturereader.py
   ```

   Este procesador de captura usa el directorio local para descargar todos los blobs de la cuenta de almacenamiento y del contenedor. Procesa los que no estén vacíos y escribe los resultados en forma de archivos .CSV en el directorio local.

## <a name="next-steps"></a>Pasos siguientes
Consulte los [ejemplos de Python en GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/eventhub/azure-eventhub/samples). 


[Azure portal]: https://portal.azure.com/
[Overview of Event Hubs Capture]: event-hubs-capture-overview.md
[1]: ./media/event-hubs-archive-python/event-hubs-python1.png
[About Azure storage accounts]:../storage/common/storage-create-storage-account.md
[Visual Studio Code]: https://code.visualstudio.com/
[Event Hubs overview]: ./event-hubs-about.md
