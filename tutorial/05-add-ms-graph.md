<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, incorporará Microsoft Graph a la aplicación. Para esta aplicación, usará la GEM [httparty](https://github.com/jnunemaker/httparty) para realizar llamadas a Microsoft Graph.

## <a name="create-a-graph-helper"></a>Crear una aplicación auxiliar de gráfico

Cree una aplicación auxiliar para administrar todas las llamadas a la API. Ejecute el siguiente comando en su CLI para generar la aplicación auxiliar.

```Shell
rails generate helper Graph
```

Abra el archivo que `./app/helpers/graph_helper.rb` acaba de crear y reemplace el contenido por lo siguiente.

```ruby
require 'httparty'

# Graph API helper methods
module GraphHelper
  GRAPH_HOST = 'https://graph.microsoft.com'.freeze

  def make_api_call(endpoint, token, params = nil)
    headers = {
      Authorization: "Bearer #{token}"
    }

    query = params || {}

    HTTParty.get "#{GRAPH_HOST}#{endpoint}",
                 headers: headers,
                 query: query
  end
end
```

Tómese un momento para revisar lo que hace el código. Realiza una sencilla solicitud GET a través del `httparty` GEM al punto de conexión solicitado. Envía el token de acceso en el `Authorization` encabezado e incluye los parámetros de consulta que se pasan.

Por ejemplo, para usar el `make_api_call` método para realizar una llamada a `https://graph.microsoft.com/v1.0/me?$select=displayName`get, puede llamarlo de esta manera:

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

A partir de entonces, podrá crear más adelante a medida que implementa más características de Microsoft Graph en la aplicación.

## <a name="get-calendar-events-from-outlook"></a>Obtener eventos de calendario de Outlook

Comencemos agregando la posibilidad de ver los eventos en el calendario del usuario. En la CLI, ejecute el siguiente comando para agregar un nuevo controlador.

```Shell
rails generate controller Calendar index
```

Ahora que la ruta está disponible, actualice el vínculo de **calendario** de la barra de `./app/view/layouts/application.html.erb` exploración en para usarla. Reemplace la línea `<a class="nav-link" href="#">Calendar</a>` por lo siguiente.

```html
<%= link_to "Calendar", {:controller => :calendar, :action => :index}, class: "nav-link#{' active' if controller.controller_name == 'calendar'}" %>
```

Agregue un nuevo método a la aplicación auxiliar de Graph para [Mostrar los eventos del usuario](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_list_events). Abra `./app/helpers/graph_helper.rb` y agregue el siguiente método al `GraphHelper` módulo.

```ruby
def get_calendar_events(token)
  get_events_url = '/v1.0/me/events'

  query = {
    '$select': 'subject,organizer,start,end',
    '$orderby': 'createdDateTime DESC'
  }

  response = make_api_call get_events_url, token, query

  raise response.parsed_response.to_s || "Request returned #{response.code}" unless response.code == 200
  response.parsed_response['value']
end
```

Tenga en cuenta lo que está haciendo este código.

- La dirección URL a la que se `/v1.0/me/events`llamará es.
- El `$select` parámetro limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.
- El `$orderby` parámetro ordena los resultados por la fecha y hora en que se crearon, con el elemento más reciente en primer lugar.
- Para una respuesta correcta, devuelve la matriz de elementos contenidos en la `value` clave.

Ahora puede probar esto. Abra `./app/controllers/calendar_controller.rb` y actualice la `index` acción para llamar a este método y representar los resultados.

```ruby
# Calendar controller
class CalendarController < ApplicationController
  include GraphHelper

  def index
    @events = get_calendar_events access_token || []
    render json: @events
  rescue RuntimeError => e
    @errors = [
      {
        message: 'Microsoft Graph returned an error getting events.',
        debug: e
      }
    ]
  end
end
```

Reinicie el servidor. Inicie sesión y haga clic en el vínculo de **calendario** en la barra de navegación. Si todo funciona, debería ver un volcado JSON de eventos en el calendario del usuario.

## <a name="display-the-results"></a>Mostrar los resultados

Ahora puede Agregar HTML y CSS para mostrar los resultados de forma más fácil de uso.

Abra `./app/views/calendar/index.html.erb` y reemplace su contenido por lo siguiente.

```html
<h1>Calendar</h1>
<table class="table">
  <thead>
    <tr>
      <th scope="col">Organizer</th>
      <th scope="col">Subject</th>
      <th scope="col">Start</th>
      <th scope="col">End</th>
    </tr>
  </thead>
  <tbody>
    <% @events.each do |event| %>
      <tr>
        <td><%= event['organizer']['emailAddress']['name'] %></td>
        <td><%= event['subject'] %></td>
        <td><%= event['start']['dateTime'].to_time(:utc).localtime.strftime('%-m/%-d/%y %l:%M %p') %></td>
        <td><%= event['end']['dateTime'].to_time(:utc).localtime.strftime('%-m/%-d/%y %l:%M %p') %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

Se recorrerá en bucle una colección de eventos y se agregará una fila de tabla para cada uno. Quite la `render json: @events` línea de la `index` acción en `./app/controllers/calendar_controller.rb` y ahora la aplicación debe representar una tabla de eventos.

![Captura de pantalla de la tabla de eventos](./images/add-msgraph-01.png)
