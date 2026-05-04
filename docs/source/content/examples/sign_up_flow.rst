Flow de Cadastro
================

.. currentmodule:: pywa.types.flows

**Neste exemplo, vamos criar um flow de cadastro que permite aos usuários se registrar e fazer login em suas contas.**

    .. image:: ../../../../_static/guides/sign-up-flow.webp
        :alt: Sign Up Flow
        :width: 100%

Pense em um Flow como uma coleção de telas relacionadas. As telas podem trocar dados entre si e com o seu servidor.

Uma tela pode ser estática: ela pode exibir conteúdo estático configurado quando o flow é criado. Por exemplo, uma tela pode
exibir uma mensagem de boas-vindas genérica sem nenhum conteúdo dinâmico, ou pode exibir uma mensagem que será diferente para cada usuário
ao fornecer o texto da mensagem quando o flow é enviado ao usuário ou quando solicitado ao servidor.

Quase todo aspecto de um componente de tela pode ser dinâmico. Por exemplo, vamos considerar o componente :class:`TextInput`, usado para
coletar a entrada do usuário (ex.: nome, e-mail, senha, etc.). O rótulo, o tipo de entrada, o texto de ajuda, o número mínimo e máximo de caracteres,
se a entrada é obrigatória ou não, se o campo está pré-preenchido com um valor e se o campo está desabilitado ou não, podem ser todos dinâmicos.

Cada :class:`Screen` possui

- Um ``id``: O ID único da tela, usado para navegação
- Um ``title``: O título da tela, que é renderizado no topo da tela
- Um ``layout``: O layout da tela, que contém os elementos exibidos nela.
- Um ``data``: Os dados que a tela espera receber. Esses dados são usados para inserir conteúdo e configurar a tela
  a fim de torná-la dinâmica.

É importante entender que não importa em qual ordem as telas são definidas, cada tela é independente
e possui seus próprios dados.

Acho que é mais fácil entender isso com exemplos, então vamos começar.

Tela Inicial
-------------

Vamos começar pela tela ``START``. Esta tela dá boas-vindas ao usuário e permite que ele escolha se deseja se cadastrar
(criar uma conta) ou fazer login na conta existente.

.. code-block:: python
    :caption: start_screen.py
    :linenos:
    :emphasize-lines: 6, 9, 26


    START = Screen(
        id="START",
        title="Home",
        layout=Layout(
            children=[
                TextHeading(
                    text="Welcome to our app",
                ),
                EmbeddedLink(
                    text="Click here to sign up",
                    on_click_action=Action(
                        name=FlowActionType.NAVIGATE,
                        next=Next(
                            type=NextType.SCREEN,
                            name="SIGN_UP",
                        ),
                        payload={
                            "first_name_initial_value": "",
                            "last_name_initial_value": "",
                            "email_initial_value": "",
                            "password_initial_value": "",
                            "confirm_password_initial_value": "",
                        },
                    ),
                ),
                EmbeddedLink(
                    text="Click here to login",
                    on_click_action=Action(
                        name=FlowActionType.NAVIGATE,
                        next=Next(
                            type=NextType.SCREEN,
                            name="LOGIN",
                        ),
                        payload={
                            "email_initial_value": "",
                            "password_initial_value": "",
                        },
                    ),
                ),
            ]
        ),
    )

Este é um exemplo de tela estática. A tela não espera receber nenhum dado e todos os seus componentes estão pré-configurados.

A tela ``START`` possui três componentes:

- Um :class:`TextHeading`, que dá boas-vindas ao usuário
- Um :class:`EmbeddedLink` com uma :class:`Action` que navega para a tela ``SIGN_UP``
- Um :class:`EmbeddedLink` com uma :class:`Action` que navega para a tela ``LOGIN``

Cada EmbeddedLink tem um ``.on_click_action`` com valor de Action, portanto, quando o usuário clica no link, a ação é acionada. Neste caso,
a ação é :class:`FlowActionType.NAVIGATE` para outra tela. O payload contém os dados que serão passados para a tela navegada; neste
caso, estamos passando os dados esperados das telas ``SIGN_UP`` e ``LOGIN``.

Veremos como isso funciona mais adiante.


Tela de Cadastro
-----------------

A tela ``SIGN_UP`` permite que o usuário se cadastre (crie uma conta). Vamos ver o layout:


.. code-block:: python
    :caption: sign_up_screen.py
    :linenos:
    :emphasize-lines: 4, 5, 6, 7, 8, 9, 10, 25, 26, 33, 38, 40, 45, 47, 52, 54, 62, 64, 71, 77, 78, 79, 80, 81, 82, 83

    SIGN_UP = Screen(
        id="SIGN_UP",
        title="Sign Up",
        data=[
            first_name_initial_value := ScreenData(key="first_name_initial_value", example="John"),
            last_name_initial_value := ScreenData(key="last_name_initial_value", example="Doe"),
            email_initial_value := ScreenData(key="email_initial_value", example="john.doe@gmail.com"),
            password_initial_value := ScreenData(key="password_initial_value", example="abc123"),
            confirm_password_initial_value := ScreenData(key="confirm_password_initial_value", example="abc123"),
        ],
        layout=Layout(
            children=[
                TextHeading(
                    text="Please enter your details",
                ),
                EmbeddedLink(
                    text="Already have an account?",
                    on_click_action=Action(
                        name=FlowActionType.NAVIGATE,
                        next=Next(
                            type=NextType.SCREEN,
                            name="LOGIN",
                        ),
                        payload={
                            "email_initial_value": "",
                            "password_initial_value": "",
                        },
                    ),
                ),
                Form(
                    name="form",
                    children=[
                        first_name := TextInput(
                            name="first_name",
                            label="First Name",
                            input_type=InputType.TEXT,
                            required=True,
                            init_value=first_name_initial_value.ref,
                        ),
                        last_name := TextInput(
                            name="last_name",
                            label="Last Name",
                            input_type=InputType.TEXT,
                            required=True,
                            init_value=last_name_initial_value.ref,
                        ),
                        email := TextInput(
                            name="email",
                            label="Email Address",
                            input_type=InputType.EMAIL,
                            required=True,
                            init_value=email_initial_value.ref,
                        ),
                        password := TextInput(
                            name="password",
                            label="Password",
                            input_type=InputType.PASSWORD,
                            min_chars=8,
                            max_chars=16,
                            helper_text="Password must contain at least one number",
                            required=True,
                            init_value=password_initial_value.ref,
                        ),
                        confirm_password := TextInput(
                            name="confirm_password",
                            label="Confirm Password",
                            input_type=InputType.PASSWORD,
                            min_chars=8,
                            max_chars=16,
                            required=True,
                            init_value=confirm_password_initial_value.ref,
                        ),
                        Footer(
                            label="Done",
                            on_click_action=Action(
                                name=FlowActionType.DATA_EXCHANGE,
                                payload={
                                    "first_name": first_name.ref,
                                    "last_name": last_name.ref,
                                    "email": email.ref,
                                    "password": password.ref,
                                    "confirm_password": confirm_password.ref,
                                },
                            ),
                        ),
                    ]
                )
            ]
        )
    )


Ok, isso é muito código. Vamos analisar parte por parte.

    Nestes exemplos estamos usando o operador walrus (:=) para atribuir valores a variáveis. Isso nos permite usar as
    variáveis mais adiante no código sem precisar declará-las fora do layout e atribuir valores depois.

A tela ``SIGN_UP`` espera receber alguns dados. Neste caso, esperamos receber alguns valores para pré-preencher os campos do formulário.

Os dados da tela são representados pela propriedade ``.data``. Os dados são uma lista de objetos :class:`ScreenData`.

Cada :class:`ScreenData` precisa ter uma ``key`` única e um valor ``example``. O valor de exemplo é usado para gerar o
esquema JSON apropriado para os dados. Além disso, estamos atribuindo cada :class:`ScreenData` a uma variável (inline com o operador walrus) para
que possamos usá-las posteriormente no código para referenciar os dados e "utilizá-los" na tela (ex.: ``first_name_initial_value.ref``).

O layout da tela ``SIGN_UP`` contém os seguintes elementos:

- Um :class:`TextHeading`, que pede ao usuário para inserir seus dados
- Um :class:`EmbeddedLink` para a tela ``LOGIN``, que permite ao usuário fazer login caso já tenha uma conta (o usuário acabou de lembrar
  que já tem uma conta)
- Um :class:`Form`, que contém os campos do formulário que o usuário precisa preencher para se cadastrar
- Um :class:`Footer`, que contém um botão que o usuário pode clicar para enviar o formulário

Os campos do :class:`Form` são:

- Um campo :class:`TextInput` para o primeiro nome, que é obrigatório
- Um campo :class:`TextInput` para o sobrenome, que é obrigatório
- Um campo :class:`TextInput` para o endereço de e-mail (o tipo de entrada está definido como :class:`InputType.EMAIL`, para que o teclado no telefone do usuário
  mostre o símbolo ``@`` e valide o endereço de e-mail; além disso, a entrada é obrigatória)
- Um campo :class:`TextInput` para a senha (o tipo de entrada está definido como :class:`InputType.PASSWORD`, para que a senha do usuário fique oculta ao digitá-la)
  Também fornecemos um texto de ajuda para informar ao usuário que a senha deve conter pelo menos um número. Além disso, o número mínimo de caracteres é 8 e o máximo é 16, e a entrada é obrigatória)
- Um campo :class:`TextInput` para confirmar a senha (o tipo de entrada está definido como :class:`InputType.PASSWORD`, para que a senha do usuário fique oculta ao redigitá-la)

Agora, cada filho do formulário é atribuído a uma variável (inline com o operador walrus) para que possamos usá-los posteriormente no
código para referenciar os campos do formulário e enviar seus "valores" ao servidor ou para outra tela (ex.: ``first_name.ref``).

O :class:`Footer` contém um botão que o usuário pode clicar para enviar o formulário. Quando o usuário clica no botão, a :class:`Action`
:class:`FlowActionType.DATA_EXCHANGE` é acionada. Este tipo de ação nos permite enviar dados ao servidor e então decidir o que fazer a seguir (por exemplo,
se o usuário já estiver registrado, podemos navegar para a tela ``LOGIN``, ou se a senha e a confirmação de senha não coincidirem,
podemos mostrar uma mensagem de erro e pedir ao usuário para tentar novamente).

O payload da :class:`Action` do :class:`Footer` contém os dados que queremos enviar ao servidor. Neste caso, estamos enviando
os valores dos campos do formulário. Os valores podem ser um :class:`ScreenDataRef` ou um :class:`ComponentRef`. Um :class:`ScreenDataRef` é usado para referenciar
os itens de ``.data`` de uma tela e um :class:`ComponentRef` é usado para referenciar filhos de :class:`Form`.
Como estamos usando o operador walrus para atribuir os campos do formulário a variáveis, podemos usar as variáveis para referenciar os campos do formulário
usando a propriedade ``.ref`` do campo (que é mais segura quanto a tipos do que usar :class:`ComponentRef` com o nome do campo do formulário).


As propriedades ``.ref`` são equivalentes ao :class:`ComponentRef` com o nome do componente do formulário e ao :class:`ScreenDataRef` com a
referência da tela, respectivamente. Na verdade, as propriedades ``.ref`` são apenas atalhos para as classes :class:`ComponentRef` e :class:`ScreenDataRef`.


Tela de Login
--------------

Ok, agora para a tela ``LOGIN``. Esta tela permite que o usuário faça login na conta existente.


.. code-block:: python
    :caption: login_screen.py
    :linenos:
    :emphasize-lines: 5, 6, 7, 8, 22, 23, 24, 25, 26, 27, 28, 34, 39, 41, 46, 52, 53, 54, 55

    LOGIN = Screen(
        id="LOGIN",
        title="Login",
        terminal=True,
        data=[
            email_initial_value := ScreenData(key="email_initial_value", example="john.doe@gmail.com"),
            password_initial_value := ScreenData(key="password_initial_value", example="abc123"),
        ],
        layout=Layout(
            children=[
                TextHeading(
                    text="Please enter your details"
                ),
                EmbeddedLink(
                    text="Don't have an account?",
                    on_click_action=Action(
                        name=FlowActionType.NAVIGATE,
                        next=Next(
                            type=NextType.SCREEN,
                            name="SIGN_UP",
                        ),
                        payload={
                            "email_initial_value": "",
                            "password_initial_value": "",
                            "confirm_password_initial_value": "",
                            "first_name_initial_value": "",
                            "last_name_initial_value": "",
                        },
                    ),
                ),
                Form(
                    name="form",
                    children=[
                        email := TextInput(
                            name="email",
                            label="Email Address",
                            input_type=InputType.EMAIL,
                            required=True,
                            init_value=email_initial_value.ref,
                        ),
                        password := TextInput(
                            name="password",
                            label="Password",
                            input_type=InputType.PASSWORD,
                            required=True,
                            init_value=password_initial_value.ref,
                        ),
                        Footer(
                            label="Done",
                            on_click_action=Action(
                                name=FlowActionType.DATA_EXCHANGE,
                                payload={
                                    "email": email.ref,
                                    "password": password.ref,
                                },
                            ),
                        ),
                    ]
                )
            ]
        )
    )


Esta tela é bastante direta. Ela tem dois elementos:

- Um campo :class:`TextInput` para o endereço de e-mail (o tipo de entrada está definido como :class:`InputType.EMAIL`, para que o teclado no telefone do usuário
  mostre o símbolo ``@`` e valide o endereço de e-mail)
- Um campo :class:`TextInput` para a senha (o tipo de entrada está definido como :class:`InputType.PASSWORD`, para que a senha do usuário fique oculta ao digitá-la)

O :class:`Footer` contém um botão que o usuário pode clicar para enviar o formulário. Quando o usuário clica no botão, usamos o
tipo de ação :class:`FlowActionType.DATA_EXCHANGE` para enviar o e-mail e a senha inseridos ao servidor e então decidir o que fazer a seguir (por exemplo,
se o usuário não estiver registrado, podemos navegar para a tela ``SIGN_UP``, ou se a senha estiver incorreta, podemos mostrar uma mensagem de erro e pedir
ao usuário para tentar novamente).

Tela de Sucesso no Login
------------------------

Agora, para a última tela, a tela ``LOGIN_SUCCESS``. Esta tela é exibida quando o usuário faz login com sucesso:

.. code-block:: python
    :caption: login_success_screen.py
    :linenos:
    :emphasize-lines: 16, 24, 25, 26

    LOGIN_SUCCESS = Screen(
        id="LOGIN_SUCCESS",
        title="Success",
        terminal=True,
        layout=Layout(
            children=[
                TextHeading(
                    text="Welcome to our store",
                ),
                TextSubheading(
                    text="You are now logged in",
                ),
                Form(
                    name="form",
                    children=[
                        stay_logged_in := OptIn(
                            name="stay_logged_in",
                            label="Stay logged in",
                        ),
                        Footer(
                            label="Done",
                            on_click_action=Action(
                                name=FlowActionType.COMPLETE,
                                payload={
                                    "stay_logged_in": stay_logged_in.ref,
                                },
                            ),
                        ),
                    ]
                )
            ]
        ),
    )

Esta tela tem dois elementos:

- Um :class:`TextHeading`, que dá boas-vindas ao usuário na loja
- Um :class:`TextSubheading`, que informa ao usuário que ele está agora conectado
- Um :class:`Form`, que contém um campo :class:`OptIn` que pergunta ao usuário se ele quer permanecer conectado

O :class:`Footer` contém um botão que o usuário pode clicar para enviar o formulário. A ação ``COMPLETE`` é usada para concluir o flow.
Quando o usuário clica no botão, usamos a ação :class:`FlowActionType.COMPLETE` para enviar o valor do campo :class:`OptIn` ao servidor e
então concluir o flow.

Esta tela é a única que pode concluir o flow, por isso estamos definindo a propriedade ``terminal`` como ``True``.


Criando o Flow
--------------

Agora, precisamos encapsular tudo em um objeto :class:`FlowJSON` e criar o flow:

.. code-block:: python
    :linenos:

    from pywa import utils
    from pywa.types.flows import FlowJSON

    SIGN_UP_FLOW_JSON = FlowJSON(
        version=utils.Version.FLOW_JSON,
        data_api_version=utils.Version.FLOW_DATA_API,
        routing_model={
            "START": ["SIGN_UP", "LOGIN"],
            "SIGN_UP": ["LOGIN"],
            "LOGIN": ["LOGIN_SUCCESS"],
            "LOGIN_SUCCESS": [],
        },
        screens=[
            START,
            SIGN_UP,
            LOGIN,
            LOGIN_SUCCESS,
        ]
    )


O objeto :class:`FlowJSON` contém as seguintes propriedades:

- ``data_api_version``: A versão da API de dados que estamos usando. Estamos usando a versão mais recente, que é ``Version.FLOW_DATA_API``
- ``routing_model``: O modelo de roteamento do flow. Ele é usado para definir a navegação do flow. Neste caso, usamos um modelo de roteamento simples
  que nos permite navegar da tela ``START`` para as telas ``SIGN_UP`` e ``LOGIN``, da tela ``SIGN_UP`` para a tela ``LOGIN`` (e vice-versa),
  e da tela ``LOGIN`` para a tela ``LOGIN_SUCCESS``. A tela ``LOGIN_SUCCESS`` não pode navegar para nenhuma outra tela.
- ``screens``: As telas do flow. Neste caso, estamos usando as telas que criamos anteriormente.

Aqui está todo o código do flow em um único lugar:

.. toggle::

    .. code-block:: python
        :linenos:

        from pywa import utils
        from pywa.types.flows import (
            FlowJSON,
            Screen,
            ScreenData,
            Form,
            Footer,
            Layout,
            Action,
            Next,
            NextType,
            FlowActionType,
            ComponentRef,
            InputType,
            TextHeading,
            TextSubheading,
            TextInput,
            OptIn,
            EmbeddedLink,
        )

        SIGN_UP_FLOW_JSON = FlowJSON(
            version=utils.Version.FLOW_JSON,
            data_api_version=utils.Version.FLOW_DATA_API,
            routing_model={
                "START": ["SIGN_UP", "LOGIN"],
                "SIGN_UP": ["LOGIN"],
                "LOGIN": ["LOGIN_SUCCESS"],
                "LOGIN_SUCCESS": [],
            },
            screens=[
                Screen(
                    id="START",
                    title="Home",
                    layout=Layout(
                        children=[
                            TextHeading(
                                text="Welcome to our app",
                            ),
                            EmbeddedLink(
                                text="Click here to sign up",
                                on_click_action=Action(
                                    name=FlowActionType.NAVIGATE,
                                    next=Next(
                                        type=NextType.SCREEN,
                                        name="SIGN_UP",
                                    ),
                                    payload={
                                        "first_name_initial_value": "",
                                        "last_name_initial_value": "",
                                        "email_initial_value": "",
                                        "password_initial_value": "",
                                        "confirm_password_initial_value": "",
                                    },
                                ),
                            ),
                            EmbeddedLink(
                                text="Click here to login",
                                on_click_action=Action(
                                    name=FlowActionType.NAVIGATE,
                                    next=Next(
                                        type=NextType.SCREEN,
                                        name="LOGIN",
                                    ),
                                    payload={
                                        "email_initial_value": "",
                                        "password_initial_value": "",
                                    },
                                ),
                            ),
                        ]
                    ),
                ),
                Screen(
                    id="SIGN_UP",
                    title="Sign Up",
                    data=[
                        first_name_initial_value := ScreenData(key="first_name_initial_value", example="John"),
                        last_name_initial_value := ScreenData(key="last_name_initial_value", example="Doe"),
                        email_initial_value := ScreenData(key="email_initial_value", example="john.doe@gmail.com"),
                        password_initial_value := ScreenData(key="password_initial_value", example="abc123"),
                        confirm_password_initial_value := ScreenData(key="confirm_password_initial_value", example="abc123"),
                    ],
                    layout=Layout(
                        children=[
                            TextHeading(
                                text="Please enter your details",
                            ),
                            EmbeddedLink(
                                text="Already have an account?",
                                on_click_action=Action(
                                    name=FlowActionType.NAVIGATE,
                                    next=Next(
                                        type=NextType.SCREEN,
                                        name="LOGIN",
                                    ),
                                    payload={
                                        "email_initial_value": "",
                                        "password_initial_value": "",
                                    },
                                ),
                            ),
                            Form(
                                name="form",
                                children=[
                                    first_name := TextInput(
                                        name="first_name",
                                        label="First Name",
                                        input_type=InputType.TEXT,
                                        required=True,
                                        init_value=first_name_initial_value.ref,
                                    ),
                                    last_name := TextInput(
                                        name="last_name",
                                        label="Last Name",
                                        input_type=InputType.TEXT,
                                        required=True,
                                        init_value=last_name_initial_value.ref,
                                    ),
                                    email := TextInput(
                                        name="email",
                                        label="Email Address",
                                        input_type=InputType.EMAIL,
                                        required=True,
                                        init_value=email_initial_value.ref,
                                    ),
                                    password := TextInput(
                                        name="password",
                                        label="Password",
                                        input_type=InputType.PASSWORD,
                                        min_chars=8,
                                        max_chars=16,
                                        helper_text="Password must contain at least one number",
                                        required=True,
                                        init_value=password_initial_value.ref,
                                    ),
                                    confirm_password := TextInput(
                                        name="confirm_password",
                                        label="Confirm Password",
                                        input_type=InputType.PASSWORD,
                                        min_chars=8,
                                        max_chars=16,
                                        required=True,
                                        init_value=confirm_password_initial_value.ref,
                                    ),
                                    Footer(
                                        label="Done",
                                        on_click_action=Action(
                                            name=FlowActionType.DATA_EXCHANGE,
                                            payload={
                                                "first_name": first_name.ref,
                                                "last_name": last_name.ref,
                                                "email": email.ref,
                                                "password": password.ref,
                                                "confirm_password": confirm_password.ref,
                                            },
                                        ),
                                    ),
                                ]
                            )
                        ]
                    )
                ),
                Screen(
                    id="LOGIN",
                    title="Login",
                    terminal=True,
                    data=[
                        email_initial_value := ScreenData(key="email_initial_value", example="john.doe@gmail.com"),
                        password_initial_value := ScreenData(key="password_initial_value", example="abc123"),
                    ],
                    layout=Layout(
                        children=[
                            TextHeading(
                                text="Please enter your details"
                            ),
                            EmbeddedLink(
                                text="Don't have an account?",
                                on_click_action=Action(
                                    name=FlowActionType.NAVIGATE,
                                    next=Next(
                                        type=NextType.SCREEN,
                                        name="SIGN_UP",
                                    ),
                                    payload={
                                        "email_initial_value": "",
                                        "password_initial_value": "",
                                        "confirm_password_initial_value": "",
                                        "first_name_initial_value": "",
                                        "last_name_initial_value": "",
                                    },
                                ),
                            ),
                            Form(
                                name="form",
                                children=[
                                    email := TextInput(
                                        name="email",
                                        label="Email Address",
                                        input_type=InputType.EMAIL,
                                        required=True,
                                        init_value=email_initial_value.ref,
                                    ),
                                    password := TextInput(
                                        name="password",
                                        label="Password",
                                        input_type=InputType.PASSWORD,
                                        required=True,
                                        init_value=password_initial_value.ref,
                                    ),
                                    Footer(
                                        label="Done",
                                        on_click_action=Action(
                                            name=FlowActionType.DATA_EXCHANGE,
                                            payload={
                                                "email": email.ref,
                                                "password": password.ref,
                                            },
                                        ),
                                    ),
                                ]
                            )
                        ]
                    ),
                ),
                Screen(
                    id="LOGIN_SUCCESS",
                    title="Success",
                    terminal=True,
                    layout=Layout(
                        children=[
                            TextHeading(
                                text="Welcome to our store",
                            ),
                            TextSubheading(
                                text="You are now logged in",
                            ),
                            Form(
                                name="form",
                                children=[
                                    stay_logged_in := OptIn(
                                        name="stay_logged_in",
                                        label="Stay logged in",
                                    ),
                                    Footer(
                                        label="Done",
                                        on_click_action=Action(
                                            name=FlowActionType.COMPLETE,
                                            payload={
                                                "stay_logged_in": stay_logged_in.ref,
                                            },
                                        ),
                                    ),
                                ]
                            )
                        ]
                    ),
                )
            ]
        )

E se você quiser ir ao `WhatsApp Flows Playground <https://business.facebook.com/wa/manage/flows>`_ e ver o flow em ação, copie o JSON equivalente para o playground:

.. toggle::

    .. code-block:: json
        :linenos:

        {
            "version": "3.0",
            "data_api_version": "3.0",
            "routing_model": {
                "START": [
                    "SIGN_UP",
                    "LOGIN"
                ],
                "SIGN_UP": [
                    "LOGIN"
                ],
                "LOGIN": [
                    "LOGIN_SUCCESS"
                ],
                "LOGIN_SUCCESS": []
            },
            "screens": [
                {
                    "id": "START",
                    "title": "Home",
                    "layout": {
                        "type": "SingleColumnLayout",
                        "children": [
                            {
                                "type": "TextHeading",
                                "text": "Welcome to our app"
                            },
                            {
                                "type": "EmbeddedLink",
                                "text": "Click here to sign up",
                                "on-click-action": {
                                    "name": "navigate",
                                    "next": {
                                        "type": "screen",
                                        "name": "SIGN_UP"
                                    },
                                    "payload": {
                                        "first_name_initial_value": "",
                                        "last_name_initial_value": "",
                                        "email_initial_value": "",
                                        "password_initial_value": "",
                                        "confirm_password_initial_value": ""
                                    }
                                }
                            },
                            {
                                "type": "EmbeddedLink",
                                "text": "Click here to login",
                                "on-click-action": {
                                    "name": "navigate",
                                    "next": {
                                        "type": "screen",
                                        "name": "LOGIN"
                                    },
                                    "payload": {
                                        "email_initial_value": "",
                                        "password_initial_value": ""
                                    }
                                }
                            }
                        ]
                    }
                },
                {
                    "id": "SIGN_UP",
                    "title": "Sign Up",
                    "data": {
                        "first_name_initial_value": {
                            "type": "string",
                            "__example__": "John"
                        },
                        "last_name_initial_value": {
                            "type": "string",
                            "__example__": "Doe"
                        },
                        "email_initial_value": {
                            "type": "string",
                            "__example__": "john.doe@gmail.com"
                        },
                        "password_initial_value": {
                            "type": "string",
                            "__example__": "abc123"
                        },
                        "confirm_password_initial_value": {
                            "type": "string",
                            "__example__": "abc123"
                        }
                    },
                    "layout": {
                        "type": "SingleColumnLayout",
                        "children": [
                            {
                                "type": "TextHeading",
                                "text": "Please enter your details"
                            },
                            {
                                "type": "EmbeddedLink",
                                "text": "Already have an account?",
                                "on-click-action": {
                                    "name": "navigate",
                                    "next": {
                                        "type": "screen",
                                        "name": "LOGIN"
                                    },
                                    "payload": {
                                        "email_initial_value": "",
                                        "password_initial_value": ""
                                    }
                                }
                            },
                            {
                                "type": "Form",
                                "name": "form",
                                "init-values": {
                                    "first_name": "${data.first_name_initial_value}",
                                    "last_name": "${data.last_name_initial_value}",
                                    "email": "${data.email_initial_value}",
                                    "password": "${data.password_initial_value}",
                                    "confirm_password": "${data.confirm_password_initial_value}"
                                },
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
                                        "label": "Email Address",
                                        "input-type": "email",
                                        "required": true
                                    },
                                    {
                                        "type": "TextInput",
                                        "name": "password",
                                        "label": "Password",
                                        "input-type": "password",
                                        "required": true,
                                        "min-chars": 8,
                                        "max-chars": 16,
                                        "helper-text": "Password must contain at least one number"
                                    },
                                    {
                                        "type": "TextInput",
                                        "name": "confirm_password",
                                        "label": "Confirm Password",
                                        "input-type": "password",
                                        "required": true,
                                        "min-chars": 8,
                                        "max-chars": 16
                                    },
                                    {
                                        "type": "Footer",
                                        "label": "Done",
                                        "on-click-action": {
                                            "name": "data_exchange",
                                            "payload": {
                                                "first_name": "${form.first_name}",
                                                "last_name": "${form.last_name}",
                                                "email": "${form.email}",
                                                "password": "${form.password}",
                                                "confirm_password": "${form.confirm_password}"
                                            }
                                        }
                                    }
                                ]
                            }
                        ]
                    }
                },
                {
                    "id": "LOGIN",
                    "title": "Login",
                    "data": {
                        "email_initial_value": {
                            "type": "string",
                            "__example__": "john.doe@gmail.com"
                        },
                        "password_initial_value": {
                            "type": "string",
                            "__example__": "abc123"
                        }
                    },
                    "terminal": true,
                    "layout": {
                        "type": "SingleColumnLayout",
                        "children": [
                            {
                                "type": "TextHeading",
                                "text": "Please enter your details"
                            },
                            {
                                "type": "EmbeddedLink",
                                "text": "Don't have an account?",
                                "on-click-action": {
                                    "name": "navigate",
                                    "next": {
                                        "type": "screen",
                                        "name": "SIGN_UP"
                                    },
                                    "payload": {
                                        "email_initial_value": "",
                                        "password_initial_value": "",
                                        "confirm_password_initial_value": "",
                                        "first_name_initial_value": "",
                                        "last_name_initial_value": ""
                                    }
                                }
                            },
                            {
                                "type": "Form",
                                "name": "form",
                                "init-values": {
                                    "email": "${data.email_initial_value}",
                                    "password": "${data.password_initial_value}"
                                },
                                "children": [
                                    {
                                        "type": "TextInput",
                                        "name": "email",
                                        "label": "Email Address",
                                        "input-type": "email",
                                        "required": true
                                    },
                                    {
                                        "type": "TextInput",
                                        "name": "password",
                                        "label": "Password",
                                        "input-type": "password",
                                        "required": true
                                    },
                                    {
                                        "type": "Footer",
                                        "label": "Done",
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
                        ]
                    }
                },
                {
                    "id": "LOGIN_SUCCESS",
                    "title": "Success",
                    "terminal": true,
                    "layout": {
                        "type": "SingleColumnLayout",
                        "children": [
                            {
                                "type": "TextHeading",
                                "text": "Welcome to our store"
                            },
                            {
                                "type": "TextSubheading",
                                "text": "You are now logged in"
                            },
                            {
                                "type": "Form",
                                "name": "form",
                                "children": [
                                    {
                                        "type": "OptIn",
                                        "name": "stay_logged_in",
                                        "label": "Stay logged in"
                                    },
                                    {
                                        "type": "Footer",
                                        "label": "Done",
                                        "on-click-action": {
                                            "name": "complete",
                                            "payload": {
                                                "stay_logged_in": "${form.stay_logged_in}"
                                            }
                                        }
                                    }
                                ]
                            }
                        ]
                    }
                }
            ]
        }




Criar o flow é muito simples usando o método :meth:`~pywa.client.WhatsApp.create_flow`:

.. code-block:: python
    :linenos:

    from pywa import WhatsApp
    from pywa.types.flows import FlowCategory

    wa = WhatsApp(
        phone_id="1234567890",
        token="abcdefg",
        business_account_id="1234567890",  # the ID of the WhatsApp Business Account
    )

    flow_id = wa.create_flow(
        name="Sign Up Flow",
        categories=[FlowCategory.SIGN_IN, FlowCategory.SIGN_UP],
    )

Como vamos trocar dados com o nosso servidor, precisamos fornecer um URI de endpoint para o flow. Este é o URI que
o WhatsApp usará para enviar dados ao nosso servidor. Podemos fazer isso usando o método :meth:`~pywa.client.WhatsApp.update_flow_metadata`:

.. code-block:: python
    :linenos:

    wa.update_flow_metadata(
        flow_id=flow_id,
        endpoint_uri="https://my-server.com/sign-up-flow",
    )

Este endpoint deve, é claro, apontar para o nosso servidor. Podemos usar serveo, localtunnel ou uma ferramenta similar para expor nosso servidor à internet.

Por fim, vamos atualizar o JSON do flow com :meth:`~pywa.client.WhatsApp.update_flow_json`:

.. code-block:: python
    :linenos:

    from pywa.errors import FlowUpdatingError

    try:
        wa.update_flow_json(
            flow_id=flow_id,
            flow_json=SIGN_UP_FLOW_JSON,
        )
        print("Flow updated successfully")
    except FlowUpdatingError as e:
        print("Flow updating failed")
        print(wa.get_flow(flow_id=flow_id).validation_errors)

Armazenando Usuários
--------------------

Após a atualização do flow ser concluída com sucesso, podemos iniciar a lógica do servidor. Primeiro, precisamos de um repositório de usuários simples para armazená-los:

.. code-block:: python
    :linenos:

    import typing

    class UserRepository:
        def __init__(self):
            self._users = {}

        def create(self, email: str, details: dict[str, typing.Any]):
            self._users[email] = details

        def get(self, email: str) -> dict[str, typing.Any] | None:
            return self._users.get(email)

        def update(self, email: str, details: dict[str, typing.Any]):
            self._users[email] = details

        def delete(self, email: str):
            del self._users[email]

        def exists(self, email: str) -> bool:
            return email in self._users

        def is_password_valid(self, email: str, password: str) -> bool:
            return self._users[email]["password"] == password

    user_repository = UserRepository()  # create an instance of the user repository


Claro, em uma aplicação real, usaríamos um banco de dados real para armazenar os usuários (e nunca armazenamos senhas em texto puro...).

Enviando o Flow
---------------

Para enviar o flow, precisamos inicializar o cliente :class:`~pywa.client.WhatsApp` com alguns parâmetros específicos:

.. code-block:: python
    :caption: main.py
    :linenos:

    import fastapi
    from pywa import WhatsApp

    fastapi_app = fastapi.FastAPI()

    wa = WhatsApp(
        phone_id="1234567890",
        token="abcdefg",
        server=fastapi_app,
        callback_url="https://my-server.com",
        webhook_endpoint="/webhook",
        verify_token="xyz123",
        app_id=123,
        app_secret="zzz",
        business_private_key=open("private.pem").read(),
        business_private_key_password="abc123",
    )


A classe :class:`~pywa.client.WhatsApp` recebe alguns parâmetros:

- ``phone_id``: O ID do telefone da conta WhatsApp que estamos usando para enviar e receber mensagens
- ``token``: O token da conta WhatsApp que estamos usando para enviar e receber mensagens
- ``server``: O app FastAPI que criamos anteriormente, que será usado para registrar as rotas
- ``callback_url``: A URL que o WhatsApp usará para nos enviar atualizações
- ``webhook_endpoint``: O endpoint que o WhatsApp usará para nos enviar atualizações
- ``verify_token``: Usado pelo WhatsApp para verificar o servidor quando registramos o webhook
- ``app_id``: O ID do App WhatsApp, necessário para registrar a URL de callback
- ``app_secret``: O segredo do App WhatsApp, necessário para registrar a URL de callback
- ``business_private_key``: A chave privada da Conta Comercial do WhatsApp, necessária para descriptografar as requisições do flow (veja `aqui <../flows/overview.html#handling-flow-requests-and-responding-to-them>`_ para mais informações)
- ``business_private_key_password``: A senha da private_key, se ela tiver uma


Primeiro, vamos enviar o flow!

.. code-block:: python
    :linenos:

    from pywa.types import FlowButton
    from pywa.types.flows import FlowStatus, FlowActionType

    wa.send_message(
        to="1234567890",
        text="Welcome to our app! Click the button below to login or sign up",
        buttons=FlowButton(
            title="Sign Up",
            flow_id=flow_id,
            flow_token="5749d4f8-4b74-464a-8405-c26b7770cc8c",
            mode=FlowStatus.DRAFT,
            flow_action_type=FlowActionType.NAVIGATE,
            flow_action_screen="START",
        )
    )

Ok, vamos analisar isso:

Enviar um flow é muito simples. Enviamos uma mensagem de texto (ou imagem, vídeo, etc.) com um :class:`~pywa.types.callback.FlowButton`. O FlowButton contém as seguintes propriedades:

- ``title``: O título do botão (o texto que o usuário verá no botão)
- ``flow_id``: O ID do flow que queremos enviar
- ``mode``: O modo do flow. Estamos usando ``FlowStatus.DRAFT`` porque ainda estamos testando o flow. Quando estivermos prontos para publicar o flow, podemos alterar o modo para ``FlowStatus.PUBLISHED``
- ``flow_action_type``: A ação que será acionada quando o usuário clicar no botão. Neste caso, usamos ``FlowActionType.NAVIGATE`` para navegar para a tela ``START``
- ``flow_action_screen``: O nome da tela para a qual queremos navegar. Neste caso, usamos ``START``

- ``flow_token``: O token único para este flow específico.

Quando a requisição do flow é enviada ao nosso servidor, não sabemos qual flow e qual usuário é para a requisição. Só sabemos o token do flow.
Portanto, o token do flow é usado para nos dar algum contexto sobre a requisição. Podemos usar o token do flow para identificar o usuário e o flow.
O token do flow pode ser salvo em um banco de dados ou em cache de memória, e ser mapeado para o ID do usuário e o ID do flow (em casos onde você tem múltiplos flows rodando na sua aplicação).
E quando as requisições chegarem, você pode usar o token do flow para identificar o usuário e o flow e tomar as ações apropriadas para a requisição.

    O token do flow também pode ser usado para invalidar o flow, ao lançar a exceção FlowTokenNoLongerValid com uma error_message adequada.

Uma boa prática é gerar um token único para cada requisição de flow. Dessa forma, podemos ter certeza de que o token é único e que podemos identificar o usuário e o flow.
Você pode usar o módulo :mod:`uuid` para gerar um token único:

.. code-block:: python
    :linenos:

    import uuid

    flow_token = str(uuid.uuid4())


Depois que criamos a instância do WhatsApp e enviamos o flow, podemos começar a ouvir as requisições do flow:

.. code-block:: python
    :linenos:

    from pywa.types.flows import FlowRequest, FlowResponse

    @wa.on_flow_request("/sign-up-flow")
    def on_sign_up_request(_: WhatsApp, flow: FlowRequest) -> FlowResponse | None:
        if flow.has_error:
            logging.error("Flow request has error: %s", flow.data)
            return

        ...


O decorador :meth:`~pywa.client.WhatsApp.on_flow_request` recebe o URI do endpoint como parâmetro. Este é o endpoint que fornecemos ao atualizar os metadados do flow.
Portanto, se o URI do endpoint for ``https://my-server.com/sign-up-flow``, o URI do endpoint que estamos ouvindo é ``/sign-up-flow``.

    Sim, você pode apontar múltiplos flows para o mesmo URI de endpoint. Mas então você precisa encontrar uma forma de identificar o flow pelo token do flow.
    Recomendo criar um URI de endpoint único para cada flow.


Nossa função de callback ``on_sign_up_request`` recebe dois parâmetros:

- ``wa``: A instância da classe :class:`~pywa.client.WhatsApp`
- ``flow``: Um objeto :class:`FlowRequest`, que contém os dados da requisição do flow

A requisição do flow contém as seguintes propriedades:

- ``version``: A versão da API de dados do flow que a requisição está usando (você deve usar esta versão na resposta)
- ``flow_token``: O token do flow (o mesmo token que fornecemos ao enviar o flow)
- ``action``: O tipo de ação que acionou a requisição. ``FlowActionType.DATA_EXCHANGE`` no nosso caso.
- ``screen``: O nome da tela em que o usuário está atualmente (temos duas telas com ações de troca de dados, então precisamos saber em qual tela o usuário está)
- ``data``: Os dados que a ação enviou ao servidor (a propriedade ``payload`` da ação)


No início da função, verificamos se a requisição do flow tem um erro. Se tiver, registramos o erro e retornamos.

    Por padrão, se o flow tiver erro, o ``pywa`` ignorará o valor de retorno do callback e reconhecerá o erro.
    Esse comportamento pode ser alterado definindo o parâmetro ``acknowledge_errors`` como ``False`` no decorador ``on_flow_request``.

Tratando Requisições do Flow de Cadastro
-----------------------------------------

Agora, vamos tratar a requisição do flow. Podemos tratar todas as telas em um único bloco de código, mas por simplicidade, trataremos cada tela separadamente:

.. code-block:: python
    :linenos:

    @on_sign_up_request.on(
        action=FlowRequestActionType.DATA_EXCHANGE,
        screen="SIGN_UP",
        filters=filters.new(lambda _, request: user_repository.exists(request.data["email"])),
    )
    def if_already_registered(_: WhatsApp, request: FlowRequest) -> FlowResponse | None:
        return FlowResponse(
            version=request.version,
            screen="LOGIN",
            error_message="You are already registered. Please login",
            data={
                "email_initial_value": request.data["email"],
                "password_initial_value": request.data["password"],
            },
        )

    @on_sign_up_request.on(
        action=FlowRequestActionType.DATA_EXCHANGE,
        screen="SIGN_UP",
        filters=filters.new(lambda _, request: request.data["password"] != request.data["confirm_password"]),
    )
    def if_passwords_dont_match(_: WhatsApp, request: FlowRequest) -> FlowResponse | None:
        return FlowResponse(
            version=request.version,
            screen=request.screen,
            error_message="Passwords do not match",
            data={
                "first_name_initial_value": request.data["first_name"],
                "last_name_initial_value": request.data["last_name"],
                "email_initial_value": request.data["email"],
                "password_initial_value": "",
                "confirm_password_initial_value": "",
            },
        )

    @on_sign_up_request.on(
        action=FlowRequestActionType.DATA_EXCHANGE,
        screen="SIGN_UP",
        filters=filters.new(lambda _, request: not any(char.isdigit() for char in request.data["password"])),
    )
    def if_password_does_not_contain_number(
        _: WhatsApp, request: FlowRequest
    ) -> FlowResponse | None:
        return FlowResponse(
            version=request.version,
            screen=request.screen,
            error_message="Password must contain at least one number",
            data={
                "first_name_initial_value": request.data["first_name"],
                "last_name_initial_value": request.data["last_name"],
                "email_initial_value": request.data["email"],
                "password_initial_value": "",
                "confirm_password_initial_value": "",
            },
        )

    @on_sign_up_request.on(action=FlowRequestActionType.DATA_EXCHANGE, screen="SIGN_UP")
    def submit_signup(_: WhatsApp, request: FlowRequest) -> FlowResponse | None:
        user_repository.create(request.data["email"], request.data)
        return FlowResponse(
            version=request.version,
            screen="LOGIN",
            data={
                "email_initial_value": request.data["email"],
                "password_initial_value": "",
            },
        )


Então, o que está acontecendo aqui?

.. note::

    O decorador :meth:`~pywa.handlers.FlowRequestCallbackWrapper.on` foi adicionado na versão ``1.22.0``.
    Antes disso, você precisa tratar a ação e a tela na própria função (ou filtrar os dados manualmente).

    .. code-block:: python
        :linenos:

        @wa.on_flow_request("/sign-up-flow")
        def on_sign_up_request(_: WhatsApp, flow: FlowRequest) -> FlowResponse | None:
            if flow.action == FlowRequestActionType.DATA_EXCHANGE:
                if flow.screen == "SIGN_UP":
                    if user_repository.exists(flow.data["email"]):
                        ...
                    elif flow.data["password"] != flow.data["confirm_password"]:
                        ...
                    elif not any(char.isdigit() for char in flow.data["password"]):
                        ...
                    else:
                        ...
                elif flow.screen == "LOGIN":
                    ...
                elif flow.screen == "LOGIN_SUCCESS":
                    ...

Esta função trata a tela ``SIGN_UP``.

Precisamos verificar algumas coisas:

- Verificar se o usuário já está registrado. Se estiver, precisamos navegar para a tela ``LOGIN`` e mostrar uma mensagem de erro
- Verificar se a senha e a confirmação de senha coincidem. Se não coincidirem, navegamos novamente para a tela ``SIGN_UP`` e mostramos uma mensagem de erro
- Verificar se a senha contém pelo menos um número. Se não contiver, navegamos novamente para a tela ``SIGN_UP`` e mostramos uma mensagem de erro
- Se tudo estiver ok, criamos o usuário e navegamos para a tela ``LOGIN`` (com o endereço de e-mail já preenchido 😋)

    Agora você entende por que a tela ``SIGN_UP`` recebe valores iniciais? Porque não queremos que o usuário insira os dados novamente se houver um erro.
    Pelo mesmo motivo, a tela ``LOGIN`` também recebe valores iniciais, então quando o cadastro for concluído com sucesso, o usuário será navegado para a tela ``LOGIN`` com o endereço de e-mail já preenchido.

Tratando Requisições do Flow de Login
--------------------------------------

Agora, vamos tratar a tela ``LOGIN``:

.. code-block:: python
    :linenos:

    @on_sign_up_request.on(
        action=FlowRequestActionType.DATA_EXCHANGE,
        screen="LOGIN",
        filters=filters.new(lambda _, request: not user_repository.exists(request.data["email"])),
    )
    def if_not_registered(_: WhatsApp, request: FlowRequest) -> FlowResponse | None:
        return FlowResponse(
            version=request.version,
            screen="SIGN_UP",
            error_message="You are not registered. Please sign up",
            data={
                "first_name_initial_value": "",
                "last_name_initial_value": "",
                "email_initial_value": request.data["email"],
                "password_initial_value": "",
                "confirm_password_initial_value": "",
            },
        )

    @on_sign_up_request.on(
        action=FlowRequestActionType.DATA_EXCHANGE,
        screen="LOGIN",
        filters=filters.new(
            lambda _, request: not user_repository.is_password_valid(request.data["email"], request.data["password"])),
    )
    def if_incorrect_password(_: WhatsApp, request: FlowRequest) -> FlowResponse | None:
        return FlowResponse(
            version=request.version,
            screen=request.screen,
            error_message="Incorrect password",
            data={
                "email_initial_value": request.data["email"],
                "password_initial_value": "",
            },
        )

    @on_sign_up_request.on(action=FlowRequestActionType.DATA_EXCHANGE, screen="LOGIN")
    def login_success(_: WhatsApp, request: FlowRequest) -> FlowResponse | None:
        return FlowResponse(
            version=request.version,
            screen="LOGIN_SUCCESS",
            data={},
        )


A tela ``LOGIN`` é muito semelhante à tela ``SIGN_UP``. Precisamos verificar algumas coisas:

- Verificar se o usuário está registrado. Se não estiver, precisamos navegar para a tela ``SIGN_UP`` e mostrar uma mensagem de erro
- Verificar se a senha está correta. Se não estiver, precisamos navegar novamente para a tela ``LOGIN`` e mostrar uma mensagem de erro
- Se tudo estiver ok, navegamos para a tela ``LOGIN_SUCCESS``

Tratando as Requisições do Flow
--------------------------------

Vamos modificar nossa função de callback ``on_sign_up_request`` para tratar as telas ``SIGN_UP`` e ``LOGIN``:

.. code-block:: python
    :linenos:

    @wa.on_flow_request("/sign-up-flow")
    def on_sign_up_request(_: WhatsApp, flow: FlowRequest) -> FlowResponse | None:
        if flow.has_error:  # you can handle this also separately by registering another callback with @on_sign_up_request.on_errors
            logging.error("Flow request has error: %s", flow.data)
            return


Tratando a Conclusão do Flow
-----------------------------

A tela ``LOGIN_SUCCESS`` conclui o flow, então não precisamos fazer nada aqui. Em vez disso, precisamos tratar a conclusão do flow:

.. code-block:: python
    :linenos:

    @wa.on_flow_completion()
    def handle_flow_completion(_: WhatsApp, flow: FlowCompletion):
        print("Flow completed successfully")
        print(flow.token)
        print(flow.response)

Agora, em uma aplicação real, este é o momento de marcar o usuário como conectado e permitir que ele realize ações em sua conta.
Você também pode implementar algum tipo de gerenciamento de sessão, para que o usuário permaneça conectado por um determinado período e depois precise fazer login novamente.

Executando o Servidor
----------------------

A última coisa que precisamos fazer é executar o servidor:

.. code-block:: bash

    fastapi dev wa.py

O Que Vem a Seguir?
-------------------

Agora que você sabe como criar e enviar um flow, pode tentar adicionar as seguintes funcionalidades ao flow:

- Uma tela ``FORGOT_PASSWORD``, que permite ao usuário redefinir sua senha caso a tenha esquecido
- Uma tela ``LOGIN_SUCCESS`` mais detalhada, que mostra o nome do usuário, endereço de e-mail e outros detalhes
- Tente adicionar uma imagem bonita à tela ``START`` para torná-la mais atraente
- Uma tela ``LOGOUT``, que permite ao usuário sair da conta
- Permitir que o usuário altere seu e-mail e senha
- Permitir que o usuário feche o flow em qualquer tela
