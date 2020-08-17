---
title: Replicación geográfica de un registro
description: Introducción a la creación y administración de un registro de contenedor de Azure con replicación geográfica, que permite que el registro atienda varias regiones con réplicas regionales de varios maestros. La replicación geográfica es una característica del nivel de servicio Premium.
author: stevelas
ms.topic: article
ms.date: 07/21/2020
ms.author: stevelas
ms.openlocfilehash: b5d016574fd85047ec349820a747b47d0582958b
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87116798"
---
# <a name="geo-replication-in-azure-container-registry"></a>Replicación geográfica en Azure Container Registry

Las empresas que quieran tener una presencia local o una copia de seguridad activa, pueden optar por ejecutar servicios desde varias regiones de Azure. Como procedimiento recomendado, si coloca un registro de contenedor en cada región donde se ejecuten imágenes, podrá realizar operaciones cercanas a la red y habilitar transferencias de capa de imagen rápidamente y de forma fiable. La replicación geográfica permite que un registro de contenedor de Azure funcione como un registro único que atienda a varias regiones con varios registros regionales maestros. 

Un registro con replicación geográfica le proporciona las siguientes ventajas:

* Nombres de registro, imágenes y etiquetas únicos que se pueden usar en varias regiones.
* Acceso al registro cercano a la red desde implementaciones regionales.
* No hay tarifas de salida adicionales, ya que las imágenes se extraen de un registro replicado local que está en la misma región que el host de contenedor.
* Administración única de un registro en varias regiones.

> [!NOTE]
> Si necesita mantener copias de las imágenes de contenedor en más de un registro de contenedor de Azure, Azure Container Registry también admite la [importación de imágenes](container-registry-import-images.md). Por ejemplo, en un flujo de trabajo de DevOps, puede importar una imagen desde un registro de desarrollo a un registro de producción, sin necesidad de usar comandos de Docker.
>

## <a name="example-use-case"></a>Ejemplo de caso de uso
Contoso ejecuta un sitio web de presencia pública ubicado en los Estados Unidos, Canadá y Europa. Para poder atender a estos mercados con contenido local y cercano a la red, Contoso ejecuta los clústeres de [Azure Kubernetes Service](../aks/index.yml) (AKS) en el Este y Oeste de EE. UU, Centro de Canadá y Oeste de Europa. La aplicación del sitio web se implementa como una imagen de Docker y utiliza el mismo código e imagen en todas las regiones. El contenido (que es local en esa región) se recupera de una base de datos que se aprovisiona de forma única en cada región. Cada implementación regional tiene su configuración única para recursos tales como la base de datos local.

El equipo de desarrollo que está ubicado en Seattle (WA) usará el centro de datos del Oeste de EE. UU.

![Inserciones en varios registros](media/container-registry-geo-replication/before-geo-replicate.png)<br />*Inserciones en varios registros*

Antes de usar las características de replicación geográfica, Contoso tenía un registro basado en los Estados Unidos de la zona Oeste de EE. UU. con un registro adicional en Oeste de Europa. Para dar servicio a estas regiones, el equipo de desarrollo tenía que insertar imágenes en dos registros diferentes.

```bash
docker push contoso.azurecr.io/public/products/web:1.2
docker push contosowesteu.azurecr.io/public/products/web:1.2
```
![Extracciones desde varios registros](media/container-registry-geo-replication/before-geo-replicate-pull.png)<br />*Extracciones desde varios registros*

Entre los desafíos típicos de tener varios registros se incluyen:

* Todos los clústeres del Este de EE. UU., el Oeste de EE. UU. y Centro de Canadá extraen contenido del registro del Oeste de EE. UU., por lo que se aplican cuotas de salida cada vez que uno de esos hosts de contenedor remotos extrae imágenes de los centros de datos del Oeste de EE. UU.
* El equipo de desarrollo debe insertar imágenes en los registros del Oeste de EE. UU. y Oeste de Europa.
* El equipo de desarrollo debe configurar y mantener todas las implementaciones regionales con nombres de imagen que hagan referencia al registro local.
* El acceso al registro debe configurarse para cada región.

## <a name="benefits-of-geo-replication"></a>Ventajas de la replicación geográfica

![Extracciones desde un registro con replicación geográfica](media/container-registry-geo-replication/after-geo-replicate-pull.png)

Al usar la característica de replicación geográfica de Azure Container Registry, disfrutará de las siguientes ventajas:

* Podrá administrar un registro único en todas las regiones: `contoso.azurecr.io`.
* Podrá administrar una única configuración de las implementaciones de imagen, ya que todas las regiones usarán la misma dirección URL de imagen: `contoso.azurecr.io/public/products/web:1.2`.
* Mientras ACR administra la replicación geográfica, realice la inserción en un único registro. Puede configurar [webhooks](container-registry-webhook.md) regionales para recibir notificaciones sobre los eventos de réplicas específicas.

## <a name="configure-geo-replication"></a>Configuración de la replicación geográfica

Configurar la replicación geográfica es tan fácil como hacer clic en las regiones de un mapa. También puede administrar la replicación geográfica mediante herramientas como los comandos [az acr replication](/cli/azure/acr/replication) de la CLI de Azure, o implementar un registro habilitado para la replicación geográfica con una [plantilla de Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-container-registry-geo-replication).

La replicación geográfica es una característica de los [registros Premium](container-registry-skus.md). Si el registro no es Premium, puede cambiarlo de Básico y Estándar a Premium en [Azure Portal](https://portal.azure.com):

![Cambio de los niveles de servicio en Azure Portal](media/container-registry-skus/update-registry-sku.png)

Para configurar la replicación geográfica del registro Premium, inicie sesión en Azure Portal en https://portal.azure.com.

Vaya a Azure Container Registry y seleccione **Replicaciones**:

![Replicaciones en la interfaz de usuario del registro de contenedor en Azure Portal](media/container-registry-geo-replication/registry-services.png)

Se mostrará un mapa que indica todas las regiones de Azure actuales:

 ![Mapa de regiones de Azure Portal](media/container-registry-geo-replication/registry-geo-map.png)

* Los hexágonos azules representan las réplicas actuales.
* Los hexágonos verdes representan las regiones de réplica posibles.
* Los hexágonos grises representan regiones de Azure que aún no están disponibles para la replicación.

Para configurar una réplica, seleccione un hexágono verde y, a continuación, seleccione **Crear**:

 ![Interfaz de usuario de la creación de la replicación en Azure Portal](media/container-registry-geo-replication/create-replication.png)

Para configurar réplicas adicionales, seleccione los hexágonos verdes de otras regiones y, a continuación, haga clic en **Crear**.

ACR comenzará entonces a sincronizar imágenes entre las réplicas configuradas. Una vez completada la operación, el portal muestra el mensaje *Listo*. Tenga en cuenta que el estado de la réplica no se actualiza de forma automática en el portal. Para ver el estado actualizado, use el botón Actualizar.

## <a name="considerations-for-using-a-geo-replicated-registry"></a>Consideraciones sobre el uso de un registro con replicación geográfica

* Cada región de un registro con replicación geográfica es independiente una vez configurada. Los Acuerdos de Nivel de Servicio de Azure Container Registry se aplican a cada región con replicación geográfica.
* Al insertar o extraer imágenes de un registro con replicación geográfica, Azure Traffic Manager en segundo plano envía la solicitud al registro ubicado en la región más cercana a usted en cuanto a latencia de red.
* Después de insertar una imagen o una actualización de etiqueta en la región más cercana, Azure Container Registry tarda un rato en replicar los manifiestos y las capas en el resto de regiones que participan. Las imágenes más grandes tardan más en replicarse que las más pequeñas. Las imágenes y etiquetas se sincronizan entre las regiones de replicación con un modelo de coherencia final.
* Para administrar flujos de trabajo que dependen de actualizaciones de inserción para un registro con replicación geográfica, se recomienda configurar [webhooks](container-registry-webhook.md) para responder a los eventos de inserción. Puede configurar webhooks regionales dentro de un registro con replicación geográfica para realizar un seguimiento de los eventos de inserción a medida que se completan en las regiones con replicación geográfica.
* Para prestar servicio a los blobs que representan capas de contenido, Azure Container Registry usa puntos de conexión de datos. Puede habilitar [puntos de conexión de datos dedicados](container-registry-firewall-access-rules.md#enable-dedicated-data-endpoints) para el registro en cada una de las regiones con replicación geográfica del registro. Estos puntos de conexión permiten la configuración de reglas de acceso de firewall con ámbito estricto. Para solucionar los problemas, tiene la opción de [deshabilitar el enrutamiento a una replicación](#temporarily-disable-routing-to-replication) mientras mantiene los datos replicados.
* Si configura un [vínculo privado](container-registry-private-link.md) para el registro mediante puntos de conexión privados de una red virtual, los puntos de conexión de datos dedicados de cada una de las regiones con replicación geográfica se habilitan de forma predeterminada. 

## <a name="delete-a-replica"></a>Eliminación de una réplica

Después de configurar una réplica para el registro, puede eliminarla en cualquier momento cuando ya no sea necesaria. Elimine una réplica mediante Azure Portal u otras herramientas, como el comando [az acr replication delete](/cli/azure/acr/replication#az-acr-replication-delete) de la CLI de Azure.

Para eliminar una réplica en Azure Portal:

1. Vaya a Azure Container Registry y seleccione **Replicaciones**.
1. Seleccione el nombre de una réplica y, luego, **Eliminar**. Confirme que quiere eliminar la réplica.

Para usar la CLI de Azure para eliminar una réplica de *myregistry* en la región Este de EE. UU.:

```azurecli
az acr replication delete --name eastus --registry myregistry
```

## <a name="geo-replication-pricing"></a>Precios de la replicación geográfica

La replicación geográfica es una característica del [nivel de servicio Premium](container-registry-skus.md) de Azure Container Registry. Cuando replica un registro en las áreas indicadas, se aplica la tarifa del registro Premium para cada región.

En el ejemplo anterior, Contoso consolidó dos registros en uno y agregó réplicas en el Este de EE. UU., Centro de Canadá y Oeste de Europa. Por ello, pagaría cuatro veces al mes la tarifa Premium y sin tener ninguna configuración o administración adicional. Ahora, cada región extrae sus imágenes de forma local, lo que mejora el rendimiento y la confiabilidad sin tener que aplicar ninguna tarifa de salida de red desde las regiones del Oeste de EE. UU., Canadá y el Este de EE. UU.

## <a name="troubleshoot-push-operations-with-geo-replicated-registries"></a>Solución de problemas de operaciones de inserción con registros con replicación geográfica
 
Es posible que un cliente de Docker que inserte una imagen en un registro con replicación geográfica no inserte todas las capas de imagen y su manifiesto en una sola región replicada. Esto puede deberse a que Azure Traffic Manager enruta las solicitudes del registro al registro replicado más cercano a la red. Si el registro tiene dos regiones de replicación *cercanas*, las capas de imagen y el manifiesto se pueden distribuir a los dos sitios, y se produce un error en la operación de extracción cuando se valida el manifiesto. Este problema se produce debido a la manera en que el nombre DNS del registro se resuelve en algunos hosts de Linux. Este problema no se produce en Windows, que proporciona una memoria caché de DNS del lado cliente.
 
Si ocurre este problema, una solución consiste en aplicar una caché DNS del lado cliente como `dnsmasq` en el host de Linux. Ayuda a garantizar que el nombre del registro se resuelva de forma coherente. Si usa una máquina virtual Linux en Azure para enviarla a un registro, consulte las opciones en [Opciones de resolución de nombres DNS para máquinas virtuales Linux en Azure](../virtual-machines/linux/azure-dns.md)en Azure.

Para optimizar la resolución DNS para la réplica más cercana al insertar imágenes, configure un registro con replicación geográfica en las mismas regiones de Azure que el origen de las operaciones de inserción o la región más cercana cuando trabaje fuera de Azure.

### <a name="temporarily-disable-routing-to-replication"></a>Deshabilitación temporal del enrutamiento a replicaciones

Para solucionar los problemas de las operaciones con un registro con replicación geográfica, debería deshabilitar temporalmente el enrutamiento de Traffic Manager a una o más replicaciones. A partir de la CLI de Azure versión 2.8, puede configurar una opción `--region-endpoint-enabled` (versión preliminar) al crear o actualizar una región replicada. Cuando se establece la opción `--region-endpoint-enabled` de una replicación en `false`, Traffic Manager deja de enrutar las solicitudes de inserción o incorporación de cambios de Docker a esa región. De forma predeterminada, el enrutamiento está habilitado a todas las replicaciones y la sincronización de datos en todas las replicaciones tiene lugar independientemente de si el enrutamiento está habilitado o deshabilitado.

Para deshabilitar el enrutamiento a una replicación existente, ejecute primero [az acr replication list][az-acr-replication-list] para mostrar las replicaciones en el registro. A continuación, ejecute [az acr replication update][az-acr-replication-update] y establezca `--region-endpoint-enabled false` para una replicación concreta. Por ejemplo, para configurar las opciones de la replicación de *westus* en *myregistry*:

```azurecli
# Show names of existing replications
az acr replication list --registry --output table

# Disable routing to replication
az acr replication update update --name westus \
  --registry myregistry --resource-group MyResourceGroup \
  --region-endpoint-enabled false
```

Para restaurar el enrutamiento a una replicación:

```azurecli
az acr replication update update --name westus \
  --registry myregistry --resource-group MyResourceGroup \
  --region-endpoint-enabled true
```

## <a name="next-steps"></a>Pasos siguientes

Consulte la serie de tres partes del tutorial [Geo-replication in Azure Container Registry (Replicación geográfica en Azure Container Registry)](container-registry-tutorial-prepare-registry.md). Aprenda a crear un registro con replicación geográfica, a compilar un contenedor y a implementarlo con un solo comando `docker push` en varias instancias regionales de Web App for Containers.

> [!div class="nextstepaction"]
> [Replicación geográfica en Azure Container Registry](container-registry-tutorial-prepare-registry.md)

[az-acr-replication-list]: /cli/azure/acr/replication#az-acr-replication-list
[az-acr-replication-update]: /cli/azure/acr/replication#az-acr-replication-update
