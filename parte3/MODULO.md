# Módulo `flaskbb/forum/`

## Propósito

O módulo `forum` é o núcleo do domínio do FlaskBB. Ele organiza o conteúdo na hierarquia categoria → fórum → tópico → post, controla quem pode ler, criar, editar, ocultar, mover, travar e excluir esse conteúdo e mantém os dados de leitura de cada usuário (o que já foi lido e quais tópicos ele acompanha). Além de persistir o conteúdo, o módulo mantém atualizados vários dados desnormalizados (último post, contagens de posts e tópicos) que as páginas usam para evitar consultas pesadas.

## Mapa dos arquivos

| Arquivo | Responsabilidade |
|---|---|
| `__init__.py` | Identifica o pacote; não contém lógica. |
| `models.py` | Define `Category`, `Forum`, `Topic`, `Post`, `Report`, `TopicsRead` e `ForumsRead`, com persistência e atualização de contadores e último post. |
| `views.py` | Implementa as rotas do fórum como `MethodView` e registra o blueprint `forum`. |
| `forms.py` | Define os formulários de post, tópico, denúncia e busca, incluindo a gravação a partir dos dados do formulário. |
| `locals.py` | Expõe `current_category`, `current_forum` e `current_topic` como proxies do contexto da requisição. |
| `utils.py` | Decide se o usuário deve ser forçado a fazer login antes de ver um fórum. |

## Entradas e saídas

**Entradas externas.** As rotas HTTP do blueprint `forum`, registradas em `flaskbb_load_blueprints` (`views.py`) com prefixo `FORUM_URL_PREFIX` (vazio por padrão). As principais são `/`, `/category/<id>`, `/forum/<id>`, `/topic/<id>`, `/<forum_id>/topic/new`, `/topic/<id>/post/new`, `/post/<id>/edit`, `/post/<id>/report`, `/forum/<id>/edit` (moderação), `/search` e `/memberlist`. O módulo também é usado diretamente pelo painel `management`, por `user/models.py` e por `utils/populate.py`, que chamam os métodos dos modelos. Não há comandos CLI próprios do módulo.

**Saídas.** Gravações nas tabelas `categories`, `forums`, `topics`, `posts`, `reports`, `topicsread`, `forumsread` e `topictracker`; atualização do índice de busca (Whooshee); páginas HTML renderizadas e mensagens `flash`; e eventos de plugin emitidos via `pluggy` (`flaskbb_event_post_save_before/after`, `flaskbb_event_topic_save_before/after`, `flaskbb_form_post_save`, `flaskbb_form_topic_save`). Falhas de acesso são comunicadas com `abort(404)` e redirecionamentos com mensagem de erro.

## Contratos documentados (docstrings)

- `flaskbb/forum/models.py:359` — `Post.hide`: explica que ocultar o primeiro post oculta o tópico inteiro, que o método retorna `None` se o post já estava oculto e quais dados desnormalizados são recalculados.
- `flaskbb/forum/models.py:400` — `Post._deal_with_last_post`: informa quando o método age, quais as cinco colunas de "último post" do fórum ele reescreve e que não faz `commit`.
- `flaskbb/forum/models.py:456` — `Post._update_counts`: explica que as contagens são recalculadas no banco (e não incrementadas) e como o estado de `hidden` muda o filtro.
- `flaskbb/forum/models.py:607` — `Topic.second_last_post` (reescrita): deixa explícito que a propriedade retorna um **id**, e não um objeto `Post`, apesar do nome.
