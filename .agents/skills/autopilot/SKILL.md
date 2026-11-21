---
name: autopilot
description: Executa um pedido do Fabinho de ponta a ponta no padrão FRC — entender, brainstorm, plano, worktree, TDD, revisão cruzada Claude Code x Codex, PR, merge e ligar com vigia, limpar e fechar com Notion e resumo. Usar quando o pedido vier como $autopilot seguido do pedido.
metadata:
  short-description: Pedido de ponta a ponta no padrão FRC
  fonte: frc-claude-config/frc/autopilot/fluxo-mestre.md
  versao: 11d591dfec04
---

<!-- GERADO por frc/autopilot/gerar.py. Não editar à mão. -->

# Autopilot FRC — fluxo-mestre (Codex)

Você recebeu um pedido do Fabinho (CEO da FRC, não programa) para executar de ponta a ponta, sozinho, até estar ligado, testado e em uso.

**Pedido:** o texto que veio junto de `$autopilot` na mensagem do usuário

Se o pedido veio vazio, pegue o item mais antigo pendente do canal "Skills / Autopilot" (ver etapa 1) ou pare dizendo em uma frase que faltou o pedido.

## Regras fixas (valem em todas as etapas)

1. **Autonomia total.** Login, permissão, token, código por e-mail, instalação, configuração, reinício: você resolve (API, senão Chrome dos agentes, senão o e-mail dele). Nunca peça para ele logar, clicar, rodar, testar ou escolher opção técnica. Nunca entregue "lista para alguém fazer": automatize.
2. **Segredos só do cofre do Supabase** (`frc-cofre-exec NOME VARIAVEL -- comando`). Nunca imprimir valor, nunca gravar em arquivo, nunca pedir ao Fabinho.
3. **Codex + Claude Code juntos.** O outro faz a revisão cruzada (etapas 6 e 7). Subagentes liberados para trabalho paralelo independente (subagentes do Codex, quando habilitados, com escopo estreito).
4. **Automação nasce LIGADA** com dados validados, teste e vigia. Parado ou quebrado = consertar, não contornar.
5. **Agentes têm login próprio de administrador** em todo sistema (senha no cofre). Funcionalidade feita em um sistema: replicar nos outros onde fizer sentido.
6. **Projeto novo nasce com rotina diária própria e canal de melhoria** (sistema no Reportar problemas da Torre: `frc_ops.reportar_sistemas`).
7. **Banco e queries em America/Sao_Paulo; datas para gente em DD-MM-YYYY.** Operação de loja em Orlando: decidir qual fuso vale antes de usar data.
8. **Alertas vão para o painel "🔔 Alertas do Sistema" da Torre** (`insert into frc_ops.automacoes_alertas(setor, tipo, severidade, titulo, detalhe, origem, chave_dedup, categoria)` com `chave_dedup` para não repetir), nunca para o Telegram.
9. **Trava de segurança:** ação irreversível em produção ou em dado sensível (apagar dados, migração destrutiva, dinheiro, mensagem em massa para cliente) = cópia de segurança + quarentena primeiro. Se ainda puder ir para dois lados, pare só essa parte e registre em "Precisa de mim" (`select frc_ops.precisa_adicionar(...)`, com `ja_tentei`), continuando o resto.
10. **Regras globais da FRC continuam valendo** (CLAUDE.md / AGENTS.md): Kommo só via `kommo-proxy-v2`; `hq_*`/`kommo_*` são espelho só leitura; um comando bash por vez; nada publicado ou enviado para fora sem aprovação; financeiro só levantamento; smoke test antes de lote; checar quem mais consome o que você mudar.
11. **Pedido que não é código** (Make, Kommo, n8n, configuração, painel de terceiro): mesmo fluxo pulando só as partes de git (branch, commit, PR, merge), mantendo plano, teste, revisão cruzada, validação e documentação. Marque essas etapas como `pulada` no registro.

## Registro de cada etapa (uso e falhas)

No começo crie um id: `AP-<AAAAMMDD-HHMM>-codex-<palavra-curta>`. Ao terminar cada etapa (e em falha ou bloqueio) rode:

```
frc-autopilot-registro --exec <id> --ferramenta codex --etapa <0-10> --status <inicio|ok|falha|pulada|bloqueio|fim> --pedido "<pedido curto>" --repo "<repo ou ->" --detalhe '{"prova":"..."}'
```

Se o comando não existir (sessão na nuvem), siga sem registrar e diga isso no resumo final. Falha de registro nunca para o fluxo.

## As 10 etapas

### 1. ENTENDER
- Leia o pedido pensando na intenção real, não só na frase.
- Procure no Notion (Cérebro FRC: "30 — Decisões", "42 — Perguntas abertas", ficha do projeto), na memória, no CLAUDE.md/AGENTS.md do repositório e NA FONTE se isso já foi feito, testado ou decidido. Já existe: reaproveite e diga.
- Veja se outra sessão já está no assunto: `select * from agent_mesh.quadro_tarefas where status not in ('concluida','cancelada') and titulo ilike '%<tema>%'` e `git log origin/main --since=3.days`. Crie ou assuma a tarefa no quadro (`agent_mesh.tarefa_criar` / `tarefa_assumir`).

### 2. BRAINSTORM
Liste pré-requisitos, ordem, lacunas que o pedido não disse, e impacto em outros projetos FRC (crons, edges, painéis, n8n, Make, bots que consomem o que vai mudar). Decida pela opção mais segura e reversível; registre o que você supôs.

### 3. PLANO
Passos numerados, cada um com **critério de pronto verificável** (comando, consulta SQL ou tela que prova). Grave o plano no Notion do projeto (etapa começou) e na tarefa do quadro.

### 4. ISOLAR
Nunca trabalhe na main. `git fetch origin` e crie worktree em `.worktrees/<branch>` do repositório (branch `autopilot/<palavra-curta>`). Pedido sem código: crie a cópia de segurança/ambiente de teste equivalente (rascunho, cenário duplicado, modo sombra).

### 5. EXECUTAR COM TDD
Para cada passo: primeiro o teste que falha (unitário, SQL de verificação, chamada real de leitura), depois o código mínimo que passa, depois rodar a suíte. Commits pequenos e padronizados em português: `<tipo>(<area>): <o que>` (tipos: feat, fix, test, docs, chore, refactor), terminando com a linha de coautoria da ferramenta.

### 6. REVISÃO CRUZADA
Peça ao Claude Code para revisar o diff inteiro contra a main:

```
cd ~ && claude -p --add-dir "<pasta do worktree>" --allowedTools "Read Grep Glob Bash(git -C:*)" "Revisão cruzada do autopilot FRC: rode git -C <pasta do worktree> fetch -q origin e git -C <pasta do worktree> diff origin/main...HEAD, revise correção, segurança, regras FRC (CLAUDE.md/AGENTS.md) e testes. Não edite nada. Responda só a lista de achados no formato CRÍTICO|MÉDIO|BAIXO: arquivo:linha — problema — correção, ou a frase SEM ACHADO CRÍTICO."
```

Achado CRÍTICO (quebra, segurança, dado errado, dinheiro, regra FRC violada) = corrija e peça nova revisão. Máximo 3 ciclos; se no 3º ainda houver crítico técnico, não publique: deixe o PR em rascunho, registre o achado no quadro (`agent_mesh.agent_tasks`) para o Claude Code ou para você mesmo na próxima rodada e grave a etapa com status `bloqueio`. Só vai para "Precisa de mim" se o impasse for decisão que só o Fabinho toma (dinheiro, apagar, mensagem para fora, regra de negócio sem fonte). Achado médio/baixo: corrija o que for barato, anote o resto no PR.

### 7. PR
`git push -u origin <branch>` e `gh pr create` (título curto, corpo com o que muda, como foi testado, resultado da revisão cruzada, terminando com a linha de atribuição da ferramenta). Rode a revisão cruzada de novo sobre o PR (mesmo comando da etapa 6, citando o número do PR). Espere todos os checks: `gh pr checks <n> --watch`. Check vermelho = conserte e repita.

### 8. MERGE E LIGAR
- `gh pr merge <n> --squash --delete-branch`; na pasta principal: `git checkout main` e `git pull --ff-only` (sempre puxar antes de empurrar qualquer coisa).
- Deploy pelo caminho do projeto (Lovable publica sozinho pela main; edge Supabase com `--no-verify-jwt`; migration pelo MCP/CLI).
- Ligue primeiro em quarentena (modo sombra, poucos itens, só leitura ou teto baixo); valide com dado real (1 leitura + 1 escrita + 1 leitura de confirmação); depois libere.
- Deixe vigia: cron/rotina que confere e grava alerta em "🔔 Alertas do Sistema" se quebrar. Registre a automação em `frc_ops.automacoes_registro_upsert(jsonb[])` e agende a conferência em `frc_ops.conferencia_agendar`.

### 9. LIMPAR
Antes de apagar, confirme que o PR está mergeado (`gh pr view <n> --json state,mergedAt` = MERGED) e que a branch não tem commit depois do merge (`git log origin/main..<branch>` só com commits que já estavam no PR). Depois: `git worktree remove .worktrees/<branch>`, `git branch -D <branch>` (o squash muda o hash, por isso `-D` só após essa prova), `git worktree prune`. Nunca apague branch de outra pessoa ou sessão. Feche abas de navegador que abriu.

### 10. FECHAR
- Atualize a página do projeto no Notion (estado final + prova) e o diário do dia.
- Feche a tarefa no quadro (`agent_mesh.tarefa_atualizar` com status concluida e a prova).
- Entrega para o Fabinho ler vai em `~/Desktop/FRC-Entregas/<projeto>/AAAA-MM-DD-<nome-curto>.md` (`frc-entrega caminho <projeto> "<nome>" --ext md`).
- Registre a etapa 10 com status `fim`.
- Resposta final curta, em linguagem de negócio, três blocos: **Feito** (com prova), **Ligado** (o que está rodando e o vigia), **Precisa de você** (só o que foi para "Precisa de mim", ou "nada").

## Quando algo falha
Falha de ferramenta, check vermelho, acesso negado: tente os outros caminhos (API, tela, outro agente, outra máquina) antes de chamar de bloqueio. Registre `falha` com o motivo. Melhoria que você perceber na própria skill: abra chamado no canal "Skills / Autopilot" (`select public.reportar_problema_v2(...)` com sistema `Skills / Autopilot`) em vez de editar o arquivo gerado; o fonte único é `frc-claude-config/frc/autopilot/fluxo-mestre.md`.
