---
name: autopilot
description: Executa um pedido do Fabinho de ponta a ponta no padrão FRC — entender, brainstorm, plano, worktree, TDD, revisão cruzada Claude Code x Codex, PR, merge e ligar com vigia, limpar e fechar com Notion e resumo. Usar quando o pedido vier como /autopilot seguido do pedido.
argument-hint: "<pedido em linguagem normal>"
disable-model-invocation: true
metadata:
  fonte: frc-claude-config/frc/autopilot/fluxo-mestre.md
  versao: 4c19cfb66922
---

<!-- GERADO por frc/autopilot/gerar.py. Não editar à mão. -->

# Autopilot FRC — fluxo-mestre (Claude Code)

Você recebeu um pedido do Fabinho (CEO da FRC, não programa) para executar de ponta a ponta, sozinho, até estar ligado, testado e em uso.

**Pedido:** $ARGUMENTS

Se o pedido veio vazio, pegue o item mais antigo pendente do canal "Skills / Autopilot" (ver etapa 1) ou pare dizendo em uma frase que faltou o pedido.

## Regras fixas (valem em todas as etapas)

1. **Autonomia total.** Login, permissão, token, código por e-mail, instalação, configuração, reinício: você resolve (API, senão Chrome dos agentes, senão o e-mail dele). Nunca peça para ele logar, clicar, rodar, testar ou escolher opção técnica. Nunca entregue "lista para alguém fazer": automatize.
2. **Segredos só do cofre do Supabase** (`frc-cofre-exec NOME VARIAVEL -- comando`). Nunca imprimir valor, nunca gravar em arquivo, nunca pedir ao Fabinho.
3. **Claude Code + Codex juntos.** O outro faz a revisão cruzada (etapas 6 e 7). Subagentes liberados para trabalho paralelo independente (ferramenta Agent, tipo fork ou general-purpose, com escopo estreito).
4. **Automação nasce LIGADA** com dados validados, teste e vigia. Parado ou quebrado = consertar, não contornar.
5. **Agentes têm login próprio de administrador** em todo sistema (senha no cofre). Funcionalidade feita em um sistema: replicar nos outros onde fizer sentido.
6. **Projeto novo nasce com rotina diária própria e canal de melhoria** (sistema no Reportar problemas da Torre: `frc_ops.reportar_sistemas`).
7. **Banco e queries em America/Sao_Paulo; datas para gente em DD-MM-YYYY.** Operação de loja em Orlando: decidir qual fuso vale antes de usar data.
8. **Alertas vão para o painel "🔔 Alertas do Sistema" da Torre** (`insert into frc_ops.automacoes_alertas(setor, tipo, severidade, titulo, detalhe, origem, chave_dedup, categoria)` com `chave_dedup` para não repetir), nunca para o Telegram.
9. **Trava de segurança:** ação irreversível em produção ou em dado sensível (apagar dados, migração destrutiva, dinheiro, mensagem em massa para cliente) = cópia de segurança + quarentena primeiro. Se ainda puder ir para dois lados, pare só essa parte e registre em "Precisa de mim" (`select frc_ops.precisa_adicionar(...)`, com `ja_tentei`), continuando o resto.
10. **Regras globais da FRC continuam valendo** (CLAUDE.md / AGENTS.md): Kommo só via `kommo-proxy-v2`; `hq_*`/`kommo_*` são espelho só leitura; um comando bash por vez; nada publicado ou enviado para fora sem aprovação; financeiro só levantamento; smoke test antes de lote; checar quem mais consome o que você mudar.
11. **Pedido que não é código** (Make, Kommo, n8n, configuração, painel de terceiro): mesmo fluxo pulando só as partes de git (branch, commit, PR, merge), mantendo plano, teste, revisão cruzada, validação e documentação. Marque essas etapas como `pulada` no registro.
12. **Contrato aprende e ensina — regra permanente do CEO, 24-11-2026.** Nenhum agente novo da FRC entra no ar sem registrar ensinamento com resultado ao fim de cada caso/rodada/erro, consultar os ensinamentos aprovados públicos e do seu setor antes de agir e receber regras promovidas. Inclui os agentes de código (Claude Code e Codex). O Super-herói é o único dono do ciclo: recebe, agrupa, testa em um agente, promove, replica e aposenta com evidência e desfazer. Outros agentes propõem ensinamentos; não promovem regras nem mantêm base separada. A fonte é `super_heroi`, com governança em `frc_law`; tabelas antigas só alimentam a fonte por adaptadores com dedupe. Respeite acesso pública/setor/liderança/CEO no servidor e guarde somente identificadores técnicos, sem dados pessoais de clientes. O aprendizado roda por caso e diariamente, sem depender de novo chat. Agente vivo que passar 48 h sem gerar nem consultar ensinamento aparece no placar como **não está aprendendo**; o Super-herói corrige a causa.

**Verificação do caminho real:** esta regra não prova que a integração já foi implantada. Consulte a ficha vigente da Escola dos Agentes e confirme no banco as assinaturas e permissões do contrato antes de chamar funções; não invente nomes de RPC nem use um contrato proposto como publicado. Na implantação inicial, os donos são as tarefas 6660/6662; depois, use o dono vigente na ficha. Se API, recibos ou trava de ativação ainda não existirem, registre a lacuna nessa tarefa técnica, implemente e teste; até isso funcionar, não ative o agente novo. Em pedidos de outro assunto com agentes já vivos, registre a lacuna para o dono da Escola e prossiga com o trabalho autorizado, sem inventar recibo nem declarar o contrato cumprido. O teste negativo precisa exercitar o mesmo caminho real de ativação, com uma peça do contrato ausente em ambiente isolado, e observar a recusa — uma asserção sobre o texto desta skill não substitui essa prova.

## Registro de cada etapa (OBRIGATÓRIO: é o que prova que o fluxo rodou)

**Primeira ação da sessão, antes de ler qualquer coisa:** crie o id `AP-<AAAAMMDD-HHMM>-claude-<palavra-curta>` e rode o registro da etapa 0 com status `inicio`. Depois, **ao fechar cada etapa abaixo (1 a 10), rode o registro dela antes de começar a próxima** (status `ok`, `pulada`, `falha` ou `bloqueio`; a 10 com `fim`). São 11 chamadas no mínimo. Execução que não chega ao `fim` vira alerta na rotina diária, e sem registro não há prova de que o fluxo rodou.

```
frc-autopilot-registro --exec <id> --ferramenta claude --etapa <0-10> --status <inicio|ok|falha|pulada|bloqueio|fim> --pedido "<pedido curto>" --repo "<repo ou ->" --detalhe '{"prova":"..."}'
```

Só se o comando não existir (sessão na nuvem), siga sem registrar e diga isso no resumo final. Falha de registro nunca para o fluxo. Quando você chamar o Codex só para revisar, ele NÃO registra (o registro é de quem executa o pedido).

## As 10 etapas

### 1. ENTENDER → ao fechar: registro etapa 1
- Leia o pedido pensando na intenção real, não só na frase.
- Procure no Notion (Cérebro FRC: "30 — Decisões", "42 — Perguntas abertas", ficha do projeto), na memória, no CLAUDE.md/AGENTS.md do repositório e NA FONTE se isso já foi feito, testado ou decidido. Já existe: reaproveite e diga.
- Veja se outra sessão já está no assunto: `select * from agent_mesh.quadro_tarefas where status not in ('concluida','cancelada') and titulo ilike '%<tema>%'` e `git log origin/main --since=3.days`. Crie ou assuma a tarefa no quadro (`agent_mesh.tarefa_criar` / `tarefa_assumir`).
- Antes de agir, consulte o contrato e os ensinamentos aprovados aplicáveis no Super-herói pelo caminho oficial vigente. O auxiliar `frc-escola consultar --agente cc-claude-code --assunto "<tema>"` pode ser usado para leitura não certificada; respeite a resposta do servidor e não amplie acesso para obter conteúdo. É legado: saída vazia, HTTP 200 ou recibo simples não comprovam consulta nem cumprimento do contrato. Confirme identidade exclusiva do processo, conteúdo recebido antes da ação, vínculo ao caso e validação do resultado na fonte. Enquanto o auxiliar usar credencial compartilhada e não cumprir a versão vigente, registre a lacuna uma vez por assunto na tarefa técnica da Escola, apenas se ainda não estiver aberta; não conte a chamada no placar. Em agentes já vivos, prossiga com o trabalho autorizado e mantenha a lacuna explícita. Falha de acesso não autoriza inventar uma base local. Verifique a versão vigente em `frc_law` e o cadastro do agente.

### 2. BRAINSTORM → ao fechar: registro etapa 2
Liste pré-requisitos, ordem, lacunas que o pedido não disse, e impacto em outros projetos FRC (crons, edges, painéis, n8n, Make, bots que consomem o que vai mudar). Decida pela opção mais segura e reversível; registre o que você supôs.

### 3. PLANO → ao fechar: registro etapa 3
Passos numerados, cada um com **critério de pronto verificável** (comando, consulta SQL ou tela que prova). Grave o plano no Notion do projeto (etapa começou) e na tarefa do quadro.

### 4. ISOLAR → ao fechar: registro etapa 4
Nunca trabalhe na main. `git fetch origin` e crie worktree em `.worktrees/<branch>` do repositório (branch `autopilot/<palavra-curta>`). Pedido sem código: crie a cópia de segurança/ambiente de teste equivalente (rascunho, cenário duplicado, modo sombra).

### 5. EXECUTAR COM TDD → ao fechar: registro etapa 5
Para cada passo: primeiro o teste que falha (unitário, SQL de verificação, chamada real de leitura), depois o código mínimo que passa, depois rodar a suíte. Commits pequenos e padronizados em português: `<tipo>(<area>): <o que>` (tipos: feat, fix, test, docs, chore, refactor), terminando com a linha de coautoria da ferramenta.

Se criar ou alterar agente, integre o contrato aos pontos reais antes da ação e ao resultado do caso, inclusive erro, com chave idempotente. Cadastre a linha no placar, setor, divisão de acesso, rota de ensino e teto diário. Teste ausência de contrato e recibos, autorização por setor e entrega de regra promovida. Cadastro preenchido, log de consulta vazio e teste sintético não provam aprendizado real nem melhora de eficácia.

### 6. REVISÃO CRUZADA → ao fechar: registro etapa 6
Peça ao Codex para revisar o diff inteiro contra a main:

```
codex-leve -C "<pasta do worktree>" "Revisão cruzada do autopilot FRC: rode git -C <pasta do worktree> fetch -q origin e git -C <pasta do worktree> diff origin/main...HEAD, revise correção, segurança, regras FRC (CLAUDE.md/AGENTS.md) e testes. Não edite nada e não rode frc-autopilot-registro. Responda só a lista de achados no formato CRÍTICO|MÉDIO|BAIXO: arquivo:linha — problema — correção, ou a frase SEM ACHADO CRÍTICO."
# sem codex-leve nesta máquina: codex exec --skip-git-repo-check -C "<pasta>" "<mesmo texto>" </dev/null
```

Achado CRÍTICO (quebra, segurança, dado errado, dinheiro, regra FRC violada) = corrija e peça nova revisão. Máximo 3 ciclos; se no 3º ainda houver crítico técnico, não publique: deixe o PR em rascunho, registre o achado no quadro (`agent_mesh.agent_tasks`) para o Codex ou para você mesmo na próxima rodada e grave a etapa com status `bloqueio`. Só vai para "Precisa de mim" se o impasse for decisão que só o Fabinho toma (dinheiro, apagar, mensagem para fora, regra de negócio sem fonte). Achado médio/baixo: corrija o que for barato, anote o resto no PR.

### 7. PR → ao fechar: registro etapa 7
`git push -u origin <branch>` e `gh pr create` (título curto, corpo com o que muda, como foi testado, resultado da revisão cruzada, terminando com a linha de atribuição da ferramenta). Rode a revisão cruzada de novo sobre o PR (mesmo comando da etapa 6, citando o número do PR). Espere todos os checks: `gh pr checks <n> --watch`. Check vermelho = conserte e repita.

### 8. MERGE E LIGAR → ao fechar: registro etapa 8
- `gh pr merge <n> --squash --delete-branch`; na pasta principal: `git checkout main` e `git pull --ff-only` (sempre puxar antes de empurrar qualquer coisa).
- Deploy pelo caminho do projeto (Lovable publica sozinho pela main; edge Supabase com `--no-verify-jwt`; migration pelo MCP/CLI).
- **Antes de ativar agente novo, inclusive em quarentena operacional:** execute a checagem oficial do cadastro/Super-herói e guarde os IDs dos recibos: registro com resultado na fonte, consulta antes de agir, recebimento de regra promovida, linha no placar e rota de ensino ligadas. Comprove também por teste negativo em ambiente isolado que a ativação é recusada quando qualquer parte obrigatória falta. **Sem prova completa, não ligue o agente**: corrija a causa técnica e repita; não peça exceção ao CEO nem marque sucesso só porque o cadastro existe. Se houver um gesto exclusivamente humano, registre-o em `frc_ops.precisa_adicionar` conforme a regra 9; isso não dispensa o contrato. Testes de desenvolvimento sem atuação operacional podem rodar antes desta liberação. Em agentes já vivos, corrija a integração sem desligar silenciosamente a operação.
- Ligue primeiro em quarentena (modo sombra, poucos itens, só leitura ou teto baixo); valide com dado real (1 leitura + 1 escrita + 1 leitura de confirmação); depois libere.
- Deixe vigia: cron/rotina que confere e grava alerta em "🔔 Alertas do Sistema" se quebrar. Registre a automação em `frc_ops.automacoes_registro_upsert(jsonb[])` e agende a conferência em `frc_ops.conferencia_agendar`.
- Reuse a vigia de aprendizado do Super-herói: confira a condição de 48 h, a marca no placar e a correção da causa, sem abrir tarefa humana por alerta de máquina. Prove a rotina por caso e a diária; não crie rotina paralela. Meça custo por agente e por ensinamento promovido; custo não medido é desconhecido, nunca zero.

### 9. LIMPAR → ao fechar: registro etapa 9
Antes de apagar, confirme que o PR está mergeado (`gh pr view <n> --json state,mergedAt` = MERGED) e que a branch não tem commit depois do merge (`git log origin/main..<branch>` só com commits que já estavam no PR). Depois: `git worktree remove .worktrees/<branch>`, `git branch -D <branch>` (o squash muda o hash, por isso `-D` só após essa prova), `git worktree prune`. Nunca apague branch de outra pessoa ou sessão. Feche abas de navegador que abriu.

### 10. FECHAR → ao fechar: registro etapa 10
- Atualize a página do projeto no Notion (estado final + prova) e o diário do dia.
- Registre o ensinamento desta execução com resultado medido e evidência técnica no Super-herói, sem promover por conta própria. Use o contrato vigente ligado à execução real. Enquanto a integração estiver pendente, `frc-escola registrar --agente cc-claude-code --texto "<observação sem dado pessoal>" --evidencia "<id técnico>" --resultado '<métricas medidas em JSON>'` pode alimentar a base como observação não certificada; confirme a linha e seu identificador na fonte, e não marque o contrato como cumprido. O auxiliar legado `frc-escola receber` não certifica aplicação de regra; não use recibo manual para preencher a prova que falta. Confirme na fonte os vínculos de consulta anterior, resultado e recebimento. Métrica não medida permanece desconhecida; observação não vira efeito comprovado. No fechamento, identifique as provas validadas e as lacunas, sem antecipar a janela de avaliação.
- Feche a tarefa no quadro (`agent_mesh.tarefa_atualizar` com status concluida e a prova).
- Entrega para o Fabinho ler vai em `~/Desktop/FRC-Entregas/<projeto>/AAAA-MM-DD-<nome-curto>.md` (`frc-entrega caminho <projeto> "<nome>" --ext md`).
- Registre a etapa 10 com status `fim`.
- Resposta final curta, em linguagem de negócio, três blocos: **Feito** (com prova), **Ligado** (o que está rodando e o vigia), **Precisa de você** (só o que foi para "Precisa de mim", ou "nada").

## Quando algo falha
Falha de ferramenta, check vermelho, acesso negado: tente os outros caminhos (API, tela, outro agente, outra máquina) antes de chamar de bloqueio. Registre `falha` com o motivo. Melhoria que você perceber na própria skill: abra chamado no canal "Skills / Autopilot" (`select public.reportar_problema_v2(...)` com sistema `Skills / Autopilot`) em vez de editar o arquivo gerado; o fonte único é `frc-claude-config/frc/autopilot/fluxo-mestre.md`.
