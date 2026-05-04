🎛️ Handlers
==================

.. currentmodule:: pywa.handlers

Para processar atualizações do WhatsApp, sua aplicação precisa receber **webhooks recebidos**.
Isso é feito executando um servidor web que ouve as requisições do WhatsApp e as passa para seus handlers.

**Por que o pywa Não Inicia o Servidor?**

O ``pywa`` foi projetado para **máxima flexibilidade** — ele não executa o servidor por você.
Em vez disso, ele apenas registra a rota que irá tratar as atualizações recebidas.

Isso significa que você pode:

- Usar qualquer framework web que preferir.
- Configurar seu servidor da forma que quiser.
- Servir outras partes da sua aplicação junto com o ``pywa`` sem restrições.

.. note::

    O Pywa tem suporte integrado para FastAPI e Flask, mas você pode usar qualquer framework que suporte o tratamento de requisições HTTP.

Configurar uma URL de Callback
------------------------------
Para que o WhatsApp envie atualizações para o seu servidor, você deve fornecer uma **URL de callback** — um endpoint público e seguro (``HTTPS``) que aponta para o seu servidor em execução.

Se você está desenvolvendo localmente, pode usar serviços de tunelamento como:

- `ngrok <https://ngrok.com/>`_
- `Cloudflare Tunnel <https://developers.cloudflare.com/pages/how-to/preview-with-cloudflare-tunnel/>`_
- `localtunnel <https://localtunnel.github.io/www/>`_

Eles criam uma URL pública e segura que encaminha o tráfego para sua máquina local.

Exemplo usando ngrok:

.. code-block:: bash
    :caption: Terminal

    ngrok http 8080

Assim que tiver uma URL pública, você deve **registrá-la** no WhatsApp — automaticamente (via pywa) ou manualmente (via o Painel do Aplicativo WhatsApp).

Opção 1: Registro Automático de URL de Callback
-----------------------------------------------
Este é o método mais simples — o ``pywa`` irá:

- Registrar sua URL de callback no WhatsApp.
- Tratar a requisição de verificação para você.

**Requisitos:**

- O **ID** e o **Secret** do seu Aplicativo WhatsApp (a menos que você esteja definindo callback_url_scope para ``PHONE`` ou ``WABA``).
  Consulte a `documentação do Facebook <https://developers.facebook.com/docs/development/create-an-app/app-dashboard/basic-settings/>`_ para saber como obtê-los.

Exemplo usando FastAPI:

.. code-block:: python
    :caption: main.py
    :linenos:
    :emphasize-lines: 4, 9, 10, 11, 12, 13

    import fastapi
    from pywa import WhatsApp

    fastapi_app = fastapi.FastAPI()

    wa = WhatsApp(
        phone_id='1234567890',
        token='xxxxxx',
        server=fastapi_app,
        callback_url='https://subdomain.ngrok.io',  # Your public URL
        verify_token='XYZ123',
        app_id=123456,
        app_secret='xxxxxx'
    )

    # Register your handlers here

Execute o servidor:

.. code-block:: bash
    :caption: Terminal

    fastapi dev main.py --port 8080

.. note::

    A porta deve corresponder à que você expõe via seu túnel.
    Exemplo: ``ngrok http 8080`` significa que seu servidor deve rodar na porta 8080.

Opção 2: Registro Manual de URL de Callback
-------------------------------------------
Se você preferir registrar a URL de callback você mesmo:

1. Inicie seu servidor para que o ``pywa`` possa tratar a requisição de verificação do WhatsApp.
2. Acesse **App Dashboard > WhatsApp > Configuration**.

.. image:: ../../../../_static/guides/register-callback-url.webp
    :alt: Register Callback URL

3. Informe:

   - A URL pública do seu servidor (por exemplo, ``https://subdomain.ngrok.io``).
   - O ``verify_token`` que você usou na inicialização do seu client ``WhatsApp``.

Exemplo usando FastAPI:

.. code-block:: python
    :caption: main.py
    :linenos:
    :emphasize-lines: 4, 9, 10

    import fastapi
    from pywa import WhatsApp

    fastapi_app = fastapi.FastAPI()

    wa = WhatsApp(
        phone_id='1234567890',
        token='xxxxxx',
        server=fastapi_app,
        verify_token='XYZ123',
    )

    # Register your handlers here

Execute o servidor:

.. code-block:: bash
    :caption: Terminal

    fastapi dev main.py --port 8080

Assinar Campos de Webhook
--------------------------
Ao registrar manualmente, você também deve assinar os campos de webhook nas configurações do seu aplicativo.

Acesse **App Dashboard > WhatsApp > Configuration** e role para baixo até a seção **Webhook Fields**.

.. image:: ../../../../_static/guides/subscribe-webhook-fields.webp
    :alt: Subscribe to Webhook Fields

Suportados pelo pywa:

- ``messages`` – todas as atualizações relacionadas ao usuário (mensagens, callbacks, status de mensagens)
- ``calls`` – atualizações de conexão, encerramento e status de chamadas
- ``message_template_status_update`` – mudanças de aprovação/rejeição de template
- ``message_template_quality_update`` – mudanças na pontuação de qualidade do template
- ``message_template_components_update`` – mudanças nos componentes do template (cabeçalho, corpo, rodapé, botões)
- ``template_category_update`` – mudanças de categoria do template
- ``user_preferences`` – preferências de marketing do usuário

Você também pode assinar outros campos, mas eles não serão processados automaticamente — use :meth:`~pywa.client.WhatsApp.on_raw_update` para tratá-los.

Assim que tudo estiver configurado corretamente, o WhatsApp começará a enviar atualizações para a sua URL de webhook.

--------------------------

Registrar Funções de Callback
------------------------------

Para tratar atualizações recebidas, você deve **registrar funções de callback**.
Essas funções são chamadas sempre que o WhatsApp envia uma atualização.

Uma função de callback deve aceitar dois argumentos:

- O objeto client do WhatsApp (:class:`~pywa.client.WhatsApp`)
- O objeto de atualização (:class:`~pywa.types.Message`, :class:`~pywa.types.CallbackButton`, etc.)

**Exemplo:**

.. code-block:: python
    :emphasize-lines: 3, 6

    from pywa import WhatsApp, types

    def echo_ok(client: WhatsApp, msg: types.Message):
        msg.reply("Ok")

    def react_to_button(client: WhatsApp, clb: types.CallbackButton):
        clb.react("❤️")

Uma vez definidas, você pode registrar callbacks de duas formas principais:



Usando decoradores
^^^^^^^^^^^^^^^^^^

A abordagem mais simples é com os decoradores ``on_...`` como :meth:`~pywa.client.WhatsApp.on_message`, :meth:`~pywa.client.WhatsApp.on_callback_button` etc.

.. code-block:: python
    :caption: main.py
    :linenos:
    :emphasize-lines: 7, 11

    from pywa import WhatsApp, types
    from fastapi import FastAPI

    fastapi_app = FastAPI()
    wa = WhatsApp(..., server=fastapi_app)

    @wa.on_message
    def handle_message(client: WhatsApp, msg: types.Message):
        print(msg)

    @wa.on_callback_button
    def handle_callback_button(client: WhatsApp, clb: types.CallbackButton):
        print(clb.data)

.. code-block:: bash
    :caption: Terminal

    fastapi dev main.py

.. tip::

    Se você não tem acesso à instância do client (por exemplo, em um módulo onde você define handlers), pode registrar handlers **diretamente na classe WhatsApp**.

    Exemplo:

    .. code-block:: python
        :caption: my_handlers.py
        :linenos:
        :emphasize-lines: 4

        from pywa import WhatsApp, types
        from fastapi import FastAPI

        @WhatsApp.on_message  # Register with the class itself
        def handle_message(client: WhatsApp, msg: types.Message):
            print(msg)

    Depois carregue os handlers no seu arquivo principal:

    .. code-block:: python
        :caption: main.py
        :linenos:
        :emphasize-lines: 4, 8

        from pywa import WhatsApp
        import my_handlers  # Import the module with your handlers

        wa = WhatsApp(..., handlers_modules=[my_handlers])

        # Or dynamically:
        wa = WhatsApp(...)
        wa.load_handlers_modules(my_handlers)


Usando objetos ``Handler``
^^^^^^^^^^^^^^^^^^^^^^^^^^

Para projetos maiores, ou quando precisa registrar handlers dinamicamente, você pode encapsular funções de callback em objetos ``Handler`` e adicioná-los via :meth:`~pywa.client.WhatsApp.add_handlers`.

**Exemplo:**

.. code-block:: python
    :caption: my_handlers.py
    :linenos:

    from pywa import WhatsApp, types

    def handle_message(client: WhatsApp, msg: types.Message):
        print(msg.text)

    def handle_callback_button(client: WhatsApp, clb: types.CallbackButton):
        print(clb.data)

.. code-block:: python
    :caption: main.py
    :linenos:
    :emphasize-lines: 3, 9-10

    from pywa import WhatsApp, handlers
    from fastapi import FastAPI
    import my_handlers  # Import the module with your handlers

    fastapi_app = FastAPI()
    wa = WhatsApp(..., server=fastapi_app)

    wa.add_handlers(
        handlers.MessageHandler(callback=my_handlers.handle_message),
        handlers.CallbackButtonHandler(callback=my_handlers.handle_callback_button),
    )

.. code-block:: bash
    :caption: Terminal

    fastapi dev main.py


Handlers Disponíveis
--------------------

.. list-table::
   :widths: 20 20 60
   :header-rows: 1

   * - Decorador
     - Handler
     - Tipo de atualização
   * - :meth:`~pywa.client.WhatsApp.on_message`
     - :class:`MessageHandler`
     - :class:`~pywa.types.message.Message`
   * - :meth:`~pywa.client.WhatsApp.on_callback_button`
     - :class:`CallbackButtonHandler`
     - :class:`~pywa.types.callback.CallbackButton`
   * - :meth:`~pywa.client.WhatsApp.on_callback_selection`
     - :class:`CallbackSelectionHandler`
     - :class:`~pywa.types.callback.CallbackSelection`
   * - :meth:`~pywa.client.WhatsApp.on_flow_completion`
     - :class:`FlowCompletionHandler`
     - :class:`~pywa.types.flows.FlowCompletion`
   * - :meth:`~pywa.client.WhatsApp.on_flow_request`
     - :class:`FlowRequestHandler`
     - :class:`~pywa.types.flows.FlowRequest`
   * - :meth:`~pywa.client.WhatsApp.on_message_status`
     - :class:`MessageStatusHandler`
     - :class:`~pywa.types.message_status.MessageStatus`
   * - :meth:`~pywa.client.WhatsApp.on_template_status_update`
     - :class:`TemplateStatusUpdateHandler`
     - :class:`~pywa.types.templates.TemplateStatusUpdate`
   * - :meth:`~pywa.client.WhatsApp.on_template_category_update`
     - :class:`TemplateCategoryUpdateHandler`
     - :class:`~pywa.types.templates.TemplateCategoryUpdate`
   * - :meth:`~pywa.client.WhatsApp.on_template_quality_update`
     - :class:`TemplateQualityUpdateHandler`
     - :class:`~pywa.types.templates.TemplateQualityUpdate`
   * - :meth:`~pywa.client.WhatsApp.on_template_components_update`
     - :class:`TemplateComponentsUpdateHandler`
     - :class:`~pywa.types.templates.TemplateComponentsUpdate`
   * - :meth:`~pywa.client.WhatsApp.on_chat_opened`
     - :class:`ChatOpenedHandler`
     - :class:`~pywa.types.chat_opened.ChatOpened`
   * - :meth:`~pywa.client.WhatsApp.on_phone_number_change`
     - :class:`PhoneNumberChangeHandler`
     - :class:`~pywa.types.system.PhoneNumberChange`
   * - :meth:`~pywa.client.WhatsApp.on_identity_change`
     - :class:`IdentityChangeHandler`
     - :class:`~pywa.types.system.IdentityChange`
   * - :meth:`~pywa.client.WhatsApp.on_call_connect`
     - :class:`CallConnectHandler`
     - :class:`~pywa.types.calls.CallConnect`
   * - :meth:`~pywa.client.WhatsApp.on_call_terminate`
     - :class:`CallTerminateHandler`
     - :class:`~pywa.types.calls.CallTerminate`
   * - :meth:`~pywa.client.WhatsApp.on_call_status`
     - :class:`CallStatusHandler`
     - :class:`~pywa.types.calls.CallStatus`
   * - :meth:`~pywa.client.WhatsApp.on_call_permission_update`
     - :class:`CallPermissionUpdateHandler`
     - :class:`~pywa.types.calls.CallPermissionUpdate`
   * - :meth:`~pywa.client.WhatsApp.on_user_marketing_preferences`
     - :class:`UserMarketingPreferencesHandler`
     - :class:`~pywa.types.user_preferences.UserMarketingPreferences`
   * - :meth:`~pywa.client.WhatsApp.on_raw_update`
     - :class:`RawUpdateHandler`
     - :class:`~pywa.types.base_update.RawUpdate`



Filtrar atualizações
---------------------

Você pode filtrar as atualizações recebidas passando filtros para seus handlers.
Isso é útil quando você só quer reagir a tipos específicos de mensagens.

.. code-block:: python
    :caption: main.py
    :linenos:
    :emphasize-lines: 5

    from pywa import WhatsApp, types, filters

    wa = WhatsApp(...)

    @wa.on_message(filters.text)  # Only handle text messages
    def echo(client: WhatsApp, msg: types.Message):
        msg.reply(text=msg.text)  # msg.text is guaranteed to exist here

.. tip::

    Explore o módulo :mod:`~pywa.filters` para filtros embutidos,
    ou crie os seus próprios. Veja mais no `guia de filtros <../filters/overview.html>`_.


Usando listeners em vez de handlers
------------------------------------

Handlers são melhores para **pontos de entrada** no seu aplicativo (por exemplo, comandos ou cliques em botões).
Quando você precisar coletar entrada adicional do usuário (como idade ou endereço),
você pode usar **listeners** em vez de registrar um novo handler em tempo de execução.

.. code-block:: python
    :caption: main.py
    :linenos:
    :emphasize-lines: 7

    from pywa import WhatsApp, types, filters

    wa = WhatsApp(...)

    @wa.on_message(filters.command("start"))
    def start(_: WhatsApp, msg: types.Message):
        age = msg.reply("Hello! What's your age?").wait_for_reply(filters.text).text
        ...

.. note::

    Leia mais sobre listeners no `guia de listeners <../listeners/overview.html>`_.


Controlar o fluxo de handlers
------------------------------

Por padrão, assim que um handler processa uma atualização, nenhum outro handler é chamado.

.. code-block:: python
    :caption: main.py
    :linenos:
    :emphasize-lines: 8

    from pywa import WhatsApp, types

    wa = WhatsApp(...)

    @wa.on_message
    def handle_message(client: WhatsApp, msg: types.Message):
        print(msg)
        # No further handlers will run

    @wa.on_message
    def handle_message2(client: WhatsApp, msg: types.Message):
        print(msg)

.. tip::

    Os handlers são executados na ordem em que foram registrados, a menos que você defina uma ``priority``.
    Um valor mais alto de ``priority`` significa que o handler é executado antes.

    .. code-block:: python
        :caption: main.py
        :linenos:
        :emphasize-lines: 5, 9

        from pywa import WhatsApp, types

        wa = WhatsApp(...)

        @wa.on_message(priority=1)
        def first(client: WhatsApp, msg: types.Message):
            print("First:", msg)

        @wa.on_message(priority=2)  # Will run before the previous handler
        def second(client: WhatsApp, msg: types.Message):
            print("Second:", msg)

Você pode alterar o comportamento padrão habilitando ``continue_handling`` ao criar o client:

.. code-block:: python
    :caption: main.py
    :linenos:
    :emphasize-lines: 1

    wa = WhatsApp(..., continue_handling=True)

    @wa.on_message
    def handler(client: WhatsApp, msg: types.Message):
        print(msg)
        # The next handler WILL also run

Você também pode decidir por mensagem dentro de um handler usando
:meth:`~pywa.types.base_update.BaseUpdate.stop_handling` ou
:meth:`~pywa.types.base_update.BaseUpdate.continue_handling`:

.. code-block:: python
    :caption: main.py
    :linenos:
    :emphasize-lines: 9, 11

    from pywa import WhatsApp, types, filters

    wa = WhatsApp(...)

    @wa.on_message(filters.text)
    def handle_message(client: WhatsApp, msg: types.Message):
        print(msg)
        if msg.text == "stop":
            msg.stop_handling()       # Stop further handlers
        else:
            msg.continue_handling()   # Allow further handlers


Validar atualizações
--------------------

O WhatsApp `recomenda <https://developers.facebook.com/docs/graph-api/webhooks/getting-started#event-notifications>`_
validar atualizações usando o cabeçalho ``X-Hub-Signature-256``.
Isso garante que a atualização foi realmente enviada pelo WhatsApp.

Para habilitar a validação, passe seu ``app_secret`` ao criar o client:

.. code-block:: python
    :caption: main.py
    :linenos:
    :emphasize-lines: 4, 5

    from pywa import WhatsApp

    wa = WhatsApp(
        validate_updates=True,  # Enabled by default
        app_secret="xxxx",
        ...
    )

Se a assinatura for inválida, o pywa responde automaticamente com
``HTTP 401 Unauthorized``.

Você pode desabilitar a validação definindo ``validate_updates=False``.

.. toctree::
    handler_decorators
    handler_objects
