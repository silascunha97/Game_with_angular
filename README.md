# MeuProjetoFullstack 🚀

Monorepo construído com [Nx](https://nx.dev) focado em escalabilidade, desacoplamento e rigor arquitetural para o ecossistema Web (Angular + NestJS + GraphQL).

---

## 📜 Evolução e Histórico do Projeto

Este projeto iniciou como uma estrutura monorepo padrão voltada para integração entre um ecossistema frontend Angular e backend NestJS/GraphQL. À medida que a visão da plataforma se expandiu para suportar múltiplos mini games, domínios educacionais e módulos de infraestrutura, tornou-se necessária uma arquitetura formal de limites (boundaries).

### Marcos de Decisão Arquitetural:
1. **Consolidação do Core Mínimo:** Decidiu-se por manter apenas as aplicações base reais no repositório (`web-portal` e `api`), evitando a criação antecipada de pastas ou aplicações "fantasmas" (como um *game-engine-host* sem engine) para não gerar débito de manutenção.
2. **Isolamento via Nx Boundaries:** Para evitar acoplamento predatório em projetos de grande porte, foi estabelecida uma política rigorosa de dependências utilizando tags (`scope:*` e `platform:*`) validadas automaticamente via ESLint (`@nx/enforce-module-boundaries`).
3. **Design Orientado a Libraries Futuras:** A pasta `libs/` nasce sob uma convenção clara de *namespaces*. O código de domínio, componentes reutilizáveis e regras de negócio não devem residir nos *apps*, mas sim serem importados de *libraries* isoladas à medida que surgirem as demandas.

---

## 🏗️ Estrutura Atual do Workspace

```text
my-game-platform/
├── apps/
│   ├── web-portal/                # App Frontend principal (React/Next/Angular/Vue)
│   │   ├── src/
│   │   │   ├── app/              # Roteamento (Dashboard, Seletor de Games, Perfil)
│   │   │   ├── main.ts
│   │   │   └── styles/
│   │   └── project.json
│   │
│   ├── game-engine-host/          # App isolado/wrapper para rodar/testar games em iframe ou canvas
│   │   └── ...
│   │
│   └── api/                       # App Backend principal (NestJS/Node)
│       ├── src/
│       │   ├── app/              # Bootstrapping da API e rotas globais
│       │   └── main.ts
│       └── project.json
│
├── libs/
│   ├── shared/                    # Recursos compartilhados por toda a plataforma
│   │   ├── ui/                    # Design System (Botões, Modais, HUDs genéricos)
│   │   ├── data-access/           # Clientes HTTP, gerenciamento de estado global (Auth, User)
│   │   ├── types/                 # Interfaces DTOs compartilhadas (User, Score, GameConfig)
│   │   └── utils/                 # Helpers genéricos (formatação, som, helpers math/canvas)
│   │
│   ├── games/                     # CORE: Onde ficam todos os mini games educacionais
│   │   ├── math-quiz/             # Mini game 1: Ex: Matemática
│   │   │   ├── src/
│   │   │   │   ├── lib/
│   │   │   │   │   ├── components/ # Telas/Canvases específicos do game
│   │   │   │   │   ├── logic/      # Regras de pontuação, verificação de acerto
│   │   │   │   │   └── assets/     # Sprites, áudios específicos do game
│   │   │   │   └── index.ts        # API pública que exporta o game para a web-portal
│   │   │   └── project.json
│   │   │
│   │   ├── physics-lab/           # Mini game 2: Ex: Física/Fase
│   │   │   └── ...
│   │   │
│   │   └── common-game-engine/    # Engine base do game (loop de jogo, gerenciador de áudio, collision)
│   │       ├── src/
│   │       └── index.ts
│   │
│   ├── educational/               # Lógica de domínio educacional
│   │   ├── progress/              # Rastreamento de progresso do aluno / Matriz de competências
│   │   └── metrics/               # Algoritmos de cálculo de pontuação/desempenho educacional
│   │
│   └── backend/                   # Módulos específicos de regras de negócio do Backend
│       ├── auth/                  # Autenticação, Tokens
│       ├── users/                 # Gestão de alunos/professores
│       ├── ranking/               # Leaderboards e pontuações
│       └── game-analytics/        # Registro de métricas de jogo em tempo real (WebSockets / Events)
│
├── tools/                         # Scripts de automação, geradores customizados do Nx
├── nx.json
├── tsconfig.base.json
└── package.json
```

### Responsabilidade de Cada App
- **`apps/web-portal`**: Shell Angular da plataforma. Contém apenas o bootstrap, roteamento base, estilos globais e provider do Apollo Client. Futuramente comporá as telas de domínio e mini games a partir das *libraries* — não deve conter regra de negócio própria.
- **`apps/api`**: Backend NestJS hospedando o Apollo Server. Contém o bootstrap do `GraphQLModule`, `PrismaModule` e resolvers básicos de infraestrutura (`HealthResolver`). Módulos de domínio entrarão como *libraries* em `libs/backend/*`.
- **`apps/*-e2e`**: Testes ponta a ponta isolados por aplicação via `implicitDependencies`.

---

## 🧩 Convenção para Bibliotecas Futuras (`libs/`)

Cada diretório abaixo representa um **namespace**. Nenhuma biblioteca deve ser criada de forma especulativa.

| Diretório | Conteúdo | Tipo de Lib | Tag `scope:*` | Tag `platform:*` |
|---|---|---|---|---|
| `libs/games/<nome>/` | Mini game isolado (lógica, componentes, assets) | `@nx/angular:library` | `scope:game` | `platform:angular` |
| `libs/games/common-game-engine/` | Código genérico compartilhado entre 2+ games | `@nx/angular:library` ou `@nx/js:library` | `scope:game-engine` | *conforme o runtime* |
| `libs/educational/<dominio>/` | Regras de negócio do domínio educacional | `@nx/js:library` | `scope:educational` | `platform:agnostic` |
| `libs/backend/<modulo>/` | Módulos de domínio do backend (auth, users, etc.) | `@nx/nest:library` | `scope:backend` | `platform:node` |
| `libs/shared/ui/` | Componentes Angular genéricos sem domínio | `@nx/angular:library` | `scope:shared` | `platform:angular` |
| `libs/shared/data-access/` | Infra/Clientes de acesso a dados genéricos | `@nx/angular/js:library` | `scope:shared` | *conforme o runtime* |
| `libs/shared/types/` | Tipos TypeScript genéricos reutilizáveis | `@nx/js:library` | `scope:shared` | `platform:agnostic` |
| `libs/shared/utils/` | Funções utilitárias puras | `@nx/js:library` | `scope:shared` | `platform:agnostic` |

> ⚠️ **Atenção sobre Contratos da API:** `libs/shared/types` **não** armazena interfaces espelhadas da API. A fonte de verdade é o GraphQL Schema (gerado via resolvers). O frontend deve consumir tipos gerados automaticamente via GraphQL Code Generator.

---

## 📐 Regras e Direção de Dependências

Para evitar dependências circulares e acoplamentos indevidos, a hierarquia de importações segue estritamente a matriz abaixo:

```text
web-portal (app) ──▶ shared
api (app)        ──▶ shared, backend

game/<nome>       ──▶ shared, game-engine, educational (quando necessário)
game/<nome>       ✕   outro game (PROIBIDO)
game/<nome>       ✕   backend (PROIBIDO)

game-engine       ──▶ shared
backend/<modulo>  ──▶ shared
educational       ──▶ shared (PROIBIDO depender de platform:angular ou platform:node)
shared            ──▶ shared (NÃO pode depender de nenhuma outra camada)
```

### Validação Automatizada (ESLint)
As restrições de importação são garantidas automaticamente através das regras em `eslint.config.mjs` usando `@nx/enforce-module-boundaries`:

```javascript
depConstraints: [
  { sourceTag: "scope:web-portal", onlyDependOnLibsWithTags: ["scope:web-portal", "scope:shared"] },
  { sourceTag: "scope:api",        onlyDependOnLibsWithTags: ["scope:api", "scope:shared", "scope:backend"] },
  { sourceTag: "scope:shared",     onlyDependOnLibsWithTags: ["scope:shared"] },
  { sourceTag: "scope:backend",    onlyDependOnLibsWithTags: ["scope:backend", "scope:shared"] },
  { sourceTag: "scope:educational",onlyDependOnLibsWithTags: ["scope:educational", "scope:shared"] },
  { sourceTag: "scope:game",       onlyDependOnLibsWithTags: ["scope:game-engine", "scope:shared", "scope:educational"] },
  { sourceTag: "scope:game-engine",onlyDependOnLibsWithTags: ["scope:game-engine", "scope:shared"] },
  { sourceTag: "platform:agnostic",notDependOnLibsWithTags: ["platform:angular", "platform:node"] }
]
```

---

## 💻 Comandos e Utilização

### Executando a Aplicação
```sh
# Rodar Frontend (Angular dev server em http://localhost:4200)
npx nx serve web-portal

# Rodar Backend (NestJS + Apollo em http://localhost:3000/api/graphql)
npx nx serve api
```

### Gerando Novas Libraries (com tags obrigatórias)
Para garantir o cumprimento das regras de boundary, utilize sempre os geradores passando as devidas `--tags`:

```sh
# Criar um novo mini game
npx nx g @nx/angular:library libs/games/math-quiz \
  --importPath=@meu-projeto-fullstack/games-math-quiz \
  --tags=scope:game,platform:angular

# Criar módulo backend
npx nx g @nx/nest:library libs/backend/auth \
  --importPath=@meu-projeto-fullstack/backend-auth \
  --tags=scope:backend,platform:node

# Criar biblioteca agnóstica de domínio educacional
npx nx g @nx/js:library libs/educational/progress \
  --importPath=@meu-projeto-fullstack/educational-progress \
  --tags=scope:educational,platform:agnostic
```

---

## 🚫 Decisões de Escopo e "Não Implementações"

Por escolha explícita de arquitetura YAGNI (*You Aren't Gonna Need It*), os seguintes pontos **não foram implementados antecipadamente**:
- Mini games ou engines fictícias.
- Módulos vazios de regra de negócio (auth, analytics, ranking).
- Pastas físicas dentro de `libs/` sem uso imediato.

Toda a infraestrutura atual foi projetada para suportar a adição contínua dessas demandas sem a necessidade de refatorar a estrutura base do repositório.
