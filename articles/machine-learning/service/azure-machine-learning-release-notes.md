---
title: Novedades de la versión
titleSuffix: Azure Machine Learning service
description: Obtenga información sobre las actualizaciones más recientes a Azure Machine Learning Service y los SDK de Python para el aprendizaje automático y la preparación de datos.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.author: larryfr
author: Blackmist
ms.date: 05/14/2019
ms.custom: seodec18
ms.openlocfilehash: 892b9bc63f9f2d9abc7108587a7bf929473e4648
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65779429"
---
# <a name="azure-machine-learning-service-release-notes"></a>Notas de la versión del servicio Azure Machine Learning

En este artículo obtendrá información acerca de las versiones del servicio Azure Machine Learning.  Para obtener una descripción completa de cada SDK, consulte los documentos de referencia de:
+ Azure Machine Learning [ **SDK principal para Python**](https://aka.ms/aml-sdk)
+ El [**SDK de preparación de datos de Azure Machine Learning**](https://aka.ms/data-prep-sdk)

Para obtener información acerca de errores conocidos y soluciones alternativas, consulte [la lista de problemas conocidos](resource-known-issues.md).

## <a name="2019-05-14"></a>2019-05-14

### <a name="azure-machine-learning-sdk-for-python-v1039"></a>Azure Machine Learning SDK for Python v1.0.39
+ **Cambios**
  + Opción de configuración de ejecución auto_prepare_environment está en desuso, con automático preparar convertirse en el valor predeterminado.

## <a name="2019-05-06"></a>2019-05-06

### <a name="azure-portal"></a>Azure Portal

En el portal de Azure, ahora puede:
+ Crear y ejecutar experimentos de aprendizaje automático automatizadas 
+ Crear una máquina virtual de Bloc de notas para probar los cuadernos de Jupyter de ejemplo o el suyo propio.
+ Nueva sección de creación (versión preliminar) en el área de trabajo de servicio de Machine Learning, que incluye automatizada de Machine Learning, la interfaz Visual y máquinas virtuales hospedadas de Bloc de notas
    + Crear automáticamente un modelo de uso automatizado machine learning 
    + Usar un arrastre y coloque una interfaz Visual para ejecutar experimentos
    + Crear una VM de Bloc de notas para explorar los datos, crear modelos e implementar servicios.
+ Gráfico y métrica de actualización en los informes de ejecución de Live y ejecutar las páginas de detalles
+ Visor del archivo actualizado para los registros, salidas y las instantáneas en las páginas de detalles de ejecución.
+ Creación de informes nueva y mejorada experiencia en la pestaña de experimentos. 
+ Se agregó la posibilidad de descargar el archivo config.json de la página de información general del área de trabajo del servicio de Azure Machine Learning.
+ Admite la creación de área de trabajo de servicio de Machine Learning desde el área de trabajo de Azure Databricks 

## <a name="2019-04-26"></a>2019-04-26

### <a name="azure-machine-learning-sdk-for-python-v1033"></a>Azure Machine Learning SDK for Python v1.0.33
+ **Nuevas características:**
  + El _Workspace.create_ método ahora acepta las configuraciones de clúster predeterminado para los clústeres de CPU y GPU.
  + Si se produce un error en la creación del área de trabajo, se limpian los recursos dependientes.
  + De forma predeterminada Azure Container Registry SKU se cambió a básico.
  + Azure Container Registry se crea de forma diferida, cuando sea necesario para la creación de imagen o de ejecución.
  + Compatibilidad con entornos para las ejecuciones de entrenamiento.

### <a name="notebook-virtual-machine"></a>Máquina Virtual de Bloc de notas 

Usar una máquina virtual de Bloc de notas como un entorno de hospedaje seguro y preparada para la empresa para cuadernos de Jupyter en el que puede programar los experimentos de aprendizaje automático, implementar modelos como puntos de conexión web y realizar todas las demás operaciones compatibles con Azure Machine Learning SDK con Python. Proporciona varias funcionalidades:
+ [Inicie rápidamente una máquina virtual del Bloc de notas preconfigurada](quickstart-run-cloud-notebook.md) que tiene la versión más reciente del SDK de Azure Machine Learning y los paquetes relacionados.
+ Acceso se protege mediante tecnologías comprobadas, como HTTPS, autorización y autenticación de Azure Active Directory.
+ Almacenamiento en la nube confiable de blocs de notas y el código de la cuenta de almacenamiento de blobs de área de trabajo de Azure Machine Learning. Puede eliminar la máquina virtual del Bloc de notas de forma segura sin perder su trabajo.
+ Preinstalado cuadernos de ejemplo para explorar y experimentar con las características del servicio de Azure Machine Learning.
+ Capacidades de personalización completa de máquinas virtuales de Azure, cualquier tipo de máquina virtual, todos los paquetes, todos los controladores. 

## <a name="2019-04-26"></a>2019-04-26

### <a name="azure-machine-learning-sdk-for-python-v1033-released"></a>Azure Machine Learning SDK para v1.0.33 Python publicado.

+ Azure ML modelos acelerados de Hardware en [FPGA](concept-accelerate-with-fpgas.md) está disponible con carácter general.
  + Ahora puede [usar el paquete de modelos de azureml aceleración](how-to-deploy-fpga-web-service.md) para:
    + Entrenar los pesos de una red neuronal profunda admitido (ResNet 50, ResNet 152, DenseNet 121, VGG-16 y SSD VGG)
    + Use la transferencia de aprendizaje con la DNN admitida
    + Registrar el modelo con el servicio Administración de modelos y el modelo de inclusión en un contenedor
    + Implementar el modelo en una máquina virtual de Azure con una FPGA en un clúster de Azure Kubernetes Service (AKS)
  + Implementar el contenedor a un [borde del cuadro de datos de Azure](https://docs.microsoft.com/azure/databox-online/data-box-edge-overview) dispositivo del servidor
  + Puntuar los datos con el punto de conexión gRPC con este [ejemplo](https://github.com/Azure-Samples/aml-hardware-accelerated-models)

### <a name="automated-machine-learning"></a>Automated Machine Learning

+ Barrido de hiperparámetros para habilitar featurizers agregar dinámicamente para optimizar el rendimiento de características. Nuevo featurizers: trabajo incrustaciones de peso de la evidencia, codificaciones de destino, codificación de destino de texto, distancia del clúster
+ Divide CV inteligente para controlar el entrenamiento o válida dentro de ML automatizada
+ Algunos cambios de optimización de memoria y mejora del rendimiento en tiempo de ejecución
+ Mejora del rendimiento en la explicación del modelo
+ Conversión de modelo ONNX para ejecución local
+ Se ha agregado compatibilidad Subsampling
+ Intelligent detiene cuando no hay criterios de salida definido
+ Árboles apiladas

+ Previsión de Series temporales
  + Nueva función previsión de predicción   
  + Ahora puede usar la validación cruzada origen gradual en los datos de series temporales
  + Nueva funcionalidad agregada para configurar el tiempo que tarda un poco serie 
  + Nueva funcionalidad agregada para admitir las funciones agregado de ventana gradual
  + Nueva detección de vacaciones y caracterizador al código de país se define en experimentar la configuración

+ Azure Databricks
  + Habilita el pronóstico de series temporales y funcionalidad explainabilty/interoperabilidad del modelo
  + Ahora puede cancelar y reanudar (continuar) ML automatizada experimentos
  + Se agregó compatibilidad para el procesamiento de varios núcleos

### <a name="mlops"></a>MLOps
+ **Implementación local y depuración para la puntuación de contenedores**<br/> Ahora puede implementar un modelo de aprendizaje automático localmente e iterar rápidamente en su archivo de puntuación y las dependencias para asegurarse de que se comportan según lo previsto.

+ **Introduced InferenceConfig & Model.deploy()**<br/> Modelo de implementación ahora admite la especificación de una carpeta de origen con un script de entrada, el mismo que un /runconfig.  Además, se ha simplificado la implementación de modelos para un solo comando.

+ **Seguimiento de referencias de GIT**<br/> Los clientes vienen solicitando capacidades de integración de Git básicas para algún tiempo ya que ayuda a mantener una pista de auditoría to-end. Hemos implementado seguimiento a través de las entidades principales de Azure Machine Learning para los metadatos relacionados con Git (repo, confirmación, estado limpio). Esta información se recopilará automáticamente mediante el SDK y la CLI.

+ **Servicio de generación de perfiles y la validación de modelos**<br/> Los clientes se quejan con frecuencia de la dificultad para cambiar correctamente el tamaño de proceso asociada a su servicio de inferencia. Con nuestro modelo de servicio de generación de perfiles, el cliente pueda proporcionar entradas de ejemplo y se generará perfiles en 16 CPU distinta o ajustar el tamaño de las configuraciones de memoria para determinar el óptimo para la implementación.

+ **Traiga su propia imagen base para la inferencia**<br/> Otra causa habitual de quejas era la dificultad de movimiento de experimentación a las dependencias de inferencia RE uso compartidas. Con nuestra nueva imagen base que pueda compartir, ahora puede reutilizar las imágenes base experimentación, dependencias y todo, para la inferencia. Esto debe acelerar las implementaciones y reducir la brecha de interno al bucle externo.

+ **Experiencia mejorada de generación de esquema de Swagger**<br/> La generación de swagger anterior método era imposible automatizar y propensas a errores. Tenemos una nueva forma en la línea de generación de esquemas de swagger desde cualquier función de Python a través de los elementos Decorator. Nos hemos abierto este código y el protocolo de generación de esquema no está asociado a la plataforma de aprendizaje automático de Azure.

+ **CLI de Azure Machine Learning está disponible con carácter general (GA)**<br/> Ahora se pueden implementar modelos con un solo comando CLI. Recibimos comentarios del cliente comunes que nadie se implementa un modelo de aprendizaje automático de un cuaderno de Jupyter. El [ **documentación de referencia CLI** ](https://aka.ms/azmlcli) se ha actualizado.


## <a name="2019-04-22"></a>2019-04-22

Azure Machine Learning SDK para v1.0.30 Python publicado.

El [ `PipelineEndpoint` ](https://docs.microsoft.com/python/api/azureml-pipeline-core/azureml.pipeline.core.pipeline_endpoint.pipelineendpoint?view=azure-ml-py) se presentan para agregar una nueva versión de una canalización publicada manteniendo el mismo punto de conexión.

## <a name="2019-04-17"></a>2019-04-17

### <a name="azure-machine-learning-data-prep-sdk-v112"></a>SDK v1.1.2 de preparación de datos de Azure Machine Learning

Nota: Ya no se instalará el SDK de Python de preparación de datos `numpy` y `pandas` paquetes. Consulte [actualizado las instrucciones de instalación](https://aka.ms/aml-data-prep-installation).

+ **Nuevas características:**
  + Ahora puede usar la transformación dinámica.
    + Guía de procedimientos: [Cuaderno de PowerPivot](https://aka.ms/aml-data-prep-pivot-nb)
  + Ahora puede usar expresiones regulares en funciones nativas.
    + Ejemplos:
      + `dflow.filter(dprep.RegEx('pattern').is_match(dflow['column_name']))`
      + `dflow.assert_value('column_name', dprep.RegEx('pattern').is_match(dprep.value))`
  + Ahora puede usar `to_upper`  y `to_lower`  funciones en lenguaje de expresiones.
  + Ahora puede ver el número de valores únicos de cada columna de un perfil de datos.
  + Algunos de los pasos de lector usadas, ahora puede pasar en el `infer_column_types` argumento. Si se establece en `True`, preparación de datos intentará detectar y convertir automáticamente los tipos de columna.
    + `inference_arguments` Ahora está en desuso.
  + Ahora puede llamar a `Dataflow.shape`.

+ **Mejoras y correcciones de errores**
  + `keep_columns` ahora acepta un argumento opcional adicional `validate_column_exists`, que comprueba si el resultado de `keep_columns` contendrá todas las columnas.
  + Todos los pasos de lector (que se leen desde un archivo) ahora aceptan un argumento opcional adicional `verify_exists`.
  + Mejorar el rendimiento de lectura de datos de pandas y obtener los perfiles de datos.
  + Se ha corregido un error donde no pudo crear segmentos de un solo paso desde un flujo de datos con un índice único.

## <a name="2019-04-15"></a>2019-04-15

### <a name="azure-portal"></a>Azure Portal
  + Ahora puede volver a enviar un Script existente que se ejecute en un clúster de cálculo remoto existente. 
  + Ahora puede ejecutar una canalización publicada con los nuevos parámetros en la ficha de canalizaciones. 
  + Detalles de ejecución ahora es compatible con un nuevo Visor del archivo de instantánea. Puede ver una instantánea del directorio cuando envía una ejecución específica. También puede descargar el Bloc de notas que se envió para iniciar la ejecución.
  + Ahora puede cancelar ejecuciones primario desde el portal de Azure.

## <a name="2019-04-08"></a>2019-04-08

### <a name="azure-machine-learning-sdk-for-python-v1023"></a>Azure Machine Learning SDK for Python v1.0.23

+ **Nuevas características:**
  + Ahora, el SDK de Azure Machine Learning admite Python 3.7.
  + Ahora, Azure Machine Learning DNN estimadores proporcionan compatibilidad con múltiples versiones. Por ejemplo, `TensorFlow`  Estimador ahora acepta una `framework_version` parámetro y los usuarios pueden especificar la versión '1.10' o '1.12'. Para obtener una lista de las versiones admitidas por la versión SDK actual, llame a `get_supported_versions()` en la clase de marco deseado (por ejemplo, `TensorFlow.get_supported_versions()`).
  Para obtener una lista de las versiones admitidas por la versión más reciente del SDK, consulte el [documentación Estimador DNN](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn?view=azure-ml-py).

### <a name="azure-machine-learning-data-prep-sdk-v111"></a>SDK v1.1.1 de preparación de datos de Azure Machine Learning

+ **Nuevas características:**
  + Puede leer varios orígenes de DataReference o ruta de datos/almacén de datos mediante transformaciones de read_.
  + Puede realizar las siguientes operaciones en las columnas para crear una nueva columna: división, floor, módulo, power, longitud.
  + Preparación de datos es ahora parte de la serie de diagnósticos de Azure Machine Learning y registrará la información de diagnóstico de forma predeterminada.
    + Para desactivar esta característica, establezca esta variable de entorno en true: DISABLE_DPREP_LOGGER

+ **Mejoras y correcciones de errores**
  + Documentación de código mejorada para funciones y clases usadas con frecuencia.
  + Se ha corregido un error en auto_read_file que no se pudo leer los archivos de Excel.
  + Se ha agregado la opción de sobrescribir la carpeta en read_pandas_dataframe.
  + Mejorar el rendimiento de la instalación de la dependencia de dotnetcore2 y se ha agregado compatibilidad para Fedora 27/28 y 1804 de Ubuntu.
  + Mejorar el rendimiento de lectura de Blobs de Azure.
  + Detección del tipo de columna ahora es compatible con las columnas de tipo Long.
  + Se ha corregido un error donde se muestran algunos valores de fecha como marcas de tiempo en lugar de objetos de fecha y hora de Python.
  + Se ha corregido un error donde se muestran algunos recuentos de tipo como dobles en lugar de enteros.

  
## <a name="2019-03-25"></a>2019-03-25

### <a name="azure-machine-learning-sdk-for-python-v1021"></a>Azure Machine Learning SDK for Python v1.0.21

+ **Nuevas características:**
  + El *azureml.core.Run.create_children* método permite la creación de baja latencia de varios secundarios se ejecuta con una sola llamada.

### <a name="azure-machine-learning-data-prep-sdk-v110"></a>SDK v1.1.0 de preparación de datos de Azure Machine Learning

+ **Cambios importantes**
  + El concepto del paquete de preparación de datos está desusado y ya no se admite. En lugar de almacenar varios flujos de datos en un solo paquete, puede conservar los flujos de datos individualmente.
    + Guía de procedimientos: [Bloc de notas al abrir y guardar flujos de datos](https://aka.ms/aml-data-prep-open-save-dataflows-nb)

+ **Nuevas características:**
  + Preparación de datos puede ahora reconocer las columnas que coinciden con un determinado tipo semántico y dividen en consecuencia. Incluir el STypes admitidas actualmente: código postal de Estados Unidos, coordenadas geográficas (latitud y longitud), las direcciones IPv4 e IPv6, número de teléfono de EE. UU. y dirección de correo electrónico.
    + Guía de procedimientos: [Notebook semántica de tipos](https://aka.ms/aml-data-prep-semantic-types-nb)
  + Preparación de datos ahora admite las siguientes operaciones para generar una columna resultante de dos columnas numéricas: restar, multiplicar, dividir y de módulo.
  + Puede llamar a `verify_has_data()` en un flujo de datos para comprobar si el flujo de datos podría producir registros si se ejecuta.

+ **Mejoras y correcciones de errores**
  + Ahora puede especificar el número de ubicaciones que se usará en un histograma para perfiles de la columna numérica.
  + El `read_pandas_dataframe` transformación requiere ahora que la cadena de la trama de datos o byte - nombres de columna de tipo.
  + Se ha corregido un error en la `fill_nulls` faltaba transformación, donde los valores no se correctamente rellena si la columna.

## <a name="2019-03-11"></a>2019-03-11

### <a name="azure-machine-learning-sdk-for-python-v1018"></a>Azure Machine Learning SDK for Python v1.0.18

 + **Cambios**
   + El paquete azureml tensorboard reemplaza azureml-contrib-tensorboard.
   + Con esta versión, puede configurar una cuenta de usuario en el clúster de proceso administrado (amlcompute), al crearlo. Esto puede realizarse pasando estas propiedades en la configuración de aprovisionamiento. Puede encontrar más detalles en el [documentación de referencia SDK](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute?view=azure-ml-py#provisioning-configuration-vm-size-----vm-priority--dedicated---min-nodes-0--max-nodes-none--idle-seconds-before-scaledown-none--admin-username-none--admin-user-password-none--admin-user-ssh-key-none--vnet-resourcegroup-name-none--vnet-name-none--subnet-name-none--tags-none--description-none-).

### <a name="azure-machine-learning-data-prep-sdk-v1017"></a>SDK v1.0.17 de preparación de datos de Azure Machine Learning

+ **Nuevas características:**
  + Ahora admite la adición de dos columnas numéricas para generar una columna resultante mediante el lenguaje de expresiones.

+ **Mejoras y correcciones de errores**
  + Se ha mejorado la documentación y la comprobación de parámetros de random_split.
  
## <a name="2019-02-27"></a>2019-02-27

### <a name="azure-machine-learning-data-prep-sdk-v1016"></a>SDK v1.0.16 de preparación de datos de Azure Machine Learning

+ **Corrección de errores**
  + Se ha corregido una entidad de servicio problema de autenticación que se produjo por un cambio de API.

## <a name="2019-02-25"></a>2019-02-25

### <a name="azure-machine-learning-sdk-for-python-v1017"></a>Azure Machine Learning SDK for Python v1.0.17

+ **Nuevas características:**

  + Ahora, Azure Machine Learning proporciona compatibilidad de primera clase para el marco DNN popular encadenador. Uso de [ `Chainer` ](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.chainer?view=azure-ml-py) los usuarios de la clase fácilmente pueden entrenar e implementar modelos de encadenador.
    + Obtenga información sobre cómo [ejecutar aprendizaje distribuido con ChainerMN](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/distributed-chainer/distributed-chainer.ipynb)
    + Obtenga información sobre cómo [ejecutar el ajuste de hiperparámetros con encadenador mediante HyperDrive](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training-with-deep-learning/train-hyperparameter-tune-deploy-with-chainer/train-hyperparameter-tune-deploy-with-chainer.ipynb)
  + Canalizaciones de Azure Machine Learning Agregar desencadenador de capacidad en función de una ejecución de canalización de modificaciones de almacén de datos. La canalización [Bloc de notas de programación](https://aka.ms/pl-schedule) se actualiza para mostrar esta característica.

+ **Mejoras y correcciones de errores**
  + Hemos agregado compatibilidad con canalizaciones de Azure Machine Learning para establecer la propiedad source_directory_data_store en un almacén de datos deseada (por ejemplo, un almacenamiento de blobs) en [RunConfigurations](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfig.runconfiguration?view=azure-ml-py) que se suministran a la [ PythonScriptStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.python_script_step.pythonscriptstep?view=azure-ml-py). De forma predeterminada pasos utilizan el almacén de archivos de Azure como el almacén de datos de respaldo, que puede surgir problemas de limitación cuando un gran número de pasos se ejecutan simultáneamente.

### <a name="azure-portal"></a>Azure Portal

+ **Nuevas características:**
  + Experiencia del editor de informes de la tabla nueva arrastrar y colocar. Los usuarios podrán arrastrar una columna desde el área para el área de la tabla donde se mostrará una vista previa de la tabla. Se pueden reorganizar las columnas.
  + Nuevo Visor del archivo de registros
  + Vínculos a experimentar ejecuciones, proceso, modelos, imágenes y las implementaciones desde la pestaña de actividades

### <a name="azure-machine-learning-data-prep-sdk-v1015"></a>SDK v1.0.15 de preparación de datos de Azure Machine Learning

+ **Nuevas características:**
  + Ahora admite la escritura de secuencias desde un flujo de datos de archivo de preparación de datos. También proporciona la capacidad de manipular los nombres de la secuencia de archivo para crear nuevos nombres de archivo.
    + Guía de procedimientos: [Cuaderno de trabajar con secuencias de archivo](https://aka.ms/aml-data-prep-file-stream-nb)

+ **Mejoras y correcciones de errores**
  + Rendimiento mejorado de t-Digest en grandes conjuntos de datos.
  + Preparación de datos ahora admite la lectura de datos desde una ruta de datos.
  + Ahora se activa una codificación funciona en las columnas booleanos y numéricas.
  + Otras correcciones de errores.

## <a name="2019-02-11"></a>2019-02-11

### <a name="azure-machine-learning-sdk-for-python-v1015"></a>SDK de Azure Machine Learning para Python v1.0.15

+ **Nuevas características:**
  + Canalizaciones de Azure Machine Learning agregado AzureBatchStep ([Bloc de notas](https://aka.ms/pl-azbatch)), HyperDriveStep (portátil) y la funcionalidad de programación basado en tiempo ([Bloc de notas](https://aka.ms/pl-schedule)).
  +  La funcionalidad DataTranferStep se actualizó para funcionar con Azure Database para PostgreSQL ([bloc de notas](https://aka.ms/pl-data-trans)).

+ **Cambios**
  + Entra en desuso `PublishedPipeline.get_published_pipeline` en favor de `PublishedPipeline.get`.
  + Entra en desuso `Schedule.get_schedule` en favor de `Schedule.get`.

### <a name="azure-machine-learning-data-prep-sdk-v1012"></a>SDK de preparación de datos de Azure Machine Learning v1.0.12

+ **Nuevas características:**
  + La preparación de los datos admite la lectura desde una base de datos de Azure SQL con Almacén de datos.
 
+ **Cambios**
  + Mejorar el rendimiento de memoria de ciertas operaciones en los datos de gran tamaño.
  + La opción `read_pandas_dataframe()` requiere que se especifique `temp_folder`.
  + La propiedad `name` en `ColumnProfile` está en desuso, use `column_name` en su lugar.

## <a name="2019-01-28"></a>28-01-2019

### <a name="azure-machine-learning-sdk-for-python-v1010"></a>SDK de Azure Machine Learning para Python v1.0.10

+ **Cambios**: 
  + SDK de Azure Machine Learning ya no tiene los paquetes azure-cli como dependencia. En concreto, las dependencias azure-cli-core y azure-cli-profile azureml-core se quitaron de azureml-core. Estos son los cambios que afectan al usuario:
    + Si va a realizar "az login" y, a continuación, utiliza aprendizaje automático de Azure sdk, el SDK hará el registro de código del explorador o dispositivo en una vez más. No usará ningún estado de las credenciales creado por "az login".
    + Para la autenticación de la CLI de Azure, como el uso de "az login", use la clase _azureml.core.authentication.AzureCliAuthentication_. Para la autenticación de la CLI de Azure, siga _pip install azure-cli_ en el entorno de Python donde ha instalado azureml-sdk.
    + Si está realizando "az login" mediante una entidad de servicio para la automatización, se recomienda usar la clase _azureml.core.authentication.ServicePrincipalAuthentication_, ya que azureml-sdk no usará el estado de las credenciales creadas mediante la CLI de Azure. 

+ **Correcciones de errores**: Esta versión contiene principalmente correcciones de errores menores.

### <a name="azure-machine-learning-data-prep-sdk-v108"></a>SDK de preparación de datos de Azure Machine Learning v1.0.8

+ **Correcciones de errores**
  + Mejorar el rendimiento de la obtención de los perfiles de datos.
  + Se corrigieron los errores menores relacionados con los informes de error.
  
### <a name="azure-portal-new-features"></a>Azure Portal: nuevas características
+ Nuevo experiencia de arrastrar y colocar gráficos en informes. Los usuarios podrán arrastrar una columna o atributo desde el pozo al área del gráfico donde el sistema seleccionará automáticamente un tipo de gráfico adecuado para el usuario según el tipo de datos. Los usuarios pueden cambiar el tipo de gráfico por otros tipos aplicables o agregar atributos adicionales.

    Tipos de gráficos compatibles:
    - Gráfico de líneas
    - Histograma
    - Gráfico de barras apiladas
    - Diagrama de caja
    - Gráfico de dispersión
    - Gráfico de burbujas
+ Ahora el portal genera dinámicamente los informes para los experimentos. Cuando un usuario envía una ejecución para un experimento, se generará automáticamente un informe con las métricas registradas y los grafos a fin de permitir la comparación entre distintas ejecuciones. 

## <a name="2019-01-14"></a>14-01-2019

### <a name="azure-machine-learning-sdk-for-python-v108"></a>SDK de Azure Machine Learning para Python v1.0.8

+ **Correcciones de errores**: Esta versión contiene principalmente correcciones de errores menores.

### <a name="azure-machine-learning-data-prep-sdk-v107"></a>SDK de preparación de datos de Azure Machine Learning v1.0.7

+ **Nuevas características:**
  + Mejoras de almacén de datos (documentadas en la [guía sobre almacenes de datos](https://aka.ms/aml-data-prep-datastore-nb))
    + Se agregó la capacidad de leer y escribir en el recurso compartido de archivos de Azure y almacenes de datos de ADLS en escalado vertical.
    + Al utilizar almacenes de datos, la preparación de datos ahora admite el uso de autenticación de la entidad de servicio en lugar de la autenticación interactiva.
    + Se agregó compatibilidad para las direcciones URL wasb y wasbs.

## <a name="2019-01-09"></a>09-01-2019

### <a name="azure-machine-learning-data-prep-sdk-v106"></a>SDK de preparación de datos de Azure Machine Learning v1.0.6

+ **Correcciones de errores**
  + Se corrigió el error en la lectura desde contenedores de Azure Blob públicamente legibles en Spark

## <a name="2018-12-20"></a>20-12-2018 

### <a name="azure-machine-learning-sdk-for-python-v106"></a>SDK de Azure Machine Learning para Python v1.0.6
+ **Correcciones de errores**: Esta versión contiene principalmente correcciones de errores menores.

### <a name="azure-machine-learning-data-prep-sdk-v104"></a>SDK de preparación de datos de Azure Machine Learning v1.0.4

+ **Nuevas características:**
  + La función `to_bool` ahora permite que los valores no coincidentes se convierten en valores Error. Este es el nuevo comportamiento de error de coincidencia predeterminado para `to_bool` y `set_column_types`, mientras que el comportamiento predeterminado anterior era convertir los valores no coincidentes en False.
  + Al llamar a `to_pandas_dataframe`, hay una nueva opción para interpretar los valores que faltan o NULL en columnas numéricas como NaN.
  + Se agregó la posibilidad de comprobar el tipo de valor devuelto de algunas expresiones para garantizar la coherencia de los tipos y errores pronto.
  + Ahora puede llamar a `parse_json` para analizar los valores de una columna como objetos JSON y expandirlos en varias columnas.

+ **Correcciones de errores**
  + Se ha corregido un error que bloqueaba a `set_column_types` en Python 3.5.2.
  + Se ha corregido un error que ocasionaba un bloqueo al conectarse al almacén de datos mediante una imagen de Azure Machine Learning.

+ **Actualizaciones**
  * [Cuadernos de ejemplo](https://aka.ms/aml-data-prep-notebooks) para obtener tutoriales de introducción, casos prácticos y guías de procedimientos.

## <a name="2018-12-04-general-availability"></a>04-12-2018: Disponibilidad general

Azure Machine Learning Service está disponible con carácter general.

### <a name="azure-machine-learning-compute"></a>Proceso de Azure Machine Learning
Con esta versión, anunciamos una nueva experiencia de proceso administrado a través del [Proceso de Machine Learning](how-to-set-up-training-targets.md#amlcompute). Este destino de proceso reemplaza el proceso de Azure Batch AI para Azure Machine Learning. 

Este destino de proceso:
+ Se usa para las inferencias de lotes y el entrenamiento del modelo
+ Es un proceso de uno a varios nodos
+ Realiza la administración del clúster y la programación de trabajos para el usuario
+ Se escala automáticamente de forma predeterminada
+ Es compatible con recursos de CPU y GPU 
+ Permite el uso de máquinas virtuales de prioridad baja por un costo reducido

El proceso de Azure Machine Learning se puede crear en Python, mediante Azure Portal o la CLI. Debe crearse en la región de su área de trabajo y no se puede asociar a ninguna otra área de trabajo. Este destino de proceso usa un contenedor de Docker para su ejecución y empaqueta sus dependencias para replicar el mismo entorno en todos sus nodos.

> [!Warning]
> Recomendamos crear una nueva área de trabajo para usar el proceso de Azure Machine Learning. Existe la remota posibilidad de que los usuarios que tratan de crear el proceso de Azure Machine Learning a partir de un área de trabajo existente puedan ver un error. El proceso existente en su área de trabajo debería continuar funcionando del modo habitual.

### <a name="azure-machine-learning-sdk-for-python-v102"></a>SDK de Azure Machine Learning para Python v1.0.2
+ **Cambios importantes**
  + Con esta versión, eliminamos el soporte para crear una máquina virtual desde Azure Machine Learning. Aún puede asociar una máquina virtual de la nube existente o un servidor local remoto. 
  + También eliminamos el soporte para BatchAI, que ahora debe recibirlo a través del proceso de Azure Machine Learning.

+ **Nuevo**
  + Para las canalizaciones de Machine Learning:
    + [EstimatorStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.estimator_step.estimatorstep?view=azure-ml-py)
    + [HyperDriveStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.hyper_drive_step.hyperdrivestep?view=azure-ml-py)
    + [MpiStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.mpi_step.mpistep?view=azure-ml-py)


+ **Updated**
  + Para las canalizaciones de Machine Learning:
    + [DatabricksStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.databricks_step.databricksstep?view=azure-ml-py) ahora acepta runconfig
    + [DataTransferStep](https://docs.microsoft.com/python/api/azureml-pipeline-steps/azureml.pipeline.steps.data_transfer_step.datatransferstep?view=azure-ml-py) ahora se copia en y desde un origen de datos de SQL
    + Programación de la funcionalidad en el SDK para crear y actualizar programaciones a fin de ejecutar canalizaciones publicadas

<!--+ **Bugs fixed**-->

### <a name="azure-machine-learning-data-prep-sdk-v052"></a>SDK de preparación de datos de Azure Machine Learning v0.5.2
+ **Cambios importantes** 
  * Se ha cambiado el nombre de `SummaryFunction.N` a `SummaryFunction.Count`.
  
+ **Correcciones de errores**
  * Use el token de ejecución de AML al leer desde almacenes de datos de ejecuciones remotas y escribir en estos. Anteriormente, si el token de ejecución de AML se actualizaba en Python, el tiempo de ejecución de preparación de datos no se actualizaba con el token de ejecución de AML actualizado.
  * Mensajes de error más claros adicionales
  * to_spark_dataframe() ya no se bloqueará cuando Spark use la serialización `Kryo`
  * Ahora, el inspector del recuento de valores puede mostrar más de 1000 valores únicos
  * La división aleatoria ya no producirá ningún error si el flujo de datos original no tiene nombre  

+ **Más información**
  * [SDK de preparación de datos de Azure Machine Learning](https://aka.ms/data-prep-sdk)

### <a name="docs-and-notebooks"></a>Documentos y cuadernos
+ Canalizaciones de Machine Learning
  + Cuadernos nuevos y actualizados para empezar a trabajar con canalizaciones, ámbitos de lote y ejemplos de transferencia del estilo: https://aka.ms/aml-pipeline-notebooks
  + Aprenda a [crear su primera canalización](how-to-create-your-first-pipeline.md)
  + Aprenda a [ejecutar predicciones por lotes mediante canalizaciones](how-to-run-batch-predictions.md)
+ Destino de proceso de Azure Machine Learning
  + Los [cuadernos de ejemplo](https://aka.ms/aml-notebooks) ahora están actualizados para usar este nuevo proceso administrado.
  + [Obtenga información sobre este proceso](how-to-set-up-training-targets.md#amlcompute)

### <a name="azure-portal-new-features"></a>Azure Portal: nuevas características
+ Cree y administre los tipos de [proceso de Azure Machine Learning](how-to-set-up-training-targets.md#amlcompute) en el portal.
+ Supervise el uso de cuota y [solicite una cuota](how-to-manage-quotas.md) para el proceso de Azure Machine Learning.
+ Vea el estado del clúster del Proceso de Machine Learning en tiempo real.
+ La compatibilidad con la red virtual se agregó para la creación de Azure Kubernetes Service y el proceso de Azure Machine Learning.
+ Vuelva a ejecutar sus canalizaciones publicadas con los parámetros existentes.
+ Nuevos [gráficos de aprendizaje automático automatizados](how-to-track-experiments.md#auto) para modelos de clasificación (gráfico de importancia de características, elevación, ganancias y calibración con explicabilidad de modelo) y modelos de regresión (gráfico de importancia de características y valores residuales con explicabilidad de modelo). 
+ Las canalizaciones se pueden ver en Azure Portal




## <a name="2018-11-20"></a>2018-11-20

### <a name="azure-machine-learning-sdk-for-python-v0180"></a>SDK de Azure Machine Learning para Python v0.1.80

+ **Cambios importantes** 
  * El espacio de nombres *azureml.train.widgets* se ha movido a *azureml.widgets*.
  * *azureml.core.compute.AmlCompute* deja de usar las siguientes clases: *azureml.core.compute.BatchAICompute* y *azureml.core.compute.DSVMCompute*. La última clase se quitará en versiones posteriores. Ahora la clase AmlCompute tiene una definición más fácil. Solo necesita los valores de vm_size y max_nodes, y escala automáticamente su clúster de 0 al valor de max_nodes cuando se envía un trabajo. Nuestros [cuadernos de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/training) se actualizaron con esta información y deberían ofrecer ejemplos de uso. Esperamos que disfrute de esta simplificación y de una gran cantidad de características atractivas adicionales en una versión posterior.

### <a name="azure-machine-learning-data-prep-sdk-v051"></a>SDK de preparación de datos de Azure Machine Learning v0.5.1 

Obtenga más información sobre el SDK de preparación de datos mediante la lectura de [documentos de referencia](https://aka.ms/data-prep-sdk).
+ **Nuevas características**
   * Se ha creado una nueva CLI de DataPrep para ejecutar paquetes de DataPrep y ver el perfil de datos de un conjunto de datos o del flujo de datos.
   * Se ha rediseñado la API de SetColumnType para mejorar la facilidad de uso.
   * El nombre del archivo smart_read_file se ha cambiado por auto_read_file.
   * Ahora incluye sesgo y curtosis en el perfil de datos.
   * Puede realizar un muestreo con el muestreo estratificado.
   * Puede leer archivos ZIP que contienen archivos CSV.
   * Puede dividir conjuntos de datos por filas con la división aleatoria (por ejemplo, en conjuntos de prueba-aprendizaje).
   * Puede obtener los tipos de datos de todas las columna de un flujo de datos o un perfil de datos mediante una llamada a `.dtypes`.
   * Puede obtener el recuento de filas de un flujo de datos o un perfil de datos mediante una llamada a `.row_count`.

+ **Correcciones de errores**
   * Se ha corregido la conversión de larga a doble. 
   * Se ha corregido la aserción después de cualquier adición de columna. 
   * Se ha corregido un problema con FuzzyGrouping, en que no se detectaban grupos en algunos casos.
   * Se ha corregido la función de ordenación para respetar el criterio de ordenación de varias columnas.
   * Se han corregido las expresiones and/or para que se gestionen como en `pandas`.
   * Se ha corregido la lectura de la ruta de acceso de dbfs.
   * Se ha simplificado la comprensión de los mensajes de error. 
   * Ya no se produce ningún error al leer en el destino de proceso remoto mediante el token AML.
   * Ya no se produce ningún error en Linux DSVM.
   * Ya no se bloquea cuando los valores que no son de cadena están en predicados de cadena.
   * Ahora controla correctamente los errores de aserción cuando el flujo de datos generaba un error.
   * Ahora es compatible con las ubicaciones de almacenamiento montadas en dbutils en Azure Databricks.

## <a name="2018-11-05"></a>05-11-2018

### <a name="azure-portal"></a>Azure Portal 
Azure Portal para el servicio Azure Machine Learning tiene las siguientes actualizaciones:
  * Una nueva pestaña **Canalizaciones** para las canalizaciones publicadas.
  * Se agregó compatibilidad para adjuntar un clúster de HDInsight existente como un destino de proceso.

### <a name="azure-machine-learning-sdk-for-python-v0174"></a>SDK de Azure Machine Learning para Python v0.1.74

+ **Cambios importantes** 
  * *Workspace.compute_targets, almacenes de datos, experimentos, imágenes, modelos y *servicios web* son propiedades en lugar de métodos. Por ejemplo, sustituya *Workspace.compute_targets()* por *Workspace.compute_targets*.
  * *Run.get_context* reemplaza a *Run.get_submitted_run*. El último método se quitará en versiones posteriores.
  * La clase *PipelineData* espera ahora un objeto de almacén de datos como un parámetro en lugar de datastore_name. De forma similar, *Canalización* acepta default_datastore en lugar de default_datastore_name.

+ **Nuevas características:**
  * El [cuaderno de ejemplo](https://github.com/Azure/MachineLearningNotebooks/tree/master/pipeline/pipeline-mpi-batch-prediction.ipynb) de las canalizaciones de Azure Machine Learning ahora usa los pasos de MPI.
  * El widget RunDetails para instancias de Jupyter Notebook se actualiza para mostrar una visualización de la canalización.

### <a name="azure-machine-learning-data-prep-sdk-v040"></a>SDK de preparación de datos de Azure Machine Learning v0.4.0 
 
+ **Nuevas características:**
  * Recuento de tipos agregados al perfil de datos. 
  * Recuento de valores e histograma ahora disponibles.
  * Más percentiles en el perfil de datos.
  * La mediana está disponible en el resumen.
  * Ahora se admite Python 3.7.
  * Cuando se guarda un flujo de datos que contiene almacenes de datos en un paquete DataPrep, la información del almacén de datos se guardará como parte del paquete DataPrep.
  * Ahora se admite la escritura en el almacén de datos. 
        
+ **Error corregido**
  * Los desbordamientos de enteros sin signo de 64 bits ahora se controlan correctamente en Linux.
  * Etiqueta de texto incorrecta corregida para archivos de texto sin formato en smart_read.
  * El tipo de columna de cadenas se muestra ahora en la vista de métricas.
  * El recuento de tipos ahora se corrige para mostrar los tipos de valores asignados a un tipo de campo única en lugar de valores individuales.
  * Write_to_csv ya no genera ningún error cuando la ruta de acceso se proporciona como una cadena.
  * Cuando se usa Reemplazar, dejar "Buscar" en blanco no generará ningún error. 

## <a name="2018-10-12"></a>12 de octubre de 2018

### <a name="azure-machine-learning-sdk-for-python-v0168"></a>SDK de Azure Machine Learning para Python v0.1.68

+ **Nuevas características:**
  * Compatibilidad con varios inquilinos al crear una nueva área de trabajo.

+ **Errores corregidos**
  * Ya no es necesario anclar la versión de la biblioteca pynacl durante la implementación del servicio web.

### <a name="azure-machine-learning-data-prep-sdk-v030"></a>SDK de preparación de datos de Azure Machine Learning v0.3.0

+ **Nuevas características:**
  * Se ha agregado el método transform_partition_with_file(script_path), lo que permite a los usuarios usar la ruta de acceso del archivo de Python que va a ejecutar

## <a name="2018-10-01"></a>01 de octubre de 2018

### <a name="azure-machine-learning-sdk-for-python-v0165"></a>SDK de Azure Machine Learning para Python v0.1.65
La [versión 0.1.65](https://pypi.org/project/azureml-sdk/0.1.65) incluye nuevas características, más documentación, correcciones de los errores y más [cuadernos de ejemplo](https://aka.ms/aml-notebooks).

Para obtener información acerca de errores conocidos y soluciones alternativas, consulte [la lista de problemas conocidos](resource-known-issues.md).

+ **Cambios importantes**
  * Workspace.experiments, Workspace.models, Workspace.compute_targets, Workspace.images, Workspace.web_services devuelven una lista de diccionarios devuelto que se ha devuelto previamente. Consulte la documentación de la API [azureml.core.Workspace](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace(class)?view=azure-ml-py).

  * Automated Machine Learning ha eliminado un error de la media cuadrática normalizada de la métrica principal.

+ **HyperDrive**
  * Varias correcciones de errores de HyperDrive para obtener mejoras bayesianas del rendimiento para la obtención de llamadas a métricas. 
  * Actualización de Tensorflow 1.9 a 1.10 
  * Optimización de la imagen de Docker para el arranque en frío. 
  * Los trabajos ahora informan acerca del estado correcto, aunque salgan con un código de error que no sea 0. 
  * Validación del atributo RunConfig en SDK. 
  * El objeto de ejecución de HyperDrive admite una cancelación similar a la de una ejecución normal: no es preciso usar parámetros. 
  * Mejoras en los widgets para mantener el estado de los valores de la lista desplegable de ejecuciones distribuidas y ejecuciones de HyperDrive. 
  * Se ha corregido el soporte de TensorBoard y otros archivos de registro para el servidor del parámetro. 
  * Compatibilidad con Intel (R) MPI en el lado del servicio. 
  * Corrección de errores en el ajuste de parámetros para la corrección de ejecuciones distribuidas durante la validación de BatchAI. 
  * El administrador de contextos ahora identifica la instancia principal. 

+ **Experiencia de Azure Portal**
  * log_table() y log_row() se admiten en los detalles de la ejecución. 
  * Creación automáticamente de gráficos para tablas y filas con 1, 2 o 3 columnas numéricas y una columna de categorías opcional.

+ **Automated Machine Learning**
  * Mejora en la control de errores y la documentación 
  * Se han corregido los problemas de rendimiento de la recuperación de la propiedad de ejecución. 
  * Se ha corregido el problema de la ejecución continua. 
  * Se han corregido los problemas de iteración del ensamblado.
  * Se ha corregido el error de falta de respuesta del aprendizaje en MAC OS.
  * Reducción de la curva de PR/ROC media de macros en un escenario de validación personalizada.
  * Se ha eliminado la lógica de índices adicional.
  * Se ha eliminado el filtro de la API get_output.

+ **Procesos**
  * Agrega un método Pipeline.publish() para publicar una canalización directamente, sin necesidad de que antes se realice una ejecución.   
  * Se ha agregado un método PipelineRun.get_pipeline_runs() para capturar las ejecuciones de la canalización que se generaron desde una canalización publicada.

+ **Project Brainwave**
  * Compatibilidad actualizada con los nuevos modelos de IA disponibles en FPGA.

### <a name="azure-machine-learning-data-prep-sdk-v020"></a>SDK de preparación de datos de Azure Machine Learning v0.2.0
La [versión 0.2.0](https://pypi.org/project/azureml-dataprep/0.2.0/) incluye las siguientes características y correcciones de errores:

+ **Nuevas características:**
  * Compatibilidad con la codificación "one-hot"
  * Compatibilidad con la transformación de cuantiles
   
+ **Error corregido:**
  * Funciona con cualquier versión de Tornado sin necesidad de bajar de versión de Tornado
  * Recuentos de valores para todos los valores, no solo para los tres primeros

## <a name="2018-09-public-preview-refresh"></a>2018-09 (Actualización de la versión preliminar pública)

Una versión nueva y actualizada de Azure Machine Learning: obtenga más información sobre esta versión: https://azure.microsoft.com/blog/what-s-new-in-azure-machine-learning-service/


## <a name="next-steps"></a>Pasos siguientes

Consulte la introducción a [Azure Machine Learning Service](../service/overview-what-is-azure-ml.md).
