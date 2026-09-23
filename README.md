# Contador Fácil

**Conte qualquer coisa. De forma simples.**

Aplicativo web responsivo para criar e usar contadores personalizados em qualquer situação: pessoas, animais, exercícios, repetições, produtos, caixas, estoque, itens vendidos, produção, pontos, voltas, tarefas — qualquer quantidade que você queira acompanhar.

O fluxo principal é direto: **abrir → escolher ou criar um contador → tocar no botão → contar.**

## Funcionalidades

- Criar quantos contadores quiser, com o nome que fizer sentido.
- Incrementar (+1) e decrementar (−1) com botões grandes, fáceis de tocar.
- O valor pode chegar a zero, mas nunca fica negativo.
- Zerar o contador (com confirmação).
- Editar o nome do contador.
- Excluir o contador (com confirmação).
- Histórico por contador, com os registros mais recentes primeiro
  (`Contador criado — 0`, `+1 — 1`, `-1 — 1`, `Contador zerado — 0`, `Nome alterado de "A" para "B"`).
- Tudo salvo localmente no dispositivo e mantido depois de fechar e reabrir o app.
- Modo claro (padrão) e modo escuro, com a escolha salva no dispositivo.

## Tecnologias

- React 19 + TypeScript
- Vite
- Tailwind CSS + componentes shadcn/ui
- React Router
- Armazenamento local do navegador (`localStorage`)

## Como rodar

Requisitos: Node.js 20+ e pnpm.

```bash
pnpm install
pnpm dev        # ambiente de desenvolvimento
pnpm build      # build de produção
pnpm lint       # ESLint
pnpm check      # lint + checagem de tipos
```

## Estrutura do projeto

```
src/
  components/              # componentes de interface do app
    ui/                    # componentes base (shadcn/ui)
    counter-card.tsx       # cartão do contador na tela inicial
    create-counter-dialog.tsx
    rename-counter-dialog.tsx
    history-dialog.tsx
    confirm-dialog.tsx     # confirmação reutilizável (zerar/excluir)
    ad-slot.tsx            # reserva para o futuro banner do AdMob
  hooks/                   # use-counters, use-theme
  lib/                     # regras de negócio, armazenamento e utilitários
  pages/
    home/                  # tela inicial
    counter/               # tela individual (/contador/:id)
  providers/               # estado compartilhado (React Context)
  types/                   # tipos do domínio
```

## Arquitetura

A lógica de negócio é **independente da interface**, para facilitar a adaptação futura para React Native/Expo:

| Camada | Arquivo | Responsabilidade |
| --- | --- | --- |
| Domínio | `src/types/counter.ts` | Tipos `Counter` e `CounterEvent` |
| Regras | `src/lib/counters.ts` | Funções puras: criar, incrementar, decrementar, zerar, renomear, ordenar |
| Persistência | `src/lib/counter-repository.ts` | Leitura/gravação e validação dos dados salvos |
| Armazenamento | `src/lib/storage.ts` | Única camada que fala com o `localStorage` |
| Histórico | `src/lib/history.ts` | Textos e datas dos registros |
| Estado | `src/providers/counters-provider.tsx` | Estado compartilhado entre as telas |
| Interface | `src/components`, `src/pages` | Apenas apresentação e interação |

Nenhuma regra de contagem vive dentro de um componente visual.

## Dados salvos

Cada contador é salvo no formato:

```ts
interface Counter {
  id: string;
  name: string;
  value: number;      // nunca menor que zero
  createdAt: number;
  updatedAt: number;
  history: CounterEvent[]; // últimos 100 registros
}
```

Os dados ficam em uma única chave do armazenamento local (`contador-facil:contadores:v1`), sempre pela camada `src/lib/storage.ts`.

## Privacidade

O aplicativo não tem login, servidor, banco de dados externo, API, analytics ou coleta de dados pessoais. Tudo funciona no dispositivo do usuário.

## Preparação para React Native/Expo

A base já está organizada para o porte. Ao criar o projeto Expo, o que muda é apenas a casca:

1. **Armazenamento:** crie um `asyncStorage` implementando a interface `KeyValueStorage` de `src/lib/storage.ts` (usando `@react-native-async-storage/async-storage`) e troque a implementação exportada. As regras e o repositório não mudam.
2. **Identificadores:** `src/lib/id.ts` isola a geração de `id` (o React Native não garante `crypto.randomUUID()`).
3. **Navegação:** troque as rotas do React Router (`/` e `/contador/:id`) pelas telas equivalentes do React Navigation.
4. **Tema:** `src/lib/theme.ts` isola a aplicação do tema; no React Native, use `useColorScheme`/`Appearance` no lugar da classe `dark` do `<html>`.
5. **Interface:** os componentes visuais (`src/components`, `src/pages`) são reescritos com `View`/`Text`/`Pressable`. Toda a lógica em `src/lib`, `src/types` e `src/providers` pode ser reaproveitada.

### Publicidade futura (Google AdMob)

`src/components/ad-slot.tsx` reserva os pontos de anúncio (rodapé da tela inicial e da tela do contador) sem renderizar nada e sem nenhuma biblioteca de anúncios. Na versão Expo, basta substituir esse componente por um banner do AdMob.

## Regras do produto

Esta versão é intencionalmente pequena, rápida e confiável. Não há login, cadastro, backend, banco de dados externo, API, pagamentos, assinatura, chat, comunidade, notificações, localização, câmera, recursos de IA ou recursos sociais.
