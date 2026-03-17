# DevRoast

Cole seu código. Receba uma nota. E talvez um trauma.

O **DevRoast** é uma ferramenta de avaliação de código que usa IA para analisar trechos enviados pelo usuário e gerar:

- nota de **0 a 10**
- veredito de qualidade
- análise técnica por severidade
- sugestão de código melhorado

Você pode escolher entre dois estilos de feedback:

- **Roast mode (sarcástico):** resposta agressiva e humorística
- **Modo sério (profissional):** resposta direta, construtiva e objetiva

Projeto construído durante o evento **NLW** da [Rocketseat](https://rocketseat.com.br).

---

## O foco da ferramenta

O objetivo principal do DevRoast é transformar revisão de código em algo rápido e didático:

1. você cola um trecho de código
2. a IA avalia qualidade, clareza, riscos e boas práticas
3. o sistema retorna nota, explicações e uma versão sugerida de melhoria

Além do aprendizado individual, o projeto também traz uma camada social com o **Shame Leaderboard**, exibindo os piores scores salvos no banco.

---

## Funcionalidades

- **Editor com syntax highlighting em tempo real**
- **Detecção automática de linguagem** (com opção de seleção manual)
- **Limite de entrada (2000 caracteres)** para manter resposta rápida
- **Avaliação por IA com score 0-10**
- **Itens de análise por severidade:** `critical`, `warning`, `good`
- **Frase-resumo (roastQuote)** adaptada ao modo sarcástico ou sério
- **Suggested fix** com versão completa melhorada do código
- **Página de resultado por submissão** (`/roast/[id]`)
- **OG image dinâmica** para compartilhamento de resultado
- **Leaderboard** com os piores códigos ranqueados por score

---

## Como funciona (arquitetura funcional)

### 1) Submissão

Na página inicial, o usuário envia:

- `code`
- `language`
- `roastMode`

Isso dispara a mutation tRPC `roast.create`.

### 2) Análise com IA

No backend, a rota `roast.create`:

- usa `generateText` (AI SDK)
- aplica schema Zod (`roastOutputSchema`) para validar a resposta estruturada
- muda o comportamento do prompt com base em `roastMode`

O retorno da IA inclui:

- `score`
- `verdict`
- `roastQuote`
- `analysisItems[]`
- `suggestedFix`

### 3) Persistência

Os dados são salvos no PostgreSQL com Drizzle:

- tabela `roasts` (dados principais da submissão)
- tabela `analysis_items` (itens de análise associados)

### 4) Exibição

- a página `/roast/[id]` mostra score, veredito, análise e diff sugerido
- o leaderboard busca os piores scores em ordem crescente
- stats globais exibem total de análises e média das notas

---

## Regras de avaliação

A IA recebe instruções para:

- pontuar de **0.0 a 10.0**
- gerar entre **3 e 6 itens de análise**
- produzir uma versão corrigida do código

Mapeamento de score para veredito:

- `0.0 - 2.0` → `needs_serious_help`
- `2.1 - 4.0` → `rough_around_edges`
- `4.1 - 6.0` → `decent_code`
- `6.1 - 8.0` → `solid_work`
- `8.1 - 10.0` → `exceptional`

---

## Stack e tecnologias

- **Frontend:** Next.js 16 (App Router), React 19, Tailwind CSS v4
- **Renderização de código:** Shiki + Highlight.js (detecção)
- **API:** tRPC v11 + TanStack React Query v5
- **Banco:** PostgreSQL 16 + Drizzle ORM
- **Validação:** Zod
- **IA:** AI SDK + `@ai-sdk/openai`
- **Tooling:** Biome, TypeScript strict, pnpm

---

## Estrutura do projeto

```text
src/
	app/                 # Páginas (home, leaderboard, roast result, OG route)
	components/          # Componentes de feature e UI reutilizável
	db/                  # Schema Drizzle, client e seed
	hooks/               # Detecção de linguagem e highlight
	lib/                 # Prompt/IA e utilitários compartilhados
	trpc/                # Infra tRPC (client, server, routers)
```

Arquivos-chave:

- `src/trpc/routers/roast.ts` → regras de consulta/criação de roast
- `src/lib/ai.ts` → schema de saída e prompt (sarcástico vs sério)
- `src/db/schema.ts` → modelagem das tabelas
- `src/app/home-editor.tsx` → fluxo de submissão do usuário
- `src/app/roast/[id]/page.tsx` → renderização detalhada do resultado

---

## Setup local

### Pré-requisitos

- Node.js 20+
- pnpm
- Docker (para PostgreSQL local)

### 1) Instalar dependências

```bash
pnpm install
```

### 2) Subir banco PostgreSQL

```bash
docker compose up -d
```

### 3) Configurar variáveis de ambiente

Crie o arquivo `.env.local` com:

```env
DATABASE_URL=postgresql://devroast:devroast@localhost:5432/devroast
OPENAI_API_KEY=sua_chave_aqui
```

### 4) Aplicar schema no banco

```bash
pnpm db:push
```

### 5) (Opcional) Popular dados de exemplo

```bash
pnpm db:seed
```

### 6) Rodar o projeto

```bash
pnpm dev
```

Acesse: [http://localhost:3000](http://localhost:3000)

---

## Scripts úteis

- `pnpm dev` — ambiente de desenvolvimento
- `pnpm build` — build de produção
- `pnpm start` — inicia build de produção
- `pnpm lint` — validação com Biome
- `pnpm format` — formatação automática
- `pnpm db:generate` — gera migrations com Drizzle
- `pnpm db:migrate` — aplica migrations
- `pnpm db:push` — sincroniza schema com banco
- `pnpm db:studio` — abre Drizzle Studio
- `pnpm db:seed` — popula dados fake

---

## Sobre o projeto

O DevRoast mistura revisão técnica com entretenimento: ele ajuda a entender problemas reais de código, mas também permite um modo zoeiro para deixar a experiência divertida.

Se você quiser, posso também preparar uma versão **README em inglês** para facilitar publicação internacional no repositório.
