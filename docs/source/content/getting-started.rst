⚙️ Primeiros Passos
===================

⬇️ Instalação
--------------

- **Instalar usando pip:**

.. code-block:: bash

    pip3 install -U pywa

- **Instalar da fonte (versão mais recente):**

.. code-block:: bash

    git clone https://github.com/david-lev/pywa.git
    cd pywa && pip3 install -U .

- **Para recursos de webhook (FastAPI ou Flask):**

.. code-block:: bash

    pip3 install -U "pywa[fastapi]"
    pip3 install -U "pywa[flask]"

- **Para recursos de Flow com criptografia/descriptografia padrão:**

.. code-block:: bash

    pip3 install -U "pywa[cryptography]"

================================

Criar um Aplicativo WhatsApp
-----------------------------

Já tem um aplicativo? Pule para `Configurar o Aplicativo <#id1>`_.

Para usar a WhatsApp Cloud API, você precisa de um Aplicativo do Facebook.
Se você não tem uma conta de desenvolvedor do Facebook, `registre-se aqui <https://developers.facebook.com/>`_.

1. Acesse `Meta for Developers > My Apps <https://developers.facebook.com/apps/>`_ e crie um novo aplicativo.
   - Ou clique `aqui <https://developers.facebook.com/apps/create/?show_additional_prod_app_info=false>`_ para ir diretamente à página de criação do aplicativo.

2. Selecione **Business** como o tipo de aplicativo e clique em **Next**.

.. toggle::

    .. image:: ../../_static/guides/select-app-type.webp
       :width: 600
       :alt: Select app type
       :align: center

4. Preencha o nome do aplicativo e o e-mail, depois clique em **Create App**.

.. toggle::

    .. image:: ../../_static/guides/fill-app-details.webp
       :width: 600
       :alt: Fill app details
       :align: center

5. Em **Add products to your app**, pesquise por **WhatsApp** e clique em **Set Up**.

.. toggle::

    .. image:: ../../_static/guides/setup-whatsapp-product.webp
       :width: 600
       :alt: Setup WhatsApp product
       :align: center

6. Selecione uma **Meta Business Account**, aceite os termos e clique em **Submit**.
   Se você não tem uma Business Account, será necessário criar uma.

.. toggle::

    .. image:: ../../_static/guides/select-meta-business-account.webp
       :width: 600
       :alt: Select meta business
       :align: center

--------------------

Configurar o Aplicativo
-----------------------

Já tem seu **Phone ID** e **Token**? Pule para `Enviar uma Mensagem <#id2>`_.

7. No menu à esquerda (em **Products**), expanda **WhatsApp** e clique em **API Setup**.

.. toggle::

    .. image:: ../../_static/guides/api-setup.webp
       :width: 600
       :alt: API setup
       :align: center

- Copie o **Temporary access token** (válido por 24h) e o **Phone number ID**.

.. note::

    Saiba `como criar um token permanente <https://developers.facebook.com/docs/whatsapp/business-management-api/get-started>`_.

.. attention::

    If you haven’t connected a real phone number, you can use a test number provided by Meta.
    You can send messages to up to 5 allowed numbers. Add them in the **Manage phone number list**.

    .. toggle::

        .. image:: ../../_static/guides/verify-phone-number-for-testing.webp
           :width: 600
           :alt: Test number setup
           :align: center

--------------------

Enviar uma Mensagem
-------------------

Agora você tem seu ``phone_id`` e ``token``. Você pode enviar mensagens:

.. code-block:: python

    from pywa import WhatsApp

    wa = WhatsApp(
        phone_id='YOUR_PHONE_ID',  # from API Setup
        token='YOUR_TOKEN'         # from API Setup
    )

.. code-block:: python

    wa.send_message(
        to='PHONE_NUMBER_TO_SEND_TO',
        text='Hi! This message was sent from pywa!'
    )

    wa.send_image(
        to='PHONE_NUMBER_TO_SEND_TO',
        image='https://www.rd.com/wp-content/uploads/2021/04/GettyImages-1053735888-scaled.jpg'
    )

.. note::

    - O parâmetro ``to`` deve incluir o código do país, por exemplo, ``+972123456789`` ou ``16315551234``.
      Leia mais sobre `formatos de número de telefone aqui <https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-messages#phone-number-formats>`_.
    - Para **Números de Teste**, adicione os destinatários à lista de números permitidos.
    - Mensagens de forma livre só podem ser recebidas se o destinatário enviou mensagem para o seu número nas últimas 24h.
      Veja a `política do WhatsApp <https://business.whatsapp.com/policy>`_.

--------------------

Início Rápido
-------------

Aqui está uma visão geral rápida do pacote ``pywa``:

- `WhatsApp <client/overview.html>`_: Client principal para enviar/receber mensagens, gerenciar configurações de perfil/negócio e registrar callbacks.
- `Handlers <handlers/overview.html>`_: Registre callbacks para lidar com atualizações recebidas (mensagens, callbacks e muito mais).
- `Listeners <listeners/overview.html>`_: Ouça atualizações de usuários recebidas.
- `Filters <filters/overview.html>`_: Filtre e trate atualizações específicas, por exemplo, mensagens de texto contendo “Hello”.
- `Updates <updates/overview.html>`_: Explore diferentes tipos de atualização, seus atributos e uso.
- `Flows <flows/overview.html>`_: Crie, atualize e envie flows.
- `Errors <errors/overview.html>`_: Aprenda sobre os erros do pacote e como tratá-los.
- `Examples <examples/overview.html>`_: Veja exemplos práticos de uso.
