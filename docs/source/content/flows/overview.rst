♻️ Flows
=========

.. currentmodule:: pywa.types.flows

PyWa tem suporte nativo a WhatsApp Flows, permitindo criar interações estruturadas com seus usuários.

De `developers.facebook.com <https://developers.facebook.com/docs/whatsapp/flows>`_:

    .. image:: ../../../../_static/guides/flows-new.webp
        :alt: WhatsApp Flows
        :width: 100%

    WhatsApp Flows é uma forma de criar interações estruturadas para mensagens empresariais. Com Flows, as empresas podem definir, configurar e personalizar mensagens com interações ricas que oferecem aos clientes mais estrutura na forma como se comunicam.

    Você pode usar Flows para agendar consultas, navegar por produtos, coletar feedback de clientes, obter novos leads de vendas ou qualquer outra situação em que a comunicação estruturada seja mais natural ou conveniente para seus clientes.

Os Flows são divididos em 4 partes:

- Criando o Flow
- Enviando o Flow
- Processando requisições do Flow e respondendo a elas (somente para flows dinâmicos)
- Obtendo a conclusão do Flow

Criando o Flow
--------------

Primeiro você precisa criar o flow, dar um nome e definir as categorias chamando :meth:`~pywa.client.WhatsApp.create_flow`:
    Você também pode criar os flows usando o `WhatsApp Flow Builder <https://business.facebook.com/wa/manage/flows/>`_.

.. code-block:: python
    :linenos:

    from pywa import WhatsApp
    from pywa.types import FlowCategory

    # WhatsApp Business Account ID (WABA) is required
    wa = WhatsApp(..., business_account_id="1234567890123456")

    created = wa.create_flow(
        name="My New Flow",
        categories=[FlowCategory.CUSTOMER_SUPPORT, FlowCategory.SURVEY]
    )
    print(wa.get_flow(created.id))

    # FlowDetails(id='1234567890123456', name='My New Flow', status=FlowStatus.DRAFT, ...)

Agora você pode começar a construir a estrutura do flow.

.. tip::

    Você também pode fornecer o flow json ao criar o flow passando o argumento ``flow_json`` para :meth:`~pywa.client.WhatsApp.create_flow`, mas aqui tratamos isso separadamente.

    .. code-block:: python
        :linenos:

        created = wa.create_flow(
            name="My New Flow",
            categories=[FlowCategory.CUSTOMER_SUPPORT, FlowCategory.SURVEY],
            flow_json=FlowJSON(...)  # The flow json to create,
            publish=True,  # If you want to publish the flow immediately
        )

Um flow é uma coleção de telas contendo componentes. As telas podem trocar dados entre si e com o seu servidor.

O flow pode ser estático: todas as configurações dos componentes são predefinidas e nenhuma interação é necessária do seu servidor.
Ou pode ser dinâmico: seu servidor pode responder às ações das telas e determinar a próxima tela a exibir (ou fechar o flow) e os dados a fornecer.

Componentes disponíveis
-----------------------

Cada componente no FlowJSON tem uma classe correspondente em :mod:`pywa.types.flows`:

.. list-table::
   :widths: 10 60
   :header-rows: 1

   * - Categoria
     - Tipos
   * - Elementos estáticos
     - :class:`RichText`,
       :class:`TextHeading`,
       :class:`TextSubheading`,
       :class:`TextBody`,
       :class:`TextCaption`,
       :class:`Image`
   * - Coletar dados
     - :class:`Form`,
       :class:`TextInput`,
       :class:`TextArea`,
       :class:`RadioButtonsGroup`,
       :class:`CheckboxGroup`,
       :class:`ChipsSelector`,
       :class:`Dropdown`,
       :class:`OptIn`,
       :class:`DatePicker`,
       :class:`CalendarPicker`,
       :class:`PhotoPicker`,
       :class:`DocumentPicker`
   * - Navegação
     - :class:`EmbeddedLink`,
       :class:`NavigationList`,
       :class:`Footer`
   * - Renderização Condicional de Componentes
     - :class:`If`,
       :class:`Switch`
   * - Ações
     - :class:`DataExchangeAction`,
       :class:`NavigateAction`,
       :class:`CompleteAction`,
       :class:`UpdateDataAction`,
       :class:`OpenURLAction`
   * - Auxiliares
     - :class:`ScreenData`,
       :class:`ScreenDataUpdate`,
       :class:`ScreenDataRef`,
       :class:`ComponentRef`,
       :class:`FlowStr`,
       :class:`Condition`,
       :class:`MathExpression`

==================

**Aqui está um exemplo de flow estático:**

.. code-block:: python
    :caption: newsletter_flow.py
    :linenos:
    :emphasize-lines: 12, 18, 24, 36-38

    from pywa.types.flows import *

    flow = FlowJSON(
        version=7.0,
        screens=[
            Screen(
                id="NEWSLETTER_SUBSCRIPTION",
                title="Subscribe to our Newsletter",
                terminal=True,
                layout=Layout(
                    children=[
                        full_name := TextInput(
                            name="full_name",
                            label="Full Name",
                            input_type=InputType.TEXT,
                            required=True,
                        ),
                        email := TextInput(
                            name="email",
                            label="Email Address",
                            input_type=InputType.EMAIL,
                            required=True,
                        ),
                        is_subscribed := OptIn(
                            name="is_subscribed",
                            label="Subscribe to our newsletter",
                            required=True,
                            on_click_action=OpenURLAction(
                                url="https://pywa.readthedocs.io/",
                            ),
                        ),
                        Footer(
                            label="Subscribe",
                            on_click_action=CompleteAction(
                                payload={
                                    "full_name": full_name.ref,
                                    "email": email.ref,
                                    "is_subscribed": is_subscribed.ref,
                                },
                            ),
                        ),
                    ],
                ),
            )
        ],
    )


Que é o equivalente ao seguinte flow json:

.. toggle::

    .. code-block:: json
        :caption: newsletter_flow.json
        :linenos:

        {
            "version": "7.0",
            "screens": [
                {
                    "id": "NEWSLETTER_SUBSCRIPTION",
                    "title": "Subscribe to our Newsletter",
                    "terminal": true,
                    "layout": {
                        "type": "SingleColumnLayout",
                        "children": [
                            {
                                "type": "TextInput",
                                "name": "full_name",
                                "label": "Full Name",
                                "input-type": "text",
                                "required": true
                            },
                            {
                                "type": "TextInput",
                                "name": "email",
                                "label": "Email Address",
                                "input-type": "email",
                                "required": true
                            },
                            {
                                "type": "OptIn",
                                "name": "is_subscribed",
                                "label": "Subscribe to our newsletter",
                                "required": true,
                                "on-click-action": {
                                    "name": "open_url",
                                    "url": "https://pywa.readthedocs.io/"
                                }
                            },
                            {
                                "type": "Footer",
                                "label": "Subscribe",
                                "on-click-action": {
                                    "name": "complete",
                                    "payload": {
                                        "full_name": "${form.full_name}",
                                        "email": "${form.email}",
                                        "is_subscribed": "${form.is_subscribed}"
                                    }
                                }
                            }
                        ]
                    }
                }
            ]
        }

E é assim que fica no WhatsApp (iOS/Android):

.. figure:: ../../../../_static/guides/simple-newsletter-flow.png
    :align: center

==================

Depois de ter o flow json, você pode atualizar o flow com :meth:`~pywa.client.WhatsApp.update_flow_json`:


.. code-block:: python
    :caption: update_flow.py
    :linenos:
    :emphasize-lines: 10

    from pywa import WhatsApp
    from pywa.types.flows import *

    your_flow_json = FlowJSON(...)  # keep edit your flow

    if __name__ == "__main__":
        wa = WhatsApp(..., business_account_id="1234567890123456") # waba id is required for creating flows
        # created = wa.create_flow(name="Newsletter Flow", categories=[FlowCategory.CONTACT_US])

        res = wa.update_flow_json(flow_id=created.id, flow_json=newsletter_flow)
        if not res: # If the flow was not updated successfully
            print("Validation errors:")
            for error in res.validation_errors:
                print(error)


O argumento ``flow_json`` pode ser :class:`FlowJSON`, um :class:`dict`, uma :class:`str` json, um arquivo json :class:`pathlib.Path` ou um objeto semelhante a arquivo.

Você pode obter os :class:`FlowDetails` do flow com :meth:`~pywa.client.WhatsApp.get_flow`:

.. code-block:: python
    :linenos:

    flow = wa.get_flow(created.id)
    print(flow)

Ou obtendo todos os flows com :meth:`~pywa.client.WhatsApp.get_flows`:

.. code-block:: python
    :linenos:

    flows = wa.get_flows()
    for flow in flows:
        print(flow)


Para testar seu flow, você precisa enviá-lo:

Enviando o Flow
---------------

.. currentmodule:: pywa.types.callback

Um flow é apenas um :class:`FlowButton` anexado a uma mensagem.
Veja como enviar uma mensagem de texto com flow:

.. currentmodule:: pywa.types.flows

.. code-block:: python
    :linenos:
    :emphasize-lines: 9-15

    from pywa import WhatsApp
    from pywa.types import FlowButton

    wa = WhatsApp(...)

    wa.send_message(
        to="1234567890",
        text="Hi, You need to finish your sign up!",
        buttons=FlowButton(
            title="Finish Sign Up", # The button title that will appear on the bottom of the message
            flow_id="1234567890123456",  # The `ewsletter_flow` flow id from above
            mode=FlowStatus.DRAFT, # If the flow is in draft mode, you must specify the mode as `FlowStatus.DRAFT`.
            flow_action_type=FlowActionType.NAVIGATE, # You tell WhatsApp what to do when the user clicks the button.
            flow_action_screen="SIGN_UP", # The screen id to navigate to when the user clicks the button.
        )
    )


Obtendo a mensagem de conclusão do Flow
---------------------------------------

Quando o usuário conclui o flow, você receberá uma requisição no seu webhook com o payload enviado ao concluir o flow.

Veja como ouvir a atualização de conclusão do flow:

.. code-block:: python
    :linenos:

    from pywa import WhatsApp
    from pywa.types import FlowCompletion

    wa = WhatsApp(...)

    @wa.on_flow_completion
    def on_flow_completion(_: WhatsApp, flow: FlowCompletion):
        print(f"The user {flow.from_user.name} just completed the flow!")
        print(flow.response)

O atributo ``.response`` é o payload enviado ao concluir o flow.

.. note::

    Se você usar componentes :class:`PhotoPicker` ou :class:`DocumentPicker`, receberá os arquivos dentro do .response da conclusão do flow.
    Você pode construí-los como objetos de mídia do pywa usando :meth:`~pywa.types.FlowCompletion.get_media`:

    .. code-block:: python
        :linenos:

        from pywa import WhatsApp, types

        wa = WhatsApp(...)

        @wa.on_flow_completion
        def on_flow_completion(_: WhatsApp, flow: FlowCompletion):
            img = flow.get_media(types.Image, key="profile_pic")
            img.download()


Processando requisições do Flow
--------------------------------

É aqui que as coisas ficam interessantes. WhatsApp Flows pode ser dinâmico, o que significa que você pode lidar com ações do usuário e responder a elas em tempo real a partir do seu servidor.


.. note::

    Como as requisições e respostas podem conter dados sensíveis, como senhas e outras informações pessoais,
    todas as requisições e respostas são criptografadas usando a `WhatsApp Business Encryption <https://developers.facebook.com/docs/whatsapp/cloud-api/reference/whatsapp-business-encryption>`_.

    Antes de continuar, você precisa assinar e fazer o upload da chave pública da empresa.
    Primeiro você precisa gerar uma chave privada e uma chave pública:

    Gere um par de chaves RSA pública e privada digitando o seguinte comando:

    >>> openssl genrsa -des3 -out private.pem 2048


    Isso gera um par de chaves RSA de 2048 bits criptografado com a senha fornecida e salvo em um arquivo.

    Em seguida, você precisa exportar a Chave Pública RSA para um arquivo.

    >>> openssl rsa -in private.pem -outform PEM -pubout -out public.pem


    Isso exporta a Chave Pública RSA para um arquivo.

    Depois de ter a chave pública, você pode fazer o upload dela usando o método :meth:`~pywa.client.WhatsApp.set_business_public_key`.

    .. code-block:: python
        :linenos:

        from pywa import WhatsApp

        wa = WhatsApp(...)

        wa.set_business_public_key(open("public.pem").read())

    Cada requisição precisa ser descriptografada usando a chave privada, portanto você precisa fornecê-la ao criar o objeto :class:`WhatsApp`:

    .. code-block:: python
        :linenos:

        from pywa import WhatsApp

        wa = WhatsApp(..., business_private_key=open("private.pem").read())

    Agora você está pronto para processar as requisições.

    Mais uma coisa: a implementação padrão de descriptografia e criptografia usa a biblioteca `cryptography <https://cryptography.io/en/latest/>`_,
    portanto você precisa instalá-la:

    >>> pip3 install cryptography

    Ou ao instalar o PyWa:

    >>> pip3 install "pywa[cryptography]"

Veja um exemplo de flow dinâmico:


.. code-block:: python
    :caption: sign_in_flow.py
    :linenos:
    :emphasize-lines: 5-11, 13, 16, 18-27, 31, 36, 38, 58-59, 66, 71, 77, 83, 87, 90, 94, 97, 102, 105, 113, 121-127, 134, 142, 146, 152


    from pywa.types.flows import *

    flow = FlowJSON(
        version="7.2",
        data_api_version="3.0",
        routing_model={
            "SIGN_IN": ["SIGN_UP", "FORGOT_PASSWORD"],
            "SIGN_UP": ["TERMS_AND_CONDITIONS"],
            "FORGOT_PASSWORD": [],
            "TERMS_AND_CONDITIONS": [],
        },
        screens=[
            signin_screen := Screen(
                id="SIGN_IN",
                title="Sign in",
                terminal=True,
                success=True,
                data=[
                    welcome := ScreenData(
                        key="welcome",
                        example="Welcome back! Please sign in to continue.",
                    ),
                    default_email := ScreenData(
                        key="default_email",
                        example="johndoe@gmail.com",
                    ),
                ],
                layout=Layout(
                    children=[
                        TextSubheading(text=welcome.ref),
                        signin_email := TextInput(
                            name="email",
                            label="Email address",
                            input_type=InputType.EMAIL,
                            required=True,
                            init_value=default_email.ref,
                        ),
                        signin_password := TextInput(
                            name="password",
                            label="Password",
                            input_type=InputType.PASSWORD,
                            required=True,
                        ),
                        EmbeddedLink(
                            text="Don't have an account? Sign up",
                            on_click_action=NavigateAction(next=Next(name="SIGN_UP")),
                        ),
                        EmbeddedLink(
                            text="Forgot password",
                            on_click_action=NavigateAction(
                                next=Next(name="FORGOT_PASSWORD"),
                            )
                        ),
                        Footer(
                            label="Sign in",
                            on_click_action=DataExchangeAction(
                                payload={
                                    "email": signin_email.ref,
                                    "password": signin_password.ref,
                                }
                            ),
                        ),
                    ]
                ),
            ),
            signup_screen := Screen(
                id="SIGN_UP",
                title="Sign up",
                layout=Layout(
                    children=[
                        first_name := TextInput(
                            name="first_name",
                            label="First Name",
                            input_type=InputType.TEXT,
                            required=True,
                        ),
                        last_name := TextInput(
                            name="last_name",
                            label="Last Name",
                            input_type=InputType.TEXT,
                            required=True,
                        ),
                        signup_email := TextInput(
                            name="email",
                            label="Email address",
                            input_type=InputType.EMAIL,
                            init_value=signin_screen / signin_email.ref,
                            required=True,
                        ),
                        signup_password := TextInput(
                            name="password",
                            label="Set password",
                            input_type=InputType.PASSWORD,
                            init_value=signin_screen / signin_password.ref,
                            required=True,
                        ),
                        confirm_password := TextInput(
                            name="confirm_password",
                            label="Confirm password",
                            helper_text="Min 8 chars, incl. 1 number & 1 special character.",
                            input_type=InputType.PASSWORD,
                            init_value=signin_screen / signin_password.ref,
                            required=True,
                        ),
                        terms_agreement := OptIn(
                            name="terms_agreement",
                            label="I agree with the terms.",
                            on_click_action=NavigateAction(
                                next=Next(type="screen", name="TERMS_AND_CONDITIONS")
                            ),
                            required=True,
                        ),
                        offers_acceptance := OptIn(
                            name="offers_acceptance",
                            label="I would like to receive news and offers.",
                        ),
                        Footer(
                            label="Sign up",
                            on_click_action=DataExchangeAction(
                                payload={
                                    "first_name": first_name.ref,
                                    "last_name": last_name.ref,
                                    "email": signup_email.ref,
                                    "password": signup_password.ref,
                                    "confirm_password": confirm_password.ref,
                                    "terms_agreement": terms_agreement.ref,
                                    "offers_acceptance": offers_acceptance.ref,
                                }
                            ),
                        ),
                    ]
                ),
            ),
            forgot_password_screen := Screen(
                id="FORGOT_PASSWORD",
                title="Forgot password",
                terminal=True,
                success=True,
                layout=Layout(
                    children=[
                        TextBody(text="Enter your email address for your account and we'll send a reset link. The single-use link will expire after 24 hours."),
                        forgot_password_email := TextInput(
                            name="email",
                            label="Email address",
                            input_type=InputType.EMAIL,
                            init_value=signin_screen / signin_email.ref,
                            required=True,
                        ),
                        Footer(
                            label="Send reset link",
                            on_click_action=DataExchangeAction(
                                payload={"email": forgot_password_email.ref}
                            ),
                        ),
                    ]
                ),
            ),
            Screen(
                id="TERMS_AND_CONDITIONS",
                title="Terms and conditions",
                layout=Layout(
                    children=[
                        TextHeading(text="Our Terms"),
                        TextSubheading(text="Data usage"),
                        TextBody(
                            text="Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed vitae odio dui. Praesent ut nulla tincidunt, scelerisque augue malesuada, volutpat lorem. Aliquam iaculis ex at diam posuere mollis. Suspendisse eget purus ac tellus interdum pharetra. In quis dolor turpis. Fusce in porttitor enim, vitae efficitur nunc. Fusce dapibus finibus volutpat. Fusce velit mi, ullamcorper ac gravida vitae, blandit quis ex. Fusce ultrices diam et justo blandit, quis consequat nisl euismod. Vestibulum pretium est sem, vitae convallis justo sollicitudin non. Morbi bibendum purus mattis quam condimentum, a scelerisque erat bibendum. Nullam sit amet bibendum lectus."
                        ),
                        TextSubheading(text="Privacy policy"),
                        TextBody(
                            text="Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed vitae odio dui. Praesent ut nulla tincidunt, scelerisque augue malesuada, volutpat lorem. Aliquam iaculis ex at diam posuere mollis. Suspendisse eget purus ac tellus interdum pharetra. In quis dolor turpis. Fusce in porttitor enim, vitae efficitur nunc. Fusce dapibus finibus volutpat. Fusce velit mi, ullamcorper ac gravida vitae, blandit quis ex. Fusce ultrices diam et justo blandit, quis consequat nisl euismod. Vestibulum pretium est sem, vitae convallis justo sollicitudin non. Morbi bibendum purus mattis quam condimentum, a scelerisque erat bibendum. Nullam sit amet bibendum lectus."
                        ),
                    ]
                ),
            ),
        ],
    )

Que é o equivalente ao seguinte flow json:

.. toggle::

    .. code-block:: json
        :caption: sign_in_flow.json
        :linenos:

        {
            "version": "7.2",
            "data_api_version": "3.0",
            "routing_model": {
                "SIGN_IN": [
                    "SIGN_UP",
                    "FORGOT_PASSWORD"
                ],
                "SIGN_UP": [
                    "TERMS_AND_CONDITIONS"
                ],
                "FORGOT_PASSWORD": [],
                "TERMS_AND_CONDITIONS": []
            },
            "screens": [
                {
                    "id": "SIGN_IN",
                    "title": "Sign in",
                    "data": {
                        "welcome": {
                            "type": "string",
                            "__example__": "Welcome back! Please sign in to continue."
                        },
                        "default_email": {
                            "type": "string",
                            "__example__": "johndoe@gmail.com"
                        }
                    },
                    "terminal": true,
                    "success": true,
                    "layout": {
                        "type": "SingleColumnLayout",
                        "children": [
                            {
                                "type": "TextSubheading",
                                "text": "${data.welcome}"
                            },
                            {
                                "type": "TextInput",
                                "name": "email",
                                "label": "Email address",
                                "input-type": "email",
                                "required": true,
                                "init-value": "${data.default_email}"
                            },
                            {
                                "type": "TextInput",
                                "name": "password",
                                "label": "Password",
                                "input-type": "password",
                                "required": true
                            },
                            {
                                "type": "EmbeddedLink",
                                "text": "Don't have an account? Sign up",
                                "on-click-action": {
                                    "name": "navigate",
                                    "next": {
                                        "name": "SIGN_UP",
                                        "type": "screen"
                                    },
                                    "payload": {}
                                }
                            },
                            {
                                "type": "EmbeddedLink",
                                "text": "Forgot password",
                                "on-click-action": {
                                    "name": "navigate",
                                    "next": {
                                        "name": "FORGOT_PASSWORD",
                                        "type": "screen"
                                    },
                                    "payload": {}
                                }
                            },
                            {
                                "type": "Footer",
                                "label": "Sign in",
                                "on-click-action": {
                                    "name": "data_exchange",
                                    "payload": {
                                        "email": "${form.email}",
                                        "password": "${form.password}"
                                    }
                                }
                            }
                        ]
                    }
                },
                {
                    "id": "SIGN_UP",
                    "title": "Sign up",
                    "layout": {
                        "type": "SingleColumnLayout",
                        "children": [
                            {
                                "type": "TextInput",
                                "name": "first_name",
                                "label": "First Name",
                                "input-type": "text",
                                "required": true
                            },
                            {
                                "type": "TextInput",
                                "name": "last_name",
                                "label": "Last Name",
                                "input-type": "text",
                                "required": true
                            },
                            {
                                "type": "TextInput",
                                "name": "email",
                                "label": "Email address",
                                "input-type": "email",
                                "required": true,
                                "init-value": "${screen.SIGN_IN.form.email}"
                            },
                            {
                                "type": "TextInput",
                                "name": "password",
                                "label": "Set password",
                                "input-type": "password",
                                "required": true,
                                "init-value": "${screen.SIGN_IN.form.password}"
                            },
                            {
                                "type": "TextInput",
                                "name": "confirm_password",
                                "label": "Confirm password",
                                "input-type": "password",
                                "required": true,
                                "helper-text": "Min 8 chars, incl. 1 number & 1 special character.",
                                "init-value": "${screen.SIGN_IN.form.password}"
                            },
                            {
                                "type": "OptIn",
                                "name": "terms_agreement",
                                "label": "I agree with the terms.",
                                "required": true,
                                "on-click-action": {
                                    "name": "navigate",
                                    "next": {
                                        "name": "TERMS_AND_CONDITIONS",
                                        "type": "screen"
                                    },
                                    "payload": {}
                                }
                            },
                            {
                                "type": "OptIn",
                                "name": "offers_acceptance",
                                "label": "I would like to receive news and offers."
                            },
                            {
                                "type": "Footer",
                                "label": "Sign up",
                                "on-click-action": {
                                    "name": "data_exchange",
                                    "payload": {
                                        "first_name": "${form.first_name}",
                                        "last_name": "${form.last_name}",
                                        "email": "${form.email}",
                                        "password": "${form.password}",
                                        "confirm_password": "${form.confirm_password}",
                                        "terms_agreement": "${form.terms_agreement}",
                                        "offers_acceptance": "${form.offers_acceptance}"
                                    }
                                }
                            }
                        ]
                    }
                },
                {
                    "id": "FORGOT_PASSWORD",
                    "title": "Forgot password",
                    "terminal": true,
                    "success": true,
                    "layout": {
                        "type": "SingleColumnLayout",
                        "children": [
                            {
                                "type": "TextBody",
                                "text": "Enter your email address for your account and we'll send a reset link. The single-use link will expire after 24 hours."
                            },
                            {
                                "type": "TextInput",
                                "name": "email",
                                "label": "Email address",
                                "input-type": "email",
                                "required": true,
                                "init-value": "${screen.SIGN_IN.form.email}"
                            },
                            {
                                "type": "Footer",
                                "label": "Send reset link",
                                "on-click-action": {
                                    "name": "data_exchange",
                                    "payload": {
                                        "email": "${form.email}"
                                    }
                                }
                            }
                        ]
                    }
                },
                {
                    "id": "TERMS_AND_CONDITIONS",
                    "title": "Terms and conditions",
                    "layout": {
                        "type": "SingleColumnLayout",
                        "children": [
                            {
                                "type": "TextHeading",
                                "text": "Our Terms"
                            },
                            {
                                "type": "TextSubheading",
                                "text": "Data usage"
                            },
                            {
                                "type": "TextBody",
                                "text": "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed vitae odio dui. Praesent ut nulla tincidunt, scelerisque augue malesuada, volutpat lorem. Aliquam iaculis ex at diam posuere mollis. Suspendisse eget purus ac tellus interdum pharetra. In quis dolor turpis. Fusce in porttitor enim, vitae efficitur nunc. Fusce dapibus finibus volutpat. Fusce velit mi, ullamcorper ac gravida vitae, blandit quis ex. Fusce ultrices diam et justo blandit, quis consequat nisl euismod. Vestibulum pretium est sem, vitae convallis justo sollicitudin non. Morbi bibendum purus mattis quam condimentum, a scelerisque erat bibendum. Nullam sit amet bibendum lectus."
                            },
                            {
                                "type": "TextSubheading",
                                "text": "Privacy policy"
                            },
                            {
                                "type": "TextBody",
                                "text": "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed vitae odio dui. Praesent ut nulla tincidunt, scelerisque augue malesuada, volutpat lorem. Aliquam iaculis ex at diam posuere mollis. Suspendisse eget purus ac tellus interdum pharetra. In quis dolor turpis. Fusce in porttitor enim, vitae efficitur nunc. Fusce dapibus finibus volutpat. Fusce velit mi, ullamcorper ac gravida vitae, blandit quis ex. Fusce ultrices diam et justo blandit, quis consequat nisl euismod. Vestibulum pretium est sem, vitae convallis justo sollicitudin non. Morbi bibendum purus mattis quam condimentum, a scelerisque erat bibendum. Nullam sit amet bibendum lectus."
                            }
                        ]
                    }
                }
            ]
        }


Este flow tem 4 telas:

- ``SIGN_IN`` - A primeira tela que o usuário vê ao abrir o flow. Contém um formulário de login e links para as telas de cadastro e recuperação de senha.
- ``SIGN_UP`` - A tela que o usuário vê ao clicar no link de cadastro na tela de login. Contém um formulário de cadastro e um link para a tela de termos e condições.
- ``FORGOT_PASSWORD`` - A tela que o usuário vê ao clicar no link de recuperação de senha na tela de login. Contém um formulário para enviar um link de redefinição ao endereço de e-mail do usuário.
- ``TERMS_AND_CONDITIONS`` - A tela que o usuário vê ao clicar no link de termos e condições na tela de cadastro. Contém o texto dos termos e condições.

Vamos explorar os principais conceitos de flows dinâmicos:

- **data_api_version**: É a versão da API de dados utilizada pelo flow. É usada para determinar como os dados são trocados entre o cliente e o servidor. A versão atual é ``3.0``.
- **routing_model**: É um dicionário que define o roteamento do flow. Mapeia IDs de telas para outros IDs de telas para as quais se pode navegar a partir da tela atual. Por exemplo, da tela ``SIGN_IN``, você pode navegar para as telas ``SIGN_UP`` ou ``FORGOT_PASSWORD``. Você pode ler mais sobre o Routing Model em `developers.facebook.com <https://developers.facebook.com/docs/whatsapp/flows/reference/flowjson#routing-model>`_.
- **data**: É uma lista de objetos :class:`ScreenData` que define os dados que devem ser fornecidos à tela ao navegar para ela. Esses dados podem ser usados para pré-preencher campos do formulário ou fornecer outras informações ao usuário. Por exemplo, na tela ``SIGN_IN``, temos um screen data ``welcome`` que fornece uma mensagem de boas-vindas e um screen data ``default_email`` que fornece um endereço de e-mail padrão para pré-preencher o campo de e-mail (você verá por que precisamos disso mais adiante).
- **ref**: É uma referência ao screen data ou ao componente que armazena uma entrada do usuário. É usada para referenciar os dados dentro do flow. Por exemplo, na tela ``SIGN_IN``, temos um componente ``signin_email`` com uma referência ao campo de e-mail. Podemos usar essa referência para obter o valor do campo de e-mail quando o usuário enviar o formulário.
- **on_click_action**: É uma ação executada quando o usuário clica em um botão ou link. Pode ser uma :class:`DataExchangeAction`, :class:`NavigateAction`, :class:`CompleteAction` ou :class:`OpenURLAction`. Por exemplo, na tela ``SIGN_IN``, temos um componente ``Footer`` com o rótulo "Sign in" que possui uma :class:`DataExchangeAction` que envia o e-mail e a senha ao servidor quando o usuário clica. O servidor então validará as credenciais e responderá com a próxima tela a exibir ou encerrará o flow.

Precisamos atualizar o flow com este json usando :meth:`~pywa.client.WhatsApp.update_flow_json` e então informar ao WhatsApp para enviar as requisições ao nosso servidor usando :meth:`~pywa.client.WhatsApp.update_flow_metadata`:

.. code-block:: python
    :linenos:
    :emphasize-lines: 7

    from pywa import WhatsApp

    wa = WhatsApp(...)

    wa.update_flow_metadata(
        flow_id="1234567890123456",  # The `sign_in_flow` flow id from above
        endpoint_uri="https://your-server.com/flow"
    )

Vamos enviar o flow, desta vez com uma imagem:

.. code-block:: python
    :linenos:
    :emphasize-lines: 12, 14

    from pywa import WhatsApp
    from pywa.types import FlowButton, FlowActionType, FlowStatus

    wa = WhatsApp(...)

    wa.send_image(
        to="1234567890",
        image="https://t3.ftcdn.net/jpg/03/82/73/76/360_F_382737626_Th2TUrj9PbvWZKcN9Kdjxu2yN35rA9nU.jpg",
        caption="Hi, You need to finish your sign up!",
        buttons=FlowButton(
            title="Finish Sign Up",
            flow_id="1234567890123456",  # The `sign_in_flow` flow id from above
            mode=FlowStatus.DRAFT,
            flow_action_type=FlowActionType.DATA_EXCHANGE,  # This time we want to exchange data
        )
    )

Aqui definimos o ``flow_action_type`` como ``FlowActionType.DATA_EXCHANGE`` porque queremos trocar dados com o servidor.
Assim, quando o usuário abre o flow, receberemos uma requisição no nosso servidor para fornecer a tela a abrir e os dados a disponibilizar.


.. code-block:: python
    :linenos:

    import datetime
    import re
    import dataclasses
    from pywa import WhatsApp
    from pywa.types import FlowRequest, FlowResponse

    @dataclasses.dataclass
    class User:
        email: str
        password: str
        first_name: str
        last_name: str
        offer_acceptance: bool
        forget_password_requested: datetime.datetime | None = None
        is_signed_in: bool = False


    class DemoDatabase:
        def __init__(self):
            self.users: dict[str, User] = {}

        def get_user(self, email: str) -> User | None:
            return self.users.get(email)

        def add_user(self, user: User) -> None:
            self.users[user.email] = user

        def is_forget_password_available(self, email: str) -> bool:
            user = self.get_user(email)
            if not user:
                return True
            if user.forget_password_requested and user.forget_password_requested > (datetime.datetime.now() - datetime.timedelta(hours=24)):
                return False
            return True

    db = DemoDatabase()

    wa = WhatsApp(
        ...,
        business_private_key=open("private.pem").read(),  # provide your business private key
    )

    @wa.on_flow_request(endpoint="/signin")
    def handle_signin_flow(_: WhatsApp, req: FlowRequest) -> FlowResponse:
        raise NotImplementedError(req)


    @handle_signin_flow.on_init
    def on_init(_: WhatsApp, req: FlowRequest) -> FlowResponse:
        return req.respond(
            screen=signin_screen,
            data={
                welcome.key: "Welcome to our service! Please sign in to continue.",
                default_email.key: "",
            },
        )


    @handle_signin_flow.on_data_exchange(screen=signin_screen)
    def on_sign_in(_: WhatsApp, req: FlowRequest) -> FlowResponse:
        user = db.get_user(req.data["email"])
        if not user:
            return req.respond(
                screen=signin_screen,
                error_message="User not found. Please sign up first.",
                data={
                    welcome.key: "Welcome to our service! Please sign in to continue.",
                    default_email.key: "",
                },
            )
        if user.password != req.data["password"]:
            return req.respond(
                screen=signin_screen,
                error_message="Incorrect password. Please try again.",
                data={
                    welcome.key: "Welcome to our service! Please sign in to continue.",
                    default_email.key: user.email,
                },
            )
        user.is_signed_in = True
        return req.respond(close_flow=True)


    @handle_signin_flow.on_data_exchange(screen=signup_screen)
    def on_sign_up(_: WhatsApp, req: FlowRequest) -> FlowResponse:
        if not re.match(r"^(?=.*\d)(?=.*[^A-Za-z0-9]).{8,}$", req.data["password"]):
            return req.respond(
                screen=signup_screen,
                error_message="Password must be at least 8 characters long and contain at least one number and one special character.",
            )
        if req.data["password"] != req.data["confirm_password"]:
            return req.respond(
                screen=signup_screen,
                error_message="Passwords do not match. Please try again.",
            )
        user = User(
            email=req.data["email"],
            password=req.data["password"],
            first_name=req.data["first_name"],
            last_name=req.data["last_name"],
            offer_acceptance=req.data["offers_acceptance"],
            is_signed_in=False
        )
        db.add_user(user)
        return req.respond(
            screen=signin_screen,
            data={
                welcome.key: "Thank you for signing up! You can now sign in with your new account.",
                default_email.key: user.email,
            },
        )

    @handle_signin_flow.on_data_exchange(screen=forgot_password_screen)
    def on_forgot_password(_: WhatsApp, req: FlowRequest) -> FlowResponse:
        if not db.is_forget_password_available(req.data["email"]):
            return req.respond(
                screen=forgot_password_screen,
                error_message="You can't request a password reset at this time. Please try again later.",
            )
        ### SEND PASSWORD RESET EMAIL HERE ###
        return req.respond(
            screen=signin_screen,
            data={
                welcome.key: "A password reset link has been sent to your email address. Please check your inbox.",
                default_email.key: req.data["email"],
            },
        )


.. note::

    Se você usar componentes :class:`PhotoPicker` ou :class:`DocumentPicker` e processar requisições contendo seus dados, você precisa
    descriptografar os arquivos usando :meth:`~pywa.types.flows.FlowRequest.decrypt_media`:

    .. code-block:: python
        :linenos:
        :emphasize-lines: 3

        @wa.on_flow_request(endpoint="/flow")
        def on_support_request(_: WhatsApp, req: FlowRequest) -> FlowResponse:
            decrypted_data = req.decrypt_media(key="driver_license", index=0)
            with open(f"media/{decrypted_data.filename}", "wb") as f:
                f.write(decrypted_data.data)
            ...

.. toctree::
    flow_json
    flow_types
