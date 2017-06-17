Use el comando [az appservice web create](/cli/azure/appservice/web#create) para crear la aplicación web. En el siguiente comando, sustituya un nombre de aplicación único donde vea el marcador de posición `<app_name>`. Se usa `<app_name>` para la aplicación web en el sitio DNS predeterminado. Si `<app_name>` no es único, obtendrá el mensaje de error descriptivo "Ya existe un sitio web con el nombre especificado <nombre_aplicación>".

```azurecli
az appservice web create --name <app_name> --resource-group myResourceGroup --plan quickStartPlan
```

Cuando se ha creado la aplicación web, la CLI de Azure muestra información similar a la del ejemplo siguiente:

```json
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app_name>.azurewebsites.net",
  "enabled": true,
  "enabledHostNames": [
    "<app_name>.azurewebsites.net",
    "<app_name>.scm.azurewebsites.net"
  ],
  "gatewaySiteName": null,
  "hostNameSslStates": [
    {
      "hostType": "Standard",
      "name": "<app_name>.azurewebsites.net",
      "sslState": "Disabled",
      "thumbprint": null,
      "toUpdate": null,
      "virtualIp": null
    }
    < JSON data removed for brevity. >
}
```

Vaya al sitio (`http://<app_name>.azurewebsites.net`) para ver la aplicación web recién creada.

![app-service-web-service-created](../articles/app-service-web/media/app-service-web-get-started-nodejs-poc/app-service-web-service-created.png)


## <a name="configure-local-git-deployment"></a>Configurar la implementación de Git local

App Service admite varias maneras de implementar contenido en una aplicación web, como FTP, Git local, GitHub, Visual Studio Team Services y Bitbucket. 

Para esta guía de inicio rápido, va a realizar la implementación mediante el uso de Git local. Esto significa que va a realizar la implementación con un comando de Git para insertar desde un repositorio local en un repositorio de Azure. 

Use el comando [az appservice web source-control config-local-git](/cli/azure/appservice/web/source-control#config-local-git) para configurar el acceso de Git local a la aplicación web.

```azurecli
az appservice web source-control config-local-git --name <app_name> --resource-group myResourceGroup --query url --output tsv
```

La salida tiene el siguiente formato:

```
https://<username>@<app_name>.scm.azurewebsites.net:443/<app_name>.git
```

Guarde el URI que se muestra. La usará en el paso siguiente. `<username>` es el [usuario de implementación](#configure-a-deployment-user) que creó en el paso anterior.

## <a name="push-to-azure-from-git"></a>Inserción en Azure desde Git

Agregue una instancia remota de Azure en el repositorio de Git local.

```bash
git remote add azure <URI del paso anterior>
```

Inserte en la instancia remota de Azure para implementar la aplicación. Se le pedirá la contraseña que proporcionó anteriormente al crear el usuario de implementación. Asegúrese de escribir la contraseña que creó en [Configuración de un usuario de implementación](#configure-a-deployment-user), no la contraseña que usa para iniciar sesión en Azure Portal.

```azurecli
git push azure master
```

El comando anterior muestra información similar a la del ejemplo siguiente:

```bash
Counting objects: 13, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (11/11), done.
Writing objects: 100% (13/13), 2.07 KiB | 0 bytes/s, done.
Total 13 (delta 2), reused 0 (delta 0)
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id 'cc39b1e4cb'.
remote: Generating deployment script.
remote: Generating deployment script for Web Site
remote: Generated deployment script files
remote: Running deployment command...
remote: Handling Basic Web Site deployment.
remote: KuduSync.NET from: 'D:\home\site\repository' to: 'D:\home\site\wwwroot'
remote: Deleting file: 'hostingstart.html'
remote: Copying file: '.gitignore'
remote: Copying file: 'LICENSE'
remote: Copying file: 'README.md'
remote: Finished successfully.
remote: Running post deployment command(s)...
remote: Deployment successful.
To https://<app_name>.scm.azurewebsites.net/<app_name>.git
 * [new branch]      master -> master
```

### <a name="browse-to-the-app"></a>Navegación hasta la aplicación


Vaya a la aplicación implementada:

```
http://<app_name>.azurewebsites.net
```

