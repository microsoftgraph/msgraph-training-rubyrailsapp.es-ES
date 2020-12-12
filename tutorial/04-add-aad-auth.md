<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD. Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph. En este paso, integrará la GEM [omniauth-OAuth2](https://github.com/omniauth/omniauth-oauth2) en la aplicación y creará una estrategia personalizada de omniauth.

1. Cree un archivo independiente para hospedar el identificador de la aplicación y el secreto. Cree un nuevo archivo llamado `oauth_environment_variables.rb` en la carpeta **./config** y agregue el código siguiente.

    :::code language="ruby" source="../demo/graph-tutorial/config/oauth_environment_variables.rb.example":::

1. Reemplace `YOUR_APP_ID_HERE` por el identificador de la aplicación del portal de registro de la aplicación y reemplace `YOUR_APP_SECRET_HERE` por la contraseña que ha generado.

    > [!IMPORTANT]
    > Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el `oauth_environment_variables.rb` archivo del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación y la contraseña.

1. Abra el **./config/Environment.RB** y agregue el siguiente código antes de la `Rails.application.initialize!` línea.

    :::code language="ruby" source="../demo/graph-tutorial/config/environment.rb" id="LoadOAuthSettingsSnippet" highlight="4-6":::

## <a name="setup-omniauth"></a>Programa de instalación OmniAuth

Ya ha instalado la `omniauth-oauth2` gema, pero para que funcione con los puntos de conexión de Azure OAuth, debe [crear una estrategia de OAuth2](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy). Se trata de una clase Ruby que define los parámetros para realizar solicitudes de OAuth en el proveedor de Azure.

1. Cree un nuevo archivo llamado `microsoft_graph_auth.rb` en la carpeta **./lib**' * * y agregue el siguiente código.

    :::code language="ruby" source="../demo/graph-tutorial/lib/microsoft_graph_auth.rb" id="AuthStrategySnippet":::

    Tómese un momento para revisar lo que hace el código.

    - Establece el `client_options` para especificar los puntos de conexión de la plataforma de identidad de Microsoft.
    - Especifica que el `scope` parámetro debe enviarse durante la fase de autorización.
    - Asigna la `id` propiedad del usuario como identificador único del usuario.
    - Usa el token de acceso para recuperar el perfil del usuario de Microsoft Graph para rellenar el `raw_info` hash.
    - Invalida la dirección URL de devolución de llamada para asegurarse de que coincide con la devolución de llamada registrada en el portal de registro de la aplicación.

1. Cree un nuevo archivo llamado `omniauth_graph.rb` en la carpeta **./config/Initializers** y agregue el siguiente código.

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/omniauth_graph.rb" id="ConfigureOmniAuthSnippet":::

    Este código se ejecutará cuando se inicie la aplicación. Carga el middleware OmniAuth con el `microsoft_graph_auth` proveedor, configurado con las variables de entorno establecidas en **oauth_environment_variables. RB**.

## <a name="implement-sign-in"></a>Implementar el inicio de sesión

Ahora que el middleware de OmniAuth está configurado, puede pasar a agregar el inicio de sesión a la aplicación.

1. Ejecute el siguiente comando en su CLI para generar un controlador de inicio y cierre de sesión.

    ```Shell
    rails generate controller Auth
    ```

1. Abra **./app/controllers/auth_controller. RB**. Agregue un método de devolución de llamada a la `AuthController` clase. Este método será llamado por el middleware OmniAuth una vez que se haya completado el flujo de OAuth.

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      data = request.env['omniauth.auth']

      # Temporary for testing!
      render json: data.to_json
    end
    ```

    Por ahora todo esto es representar el hash proporcionado por OmniAuth. Usará esto para comprobar que el inicio de sesión está funcionando antes de continuar.

1. Agregue las rutas a **./config/Routes.RB**.

    ```ruby
    # Add route for OmniAuth callback
    match '/auth/:provider/callback', :to => 'auth#callback', :via => [:get, :post]
    ```

1. Inicie el servidor y vaya a `https://localhost:3000` . Haga clic en el botón de inicio de sesión y se le redirigirá a `https://login.microsoftonline.com` . Inicie sesión con su cuenta de Microsoft y dé su consentimiento a los permisos solicitados. El explorador redirige a la aplicación, que muestra el hash generado por OmniAuth.

    ```json
    {
      "provider": "microsoft_graph_auth",
      "uid": "eb52b3b2-c4ac-4b4f-bacd-d5f7ece55df0",
      "info": {
        "name": null
      },
      "credentials": {
        "token": "eyJ0eXAi...",
        "refresh_token": "OAQABAAA...",
        "expires_at": 1529517383,
        "expires": true
      },
      "extra": {
        "raw_info": {
          "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users(displayName,mail,mailboxSettings,userPrincipalName)/$entity",
          "displayName": "Lynne Robbins",
          "mail": "LynneR@contoso.OnMicrosoft.com",
          "userPrincipalName": "LynneR@contoso.OnMicrosoft.com",
          "id": "d294e784-840e-4f9f-bb1e-95c0a75f2f18@2d18179c-4386-4cbd-8891-7fd867c4f62e",
          "mailboxSettings": {
            "archiveFolder": "AAMkAGI2...",
            "timeZone": "Pacific Standard Time",
            "delegateMeetingMessageDeliveryOptions": "sendToDelegateOnly",
            "dateFormat": "M/d/yyyy",
            "timeFormat": "h:mm tt",
            "automaticRepliesSetting": {
              "status": "disabled",
              "externalAudience": "all",
              "internalReplyMessage": "",
              "externalReplyMessage": "",
              "scheduledStartDateTime": {
                "dateTime": "2020-12-09T17:00:00.0000000",
                "timeZone": "UTC"
              },
              "scheduledEndDateTime": {
                "dateTime": "2020-12-10T17:00:00.0000000",
                "timeZone": "UTC"
              }
            },
            "language": {
              "locale": "en-US",
              "displayName": "English (United States)"
            },
            "workingHours": {
              "daysOfWeek": [
                "monday",
                "tuesday",
                "wednesday",
                "thursday",
                "friday"
              ],
              "startTime": "08:00:00.0000000",
              "endTime": "17:00:00.0000000",
              "timeZone": {
                "name": "Pacific Standard Time"
              }
            }
          }
        }
      }
    }
    ```

## <a name="storing-the-tokens"></a>Almacenamiento de tokens

Ahora que puede obtener tokens, es el momento de implementar una forma de almacenarlos en la aplicación. Dado que se trata de una aplicación de ejemplo, por razones de simplicidad, se almacenará en la sesión. Una aplicación real usaría una solución de almacenamiento seguro más confiable, como una base de datos.

1. Abra **./app/controllers/application_controller. RB**. Agregue el método siguiente a la clase `ApplicationController`.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="SaveInSessionSnippet":::

    El método toma el hash OmniAuth como un parámetro y extrae los bits relevantes de información y, a continuación, lo almacena en la sesión.

1. Agregue funciones de descriptor de acceso a la `ApplicationController` clase para recuperar el nombre de usuario, la dirección de correo electrónico y el token de acceso de la sesión.

    ```ruby
    def user_name
      session[:user_name]
    end

    def user_email
      session[:user_email]
    end

    def user_timezone
      session[:user_timezone]
    end

    def access_token
      session[:graph_token_hash][:token]
    end
    ```

1. Agregue el siguiente código a la `ApplicationController` clase que se ejecutará antes de que se procese ninguna acción.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="BeforeActionSnippet":::

    Este método establece las variables que el diseño (en **application.html. Erb**) utiliza para mostrar la información del usuario en la barra de navegación. Si lo agrega aquí, no tendrá que agregar este código en todas las acciones de controlador. Sin embargo, también se ejecutará para las acciones del `AuthController` , lo que no es óptimo.

1. Agregue el siguiente código a la `AuthController` clase en **./app/Controllers/auth_controller. RB** para omitir la acción anterior.

    ```ruby
    skip_before_action :set_user
    ```

1. Actualice la `callback` función de la `AuthController` clase para almacenar los tokens en la sesión y volver a redirigir a la Página principal. Reemplace la función `callback` existente por lo siguiente.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="CallbackSnippet":::

## <a name="implement-sign-out"></a>Implementación de cierre de sesión

Antes de probar esta nueva característica, agregue una forma de cerrar sesión.

1. Agregue la siguiente acción a la `AuthController` clase.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="SignOutSnippet":::

1. Agregue esta acción a **./config/Routes.RB**.

    ```ruby
    get 'auth/signout'
    ```

1. Reinicie el servidor y pase por el proceso de inicio de sesión. Deberás volver a la Página principal, pero la interfaz de usuario debe cambiar para indicar que has iniciado sesión.

    ![Una captura de pantalla de la Página principal después de iniciar sesión](./images/add-aad-auth-01.png)

1. Haga clic en el avatar de usuario en la esquina superior derecha para acceder al vínculo **Cerrar sesión** . Al hacer clic en **cerrar** sesión se restablece la sesión y se vuelve a la Página principal.

    ![Captura de pantalla del menú desplegable con el vínculo cerrar sesión](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Actualizar tokens

Si observa atentamente el hash generado por OmniAuth, verá que hay dos tokens en el hash: `token` y `refresh_token` . El valor en `token` es el token de acceso, que se envía en el `Authorization` encabezado de las llamadas a la API. Este es el token que permite que la aplicación tenga acceso a Microsoft Graph en nombre del usuario.

Sin embargo, este token es de corta duración. El token expira una hora después de su emisión. Aquí es donde el `refresh_token` valor se vuelve útil. El token de actualización permite que la aplicación solicite un nuevo token de acceso sin que el usuario tenga que iniciar sesión de nuevo. Actualice el código de administración de tokens para implementar la actualización de tokens.

1. Abra **./app/controllers/application_controller. RB** y agregue las siguientes `require` instrucciones en la parte superior:

    ```ruby
    require 'microsoft_graph_auth'
    require 'oauth2'
    ```

1. Agregue el método siguiente a la clase `ApplicationController`.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="RefreshTokensSnippet":::

    Este método usa la gema de [OAuth2](https://github.com/oauth-xx/oauth2) (una dependencia de la `omniauth-oauth2` gema) para actualizar los tokens y actualiza la sesión.

1. Reemplace el `access_token` método actual por lo siguiente.

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="AccessTokenSnippet":::

    En lugar de simplemente devolver el token de la sesión, se comprobará primero si está cerca de la fecha de expiración. Si es así, se actualizará antes de devolver el token.
