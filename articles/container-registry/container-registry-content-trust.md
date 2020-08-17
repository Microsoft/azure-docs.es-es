---
title: Administrar imágenes firmadas
description: Aprenda a habilitar la confianza en el contenido para Azure Container Registry y a insertar y extraer imágenes firmadas. La confianza de contenido es una característica del nivel de servicio Premium.
ms.topic: article
ms.date: 09/06/2019
ms.openlocfilehash: 34bb56bab869cb1f12541b65c59b06a73b215377
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076857"
---
# <a name="content-trust-in-azure-container-registry"></a>Confianza en el contenido en Azure Container Registry

Azure Container Registry implementa el modelo de [confianza del contenido][docker-content-trust] de Docker, lo que permite insertar y extraer las imágenes firmadas. Este artículo le permite comenzar a habilitar la confianza del contenido en los registros del contenedor.

> [!NOTE]
> La confianza de contenido es una característica del [nivel de servicio Premium](container-registry-skus.md) de Azure Container Registry.

## <a name="how-content-trust-works"></a>Cómo funciona la confianza en el contenido

En cualquier sistema distribuido diseñado teniendo en cuenta la seguridad es importante comprobar tanto el *origen* como la *integridad* de los datos que entran en el sistema. Los consumidores de los datos deben ser capaces tanto de comprobar el publicador (origen) de los datos, así como de asegurarse de que no se han modificado después de ser publicados (integridad). 

Como publicador de imágenes, la confianza en el contenido le permite **firmar** las imágenes que inserta en el registro. Los consumidores de las imágenes (personas o sistemas que extraen imágenes del registro) pueden configurar sus clientes para extraer *solo* imágenes firmadas. Cuando un consumidor de imágenes extrae una imagen firmada, su cliente de Docker comprueba la integridad de la imagen. En este modelo, los consumidores se aseguran de que las imágenes firmadas del registro fueron publicadas por usted y no se han modificado desde su publicación.

### <a name="trusted-images"></a>Imágenes de confianza

La confianza en el contenido funciona con las **etiquetas** de un repositorio. Los repositorios de imágenes pueden contener imágenes con etiquetas firmadas y sin firmar. Por ejemplo, podría firmar únicamente las imágenes `myimage:stable` y `myimage:latest`, pero no `myimage:dev`.

### <a name="signing-keys"></a>Claves de firma

La confianza en el contenido se administra mediante el uso de un conjunto de claves de firma criptográficas. Estas claves están asociadas a un repositorio específico de un registro. Los clientes de Docker y el registro utilizan varios tipos de claves de firma en la administración de la confianza para las etiquetas de un repositorio. Debe administrar cuidadosamente estas claves al habilitar la confianza en el contenido e integrarla en la canalización de publicación y consumo del contenedor. Para más información, consulte [Administración de claves](#key-management) más adelante en este artículo y [Administración de claves para la confianza en el contenido][docker-manage-keys] en la documentación de Docker.

> [!TIP]
> Esta ha sido una introducción muy general al modelo de confianza en el contenido de Docker. Para una explicación detallada de la confianza en el contenido, consulte [Confianza en el contenido en Docker][docker-content-trust].

## <a name="enable-registry-content-trust"></a>Habilitación de la confianza en el contenido del registro

El primer paso es habilitar la confianza en el contenido en el nivel de registro. Una vez que habilite la confianza en el contenido, los clientes (usuarios o servicios) pueden insertar imágenes firmadas en el registro. Habilitar la confianza en el contenido en el registro no restringe el uso del registro solo a los consumidores con la confianza en el contenido habilitada. Los consumidores sin la confianza en el contenido habilitada pueden seguir usando el registro de la forma habitual. Los consumidores que han habilitado la confianza en el contenido en sus clientes, sin embargo, serán capaces de ver *solo* las imágenes firmadas del registro.

Para habilitar la confianza en el contenido para el registro, primero vaya al registro en Azure Portal. En **Directivas**, seleccione **Confianza del contenido** > **Habilitada** > **Guardar**. También puede usar el comando [az acr config content-trust update][az-acr-config-content-trust-update] en la CLI de Azure.

![Habilitación de la confianza en el contenido para un registro en Azure Portal][content-trust-01-portal]

## <a name="enable-client-content-trust"></a>Habilitación de la confianza en el contenido en el cliente

Para trabajar con imágenes de confianza, tanto los publicadores de imágenes como los consumidores deben habilitar la confianza en el contenido en sus clientes de Docker. Como publicador, puede firmar las imágenes que inserta en un registro con la confianza en el contenido habilitada. Como consumidor, habilitar la confianza en el contenido limita la vista de un registro a únicamente las imágenes firmadas. La confianza en el contenido está deshabilitada de forma predeterminada en los clientes de Docker, pero puede habilitarla para una sesión del shell o para un comando.

Para habilitar la confianza en el contenido para una sesión del shell, establezca la variable de entorno `DOCKER_CONTENT_TRUST` en **1**. Por ejemplo, en el shell de Bash:

```bash
# Enable content trust for shell session
export DOCKER_CONTENT_TRUST=1
```

Si, en su lugar, desea habilitar o deshabilitar la confianza en el contenido para un único comando, varios comandos de Docker admiten el argumento `--disable-content-trust`. Para habilitar la confianza en el contenido para un único comando:

```bash
# Enable content trust for single command
docker build --disable-content-trust=false -t myacr.azurecr.io/myimage:v1 .
```

Si ha habilitado la confianza en el contenido para la sesión del shell y desea deshabilitarla para un único comando:

```bash
# Disable content trust for single command
docker build --disable-content-trust -t myacr.azurecr.io/myimage:v1 .
```

## <a name="grant-image-signing-permissions"></a>Concesión de permisos de firma de imágenes

Solo los usuarios o sistemas a los que se ha concedido permiso pueden insertar imágenes de confianza en el registro. Para conceder permiso de inserción de imágenes de confianza a un usuario (o a un sistema con una entidad de servicio), debe conceder a sus identidades de Azure Active Directory el rol `AcrImageSigner`. Esto se agrega al rol `AcrPush` (o equivalente) necesario para insertar imágenes en el registro. Para más información, consulte [Roles y permisos de Azure Container Registry](container-registry-roles.md).

> [!NOTE]
> No se puede conceder el permiso de inserción de imagen de confianza a la [cuenta de administrador](container-registry-authentication.md#admin-account) de Azure Container Registry.

A continuación, puede ver los detalles de la concesión del rol `AcrImageSigner` en Azure Portal y la CLI de Azure.

### <a name="azure-portal"></a>Azure portal

Vaya al registro en Azure Portal y, a continuación, seleccione **Control de acceso (IAM)**  > **Agregar asignación de rol**. En **Agregar asignación de rol**, seleccione `AcrImageSigner` para el **Rol**, **seleccione** uno o más usuarios o entidades de servicio y, a continuación, **Guardar**.

En este ejemplo, se ha asignado el rol `AcrImageSigner` a dos entidades: una entidad de servicio llamada "service-principal" y un usuario llamado "Azure User".

![Habilitación de la confianza en el contenido para un registro en Azure Portal][content-trust-02-portal]

### <a name="azure-cli"></a>Azure CLI

Para conceder permisos de firma a un usuario con la CLI de Azure, asigne el rol `AcrImageSigner` al usuario, en el ámbito del registro. El formato del comando es:

```azurecli
az role assignment create --scope <registry ID> --role AcrImageSigner --assignee <user name>
```

Por ejemplo, para concederse el rol a usted mismo, puede ejecutar los siguientes comandos en una sesión autenticada de la CLI de Azure. Modifique el valor de `REGISTRY` para que refleje el nombre del registro de contenedor de Azure.

```bash
# Grant signing permissions to authenticated Azure CLI user
REGISTRY=myregistry
USER=$(az account show --query user.name --output tsv)
REGISTRY_ID=$(az acr show --name $REGISTRY --query id --output tsv)
```

```azurecli
az role assignment create --scope $REGISTRY_ID --role AcrImageSigner --assignee $USER
```

También puede conceder a una [entidad de servicio](container-registry-auth-service-principal.md) derechos para insertar imágenes de confianza en el registro. El uso de una entidad de servicio es útil para sistemas de compilación y otros sistemas desatendidos que deban insertar imágenes de confianza en el registro. El formato es similar al utilizado para conceder un permiso de usuario, pero debe especificar un identificador de la entidad de servicio para el valor `--assignee`.

```azurecli
az role assignment create --scope $REGISTRY_ID --role AcrImageSigner --assignee <service principal ID>
```

`<service principal ID>` puede ser el valor de **appId**, de **objectId** o uno de los valores de **servicePrincipalNames** de la entidad de servicio. Para más información sobre cómo trabajar con entidades de servicio y Azure Container Registry, consulte [Autenticación de Azure Container Registry con entidades de servicio](container-registry-auth-service-principal.md).

> [!IMPORTANT]
> Después de realizar cualquier cambio de rol, ejecute `az acr login` para actualizar el token de identidad local para la CLI de Azure, con el fin de que los nuevos roles surtan efecto. Para más información sobre la comprobación de roles para una identidad, consulte el artículo sobre la [administración del acceso a los recursos de Azure mediante RBAC y la CLI de Azure](../role-based-access-control/role-assignments-cli.md) y [Solución de problemas de Azure RBAC](../role-based-access-control/troubleshooting.md).

## <a name="push-a-trusted-image"></a>Inserción de una imagen de confianza

Para insertar una etiqueta de imagen de confianza en el registro de contenedor, habilite la confianza en el contenido e inserte la imagen con `docker push`. La primera vez que inserte una etiqueta firmada, se le solicitará que cree una frase de contraseña para una clave de firma de raíz y una clave de firma del repositorio. Las claves de raíz y del repositorio se generan y almacenan localmente en su equipo.

```console
$ docker push myregistry.azurecr.io/myimage:v1
The push refers to repository [myregistry.azurecr.io/myimage]
ee83fc5847cb: Pushed
v1: digest: sha256:aca41a608e5eb015f1ec6755f490f3be26b48010b178e78c00eac21ffbe246f1 size: 524
Signing and pushing trust metadata
You are about to create a new root signing key passphrase. This passphrase
will be used to protect the most sensitive key in your signing system. Please
choose a long, complex passphrase and be careful to keep the password and the
key file itself secure and backed up. It is highly recommended that you use a
password manager to generate the passphrase and keep it safe. There will be no
way to recover this key. You can find the key in your config directory.
Enter passphrase for new root key with ID 4c6c56a:
Repeat passphrase for new root key with ID 4c6c56a:
Enter passphrase for new repository key with ID bcd6d98:
Repeat passphrase for new repository key with ID bcd6d98:
Finished initializing "myregistry.azurecr.io/myimage"
Successfully signed myregistry.azurecr.io/myimage:v1
```

Después del primer uso de `docker push` con la confianza en el contenido habilitada, el cliente de Docker usa la misma clave de raíz para las inserciones posteriores. En cada inserción posterior en el mismo repositorio, solo se le pedirá la clave del repositorio. Cada vez que inserte una imagen de confianza en un nuevo repositorio, se le pide que proporcione una frase de contraseña para una nueva clave del repositorio.

## <a name="pull-a-trusted-image"></a>Extracción de una imagen de confianza

Para extraer una imagen de confianza, habilite la confianza en el contenido y ejecute el comando `docker pull` como hace habitualmente. Para extraer imágenes de confianza, el rol `AcrPull` es suficiente para los usuarios normales. No se requieren roles adicionales, como `AcrImageSigner`. Los consumidores con la confianza en el contenido habilitada pueden extraer solo imágenes con etiquetas firmadas. Este es un ejemplo de extracción de una etiqueta firmada:

```console
$ docker pull myregistry.azurecr.io/myimage:signed
Pull (1 of 1): myregistry.azurecr.io/myimage:signed@sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b
sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b: Pulling from myimage
8e3ba11ec2a2: Pull complete
Digest: sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b
Status: Downloaded newer image for myregistry.azurecr.io/myimage@sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b
Tagging myregistry.azurecr.io/myimage@sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b as myregistry.azurecr.io/myimage:signed
```

Si un cliente con la confianza en el contenido habilitada trata de extraer una etiqueta sin firmar, se produce un error en la operación:

```console
$ docker pull myregistry.azurecr.io/myimage:unsigned
No valid trust data for unsigned
```

### <a name="behind-the-scenes"></a>Entre bambalinas

Al ejecutar `docker pull`, el cliente de Docker usa la misma biblioteca que en la [CLI de Notary][docker-notary-cli] para solicitar la asignación de hash de etiqueta a SHA-256 para la etiqueta que va a extraer. Después de comprobar las firmas en los datos de confianza, el cliente indica al motor de Docker que realice una "extracción por hash". Durante la extracción, el motor utiliza la suma de comprobación de SHA-256 como una dirección de contenido para solicitar y validar el manifiesto de imagen desde el registro de contenedor de Azure.

## <a name="key-management"></a>Administración de claves

Como se indica en la salida de `docker push` cuando se inserta la primera imagen de confianza, la clave de raíz es la más confidencial. No olvide hacer una copia de seguridad de la clave de raíz y almacenarla en una ubicación segura. De forma predeterminada, el cliente de Docker almacena las claves de firma en el directorio siguiente:

```sh
~/.docker/trust/private
```

Puede hacer una copia de seguridad de las claves de raíz y del repositorio mediante su compresión en un archivo y su almacenamiento en una ubicación segura. Por ejemplo, en Bash:

```bash
umask 077; tar -zcvf docker_private_keys_backup.tar.gz ~/.docker/trust/private; umask 022
```

Junto con las claves de raíz y del repositorio que se generan localmente, Azure Container Registry genera y almacena otras claves cuando se inserta una imagen de confianza. Para una explicación detallada de las diversas claves en la implementación de la confianza en el contenido de Docker, incluida guía de administración adicional, consulte [Administración de claves para la confianza en el contenido][docker-manage-keys] en la documentación de Docker.

### <a name="lost-root-key"></a>Pérdida de la clave de raíz

Si pierde el acceso a la clave de raíz, perderá el acceso a las etiquetas firmadas en cualquiera de los repositorios cuyas etiquetas se firmaron con esa clave. Azure Container Registry no puede restaurar el acceso a las etiquetas de imagen firmadas con una clave de raíz perdida. Para eliminar todos los datos de confianza (firmas) del registro, deshabilite y vuelva a habilitar la confianza en el contenido para el registro.

> [!WARNING]
> Deshabilitar y volver a habilitar la confianza en el contenido en el registro **elimina todos los datos de confianza de todas las etiquetas firmadas de cada repositorio del registro**. Esta acción es irreversible: Azure Container Registry no puede recuperar los datos de confianza eliminados. La deshabilitación de la confianza en el contenido no elimina las imágenes en sí.

Para deshabilitar la confianza en el contenido para el registro, vaya al registro en Azure Portal. En **Directivas**, seleccione **Confianza del contenido** > **Deshabilitada** > **Guardar**. Se le advertirá de la pérdida de todas las firmas del registro. Seleccione **Aceptar** para eliminar permanentemente todas las firmas del registro.

![Deshabilitación de la confianza en el contenido para un registro en Azure Portal][content-trust-03-portal]

## <a name="next-steps"></a>Pasos siguientes

* Para más información acerca de la confianza del contenido, consulte [Content trust in Docker][docker-content-trust] (Confianza en el contenido de Docker). Aunque se han tratado varios aspectos importantes en este artículo, la confianza en el contenido es un tema amplio y se describe con más profundidad en la documentación de Docker.

* Consulte la documentación de [Azure Pipelines](/azure/devops/pipelines/build/content-trust) para un ejemplo de uso de la confianza en contenido al compilar una imagen de Docker e insertarla.

<!-- IMAGES> -->
[content-trust-01-portal]: ./media/container-registry-content-trust/content-trust-01-portal.png
[content-trust-02-portal]: ./media/container-registry-content-trust/content-trust-02-portal.png
[content-trust-03-portal]: ./media/container-registry-content-trust/content-trust-03-portal.png

<!-- LINKS - external -->
[docker-content-trust]: https://docs.docker.com/engine/security/trust/content_trust
[docker-manage-keys]: https://docs.docker.com/engine/security/trust/trust_key_mng/
[docker-notary-cli]: https://docs.docker.com/notary/getting_started/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-config-content-trust-update]: /cli/azure/acr/config/content-trust#az-acr-config-content-trust-update
