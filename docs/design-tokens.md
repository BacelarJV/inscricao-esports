# 🎨 Tokens de Design

**Projeto:** Sistema de Inscrição para Campeonatos de E-sports  
**Versão:** 1.1.0  
**Última atualização:** 2026-10-14  

> 🤖 **Este documento existe para a IA parar de inventar um botão diferente a cada
> tela.** Não é um design system — é o mínimo que dá à prototipagem assistida algo a
> que obedecer.
>
> ✍️ **Não preencha na mão:** rode `/utf-design` (depois do `/utf-flows`).

---

## Paleta

Nome semântico, nunca `azul-2` — a cor muda, o papel dela não.

| Token | Valor | Onde se usa |
| --- | --- | --- |
| `primaria` | `#3B82F6` (hover: `#2563EB`, acento: `#38BDF8`) | Ação principal, botões de destaque, links e acentos |
| `superficie` | `#171B26` (borda: `#222938`) | Fundo de cards, modais e painéis |
| `fundo` | `#0A0E18` | Fundo global da aplicação / página |
| `texto` | `#F8FAFC` | Texto principal, dados de alto contraste e títulos |
| `texto-suave` | `#94A3B8` (apoio discreto: `#64748B`) | Legendas, rótulos secundários, apoios e placeholders |
| `perigo` | `#EF4444` (fundo container: `#2A1318`) | Erros, alertas críticos, avisos de prazo e cancelamentos |
| `sucesso` | `#10B981` (acento: `#00F2B6`, fundo container: `#0D2320`) | Confirmações, vaga garantida e pagamento aprovado |
| `desabilitado` | `#1E2433` (fundo) / `#475569` (texto/ícone) | Elementos inativos, botões bloqueados e badges neutras |

## Escala de espaçamento

Uma progressão só, usada em tudo (1rem = 16px).

| Token | Valor | Onde se usa |
| --- | --- | --- |
| `xs` | `0.25rem` | Microespaços, tags, separadores internos |
| `sm` | `0.5rem` | Gaps pequenos, padding interno de badges e botões compactos |
| `md` | `1rem` | Espaçamento padrão entre elementos, padding de cards simples |
| `lg` | `1.5rem` | Padding de painéis e cards, espaçamento entre seções |
| `xl` | `2rem` | Respiros de seções e margens de blocos principais |
| `2xl` | `3rem` | Separação entre grandes blocos e contêineres de página |

## Tipografia

Família principal: `Inter`, sans-serif. Tamanhos em rem (1rem = 16px).

| Token | Família · tamanho · peso | Papel |
| --- | --- | --- |
| `titulo-pagina` | `Inter` · 1.75rem · Bold (700) | Cabeçalhos principais de página e nomes de campeonatos |
| `titulo-secao` | `Inter` · 1.25rem · SemiBold (600) | Subtítulos de seções, modais e divisões de etapa |
| `titulo-card` | `Inter` · 1rem · SemiBold (600) | Títulos de times, status e cards de campeonatos |
| `corpo` | `Inter` · 0.875rem · Regular (400) / Medium (500) | Textos corridos, inputs de formulário e descrições |
| `legenda` | `Inter` · 0.75rem · Regular (400) / Medium (500) | Rótulos auxiliares, timestamps e badges de status |

## Estados de botão

| Estado | Aparência |
| --- | --- |
| `normal` | Fundo `#3B82F6` (`primaria`), texto `#F8FAFC`, bordas arredondadas (0.5rem) |
| `hover` | Fundo `#2563EB` (feedback tátil suave) |
| `foco (teclado)` | Anel de contorno externo duplo (`#38BDF8` com respiro de 2px do fundo `#0A0E18`) |
| `desabilitado` | Fundo `#1E2433`, texto `#475569`, sem efeito hover, cursor `not-allowed` |
| `carregando` | Fundo `#3B82F6` com opacidade reduzida, indicador giratório (spinner) ativo, cliques bloqueados |

## Protótipo

**Link:** [Protótipo no Stitch](https://stitch.withgoogle.com/preview/11000728704193360087?node-id=79ea04bf0cec42efa1638948eb9157d7)  
**Telas:**
1. **Login do Capitão**
2. **Cadastro de Capitão**
3. **Registrar Nova Equipe**
4. **Detalhes do Campeonato**
5. **Confirmar Inscrição da Equipe**
6. **Checkout de Pagamento (Sandbox)**
7. **Pagamento em Processamento**
8. **Convidar Jogadores e Composição da Line-up**
9. **Aceite de Convite (link do jogador)**

**Telas futuras (fora do protótipo da Entrega 1):**
- **Criar Campeonato** (US01): o organizador define nome, jogo, taxa, vagas, mínimo e máximo de jogadores e prazo final.
- **Lista de Espera** (US03): mostra a posição na fila e o aviso de 24 horas para pagar quando o time é promovido (RN07).
- **Inscrição Confirmada** (US05): tela final depois do webhook do gateway.

Essas telas seguem os mesmos tokens deste documento e serão prototipadas antes das histórias correspondentes entrarem em desenvolvimento.
