---
title: Administración de certificados en un clúster de Service Fabric
description: Obtenga información sobre la administración de certificados en un clúster de Service Fabric protegido con certificados X.509.
ms.topic: conceptual
ms.date: 04/10/2020
ms.custom: sfrev
ms.openlocfilehash: aba681157d71f94914462b8d9fc13b90d4d6b153
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88653671"
---
# <a name="certificate-management-in-service-fabric-clusters"></a>Administración de certificados en clústeres de Service Fabric

En este artículo se tratan los aspectos de administración de los certificados que se usan para proteger la comunicación en los clústeres de Azure Service Fabric, y complementa la introducción a la [seguridad de clústeres de Service Fabric](service-fabric-cluster-security.md), así como la explicación sobre la [autenticación basada en certificados X.509 en Service Fabric](cluster-security-certificates.md). Damos por supuesto que el lector está familiarizado con los conceptos fundamentales de seguridad y también con los controles que Service Fabric expone para configurar la seguridad de un clúster.  

Aspectos que se tratan en este título:

* ¿Qué es exactamente la "administración de certificados"?
* Roles y entidades involucrados en la administración de certificados
* Recorrido de un certificado
* Profundización con un ejemplo
* Solución de problemas y preguntas frecuentes

Pero antes, una declinación de responsabilidades: en este artículo se intenta reunir el lado teórico con ejemplos prácticos, que requieren aspectos específicos de servicios, tecnologías, etc. Puesto que una parte considerable de la audiencia es interna de Microsoft, haremos referencia a servicios, tecnologías y productos específicos de Microsoft Azure. No dude en hacer preguntas en la sección Comentarios para obtener aclaraciones o instrucciones, cuando los detalles específicos de Microsoft no se apliquen a su caso.

## <a name="defining-certificate-management"></a>Definición de la administración de certificados
Como hemos visto en el [artículo complementario](cluster-security-certificates.md), un certificado es un objeto criptográfico que básicamente enlaza un par de claves asimétricas con atributos que describen la entidad que representa. Sin embargo, también es un objeto "perecedero", con una duración finita, y es susceptible de ponerle en peligro: una revelación accidental o un ataque pueden hacer que un certificado sea inútil desde el punto de vista de la seguridad. Esto implica la necesidad de cambiar los certificados, ya sea de forma rutinaria o en respuesta a un incidente de seguridad. Otro aspecto de la administración (y es un tema completo por sí mismo) es la protección de las claves privadas de los certificados, o de secretos que protegen las identidades de las entidades involucradas en la obtención y el aprovisionamiento de certificados. Por "administración de certificados" nos referimos a los procesos y procedimientos usados para obtener certificados y a transportarlos de forma segura (y protegida) a donde sean necesarios. Algunas de las operaciones de administración, como la inscripción, la configuración de directivas y los controles de autorización, están fuera del ámbito de este artículo. Aunque otras, como el aprovisionamiento, la renovación, la regeneración de claves o la revocación, solo se relacionan de manera incidental con Service Fabric; no obstante, las abordaremos aquí hasta cierto punto, ya que comprender estas operaciones puede ayudarle a proteger correctamente nuestro clúster. 

El objetivo es automatizar la administración de certificados tanto como sea posible para garantizar la disponibilidad ininterrumpida del clúster y ofrecer garantías de seguridad, dado que en el proceso no interviene el usuario. Este objetivo se alcanza actualmente en los clústeres de Azure Service Fabric; en el resto del artículo se deconstruirá primero la administración de certificados y, más adelante, se centrará en habilitar la sustitución automática.

En concreto, los temas abordados son los siguientes:
  - suposiciones relacionadas con la separación de atribuciones entre el propietario y la plataforma, en el contexto de la administración de certificados
  - la canalización larga de certificados desde la emisión hasta el consumo
  - rotación de certificados: por qué, cómo y cuándo
  - ¿qué podría salir mal?

Quedan fuera del ámbito de este artículo aspectos como la protección y administración de nombres de dominio, la inscripción en certificados o la configuración de controles de autorización para aplicar la emisión de certificados. Consulte con la entidad de registro (RA) de su servicio de infraestructura de clave pública (PKI) favorito. Los consumidores internos de Microsoft deben ponerse en contacto con Azure Security.

## <a name="roles-and-entities-involved-in-certificate-management"></a>Roles y entidades involucrados en la administración de certificados
El enfoque de seguridad en un clúster de Service Fabric es un caso de "el propietario del clúster lo declara, el runtime de Service Fabric lo aplica". Por esto, nos referimos a que la mayoría de los certificados, claves u otras credenciales de identidades que participan en el funcionamiento de un clúster proceden del propio servicio; el propietario del clúster los declara a todos ellos. Además, el propietario del clúster también es responsable de aprovisionar los certificados en el clúster, renovarlos según sea necesario y garantizar su seguridad en todo momento. Más concretamente, el propietario del clúster debe asegurarse de lo siguiente:
  - Los certificados declarados en la sección NodeType del manifiesto de clúster se pueden encontrar en cada nodo de ese tipo, de acuerdo con las [reglas de presentación](cluster-security-certificates.md#presentation-rules).
  - Los certificados declarados anteriormente se instalan con sus claves privadas correspondientes.
  - Los certificados declarados en las reglas de presentación deben pasar las [reglas de validación](cluster-security-certificates.md#validation-rules) 

Por su parte, Service Fabric asume las responsabilidades siguientes:
  - Ubica o busca certificados que coincidan con las declaraciones de la definición del clúster  
  - Concede acceso a las claves privadas correspondientes a las entidades controladas por Service Fabric "según sea necesario".
  - Valida los certificados en estricto cumplimiento de los procedimientos recomendados de seguridad establecidos y la definición del clúster.
  - Genera alertas sobre la expiración inminente de los certificados o sobre los errores para cumplir los pasos básicos de la validación de certificados.
  - Valida (hasta cierto punto) que la configuración subyacente de los hosts cumpla los aspectos relacionados con el certificado de la definición del clúster. 

De esto se desprende que la carga de la administración de certificados (como operaciones activas) recae exclusivamente en el propietario del clúster. En las secciones siguientes veremos más detalladamente cada una de las operaciones de administración, con los mecanismos disponibles y su impacto en el clúster.

## <a name="the-journey-of-a-certificate"></a>Recorrido de un certificado
Vamos a revisar rápidamente la progresión de un certificado desde su emisión hasta su consumo en el contexto de un clúster de Service Fabric:

  1. Un propietario de dominio registra en la entidad de registro (RA) de una PKI un dominio o firmante que quisiera asociar con certificados subsiguientes; los certificados, a su vez, constituirán las pruebas de propiedad de ese dominio o firmante.
  2. El propietario del dominio también designa en la RA las identidades de los solicitantes autorizados: entidades que tienen derecho a solicitar la inscripción de certificados en el dominio o firmante especificados; en Microsoft Azure, el proveedor de identidades predeterminado es Azure Active Directory, y los solicitantes autorizados se designan mediante su identidad de AAD correspondiente (o a través de grupos de seguridad).
  3. Un solicitante autorizado se inscribe en un certificado a través de un servicio de administración de secretos; en Microsoft Azure, el servicio elegido es Azure Key Vault (AKV), que almacena de forma segura secretos o certificados por parte de las entidades autorizadas y permite su recuperación. AKV también renueva el certificado o regenera sus claves según lo configurado en la directiva de certificados asociada. (AKV usa AAD como proveedor de identidades).
  4. Un recuperador autorizado, al que haremos referencia como "agente de aprovisionamiento", recupera el certificado, incluida su clave privada, del almacén y lo instala en las máquinas que hospedan el clúster.
  5. El servicio Service Fabric (que se ejecuta con privilegios elevados en cada nodo) concede a las entidades de Service Fabric permitidas acceso al certificado; los grupos locales designan estas entidades, que se dividen entre ServiceFabricAdministrators y ServiceFabricAllowedUsers.
  6. El runtime de Service Fabric accede al certificado y lo usa para establecer la federación, o para autenticarse en las solicitudes entrantes de clientes autorizados.
  7. El agente de aprovisionamiento supervisa el certificado del almacén y desencadena el flujo de aprovisionamiento al detectar la renovación; posteriormente, el propietario del clúster actualiza la definición del clúster, si es necesario, para indicar la intención de sustituir el certificado.
  8. El agente de aprovisionamiento o el propietario del clúster también son responsables de limpiar o eliminar los certificados sin usar.
  
Para nuestros fines, los primeros dos pasos de la secuencia anterior no están demasiado relacionados; su única conexión es que el nombre común del firmante del certificado es el nombre DNS declarado en la definición del clúster.

Estos pasos se ilustran a continuación; tenga en cuenta las diferencias en el aprovisionamiento entre los certificados declarados por la huella digital y el nombre común, respectivamente.

*Fig. 1.* Flujo de emisión y aprovisionamiento de certificados declarados por huella digital.
![Aprovisionamiento de certificados declarados por huella digital][Image1]

*Fig. 2.* Flujo de emisión y aprovisionamiento de certificados declarados por nombre común del firmante.
![Aprovisionamiento de certificados declarados por nombre común del firmante][Image2]

### <a name="certificate-enrollment"></a>Inscripción de certificados
Este tema se trata con más detalle en la [documentación](../key-vault/certificates/create-certificate.md) de Key Vault; aquí se incluye un resumen para mantener la continuidad y como referencia más sencilla. Continuando con Azure como contexto y usando Azure Key Vault como servicio de administración de secretos, un solicitante de certificados autorizado debe tener al menos permisos de administración de certificados en el almacén, concedido por el propietario del almacén; a continuación, el solicitante se inscribiría en un certificado de la manera siguiente:
    - Crea una directiva de certificado en Azure Key Vault (AKV), que especifica el dominio o firmante del certificado, el emisor deseado, el tipo de clave y su longitud, el uso previsto de claves, etc.; para obtener más información, consulte [Certificados en Azure Key Vault](../key-vault/certificates/certificate-scenarios.md). 
    - Crea un certificado en el mismo almacén con la directiva especificada anteriormente; esto, a su vez, genera un par de claves como objetos del almacén, una solicitud de firma de certificado firmada con la clave privada, que luego se reenvía al emisor designado para su firma.
    - Una vez que el emisor (la entidad de certificación) responde con el certificado firmado, el resultado se combina en el almacén y el certificado está disponible para las operaciones siguientes:
      - en {UriDelAlmacén}/certificates/{nombre}: el certificado, incluida la clave pública y los metadatos
      - en {UriDelAlmacén}/keys/{nombre}: la clave privada del certificado, disponible para las operaciones criptográficas (encapsular/desencapsular, firmar/verificar)
      - en{vaultUri}/secrets/{name}: el certificado, incluida su clave privada, disponible para su descarga como un archivo PFX o PEM sin proteger  
    Recuerde que un certificado de almacén es, en realidad, una línea cronológica de instancias de certificado que comparten una directiva. Las versiones del certificado se crearán según los atributos de duración y renovación de la directiva. Se recomienda encarecidamente que los certificados del almacén no compartan firmantes ni dominios/nombres DNS; puede ser perjudicial en un clúster aprovisionar instancias de certificado de distintos certificados del almacén, con firmantes idénticos, pero con otros atributos sustancialmente diferentes, como el emisor, los usos de la clave, etc.

En este momento, existe un certificado en el almacén, listo para su consumo. Ahora veamos:

### <a name="certificate-provisioning"></a>Aprovisionamiento del certificado
Mencionamos un "agente de aprovisionamiento", que es una entidad que recupera el certificado, incluida su clave privada, del almacén y lo instala en cada uno de los hosts del clúster. (Recuerde que Service Fabric no aprovisiona los certificados). En nuestro contexto, el clúster se hospedará en una colección de VM de Azure o en conjuntos de escalado de máquinas virtuales. En Azure, el aprovisionamiento de un certificado desde un almacén hacia una VM/VMSS se puede lograr con los siguientes mecanismos: suponiendo que, como se indica anteriormente, el propietario del almacén ha concedido previamente los permisos "get" al agente de aprovisionamiento en el almacén: 
  - Ad hoc: un operador recupera el certificado del almacén (como PFX/PKCS12 o PEM) y lo instala en cada nodo.
  - Como "secreto" del conjunto de escalado de máquinas virtuales durante la implementación: el servicio Compute recupera, usando la identidad de su primera entidad en nombre del operador, el certificado de un almacén habilitado para la implementación de plantillas y lo instala en cada nodo del conjunto de escalado de máquinas virtuales ([de este modo](../virtual-machine-scale-sets/virtual-machine-scale-sets-faq.md#certificates)); tenga en cuenta que esto permite el aprovisionamiento solo de secretos con versión.
  - Mediante la [extensión de VM de Key Vault](../virtual-machines/extensions/key-vault-windows.md); esto permite el aprovisionamiento de certificados con declaraciones sin versión, con una actualización periódica de los certificados observados. En este caso, se espera que la VM/VMSS tenga una [identidad administrada](../virtual-machines/security-policy.md#managed-identities-for-azure-resources), una identidad a la que se le ha concedido acceso a los almacenes que contienen los certificados observados.

No se recomienda el mecanismo ad hoc por varios motivos, desde la seguridad hasta la disponibilidad, por lo que no se tratará más en adelante; para obtener más información, consulte [certificados en conjuntos de escalado de máquinas virtuales](../virtual-machine-scale-sets/virtual-machine-scale-sets-faq.md#certificates).

El aprovisionamiento basado en VMSS/Compute presenta las ventajas de seguridad y disponibilidad, pero también presenta restricciones. Por su diseño, exige la declaración de certificados como secretos con versión, lo que hace que sea adecuado solo para los clústeres protegidos con certificados declarados por huella digital. Por el contrario, el aprovisionamiento basado en la extensión de VM de Key Vault siempre instalará la versión más reciente de cada certificado observado, lo que hace que sea adecuado solo para los clústeres protegidos con certificados declarados por nombre común del firmante. Para enfatizar, no use un mecanismo de aprovisionamiento de actualización automática (como la extensión KVVM) para los certificados declarados por instancia (es decir, por huella digital), el riesgo de perder disponibilidad es considerable.

Pueden existir otros mecanismos de aprovisionamiento; actualmente, los clústeres de Azure Service Fabric aceptan los anteriores.

### <a name="certificate-consumption-and-monitoring"></a>Consumo y supervisión de certificados
Como se mencionó anteriormente, el runtime de Service Fabric es responsable de ubicar y usar los certificados declarados en la definición del clúster. En el artículo sobre la [autenticación basada en certificados](cluster-security-certificates.md) se explica con detalle de qué manera Service Fabric implementa las reglas de presentación y validación, respectivamente, y no se volverá a abordar aquí. Vamos a analizar el acceso y la concesión de permisos, así como la supervisión.

Recuerde que los certificados en Service Fabric se usan para una gran variedad de propósitos, desde la autenticación mutua en el nivel de federación hasta la autenticación TLS para los puntos de conexión de administración; esto exige que varios componentes o servicios del sistema tengan acceso a la clave privada del certificado. El runtime de Service Fabric examina periódicamente el almacén de certificados en busca de coincidencias para cada una de las reglas de presentación conocidas; para cada uno de los certificados coincidentes, se busca la clave privada correspondiente, y su lista de control de acceso discrecional se actualiza para incluir los permisos (normalmente Lectura y Ejecución) concedidos a la identidad correspondiente que los requiera. (Este proceso se puede llamar informalmente "aplicar ACL"). El proceso se ejecuta con una cadencia de 1 minuto y también abarca los certificados de "aplicación", como los que se usan para cifrar la configuración o como certificados de puntos de conexión. La aplicación de ACL sigue las reglas de presentación, por lo que es importante tener en cuenta que no se podrá acceder a los certificados que se declaran por huella digital y que se actualizan de forma automática sin la actualización de la configuración de clúster subsiguiente.    

### <a name="certificate-rotation"></a>Rotación de certificados
Como nota al margen: [RFC 3647](https://tools.ietf.org/html/rfc3647) de IETF define formalmente la [renovación](https://tools.ietf.org/html/rfc3647#section-4.4.6) (renewal) como la emisión de un certificado con los mismos atributos que el certificado que se está reemplazando: el emisor, la clave pública del firmante y la información se conservan; y la [regeneración de una clave](https://tools.ietf.org/html/rfc3647#section-4.4.7) (re-key) como la emisión de un certificado con un nuevo par de claves, sin restricciones sobre si el emisor puede cambiar o no. Dado que la distinción puede ser importante (considere el caso de los certificados declarados por nombre común del firmante con asignación del emisor), optaremos por el término neutro "rotación" para abarcar ambos escenarios. (Tenga en cuenta que, cuando se usa de manera informal, la "renovación" hace referencia, de hecho, a regenerar la clave). 

Anteriormente hemos visto que Azure Key Vault admite la rotación automática de certificados: la directiva de certificados asociada define el momento dado, ya sea en días antes de la expiración o en porcentaje de la duración total, cuando el certificado rota en el almacén. Debe invocarse al agente de aprovisionamiento después de dicho momento y antes de la expiración del certificado anterior, para distribuir este nuevo certificado a todos los nodos del clúster. Para ayudar en esto, Service Fabric generará advertencias de estado cuando la fecha de expiración de un certificado (y que está actualmente en uso en el clúster) se produzca antes de un intervalo predeterminado. Un agente de aprovisionamiento automático (es decir, la extensión KeyVault VM), configurado para observar el certificado del almacén, sondeará periódicamente el almacén, detectará la rotación y recuperará e instalará el nuevo certificado. El aprovisionamiento realizado a través de la característica "secretos" de la VM/VMSS requerirá que un operador autorizado actualice la VM/VMSS con el URI de KeyVault con versión correspondiente al nuevo certificado.

En cualquier caso, el certificado rotado ahora está aprovisionado en todos los nodos, ya hemos descrito el mecanismo que emplea Service Fabric para detectar rotaciones; vamos a examinar lo que ocurre después: suponiendo una rotación aplicada al certificado del clúster declarada por nombre común del firmante (todo vigente en el momento de redactar este documento y una versión del runtime de Service Fabric 7.1.409):
  - En el caso de nuevas conexiones dentro del clúster, así como con el clúster, el runtime de Service Fabric buscará y seleccionará el certificado coincidente con la fecha de expiración más lejana (la propiedad "NotAfter" del certificado, a menudo abreviada como "na").
  - Las conexiones existentes se mantendrán activas/permitidas hasta la expiración natural o, de lo contrario, hasta que finalicen; un controlador interno recibirá una notificación de que existe una nueva coincidencia.

Esto se traduce en las siguientes observaciones importantes:
  - El certificado de renovación puede omitirse si su fecha de expiración es anterior a la del certificado actualmente en uso.
  - La disponibilidad del clúster, o de las aplicaciones hospedadas, tiene prioridad sobre la directiva para rotar el certificado; el clúster convergerá en el nuevo certificado en algún momento, pero sin garantías de plazos. Así pues:
  - Es posible que no resulte evidente de inmediato para un observador que el certificado rotado ha reemplazado por completo a su predecesor; la única manera de asegurarse de esto (en el caso de los certificados del clúster) es reiniciar las máquinas host. Tenga en cuenta que no basta con reiniciar los nodos de Service Fabric, ya que los componentes de modo kernel que forman las conexiones de concesión en un clúster no se verán afectados. Tenga en cuenta también que el reinicio de la VM/VMSS puede provocar una pérdida temporal de disponibilidad. (En el caso de los certificados de aplicación, basta con reiniciar las instancias de aplicación correspondientes).
  - La introducción de un certificado con una clave regenerada, que no cumple las reglas de validación, puede interrumpir verdaderamente el clúster. El ejemplo más común es el caso de un emisor inesperado: los certificados del clúster se declaran por nombre común del firmante con asignación del emisor, pero un emisor nuevo o no declarado ha emitido el certificado rotado.     

### <a name="certificate-cleanup"></a>Limpieza de certificados
En este momento, no hay ninguna disposición en Azure para la eliminación explícita de certificados. A menudo, determinar si un cierto certificado determinado está en uso o no en un momento dado es una tarea que no es trivial. Esto es más difícil para los certificados de aplicación que para los certificados de clúster. Service Fabric, al no ser el agente de aprovisionamiento, no eliminará un certificado declarado por el usuario en ningún caso. Para los mecanismos de aprovisionamiento estándar:
  - Los certificados declarados como secretos de VM/VMSS se aprovisionarán siempre que se haga referencia a ellos en la definición de la VM/VMSS y se puedan recuperar desde el almacén (si se elimina un secreto o certificado de un almacén, se producirá un error en las implementaciones posteriores de la VM/VMSS, que hacen referencia a esa versión del secreto).
  - Las versiones anteriores de certificados aprovisionados a través de la extensión KeyVault VM pueden estar presentes o no en un nodo de VM/VMSS. El agente solo recupera e instala la versión actual y no quita ningún certificado. Restablecer la imagen inicial de un nodo (lo que habitualmente suele producirse cada mes) restablecerá el almacén de certificados al contenido de la imagen del sistema operativo, por lo que las versiones anteriores se quitarán implícitamente. Sin embargo, tenga en cuenta que, al escalar verticalmente un conjunto de escalado de máquinas virtuales, solo se instalará la versión actual de los certificados observados; no suponga la homogeneidad de los nodos con respecto a los certificados instalados.   

## <a name="simplifying-management---an-autorollover-example"></a>Simplificación de la administración: ejemplo de sustitución automática
Hemos descrito mecanismos, restricciones, hemos esbozados reglas y definiciones intrincadas, y hemos hecho predicciones terribles acerca de interrupciones. Es posible que sea momento de mostrar cómo configurar la administración automática de certificados para evitar todas estas inquietudes. Lo hacemos en el contexto de un clúster de Azure Service Fabric que se ejecuta en un conjunto de escalado de máquinas virtuales PaaSv2, usando Azure Key Vault para la administración de secretos y aprovechando las identidades administradas, como se indica a continuación:
  - La validación de certificados se cambia de la asignación de huella digital a la asignación de firmante + emisor: todo certificado con un firmante dado de un emisor determinado tiene la misma confianza.
    - Los certificados se inscriben y se obtienen de un almacén de confianza (Key Vault) y se actualizan mediante un agente; en este caso, la extensión KeyVault VM.
    - El aprovisionamiento de certificados se cambia de uno basado en el tiempo de implementación y la versión (como lo hace ComputeRP) a uno posterior a la implementación y usando los URI de KeyVault sin versión.
    - El acceso a KeyVault se concede a través de identidades administradas asignadas por el usuario; la identidad de UA se crea y se asigna al conjunto de escalado de máquinas virtuales durante la implementación.
    - Después de la implementación, el agente (la extensión de VM de KV) sondeará y actualizará los certificados observados en cada nodo del conjunto de escalado de máquinas virtuales; por tanto, la rotación de certificados está completamente automatizada, ya que Service Fabric seleccionará automáticamente el certificado con mayor validez.

La secuencia es admite script o está automatizada en su totalidad, y permite, sin intervención del usuario, una implementación inicial de un clúster configurado para la sustitución automática de certificados. A continuación se proporcionan los pasos detallados. Usaremos una combinación de cmdlets de PowerShell y fragmentos de plantillas JSON. La misma funcionalidad puede lograrse con todos los medios admitidos para interactuar con Azure.

[!NOTE] En este ejemplo se supone que ya existe un certificado en el almacén; la inscripción y la renovación de un certificado administrado por KeyVault requieren pasos manuales previos, como se describió anteriormente en este artículo. Para entornos de producción, use certificados administrados por KeyVault; a continuación se incluye un script de ejemplo específico para una PKI interna de Microsoft.
La sustitución automática de certificados solo tiene sentido para los certificados emitidos por una entidad de certificación; mediante el uso de certificados autofirmados, incluidos los generados al implementar un clúster de Service Fabric en Azure Portal, es absurda, aunque posible para implementaciones locales u hospedadas por desarrolladores, al declarar la huella digital del emisor para que sea la misma que la del certificado de hoja.

### <a name="starting-point"></a>Punto de partida
Por motivos de brevedad, supondremos el siguiente estado de partida:
  - El clúster de Service Fabric existe y está protegido con un certificado emitido por una entidad de certificación, declarado por huella digital.
  - El certificado se almacena en un almacén y se aprovisiona como secreto de un conjunto de escalado de máquinas virtuales.
  - El mismo certificado se utilizará para convertir el clúster en declaraciones de certificado basadas en nombres comunes, de modo que el firmante y el emisor pueden validarlo. De no ser así, obtenga el certificado emitido por la entidad de certificación destinado a tales fines y agréguelo a la definición del clúster mediante la huella digital, tal y como se explica [aquí](service-fabric-cluster-security-update-certs-azure.md).

A continuación se muestra un fragmento de JSON de una plantilla que corresponde a este tipo de estado; tenga en cuenta que se omiten muchos valores obligatorios, y solo se muestran los aspectos relacionados con el certificado:
```json
  "resources": [
    {   ## VMSS definition
      "apiVersion": "[variables('vmssApiVersion')]",
      "type": "Microsoft.Compute/virtualMachineScaleSets",
      "name": "[variables('vmNodeTypeName')]",
      "location": "[variables('computeLocation')]",
      "properties": {
        "virtualMachineProfile": {
          "extensionProfile": {
            "extensions": [
            {
                "name": "[concat('ServiceFabricNodeVmExt','_vmNodeTypeName')]",
                "properties": {
                  "type": "ServiceFabricNode",
                  "autoUpgradeMinorVersion": true,
                  "publisher": "Microsoft.Azure.ServiceFabric",
                  "settings": {
                    "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                    "nodeTypeRef": "[variables('vmNodeTypeName')]",
                    "dataPath": "D:\\SvcFab",
                    "durabilityLevel": "Bronze",
                    "certificate": {
                        "thumbprint": "[parameters('primaryClusterCertificateTP')]",
                        "x509StoreName": "[parameters('certificateStoreValue')]"
                    }
                  },
                  "typeHandlerVersion": "1.1"
                }
            },}},
          "osProfile": {
            "adminPassword": "[parameters('adminPassword')]",
            "adminUsername": "[parameters('adminUsername')]",
            "secrets": [
            {
                "sourceVault": {
                    "id": "[resourceId('Microsoft.KeyVault/vaults', parameters('keyVaultName'))]"
                },
                "vaultCertificates": [
                {
                    "certificateStore": "[parameters('certificateStoreValue')]",
                    "certificateUrl": "[parameters('clusterCertificateUrlValue')]"
                },
            ]}]
        },
    },
    {   ## cluster definition
        "apiVersion": "[variables('sfrpApiVersion')]",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        "certificate": {
            "thumbprint": "[parameters('primaryClusterCertificateTP')]",
            "x509StoreName": "[parameters('certificateStoreValue')]"
        },
    }
  ]
```   

En esencia, lo anterior indica que el certificado con la huella digital ```json [parameters('primaryClusterCertificateTP')] ``` y que se encuentra en el URI de KeyVault ```json [parameters('clusterCertificateUrlValue')] ``` se declara como el certificado único del clúster, por huella digital. A continuación, configuraremos los recursos adicionales necesarios para asegurar la sustitución automática del certificado.

### <a name="setting-up-prerequisite-resources"></a>Configuración de los recursos previos necesarios
Como se mencionó anteriormente, el servicio Proveedor de recursos Microsoft.Compute recupera del almacén un certificado aprovisionado como secreto del conjunto de escalado de máquinas virtuales, usando su identidad propia y en nombre del operador de implementación. Para la sustitución automática, que cambiará, cambiaremos para usar una identidad administrada, asignada al conjunto de escalado de máquinas virtuales, y a la que se conceden permisos a los secretos del almacén.

Todos los extractos subsiguientes deben implementarse concomitantemente: se enumeran de forma individual para su análisis y explicaciones detallados.

En primer lugar, defina una identidad asignada por el usuario (los valores predeterminados se incluyen como ejemplos). Consulte la [documentación oficial](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-arm.md#create-a-user-assigned-managed-identity) para obtener información actualizada:
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "userAssignedIdentityName": {
      "type": "string",
      "defaultValue": "sftstuaicus",
      "metadata": {
        "description": "User-assigned managed identity name"
      }
    },
  },
  "variables": {
      "vmssApiVersion": "2018-06-01",
      "sfrpApiVersion": "2018-02-01",
      "miApiVersion": "2018-11-30",
      "kvApiVersion": "2018-02-14",
      "userAssignedIdentityResourceId": "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('userAssignedIdentityName'))]"
  },    
  "resources": [
    {
      "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
      "name": "[parameters('userAssignedIdentityName')]",
      "apiVersion": "[variables('miApiVersion')]",
      "location": "[resourceGroup().location]"
    },
  ]}
```

A continuación, conceda a esta identidad acceso a los secretos del almacén. Consulte la [documentación oficial](/rest/api/keyvault/vaults/updateaccesspolicy) para obtener información actual:
```json
  "resources":
  [{
      "type": "Microsoft.KeyVault/vaults/accessPolicies",
      "name": "[concat(parameters('keyVaultName'), '/add')]",
      "apiVersion": "[variables('kvApiVersion')]",
      "properties": {
        "accessPolicies": [
          {
            "tenantId": "[reference(variables('userAssignedIdentityResourceId'), variables('miApiVersion')).tenantId]",
            "objectId": "[reference(variables('userAssignedIdentityResourceId'), variables('miApiVersion')).principalId]",
            "dependsOn": [
              "[variables('userAssignedIdentityResourceId')]"
            ],
            "permissions": {
              "secrets": [
                "get",
                "list"
              ]}}]}}]
```

En el siguiente paso, haremos lo siguiente:
  - Asignar al conjunto de escalado de máquinas virtuales la identidad asignada por el usuario.
  - Declarar la dependencia del conjunto de escalado de máquinas virtuales en la creación de la identidad administrada y en el resultado de concederle acceso al almacén.
  - Declarar la extensión KeyVault VM, y exigir que recupere los certificados observados en el inicio ([documentación oficial](../virtual-machines/extensions/key-vault-windows.md)).
  - Actualizar la definición de la extensión de VM de Service Fabric para que dependa de la extensión KVVM, y para convertir el certificado del clúster al nombre común (vamos a realizar estos cambios en un solo paso, ya que se encuentran bajo el ámbito del mismo recurso).

```json
  "parameters": {
    "kvvmextPollingInterval": {
      "type": "string",
      "defaultValue": "3600",
      "metadata": {
        "description": "kv vm extension polling interval in seconds"
      }
    },
    "kvvmextLocalStoreName": {
      "type": "string",
      "defaultValue": "MY",
      "metadata": {
        "description": "kv vm extension local store name"
      }
    },
    "kvvmextLocalStoreLocation": {
      "type": "string",
      "defaultValue": "LocalMachine",
      "metadata": {
        "description": "kv vm extension local store location"
      }
    },
    "kvvmextObservedCertificates": {
      "type": "array",
      "defaultValue": [
                "https://sftestcus.vault.azure.net/secrets/sftstcncluster",
                "https://sftestcus.vault.azure.net/secrets/sftstcnserver"
            ],
      "metadata": {
        "description": "kv vm extension observed certificates versionless uri"
      }
    },
    "certificateCommonName": {
      "type": "string",
      "defaultValue": "cus.cluster.sftstcn.system.servicefabric.azure-int",
      "metadata": {
        "description": "Certificate Common name"
      }
    },
  },
  "resources": [
  {
    "apiVersion": "[variables('vmssApiVersion')]",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    "name": "[variables('vmNodeTypeName')]",
    "location": "[variables('computeLocation')]",
    "dependsOn": [
      "[variables('userAssignedIdentityResourceId')]",
      "[concat('Microsoft.KeyVault/vaults/', concat(parameters('keyVaultName'), '/accessPolicies/add'))]"
    ],
    "identity": {
      "type": "UserAssigned",
      "userAssignedIdentities": {
        "[variables('userAssignedIdentityResourceId')]": {}
      }
    },
    "virtualMachineProfile": {
      "extensionProfile": {
        "extensions": [
        {
          "name": "KVVMExtension",
          "properties": {
            "publisher": "Microsoft.Azure.KeyVault",
            "type": "KeyVaultForWindows",
            "typeHandlerVersion": "1.0",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "secretsManagementSettings": {
                    "pollingIntervalInS": "[parameters('kvvmextPollingInterval')]",
                    "linkOnRenewal": false,
                    "observedCertificates": "[parameters('kvvmextObservedCertificates')]",
                    "requireInitialSync": true
                }
            }
          }
        },
        {
        "name": "[concat('ServiceFabricNodeVmExt','_vmNodeTypeName')]",
        "properties": {
          "type": "ServiceFabricNode",
          "provisionAfterExtensions" : [ "KVVMExtension" ],
          "publisher": "Microsoft.Azure.ServiceFabric",
          "settings": {
            "certificate": {
                "commonNames": [
                    "[parameters('certificateCommonName')]"
                ],
                "x509StoreName": "[parameters('certificateStoreValue')]"
            }
            },
            "typeHandlerVersion": "1.0"
          }
        },
  ] } ## extension profile
  },  ## VM profile
  "osProfile": {
    "adminPassword": "[parameters('adminPassword')]",
    "adminUsername": "[parameters('adminUsername')]",
  } 
  }
  ]
```
Tenga en cuenta que, aunque no se indicó explícitamente anteriormente, se quitó la dirección URL del certificado del almacén de la sección "OsProfile" del conjunto de escalado de máquinas virtuales.
El último paso es actualizar la definición del clúster para cambiar la declaración del certificado de huella digital a nombre común; aquí también se asignan las huellas digitales del emisor:

```json
  "parameters": {
    "certificateCommonName": {
      "type": "string",
      "defaultValue": "cus.cluster.sftstcn.system.servicefabric.azure-int",
      "metadata": {
        "description": "Certificate Common name"
      }
    },
    "certificateIssuerThumbprint": {
      "type": "string",
      "defaultValue": "1b45ec255e0668375043ed5fe78a09ff1655844d,d7fe717b5ff3593764f4d90654d86e8362ec26c8,3ac7c3cac8de0dd392c02789c8be97474f456960,96ea05926e2e42cc207e358668be2c316857fb5e",
      "metadata": {
        "description": "Certificate issuer thumbprints separated by comma"
      }
    },
  },
  "resources": [
    {
      "apiVersion": "[variables('sfrpApiVersion')]",
      "type": "Microsoft.ServiceFabric/clusters",
      "name": "[parameters('clusterName')]",
      "location": "[parameters('clusterLocation')]",
      "properties": {
        "certificateCommonNames": {
          "commonNames": [{
              "certificateCommonName": "[parameters('certificateCommonName')]",
              "certificateIssuerThumbprint": "[parameters('certificateIssuerThumbprint')]"
          }],
          "x509StoreName": "[parameters('certificateStoreValue')]"
        },
  ]
```

En este momento, puede ejecutar las actualizaciones mencionadas anteriormente en una sola implementación; por su parte, el servicio Proveedor de recursos de Service Fabric dividirá la actualización del clúster en varios pasos, como se describe en el segmento sobre [Conversión de un clúster de declaraciones de certificado basadas en huellas digitales a declaraciones de certificado basadas en nombres comunes](cluster-security-certificates.md#converting-a-cluster-from-thumbprint--to-common-name-based-certificate-declarations).

### <a name="analysis-and-observations"></a>Análisis y observaciones
Esta es una sección general para explicar los pasos descritos anteriormente, así como llamar la atención sobre aspectos importantes.

#### <a name="certificate-provisioning-explained"></a>Explicación sobre el aprovisionamiento de certificados
La extensión KVVM, como agente de aprovisionamiento, se ejecuta de forma continua a una frecuencia predeterminada. En caso de no recuperar un certificado observado, continuaría con el siguiente en la línea y, a continuación, hibernaría hasta el siguiente ciclo. La extensión SFVM, como agente de arranque del clúster, requerirá los certificados declarados antes de que se pueda formar el clúster. Esto, a su vez, significa que la extensión SFVM solo se puede ejecutar después de la recuperación correcta de los certificados del clúster, que se indica aquí mediante la cláusula ```json "provisionAfterExtensions" : [ "KVVMExtension" ]"``` y mediante el valor ```json "requireInitialSync": true``` de la extensión KeyVaultVM. Esto indica a la extensión KVVM que en la primera ejecución (después de la implementación o de un reinicio) debe recorrer los certificados observados hasta que todos se hayan descargado correctamente. Si se establece este parámetro en false, junto con un error al recuperar los certificados del clúster, se producirá un error en la implementación del clúster. Por el contrario, si se requiere una sincronización inicial con una lista incorrecta o no válida de certificados observados, se producirá un error de la extensión KVVM y, por lo tanto, nuevamente un error al implementar el clúster.  

#### <a name="certificate-linking-explained"></a>Explicación de la vinculación de certificados
Es posible que haya observado la marca "linkOnRenewal" de la extensión KVVM y el hecho de que se ha establecido en false. Aquí abordaremos en profundidad el comportamiento que controla esta marca y sus consecuencias en el funcionamiento de un clúster. Tenga en cuenta que este comportamiento es específico de Windows.

Según su [definición](../virtual-machines/extensions/key-vault-windows.md#extension-schema):
```json
"linkOnRenewal": <Only Windows. This feature enables auto-rotation of SSL certificates, without necessitating a re-deployment or binding.  e.g.: false>,
```

Los certificados usados para establecer una conexión TLS suelen [adquirirse como controlador](/windows/win32/api/sspi/nf-sspi-acquirecredentialshandlea) a través del proveedor de compatibilidad para seguridad Schannel, es decir, el cliente no accede directamente a la clave privada del certificado mismo. Schannel admite el redireccionamiento (vinculación) de credenciales en forma de una extensión del certificado ([CERT_RENEWAL_PROP_ID](/windows/win32/api/wincrypt/nf-wincrypt-certsetcertificatecontextproperty#cert_renewal_prop_id)): si se establece esta propiedad, su valor representa la huella digital del certificado de "renovación" y, por tanto, el Schannel intentará cargar el certificado vinculado. De hecho, atravesará esta lista vinculada (y, esperemos, acíclica) hasta que acabe con el certificado "final", uno sin una marca de renovación. Esta característica, si se usa con prudencia, es una excelente mitigación de la pérdida de disponibilidad causada por certificados expirados (por ejemplo). En otros casos, puede ser la causa de interrupciones difíciles de diagnosticar y mitigar. Schannel ejecuta el recorrido de los certificados en sus propiedades de renovación de manera incondicional, independientemente del firmante, los emisores o cualquier otro atributo específico que participe en la validación del certificado resultante por parte del cliente. De hecho, es posible que el certificado resultante no tenga ninguna clave privada asociada o que la clave no se haya incluido en la ACL para su consumidor potencial. 
 
Si la vinculación está habilitada, la extensión KeyVault VM, después de recuperar un certificado observado del almacén, intentará encontrar los certificados existentes coincidentes para vincularlos a través de la propiedad de extensión de renovación. La coincidencia se basa (exclusivamente) en el nombre alternativo del firmante (SAN) y funciona como se muestra a continuación.
Suponga que existen dos certificados, como se indica a continuación: A. CN = “Alice's accessories”, SAN = {“alice.universalexports.com”}, renewal = ‘’ B: CN = “Bob's bits”, SAN = {“bob.universalexports.com”, “bob.universalexports.net”}, renewal = ‘’
 
Supongamos que la extensión KVVM recupera un certificado C: CN = “Mallory's malware”, SAN = {“alice.universalexports.com”, “bob.universalexports.com”, “mallory.universalexports.com”}
 
La lista de SAN de A está totalmente incluida en la de C y, por tanto, A.renewal = C.thumbprint; la lista SAN de B tiene una intersección común con la de C, pero no está totalmente incluida en ella, por lo que B.renewal permanece vacía.
 
Cualquier intento de invocar AcquireCredentialsHandle (Schannel) en este estado en el certificado A en realidad terminará enviando C a la parte remota. En el caso de Service Fabric, el [subsistema de transporte](service-fabric-architecture.md#transport-subsystem) de un clúster utiliza Schannel para la autenticación mutua, por lo que el comportamiento descrito anteriormente afecta directamente a la comunicación fundamental del clúster. Siguiendo con el ejemplo anterior, y suponiendo que A es el certificado del clúster, lo que sucede después depende de lo siguiente:
  - Si la clave privada de C no se incluye en la ACL para la cuenta en la que se ejecuta Fabric, veremos que hay errores para adquirir la clave privada (SEC_E_UNKNOWN_CREDENTIALS o algo similar).
  - Si la clave privada de C es accesible, veremos que los otros nodos devuelven errores de autorización (CertificateNotMatched, no autorizado, etc.). 
 
En todo caso, se produce un error en el transporte y el clúster puede dejar de funcionar; los síntomas varían. Para empeorar las cosas, la vinculación depende del orden de renovación, que viene determinado por el orden de la lista de certificados observados de la extensión KVVM, la programación de renovación del almacén o, incluso, los errores transitorios que modificarían el orden de recuperación.

Para mitigar estos incidentes, se recomienda lo siguiente:
  - No mezcle los SAN de distintos certificados del almacén; cada certificado del almacén debe servir para un propósito distinto, y su firmante y SAN deberían reflejar eso de manera específica.
  - Incluya el nombre común del firmante en la lista de SAN (como, literalmente, "CN=<subject common name>").  
  - Si no está seguro, deshabilite la vinculación al renovar los certificados aprovisionados con la extensión KVVM. 

#### <a name="why-use-a-user-assigned-managed-identity-what-are-the-implications-of-using-it"></a>¿Por qué usar una identidad administrada asignada por el usuario? ¿Cuáles son las consecuencias de usarla?
Como surgió de los fragmentos de código JSON anteriores, se requiere una secuencia específica de las operaciones o actualizaciones para garantizar una correcta conversión y mantener la disponibilidad del clúster. En concreto, el recurso del conjunto de escalado de máquinas virtuales declara y usa su identidad para recuperar los secretos en una única actualización (desde la perspectiva del usuario). La extensión Service Fabric VM (que arranca el clúster) depende de la finalización de la extensión KeyVault VM, que depende de la recuperación correcta de los certificados observados. La extensión KVVM usa la identidad del conjunto de escalado de máquinas virtuales para acceder al almacén, lo que significa que la directiva de acceso en el almacén debe haberse actualizado antes de la implementación del conjunto de escalado de máquinas virtuales. 

Para disponer la creación de una identidad administrada o asignarla a otro recurso, el operador de implementación debe tener el rol necesario (ManagedIdentityOperator) en la suscripción o el grupo de recursos, además de los roles necesarios para administrar los demás recursos a los que se hace referencia en la plantilla. 

Desde el punto de vista de la seguridad, recuerde que (el conjunto de escalado de) la máquina virtual se considera un límite de seguridad con respecto a su identidad de Azure. Esto significa que cualquier aplicación hospedada en la VM podría, en principio, obtener un token de acceso que represente a la VM: los tokens de acceso de la identidad administrada se obtienen del punto de conexión de IMDS sin autenticar. Si considera que la VM es un entorno compartido o de varios inquilinos, es posible que este método de recuperación de certificados de clúster no sea el indicado. No obstante, es el único mecanismo de aprovisionamiento adecuado para la sustitución automática de certificados.

## <a name="troubleshooting-and-frequently-asked-questions"></a>Solución de problemas y preguntas frecuentes

*P.* : ¿Cómo inscribirse mediante programación en un certificado administrado por KeyVault?
*R.* : Averigüe el nombre del emisor en la documentación de KeyVault y reemplácelo en el script siguiente.  
```PowerShell
  $issuerName=<depends on your PKI of choice>
    $clusterVault="sftestcus"
    $clusterCertVaultName="sftstcncluster"
    $clusterCertCN="cus.cluster.sftstcn.system.servicefabric.azure-int"
    Set-AzKeyVaultCertificateIssuer -VaultName $clusterVault -Name $issuerName -IssuerProvider $issuerName
    $distinguishedName="CN=" + $clusterCertCN
    $policy = New-AzKeyVaultCertificatePolicy `
        -IssuerName $issuerName `
        -SubjectName $distinguishedName `
        -SecretContentType 'application/x-pkcs12' `
        -Ekus "1.3.6.1.5.5.7.3.1", "1.3.6.1.5.5.7.3.2" `
        -ValidityInMonths 4 `
        -KeyType 'RSA' `
        -RenewAtPercentageLifetime 75        
    Add-AzKeyVaultCertificate -VaultName $clusterVault -Name $clusterCertVaultName -CertificatePolicy $policy
    
    # poll the result of the enrollment
    Get-AzKeyVaultCertificateOperation -VaultName $clusterVault -Name $clusterCertVaultName 
```

*P.* : ¿Qué ocurre cuando un emisor no declarado o no especificado emite un certificado? ¿Dónde puedo obtener la lista exhaustiva de emisores activos de una PKI determinada?
*R.* : Si la declaración del certificado especifica huellas digitales de emisor, y el emisor directo del certificado no está incluido en la lista de emisores asignados, el certificado no se considerará válido, independientemente de si el cliente confía en su raíz o no. Por lo tanto, es fundamental asegurarse de que la lista de emisores esté actualizada. La introducción de un nuevo emisor es un evento poco habitual y debe anunciarse ampliamente antes de que empiece a emitir certificados. 

En general, una PKI publicará y mantendrá una declaración de prácticas de certificación, de acuerdo con [RFC 7382](https://tools.ietf.org/html/rfc7382) de IETF. Entre otros datos, incluirá todos los emisores activos. La recuperación de esta lista mediante programación puede diferir de una PKI a otra.   

Para PKI internas de Microsoft, consulte la documentación interna sobre los puntos de conexión o SDK que se utilizan para recuperar los emisores autorizados; es responsabilidad del propietario del clúster revisar esta lista periódicamente y asegurarse de que su definición del clúster incluya *todos* los emisores esperados.

*P.* : ¿Se admiten varias PKI? 
*R.* : Sí, no puede declarar varias entradas de CN en el manifiesto de clúster con el mismo valor, pero puede enumerar emisores de varias PKI correspondientes al mismo CN. No es una práctica recomendada, y las prácticas de transparencia de certificados pueden impedir que se emitan dichos certificados. Sin embargo, como medio para migrar de una PKI a otra, se considera un mecanismo aceptable.

*P.* : ¿Qué ocurre si el certificado del clúster actual no está emitido por una CA o no tiene el firmante previsto? 
*R.* : Obtenga un certificado con el firmante previsto y agréguelo a la definición del clúster como secundario, por huella digital. Una vez que la actualización se haya completado correctamente, inicie otra actualización de la configuración del clúster para convertir la declaración del certificado al nombre común. 

[Image1]:./media/security-cluster-certificate-mgmt/certificate-journey-thumbprint.png
[Image2]:./media/security-cluster-certificate-mgmt/certificate-journey-common-name.png
