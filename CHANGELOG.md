# Changelog

## Unreleased

### Changed

- **Per-prompt model selection**
  ([`src/m-chatgpt-api.c`](src/m-chatgpt-api.c)): prompts may now specify an
  optional `model` field. Its value is sent in the Chat Completions request for
  that prompt; prompts without the field retain the `gpt-4o` default. This lets
  different prompts use different models without recompiling the plugin.

- **Model-selection debug output**
  ([`src/m-chatgpt-api.c`](src/m-chatgpt-api.c)): debug logging now reports
  both the model sent to the API and the prompt selected by the user, for
  example `Using model: gpt-5.6 (Proofread)`. The `ai-proofread-` action prefix
  is omitted so the displayed prompt name matches the toolbar and menu label.

### Fixed

- **Missing Evolution Mail include paths during compilation**
  ([`src/CMakeLists.txt`](src/CMakeLists.txt)): the plugin target linked against
  `evolution-mail-3.0` but did not add its include directories. On current
  Ubuntu/Evolution installations, this caused compilation to fail while
  including `libebook/libebook.h` from Evolution's composer headers.
  `${EVOLUTION_MAIL_INCLUDE_DIRS}` is now included in the target's private
  include paths.

- **Use-after-free in asynchronous proofreading**
  ([`src/m-msg-composer-extension.c`](src/m-msg-composer-extension.c), lines
  21-53, 135-228, 247-259, and 282-293): the `ProofreadContext` previously
  retained borrowed pointers to `EContentEditor`, `MMsgComposerExtension`, and
  the selected prompt ID while `e_content_editor_get_content()` completed
  asynchronously. Closing the composer or changing/destroying its UI before
  `msg_text_cb()` ran could leave the callback accessing freed objects or a
  freed prompt string. The context now owns GObject references to the editor
  and extension, and an allocated copy of the prompt ID. `proofread_context_free()`
  releases all three consistently after the callback has completed.

- **Leaked editor data on proofreading failures**
  ([`src/m-msg-composer-extension.c`](src/m-msg-composer-extension.c), lines
  175-213): API-error and empty-response paths returned without releasing the
  extracted message content and `EContentEditorContentHash`. Both resources are
  now released before returning.

- **Out-of-bounds read while reporting HTTP errors**
  ([`src/m-chatgpt-api.c`](src/m-chatgpt-api.c), lines 127-140):
  `g_bytes_get_data()` returns a byte buffer with an explicit length, not a
