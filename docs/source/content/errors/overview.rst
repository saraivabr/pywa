⚠️ Errors
==========

.. currentmodule:: pywa.errors

Exceções em ``pywa`` são um mecanismo fundamental para informar **o que deu errado e por quê**.
Elas aparecem de duas formas principais:

1. **Exceções lançadas** — quando algo falha imediatamente (por ex., parâmetros inválidos).
2. **Erros retornados** — quando a API relata um erro de forma assíncrona por meio de uma atualização de status de mensagem.

------------------------------

Exemplo Básico
--------------

A maioria das exceções é lançada diretamente quando você tenta uma ação inválida:

.. code-block:: python
    :emphasize-lines: 8-9, 11

    import logging
    from pywa import WhatsApp, types, errors

    wa = WhatsApp(...)

    try:
        wa.send_message(..., buttons=[
            types.Button(title="click 1", callback_data="click"),
            types.Button(title="click 2", callback_data="click"),  # ⚠️duplicate callback_data
        ])
    except errors.InvalidParameter as e:
        logging.error(f"Duplicated `callback_data` in buttons: {e}")

------------------------------

Erros de Status de Mensagem
----------------------------

Alguns erros **não são lançados imediatamente**, mas aparecem como parte de uma atualização :class:`~pywa.types.message_status.MessageStatus`.

Por exemplo:

- Enviar uma mensagem não-template **fora da janela de conversa de 24h** →
  :class:`~pywa.errors.ReEngagementMessage`
- Enviar mídia inválida (tipo de arquivo incorreto, muito grande, URL inválida, etc.) →
  :class:`~pywa.errors.MediaUploadError`

Esses erros surgem na **atualização de status** em vez de lançar uma exceção diretamente.

Por isso é **importante sempre registrar um handler** para status de mensagens com falha:

.. code-block:: python
    :emphasize-lines: 6

    import logging
    from pywa import WhatsApp, types, filters

    wa = WhatsApp(...)

    @wa.on_message_status(filters.failed)
    def handle_failed_message(client: WhatsApp, status: types.MessageStatus):
        logging.error("Message failed to send to %s: %s",
            status.sender, status.error
        )

------------------------------

Tratando Erros Específicos
--------------------------

Você também pode filtrar e tratar tipos específicos de erros:

.. code-block:: python
    :linenos:
    :emphasize-lines: 18, 24, 31

    import logging
    from pywa import WhatsApp, filters, errors

    wa = WhatsApp(...)

    wa.send_message(to="972501234567", text="Hello")  # 24h window closed
    wa.send_image(  # nonexistent image
        to="972501234567",
        image="https://example.com/this-image-does-not-exist.jpg",
        caption="Not found"
    )
    wa.send_document(  # file too large
        to="972501234567",
        document="https://example.com/document-size-is-too-big.pdf",
        filename="big.pdf"
    )

    @wa.on_message_status(filters.failed_with(errors.ReEngagementMessage))
    def handle_failed_reengagement(client: WhatsApp, status: types.MessageStatus):
        logging.error("Message failed to send to %s: %s",
            status.from_user.wa_id, status.error
        )

    @wa.on_status_message(filters.failed_with(errors.MediaUploadError))
    def handle_failed_sent_media(client: WhatsApp, status: types.MessageStatus):
        logging.error("Message failed to send to %s: %s",
            status.from_user.wa_id, status.error
        )
        status.reply_text("Sorry, I can't upload this file")

    @wa.on_status_message(filters.failed_with(errors.MediaDownloadError))
    def handle_failed_received_media(client: WhatsApp, status: types.MessageStatus):
        logging.error("Got a media download error from %s: %s",
            status.from_user.wa_id, status.error
        )
        status.reply_text("Sorry, I can't download this file")

------------------------------

Erros Recebidos (Mensagens Não Suportadas)
------------------------------------------

Se um usuário enviar um tipo de mensagem não suportado (por ex., enquete), você receberá uma :class:`~pywa.types.Message` com tipo :class:`~pywa.types.MessageType.UNSUPPORTED` e um erro de :class:`~UnsupportedMessageType`.

.. code-block:: python
    :emphasize-lines: 5

    from pywa import WhatsApp, types, filters

    wa = WhatsApp(...)

    @wa.on_message(filters.unsupported)
    def handle_unsupported_message(client: WhatsApp, msg: types.Message):
        msg.reply_text("Sorry, I don't support this message type yet")

------------------------------

Capturando Todas as Exceções
-----------------------------

Como todas as exceções herdam de :class:`~WhatsAppError`, você pode capturar tudo com um único bloco:

.. code-block:: python
    :linenos:
    :emphasize-lines: 7

    from pywa import WhatsApp, errors

    wa = WhatsApp(...)

    try:
        wa.send_message(...)
    except errors.WhatsAppError as e:
        print(f"Error: {e}")

------------------------------

Exceção Base
-------------

.. autoclass:: WhatsAppError()
    :show-inheritance:

------------------------------

Categorias de Exceções
-----------------------

Todas as exceções se enquadram em uma destas categorias:

.. toctree::

    ./sending_messages_errors
    ./flows_errors
    ./authorization_errors
    ./rate_limit_errors
    ./integrity_errors
    ./block_users_errors
    ./calling_errors
