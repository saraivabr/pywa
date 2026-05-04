📄 Templates
==============

.. currentmodule:: pywa.types.templates

No universo das mensagens WhatsApp, os templates são estruturas de mensagem pré-aprovadas que as empresas podem usar para iniciar conversas com usuários. Esses templates são essenciais para enviar notificações, atualizações ou qualquer mensagem que exija aprovação prévia do WhatsApp.

Pense nos templates como modelos de mensagem reutilizáveis que garantem que suas comunicações sejam consistentes, conformes e prontas para engajar seu público. Eles podem incluir vários componentes, como cabeçalhos, corpos, rodapés e botões, permitindo mensagens ricas e interativas.

PyWa oferece uma interface abrangente e intuitiva para criar, gerenciar e enviar esses templates de forma integrada. Seja para enviar ofertas promocionais, atualizações de conta ou códigos de autenticação, o sistema de templates do PyWa garante que suas mensagens sejam estruturadas, consistentes e em conformidade com as diretrizes do WhatsApp.

- De `developers.facebook.com <https://developers.facebook.com/docs/whatsapp/business-management-api/message-templates>`_:

    Templates são ativos da conta WhatsApp Business que podem ser enviados em mensagens de template via Cloud API ou Marketing Messages Lite API. Mensagens de template são o único tipo de mensagem que pode ser enviado a usuários do WhatsApp fora de uma `janela de atendimento ao cliente <https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-messages/#customer-service-windows>`_, por isso os templates são comumente usados ao enviar mensagens em massa ou quando você precisa enviar uma mensagem a um usuário sem que uma janela de atendimento esteja aberta entre vocês.

Definindo o Template
---------------------

Para criar um template, você precisa definir sua estrutura usando a classe :class:`Template` do módulo :mod:`pywa.types.templates`. Isso envolve especificar o nome, idioma, categoria, formato de parâmetro e os componentes que compõem o template.

No exemplo abaixo, criamos um template simples de confirmação de pedido com cabeçalho, corpo, rodapé e botões.

.. code-block:: python
    :caption: order_confirmation_template.py
    :linenos:

    from pywa.types.templates import *

    order_confirmation = Template(
        name="order_confirmation",
        language=TemplateLanguage.ENGLISH_US,
        category=TemplateCategory.MARKETING,
        parameter_format=ParamFormat.NAMED,
        components=[
            HeaderText(text="Order Confirmation"),
            BodyText(
                "Hi {{name}}, your order {{order_id}} has been confirmed and will be delivered by {{delivery_date}}.",
                name="John Doe",
                order_id=12345,
                delivery_date="August 5, 2025",
            ),
            Buttons(
                buttons=[
                    QuickReplyButton(
                        text="Contact Support",
                    ),
                    URLButton(
                        text="Track Order",
                        url="https://www.example.com/track-order?order_id={{1}}",
                        example="12345",
                    ),
                ]
            ),
            FooterText(text="Powered by Pywa"),
        ],
    )

Neste exemplo, definimos um template chamado "order_confirmation" em inglês (EUA) para fins de marketing.
Usamos um formato de parâmetro nomeado, o que nos permite usar nomes descritivos para os parâmetros, como ``name``, ``order_id`` e ``delivery_date`` no texto do corpo.
O template inclui um cabeçalho com texto, um corpo com parâmetros dinâmicos, botões de resposta rápida e URL, e um rodapé com informações adicionais.


.. figure:: ../../../../_static/examples/order-confirmation-template.png
    :align: center

    É assim que o template ficará no WhatsApp após o envio.


Componentes do Template
------------------------

.. list-table::
   :widths: 10 60
   :header-rows: 1

   * - Categoria
     - Tipos
   * - Cabeçalhos
     - :class:`HeaderText`,
       :class:`HeaderImage`,
       :class:`HeaderVideo`,
       :class:`HeaderDocument`,
       :class:`HeaderGIF`,
       :class:`HeaderLocation`,
       :class:`HeaderProduct`
   * - Corpos
     - :class:`BodyText`,
       :class:`AuthenticationBody`
   * - Rodapés
     - :class:`FooterText`,
       :class:`AuthenticationFooter`
   * - Botões
     - :class:`Buttons`,
       :class:`QuickReplyButton`,
       :class:`URLButton`,
       :class:`CopyCodeButton`,
       :class:`CatalogButton`,
       :class:`FlowButton`,
       :class:`CallPermissionRequestButton`,
       :class:`VoiceCallButton`,
       :class:`MPMButton`,
       :class:`SPMButton`,
       :class:`OneTapOTPButton`,
       :class:`ZeroTapOTPButton`,
       :class:`CopyCodeOTPButton`
   * - Outros
     - :class:`Carousel`,
       :class:`LimitedTimeOffer`


Criando o Template
-------------------

Após definir seu template, você pode criá-lo usando o método :meth:`~pywa.client.WhatsApp.create_template`:

.. code-block:: python
    :caption: create_template.py
    :linenos:

    from pywa import WhatsApp

    wa = WhatsApp(business_account_id=...)

    wa.create_template(order_confirmation)
    # CreatedTemplate(id='...', category=TemplateCategory.MARKETING, status=TemplateStatus.PENDING)

Após criar um template, você precisa aguardar a aprovação do WhatsApp. Esse processo pode levar algum tempo, e você pode verificar o status do seu template usando o método :meth:`~pywa.client.WhatsApp.get_template` ou acessando o `WhatsApp Manager Dashboard <https://business.facebook.com/latest/whatsapp_manager/>`_ em Gerenciar templates.

.. seealso::
    Você também pode usar o método :meth:`~pywa.client.WhatsApp.get_templates` para recuperar uma lista de todos os templates associados à sua conta empresarial.

Você também pode usar :meth:`~pywa.types.templates.CreatedTemplate.wait_until_approved` para aguardar a aprovação do template, o que bloqueará a execução até que o template seja aprovado ou rejeitado.

.. code-block:: python
    :caption: wait_until_approved.py
    :linenos:

    from pywa import WhatsApp

    wa = WhatsApp(business_account_id=...)

    template = wa.create_template(order_confirmation)
    template.wait_until_approved()

Ou você pode lidar com o status do template usando o decorador :meth:`~pywa.client.WhatsApp.on_template_status_update`, que permite definir uma função de callback que será chamada sempre que o status do template mudar:

.. code-block:: python
    :caption: on_template_status_update.py
    :linenos:

    from pywa import WhatsApp, types
    from pywa.types import TemplateStatusUpdate
    from pywa.types.templates import TemplateStatus

    wa = WhatsApp(business_account_id=..., token=...)

    @wa.on_template_status_update
    def handle_template_status_update(_: WhatsApp, update: TemplateStatusUpdate):
        if update.new_status == TemplateStatus.APPROVED:
            print(f"Template {update.template_id} approved!")
        elif update.new_status == TemplateStatus.REJECTED:
            print(f"Template {update.template_id} rejected: {update.reason}")


Enviando o Template
--------------------

Assim que seu template for aprovado, você pode enviá-lo usando o método :meth:`~pywa.client.WhatsApp.send_template`.

.. code-block:: python
    :caption: send_template.py
    :linenos:
    :emphasize-lines: 10

    from pywa import WhatsApp
    from pywa.types.templates import *

    wa = WhatsApp(phone_id=..., token=...)

    wa.send_template(
        to="972123456789",
        name="order_confirmation",
        language=TemplateLanguage.ENGLISH_US,
        params=[...]
    )

Agora, a lista ``params`` deve conter os valores para os componentes definidos no template.

A melhor prática é usar o objeto Template criado anteriormente como referência para os parâmetros, garantindo que os valores correspondam ao formato esperado:

.. code-block:: python
    :caption: send_template_with_object.py
    :linenos:
    :emphasize-lines: 11, 19, 22, 37-38, 40-42

    from pywa import WhatsApp
    from pywa.types.templates import *

    order_confirmation = Template(
        name="order_confirmation",
        language=TemplateLanguage.ENGLISH_US,
        category=TemplateCategory.MARKETING,
        parameter_format=ParamFormat.NAMED,
        components=[
            HeaderText(text="Order Confirmation"),
            bdy := BodyText(
                "Hi {{name}}, your order {{order_id}} has been confirmed and will be delivered by {{delivery_date}}.",
                name="John Doe",
                order_id=12345,
                delivery_date="August 5, 2025",
            ),
            Buttons(
                buttons=[
                    qrb := QuickReplyButton(
                        text="Contact Support",
                    ),
                    urlb := URLButton(
                        text="Track Order",
                        url="https://www.example.com/track-order?order_id={{1}}",
                        example="12345",
                    ),
                ]
            ),
            FooterText(text="Powered by Pywa"),
        ],
    )

    wa = WhatsApp(phone_id=..., token=...)

    wa.send_template(
        to="972123456789",
        name=order_confirmation.name,
        language=order_confirmation.language,
        params=[
            bdy.params(name="Jane Doe", order_id=67890, delivery_date=DateTime(fallback_value="September 10, 2025")),
            qrb.params(callback_data="contact-support", index=0),
            urlb.params(url_variable="67890", index=1),
        ],
    )

Como você pode ver, usamos o método `params` de cada componente para gerar os parâmetros necessários para o template.

.. tip::

    Usar ``params`` na instância garante que os parâmetros estejam corretamente formatados e correspondam aos tipos esperados, reduzindo o risco de erros ao enviar o template. Mas às vezes você não tem acesso às instâncias dos componentes do template (por ex. ao criar pelo WhatsApp Manager Dashboard) e precisa criar os parâmetros manualmente. Nesse caso, você ainda pode usar o método ``params`` em cada componente para criar os parâmetros diretamente, como mostrado abaixo:

    .. code-block:: python
        :caption: send_template_without_object.py
        :linenos:
        :emphasize-lines: 11-13

        from pywa import WhatsApp
        from pywa.types.templates import *

        wa = WhatsApp(phone_id=..., token=...)

        wa.send_template(
            to="972123456789",
            name="order_confirmation",
            language=TemplateLanguage.ENGLISH_US,
            params=[
                BodyText.params(name="Jane Doe", order_id=67890, delivery_date=DateTime(fallback_value="September 10, 2025")),
                QuickReplyButton.params(callback_data="contact-support", index=0),
                URLButton.params(url_variable="67890", index=1),
            ],
        )


Templates com Mídia
--------------------

Ao criar templates que incluam mídia, como imagens, vídeos ou documentos, você precisa fornecer um exemplo da mídia na definição do template. Isso é feito usando o parâmetro `example` nos componentes de mídia.

.. note::

    Os exemplos de mídia são automaticamente enviados via `Upload Resumable API <https://developers.facebook.com/docs/graph-api/guides/upload>`_, o que requer o fornecimento de ``app_id`` ao inicializar o cliente WhatsApp. Se você não fornecer o ``app_id``, precisará fazer o upload da mídia manualmente usando a Upload Resumable API antes de criar o template e fornecer o identificador do arquivo no parâmetro `example`.

.. code-block:: python
    :caption: media_template.py
    :linenos:
    :emphasize-lines: 8

    from pywa.types.templates import *

    media_template = Template(
        name="media_template",
        language=TemplateLanguage.ENGLISH_US,
        category=TemplateCategory.MARKETING,
        components=[
            HeaderImage(example="https://www.example.com/image-example.jpg"),
            BodyText(text="Check out our new product!"),
            FooterText(text="Visit our website for more details."),
        ],
    )

O parâmetro ``example`` pode ser uma URL, caminho de arquivo, :class:`~pywa.types.media.Media` ou bytes. Essa mídia de exemplo será usada pelo WhatsApp para verificar o template e garantir que ele atenda às diretrizes deles.

.. tip::

    PyWa faz o upload da mídia automaticamente ao criar o template e armazena em cache o identificador do arquivo (da Upload Resumable API) no objeto de cabeçalho. Isso significa que, se você criar vários templates com o mesmo exemplo de mídia, pode reutilizar o mesmo objeto de mídia para evitar uploads repetidos da mesma mídia. Isso economizará tempo e recursos.


    .. code-block:: python
        :caption: media_template_with_cache.py
        :linenos:
        :emphasize-lines: 8, 16

        from pywa import WhatsApp
        from pywa.types.media import Media
        from pywa.types.templates import *

        wa = WhatsApp(token=..., business_account_id=..., app_id=...)

        image = pathlib.Path("path/to/image.jpg")
        header_image = HeaderImage(example=image)

        for lang in [TemplateLanguage.ENGLISH, TemplateLanguage.ENGLISH_US, TemplateLanguage.ENGLISH_UK]:
            media_template = Template(
                name="media_template",
                language=lang,
                category=TemplateCategory.MARKETING,
                components=[
                    header_image,  # Reuse the same header image
                    BodyText(text="Check out our new product!"),
                    FooterText(text="Visit our website for more details."),
                ],
            )

            wa.create_template(media_template)


    O mesmo se aplica a outros componentes de mídia (:class:`HeaderVideo` e :class:`HeaderDocument`): você pode reutilizar o mesmo objeto de mídia em vários templates para evitar uploads repetidos.


Ao enviar um template com mídia, você pode usar a mesma abordagem de antes, mas agora precisa fornecer a mídia como parâmetro no template:

.. code-block:: python
    :caption: send_media_template.py
    :linenos:
    :emphasize-lines: 6, 17

    media_template = Template(
        name="media_template",
        language=TemplateLanguage.ENGLISH_US,
        category=TemplateCategory.MARKETING,
        components=[
            hi := HeaderImage(example="https://www.example.com/image-example.jpg"),
            BodyText(text="Check out our new product!"),
            FooterText(text="Visit our website for more details."),
        ],
    )

    wa.send_template(
        to="972123456789",
        name=media_template.name,
        language=media_template.language,
        params=[
            hi.params(image="https://www.my-cdn.com/image.jpg"),
        ],
    )

.. tip::

    Se você estiver enviando o mesmo template várias vezes, pode armazenar em cache o objeto de mídia e reutilizá-lo em múltiplos envios de template. Isso economizará tempo e recursos, pois a mídia não será enviada novamente.

    .. code-block:: python
        :caption: send_media_template_with_cache.py
        :linenos:
        :emphasize-lines: 11, 17, 24

        from pywa import WhatsApp
        from pywa.types.templates import *

        wa = WhatsApp(token=..., business_account_id=..., app_id=...)

        media_template = Template(
            name="media_template",
            language=TemplateLanguage.ENGLISH_US,
            category=TemplateCategory.MARKETING,
            components=[
                hi := HeaderImage(example="https://www.example.com/image-example.jpg"),
                BodyText(text="Check out our new product!"),
                FooterText(text="Visit our website for more details."),
            ],
        )

        hi_param = hi.params(image=pathlib.Path("path/to/image.jpg"))

        for phone_number in ["972123456789", "972987654321", "972456789123"]:
            wa.send_template(
                to=phone_number,
                name=media_template.name,
                language=media_template.language,
                params=[hi_param],  # Reuse the same header image parameter
            )


Templates de Autenticação
--------------------------

Templates de autenticação são um tipo especial de template usado para enviar códigos de autenticação aos usuários. Se você precisar enviar OTPs (One-Time Passwords) para que os usuários verifiquem sua identidade ou concluam uma transação em outro aplicativo, pode usar templates de autenticação.

Criar um template de autenticação é semelhante a criar um template comum, mas inclui componentes específicos para autenticação, como :class:`AuthenticationBody` e :class:`AuthenticationFooter`.

.. code-block:: python
    :caption: authentication_template.py
    :linenos:
    :emphasize-lines: 6-15

    from pywa.types.templates import *

    auth_template = Template(
        name="auth_code",
        language=TemplateLanguage.ENGLISH_US,
        category=TemplateCategory.AUTHENTICATION,
        components=[
            AuthenticationBody(add_security_recommendation=True),
            AuthenticationFooter(code_expiration_minutes=5),
            Buttons(
                buttons=[
                    # An OTP Button
                ],
            ),
        ],
    )

O botão de OTP pode ser de um dos seguintes tipos:

- :class:`OneTapOTPButton`: Um botão que permite ao usuário tocar e preencher automaticamente o código OTP no aplicativo:

    .. code-block:: python
        :caption: one_tap_otp_button.py
        :linenos:

        from pywa.types.templates import *

        otp_button = OneTapOTPButton(
            text="Autofill Code",
            autofill_text="Autofill",
            supported_apps=[
                OTPSupportedApp(
                    package_name="com.example.myapp",
                    signature_hash="12345678901"
                ),
            ],
        )

.. figure:: ../../../../_static/examples/one-tap-auth-template.webp
    :align: center
    :width: 50%


--

- :class:`ZeroTapOTPButton`: Um botão que permite ao usuário receber o código OTP sem nenhuma interação.

    .. code-block:: python
        :caption: zero_tap_otp_button.py
        :linenos:

        from pywa.types.templates import *

        otp_button = ZeroTapOTPButton(
            text="Autofill Code",
            autofill_text="Autofill",
            zero_tap_terms_accepted=5,
            supported_apps=[
                OTPSupportedApp(
                    package_name="com.example.myapp",
                    signature_hash="12345678901"
                ),
            ],
        )

.. figure:: ../../../../_static/examples/zero-tap-auth-template.png
    :align: center
    :width: 50%

--

- :class:`CopyCodeOTPButton`: Um botão que permite ao usuário copiar o código OTP para a área de transferência e usá-lo em outro aplicativo.

    .. code-block:: python
        :caption: copy_code_otp_button.py
        :linenos:

        from pywa.types.templates import *

        otp_button = CopyCodeOTPButton()


.. figure:: ../../../../_static/examples/copy-code-auth-template.webp
    :align: center
    :width: 50%

--

Ao enviar um template de autenticação, você precisa fornecer o código OTP como parâmetro para o :class:`AuthenticationBody` e para o botão OTP que você está usando.

.. code-block:: python
    :caption: send_authentication_template.py
    :linenos:
    :emphasize-lines: 11-12

    from pywa import WhatsApp
    from pywa.types.templates import *

    wa = WhatsApp(phone_id=..., token=...)

    wa.send_template(
        to="972123456789",
        name="auth_code",
        language=TemplateLanguage.ENGLISH_US,
        params=[
            AuthenticationBody.params(otp="123456"),
            OneTapOTPButton.params(otp="123456"),
        ],
    )

.. tip::

    Os textos dos templates de autenticação são fixos e não podem ser editados (exceto o título do botão), então o WhatsApp permite criar e atualizar vários templates de autenticação com o mesmo nome. Isso significa que você pode criar vários templates de autenticação com o mesmo nome, mas para idiomas diferentes.

    Para criar um template de autenticação para vários idiomas, você pode usar o método :meth:`~pywa.client.WhatsApp.upsert_authentication_template`, que permite criar ou atualizar um template de autenticação para vários idiomas de uma vez.

    .. code-block:: python
        :caption: create_authentication_template_multiple_languages.py
        :linenos:

        from pywa import WhatsApp
        from pywa.types.templates import *

        wa = WhatsApp(business_account_id=..., token=...)

        templates = wa.upsert_authentication_template(
            name='one_tap_authentication',
            languages=[TemplateLanguage.ENGLISH_US, TemplateLanguage.FRENCH, TemplateLanguage.SPANISH],
            otp_button=OneTapOTPButton(supported_apps=...),
            add_security_recommendation=True,
            code_expiration_minutes=5,
        )
        for template in templates:
            print(f'Template {template.id} created with status {template.status}')

Biblioteca de Templates
------------------------

PyWa também oferece uma forma fácil de criar templates a partir da Biblioteca de Templates. A biblioteca contém uma coleção de templates predefinidos que você pode usar para criar e enviar mensagens rapidamente sem precisar defini-los do zero.

De `developers.facebook.com <https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-message-templates/template-library>`_:

    .. image:: ../../../../_static/guides/template-library.webp
        :alt: Template Library
        :width: 100%

    A Biblioteca de Templates torna mais rápido e fácil para as empresas criarem templates utilitários para casos de uso comuns, como lembretes de pagamento, atualizações de entrega — e templates de autenticação para casos comuns de verificação de identidade.

    Esses templates pré-escritos já foram categorizados como utilitário ou autenticação. Os templates da biblioteca contêm conteúdo fixo que não pode ser editado e parâmetros que você pode adaptar para informações específicas da empresa ou do usuário.

    Você pode navegar e criar templates usando a Biblioteca de Templates no WhatsApp Manager ou de forma programática via API.

Para criar um template da biblioteca, você precisa obter o nome do template da biblioteca e passá-lo para o método :meth:`~pywa.client.WhatsApp.create_template`:

.. code-block:: python
    :caption: create_template_from_library.py
    :linenos:

    from pywa import WhatsApp

    wa = WhatsApp(business_account_id=..., token=...)

    # Define the template
    order_update = LibraryTemplate(
        name="order_update",
        library_template_name="order_update_no_cta_1",
        category=TemplateCategory.UTILITY,
        language=TemplateLanguage.ENGLISH_US,
    )

    # Create the template
    wa.create_template(order_update)
    # CreatedTemplate(id='...', category=TemplateCategory.UTILITY, status=TemplateStatus.PENDING)


Se o template da biblioteca requer parâmetros, você precisa fornecê-los ao criar o template. Os parâmetros devem corresponder aos definidos no template da biblioteca:

.. code-block:: python
    :caption: create_template_from_library_with_params.py
    :linenos:

    from pywa import WhatsApp
    from pywa.types.templates import *

    wa = WhatsApp(business_account_id=..., token=...)

    # Define the template with parameters
    order_update = LibraryTemplate(
        name="order_update",
        library_template_name="order_update_1",
        category=TemplateCategory.UTILITY,
        language=TemplateLanguage.ENGLISH_US,
        library_template_button_inputs=[
            URLButton.library_input(
                base_url="https://www.example.com/track-order/{{1}}",
                url_suffix_example="https://www.example.com/track-order/12345",
            ),
        ]
    )

    # Create the template
    wa.create_template(order_update)
    # CreatedTemplate(id='...', category=TemplateCategory.UTILITY, status=TemplateStatus.PENDING)

.. toctree::
    types
