<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio usará [Ruby on Rails](https://rubyonrails.org/) para crear una aplicación Web. Si aún no tiene guías instaladas, puede instalarla desde la interfaz de línea de comandos (CLI) con el siguiente comando.

```Shell
gem install rails
```

Abra la CLI, navegue hasta un directorio donde tenga derechos para crear archivos y ejecute el siguiente comando para crear una nueva aplicación de guías.

```Shell
rails new graph-tutorial
```

Rails crea un nuevo directorio denominado `graph-tutorial` y scaffolding una aplicación guías. Navegue a este nuevo directorio y escriba el siguiente comando para iniciar un servidor Web local.

```Shell
rails server
```

Abra el explorador y vaya a `http://localhost:3000`. Si todo funciona, verá un error "Yay! ¡ Estás en raíles! " Mensaje. Si no ve ese mensaje, consulte la [Guía de introducción](http://guides.rubyonrails.org/)a los rieles.

Antes de continuar, instale algunas gemas adicionales que usará más adelante:

- [omniauth-OAuth2](https://github.com/omniauth/omniauth-oauth2) para controlar los flujos de tokens de OAuth y inicio de sesión.
- [httparty](https://github.com/jnunemaker/httparty) para realizar llamadas a Microsoft Graph.
- [Nokogiri](https://github.com/sparklemotion/nokogiri) para procesar cuerpos HTML de correo electrónico.
- [ActiveRecord-session_store](https://github.com/rails/activerecord-session_store) para almacenar sesiones en la base de datos.

Ejecute los siguientes comandos en su CLI.

```Shell
bundle add omniauth-oauth2
bundle add httparty
bundle add nokogiri
bundle add activerecord-session_store
rails generate active_record:session_migration
```

El último comando genera un resultado similar al siguiente:

```Shell
create  db/migrate/20180618172216_add_sessions_table.rb
```

Abra el archivo que se ha creado y busque la siguiente línea.

```ruby
class AddSessionsTable < ActiveRecord::Migration
```

Cambie esa línea a la siguiente.

```ruby
class AddSessionsTable < ActiveRecord::Migration[5.2]
```

> [!NOTE]
> Se supone que se usan los carriles 5.2. x. Si usa una versión diferente, reemplace `5.2` por su versión.

Guarde el archivo y ejecute el siguiente comando.

```Shell
rake db:migrate
```

Por último, configure raíles para usar el nuevo almacén de sesión. Cree un nuevo archivo llamado `session_store.rb` en el `./config/initializers` directorio y agregue el siguiente código.

```ruby
Rails.application.config.session_store :active_record_store, key: '_graph_app_session'
```

## <a name="design-the-app"></a>Diseñar la aplicación

Para empezar, actualice el diseño global de la aplicación. Abra `./app/views/layouts/application.html.erb` y reemplace su contenido por lo siguiente.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Ruby Graph Tutorial</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css"
      integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.1.0/css/all.css"
      integrity="sha384-lKuwvrZot6UHsBSfcMvOkWwlCMgc0TaWr+30HWe3a4ltaBwTZhyTEggF5tJv8tbt" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js"
      integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js"
      integrity="sha384-smHYKdLADwkXOn1EmN1qk/HfnUcbVRZyYmZ4qpPea6sjB/pTJ0euyQp0Mk8ck+5T" crossorigin="anonymous"></script>
    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <div class="container">
        <%= link_to "Ruby Graph Tutorial", root_path, class: "navbar-brand" %>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse"
          aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav mr-auto">
            <li class="nav-item">
              <%= link_to "Home", root_path, class: "nav-link#{' active' if controller.controller_name == 'home'}" %>
            </li>
            <% if @user_name %>
              <li class="nav-item" data-turbolinks="false">
                <a class="nav-link" href="#">Calendar</a>
              </li>
            <% end %>
          </ul>
          <ul class="navbar-nav justify-content-end">
            <li class="nav-item">
              <a class="nav-link" href="https://developer.microsoft.com/graph/docs/concepts/overview" target="_blank">
                <i class="fas fa-external-link-alt mr-1"></i>Docs
              </a>
            </li>
            <% if @user_name %>
              <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" role="button" aria-haspopup="true" aria-expanded="false">
                  <% if @user_avatar %>
                    <img src=<%= @user_avatar %> class="rounded-circle align-self-center mr-2" style="width: 32px;">
                  <% else %>
                    <i class="far fa-user-circle fa-lg rounded-circle align-self-center mr-2" style="width: 32px;"></i>
                  <% end %>
                </a>
                <div class="dropdown-menu dropdown-menu-right">
                  <h5 class="dropdown-item-text mb-0"><%= @user_name %></h5>
                  <p class="dropdown-item-text text-muted mb-0"><%= @user_email %></p>
                  <div class="dropdown-divider"></div>
                  <a href="#" class="dropdown-item">Sign Out</a>
                </div>
              </li>
            <% else %>
              <li class="nav-item">
                <a href="#" class="nav-link">Sign In</a>
              </li>
            <% end %>
          </ul>
        </div>
      </div>
    </nav>
    <main role="main" class="container">
      <% if @errors %>
        <% @errors.each do |error| %>
          <div class="alert alert-danger" role="alert">
            <p class="mb-3"><%= error[:message] %></p>
            <%if error[:debug] %>
              <pre class="alert-pre border bg-light p-2"><code><%= error[:debug] %></code></pre>
            <% end %>
          </div>
        <% end %>
      <% end %>
      <%= yield %>
    </main>
  </body>
</html>
```

Este código agrega un [bootstrap](http://getbootstrap.com/) para los estilos sencillos y la [fuente maravilla](https://fontawesome.com/) para algunos iconos simples. También define un diseño global con una barra de navegación.

Ahora, `./app/assets/stylesheets/application.css` abra y agregue lo siguiente al final del archivo.

```css
body {
  padding-top: 4.5rem;
}

.alert-pre {
  word-wrap: break-word;
  word-break: break-all;
  white-space: pre-wrap;
}
```

Ahora, reemplace la página predeterminada por otra nueva. Genere un controlador de páginas principales con el siguiente comando.

```Shell
rails generate controller Home index
```

A continuación, `index` configure la `Home` acción en el controlador como la página predeterminada para la aplicación. Abra `./config/routes.rb` y reemplace el contenido por el siguiente

```ruby
Rails.application.routes.draw do
  get 'home/index'
  root 'home#index'

  # Add future routes here

end
```

Ahora, Abra `./app/view/home/index.html.erb` el archivo y reemplace el contenido por lo siguiente.

```html
<div class="jumbotron">
  <h1>Ruby Graph Tutorial</h1>
  <p class="lead">This sample app shows how to use the Microsoft Graph API to access Outlook and OneDrive data from Ruby</p>
  <% if @user_name %>
    <h4>Welcome <%= @user_name %>!</h4>
    <p>Use the navigation bar at the top of the page to get started.</p>
  <% else %>
    <a href="#" class="btn btn-primary btn-large">Click here to sign in</a>
  <% end %>
</div>
```

Guarde todos los cambios y reinicie el servidor. Ahora, la aplicación debe tener un aspecto muy diferente.

![Una captura de pantalla de la Página principal rediseñada](./images/create-app-01.png)
