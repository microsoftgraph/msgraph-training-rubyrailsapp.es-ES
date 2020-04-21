<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="a22ed-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="a22ed-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="a22ed-102">Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="a22ed-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="a22ed-103">En este paso, integrará la GEM [omniauth-OAuth2](https://github.com/omniauth/omniauth-oauth2) en la aplicación y creará una estrategia personalizada de omniauth.</span><span class="sxs-lookup"><span data-stu-id="a22ed-103">In this step you will integrate the [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem into the application, and create a custom OmniAuth strategy.</span></span>

1. <span data-ttu-id="a22ed-104">Cree un archivo independiente para hospedar el identificador de la aplicación y el secreto.</span><span class="sxs-lookup"><span data-stu-id="a22ed-104">Create a separate file to hold your app ID and secret.</span></span> <span data-ttu-id="a22ed-105">Cree un nuevo archivo llamado `oauth_environment_variables.rb` en la carpeta **./config** y agregue el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="a22ed-105">Create a new file called `oauth_environment_variables.rb` in the **./config** folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/oauth_environment_variables.rb.example":::

1. <span data-ttu-id="a22ed-106">Reemplace `YOUR_APP_ID_HERE` por el identificador de la aplicación del portal de registro de la `YOUR_APP_SECRET_HERE` aplicación y reemplace por la contraseña que ha generado.</span><span class="sxs-lookup"><span data-stu-id="a22ed-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="a22ed-107">Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el `oauth_environment_variables.rb` archivo del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación y la contraseña.</span><span class="sxs-lookup"><span data-stu-id="a22ed-107">If you're using source control such as git, now would be a good time to exclude the `oauth_environment_variables.rb` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="a22ed-108">Abra el **./config/Environment.RB** y agregue el siguiente código antes `Rails.application.initialize!` de la línea.</span><span class="sxs-lookup"><span data-stu-id="a22ed-108">Open **./config/environment.rb** and add the following code before the `Rails.application.initialize!` line.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/environment.rb" id="LoadOAuthSettingsSnippet" highlight="4-6":::

## <a name="setup-omniauth"></a><span data-ttu-id="a22ed-109">Programa de instalación OmniAuth</span><span class="sxs-lookup"><span data-stu-id="a22ed-109">Setup OmniAuth</span></span>

<span data-ttu-id="a22ed-110">Ya ha instalado la `omniauth-oauth2` gema, pero para que funcione con los puntos de conexión de Azure OAuth, debe [crear una estrategia de OAuth2](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span><span class="sxs-lookup"><span data-stu-id="a22ed-110">You've already installed the `omniauth-oauth2` gem, but in order to make it work with the Azure OAuth endpoints, you need to [create an OAuth2 strategy](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span></span> <span data-ttu-id="a22ed-111">Se trata de una clase Ruby que define los parámetros para realizar solicitudes de OAuth en el proveedor de Azure.</span><span class="sxs-lookup"><span data-stu-id="a22ed-111">This is a Ruby class that defines the parameters for making OAuth requests to the Azure provider.</span></span>

1. <span data-ttu-id="a22ed-112">Cree un nuevo archivo llamado `microsoft_graph_auth.rb` en la carpeta **./lib**' \* \* y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="a22ed-112">Create a new file called `microsoft_graph_auth.rb` in the **./lib**\`\*\* folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/lib/microsoft_graph_auth.rb" id="AuthStrategySnippet":::

    <span data-ttu-id="a22ed-113">Tómese un momento para revisar lo que hace el código.</span><span class="sxs-lookup"><span data-stu-id="a22ed-113">Take a moment to review what this code does.</span></span>

    - <span data-ttu-id="a22ed-114">Establece el `client_options` para especificar los puntos de conexión de la plataforma de identidad de Microsoft.</span><span class="sxs-lookup"><span data-stu-id="a22ed-114">It sets the `client_options` to specify the Microsoft identity platform endpoints.</span></span>
    - <span data-ttu-id="a22ed-115">Especifica que el `scope` parámetro debe enviarse durante la fase de autorización.</span><span class="sxs-lookup"><span data-stu-id="a22ed-115">It specifies that the `scope` parameter should be sent during the authorize phase.</span></span>
    - <span data-ttu-id="a22ed-116">Asigna la `id` propiedad del usuario como identificador único del usuario.</span><span class="sxs-lookup"><span data-stu-id="a22ed-116">It maps the `id` property of the user as the unique ID for the user.</span></span>
    - <span data-ttu-id="a22ed-117">Usa el token de acceso para recuperar el perfil del usuario de Microsoft Graph para rellenar `raw_info` el hash.</span><span class="sxs-lookup"><span data-stu-id="a22ed-117">It uses the access token to retrieve the user's profile from Microsoft Graph to fill in the `raw_info` hash.</span></span>
    - <span data-ttu-id="a22ed-118">Invalida la dirección URL de devolución de llamada para asegurarse de que coincide con la devolución de llamada registrada en el portal de registro de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="a22ed-118">It overrides the callback URL to ensure that it matches the registered callback in the app registration portal.</span></span>

1. <span data-ttu-id="a22ed-119">Cree un nuevo archivo llamado `omniauth_graph.rb` en la carpeta **./config/Initializers** y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="a22ed-119">Create a new file called `omniauth_graph.rb` in the **./config/initializers** folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/omniauth_graph.rb" id="ConfigureOmniAuthSnippet":::

    <span data-ttu-id="a22ed-120">Este código se ejecutará cuando se inicie la aplicación.</span><span class="sxs-lookup"><span data-stu-id="a22ed-120">This code will execute when the app starts.</span></span> <span data-ttu-id="a22ed-121">Carga el middleware OmniAuth con el `microsoft_graph_auth` proveedor, configurado con las variables de entorno establecidas en **oauth_environment_variables. RB**.</span><span class="sxs-lookup"><span data-stu-id="a22ed-121">It loads up the OmniAuth middleware with the `microsoft_graph_auth` provider, configured with the environment variables set in **oauth_environment_variables.rb**.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="a22ed-122">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="a22ed-122">Implement sign-in</span></span>

<span data-ttu-id="a22ed-123">Ahora que el middleware de OmniAuth está configurado, puede pasar a agregar el inicio de sesión a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="a22ed-123">Now that the OmniAuth middleware is configured, you can move on to adding sign-in to the app.</span></span>

1. <span data-ttu-id="a22ed-124">Ejecute el siguiente comando en su CLI para generar un controlador de inicio y cierre de sesión.</span><span class="sxs-lookup"><span data-stu-id="a22ed-124">Run the following command in your CLI to generate a controller for sign-in and sign-out.</span></span>

    ```Shell
    rails generate controller Auth
    ```

1. <span data-ttu-id="a22ed-125">Abra **./app/controllers/auth_controller. RB**.</span><span class="sxs-lookup"><span data-stu-id="a22ed-125">Open **./app/controllers/auth_controller.rb**.</span></span> <span data-ttu-id="a22ed-126">Agregue un método de devolución de `AuthController` llamada a la clase.</span><span class="sxs-lookup"><span data-stu-id="a22ed-126">Add a callback method to the `AuthController` class.</span></span> <span data-ttu-id="a22ed-127">Este método será llamado por el middleware OmniAuth una vez que se haya completado el flujo de OAuth.</span><span class="sxs-lookup"><span data-stu-id="a22ed-127">This method will be called by the OmniAuth middleware once the OAuth flow is complete.</span></span>

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      data = request.env['omniauth.auth']

      # Temporary for testing!
      render json: data.to_json
    end
    ```

    <span data-ttu-id="a22ed-128">Por ahora todo esto es representar el hash proporcionado por OmniAuth.</span><span class="sxs-lookup"><span data-stu-id="a22ed-128">For now all this does is render the hash provided by OmniAuth.</span></span> <span data-ttu-id="a22ed-129">Usará esto para comprobar que el inicio de sesión está funcionando antes de continuar.</span><span class="sxs-lookup"><span data-stu-id="a22ed-129">You'll use this to verify that sign-in is working before moving on.</span></span>

1. <span data-ttu-id="a22ed-130">Agregue las rutas a **./config/Routes.RB**.</span><span class="sxs-lookup"><span data-stu-id="a22ed-130">Add the routes to **./config/routes.rb**.</span></span>

    ```ruby
    # Add route for OmniAuth callback
    match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
    ```

1. <span data-ttu-id="a22ed-131">Inicie el servidor y vaya a `https://localhost:3000`.</span><span class="sxs-lookup"><span data-stu-id="a22ed-131">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="a22ed-132">Haga clic en el botón de inicio de sesión y se le `https://login.microsoftonline.com`redirigirá a.</span><span class="sxs-lookup"><span data-stu-id="a22ed-132">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="a22ed-133">Inicie sesión con su cuenta de Microsoft y dé su consentimiento a los permisos solicitados.</span><span class="sxs-lookup"><span data-stu-id="a22ed-133">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="a22ed-134">El explorador redirige a la aplicación, que muestra el hash generado por OmniAuth.</span><span class="sxs-lookup"><span data-stu-id="a22ed-134">The browser redirects to the app, showing the hash generated by OmniAuth.</span></span>

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

## <a name="storing-the-tokens"></a><span data-ttu-id="a22ed-135">Almacenamiento de tokens</span><span class="sxs-lookup"><span data-stu-id="a22ed-135">Storing the tokens</span></span>

<span data-ttu-id="a22ed-136">Ahora que puede obtener tokens, es el momento de implementar una forma de almacenarlos en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="a22ed-136">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="a22ed-137">Dado que se trata de una aplicación de ejemplo, por razones de simplicidad, se almacenará en la sesión.</span><span class="sxs-lookup"><span data-stu-id="a22ed-137">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="a22ed-138">Una aplicación real usaría una solución de almacenamiento seguro más confiable, como una base de datos.</span><span class="sxs-lookup"><span data-stu-id="a22ed-138">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="a22ed-139">Abra **./app/controllers/application_controller. RB**.</span><span class="sxs-lookup"><span data-stu-id="a22ed-139">Open **./app/controllers/application_controller.rb**.</span></span> <span data-ttu-id="a22ed-140">Agregue el método siguiente a la clase `ApplicationController`.</span><span class="sxs-lookup"><span data-stu-id="a22ed-140">Add the following method to the `ApplicationController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="SaveInSessionSnippet":::

    <span data-ttu-id="a22ed-141">El método toma el hash OmniAuth como un parámetro y extrae los bits relevantes de información y, a continuación, lo almacena en la sesión.</span><span class="sxs-lookup"><span data-stu-id="a22ed-141">The method takes the OmniAuth hash as a parameter and extracts the relevant bits of information, then stores that in the session.</span></span>

1. <span data-ttu-id="a22ed-142">Agregue funciones de descriptor de acceso a la `ApplicationController` clase para recuperar el nombre de usuario, la dirección de correo electrónico y el token de acceso de la sesión.</span><span class="sxs-lookup"><span data-stu-id="a22ed-142">Add accessor functions to the `ApplicationController` class to retrieve the user name, email address, and access token back out of the session.</span></span>

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

1. <span data-ttu-id="a22ed-143">Agregue el siguiente código a la `ApplicationController` clase que se ejecutará antes de que se procese ninguna acción.</span><span class="sxs-lookup"><span data-stu-id="a22ed-143">Add the following code to the `ApplicationController` class that will run before any action is processed.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="BeforeActionSnippet":::

    <span data-ttu-id="a22ed-144">Este método establece las variables que el diseño (en **Application. html. Erb**) utiliza para mostrar la información del usuario en la barra de navegación.</span><span class="sxs-lookup"><span data-stu-id="a22ed-144">This method sets the variables that the layout (in **application.html.erb**) uses to show the user's information in the nav bar.</span></span> <span data-ttu-id="a22ed-145">Si lo agrega aquí, no tendrá que agregar este código en todas las acciones de controlador.</span><span class="sxs-lookup"><span data-stu-id="a22ed-145">By adding it here, you don't have to add this code in every single controller action.</span></span> <span data-ttu-id="a22ed-146">Sin embargo, también se ejecutará para las acciones `AuthController`del, lo que no es óptimo.</span><span class="sxs-lookup"><span data-stu-id="a22ed-146">However, this will also run for actions in the `AuthController`, which isn't optimal.</span></span>

1. <span data-ttu-id="a22ed-147">Agregue el siguiente código a la `AuthController` clase en **./app/Controllers/auth_controller. RB** para omitir la acción anterior.</span><span class="sxs-lookup"><span data-stu-id="a22ed-147">Add the following code to the `AuthController` class in **./app/controllers/auth_controller.rb** to skip the before action.</span></span>

    ```ruby
    skip_before_action :set_user
    ```

1. <span data-ttu-id="a22ed-148">Actualice la `callback` función de la `AuthController` clase para almacenar los tokens en la sesión y volver a redirigir a la Página principal.</span><span class="sxs-lookup"><span data-stu-id="a22ed-148">Update the `callback` function in the `AuthController` class to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="a22ed-149">Reemplace la función `callback` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="a22ed-149">Replace the existing `callback` function with the following.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="CallbackSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="a22ed-150">Implementación de cierre de sesión</span><span class="sxs-lookup"><span data-stu-id="a22ed-150">Implement sign-out</span></span>

<span data-ttu-id="a22ed-151">Antes de probar esta nueva característica, agregue una forma de cerrar sesión.</span><span class="sxs-lookup"><span data-stu-id="a22ed-151">Before you test this new feature, add a way to sign out.</span></span>

1. <span data-ttu-id="a22ed-152">Agregue la siguiente acción a la `AuthController` clase.</span><span class="sxs-lookup"><span data-stu-id="a22ed-152">Add the following action to the `AuthController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="SignOutSnippet":::

1. <span data-ttu-id="a22ed-153">Agregue esta acción a **./config/Routes.RB**.</span><span class="sxs-lookup"><span data-stu-id="a22ed-153">Add this action to **./config/routes.rb**.</span></span>

    ```ruby
    get 'auth/signout'
    ```

1. <span data-ttu-id="a22ed-154">Reinicie el servidor y pase por el proceso de inicio de sesión.</span><span class="sxs-lookup"><span data-stu-id="a22ed-154">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="a22ed-155">Deberás volver a la Página principal, pero la interfaz de usuario debe cambiar para indicar que has iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="a22ed-155">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Una captura de pantalla de la Página principal después de iniciar sesión](./images/add-aad-auth-01.png)

1. <span data-ttu-id="a22ed-157">Haga clic en el avatar de usuario en la esquina superior derecha para acceder al vínculo **Cerrar sesión** .</span><span class="sxs-lookup"><span data-stu-id="a22ed-157">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="a22ed-158">Al hacer clic en **cerrar** sesión se restablece la sesión y se vuelve a la Página principal.</span><span class="sxs-lookup"><span data-stu-id="a22ed-158">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Captura de pantalla del menú desplegable con el vínculo cerrar sesión](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="a22ed-160">Actualizar tokens</span><span class="sxs-lookup"><span data-stu-id="a22ed-160">Refreshing tokens</span></span>

<span data-ttu-id="a22ed-161">Si observa atentamente el hash generado por OmniAuth, verá que hay dos tokens en el hash: `token` y. `refresh_token`</span><span class="sxs-lookup"><span data-stu-id="a22ed-161">If you look closely at the hash generated by OmniAuth, you'll notice there are two tokens in the hash: `token` and `refresh_token`.</span></span> <span data-ttu-id="a22ed-162">El valor en `token` es el token de acceso, que se envía en `Authorization` el encabezado de las llamadas a la API.</span><span class="sxs-lookup"><span data-stu-id="a22ed-162">The value in `token` is the access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="a22ed-163">Este es el token que permite que la aplicación tenga acceso a Microsoft Graph en nombre del usuario.</span><span class="sxs-lookup"><span data-stu-id="a22ed-163">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="a22ed-164">Sin embargo, este token es de corta duración.</span><span class="sxs-lookup"><span data-stu-id="a22ed-164">However, this token is short-lived.</span></span> <span data-ttu-id="a22ed-165">El token expira una hora después de su emisión.</span><span class="sxs-lookup"><span data-stu-id="a22ed-165">The token expires an hour after it is issued.</span></span> <span data-ttu-id="a22ed-166">Aquí es donde el `refresh_token` valor se vuelve útil.</span><span class="sxs-lookup"><span data-stu-id="a22ed-166">This is where the `refresh_token` value becomes useful.</span></span> <span data-ttu-id="a22ed-167">El token de actualización permite que la aplicación solicite un nuevo token de acceso sin que el usuario tenga que iniciar sesión de nuevo.</span><span class="sxs-lookup"><span data-stu-id="a22ed-167">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="a22ed-168">Actualice el código de administración de tokens para implementar la actualización de tokens.</span><span class="sxs-lookup"><span data-stu-id="a22ed-168">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="a22ed-169">Abra **./app/controllers/application_controller. RB** y agregue las siguientes `require` instrucciones en la parte superior:</span><span class="sxs-lookup"><span data-stu-id="a22ed-169">Open **./app/controllers/application_controller.rb** and add the following `require` statements at the top:</span></span>

    ```ruby
    require 'microsoft_graph_auth'
    require 'oauth2'
    ```

1. <span data-ttu-id="a22ed-170">Agregue el método siguiente a la clase `ApplicationController`.</span><span class="sxs-lookup"><span data-stu-id="a22ed-170">Add the following method to the `ApplicationController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="RefreshTokensSnippet":::

    <span data-ttu-id="a22ed-171">Este método usa la gema de [OAuth2](https://github.com/oauth-xx/oauth2) (una dependencia de `omniauth-oauth2` la gema) para actualizar los tokens y actualiza la sesión.</span><span class="sxs-lookup"><span data-stu-id="a22ed-171">This method uses the [oauth2](https://github.com/oauth-xx/oauth2) gem (a dependency of the `omniauth-oauth2` gem) to refresh the tokens, and updates the session.</span></span>

1. <span data-ttu-id="a22ed-172">Reemplace el método `access_token` actual por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="a22ed-172">Replace the current `access_token` method with the following.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="AccessTokenSnippet":::

    <span data-ttu-id="a22ed-173">En lugar de simplemente devolver el token de la sesión, se comprobará primero si está cerca de la fecha de expiración.</span><span class="sxs-lookup"><span data-stu-id="a22ed-173">Instead of just returning the token from the session, it will first check if it is close to expiration.</span></span> <span data-ttu-id="a22ed-174">Si es así, se actualizará antes de devolver el token.</span><span class="sxs-lookup"><span data-stu-id="a22ed-174">If it is, then it will refresh before returning the token.</span></span>
