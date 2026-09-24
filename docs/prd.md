# 📄 Product Requirements Document (PRD)

**Projeto:** Sistema de Inscrição para Campeonatos de E-sports  
**Versão:** 1.0.0  
**Última atualização:** 2026-09-24  

> 🤖 **Este documento é a fonte da verdade sobre o QUE o produto faz.** Regra de
> negócio que não estiver aqui não existe — nem para a equipe, nem para a IA.
> Tecnologia **não** se discute aqui: isso é assunto do `architecture.md`.

---

## 🎯 1. Visão Geral e Objetivo

**O problema:** Organizações de eventos de e-sports enfrentam dificuldades com processos manuais e descentralizados no cadastro de equipes, conferência de pagamentos e validação das vagas dos torneios.

**A solução:** Uma plataforma onde organizadores publicam campeonatos e capitães inscrevem suas equipes, realizam o pagamento da taxa de inscrição de forma automatizada e gerenciam o elenco convidando os jogadores.

**Como saberemos que deu certo:** Um capitão consegue cadastrar sua equipe, inscrevê-la em um campeonato disponível, efetuar o pagamento da taxa e ter a inscrição confirmada automaticamente no torneio, ficando apto a convidar os jogadores para o elenco.

---

## 📖 2. Glossário Ubíquo

> Os termos do negócio, como o cliente fala. É daqui que o `architecture.md`
> deriva os nomes das entidades.

| Termo | Significa | Não confundir com |
| :---- | :-------- | :---------------- |
| **Campeonato** | Evento competitivo com regras, vagas limitadas e taxa de inscrição definida. | Partida individual ou Fase do torneio. |
| **Time (Equipe)** | Agrupamento de jogadores registrado por um capitão para disputar torneios. | Organização do torneio ou Inscrição. |
| **Capitão** | Usuário líder da equipe, responsável por criar o time, inscrevê-lo, pagar a taxa e convidar jogadores. | Jogador convidado comum ou Organizador. |
| **Jogador** | Usuário que recebe e aceita convite para compor o elenco de um time. | Capitão (não possui poderes de gestão ou pagamento do time). |
| **Organizador** | Usuário/entidade responsável por criar o campeonato e gerenciar suas inscrições. | Capitão ou Jogador. |
| **Inscrição** | Vínculo de um time a um campeonato específico, garantida após confirmação de pagamento. | Cadastro do time na plataforma. |
| **Pagamento** | Transação financeira que liquida a taxa de inscrição e valida a vaga do time no campeonato. | Premiação do campeonato. |

---

## 👤 3. Atores e Permissões

> ⚠️ A coluna **"Não pode"** vira Guard e controle de role na API.

| Ator | Quem é | Pode | Não pode |
| :--- | :----- | :--- | :------- |
| **Organizador** | Responsável pelo evento de e-sports. | Criar e publicar campeonatos; definir taxa de inscrição e limite de vagas; visualizar inscrições e status de pagamento. | Inscrever times em campeonatos de terceiros; alterar escalação dos times inscritos; aprovar pagamentos manualmente. |
| **Capitão** | Líder da equipe inscrita ou a inscrever. | Criar seu time; inscrever o time em campeonatos abertos; efetuar pagamento da taxa de inscrição; convidar e remover jogadores do elenco do seu time. | Criar campeonatos; alterar taxa de torneio; inscrever times que não sejam o seu; convidar membros para times de outros capitães. |
| **Jogador** | Atleta convidado para compor um elenco. | Visualizar convites recebidos; aceitar ou recusar convites de times; visualizar os detalhes do campeonato e do seu time. | Inscrever o time em campeonatos; pagar taxa de inscrição em nome do time; convidar outros membros; criar campeonatos. |

---

## 📝 4. Escopo Funcional (User Stories)

> Cada story carrega dois eixos:
> **Prioridade (MoSCoW)** — `Must Have` é o escopo comprometido do projeto
> (o escopo mínimo da ficha é `Must Have` por definição); `Should`/`Could`
> entram se sobrar tempo, mas ficam documentadas; o `Won't Have` vira item da
> seção *Fora de Escopo* — e **Tamanho (esforço)** — `S` cabe numa sessão,
> `M` vira algumas tarefas no plano, `L` pede divisão.
> Toda story nasce `⚪ Draft` — **só o aluno promove a `🟡 Ready`**; `🟢 Live`
> só depois do PR mesclado (o auditor final confere).

### US01 — Criação de Campeonato · `Must Have` · `M` · Status: `⚪ Draft`

<!-- Status: `⚪ Draft` (não codificar) · `🟡 Ready` (vira Issue) · `🟢 Live` (PR mesclado) -->

**Como** Organizador, **eu quero** criar um campeonato definindo nome, taxa de inscrição, número máximo de vagas, número mínimo e máximo de jogadores por time e período de inscrição.

**Critérios de aceite:**
- [ ] **Dado** que informo mínimo de jogadores maior que o máximo, quando submeto a criação, então o sistema rejeita com erro de validação.
- [ ] **Dado** que sou um organizador autenticado, **quando** submeto dados válidos (nome, taxa > 0, vagas >= 2 e datas válidas), **então** o campeonato é criado com status "Aberto" e listado para inscrições.
- [ ] **Dado** dados inválidos (ex.: taxa negativa, vagas zeradas ou data de fim anterior à de início), **quando** tento criar o torneio, **então** a criação é rejeitada com mensagem de erro descritiva.
- [ ] **Dado** um usuário com perfil comum (não organizador), **quando** tenta criar um campeonato, **então** o acesso é bloqueado com erro 403 Forbidden.

**Regras relacionadas:** RN02

---

### US02 — Cadastro de Equipe · `Must Have` · `S` · Status: `⚪ Draft`

**Como** Capitão, **eu quero** cadastrar uma equipe com nome e tag **para que** eu possa representar meu time nos campeonatos.

**Critérios de aceite:**

- [ ] **Dado** que sou um usuário autenticado, **quando** informo um nome e tag únicos, **então** a equipe é criada associando meu perfil como Capitão.
- [ ] **Dado** que o nome ou tag da equipe já esteja em uso por outro time, **quando** tento submeter o cadastro, **então** a criação é recusada com erro de conflito.
- [ ] **Dado** que envio campos em branco ou fora do tamanho limite, **quando** tento cadastrar, **então** o sistema rejeita a criação antes de persistir os dados.

**Regras relacionadas:** RN04

---

### US03 — Inscrição da Equipe no Torneio · `Must Have` · `M` · Status: `⚪ Draft`

**Como** Capitão, **eu quero** inscrever minha equipe em um campeonato aberto **para que** seja gerado o pedido de inscrição para posterior pagamento.

**Critérios de aceite:**

- [ ] **Dado** que sou capitão de um time e o campeonato está com inscrições abertas e vagas disponíveis, **quando** solicito a inscrição, **então** uma inscrição com status "Pendente de Pagamento" é gerada vinculando meu time ao campeonato.
- [ ] **Dado** que o campeonato já atingiu o limite de vagas disponíveis, **quando** solicito a inscrição, **então** o time entra na lista de espera com status "Aguardando Vaga", sem ser recusado.
- [ ] **Dado** que minha equipe já possui inscrição ativa (pendente ou confirmada) no mesmo torneio, **quando** tento inscrever novamente, **então** a operação é impedida com mensagem de inscrição duplicada.
- [ ] **Dado** que uma vaga é liberada (inscrição expirada ou cancelada), **quando** há times na lista de espera, **então** o primeiro da fila é promovido automaticamente para "Pendente de Pagamento".

**Regras relacionadas:** RN01, RN02, RN04

---

### US04 — Pagamento da Taxa de Inscrição · `Must Have` · `M` · Status: `⚪ Draft`

**Como** Capitão, **eu quero** iniciar o pagamento da taxa da inscrição pendente **para que** eu possa quitar o valor e assegurar a vaga do meu time.

**Critérios de aceite:**

- [ ] **Dado** uma inscrição pendente do meu time, **quando** inicio o checkout de pagamento, **então** é gerada a cobrança/sessão no gateway com o valor exato da taxa do torneio.
- [ ] **Dado** que a inscrição já foi paga anteriormente, **quando** tento gerar nova cobrança, **então** o sistema bloqueia a ação informando que a inscrição já está quitada.
- [ ] **Dado** que a janela de inscrições do campeonato foi encerrada antes da tentativa de pagamento, **quando** tento pagar, **então** a cobrança não é permitida e a inscrição é expirada.

**Regras relacionadas:** RN02, RN03, RN04

---

### US05 — Confirmação de Inscrição via Webhook · `Must Have` · `M` · Status: `⚪ Draft`

**Como** Sistema / Organizador, **eu quero** processar as notificações assíncronas do gateway de pagamento **para que** o status da inscrição seja atualizado automaticamente para "Confirmada".

**Critérios de aceite:**

- [ ] **Dado** que o gateway envia um webhook com assinatura válida indicando pagamento aprovado, **quando** o evento é processado, **então** o status do pagamento é registrado como "Aprovado" e a inscrição passa para "Confirmada", garantindo a vaga do time.
- [ ] **Dado** um webhook com assinatura inválida ou cabeçalho adulterado, **quando** a notificação chega, **então** o sistema rejeita a requisição (status 401/400) e nenhuma alteração de status ocorre.
- [ ] **Dado** um webhook notificando recusa ou falha no pagamento, **quando** processado, **então** o status do pagamento é marcado como "Recusado" e a inscrição permanece não confirmada.

**Regras relacionadas:** RN03, RN06

---

### US06 — Convidar Jogadores para o Elenco · `Must Have` · `M` · Status: `⚪ Draft`

**Como** Capitão, **eu quero** convidar jogadores para o elenco do meu time **para que** possamos completar a escalação necessária para disputar o campeonato.

**Critérios de aceite:**

- [ ] **Dado** que sou o capitão do time, **quando** envio convite informando o identificador/e-mail de um jogador válido, **então** um convite com status "Pendente" é registrado para aquele jogador.
- [ ] **Dado** que o jogador convidado já faz parte da equipe ou já possui convite pendente para a mesma equipe, **quando** tento enviar o convite, **então** a ação é barrada com aviso de duplicidade.
- [ ] **Dado** que o limite máximo de membros da equipe foi atingido, **quando** o capitão tenta enviar um novo convite, **então** o envio é bloqueado por limite de elenco.

**Regras relacionadas:** RN04, RN05

---

### US07 — Aceite ou Recusa de Convite pelo Jogador · `Must Have` · `S` · Status: `⚪ Draft`

**Como** Jogador, **eu quero** responder aos convites de equipe recebidos **para que** eu decida se faço ou não parte do time.

**Critérios de aceite:**

- [ ] **Dado** que tenho um convite pendente, **quando** escolho "Aceitar", **então** meu vínculo com a equipe é efetivado como membro ativo e o convite marcado como "Aceito".
- [ ] **Dado** que tenho um convite pendente, **quando** escolho "Recusar", **então** o convite é arquivado como "Recusado" sem criar vínculo com o time.
- [ ] **Dado** que o jogador não possui convites pendentes, **quando** acessa a área de convites, **então** é exibida uma mensagem de lista vazia.

**Regras relacionadas:** RN05

---

### US08 — Painel Público de Equipes Confirmadas · `Could Have` · `S` · Status: `⚪ Draft`

**Como** Visitante ou Competidor, **eu quero** visualizar a lista de equipes com inscrição confirmada no campeonato **para que** eu possa acompanhar quem já garantiu vaga.

**Critérios de aceite:**

- [ ] **Dado** um campeonato existente, **quando** qualquer usuário acessa os detalhes do torneio, **então** é exibida a lista de equipes cuja inscrição está com status "Confirmada".
- [ ] **Dado** que nenhuma equipe teve inscrição confirmada até o momento, **quando** a página é acessada, **então** exibe-se um estado de lista vazia amigável ("Nenhuma equipe confirmada até o momento").
- [ ] **Dado** uma equipe com inscrição "Pendente", **quando** a lista pública é consultada, **então** essa equipe NÃO aparece entre as vagas confirmadas.

**Regras relacionadas:** RN03

### US09 — Encerramento Automático por Elenco Incompleto · `Must Have` · `M` · Status: `⚪ Draft`

**Como** Sistema, **eu quero** verificar se cada time atingiu o mínimo de jogadores até o prazo de inscrição **para que** vagas de times incompletos sejam liberadas automaticamente.

**Critérios de aceite:**

- [ ] **Dado** que o prazo de inscrição se encerra e o time não atingiu o mínimo de jogadores configurado, **quando** a verificação roda, **então** a inscrição é cancelada automaticamente e a vaga é liberada.
- [ ] **Dado** que o time atingiu o mínimo antes do prazo, **quando** o prazo se encerra, **então** a inscrição permanece válida.

**Regras relacionadas:** RN05, RN08

---

## 🛡️ 5. Regras de Negócio (Constraints)

| ID | Regra | Histórias Relacionadas |
| :-- | :---- | :--------------------- |
| **RN01** | **Unicidade de Inscrição:** Uma equipe só pode possuir uma única inscrição (pendente ou confirmada) por campeonato. | US03 |
| **RN02** | **Capacidade e Prazo de Vagas:** Inscrições só podem ser abertas ou pagas enquanto o campeonato estiver com status "Aberto", dentro do prazo limite e com vagas disponíveis. | US01, US03, US04 |
| **RN03** | **Confirmação Exclusiva por Pagamento:** A vaga do time no campeonato só é garantida e alterada para "Confirmada" após a confirmação de liquidação via webhook do gateway de pagamento. | US04, US05, US08 |
| **RN04** | **Liderança Exclusiva do Capitão:** Somente o criador/capitão da equipe tem poderes para inscrever o time, gerar ordens de pagamento e convidar/remover membros do elenco. | US02, US03, US04, US06 |
| **RN05** | **Composição e Limite de Elenco:** O elenco da equipe possui um limite máximo de membros (ex.: 5 titulares + até 2 reservas). Um jogador não pode ser adicionado em duplicidade ao mesmo time. | US06, US07 |
| **RN06** | **Imutabilidade de Inscrição Quitada:** Uma vez que o pagamento foi aprovado e a vaga confirmada, o status não pode ser alterado de volta para "Pendente" nem cancelado unilateralmente pelo capitão. | US05 |
| **RN07** | **Lista de Espera:** Quando as vagas se esgotam, novas inscrições entram em fila ordenada por ordem de chegada; vaga liberada promove automaticamente o primeiro da fila. | US03 |
| **RN08** | **Mínimo de Elenco Obrigatório:** Toda inscrição deve atingir o número mínimo de jogadores configurado pelo organizador até o fim do prazo, sob pena de cancelamento automático. | US06, US09 |

---

## 🚫 6. Fora de Escopo (Non-goals)

> O que o produto deliberadamente **não** faz neste semestre — o `Won't Have`
> do MoSCoW, com o motivo de cada corte.

- **Chaveamento Automático de Partidas (Won't Have):** O sistema não realiza chaveamento de torneio (brackets/tabelas), matchmaking ou controle de pontuação/vencedores das partidas.  
  *Motivo:* O escopo do projeto foca estritamente na fase de organização pré-torneio (publicação, cadastro de times, pagamento da taxa de inscrição e formação do elenco).
- **Split Payment (Divisão de Taxa entre Jogadores):** O sistema não divide a cobrança da taxa de inscrição entre os membros do time; o pagamento é gerado sob responsabilidade exclusiva do Capitão.  
  *Motivo:* Simplificar o fluxo de checkout e integração com o gateway de pagamento.
- **Integração com APIs Oficiais de Jogos (Riot, Valve, etc.):** Não haverá validação automática de Nickname/Rank diretamente nos servidores das desenvolvedoras dos jogos.  
  *Motivo:* O sistema adota o modelo de jogo genérico, evitando dependência de credenciais restritas de publishers externos.
- **Estorno/Reembolso Automático no Gateway:** O cancelamento com devolução automática de taxa não será automatizado via painel da plataforma.  
  *Motivo:* Eventuais casos de desistência ou cancelamento serão tratados administrativamente fora do sistema neste primeiro momento.

---

## ⚙️ 7. Requisitos Não Funcionais (Qualidade)

> Só os que você consegue justificar na defesa.

- **RNF01 — Segurança e Controle de Acesso:** O sistema deve implementar autenticação segura e controle estrito de permissões baseado nos papéis (Organizador, Capitão, Jogador), impedindo que usuários executem ações não autorizadas (ex.: capitão alterar torneio ou jogador inscrever time).
- **RNF02 — Integridade e Segurança em Transações:** A integração com o gateway de pagamento deve operar em ambiente seguro (HTTPS); nenhuma informação sensível de cartão de crédito será armazenada no banco do sistema; e as notificações de webhook devem ser obrigatoriamente validadas via verificação de assinatura criptográfica.
- **RNF03 — Idempotência no Tratamento de Pagamentos:** O recebimento de webhooks de pagamento deve ser idempotente, garantindo que o reenvio da mesma notificação pelo gateway não cause duplicidade de processamento nem inconsistência no status da vaga.
- **RNF04 — Responsividade:** As interfaces de usuário devem ser responsivas, adaptando-se a telas de desktop e dispositivos móveis (permitindo que capitães e jogadores consultem inscrições e aceitem convites pelo celular).

---

## 🛠️ 8. Histórico

| Data | Versão | O que mudou |
| :--- | :----- | :---------- |
| 2026-09-24 | 1.0.0 | Versão inicial gerada e aprovada via entrevista `/utf-prd` |
| 2026-09-24 | 1.1.0 | Ajustes pós-revisão: lista de espera, mínimo de jogadores por time, US07 promovida a Must Have, remoção de non-goal de streaming |
