---
title: Uso de claves administradas por el cliente para cifrar datos en Azure HPC Cache
description: Cómo usar Azure Key Vault con Azure HPC Cache para controlar el acceso con claves de cifrado en lugar de usar las claves de cifrado administradas por Microsoft predeterminadas
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 07/20/2020
ms.author: v-erkel
ms.openlocfilehash: 2cd97e205d88fe7ead02889f5ae9ad9df0985f07
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092531"
---
# <a name="use-customer-managed-encryption-keys-for-azure-hpc-cache"></a>Uso de claves de cifrado administradas por el cliente para Azure HPC Cache

Puede usar Azure Key Vault para controlar la titularidad de las claves usadas para cifrar los datos en Azure HPC Cache. En este artículo se explica cómo usar claves administradas por el cliente para el cifrado de datos en caché.

> [!NOTE]
> Todos los datos almacenados en Azure, incluidos los discos de caché, se cifran en reposo mediante claves administradas por Microsoft de forma predeterminada. Solo tiene que seguir los pasos de este artículo si desea administrar las claves usadas para cifrar los datos.

Azure HPC Cache también está protegida por el [cifrado del host de la máquina virtual](../virtual-machines/linux/disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data) en los discos administrados que contienen los datos almacenados en caché, incluso si agrega una clave de cliente para los discos de caché. La adición de una clave administrada por el cliente para el cifrado doble proporciona un nivel de seguridad adicional a los clientes con necesidades de alta seguridad. Consulte el artículo [Cifrado del lado servidor de Azure Disk Storage](../virtual-machines/linux/disk-encryption.md) para más información.

Esta característica solo está disponible en algunas de las regiones de Azure donde está disponible Azure HPC Cache. Consulte la lista de [disponibilidad de regiones](hpc-cache-overview.md#region-availability) para más información.

Hay tres pasos para habilitar el cifrado de claves administradas por el cliente para Azure HPC Cache:

1. Configure una instancia de Azure Key Vault para almacenar las claves.
1. Al crear la instancia de Azure HPC Cache, elija el cifrado de claves administradas por el cliente y especifique el almacén de claves y la clave que se usarán.
1. Una vez creada la memoria caché, autorícela para que tenga acceso al almacén de claves.

El cifrado no se configura por completo hasta que se le autorice desde la caché recién creada (paso 3). Esto es porque debe pasar la identidad de la caché al almacén de claves para que sea un usuario autorizado. No se puede hacer esto antes de crear la memoria caché, ya que la identidad no existe hasta que se crea la memoria caché.

Después de crear la caché, no puede cambiar entre claves administradas por el cliente y claves administradas por Microsoft. Sin embargo, si la memoria caché usa claves administradas por el cliente, puede [cambiar](#update-key-settings) la clave de cifrado, la versión de la clave y el almacén de claves según sea necesario.

## <a name="understand-key-vault-and-key-requirements"></a>Descripción de los requisitos del almacén de claves y de la clave

El almacén de claves y la clave deben cumplir con estos requisitos para funcionar con Azure HPC Cache.

Propiedades del almacén de claves:

* **Suscripción**: Use la misma suscripción que se usa para la memoria caché.
* **Región**: El almacén de claves debe estar en la misma región que la instancia de Azure HPC Cache.
* **Plan de tarifa**: El nivel Estándar es suficiente para su uso con Azure HPC Cache.
* **Eliminación temporal**: Azure HPC Cache habilitará la eliminación temporal si aún no está configurada en el almacén de claves.
* **Protección de purga**: La protección de purga debe estar habilitada.
* **Directiva de acceso**: La configuración predeterminada es suficiente.
* **Conectividad de red**: Azure HPC Cache debe poder acceder al almacén de claves con independencia de la configuración del punto de conexión que elija.

Propiedades de la clave:

* **Tipo de clave**: RSA
* **Tamaño de la clave RSA**: 2048
* **Habilitado**: Sí

Permisos de acceso al almacén de claves:

* El usuario que crea la instancia de Azure HPC Cache debe tener permisos equivalentes al [rol de colaborador de Key Vault](../role-based-access-control/built-in-roles.md#key-vault-contributor). Se necesitan los mismos permisos para configurar y administrar Azure Key Vault.

  Para obtener más información, consulte [Protección del acceso a un almacén de claves](../key-vault/key-vault-secure-your-key-vault.md).

## <a name="1-set-up-azure-key-vault"></a>1. Configuración de Azure Key Vault

Puede configurar un almacén de claves y una clave antes de crear la memoria caché, o bien hacerlo como parte de la creación de la memoria caché. Asegúrese de que estos recursos cumplan los requisitos descritos [anteriormente](#understand-key-vault-and-key-requirements).

En el momento de la creación de la memoria caché, debe especificar un almacén, una clave y una versión de clave que se usarán para el cifrado de la caché.

Para más detalles, consulte la [documentación de Azure Key Vault](../key-vault/key-vault-overview.md).

> [!NOTE]
> Azure Key Vault debe usar la misma suscripción y estar en la misma región que Azure HPC Cache. Asegúrese de que la región que elija [admita la característica de claves administradas por el cliente](hpc-cache-overview.md#region-availability).

## <a name="2-create-the-cache-with-customer-managed-keys-enabled"></a>2. Creación de la memoria caché con claves administradas por el cliente habilitadas

Debe especificar el origen de la clave de cifrado al crear su instancia de Azure HPC Cache. Siga las instrucciones de [Creación de una instancia de Azure HPC Cache](hpc-cache-create.md) y especifique el almacén de claves y la clave en la página **Disk encryption keys** (Claves de cifrado de disco). Puede crear un almacén de claves y una clave nuevos durante la creación de la memoria caché.

> [!TIP]
> Si la página **Disk encryption keys** (Claves de cifrado de disco) no aparece, asegúrese de que la memoria caché se encuentre en una de las regiones admitidas.

El usuario que crea la memoria caché debe tener privilegios iguales a los del [rol colaborador de Key Vault](../role-based-access-control/built-in-roles.md#key-vault-contributor) o superior.

1. Haga clic en el botón para habilitar las claves administradas de forma privada. Después de cambiar esta configuración, aparecerá la configuración del almacén de claves.

1. Haga clic en **Seleccionar un almacén de claves** para abrir la página de selección de claves. Elija o cree el almacén de claves y la clave para cifrar los datos en los discos de caché.

   Si su instancia de Azure Key Vault no aparece en la lista, compruebe estos requisitos:

   * ¿Se encuentra la memoria caché en la misma suscripción que el almacén de claves?
   * ¿Se encuentra la memoria caché en la misma región que el almacén de claves?
   * ¿Hay conectividad de red entre Azure Portal y el almacén de claves?

1. Después de seleccionar un almacén, seleccione la clave individual en las opciones disponibles o cree una clave nueva. Debe ser una clave RSA de 2048 bits.

1. Especifique la versión de la clave seleccionada. Obtenga más información sobre el control de versiones en la [documentación de Azure Key Vault](../key-vault/about-keys-secrets-and-certificates.md#objects-identifiers-and-versioning).

Continúe con el resto de las especificaciones y cree la memoria caché como se describe en [Creación de una instancia de Azure HPC Cache](hpc-cache-create.md).

## <a name="3-authorize-azure-key-vault-encryption-from-the-cache"></a>3. Autorización del cifrado de Azure Key Vault desde la memoria caché
<!-- header is linked from create article, update if changed -->

Después de unos minutos, la nueva instancia de Azure HPC Cache aparece en Azure Portal. Vaya a la página **Información general** para autorizarla a acceder a su instancia de Azure Key Vault y habilitar el cifrado de claves administradas por el cliente.

> [!TIP]
> La caché puede aparecer en la lista de recursos antes de que se borren los mensajes de "implementación en curso". Compruebe la lista de recursos al cabo de un minuto o dos en lugar de esperar una notificación de operación correcta.

Este proceso de dos pasos es necesario porque la instancia de Azure HPC Cache necesita una identidad para pasar a la instancia de Azure Key Vault para la autorización. La identidad de la caché no existe hasta que se hayan completado los pasos de creación iniciales.

> [!NOTE]
> Debe autorizar el cifrado en un plazo de 90 minutos después de crear la memoria caché. Si no completa este paso, se agotará el tiempo de espera de la caché y se producirá un error. Una caché con errores debe volver a crearse, no se puede corregir.

La memoria caché muestra el estado **Esperando la clave**. Haga clic en el botón **Habilitar cifrado** en la parte superior de la página para autorizar a la memoria caché a acceder al almacén de claves especificado.

![Captura de pantalla de la página de información general de la caché en el portal, con resaltado en el botón Habilitar cifrado (fila superior) y Estado: Esperando la clave](media/waiting-for-key.png)

Haga clic en **Habilitar cifrado** y, a continuación, haga clic en el botón **Sí** para autorizar a la memoria caché a usar la clave de cifrado. Esta acción también habilita la eliminación temporal y protección de purga (si aún no están habilitadas) en el almacén de claves.

![Captura de pantalla de la página de información general de la caché en el portal, con un mensaje de banner en la parte superior que pide al usuario que habilite el cifrado haciendo clic en Sí](media/enable-keyvault.png)

Después de que la memoria caché solicita acceso al almacén de claves, puede crear y cifrar los discos que almacenan los datos en caché.

Después de autorizar el cifrado, Azure HPC Cache pasa varios minutos más de configuración para crear los discos cifrados y la infraestructura relacionada.

## <a name="update-key-settings"></a>Actualización de la configuración de claves

Puede cambiar el almacén de claves, la clave o la versión de clave de la memoria caché desde Azure Portal. Haga clic en el vínculo configuración **Cifrado** de la memoria caché para abrir la página **Configuración de clave de cliente**.

No se puede cambiar una caché entre claves administradas por el cliente y claves administradas por el sistema.

![Captura de pantalla de la página "Configuración de clave de cliente", a la que se accede al hace clic en Configuración > Cifrado en la página de caché de Azure Portal](media/change-key-click.png)

Haga clic en el vínculo **Cambiar clave** y, a continuación, haga clic en **Cambiar el almacén de claves, la clave o la versión** para abrir el selector de claves.

![Captura de pantalla de la página "Seleccionar clave en Azure Key Vault" con tres selectores desplegables para elegir el almacén de claves, la clave y la versión](media/select-new-key.png)

Los almacenes de claves de la misma suscripción y la misma región que esta caché se muestran en la lista.

Después de elegir los nuevos valores de clave de cifrado, haga clic en **Seleccionar**. Aparece una página de confirmación con los nuevos valores. Haga clic en **Guardar** para finalizar la selección.

![Captura de pantalla de la página de confirmación con el botón Guardar en la parte superior izquierda](media/save-key-settings.png)

## <a name="read-more-about-customer-managed-keys-in-azure"></a>Más información sobre las claves administradas por el cliente en Azure

En estos artículos se explica más sobre el uso de Azure Key Vault y las claves administradas por el cliente para cifrar datos en Azure:

* [Información general del cifrado de almacenamiento en Azure](../storage/common/storage-service-encryption.md)
* [Cifrado de discos con claves administradas por el cliente](../virtual-machines/linux/disk-encryption.md#customer-managed-keys): documentación sobre el uso de Azure Key Vault y discos administrados, que es un escenario similar al de Azure HPC Cache.

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya creado la instancia de Azure HPC Cache y haya autorizado el cifrado basado en Key Vault, continúe con la configuración de la memoria caché concediéndole acceso a los orígenes de datos.

* [Incorporación de destinos de almacenamiento](hpc-cache-add-storage.md)
