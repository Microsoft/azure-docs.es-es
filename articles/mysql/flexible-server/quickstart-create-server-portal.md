---
title: 'Inicio rápido: Creación de una instancia de Azure Database for MySQL con la opción Servidor flexible mediante Azure Portal'
description: Este artículo le ayudará a usar Azure Portal para crear rápidamente una instancia de Azure Database for MySQL con la opción Servidor flexible en cuestión de minutos.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 9/21/2020
ms.openlocfilehash: 4d4d65f9ad04ca6bf99375647684a75e8662bb4d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945691"
---
# <a name="quickstart-use-the-azure-portal-to-create-an-azure-database-for-mysql-flexible-server"></a>Inicio rápido: Uso de Azure Portal para crear una instancia de Azure Database for MySQL con la opción Servidor flexible

Azure Database for MySQL con la opción Servidor flexible es un servicio administrado que se usa para ejecutar, administrar y escalar servidores MySQL de alta disponibilidad en la nube. En este inicio rápido se muestra cómo crear un servidor flexible en cuestión de minutos mediante Azure Portal.

> [!IMPORTANT] 
> Actualmente, Azure Database for MySQL con la opción Servidor flexible está en versión preliminar pública.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal
Abra el explorador web y vaya a [Azure Portal](https://portal.azure.com/). Introduzca sus credenciales para iniciar sesión en el portal. La vista predeterminada es el panel del servicio.

## <a name="create-an-azure-database-for-mysql-flexible-server"></a>Creación de una instancia de Azure Database for MySQL con la opción Servidor flexible

Puede crear un servidor flexible mediante un conjunto definido de [recursos de proceso y almacenamiento](./concepts-compute-storage.md). Cree el servidor dentro en un [grupo de recursos de Azure](https://docs.microsoft.com/azure/azure-resource-manager/management/overview).

Siga estos pasos para crear un servidor flexible.

1. En la esquina superior izquierda del portal, seleccione **Crear un recurso** (+).

2. Seleccione **Bases de datos** > **Azure Database for MySQL**. También puede escribir **MySQL** en el cuadro de búsqueda para encontrar el servicio.

    >[!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstart-create-server-portal/navigate-to-mysql.png" alt-text="Opción Azure Database for MySQL":::

3. Seleccione **Servidor flexible** como opción de implementación.
     
    >[!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstart-create-server-portal/deployment-option.png" alt-text="Elija la opción de implementación":::.    

4. Complete el formulario de **aspectos básicos** con la información siguiente: 

    >[!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstart-create-server-portal/create-form.png" alt-text="Formulario de creación del servidor"::: 
                                    
    |**Configuración**|**Valor sugerido**|**Descripción**|
    |---|---|---|
    Subscription|Nombre de la suscripción|La suscripción de Azure que desea usar para el servidor. Si tiene varias suscripciones, elija aquella en la que quiere que se le facture el recurso.|
    Resource group|*myresourcegroup*| Un nuevo nombre de grupo de recursos o uno existente de la suscripción.|
    Nombre de servidor |*mydemoserver*|Escriba un nombre único que identifique su servidor flexible. El nombre de dominio *mysql.database.azure.com* se anexa al nombre del servidor proporcionado. El servidor solo puede contener letras minúsculas, números y el carácter de guion (-). Debe contener al menos entre tres y 63 caracteres.|
    Nombre de usuario administrador |*mydemouser*| Su propia cuenta de inicio de sesión para usarla al conectarse al servidor. El nombre de inicio de sesión de administrador no puede ser **azure_superuser**, **admin**, **administrator**, **root**, **guest** ni **public**.|
    Contraseña |La contraseña| Una contraseña nueva para la cuenta de administrador del servidor. Debe tener entre 8 y 128 caracteres. La contraseña debe contener caracteres de tres de las siguientes categorías: Letras del alfabeto inglés mayúsculas y minúsculas, números (0 a 9) y caracteres no alfanuméricos (!, $, #, %, etc.).|
    Region|Región más cercana a los usuarios| La ubicación más cercana a los usuarios.|
    Versión|5.7| Versión principal de MySQL.|
    Proceso y almacenamiento | **Flexible**, **Standard_B1ms**, **10 GiB** y **7 días** | Configuración de los recursos de proceso, almacenamiento y copia de seguridad para el nuevo servidor. Seleccione **Configurar servidor**. *Flexible*, *Standard_B1ms*, *10 GiB*y *7 días* son los valores predeterminados para el **nivel de proceso**, el **tamaño de proceso**, el **almacenamiento** y el **período de retención de copia de seguridad**. Puede dejar esos controles deslizantes tal como están o ajustarlos. Para guardar este proceso y la selección de almacenamiento, seleccione **Guardar** a fin de continuar con las configuraciones. En la captura de pantalla siguiente se muestran las opciones de proceso y almacenamiento.|
    
    >[!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstart-create-server-portal/compute-storage.png" alt-text="Proceso y almacenamiento":::

5. Configuración de opciones de redes

    En la pestaña Redes, puede elegir cómo se puede tener acceso al servidor. Azure Database for MySQL con la opción Servidor flexible proporciona las dos opciones siguientes para conectarse a su servidor: *Acceso público (direcciones IP permitidas)* y *Acceso privado (integración con red virtual)* . Con *Acceso público (direcciones IP permitidas)* , el acceso a su servidor se limita a las direcciones IP permitidas agregadas a una regla de firewall. Evita que herramientas y aplicaciones externas se conecten al servidor o a las bases de datos de este, a menos que cree una regla para abrir el firewall para una dirección IP o un intervalo específico. Con la opción *Acceso privado (integración con red virtual)* , el acceso a su servidor se limita a la red virtual. En este inicio rápido, le mostramos cómo habilitar el acceso público para conectarse al servidor. Obtenga más información sobre los métodos de conectividad en el [artículo de conceptos](./concepts-networking.md).

    > [!NOTE]
    > El método de conectividad no se puede cambiar después de crear el servidor. Por ejemplo, si seleccionó la opción *Acceso público (direcciones IP permitidas)* durante la creación, no podrá cambiar a la opción *Acceso público (direcciones IP permitidas)* después de la creación. Se recomienda encarecidamente crear un servidor con acceso privado para acceder de forma segura a su servidor mediante la integración con la red virtual. Obtenga más información sobre el acceso privado en el [artículo de conceptos](./concepts-networking.md).

    >[!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstart-create-server-portal/networking.png" alt-text="Configuración de las redes":::  

    <!--:::image type="content" source="./media/quickstart-create-database-portal/6-add-client-ip.png" alt-text="Select "Add current client IP address"":::-->

6. Seleccione **Revisar y crear** para revisar la configuración del servidor flexible.

7. Seleccione **Crear** para realizar el aprovisionamiento del servidor. El aprovisionamiento puede tardar unos minutos.

8. Haga clic en **Notificaciones** en la barra de herramientas (icono de campana) para supervisar el proceso de implementación. Una vez realizada la implementación, puede seleccionar **Anclar al panel** para crear un icono para este servidor flexible en el panel de Azure Portal como un acceso directo a la página **Información general** del servidor. Al seleccionar **Ir al recurso**, se abre la página **Información general** del servidor.

De forma predeterminada, se crean las bases de datos siguientes en el servidor: **information_schema**, **mysql**, **performance_schema** y **sys**.

> [!NOTE]
> Compruebe si la red permite el tráfico de salida por el puerto 3306 que usa Azure Database for MySQL con la opción Servidor flexible para evitar problemas de conectividad.  

## <a name="connect-to-using-mysql-command-line-client"></a>Conexión mediante el cliente de línea de comandos mysql

Si creó el servidor flexible con la opción *Acceso privado (integración con red la virtual)* , deberá conectarse a su servidor desde un recurso de la misma red virtual que el servidor. Puede crear una máquina virtual y agregarla a la red virtual creada con el servidor flexible.

Si creó el servidor flexible con la opción *Acceso público (direcciones IP permitidas)* , puede agregar la dirección IP local a la lista de reglas de firewall del servidor.

Puede elegir [mysql.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql.html) o [MySQL Workbench](./connect-workbench.md) para conectarse al servidor desde su entorno local. 

Con mysql.exe, conéctese con el siguiente comando. Reemplace los valores por el nombre real del servidor y la contraseña. 

```bash
 mysql -h mydemoserver.mysql.database.azure.com -u mydemouser -p
```
## <a name="clean-up-resources"></a>Limpieza de recursos
Ha creado correctamente una instancia de Azure Database for MySQL con la opción Servidor flexible en un grupo de recursos.  Si no cree que vaya a necesitar estos recursos en el futuro, puede eliminarlos mediante la eliminación del grupo de recursos o bien puede eliminar simplemente el servidor de MySQL. Para eliminar el grupo de recursos, siga estos pasos:

1. En Azure Portal, busque y seleccione **Grupos de recursos**.
1. En la lista de los grupos de recursos, elija el nombre del grupo de recursos.
1. En la página de información general del grupo de recursos, seleccione **Eliminar grupo de recursos**.
1. En el cuadro de diálogo de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.

Para eliminar el servidor, puede hacer clic en botón **Eliminar** de la página **Información general** del servidor, como se muestra a continuación:

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/quickstart-create-server-portal/delete-server.png" alt-text="Eliminación de los recursos":::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Compilación de una aplicación web PHP (Laravel) con MySQL](tutorial-php-database-app.md)
