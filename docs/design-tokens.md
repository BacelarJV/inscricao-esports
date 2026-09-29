# 🎨 Tokens de Design

**Projeto:** Sistema de Inscrição para Campeonatos de E-sports  
**Versão:** 1.0.0  
**Última atualização:** 2026-09-29  

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

Uma progressão só, usada em tudo.

| Token | Valor | Onde se usa |
| --- | --- | --- |
| `xs` | `4px` | Microespaços, tags, separadores internos |
| `sm` | `8px` | Gaps pequenos, padding interno de badges e botões compactos |
| `md` | `16px` | Espaçamento padrão entre elementos, padding de cards simples |
| `lg` | `24px` | Padding de painéis e cards, espaçamento entre seções |
| `xl` | `32px` | Respiros de seções e margens de blocos principais |
| `2xl` | `48px` | Separação entre grandes blocos e contêineres de página |

## Tipografia

Família principal: `Inter`, sans-serif.

| Token | Família · tamanho · peso | Papel |
| --- | --- | --- |
| `titulo-pagina` | `Inter` · 28px · Bold (700) | Cabeçalhos principais de página e nomes de campeonatos |
| `titulo-secao` | `Inter` · 20px · SemiBold (600) | Subtítulos de seções, modais e divisões de etapa |
| `titulo-card` | `Inter` · 16px · SemiBold (600) | Títulos de times, status e cards de campeonatos |
| `corpo` | `Inter` · 14px · Regular (400) / Medium (500) | Textos corridos, inputs de formulário e descrições |
| `legenda` | `Inter` · 12px · Regular (400) / Medium (500) | Rótulos auxiliares, timestamps e badges de status |

## Estados de botão

| Estado | Aparência |
| --- | --- |
| `normal` | Fundo `#3B82F6` (`primaria`), texto `#F8FAFC`, bordas arredondadas (8px) |
| `hover` | Fundo `#2563EB` (feedback tátil suave) |
| `foco (teclado)` | Anel de contorno externo duplo (`#38BDF8` com respiro de 2px do fundo `#0A0E18`) |
| `desabilitado` | Fundo `#1E2433`, texto `#475569`, sem efeito hover, cursor `not-allowed` |
| `carregando` | Fundo `#3B82F6` com opacidade reduzida, indicador giratório (spinner) ativo, cliques bloqueados |

## Protótipo

**Link:** [Protótipo no Stitch](https://stitch.withgoogle.com/preview/11000728704193360087?node-id=d2cf76c9d7d9414cba740122ff3cab60)  
**Telas (Jornada 1):**
1. **Detalhes do Campeonato & Inscrição de Equipe** (listagem de vagas disponíveis e ação principal de confirmação de inscrição)
2. **Checkout da Taxa de Inscrição** (resumo de cobrança com vaga retida até encerramento do prazo)
3. **Confirmação de Inscrição Garantida** (status aprovado, slot travado com equipe confirmada)
4. **Processando Pagamento / Painel de Retomada** (acompanhamento de status e reabertura do checkout)
5. **Lista de Espera** (alerta de encerramento de vagas regulares e posicionamento na fila)
