<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="887b5-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="887b5-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="887b5-102">Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="887b5-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="887b5-103">En este paso, integrará la GEM [omniauth-OAuth2](https://github.com/omniauth/omniauth-oauth2) en la aplicación y creará una estrategia personalizada de omniauth.</span><span class="sxs-lookup"><span data-stu-id="887b5-103">In this step you will integrate the [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem into the application, and create a custom OmniAuth strategy.</span></span>

<span data-ttu-id="887b5-104">En primer lugar, cree un archivo independiente para hospedar el identificador de la aplicación y el secreto.</span><span class="sxs-lookup"><span data-stu-id="887b5-104">First, create a separate file to hold your app ID and secret.</span></span> <span data-ttu-id="887b5-105">Cree un nuevo archivo llamado `oauth_environment_variables.rb` en la `./config` carpeta y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="887b5-105">Create a new file called `oauth_environment_variables.rb` in the `./config` folder, and add the following code.</span></span>

```ruby
ENV['AZURE_APP_ID'] = 'YOUR_APP_ID_HERE'
ENV['AZURE_APP_SECRET'] = 'YOUR_APP_SECRET_HERE'
ENV['AZURE_SCOPES'] = 'openid profile email offline_access user.read calendars.read'
```

<span data-ttu-id="887b5-106">Reemplace `YOUR_APP_ID_HERE` por el identificador de la aplicación del portal de registro de la `YOUR_APP_SECRET_HERE` aplicación y reemplace por la contraseña que ha generado.</span><span class="sxs-lookup"><span data-stu-id="887b5-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="887b5-107">Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el `oauth_environment_variables.rb` archivo del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación y la contraseña.</span><span class="sxs-lookup"><span data-stu-id="887b5-107">If you're using source control such as git, now would be a good time to exclude the `oauth_environment_variables.rb` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

<span data-ttu-id="887b5-108">Ahora, agregue código para cargar este archivo si está presente.</span><span class="sxs-lookup"><span data-stu-id="887b5-108">Now add code to load this file if it's present.</span></span> <span data-ttu-id="887b5-109">Abra el `./config/environment.rb` archivo y agregue el siguiente código antes de `Rails.application.initialize!` la línea.</span><span class="sxs-lookup"><span data-stu-id="887b5-109">Open the `./config/environment.rb` file and add the following code before the `Rails.application.initialize!` line.</span></span>

```ruby
# Load OAuth settings
oauth_environment_variables = File.join(Rails.root, 'config', 'oauth_environment_variables.rb')
load(oauth_environment_variables) if File.exist?(oauth_environment_variables)
```

## <a name="setup-omniauth"></a><span data-ttu-id="887b5-110">Programa de instalación OmniAuth</span><span class="sxs-lookup"><span data-stu-id="887b5-110">Setup OmniAuth</span></span>

<span data-ttu-id="887b5-111">Ya ha instalado la `omniauth-oauth2` gema, pero para que funcione con los puntos de conexión de Azure OAuth, debe [crear una estrategia de OAuth2](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span><span class="sxs-lookup"><span data-stu-id="887b5-111">You've already installed the `omniauth-oauth2` gem, but in order to make it work with the Azure OAuth endpoints, you need to [create an OAuth2 strategy](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span></span> <span data-ttu-id="887b5-112">Se trata de una clase Ruby que define los parámetros para realizar solicitudes de OAuth en el proveedor de Azure.</span><span class="sxs-lookup"><span data-stu-id="887b5-112">This is a Ruby class that defines the parameters for making OAuth requests to the Azure provider.</span></span>

<span data-ttu-id="887b5-113">Cree un nuevo archivo llamado `microsoft_graph_auth.rb` en la `./lib` carpeta y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="887b5-113">Create a new file called `microsoft_graph_auth.rb` in the `./lib` folder, and add the following code.</span></span>

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

<span data-ttu-id="887b5-114">Tómese un momento para revisar lo que hace el código.</span><span class="sxs-lookup"><span data-stu-id="887b5-114">Take a moment to review what this code does.</span></span>

- <span data-ttu-id="887b5-115">Establece el `client_options` para especificar los puntos de conexión de la plataforma de identidad de Microsoft.</span><span class="sxs-lookup"><span data-stu-id="887b5-115">It sets the `client_options` to specify the Microsoft identity platform endpoints.</span></span>
- <span data-ttu-id="887b5-116">Especifica que el `scope` parámetro debe enviarse durante la fase de autorización.</span><span class="sxs-lookup"><span data-stu-id="887b5-116">It specifies that the `scope` parameter should be sent during the authorize phase.</span></span>
- <span data-ttu-id="887b5-117">Asigna la `id` propiedad del usuario como identificador único del usuario.</span><span class="sxs-lookup"><span data-stu-id="887b5-117">It maps the `id` property of the user as the unique ID for the user.</span></span>
- <span data-ttu-id="887b5-118">Usa el token de acceso para recuperar el perfil del usuario de Microsoft Graph para rellenar `raw_info` el hash.</span><span class="sxs-lookup"><span data-stu-id="887b5-118">It uses the access token to retrieve the user's profile from Microsoft Graph to fill in the `raw_info` hash.</span></span>
- <span data-ttu-id="887b5-119">Invalida la dirección URL de devolución de llamada para asegurarse de que coincide con la devolución de llamada registrada en el portal de registro de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="887b5-119">It overrides the callback URL to ensure that it matches the registered callback in the app registration portal.</span></span>

<span data-ttu-id="887b5-120">Ahora que hemos definido la estrategia, necesitamos configurar OmniAuth para usarla.</span><span class="sxs-lookup"><span data-stu-id="887b5-120">Now that we've defined the strategy, we need to configure OmniAuth to use it.</span></span> <span data-ttu-id="887b5-121">Cree un nuevo archivo llamado `omniauth_graph.rb` en la `./config/initializers` carpeta y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="887b5-121">Create a new file called `omniauth_graph.rb` in the `./config/initializers` folder, and add the following code.</span></span>

```ruby
require 'microsoft_graph_auth'

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :microsoft_graph_auth,
           ENV['AZURE_APP_ID'],
           ENV['AZURE_APP_SECRET'],
           scope: ENV['AZURE_SCOPES']
end
```

<span data-ttu-id="887b5-122">Este código se ejecutará cuando se inicie la aplicación.</span><span class="sxs-lookup"><span data-stu-id="887b5-122">This code will execute when the app starts.</span></span> <span data-ttu-id="887b5-123">Carga el middleware OmniAuth con el `microsoft_graph_auth` proveedor, configurado con las variables de entorno establecidas en. `oauth_environment_variables.rb`</span><span class="sxs-lookup"><span data-stu-id="887b5-123">It loads up the OmniAuth middleware with the `microsoft_graph_auth` provider, configured with the environment variables set in `oauth_environment_variables.rb`.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="887b5-124">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="887b5-124">Implement sign-in</span></span>

<span data-ttu-id="887b5-125">Ahora que el middleware de OmniAuth está configurado, puede pasar a agregar el inicio de sesión a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="887b5-125">Now that the OmniAuth middleware is configured, you can move on to adding sign-in to the app.</span></span> <span data-ttu-id="887b5-126">Ejecute el siguiente comando en su CLI para generar un controlador de inicio y cierre de sesión.</span><span class="sxs-lookup"><span data-stu-id="887b5-126">Run the following command in your CLI to generate a controller for sign-in and sign-out.</span></span>

```Shell
rails generate controller Auth
```

<span data-ttu-id="887b5-127">Abra el archivo `./app/controllers/auth_controller.rb`.</span><span class="sxs-lookup"><span data-stu-id="887b5-127">Open the `./app/controllers/auth_controller.rb` file.</span></span> <span data-ttu-id="887b5-128">Agregue el método siguiente a la clase `AuthController`.</span><span class="sxs-lookup"><span data-stu-id="887b5-128">Add the following method to the `AuthController` class.</span></span>

```ruby
def signin
  redirect_to '/auth/microsoft_graph_auth'
end
```

<span data-ttu-id="887b5-129">Todo este método se redirige a la ruta que OmniAuth espera para invocar nuestra estrategia personalizada.</span><span class="sxs-lookup"><span data-stu-id="887b5-129">All this method does is redirect to the route that OmniAuth expects to invoke our custom strategy.</span></span>

<span data-ttu-id="887b5-130">A continuación, agregue un método de devolución `AuthController` de llamada a la clase.</span><span class="sxs-lookup"><span data-stu-id="887b5-130">Next, add a callback method to the `AuthController` class.</span></span> <span data-ttu-id="887b5-131">Este método será llamado por el middleware OmniAuth una vez que se haya completado el flujo de OAuth.</span><span class="sxs-lookup"><span data-stu-id="887b5-131">This method will be called by the OmniAuth middleware once the OAuth flow is complete.</span></span>

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Temporary for testing!
  render json: data.to_json
end
```

<span data-ttu-id="887b5-132">Por ahora todo esto es representar el hash proporcionado por OmniAuth.</span><span class="sxs-lookup"><span data-stu-id="887b5-132">For now all this does is render the hash provided by OmniAuth.</span></span> <span data-ttu-id="887b5-133">Usaremos esto para comprobar que el inicio de sesión está funcionando antes de continuar.</span><span class="sxs-lookup"><span data-stu-id="887b5-133">We'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="887b5-134">Antes de realizar las pruebas, es necesario agregar las rutas `./config/routes.rb`a.</span><span class="sxs-lookup"><span data-stu-id="887b5-134">Before we test, we need to add the routes to `./config/routes.rb`.</span></span>

```ruby
get 'auth/signin'

# Add route for OmniAuth callback
match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
```

<span data-ttu-id="887b5-135">Ahora, actualice las vistas para usar `signin` la acción.</span><span class="sxs-lookup"><span data-stu-id="887b5-135">Now update the views to use the `signin` action.</span></span> <span data-ttu-id="887b5-136">Abierto `./app/views/layouts/application.html.erb`.</span><span class="sxs-lookup"><span data-stu-id="887b5-136">Open `./app/views/layouts/application.html.erb`.</span></span> <span data-ttu-id="887b5-137">Reemplace la línea `<a href="#" class="nav-link">Sign In</a>` por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="887b5-137">Replace the line `<a href="#" class="nav-link">Sign In</a>` with the following.</span></span>

```html
<%= link_to "Sign In", {:controller => :auth, :action => :signin}, :class => "nav-link" %>
```

<span data-ttu-id="887b5-138">Abra el `./app/views/home/index.html.erb` archivo y reemplace la `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` línea con la siguiente.</span><span class="sxs-lookup"><span data-stu-id="887b5-138">Open the `./app/views/home/index.html.erb` file and replace the `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` line with the following.</span></span>

```html
<%= link_to "Click here to sign in", {:controller => :auth, :action => :signin}, :class => "btn btn-primary btn-large" %>
```

<span data-ttu-id="887b5-139">Inicie el servidor y vaya a `https://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="887b5-139">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="887b5-140">Haga clic en el botón de inicio de sesión y se le `https://login.microsoftonline.com`redirigirá a.</span><span class="sxs-lookup"><span data-stu-id="887b5-140">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="887b5-141">Inicie sesión con su cuenta de Microsoft y dé su consentimiento a los permisos solicitados.</span><span class="sxs-lookup"><span data-stu-id="887b5-141">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="887b5-142">El explorador redirige a la aplicación, que muestra el hash generado por OmniAuth.</span><span class="sxs-lookup"><span data-stu-id="887b5-142">The browser redirects to the app, showing the hash generated by OmniAuth.</span></span>

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

## <a name="storing-the-tokens"></a><span data-ttu-id="887b5-143">Almacenamiento de tokens</span><span class="sxs-lookup"><span data-stu-id="887b5-143">Storing the tokens</span></span>

<span data-ttu-id="887b5-144">Ahora que puede obtener tokens, es el momento de implementar una forma de almacenarlos en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="887b5-144">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="887b5-145">Dado que se trata de una aplicación de ejemplo, por razones de simplicidad, se almacenará en la sesión.</span><span class="sxs-lookup"><span data-stu-id="887b5-145">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="887b5-146">Una aplicación real usaría una solución de almacenamiento seguro más confiable, como una base de datos.</span><span class="sxs-lookup"><span data-stu-id="887b5-146">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="887b5-147">Abra el archivo `./app/controllers/application_controller.rb`.</span><span class="sxs-lookup"><span data-stu-id="887b5-147">Open the `./app/controllers/application_controller.rb` file.</span></span> <span data-ttu-id="887b5-148">Agregará todos los métodos de administración de tokens aquí.</span><span class="sxs-lookup"><span data-stu-id="887b5-148">You'll add all of our token management methods here.</span></span> <span data-ttu-id="887b5-149">Como todos los demás controladores heredan la `ApplicationController` clase, podrán usar estos métodos para tener acceso a los tokens.</span><span class="sxs-lookup"><span data-stu-id="887b5-149">Because all of the other controllers inherit the `ApplicationController` class, they'll be able to use these methods to access the tokens.</span></span>

<span data-ttu-id="887b5-150">Agregue el método siguiente a la clase `ApplicationController`.</span><span class="sxs-lookup"><span data-stu-id="887b5-150">Add the following method to the `ApplicationController` class.</span></span> <span data-ttu-id="887b5-151">El método toma el hash OmniAuth como un parámetro y extrae los bits relevantes de información y, a continuación, lo almacena en la sesión.</span><span class="sxs-lookup"><span data-stu-id="887b5-151">The method takes the OmniAuth hash as a parameter and extracts the relevant bits of information, then stores that in the session.</span></span>

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

<span data-ttu-id="887b5-152">Ahora, agregue funciones de descriptor de acceso para recuperar el nombre de usuario, la dirección de correo electrónico y el token de acceso de la sesión.</span><span class="sxs-lookup"><span data-stu-id="887b5-152">Now add accessor functions to retrieve the user name, email address, and access token back out of the session.</span></span>

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

<span data-ttu-id="887b5-153">Por último, agregue código que se ejecutará antes de que se procese ninguna acción.</span><span class="sxs-lookup"><span data-stu-id="887b5-153">Finally, add some code that will run before any action is processed.</span></span>

```ruby
before_action :set_user

def set_user
  @user_name = user_name
  @user_email = user_email
end
```

<span data-ttu-id="887b5-154">Este método establece las variables que usa el diseño ( `application.html.erb`en) para mostrar la información del usuario en la barra de navegación.</span><span class="sxs-lookup"><span data-stu-id="887b5-154">This method sets the variables that the layout (in `application.html.erb`) uses to show the user's information in the nav bar.</span></span> <span data-ttu-id="887b5-155">Si lo agrega aquí, no tendrá que agregar este código en todas las acciones de controlador.</span><span class="sxs-lookup"><span data-stu-id="887b5-155">By adding it here, you don't have to add this code in every single controller action.</span></span> <span data-ttu-id="887b5-156">Sin embargo, también se ejecutará para las acciones `AuthController`del, lo que no es óptimo.</span><span class="sxs-lookup"><span data-stu-id="887b5-156">However, this will also run for actions in the `AuthController`, which isn't optimal.</span></span> <span data-ttu-id="887b5-157">Agregue el siguiente código a la `AuthController` clase en `./app/controllers/auth_controller.rb` para omitir la acción de antes.</span><span class="sxs-lookup"><span data-stu-id="887b5-157">Add the following code to the `AuthController` class in `./app/controllers/auth_controller.rb` to skip the before action.</span></span>

```ruby
skip_before_action :set_user
```

<span data-ttu-id="887b5-158">A continuación, actualice `callback` la función de `AuthController` la clase para almacenar los tokens en la sesión y volver a redirigir a la Página principal.</span><span class="sxs-lookup"><span data-stu-id="887b5-158">Then, update the `callback` function in the `AuthController` class to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="887b5-159">Reemplace la función `callback` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="887b5-159">Replace the existing `callback` function with the following.</span></span>

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Save the data in the session
  save_in_session data

  redirect_to root_url
end
```

## <a name="implement-sign-out"></a><span data-ttu-id="887b5-160">Implementación de cierre de sesión</span><span class="sxs-lookup"><span data-stu-id="887b5-160">Implement sign-out</span></span>

<span data-ttu-id="887b5-161">Antes de probar esta nueva característica, agregue una forma de cerrar sesión. Agregue la siguiente acción a la `AuthController` clase.</span><span class="sxs-lookup"><span data-stu-id="887b5-161">Before you test this new feature, add a way to sign out. Add the following action to the `AuthController` class.</span></span>

```ruby
def signout
  reset_session
  redirect_to root_url
end
```

<span data-ttu-id="887b5-162">Agregue esta acción a `./config/routes.rb`.</span><span class="sxs-lookup"><span data-stu-id="887b5-162">Add this action to `./config/routes.rb`.</span></span>

```ruby
get 'auth/signout'
```

<span data-ttu-id="887b5-163">Ahora, actualice la vista para usar `signout` la acción.</span><span class="sxs-lookup"><span data-stu-id="887b5-163">Now update the view to use the `signout` action.</span></span> <span data-ttu-id="887b5-164">Abierto `./app/views/layouts/application.html.erb`.</span><span class="sxs-lookup"><span data-stu-id="887b5-164">Open `./app/views/layouts/application.html.erb`.</span></span> <span data-ttu-id="887b5-165">Reemplace la línea `<a href="#" class="dropdown-item">Sign Out</a>` por:</span><span class="sxs-lookup"><span data-stu-id="887b5-165">Replace the line `<a href="#" class="dropdown-item">Sign Out</a>` with:</span></span>

```html
<%= link_to "Sign Out", {:controller => :auth, :action => :signout}, :class => "dropdown-item" %>
```

<span data-ttu-id="887b5-166">Reinicie el servidor y pase por el proceso de inicio de sesión.</span><span class="sxs-lookup"><span data-stu-id="887b5-166">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="887b5-167">Deberás volver a la Página principal, pero la interfaz de usuario debe cambiar para indicar que has iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="887b5-167">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![Una captura de pantalla de la Página principal después de iniciar sesión](./images/add-aad-auth-01.png)

<span data-ttu-id="887b5-169">Haga clic en el avatar de usuario en la esquina superior derecha para acceder al vínculo **Cerrar sesión** .</span><span class="sxs-lookup"><span data-stu-id="887b5-169">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="887b5-170">Al hacer clic en **cerrar** sesión se restablece la sesión y se vuelve a la Página principal.</span><span class="sxs-lookup"><span data-stu-id="887b5-170">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![Captura de pantalla del menú desplegable con el vínculo cerrar sesión](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="887b5-172">Actualizar tokens</span><span class="sxs-lookup"><span data-stu-id="887b5-172">Refreshing tokens</span></span>

<span data-ttu-id="887b5-173">Si observa atentamente el hash generado por OmniAuth, verá que hay dos tokens en el hash: `token` y. `refresh_token`</span><span class="sxs-lookup"><span data-stu-id="887b5-173">If you look closely at the hash generated by OmniAuth, you'll notice there are two tokens in the hash: `token` and `refresh_token`.</span></span> <span data-ttu-id="887b5-174">El valor en `token` es el token de acceso, que se envía en `Authorization` el encabezado de las llamadas a la API.</span><span class="sxs-lookup"><span data-stu-id="887b5-174">The value in `token` is the access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="887b5-175">Este es el token que permite que la aplicación tenga acceso a Microsoft Graph en nombre del usuario.</span><span class="sxs-lookup"><span data-stu-id="887b5-175">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="887b5-176">Sin embargo, este token es de corta duración.</span><span class="sxs-lookup"><span data-stu-id="887b5-176">However, this token is short-lived.</span></span> <span data-ttu-id="887b5-177">El token expira una hora después de su emisión.</span><span class="sxs-lookup"><span data-stu-id="887b5-177">The token expires an hour after it is issued.</span></span> <span data-ttu-id="887b5-178">Aquí es donde el `refresh_token` valor se vuelve útil.</span><span class="sxs-lookup"><span data-stu-id="887b5-178">This is where the `refresh_token` value becomes useful.</span></span> <span data-ttu-id="887b5-179">El token de actualización permite que la aplicación solicite un nuevo token de acceso sin que el usuario tenga que iniciar sesión de nuevo.</span><span class="sxs-lookup"><span data-stu-id="887b5-179">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="887b5-180">Actualice el código de administración de tokens para implementar la actualización de tokens.</span><span class="sxs-lookup"><span data-stu-id="887b5-180">Update the token management code to implement token refresh.</span></span>

<span data-ttu-id="887b5-181">Abra `./app/controllers/application_controller.rb` y agregue las siguientes `require` instrucciones en la parte superior:</span><span class="sxs-lookup"><span data-stu-id="887b5-181">Open `./app/controllers/application_controller.rb` and add the following `require` statements at the top:</span></span>

```ruby
require 'microsoft_graph_auth'
require 'oauth2'
```

<span data-ttu-id="887b5-182">A continuación, agregue el siguiente método `ApplicationController` a la clase.</span><span class="sxs-lookup"><span data-stu-id="887b5-182">Then add the following method to the `ApplicationController` class.</span></span>

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

<span data-ttu-id="887b5-183">Este método usa la gema de [OAuth2](https://github.com/oauth-xx/oauth2) (una dependencia de `omniauth-oauth2` la gema) para actualizar los tokens y actualiza la sesión.</span><span class="sxs-lookup"><span data-stu-id="887b5-183">This method uses the [oauth2](https://github.com/oauth-xx/oauth2) gem (a dependency of the `omniauth-oauth2` gem) to refresh the tokens, and updates the session.</span></span>

<span data-ttu-id="887b5-184">Ahora, ponga este método en uso.</span><span class="sxs-lookup"><span data-stu-id="887b5-184">Now put this method to use.</span></span> <span data-ttu-id="887b5-185">Para ello, se recomienda que `access_token` el descriptor de acceso de la `ApplicationController` clase sea un poco más inteligente.</span><span class="sxs-lookup"><span data-stu-id="887b5-185">To do that, make the `access_token` accessor in the `ApplicationController` class a bit smarter.</span></span> <span data-ttu-id="887b5-186">En lugar de simplemente devolver el token de la sesión, se comprobará primero si está cerca de la fecha de expiración.</span><span class="sxs-lookup"><span data-stu-id="887b5-186">Instead of just returning the token from the session, it will first check if it is close to expiration.</span></span> <span data-ttu-id="887b5-187">Si es así, se actualizará antes de devolver el token.</span><span class="sxs-lookup"><span data-stu-id="887b5-187">If it is, then it will refresh before returning the token.</span></span> <span data-ttu-id="887b5-188">Reemplace el método `access_token` actual por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="887b5-188">Replace the current `access_token` method with the following.</span></span>

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
