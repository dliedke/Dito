# Dito

Jogo de adivinhar palavras em português, estilo Wordle/Termo. O jogo em si é
**100% front-end** (HTML/CSS/JS puro, sem build, sem framework, sem
dependência de servidor). Publicado em https://dliedke.github.io/Dito/.

## Estrutura

```
Dito.slnx                     → solution do Visual Studio
Dito.csproj                   → projeto ASP.NET Core, só hospeda os arquivos estáticos
Program.cs                    → host mínimo (UseDefaultFiles + UseStaticFiles), sem rotas/API
Properties/launchSettings.json→ perfis de execução (F5 abre o navegador em localhost:5080)
wwwroot/
  index.html                  → estrutura da página
  style.css                   → visual, temas claro/escuro, responsividade
  script.js                   → TODO o jogo: dicionário, estado, regras, UI
.github/workflows/pages.yml   → publica wwwroot/ no GitHub Pages a cada push na main
```

O projeto .NET (`Dito.csproj`/`Program.cs`) existe **só** para dar um jeito
prático de abrir o repositório como solution no Visual Studio e rodar com F5.
Ele não tem nenhuma lógica de jogo, API ou estado no servidor — é puramente um
servidor de arquivos estáticos para `wwwroot/`. Toda a lógica do jogo vive em
`wwwroot/script.js`, rodando inteiramente no navegador.

## Rodando localmente

- Visual Studio: abrir `Dito.slnx`, F5.
- CLI: `dotnet run` (na raiz do repo).
- Sem .NET: abrir `wwwroot/index.html` direto no navegador, ou
  `npx serve wwwroot` / `python -m http.server --directory wwwroot`.

Não há testes automatizados nem processo de build — para validar uma mudança,
rode o jogo (F5 ou `dotnet run`) e jogue algumas partidas testando o cenário
alterado.

## Arquitetura de `script.js`

Um único arquivo, sem módulos, dividido em seções por comentário (dicionário,
estado, som, config, novo jogo, teclado, dicas, entrada, avaliação, fim de
jogo, painéis/tema, início). Pontos importantes:

- **`RAW` / `DICT`**: dicionário de palavras em português por tamanho (3 a 8
  letras). `DICT[n].keys` são as versões **normalizadas** (sem acento/til,
  minúsculas, via `norm()`) usadas para todo o jogo — comparação de palpite,
  chave de matching etc. `DICT[n].words` mapeia normalizada → original (com
  acento). A resposta da rodada fica em `state.answer` (normalizada) e
  `state.answerRaw` (com acento, só para exibição).
- **Acentos**: o jogador digita sem acento (`avo` conta como `avô`); toda a
  lógica de avaliação/comparação usa sempre a forma normalizada. Qualquer
  exibição de letra que deveria estar "confirmada certa" (verde-limão, ou o
  resultado final) precisa usar `state.answerRaw`, não o palpite digitado —
  senão o acento some da tela mesmo a letra estando correta.
- **`state.locked`**: conjunto de posições travadas (verde-limão) que foram
  carregadas automaticamente para a próxima linha porque já foram acertadas em
  alguma tentativa anterior. Casinhas travadas nunca devem ser editáveis nem
  apagáveis — qualquer mudança em `press()`/`renderCurrent()` que mexa em
  cursor ou backspace precisa preservar essa invariante, incluindo o caso de
  borda em que a **última** casinha da linha é a travada.
- **`GROUPS_RAW` / `GROUPS` / `wordGroup` / `GROUP_LABEL`**: grupos temáticos
  usados pelo botão "💡 Dica de grupo" (`useGroupHint()`, uso único por
  partida). `GROUPS_RAW` é categoria → string de palavras **normalizadas**,
  no mesmo estilo de `RAW`; `GROUPS` e o mapa reverso `wordGroup` são
  derivados dele, e `GROUP_LABEL` guarda o nome exibido (com acento) de cada
  categoria. Invariantes a manter ao mexer no dicionário ou nos grupos:
  **toda** palavra de `DICT` precisa ter grupo, nenhuma palavra pode estar em
  dois grupos, nenhum grupo pode listar palavra que não existe em `DICT`,
  todo grupo precisa de pelo menos 2 palavras (senão não há "outra palavra"
  para sugerir) e toda categoria precisa de um `GROUP_LABEL`.
- **`state.gameId`**: contador incrementado a cada `newGame()`. Todo
  `setTimeout` de `submit()` (revelação das casinhas, teclado, dica
  automática, tela de fim) precisa capturar o `gameId` atual e desistir se
  ele tiver mudado — sem isso, um callback de uma partida antiga roda em cima
  do estado da partida nova e corrompe a tela de fim.
- **`state.status === 'ending'`**: estado transitório entre detectar
  acerto/erro final e a animação de revelação terminar. `press()` ignora tudo
  nesse intervalo; só depois o status vira `win`/`lose` e o Enter volta a
  reiniciar o jogo.
- **Modo difícil** (`state.hard`): reaproveita `hardModeError()` para exigir
  que pistas já reveladas (verde/azul) apareçam nos palpites seguintes.
- **Sem framework de build**: `index.html` carrega `style.css` e `script.js`
  com um parâmetro `?v=N` para cache-busting. **Sempre que `script.js` ou
  `style.css` mudar, incrementar esse `N` em `index.html`** — sem isso,
  navegadores (principalmente no celular) continuam servindo a versão antiga
  em cache.

## Convenções

- Comentários e mensagens de commit em **português**.
- Sem ponto e vírgula obrigatório em todo lugar, mas o arquivo já usa `;` de
  forma consistente — seguir o estilo existente em vez de introduzir um novo.
- Funções pequenas e diretas, nomeadas por responsabilidade (`newGame`,
  `drawBoard`, `evaluate`, `submit`, `finish`...); evitar abstrações novas
  para um projeto deste tamanho.
- Mensagens de commit curtas, descrevendo o efeito visível da mudança (ex.:
  "Corrige duplicação da linha verde após já ter acertado a palavra").

## Erros já corrigidos (não reintroduzir)

- Letra correta perdendo o acento/til quando reaparecia como travada
  (verde-limão) na linha atual — corrigido usando `state.answerRaw[i]` na
  exibição em vez do palpite digitado.
- Backspace apagando uma casinha travada (verde-limão) quando o cursor pousava
  nela — causado por um off-by-one ao pular casinhas travadas para frente
  (`i < state.len-1` deixava a cursor parar bem na última casinha, mesmo
  travada). O guard de `press('⌫')` também precisa checar `state.locked`
  diretamente, não só se a casinha está vazia.
- Enter no desktop reiniciando o jogo sozinho / acusando "Faltam letras" logo
  após um palpite válido — eram duas causas: o auto-repeat do teclado (segurar
  Enter) disparando vários `submit()`/`newGame()` seguidos, e `setTimeout`s da
  partida anterior rodando `finish()` em cima do estado da partida nova
  (resultado: tela de fim com "undefined" e "Acertou em 0 de N"). Corrigido com
  `if(e.repeat) return` no handler de `keydown`, o estado `'ending'` e o
  `state.gameId`. Qualquer trabalho adiado novo em `submit()` precisa checar o
  `gameId`.
