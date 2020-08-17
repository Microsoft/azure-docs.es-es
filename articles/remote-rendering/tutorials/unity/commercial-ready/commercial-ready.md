---
title: Creación de una aplicación de Azure Remote Rendering lista para su comercialización
description: Estrategias y consideraciones para la creación de una aplicación lista para su comercialización con Azure Remote Rendering
author: FlorianBorn71
ms.author: flborn
ms.date: 06/15/2020
ms.topic: tutorial
ms.openlocfilehash: e827f7eff707f5a7c467f53eacab6973bff2ef2f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076432"
---
# <a name="tutorial-creating-a-commercial-ready-azure-remote-rendering-application"></a>Tutorial: Creación de una aplicación de Azure Remote Rendering lista para su comercialización

En este tutorial, obtendrá información sobre lo siguiente:

> [!div class="checklist"]
>
> * Administración de sesiones para aplicaciones comerciales
> * Seguimiento de sesiones para la facturación
> * Optimización de la experiencia del usuario en torno al tiempo de carga de la sesión
> * Consideraciones sobre la latencia de red

## <a name="prerequisites"></a>Requisitos previos

* Este tutorial se basa en [Tutorial: Protección de Azure Remote Rendering y almacenamiento de modelos](../security/security.md).

## <a name="introduction-to-commercial-readiness"></a>Introducción al concepto de aplicación lista para su comercialización

Azure Remote Rendering amplía lo que es posible realizar con la realidad mixta. Una vez que los conceptos básicos están integrados en la solución, hay una serie de consideraciones adicionales para asegurarse de que la solución es segura y escalable y está lista para ofrecer valor.

Este módulo presenta algunas funcionalidades adicionales que puede tener en cuenta para su aplicación comercial.

Para obtener una amplia información general sobre los procedimientos recomendados de arquitectura en todo el sistema, consulte:

* [Centro de arquitectura de Azure](https://docs.microsoft.com/azure/architecture/)
* [Guía de introducción para desarrolladores de Azure](https://docs.microsoft.com/azure/guides/developer/azure-developer-guide)

## <a name="analytics"></a>Análisis

La integración de herramientas de análisis puede ayudarle a administrar, controlar y mejorar la solución.

Para obtener una lista completa de los recursos de análisis disponibles, consulte:

* [Servicios de Azure Analytics](https://azure.microsoft.com/product-categories/analytics/)

### <a name="tracking-usage-for-billing"></a>Seguimiento del uso para la facturación

El seguimiento del consumo de Azure Remote Rendering por parte de varios equipos internos o clientes externos es una consideración importante, especialmente en situaciones de varios inquilinos.

Para lograr esto, Azure ofrece un servicio llamado etiquetado de recursos, que asocia el consumo del servicio Azure Remote Rendering con cada cliente.

Para más información sobre el etiquetado y la asignación de nombres de recursos, un buen punto de partida es:

* [Guía de decisiones de nomenclatura y etiquetado de recursos](https://docs.microsoft.com/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json)

### <a name="diagnostics"></a>Diagnóstico

Herramientas eficaces como el Seguimiento de eventos para Windows (ETW) y el Registro de seguimiento de eventos (ETL) facilitan la generación de eventos de seguimiento dentro de la aplicación y pueden ayudar a diagnosticar problemas de red, de ingesta de contenido, de sesión o de aplicación y otros problemas que pueden surgir en la implementación de una solución comercial.

Para más información, consulte:

* [Creación de seguimientos del rendimiento en el cliente](https://docs.microsoft.com/azure/remote-rendering/how-tos/performance-tracing)
* [Procedimiento para recopilar datos de seguimiento de eventos para Windows (ETW)](https://docs.microsoft.com/visualstudio/profiling/how-to-collect-event-tracing-for-windows-etw-data)
* [Uso del Portal de dispositivos Windows: registro](https://docs.microsoft.com/windows/mixed-reality/using-the-windows-device-portal)

### <a name="usage-analysis"></a>Análisis del uso

Azure Application Insights le ayuda a comprender cómo usan las personas su aplicación de Azure Remote Rendering. Cada vez que actualice la aplicación, podrá evaluar cómo funciona de cara a los usuarios y mejorar la solución en consecuencia. Con este conocimiento, puede tomar decisiones basadas en datos sobre los ciclos de desarrollo siguientes.

Para más información, consulte:

* [Análisis del uso con Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/usage-overview)

## <a name="fast-startup-time-strategies"></a>Estrategias de tiempo de inicio rápido

El caso de uso puede requerir un inicio rápido desde el inicio de la aplicación hasta la visualización del modelo 3D. Por ejemplo, durante una reunión importante en la que es fundamental tener todo en funcionamiento con anterioridad. Otro ejemplo es durante la revisión de un modelo 3D de CAD, en el que la iteración de diseño rápida entre una aplicación CAD y la realidad mixta es fundamental para la eficacia.

Azure Remote Rendering requiere modelos 3D procesados previamente y Azure actualmente tarda varios minutos en crear una máquina virtual y cargar un modelo para su representación. Para que este proceso sea lo más sencillo y rápido posible, es necesario preparar los datos del modelo 3D y la sesión de ARR de antemano.

Las sugerencias compartidas aquí no forman parte de Azure Remote Rendering estándar, pero puede implementarlas por su cuenta para agilizar los tiempos de inicio.

### <a name="initiate-early"></a>Inicio temprano

Para reducir el tiempo de inicio, la solución más sencilla consiste en trasladar la creación y la inicialización de la máquina virtual lo antes posible en el flujo de trabajo del usuario. Una estrategia consiste en inicializar la sesión tan pronto se sabe que se necesitará una sesión de ARR. Suele ser cuando el usuario empieza a cargar un modelo 3D en Azure Blob Storage para usarlo con Azure Remote Rendering. En este caso, la creación de la sesión y la inicialización de la máquina virtual se pueden iniciar al mismo tiempo que la carga del modelo 3D, de modo que ambos flujos de trabajo se ejecuten en paralelo.

Este proceso se puede reducir aún más si se asegura de que los contenedores de entrada y salida de Azure Blob Storage elegidos están en el mismo centro de datos regional que la sesión de Azure Remote Rendering.

### <a name="scheduling"></a>Scheduling

Si sabe que va a tener una necesidad futura de Azure Remote Rendering, puede programar una fecha y hora específicas para iniciar la sesión de Azure Remote Rendering.

Esta opción se puede ofrecer mediante un portal web en el que los usuarios pueden cargar un modelo 3D y programar una hora para verlo en el futuro. Este también sería un buen lugar para solicitar otras preferencias, como la representación Estándar o Premium. La representación Premium puede ser adecuada si se desea mostrar una combinación de recursos en la que el tamaño ideal es más difícil de determinar automáticamente o si existe una necesidad de asegurarse de que la región de Azure tiene máquinas virtuales disponibles en el momento especificado.

### <a name="session-pooling"></a>Agrupación de sesiones

En las situaciones más exigentes, otra opción es la agrupación de sesiones, en la que se crean e inicializan en todo momento una o varias sesiones. Esto crea un grupo de sesiones para su uso inmediato por parte de un usuario que lo solicite. El inconveniente de este enfoque es que, una vez inicializada la máquina virtual, se inicia la facturación del servicio. Puede que no sea rentable mantener un grupo de sesiones en ejecución en todo momento, pero en función de un análisis, es posible predecir las cargas máximas o se puede combinar con la estrategia de programación anterior para predecir cuándo se necesitarán las sesiones y aumentar y disminuir el grupo de sesiones en consecuencia.

Esta estrategia también ayuda a optimizar la elección entre las sesiones Estándar y Premium de forma más dinámica, ya que es mucho más rápido cambiar entre los dos tipos dentro de una sesión de un solo usuario, como el caso en el que se visualiza primero un modelo de complejidad Premium, seguido de uno que puede funcionar en el nivel Estándar. Si estas sesiones de usuario son bastante largas, puede haber un ahorro significativo en los costos.

Para más información sobre las sesiones de Azure Remote Rendering, consulte:

* [Sesiones de Remote Rendering](https://docs.microsoft.com/azure/remote-rendering/concepts/sessions)

## <a name="standard-vs-premium-vm-routing-strategies"></a>Estrategias de enrutamiento de máquina virtual Estándar frente a Premium

La necesidad de seleccionar si se crea una máquina virtual Estándar o Premium supone un reto en el diseño de la experiencia del usuario y el sistema de un extremo a otro. Aunque utilizar únicamente sesiones Premium es una opción, las sesiones Estándar usan muchos menos recursos de proceso de Azure y son menos costosas que las Premium. Esto proporciona una motivación firme para usar sesiones Estándar siempre que sea posible y usar las Premium solo cuando sea necesario.

Aquí se comparten varias opciones, desde las menos completas hasta las más completas, para abordar la necesidad de administrar las opciones de sesión.

### <a name="use-only-standard-or-premium"></a>Usar solo Estándar o solo Premium

Si está seguro de que sus necesidades *siempre* quedan por debajo del umbral entre Estándar y Premium, esto simplifica considerablemente su decisión. Utilice solo Estándar. Tenga en cuenta que el impacto en la experiencia del usuario es significativo si se rechaza la suma total de la complejidad de los recursos cargados por ser demasiado compleja para una sesión Estándar.

Del mismo modo, si se espera que una gran parte de los usos superen el umbral entre Estándar y Premium o bien el costo no es un factor clave en el caso de uso, elegir siempre Premium también es una opción para simplificar.

### <a name="ask-the-user"></a>Preguntar al usuario

Si desea admitir tanto Estándar como Premium, la manera más fácil de determinar el tipo de sesión de máquina virtual que se va a usar para crear una instancia es preguntar al usuario cuando seleccione los recursos 3D para verlos. El reto de este enfoque es que requiere que el usuario conozca la complejidad del recurso 3D (o incluso de varios recursos) que se va a visualizar. Normalmente, no se recomienda por esa razón. Si el usuario selecciona incorrectamente y elige Estándar, la experiencia de usuario resultante podría verse comprometida en un momento inoportuno.

### <a name="analyze-the-3d-model"></a>Analizar el modelo 3D

Otro enfoque relativamente sencillo consiste en analizar la complejidad de los recursos 3D seleccionados. Si la complejidad del modelo está por debajo del umbral de Estándar, inicie una sesión Estándar; de lo contrario, inicie una sesión Premium. En este caso, el desafío es que se puede usar una sola sesión para ver varios modelos de los cuales algunos pueden superar el umbral de complejidad de una sesión Estándar, lo que da lugar a la imposibilidad de usar la misma sesión sin problemas para una secuencia de recursos 3D diferentes.

### <a name="automatic-switching"></a>Cambio automático

El cambio automático entre las sesiones Estándar y Premium puede tener mucho sentido en un diseño del sistema que también incluya la agrupación de sesiones. Esta estrategia permite optimizar aún más el uso de los recursos. A medida que el usuario carga modelos para verlos, se determina la complejidad y se solicita el tamaño de sesión correcto al servicio de agrupación de sesiones.

## <a name="working-with-networks"></a>Trabajo con redes

### <a name="diagnostics"></a>Diagnóstico

Azure Remote Rendering requiere una conexión rápida a Internet con baja latencia. La calidad de la red del usuario puede tener un impacto significativo en la calidad de la experiencia. Dado que es probable que los clientes tengan diferentes configuraciones de red y, en ocasiones, una latencia de red deficiente, las herramientas de diagnóstico resultan fundamentales.  

Para asegurarse de que puede ofrecer una experiencia de alta calidad de forma coherente, se recomienda integrar herramientas de análisis de cliente y de servidor en las aplicaciones de Azure Remote Rendering. Al hacerlo, se le proporciona la información que necesita para diagnosticar y mitigar los problemas de red que puedan experimentar los clientes.

### <a name="client-network-configurations"></a>Configuraciones de red de cliente

Uno de los mayores desafíos en el desarrollo de soluciones de colaboración sólidas que se implementan en una amplia variedad de entornos empresariales consiste en estar preparado para las diferentes topologías de red y configuraciones de firewall de empresa que pueden usar los clientes.

Muchas empresas bloquean todo el tráfico punto a punto dentro de una LAN. Esto hace que sea difícil aprovechar la simplicidad y la experiencia de usuario simplificada de la detección automática de LAN para establecer una sesión compartida local entre todas las instancias detectadas de la aplicación de realidad mixta.

Otros posibles puntos de error son los enrutadores configurados para limitar intencionadamente el ancho de banda y los firewalls que bloquean la mayoría de los puertos TCP/IP.

Siempre que tenga previsto usar Azure Remote Rendering en una red desconocida, se recomienda lo siguiente:

* Proporcione una lista de comprobación previa a la reunión para evaluar la preparación de la red.
* Asegúrese de que el centro de datos regional correspondiente puede atender la solicitud.
* Asigne tiempo suficiente para diagnosticar cualquier problema.
* Incorpore una zona activa móvil con un plan de datos con un ancho de banda alto como alternativa de respaldo.

### <a name="end-to-end-bandwidth"></a>Ancho de banda de un extremo a otro

Es importante evaluar las capacidades de ancho de banda de cada segmento de la red que pueda existir entre la máquina virtual de Azure Remote Rendering y el cliente final. Tenga en cuenta que el segmento de red que va desde el centro de datos de Azure al ISP del cliente puede ser un factor con más limitaciones que el que va del ISP al cliente. Se puede usar la prueba de velocidad de descarga de blobs para ayudar a diagnosticar estos problemas.

### <a name="bandwidth-competition"></a>Competencia por el ancho de banda

Al diseñar una aplicación de realidad mixta, tenga en cuenta que diferentes características de la aplicación pueden competir con Azure Remote Rendering por el ancho de banda. Un ejemplo imprevisto muy probable se da cuando varios participantes en una única sala esperan para utilizar simultáneamente ARR para visualizar un recurso 3D. Cada uno de los tramos del flujo de datos de red necesitará tener capacidad para transportar la suma total de todos los flujos de datos de ARR combinados.

Otros ejemplos incluyen el vídeo en streaming, las cargas en segundo plano simultáneas de otro contenido relacionado y el chat de voz, especialmente cuando hay muchos participantes y el sistema usa un enfoque de punto a punto distribuido en lugar de un enfoque con un servidor de mezcla de audio.

Para más información sobre el análisis de redes, consulte:

* [Prueba de velocidad de descarga de Azure Storage Blob](https://www.azurespeed.com/Azure/Download)
* [Estadísticas de latencia de recorrido de ida y vuelta de red de Azure](https://docs.microsoft.com/azure/networking/azure-network-latency)
* [Consultas de rendimiento del lado del servidor](https://docs.microsoft.com/azure/remote-rendering/overview/features/performance-queries)
* [Creación de seguimientos del rendimiento en el cliente](https://docs.microsoft.com/azure/remote-rendering/how-tos/performance-tracing)

## <a name="collaboration-considerations"></a>Consideraciones en materia de colaboración

Algunos de los usos más valiosos de Azure Remote Rendering implican la colaboración entre varios participantes que ven la misma experiencia 3D al mismo tiempo. En estas sesiones compartidas, es importante reconocer que cada participante necesitará una sesión única de Azure Remote Rendering, independientemente de si se encuentran en el mismo lugar, en la misma red o no.

Esto se debe a que cada participante realmente ve la misma experiencia desde distintos puntos de vista, lo que requiere que se representen los mismos recursos 3D desde cada una de esas perspectivas de forma simultánea.

### <a name="multiple-azure-remote-rendering-sessions"></a>Varias sesiones de Azure Remote Rendering

Si tiene previsto admitir experiencias compartidas con Azure Remote Rendering, los sistemas que ponga en marcha para crear y administrar sesiones de ARR deberán estar preparados para iniciar varias sesiones. Es posible que sea necesario inicializar estas sesiones en diferentes centros de datos de Azure si los participantes están dispersos geográficamente.

El sistema también debe administrar la posibilidad de que uno o varios de los participantes se encuentren en una región geográfica que actualmente no es compatible con Azure Remote Rendering o que actualmente no tiene disponible ninguna instancia de máquina virtual de Azure Remote Rendering.

Esta administración de varias sesiones simultáneas se puede simplificar aún más cuando se combina con la agrupación de sesiones y otras estrategias descritas en este documento.

### <a name="azure-blob-storage-considerations"></a>Consideraciones sobre Azure Blob Storage

Todas las sesiones de ARR simultáneas pueden hacer referencia al mismo identificador URI de SAS para visualizar el modelo convertido. Esto permite cargar y convertir los recursos 3D deseados una vez y compartirlos en todas las sesiones. Esto es especialmente cierto cuando los participantes están en la misma ubicación y usan el mismo centro de datos, cuando no hay ningún problema de rendimiento relacionado con que la instancia de Azure Blob Storage se encuentre en un centro de datos diferente que el servidor de Azure Remote Rendering y el usuario.

Si normalmente se cargan recursos 3D para una única sesión de visualización y, a continuación, se descartan, como por ejemplo en una sesión de revisión de diseño, la región geográfica de la instancia de Azure Blob Storage en relación con el servidor de Azure Remote Rendering también es menos importante.

Sin embargo, en el caso de recursos 3D que se van a usar de forma repetida, como en un caso de uso de entrenamiento, se recomienda mantener los recursos 3D listos para su uso en el almacenamiento de blobs de cada centro de datos regional en el que planea usar Azure Remote Rendering. Esto se puede automatizar mediante la redundancia de Azure Storage. También se suele usar la red CDN para este propósito, pero aún no es una opción para Azure Remote Rendering.

Para obtener más información:

* [Experiencias compartidas en realidad mixta](https://docs.microsoft.com/windows/mixed-reality/shared-experiences-in-mixed-reality)
* [Redundancia de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-redundancy)

## <a name="managing-model-access"></a>Administración del acceso al modelo

El uso completo de Azure Remote Rendering requiere una consideración cuidadosa de la infraestructura de un extremo a otro para la administración de los modelos 3D.

Una ventaja de usar Azure Remote Rendering es que nunca es necesario transmitir los recursos 3D de gran tamaño directamente al dispositivo de realidad mixta antes de que se vean.  Además, una vez que un recurso 3D se ha cargado y convertido para su uso con Azure Remote Rendering, cualquier número de usuarios puede compartir esa única instancia del modelo 3D.

### <a name="considerations-for-3d-model-access"></a>Consideraciones sobre el acceso al modelo 3D

Estas son algunas consideraciones clave a la hora de decidir la estrategia de acceso al modelo.

En función del caso de uso previsto, determine el mejor lugar o la mejor combinación de lugares para permitir que un usuario seleccione los recursos 3D para verlos. Algunas opciones comunes son:

* Directamente dentro de la experiencia de realidad mixta
* Mediante un portal web complementario
* En una aplicación de escritorio o móvil complementaria

Si el caso de uso tiene patrones de uso en los que se puede cargar el mismo recurso 3D varias veces, el back-end realizará un seguimiento de los modelos que ya se han convertido para su uso con ARR, de modo que un modelo solo se procese previamente una vez para varias selecciones futuras. Se daría un ejemplo de revisión de diseño cuando un equipo tiene acceso a un recurso 3D original común. Se espera que cada miembro del equipo revise el modelo con ARR en algún momento de su flujo de trabajo. Solo la primera visualización desencadenaría el paso de procesamiento previo. Las siguientes visualizaciones buscarían el archivo procesado asociado en el contenedor de salida de SAS.

En función del caso de uso, es probable que desee determinar y potencialmente conservar el tamaño de máquina virtual de Azure Remote Rendering correcto, Estándar o Premium, para cada recurso o grupo de recursos 3D que se va a visualizar en la misma sesión.  

### <a name="on-device-model-selection-list"></a>Lista de selección de modelos en el dispositivo

En muchos casos de uso, como un entrenamiento, una guía de tareas o una aplicación de marketing, el conjunto de recursos 3D que se va a visualizar normalmente en Azure Remote Rendering puede ser bastante estático. En estas situaciones, se puede convertir previamente un conjunto seleccionado de recursos 3D y hacerlos disponibles mediante una base de datos que contiene la información necesaria para rellenar una lista de selección de recursos seleccionados previamente. Estos datos se pueden recuperar desde la aplicación de realidad mixta para rellenar un menú de selección.

El siguiente paso podría ser ofrecer también una manera de cargar recursos 3D privados, únicos para cada individuo o grupo. A continuación, esa lista de recursos privados se podría combinar con la lista de recursos comunes seleccionados en la experiencia del usuario para elegir los recursos 3D que se desea ver.

### <a name="on-device-onedrive-access"></a>Acceso a OneDrive en el dispositivo

Dado que existe un selector de archivos de OneDrive integrado de forma nativa en los dispositivos de realidad mixta de Microsoft, la selección de recursos 3D en el dispositivo desde OneDrive es atractiva, especialmente en casos de uso en los que es habitual cargar modelos 3D diferentes o modificados. En este escenario, el usuario seleccionaría uno o más recursos 3D mediante el selector de archivos de OneDrive dentro de la aplicación de realidad mixta. Los recursos 3D se migrarán a un contenedor de entrada de SAS, se convertirán en un contenedor de salida de SAS y se adjuntarán a la sesión de ARR. Idealmente, la aplicación de realidad mixta invocaría un proceso basado en la nube para realizar estos pasos, en lugar de mover todos los bits desde OneDrive al dispositivo y de vuelta a Azure Blob Storage.

Este enfoque se puede llevar un paso más allá si se conserva una asociación entre los recursos 3D que se han visualizado anteriormente, de modo que cuando se elige de nuevo el mismo modelo desde OneDrive, la aplicación puede omitir el proceso de conversión y cargar directamente el recurso 3D convertido asociado mediante su identificador URI de SAS.

Para obtener más información:

* [Plantilla de Microsoft Power Automate para la replicación de OneDrive en Azure Storage](https://flow.microsoft.com/galleries/public/templates/2f90b5d3-029b-4e2e-ad37-1c0fe6d187fe/when-a-file-is-uploaded-to-onedrive-copy-it-to-azure-storage-container/)
* [Información general sobre la API de almacenamiento de archivos de OneDrive](https://docs.microsoft.com/graph/onedrive-concept-overview)

### <a name="direct-cad-access"></a>Acceso directo a CAD

Un caso de uso atractivo para la realidad mixta son las revisiones de diseño de un trabajo en curso de CAD. En este escenario, el tiempo de carga más rápido entre el escritorio y la realidad mixta es fundamental. Una solución ideal podría implicar el desarrollo de complementos para aplicaciones de CAD específicas. Estos complementos podrían administrar directamente cada aspecto del proceso de carga, conversión y visualización:

* Proporcionar una experiencia de usuario para:
  * Emparejar la aplicación de CAD con un dispositivo de realidad mixta específico (una vez).
  * Solicitar que la geometría seleccionada se visualice en ese dispositivo de realidad mixta.
* Si aún no está en ejecución, iniciar la sesión de Azure Remote Rendering para que pueda procesarse en paralelo mientras se carga y se convierte el archivo de CAD.
* Normalizar los datos de geometría de CAD a uno de los formatos admitidos por Azure Remote Rendering.
* Transmitir los datos normalizados directamente al contenedor de entrada de Azure Blob Storage.
* Iniciar el proceso de conversión del modelo.
* Vincular el identificador URI de SAS del contenedor de salida del modelo a la sesión de Azure Remote Rendering.
* Notificar a la aplicación de realidad mixta emparejada que el modelo está disponible y listo para su visualización y proporcionar el identificador URI de SAS del contenedor de salida para que la aplicación pueda adjuntarlo a la sesión.

Un enfoque mucho más sencillo pero ligeramente menos optimizado podría automatizar el proceso de guardar el modelo 3D en una unidad de disco duro local y, a continuación, iniciar un proceso para transmitir ese archivo guardado al contenedor de entrada de SAS.

### <a name="azure-marketplace"></a>Azure Marketplace

Muchos clientes de empresa exigen que los entornos Azure Stack se puedan implementar con sus propias cuentas y credenciales de Azure por motivos de seguridad. Para que esto sea posible, considere la posibilidad de empaquetar la aplicación administrada de Azure para que se pueda publicar en Azure Marketplace como una oferta de aplicación de Azure.

Para obtener más información:

* [Azure Marketplace](https://azure.microsoft.com/marketplace/)
* [Tutorial: Publicación de aplicaciones administradas de Azure en Marketplace](https://docs.microsoft.com/azure/azure-resource-manager/managed-applications/publish-marketplace-app)

### <a name="security"></a>Seguridad

Es fundamental crear la solución de Azure Remote Rendering de un extremo a otro con vistas a la seguridad desde su comienzo. Hay muchos aspectos de la seguridad que se deben tener en cuenta en el diseño de la solución de un extremo a otro, entre los que se incluyen:

* Estrategias de autenticación
* Administración del acceso: grupos, directivas y permisos
* Servicios multiinquilino
* Cifrado del almacenamiento y la transferencia de datos
* Tokens de uso temporal
* Ataques de denegación de servicio distribuido (DDoS)
* Detección de amenazas
* VPN y redes seguras
* Firewalls
* Administración de certificados y claves secretas
* Vulnerabilidades de la aplicación

Para la autenticación, es aconsejable trasladar la mayor parte posible de la autenticación y la administración de sesiones de ARR a un servicio web de Azure. Esto dará lugar a una solución mejor administrada y más segura.

Para obtener más información:

* [Autenticación de servicio de Azure AD](https://docs.microsoft.com/azure/spatial-anchors/concepts/authentication?tabs=csharp#azure-ad-service-authentication)
* [Refuerce su seguridad con Azure](https://azure.microsoft.com/overview/security/)
* [Seguridad en la nube](https://azure.microsoft.com/product-categories/security/)
