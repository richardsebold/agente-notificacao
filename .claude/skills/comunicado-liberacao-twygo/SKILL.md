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
  só cole a mensagem de "segue algumas liberações para hoje" com links do Artia.
---

# Comunicado de liberação na Twygo

Você vai transformar uma lista de projetos + URLs do Artia em um **Comunicado de liberação** padronizado, descrevendo as atividades que foram para o ambiente de produção. O comunicado é lido por toda a equipe ("Twygeers"), então precisa ser claro, objetivo e seguir exatamente a estrutura abaixo.

O fluxo tem três partes: **(1)** entender a entrada, **(2)** ler as atividades no Artia pelo navegador, **(3)** montar o comunicado.

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
- **Categoria**: o usuário informa a categoria de cada projeto (ex.: Sustentação/N2, Garantia, Inovação). Se a categoria não estiver clara, **pergunte** antes de continuar — a categoria define como a seção é formatada (veja o Passo 3). Não adivinhe silenciosamente.

A URL aponta para uma atividade específica dentro de uma **pasta de projeto** (`/f/<id>/`). O que interessa para o comunicado **não é** a atividade "Gerar versão para deploy" em si, e sim as **outras atividades do mesmo projeto que estão concluídas e validadas** — essas são as que efetivamente foram para produção.

---

## Passo 2 — Ler as atividades no Artia (Claude in Chrome)

O Artia exige login, então use as ferramentas do **Claude in Chrome** (navegador com a sessão já logada do usuário). Se o Claude in Chrome não estiver disponível/conectado, avise o usuário e peça para ele colar o conteúdo das atividades manualmente, em vez de inventar.

Para cada projeto:

1. **Abra a URL** fornecida (a atividade "Gerar versão para deploy").
2. **Navegue até o quadro/pasta do projeto** (`/f/<id>/`) para ver a lista de atividades.
3. **Identifique as atividades concluídas e validadas.** No Artia, são as que estão na coluna/status de concluído + validado. São essas que entraram em produção e devem aparecer no comunicado. Ignore atividades em andamento, em teste ou não validadas.
4. **Para cada atividade concluída e validada, abra-a e leia:**
   - **Título** da atividade.
   - **Link** (URL da atividade — copie exatamente, preservando o domínio `app.artia.com` ou `app2.artia.com`).
   - **Descrição** da atividade → vira o campo *Descrição* (o problema / contexto).
   - **Comentários** da atividade → fonte do campo *Solução* (o que foi feito e a validação em stage). Leia o(s) comentário(s) mais relevante(s), normalmente o que descreve a correção/validação.

Capture esses dados de forma organizada por projeto antes de montar o texto. Se uma atividade não tiver descrição ou comentário suficientes, registre o que houver e siga em frente — não invente solução.

---

## Passo 3 — Montar o comunicado

Use **sempre** a saudação "Bom dia" e a **data de ontem** (data atual − 1 dia, formato `DD/MM/YYYY`) em todos os lugares de data do cabeçalho e dos títulos de seção, a menos que o usuário peça outra data explicitamente.

Estrutura geral (preserve a indentação dos títulos de seção, que usam recuo):

```
Comunicado de liberação na Twygo - <DATA>
Bom dia, pessoal, segue a lista das atividades liberadas para ambiente de produção do dia de ontem 

Twygeers - A toca da coruja!

Atividades referente a data do dia <DATA>.

        <Categoria> (Liberações <DATA>).

<subseções e atividades da categoria>
```

Onde `<DATA>` é a data de ontem. Agrupe as atividades por categoria e ordene as seções **sempre** nesta ordem fixa: **Sustentação/N2 → Garantia → Inovação**. Omita as categorias que não tiverem atividades. Cada categoria vira um bloco com seu título recuado.

### Como formatar cada categoria

Há dois formatos, e a categoria define qual usar — porque cada tipo de trabalho comunica coisas diferentes:

**A) Sustentação, Garantia, N2 e correções em geral** → formato **por atividade**, porque o time quer saber qual problema foi resolvido e que está validado. Para cada atividade:

```
Título: <título da atividade>
Link: <url da atividade>
Descrição: <o problema / contexto, de forma objetiva>
Solução: <o que foi feito e a validação em stage>
```

**Subdivisão por projeto:** quando uma categoria tiver atividades de **mais de um projeto**, separe os itens por projeto, com uma linha de subtítulo (o nome do projeto) antes de cada grupo. Use como subtítulo o **nome do projeto exatamente como veio na mensagem de liberações** (ex.: "Competências", "Kit de marca", "Migrar API V1 > V2"). O texto **entre colchetes** no título da atividade serve para **identificar a qual projeto** cada item pertence (ex.: `P1 [Kit de Marca] Alterar componente de filtro` → projeto "Kit de marca"), mas o subtítulo deve seguir o nome da mensagem, não o texto cru do colchete. Em N2/Sustentação isso normalmente aparece como o subtítulo "Sustentação / N2". Se houver só um projeto na categoria, não precisa do subtítulo.

**B) Inovação e projetos novos** → formato **por projeto** (visão de alto nível), porque são iniciativas novas, não correções pontuais — não há "problema/solução", e sim um objetivo:

```
<Nome do projeto>
<url do projeto>
Descrição: <objetivo do projeto, resumido>
```

Separe cada atividade/projeto e cada seção com uma linha em branco, como no exemplo de referência.

### Estilo da escrita

Siga o padrão do exemplo de referência fielmente — é o tom que a equipe espera:

- **Descrição**: explica o problema/contexto. Em correções (Sustentação/Garantia/N2), comece tipicamente com "Problema:" — ex.: `Descrição: Problema: Menus de links customizados perdiam o nome configurado e assumiam o texto genérico "Link customizado" durante a cópia para novas contas trial.`
- **Solução**: diz o que foi feito e que foi validado, normalmente em stage. Use frases no padrão do exemplo, como `Solução: Correção validada em stage, garantindo que o nome original do menu seja mantido durante a criação do trial.` ou `Solução: Correção foi realizada e testada em stage com sucesso.`
- Seja conciso e factual: resuma em 1–2 frases por campo, mantendo o sentido técnico. Não copie textão do Artia.
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

Título: Agente de atendimento - Vídeos grandes não indexava no Pinecone
Link: https://app2.artia.com/a/4874953/f/4883952/activities/32713273
Descrição: A indexação de vídeos grandes (≈1GB) no Pinecone travava em looping infinito e gerava erro no processamento.
Solução: Correção validada em stage, garantindo a indexação completa do vídeo e confirmada pelo retorno de respostas coerentes pela IA de atendimento.

Título: [BUG] Aumento no valor ao realizar parcelamento em cursos pagos
Link: https://app2.artia.com/a/4874953/f/4883952/activities/32840466
Descrição: Problema: Erro no checkout de cursos pagos onde o valor das parcelas estava incorreto e era aplicado em duplicidade o desconto.
Solução: Correção foi realizada e testada e stage com sucesso.

Título: [Cópia trial] Menu com "link customizado" está sendo copiado com nome incorreto
Link: https://app.artia.com/a/4874953/f/4883952/activities/32835983
Descrição: Problema: Menus de links customizados perdiam o nome configurado e assumiam o texto genérico "Link customizado" durante a cópia para novas contas trial.
Solução: Correção validada em stage, garantindo que o nome original do menu seja mantido durante a criação do trial.

        Inovação

Créditos de IA
https://app2.artia.com/a/4874953/f/6392535/activities
Descrição: O projeto tem como objetivo desenvolver um sistema centralizado de gestão e acompanhamento dos créditos de Inteligência Artificial contratados pelos clientes da plataforma Twygo, permitindo monitoramento em tempo real do consumo, configuração e políticas de uso.
```

---

## Ao final

Entregue o comunicado pronto em texto, dentro de um bloco para fácil cópia. Se algum projeto ficou sem atividades validadas ou sem dados suficientes, avise o usuário separadamente para ele decidir se inclui ou não.
