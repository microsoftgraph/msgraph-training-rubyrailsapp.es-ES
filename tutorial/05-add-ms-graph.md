<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, incorporará Microsoft Graph a la aplicación. Para esta aplicación, usará la GEM [httparty](https://github.com/jnunemaker/httparty) para realizar llamadas a Microsoft Graph.

## <a name="create-a-graph-helper"></a>Crear una aplicación auxiliar de gráfico

1. Cree una aplicación auxiliar para administrar todas las llamadas a la API. Ejecute el siguiente comando en su CLI para generar la aplicación auxiliar.

    ```Shell
    rails generate helper Graph
    ```

1. Abra **./app/helpers/graph_helper. RB** y reemplace el contenido por lo siguiente.

    ```ruby
    require 'httparty'

    # Graph API helper methods
    module GraphHelper
      GRAPH_HOST = 'https://graph.microsoft.com'.freeze

      def make_api_call(method, endpoint, token, headers = nil, params = nil, payload = nil)
        headers ||= {}
        headers[:Authorization] = "Bearer #{token}"
        headers[:Accept] = 'application/json'

        params ||= {}

        case method.upcase
        when 'GET'
          HTTParty.get "#{GRAPH_HOST}#{endpoint}",
                       :headers => headers,
                       :query => params
        when 'POST'
          headers['Content-Type'] = 'application/json'
          HTTParty.post "#{GRAPH_HOST}#{endpoint}",
                        :headers => headers,
                        :query => params,
                        :body => payload ? payload.to_json : nil
        else
          raise "HTTP method #{method.upcase} not implemented"
        end
      end
    end
    ```

Tómese un momento para revisar lo que hace el código. Realiza una solicitud GET o POST sencilla a través de la `httparty` GEM para el punto de conexión solicitado. Envía el token de acceso en el `Authorization` encabezado e incluye los parámetros de consulta que se pasan.

Por ejemplo, para usar el `make_api_call` método para realizar una llamada a get `https://graph.microsoft.com/v1.0/me?$select=displayName` , puede llamarlo de esta manera:

```ruby
make_api_call 'GET', '/v1.0/me', access_token, { '$select': 'displayName' }
```

A partir de entonces, podrá crear más adelante a medida que implementa más características de Microsoft Graph en la aplicación.

## <a name="get-calendar-events-from-outlook"></a>Obtener eventos de calendario de Outlook

1. En la CLI, ejecute el siguiente comando para agregar un nuevo controlador.

    ```Shell
    rails generate controller Calendar index new
    ```

1. Agregue la nueva ruta a **./config/Routes.RB**.

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. Agregue un nuevo método a la aplicación auxiliar de Graph para [obtener una vista de calendario](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0). Abra **./app/helpers/graph_helper. RB** y agregue el siguiente método al `GraphHelper` módulo.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    Tenga en cuenta lo que está haciendo este código.

    - La dirección URL a la que se llamará es `/v1.0/me/calendarview` .
        - El `Prefer: outlook.timezone` encabezado hace que las horas de inicio y finalización de los resultados se ajusten a la zona horaria del usuario.
        - Los `startDateTime` `endDateTime` parámetros y establecen el inicio y el final de la vista.
        - El `$select` parámetro limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.
        - El `$orderby` parámetro ordena los resultados por hora de inicio.
        - El `$top` parámetro limita los resultados a 50 eventos.
    - Para una respuesta correcta, devuelve la matriz de elementos contenidos en la `value` clave.

1. Agregue un nuevo método a la aplicación auxiliar de Graph para buscar un [identificador de zona horaria de IANA](https://www.iana.org/time-zones) basado en un nombre de zona horaria de Windows. Esto es necesario porque Microsoft Graph puede devolver zonas horarias como nombres de zona horaria de Windows y la clase **DateTime** Ruby requiere identificadores de zona horaria de IANA.

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. Abra **./app/controllers/calendar_controller. RB** y reemplace todo el contenido por lo siguiente.

    ```ruby
    # Calendar controller
    class CalendarController < ApplicationController
      include GraphHelper

      def index
        # Get the IANA identifier of the user's time zone
        time_zone = get_iana_from_windows(user_timezone)

        # Calculate the start and end of week in the user's time zone
        start_datetime = Date.today.beginning_of_week(:sunday).in_time_zone(time_zone).to_time
        end_datetime = start_datetime.advance(:days => 7)

        @events = get_calendar_view access_token, start_datetime, end_datetime, user_timezone || []
        render json: @events
      rescue RuntimeError => e
        @errors = [
          {
            :message => 'Microsoft Graph returned an error getting events.',
            :debug => e
          }
        ]
      end
    end
    ```

1. Reinicie el servidor. Inicie sesión y haga clic en el vínculo de **calendario** en la barra de navegación. Si todo funciona, debería ver un volcado JSON de eventos en el calendario del usuario.

## <a name="display-the-results"></a>Mostrar los resultados

Ahora puede Agregar HTML para mostrar los resultados de forma más fácil de uso.

1. Abra **./app/views/calendar/index.html. Erb** y reemplace su contenido por lo siguiente.

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    Se recorrerá en bucle una colección de eventos y se agregará una fila de tabla para cada uno.

1. Quite la `render json: @events` línea de la `index` acción en **./app/Controllers/calendar_controller. RB**.

1. Actualice la página y la aplicación ahora debería representar una tabla de eventos.

    ![Captura de pantalla de la tabla de eventos](./images/add-msgraph-01.png)
