---
name: comunicado-liberacao-twygo
description: >
  Gera o "Comunicado de liberação na Twygo": a partir de uma ou mais URLs de atividades
  do Artia (app.artia.com / app2.artia.com), navega no Artia pelo Claude in Chrome, lê as
  atividades concluídas e validadas de cada projeto (descrição + comentários) e monta o
  comunicado padronizado das liberações que foram para produção. Use esta skill SEMPRE que
  o usuário pedir: "comunicado de liberação", "notificação de liberação", "monta o comunicado
  das liberações", "gerar comunicado da Twygo", "liberações de ontem", "comunicado de produção",
  ou colar uma lista de projetos com URLs do Artia pedindo o comunicado. Use mesmo que o usuário
  só cole a mensagem de "segue algumas liberações para hoje" com links do Artia. Use TAMBÉM
  quando o usuário pedir apenas "faça a notificação de liberação" / "comunicado de ontem"
  SEM colar nenhum link: nesse caso os links são buscados automaticamente no grupo "Deploy"
  do Microsoft Teams, pelo MCP do Microsoft 365 (Passo 0).
---

# Comunicado de liberação na Twygo

Você vai transformar uma lista de projetos + URLs do Artia em um **Comunicado de liberação** padronizado, descrevendo as atividades que foram para o ambiente de produção. O comunicado é lido por toda a equipe ("Twygeers"), então precisa ser claro, objetivo e seguir exatamente a estrutura abaixo.

O fluxo tem quatro partes: **(0)** obter os links das liberações no grupo "Deploy" do Teams, quando o usuário não os colar, **(1)** entender a entrada, **(2)** ler as atividades no Artia pelo navegador, **(3)** montar o comunicado.

---

## Passo 0 — Buscar as liberações no grupo "Deploy" do Teams (quando o usuário não colar os links)

Se o pedido vier **sem links** — "faça a notificação de liberação", "monta o comunicado de ontem", "comunicado de liberação" e nada mais — **não pergunte os links ao usuário**. Busque você mesmo, no grupo **"Deploy"** do Microsoft Teams, usando o **MCP do Microsoft 365**:

1. **Ache o chat.** `teams_list_chats` e procure o item com `chatType: "group"` e `topic: "Deploy"`. Em 20/08/2026 o id era `19:d34e93a31bd64e1ba0beafc8c5bf698b@thread.v2` — pode tentar direto por ele, mas **confirme pelo topic**, porque o id muda se o grupo for recriado.
2. **Liste as mensagens.** `read_resource` com a URI `teams:///chats/<chatId>/messages` (a URI **precisa** terminar em `/messages`; sem isso o MCP rejeita com `VALIDATION_ERROR`). A listagem traz as ~20 mensagens mais recentes, com `from`, `createdDateTime` e um `bodyPreview` **truncado**.
3. **Abra cada mensagem candidata.** O `bodyPreview` corta a lista de links, então leia o corpo completo de cada mensagem de deploy com `teams:///chats/<chatId>/messages/<messageId>` — é de lá que saem todos os links, a OBS e a assinatura.
4. **Filtre pelo dia anterior, no horário de Brasília.** `createdDateTime` vem em **UTC**: converta para UTC−3 antes de decidir o dia (uma mensagem de `2026-08-18T00:42Z` é de **17/08** à noite no horário local). Interessam as mensagens postadas **ontem** (data atual − 1 dia) que contenham links de atividade do Artia (`app.artia.com` / `app2.artia.com` com `/activities/<id>`) — em geral as de "Segue atividade para deploy" / "Segue atividade para liberação". Junte **todas** as mensagens de ontem: é comum o pedido vir em partes, com projetos acrescentados ao longo do dia. Ignore o bate-papo do grupo ("deploy concluído com sucesso", combinações de horário).
5. **Extraia de cada bloco** o que o Passo 1 espera: nome do projeto (o texto entre colchetes ou o título antes de "Gerar versão para deploy"), a URL da atividade, o **solicitante** (quem assinou/postou a mensagem, quando não houver o campo na atividade) e eventuais OBS (ex.: "realizar deploy assim que possível").
6. **Deduplique.** O mesmo link costuma aparecer em mais de uma mensagem/deploy do dia — cada atividade entra **uma única vez** no comunicado.
7. **Se der 429 (rate limit)**, o erro traz `graphRetryAfterSeconds`: aguarde esse tempo e repita. Como alternativa existe `chat_message_search` (`query: "artia.com"` + `afterDateTime`/`beforeDateTime`), mas ele varre chat por chat e pode não alcançar o grupo Deploy — o `read_resource` direto é o caminho confiável.
8. **Se não houver mensagem de ontem** com links do Artia, mostre ao usuário o que encontrou (as mensagens mais recentes e suas datas) e pergunte como seguir. **Nunca invente links ou atividades.**

Quando o usuário **colar** os links na mensagem, pule este passo e use o que ele mandou.

---

## Passo 1 — Entender a entrada

O usuário fornece uma mensagem com vários blocos, cada um sendo um projeto que será liberado. Normalmente vem assim:

```
Boa tarde pessoal, segue algumas liberações para hoje:

Competências - Gerar versão para deploy da Twygo - 13/05/2026
https://app.artia.com/a/4874953/f/6257028/activities/32856832

Kit de marca - Gerar versão para deploy da Twygo - 13/05/2026
https://app2.artia.com/a/4874953/f/6414891/activities/32866954
```

Para cada bloco, extraia:
- **Nome do projeto** (ex.: "Competências", "Kit de marca", "Migrar API V1 > V2").
- **URL** da atividade "Gerar versão para deploy" (é o link de entrada para chegar no projeto).
- **Categoria**: quando o usuário informar a categoria de cada projeto (ex.: Sustentação/N2, Garantia, Inovação), use a dele. Quando não informar — o caso normal do Passo 0 —, **deduza pelo campo "Projeto / Pasta" da atividade no Artia**, sem parar para perguntar:
  - pasta `99 - Sustentação > ... > N2` → **Sustentação / N2**;
  - pasta `01 - Desenvolvimento > 05 - Em Beta > <projeto>` → **Garantia**, subdividida pelo nome do projeto;
  - pasta `01 - Desenvolvimento > 04 - Em Desenvolvimento > <projeto>` ou projeto novo → **Inovação**, no formato de alto nível.
  Só pergunte se a pasta não permitir decidir. A categoria define como a seção é formatada (veja o Passo 3) — não adivinhe silenciosamente fora dessas regras.

A URL aponta para uma atividade específica dentro de uma **pasta de projeto** (`/f/<id>/`). O que interessa para o comunicado **não é** a atividade "Gerar versão para deploy" em si, e sim as **outras atividades do mesmo projeto que estão concluídas e validadas** — essas são as que efetivamente foram para produção.

---

## Passo 2 — Ler as atividades no Artia (Claude in Chrome)

O Artia exige login, então use as ferramentas do **Claude in Chrome** (navegador com a sessão já logada do usuário). Se o Claude in Chrome não estiver disponível/conectado, avise o usuário e peça para ele colar o conteúdo das atividades manualmente, em vez de inventar.

Para cada projeto:

1. **Abra a URL** fornecida (a atividade "Gerar versão para deploy").
2. **Navegue até o quadro/pasta do projeto** (`/f/<id>/`) para ver a lista de atividades.
3. **Identifique as atividades concluídas e validadas.** No Artia, são as que estão na coluna/status de concluído + validado. São essas que entraram em produção e devem aparecer no comunicado. Ignore atividades em andamento, em teste ou não validadas. A existência (ou não) de um Pull Request **não** é o critério: uma atividade concluída e validada entra no comunicado mesmo sem PR de código próprio (ex.: ajuste de configuração ou correção embarcada em outro PR) — nesse caso a *Solução* segue normalmente como "Correção validada em stage".
4. **Para cada atividade concluída e validada, abra-a e leia:**
   - **Título** da atividade.
   - **Link** (URL da atividade — copie exatamente, preservando o domínio `app.artia.com` ou `app2.artia.com`).
   - **Descrição** da atividade → vira o campo *Descrição* (o problema / contexto).
   - **Pull Request** (quando a atividade trouxer link de PR) → **leia o PR** (descrição + diff) para entender o que de fato mudou; Descrição/Solução derivadas só do título ficam rasas. Traduza o achado para o efeito prático ao usuário, nunca para o detalhe de implementação. **Realidade atual:** `github.com/Twygo/twyg-app` é repositório privado, o `gh` não está instalado nesta máquina e o perfil do navegador não tem sessão do GitHub (retorna 404) — então, na prática, o PR quase sempre fica ilegível. Nesse caso **não force**: use a descrição e os comentários do Artia, que normalmente já traçam causa, correção e validação. Se o `gh` estiver disponível e autenticado, `gh pr view <url>` é o caminho mais rápido.
   - **Comentários** da atividade → fonte do campo *Solução* (o que foi feito e a validação em stage). Leia o(s) comentário(s) mais relevante(s), normalmente o que descreve a correção/validação.
   - **Solicitante** da atividade → vira o campo *Solicitante* (quem pediu a atividade). **Onde procurar depende da categoria do projeto:**
     - **Sustentação/N2 e Garantia** → use o campo **"Solicitante"** da própria atividade no Artia (o campo/rótulo que marca quem é o solicitante). É esse nome que vai para o comunicado.
     - **DHO** → o solicitante **não** é quem criou nem quem está marcado como solicitante da atividade. Ele está escrito **na descrição da atividade** — leia a descrição e extraia o nome da pessoa que de fato solicitou. Use esse nome, ignorando o campo "Solicitante" do Artia. Se a descrição não deixar claro quem solicitou, registre o que houver e sinalize ao usuário em vez de assumir o criador da atividade.

Capture esses dados de forma organizada por projeto antes de montar o texto. Se uma atividade não tiver descrição ou comentário suficientes, registre o que houver e siga em frente — não invente solução.

---

## Passo 3 — Montar o comunicado

Use **sempre** a saudação "Bom dia" e a **data de ontem** (data atual − 1 dia, formato `DD/MM/YYYY`) em todos os lugares de data do cabeçalho e dos títulos de seção, a menos que o usuário peça outra data explicitamente.

Estrutura geral:

```
Comunicado de liberação na Twygo - <DATA>
Bom dia, pessoal, segue a lista das atividades liberadas para ambiente de produção do dia de ontem 

Twygeers - A toca da coruja!

Atividades referente a data do dia <DATA>.

<Categoria> (Liberações <DATA>).

<subseções e atividades da categoria>
```

Onde `<DATA>` é a data de ontem. Agrupe as atividades por categoria e ordene as seções **sempre** nesta ordem fixa: **Sustentação/N2 → Garantia → Inovação**. Omita as categorias que não tiverem atividades.

### Espaçamento e indentação (siga exatamente)

- **Nenhuma linha leva recuo.** Tudo alinhado à esquerda, na coluna 1: a saudação, "Twygeers - A toca da coruja!", "Atividades referente...", os **títulos de categoria** (ex.: `Sustentação / N2 (Liberações <DATA>).` e `Inovação`), os subtítulos de projeto e todos os campos das atividades (Título/Link/Descrição/Solução/Solicitante) e blocos de Inovação.
- Não use espaços, tabulação ou `&nbsp;` no início de nenhuma linha — nem nos títulos de categoria.
- A separação entre blocos é feita **só** com linha em branco: uma linha em branco entre cada atividade/projeto e entre cada seção.

### Como formatar cada categoria

Há dois formatos, e a categoria define qual usar — porque cada tipo de trabalho comunica coisas diferentes:

**A) Sustentação, Garantia, N2 e correções em geral** → formato **por atividade**, porque o time quer saber qual problema foi resolvido e que está validado. Para cada atividade:

```
**Título: <título da atividade>**
**Link:** <url da atividade>
**Descrição:** <o problema / contexto, em linguagem simples>
**Solução:** <o que foi feito e a validação em stage, em linguagem simples>
**Solicitante:** <nome da pessoa solicitante da atividade>
```

**Negrito:** apenas a **linha do Título** fica inteira em negrito (rótulo + texto no mesmo `**...**`). Em **Link, Descrição, Solução e Solicitante**, só o **rótulo** (a palavra antes dos dois pontos) fica em negrito — ex.: `**Link:** https://...`; o texto que vem depois fica normal. O subtítulo de projeto (ex.: "Sustentação / N2") e o título de categoria seguem como estão, sem negrito extra.

O campo **Solicitante** aparece **abaixo da Solução**. A origem do nome depende da categoria (veja o Passo 2): em **Sustentação/N2** e **Garantia** vem do campo "Solicitante" da própria atividade; em **DHO** vem da **descrição** da atividade (não do criador nem do campo "Solicitante"). Se não for possível determinar o solicitante, deixe o campo sinalizado (ex.: `Solicitante: (não identificado)`) e avise o usuário.

**Subdivisão por projeto:** quando uma categoria tiver atividades de **mais de um projeto**, separe os itens por projeto, com uma linha de subtítulo (o nome do projeto) antes de cada grupo. Use como subtítulo o **nome do projeto exatamente como veio na mensagem de liberações** (ex.: "Competências", "Kit de marca", "Migrar API V1 > V2"). O texto **entre colchetes** no título da atividade serve para **identificar a qual projeto** cada item pertence (ex.: `P1 [Kit de Marca] Alterar componente de filtro` → projeto "Kit de marca"), mas o subtítulo deve seguir o nome da mensagem, não o texto cru do colchete. Em N2/Sustentação isso normalmente aparece como o subtítulo "Sustentação / N2". Se houver só um projeto na categoria, não precisa do subtítulo.

**B) Inovação e projetos novos** → formato **por projeto** (visão de alto nível), porque são iniciativas novas, não correções pontuais — não há "problema/solução", e sim um objetivo:

```
**<Nome do projeto>**
<url do projeto>
**Descrição:** <objetivo do projeto, resumido, em linguagem simples>
```

Aqui o **Nome do projeto** faz o papel do Título (linha inteira em negrito), a URL fica normal e em **Descrição** só o rótulo é negrito.

Separe cada atividade/projeto e cada seção com uma linha em branco, como no exemplo de referência.

### Estilo da escrita

Siga o padrão do exemplo de referência fielmente — é o tom que a equipe espera:

- **Sem termos técnicos (regra principal):** o comunicado é lido por toda a equipe, não só por devs. **Não use jargão técnico** — nomes de código, classes, métodos, tabelas, campos internos, bibliotecas, siglas de implementação, mensagens/stack de erro cruas etc. Prefira sempre a palavra do dia a dia e descreva o efeito para o usuário (o que ele passou a ver funcionando), não o "como" da implementação.
- **Quando um termo técnico for realmente inevitável**, escreva-o de forma simples e coloque ao lado uma **explicação muito breve entre parênteses**, ex.: `a indexação (etapa em que o conteúdo do vídeo é preparado para a IA conseguir consultá-lo)` ou `a conta de teste (trial)`. A explicação é curta — uma frase, no máximo.
- **Título:** copie-o exatamente como está no Artia (não invente/traduza o título). Se o próprio título tiver um termo técnico, mantenha-o assim mesmo, mas escreva a Descrição e a Solução em linguagem simples.
- **Descrição**: explica o problema/contexto em linguagem simples. Em correções (Sustentação/Garantia/N2), comece tipicamente com "Problema:" — ex.: `Descrição: Problema: Menus de links personalizados perdiam o nome configurado e ficavam com o texto genérico "Link customizado" ao criar novas contas de teste (trial).`
- **Solução**: diz o que foi feito e que foi validado, normalmente em stage, também em linguagem simples. Use frases no padrão do exemplo, como `Solução: Correção validada em stage, garantindo que o nome original do menu seja mantido na criação da conta de teste.` ou `Solução: Correção foi realizada e testada em stage com sucesso.`
- **Detalhe objetivo na Descrição, sem jargão**: quando a atividade trouxer dados que deixam o problema inequívoco, inclua-os — mas descreva o **sintoma que o usuário via** (ex.: "a tela travava e não carregava a lista"), o valor configurado vs. o observado (ex.: "mínimo de pares era 2, mas aprovou com 1") e a tela/ação exatas. Não cole mensagem/stack de erro crua; se o erro for essencial, resuma o que ele significava em palavras simples. Detalhe esclarece; não vira textão.
- Seja conciso e factual: resuma em 1–2 frases por campo, mantendo o sentido, em linguagem acessível. Não copie textão do Artia.
- **Baseie-se no Pull Request para entender o que de fato mudou** — mas traduza isso para o efeito prático ao usuário, não para o detalhe de implementação. Não parafraseie só o título.
- Mantenha os links exatamente como estão no Artia.
- Não invente atividades, links ou soluções. Se faltar informação, sinalize ao usuário.

---

## Exemplo de referência (saída completa)

Use como modelo de tom, estrutura e nível de detalhe:

```
Comunicado de liberação na Twygo - 11/05/2026
Bom dia, pessoal, segue a lista das atividades liberadas para ambiente de produção do dia de ontem 

Twygeers - A toca da coruja!

Atividades referente a data do dia 11/05/2026.

Sustentação (Liberações 11/05/2026).

Sustentação / N2

**Título: Agente de atendimento - Vídeos grandes não indexava no Pinecone**
**Link:** https://app2.artia.com/a/4874953/f/4883952/activities/32713273
**Descrição:** Vídeos muito grandes (cerca de 1 GB) não conseguiam ser preparados para consulta (indexação — etapa em que o conteúdo do vídeo é organizado para a IA conseguir buscá-lo), travando o processo e gerando erro.
**Solução:** Correção validada em stage: os vídeos grandes passaram a ser preparados por completo e a IA de atendimento voltou a responder corretamente sobre eles.
**Solicitante:** Christofer Bastos

**Título: [BUG] Aumento no valor ao realizar parcelamento em cursos pagos**
**Link:** https://app2.artia.com/a/4874953/f/4883952/activities/32840466
**Descrição:** Problema: No pagamento de cursos pagos, o valor das parcelas saía errado e o desconto era aplicado duas vezes.
**Solução:** Correção realizada e testada em stage com sucesso.
**Solicitante:** Vinicius Lisboa

**Título: [Cópia trial] Menu com "link customizado" está sendo copiado com nome incorreto**
**Link:** https://app.artia.com/a/4874953/f/4883952/activities/32835983
**Descrição:** Problema: Ao criar novas contas de teste (trial), os menus de link personalizado perdiam o nome configurado e ficavam com o texto genérico "Link customizado".
**Solução:** Correção validada em stage, garantindo que o nome original do menu seja mantido na criação da conta de teste.
**Solicitante:** Christofer Bastos

Inovação

**Créditos de IA**
https://app2.artia.com/a/4874953/f/6392535/activities
**Descrição:** O projeto cria um painel central para acompanhar os créditos de Inteligência Artificial contratados pelos clientes da Twygo, permitindo ver o consumo em tempo real e configurar as regras de uso.
```

---

## Ao final

**Sempre** salve o comunicado em um arquivo e retorne o caminho dele ao usuário no final de cada execução:

1. Escreva o comunicado completo em `outputs/comunicado-liberacao-<AAAA-MM-DD>.md`, na raiz do projeto, onde `<AAAA-MM-DD>` é a data de ontem (a mesma `<DATA>` do comunicado, no formato ISO). Crie a pasta `outputs/` se ela não existir.
2. O conteúdo do arquivo deve ser **exatamente** o comunicado, já seguindo o padrão de espaçamento definido acima (todas as linhas alinhadas à esquerda, sem recuo). Não adicione cercas de código (```) nem texto extra dentro do arquivo.
3. Na sua resposta, informe o caminho do arquivo gerado e cole o conteúdo do comunicado em um bloco para fácil cópia.

### Versão para colar no Teams (negrito de verdade, sem asteriscos)

O comunicado é colado no **Microsoft Teams**, que **não interpreta os `**` do Markdown** ao colar texto puro — os asteriscos apareceriam literalmente. Por isso, além do `.md`, **sempre** gere uma versão em **texto rico (HTML)** e coloque-a na **área de transferência** no formato CF_HTML, para o usuário colar direto no Teams com o negrito preservado e **sem nenhum asterisco**.

1. Gere `outputs/comunicado-liberacao-<AAAA-MM-DD>.html` com o mesmo conteúdo, convertendo a formatação para HTML:
   - Cada linha vira um `<div>...</div>`. As **linhas em branco** viram `<div>&#10240;</div>` — o caractere *braille blank* (U+2800). É o único separador que o Teams **não colapsa**: `&nbsp;`, `<br>`, `<div></div>` e margens de `<p>` são todos removidos pelo Teams ao colar e o espaçamento some.
   - **Negrito** (mesma regra do comunicado): a **linha do Título** inteira dentro de `<strong>...</strong>`; em **Link, Descrição, Solução e Solicitante** apenas o rótulo em `<strong>` (ex.: `<strong>Link:</strong> https://...`). O **Nome do projeto** (Inovação) inteiro em `<strong>`.
   - Nenhum recuo: não gere `&nbsp;` no início das linhas, nem nos títulos de categoria. Não use `**` no HTML.
2. Copie esse HTML para a área de transferência no formato CF_HTML com PowerShell (é o que faz o Teams manter o negrito e descartar os asteriscos):

   ```powershell
   Add-Type -AssemblyName System.Windows.Forms
   $path = 'outputs/comunicado-liberacao-<AAAA-MM-DD>.html'  # caminho absoluto
   $raw = Get-Content -Raw -Encoding UTF8 $path
   # IMPORTANTE: converte todo caractere acentuado/não-ASCII em entidade HTML numérica.
   # Sem isso, o clipboard HTML do .NET corrompe acentos (á, ç, ã viram "diamante").
   $sb = New-Object System.Text.StringBuilder
   foreach ($ch in $raw.ToCharArray()) {
     $code = [int][char]$ch
     if ($code -gt 127) { [void]$sb.Append("&#$code;") } else { [void]$sb.Append($ch) }
   }
   $html = $sb.ToString()
   $pre  = "<html><body><!--StartFragment-->"
   $post = "<!--EndFragment--></body></html>"
   $body = $pre + $html + $post
   $enc = [System.Text.Encoding]::UTF8
   $tmpl = "Version:0.9`r`nStartHTML:{0:0000000000}`r`nEndHTML:{1:0000000000}`r`nStartFragment:{2:0000000000}`r`nEndFragment:{3:0000000000}`r`n"
   $hlen = $enc.GetByteCount(($tmpl -f 0,0,0,0))
   $cf = ($tmpl -f $hlen, ($hlen + $enc.GetByteCount($body)), ($hlen + $enc.GetByteCount($pre)), ($hlen + $enc.GetByteCount($pre) + $enc.GetByteCount($html))) + $body
   [System.Windows.Forms.Clipboard]::SetText($cf, [System.Windows.Forms.TextDataFormat]::Html)
   ```
3. Avise o usuário que o comunicado já está na área de transferência: basta colar (Ctrl+V) no Teams, sem copiar outra coisa antes.

Se algum projeto ficou sem atividades validadas ou sem dados suficientes, avise o usuário separadamente para ele decidir se inclui ou não.
