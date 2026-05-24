Comunicado de liberação na Twygo - 23/05/2026
Bom dia, pessoal, segue a lista das atividades liberadas para ambiente de produção do dia de ontem 

Twygeers - A toca da coruja!

Atividades referente a data do dia 23/05/2026.

        Sustentação (Liberações 23/05/2026).

Sustentação / N2

Título: [BUG] Inscrição de usuários fora do ambiente com SSO ativo Keyence
Link: https://app2.artia.com/a/4874953/f/4883952/activities/32582877
Descrição: Problema: Em clientes com SSO ativo, era permitida a tentativa de inscrição de usuários que não pertenciam ao ambiente.
Solução: Correção validada em stage, bloqueando a inscrição de usuários externos quando o SSO está ativo (validadas as requisições de inscrição em massa e em conteúdos).

Título: [BUG] Delay no carregamento do banner exibe imagem padrão da plataforma temporariamente
Link: https://app2.artia.com/a/4874953/f/4883952/activities/32852578
Descrição: Problema: O banner personalizado da organização demorava a carregar na visão do aluno, exibindo temporariamente a imagem padrão da plataforma (reportado pelo cliente Blockbit).
Solução: Correção validada em stage, com o banner personalizado carregando diretamente, sem exibição temporária da imagem padrão.

        Garantia (Liberações 23/05/2026).

Competências

Título: P1 - [Competências] Erro ao realizar upload de arquivo grande
Link: https://app.artia.com/a/4874953/f/6257028/activities/32850745
Descrição: Problema: Erro ao realizar upload de arquivos grandes em qualquer ponto da funcionalidade de Competências (Skills) que permite upload.
Solução: Correção validada em stage em todos os pontos de upload de arquivo nas Skills.

Kit de marca

Título: P1 [Kit de Marca] Remover coluna "Tipo de Experiência" da tabela de compartilhar
Link: https://app2.artia.com/a/4874953/f/6414891/activities/32792239
Descrição: Problema: A tela de compartilhar exibia a coluna "Tipo de experiência", que não deveria aparecer nas regras do Kit de Marca.
Solução: Correção validada em stage, com a coluna "Tipo de experiência" removida da tela de compartilhar.

Título: P1 [Kit de Marca] Alterar componente de filtro
Link: https://app2.artia.com/a/4874953/f/6414891/activities/32792848
Descrição: Problema: O filtro do Kit de Marca não utilizava o componente de filtro atual, sem o botão para excluir filtro padrão.
Solução: Correção validada em stage, com o componente de filtro atual aplicado ao Kit de Marca, incluindo o botão de excluir filtro padrão.

Migrar API V1 > V2

Título: P1 [Migrar API V01>V02] Retornar informações de usuário inativado na consulta por usuário
Link: https://app2.artia.com/a/4874953/f/6435794/activities/32793590
Descrição: Problema: No endpoint de consulta por usuário, a resposta não retornava os dados do usuário inativo, apenas uma resposta genérica.
Solução: Correção validada em stage, com a consulta retornando as informações do usuário inativo corretamente.

Título: P1 [Migrar API V01>V02] Trocar nomenclatura do endpoint "Restaurar usuário"
Link: https://app2.artia.com/a/4874953/f/6435794/activities/32802416
Descrição: Problema: O nome da rota de ativação de usuário inativo estava incorreto; conforme review do projeto, deveria ser "Ativar usuário".
Solução: Correção validada em stage, com a reativação de usuário e o nome da action da rota ajustados.
