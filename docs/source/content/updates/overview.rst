💬 Updates
==========

.. currentmodule:: pywa.types

Atualizações são os **eventos recebidos** da API do WhatsApp Cloud.
Elas são enviadas para a URL do seu webhook e convertidas pelo PyWa em objetos com tipagem segura, fáceis de lidar.

Na API do WhatsApp Cloud, as atualizações são chamadas de **fields** (campos), e você precisa se inscrever nelas para recebê-las na URL do seu webhook.

-----------------
Campos Suportados
-----------------

Os campos atualmente suportados pelo PyWa são:

- ``messages`` → todas as atualizações relacionadas ao usuário (mensagens, callbacks, atualizações de status de mensagem)
- ``calls`` → conectar chamada, encerrar e atualizações de status
- ``message_template_status_update`` → template aprovado, rejeitado, etc.
- ``message_template_quality_update`` → pontuação de qualidade do template alterada
- ``message_template_components_update`` → componentes do template alterados (cabeçalho, corpo, rodapé, botões)
- ``template_category_update`` → categoria do template alterada
- ``user_preferences`` → preferências de marketing do usuário

.. tip::

   Se você quiser lidar com outros tipos de atualizações, use o decorador :func:`~pywa.client.WhatsApp.on_raw_update` ou a classe :py:class:`~pywa.handlers.RawUpdateHandler`.

   .. code-block:: python

      from pywa import WhatsApp, types

      wa = WhatsApp(...)

      @wa.on_raw_update
      def handle_raw_update(wa: WhatsApp, raw: types.RawUpdate):
          print("Received raw update:", raw)

-----------------
Tipos de Atualização
-----------------

Os campos suportados são processados automaticamente pelo PyWa e convertidos em classes Python.

👉 Para aprender a lidar com eles, veja: `Handlers <../handlers/overview.html>`_

**Atualizações relacionadas ao usuário:**

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - Tipo
     - Descrição
   * - :py:class:`~pywa.types.message.Message`
     - Uma mensagem enviada por um usuário (texto, mídia, pedido, localização, etc.)
   * - :py:class:`~pywa.types.callback.CallbackButton`
     - Um :py:class:`~pywa.types.callback.Button` | :py:class:`~pywa.types.templates.QuickReplyButton` pressionado por um usuário
   * - :py:class:`~pywa.types.callback.CallbackSelection`
     - Um :py:class:`~pywa.types.callback.SectionRow` escolhido por um usuário
   * - :py:class:`~pywa.types.flows.FlowCompletion`
     - Um flow concluído por um usuário
   * - :py:class:`~pywa.types.message_status.MessageStatus`
     - Uma atualização de status de mensagem (entregue, vista, etc.)
   * - :py:class:`~pywa.types.chat_opened.ChatOpened`
     - Um chat aberto por um usuário
   * - :py:class:`~pywa.types.system.PhoneNumberChange`
     - O número de telefone de um usuário foi alterado
   * - :py:class:`~pywa.types.system.IdentityChange`
     - A identidade de um usuário foi alterada
   * - :py:class:`~pywa.types.calls.CallConnect`
     - Uma chamada conectada por um usuário
   * - :py:class:`~pywa.types.calls.CallTerminate`
     - Uma chamada encerrada por um usuário
   * - :py:class:`~pywa.types.calls.CallStatus`
     - Uma atualização de status de chamada (tocando, ocupado, etc.)
   * - :py:class:`~pywa.types.calls.CallPermissionUpdate`
     - Uma atualização de permissão de chamada (permissão concedida ou negada)
   * - :py:class:`~pywa.types.user_preferences.UserMarketingPreferences`
     - Uma atualização de preferências de marketing do usuário (por ex. optou por receber, optou por não receber)

**Atualizações relacionadas à conta:**

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - Tipo
     - Descrição
   * - :py:class:`~pywa.types.templates.TemplateStatusUpdate`
     - Uma atualização de status de template (aprovado, rejeitado, etc.)
   * - :py:class:`~pywa.types.templates.TemplateCategoryUpdate`
     - Uma atualização de categoria de template (categoria alterada)
   * - :py:class:`~pywa.types.templates.TemplateQualityUpdate`
     - Uma atualização de qualidade de template (pontuação de qualidade alterada)
   * - :py:class:`~pywa.types.templates.TemplateComponentsUpdate`
     - Uma atualização de componentes de template (cabeçalho, corpo, rodapé, botões alterados)

-----------------
Propriedades Comuns
-----------------

.. currentmodule:: pywa.types.base_update

Todas as atualizações compartilham métodos e propriedades comuns:

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - Propriedade
     - Descrição
   * - :attr:`~BaseUpdate.id`
     - O ID da atualização
   * - :attr:`~BaseUpdate.raw`
     - Os dados brutos da atualização
   * - :attr:`~BaseUpdate.timestamp`
     - O timestamp da atualização (UTC)
   * - :attr:`~BaseUpdate.shared_data`
     - Um dicionário para compartilhar dados entre handlers
   * - :meth:`~BaseUpdate.stop_handling`
     - Impede que outros handlers processem a atualização
   * - :meth:`~BaseUpdate.continue_handling`
     - Força a atualização a continuar para o próximo handler
   * - :meth:`~BaseUpdate.handle_again`
     - Re-processa a atualização a partir do primeiro handler

**Atualizações relacionadas ao usuário** compartilham propriedades adicionais:

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - Método / Propriedade
     - Descrição
   * - :attr:`~BaseUserUpdate.sender`
     - O ID de telefone do remetente
   * - :attr:`~BaseUserUpdate.recipient`
     - O ID de telefone do destinatário
   * - :attr:`~BaseUserUpdate.message_id_to_reply`
     - O ID da mensagem para responder
   * - :meth:`~BaseUserUpdate.reply_text`
     - Responder com uma mensagem de texto
   * - :meth:`~BaseUserUpdate.reply_image`
     - Responder com uma mensagem de imagem
   * - :meth:`~BaseUserUpdate.reply_video`
     - Responder com uma mensagem de vídeo
   * - :meth:`~BaseUserUpdate.reply_audio`
     - Responder com uma mensagem de áudio
   * - :meth:`~BaseUserUpdate.reply_voice`
     - Responder com uma mensagem de voz
   * - :meth:`~BaseUserUpdate.reply_document`
     - Responder com uma mensagem de documento
   * - :meth:`~BaseUserUpdate.reply_location`
     - Responder com uma mensagem de localização
   * - :meth:`~BaseUserUpdate.reply_location_request`
     - Request the user’s location
   * - :meth:`~BaseUserUpdate.reply_contact`
     - Responder com uma mensagem de contato
   * - :meth:`~BaseUserUpdate.reply_sticker`
     - Responder com uma mensagem de sticker
   * - :meth:`~BaseUserUpdate.reply_template`
     - Responder com uma mensagem de template
   * - :meth:`~BaseUserUpdate.reply_catalog`
     - Responder com uma mensagem de catálogo
   * - :meth:`~BaseUserUpdate.reply_product`
     - Responder com uma mensagem de produto
   * - :meth:`~BaseUserUpdate.reply_products`
     - Responder com uma lista de mensagens de produtos
   * - :meth:`~BaseUserUpdate.react`
     - Reagir à atualização com um emoji
   * - :meth:`~BaseUserUpdate.unreact`
     - Remover uma reação
   * - :meth:`~BaseUserUpdate.mark_as_read`
     - Marcar a atualização como lida
   * - :meth:`~BaseUserUpdate.indicate_typing`
     - Indicar digitação ao usuário
   * - :meth:`~BaseUserUpdate.block_sender`
     - Bloquear o remetente
   * - :meth:`~BaseUserUpdate.unblock_sender`
     - Desbloquear o remetente
   * - :meth:`~BaseUserUpdate.call`
     - Iniciar uma chamada com o remetente

.. toctree::
    message
    callback_button
    callback_selection
    flow_completion
    message_status
    chat_opened
    phone_number_change
    identity_change
    call_connect
    call_terminate
    call_status
    call_permission_update
    user_marketing_preferences
    template_status_update
    template_category_update
    template_quality_update
    template_components_update
    raw_update
    common_methods
