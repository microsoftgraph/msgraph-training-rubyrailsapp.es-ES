<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD. Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph. En este paso, integrará la GEM [omniauth-OAuth2](https://github.com/omniauth/omniauth-oauth2) en la aplicación y creará una estrategia personalizada de omniauth.

En primer lugar, cree un archivo independiente para hospedar el identificador de la aplicación y el secreto. Cree un nuevo archivo llamado `oauth_environment_variables.rb` en la `./config` carpeta y agregue el siguiente código.

```ruby
ENV['AZURE_APP_ID'] = 'YOUR_APP_ID_HERE'
ENV['AZURE_APP_SECRET'] = 'YOUR_APP_SECRET_HERE'
ENV['AZURE_SCOPES'] = 'openid profile email offline_access user.read calendars.read'
```

Reemplace `YOUR_APP_ID_HERE` por el identificador de la aplicación del portal de registro de la `YOUR_APP_SECRET_HERE` aplicación y reemplace por la contraseña que ha generado.

> [!IMPORTANT]
> Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el `oauth_environment_variables.rb` archivo del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación y la contraseña.

Ahora, agregue código para cargar este archivo si está presente. Abra el `./config/environment.rb` archivo y agregue el siguiente código antes de `Rails.application.initialize!` la línea.

```ruby
# Load OAuth settings
oauth_environment_variables = File.join(Rails.root, 'config', 'oauth_environment_variables.rb')
load(oauth_environment_variables) if File.exist?(oauth_environment_variables)
```

## <a name="setup-omniauth"></a>Programa de instalación OmniAuth

Ya ha instalado la `omniauth-oauth2` gema, pero para que funcione con los puntos de conexión de Azure OAuth, debe [crear una estrategia de OAuth2](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy). Se trata de una clase Ruby que define los parámetros para realizar solicitudes de OAuth en el proveedor de Azure.

Cree un nuevo archivo llamado `microsoft_graph_auth.rb` en la `./lib` carpeta y agregue el siguiente código.

```ruby
require 'omniauth-oauth2'

module OmniAuth
  module Strategies
    # Implements an OmniAuth strategy to get a Microsoft Graph
    # compatible token from Azure AD
    class MicrosoftGraphAuth < OmniAuth::Strategies::OAuth2
      option :name, :microsoft_graph_auth

      DEFAULT_SCOPE = 'openid email profile User.Read'.freeze

      # Configure the Microsoft identity platform endpoints
      option  :client_options,
              site:          'https://login.microsoftonline.com',
              authorize_url: '/common/oauth2/v2.0/authorize',
              token_url:     '/common/oauth2/v2.0/token'

      # Send the scope parameter during authorize
      option :authorize_options, [:scope]

      # Unique ID for the user is the id field
      uid { raw_info['id'] }

      # Get additional information after token is retrieved
      extra do
        {
          'raw_info' => raw_info
        }
      end

      def raw_info
        # Get user profile information from the /me endpoint
        @raw_info ||= access_token.get('https://graph.microsoft.com/v1.0/me').parsed
      end

      def authorize_params
        super.tap do |params|
          params['scope'.to_sym] = request.params['scope'] if request.params['scope']
          params[:scope] ||= DEFAULT_SCOPE
        end
      end

      # Override callback URL
      # OmniAuth by default passes the entire URL of the callback, including
      # query parameters. Azure fails validation because that doesn't match the
      # registered callback.
      def callback_url
        options[:redirect_uri] || (full_host + script_name + callback_path)
      end
    end
  end
end
```

Tómese un momento para revisar lo que hace el código.

- Establece el `client_options` para especificar los puntos de conexión de la plataforma de identidad de Microsoft.
- Especifica que el `scope` parámetro debe enviarse durante la fase de autorización.
- Asigna la `id` propiedad del usuario como identificador único del usuario.
- Usa el token de acceso para recuperar el perfil del usuario de Microsoft Graph para rellenar `raw_info` el hash.
- Invalida la dirección URL de devolución de llamada para asegurarse de que coincide con la devolución de llamada registrada en el portal de registro de la aplicación.

Ahora que hemos definido la estrategia, necesitamos configurar OmniAuth para usarla. Cree un nuevo archivo llamado `omniauth_graph.rb` en la `./config/initializers` carpeta y agregue el siguiente código.

```ruby
require 'microsoft_graph_auth'

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :microsoft_graph_auth,
           ENV['AZURE_APP_ID'],
           ENV['AZURE_APP_SECRET'],
           scope: ENV['AZURE_SCOPES']
end
```

Este código se ejecutará cuando se inicie la aplicación. Carga el middleware OmniAuth con el `microsoft_graph_auth` proveedor, configurado con las variables de entorno establecidas en. `oauth_environment_variables.rb`

## <a name="implement-sign-in"></a>Implementar el inicio de sesión

Ahora que el middleware de OmniAuth está configurado, puede pasar a agregar el inicio de sesión a la aplicación. Ejecute el siguiente comando en su CLI para generar un controlador de inicio y cierre de sesión.

```Shell
rails generate controller Auth
```

Abra el archivo `./app/controllers/auth_controller.rb`. Agregue el método siguiente a la clase `AuthController`.

```ruby
def signin
  redirect_to '/auth/microsoft_graph_auth'
end
```

Todo este método se redirige a la ruta que OmniAuth espera para invocar nuestra estrategia personalizada.

A continuación, agregue un método de devolución `AuthController` de llamada a la clase. Este método será llamado por el middleware OmniAuth una vez que se haya completado el flujo de OAuth.

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Temporary for testing!
  render json: data.to_json
end
```

Por ahora todo esto es representar el hash proporcionado por OmniAuth. Usaremos esto para comprobar que el inicio de sesión está funcionando antes de continuar. Antes de realizar las pruebas, es necesario agregar las rutas `./config/routes.rb`a.

```ruby
get 'auth/signin'

# Add route for OmniAuth callback
match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
```

Ahora, actualice las vistas para usar `signin` la acción. Abierto `./app/views/layouts/application.html.erb`. Reemplace la línea `<a href="#" class="nav-link">Sign In</a>` por lo siguiente.

```html
<%= link_to "Sign In", {:controller => :auth, :action => :signin}, :class => "nav-link" %>
```

Abra el `./app/views/home/index.html.erb` archivo y reemplace la `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` línea con la siguiente.

```html
<%= link_to "Click here to sign in", {:controller => :auth, :action => :signin}, :class => "btn btn-primary btn-large" %>
```

Inicie el servidor y vaya a `https://localhost:3000`. Haga clic en el botón de inicio de sesión y se le `https://login.microsoftonline.com`redirigirá a. Inicie sesión con su cuenta de Microsoft y dé su consentimiento a los permisos solicitados. El explorador redirige a la aplicación, que muestra el hash generado por OmniAuth.

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
      "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
      "id": "eb52b3b2-c4ac-4b4f-bacd-d5f7ece55df0",
      "businessPhones": [
        "+1 425 555 0109"
      ],
      "displayName": "Adele Vance",
      "givenName": "Adele",
      "jobTitle": "Retail Manager",
      "mail": "AdeleV@contoso.onmicrosoft.com",
      "mobilePhone": null,
      "officeLocation": "18/2111",
      "preferredLanguage": "en-US",
      "surname": "Vance",
      "userPrincipalName": "AdeleV@contoso.onmicrosoft.com"
    }
  }
}
```

## <a name="storing-the-tokens"></a>Almacenamiento de tokens

Ahora que puede obtener tokens, es el momento de implementar una forma de almacenarlos en la aplicación. Dado que se trata de una aplicación de ejemplo, por razones de simplicidad, se almacenará en la sesión. Una aplicación real usaría una solución de almacenamiento seguro más confiable, como una base de datos.

Abra el archivo `./app/controllers/application_controller.rb`. Agregará todos los métodos de administración de tokens aquí. Como todos los demás controladores heredan la `ApplicationController` clase, podrán usar estos métodos para tener acceso a los tokens.

Agregue el método siguiente a la clase `ApplicationController`. El método toma el hash OmniAuth como un parámetro y extrae los bits relevantes de información y, a continuación, lo almacena en la sesión.

```ruby
def save_in_session(auth_hash)
  # Save the token info
  session[:graph_token_hash] = auth_hash.dig(:credentials)
  # Save the user's display name
  session[:user_name] = auth_hash.dig(:extra, :raw_info, :displayName)
  # Save the user's email address
  # Use the mail field first. If that's empty, fall back on
  # userPrincipalName
  session[:user_email] = auth_hash.dig(:extra, :raw_info, :mail) ||
                         auth_hash.dig(:extra, :raw_info, :userPrincipalName)
end
```

Ahora, agregue funciones de descriptor de acceso para recuperar el nombre de usuario, la dirección de correo electrónico y el token de acceso de la sesión.

```ruby
def user_name
  session[:user_name]
end

def user_email
  session[:user_email]
end

def access_token
  session[:graph_token_hash][:token]
end
```

Por último, agregue código que se ejecutará antes de que se procese ninguna acción.

```ruby
before_action :set_user

def set_user
  @user_name = user_name
  @user_email = user_email
end
```

Este método establece las variables que usa el diseño ( `application.html.erb`en) para mostrar la información del usuario en la barra de navegación. Si lo agrega aquí, no tendrá que agregar este código en todas las acciones de controlador. Sin embargo, también se ejecutará para las acciones `AuthController`del, lo que no es óptimo. Agregue el siguiente código a la `AuthController` clase en `./app/controllers/auth_controller.rb` para omitir la acción de antes.

```ruby
skip_before_action :set_user
```

A continuación, actualice `callback` la función de `AuthController` la clase para almacenar los tokens en la sesión y volver a redirigir a la Página principal. Reemplace la función `callback` existente por lo siguiente.

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Save the data in the session
  save_in_session data

  redirect_to root_url
end
```

## <a name="implement-sign-out"></a>Implementación de cierre de sesión

Antes de probar esta nueva característica, agregue una forma de cerrar sesión. Agregue la siguiente acción a la `AuthController` clase.

```ruby
def signout
  reset_session
  redirect_to root_url
end
```

Agregue esta acción a `./config/routes.rb`.

```ruby
get 'auth/signout'
```

Ahora, actualice la vista para usar `signout` la acción. Abierto `./app/views/layouts/application.html.erb`. Reemplace la línea `<a href="#" class="dropdown-item">Sign Out</a>` por:

```html
<%= link_to "Sign Out", {:controller => :auth, :action => :signout}, :class => "dropdown-item" %>
```

Reinicie el servidor y pase por el proceso de inicio de sesión. Deberás volver a la Página principal, pero la interfaz de usuario debe cambiar para indicar que has iniciado sesión.

![Una captura de pantalla de la Página principal después de iniciar sesión](./images/add-aad-auth-01.png)

Haga clic en el avatar de usuario en la esquina superior derecha para acceder al vínculo **Cerrar sesión** . Al hacer clic en **cerrar** sesión se restablece la sesión y se vuelve a la Página principal.

![Captura de pantalla del menú desplegable con el vínculo cerrar sesión](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Actualizar tokens

Si observa atentamente el hash generado por OmniAuth, verá que hay dos tokens en el hash: `token` y. `refresh_token` El valor en `token` es el token de acceso, que se envía en `Authorization` el encabezado de las llamadas a la API. Este es el token que permite que la aplicación tenga acceso a Microsoft Graph en nombre del usuario.

Sin embargo, este token es de corta duración. El token expira una hora después de su emisión. Aquí es donde el `refresh_token` valor se vuelve útil. El token de actualización permite que la aplicación solicite un nuevo token de acceso sin que el usuario tenga que iniciar sesión de nuevo. Actualice el código de administración de tokens para implementar la actualización de tokens.

Abra `./app/controllers/application_controller.rb` y agregue las siguientes `require` instrucciones en la parte superior:

```ruby
require 'microsoft_graph_auth'
require 'oauth2'
```

A continuación, agregue el siguiente método `ApplicationController` a la clase.

```ruby
def refresh_tokens(token_hash)
  oauth_strategy = OmniAuth::Strategies::MicrosoftGraphAuth.new(
    nil, ENV['AZURE_APP_ID'], ENV['AZURE_APP_SECRET']
  )

  token = OAuth2::AccessToken.new(
    oauth_strategy.client, token_hash[:token],
    refresh_token: token_hash[:refresh_token]
  )

  # Refresh the tokens
  new_tokens = token.refresh!.to_hash.slice(:access_token, :refresh_token, :expires_at)

  # Rename token key
  new_tokens[:token] = new_tokens.delete :access_token

  # Store the new hash
  session[:graph_token_hash] = new_tokens
end
```

Este método usa la gema de [OAuth2](https://github.com/oauth-xx/oauth2) (una dependencia de `omniauth-oauth2` la gema) para actualizar los tokens y actualiza la sesión.

Ahora, ponga este método en uso. Para ello, se recomienda que `access_token` el descriptor de acceso de la `ApplicationController` clase sea un poco más inteligente. En lugar de simplemente devolver el token de la sesión, se comprobará primero si está cerca de la fecha de expiración. Si es así, se actualizará antes de devolver el token. Reemplace el método `access_token` actual por lo siguiente.

```ruby
def access_token
  token_hash = session[:graph_token_hash]

  # Get the expiry time - 5 minutes
  expiry = Time.at(token_hash[:expires_at] - 300)

  if Time.now > expiry
    # Token expired, refresh
    new_hash = refresh_tokens token_hash
    new_hash[:token]
  else
    token_hash[:token]
  end
end
```
