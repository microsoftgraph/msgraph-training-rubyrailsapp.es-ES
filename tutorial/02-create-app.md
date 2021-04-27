<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, [usarás Ruby on Rails](https://rubyonrails.org/) para crear una aplicación web.

1. Si aún no tiene rails instalado, puede instalarlo desde la interfaz de línea de comandos (CLI) con el siguiente comando.

    ```Shell
    gem install rails -v 6.1.3.1
    ```

1. Abre la CLI, navega a un directorio donde tienes derechos para crear archivos y ejecuta el siguiente comando para crear una nueva aplicación Rails.

    ```Shell
    rails new graph-tutorial
    ```

1. Navegue a este nuevo directorio y escriba el siguiente comando para iniciar un servidor web local.

    ```Shell
    rails server
    ```

1. Abra el explorador y vaya a `http://localhost:3000`. Si todo funciona, verá un "Yay! You're on Rails!" Mensaje. Si no ve ese mensaje, consulte la guía de introducción a [Rails](http://guides.rubyonrails.org/).

## <a name="install-gems"></a>Instalar gemas

Antes de seguir adelante, instala algunas gemas adicionales que usarás más adelante:

- [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) para controlar los flujos de tokens de inicio de sesión y OAuth.
- [omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) para agregar protección CSRF a OmniAuth.
- [httparty](https://github.com/jnunemaker/httparty) para realizar llamadas a Microsoft Graph.
- [activerecord-session_store](https://github.com/rails/activerecord-session_store) para almacenar sesiones en la base de datos.

1. Abra **./Gemfile** y agregue las siguientes líneas.

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. En la CLI, ejecute el siguiente comando.

    ```Shell
    bundle install
    ```

1. En la CLI, ejecute los siguientes comandos para configurar la base de datos para almacenar sesiones.

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. Cree un nuevo archivo llamado `session_store.rb` en el directorio **./config/initializers** y agregue el siguiente código.

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a>Diseñar la aplicación

En esta sección, crearás la interfaz de usuario básica para la aplicación.

1. Abra **./app/views/layouts/application.html.erb** y reemplace su contenido por lo siguiente.

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    Este código agrega [Bootstrap para](http://getbootstrap.com/) un estilo sencillo y [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) para algunos iconos sencillos. También define un diseño global con una barra de navegación.

1. Abra **./app/assets/stylesheets/application.css** y agregue lo siguiente al final del archivo.

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. Genere un controlador de página principal con el siguiente comando.

    ```Shell
    rails generate controller Home index
    ```

1. Configura la `index` acción en el controlador como la página predeterminada de la `Home` aplicación. Abra **./config/routes.rb y** reemplace su contenido por el siguiente

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. Abra **./app/view/home/index.html.erb** y reemplace su contenido por lo siguiente.

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. Agregue un archivo PNG denominado **no-profile-photo.png** en el directorio **./app/assets/images.**

1. Guarde todos los cambios y reinicie el servidor. Ahora, la aplicación debe tener un aspecto muy diferente.

    ![Una captura de pantalla de la página de inicio rediseñada](./images/create-app-01.png)
