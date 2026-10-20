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
  alguma tentativa anterior. **Só no modo difícil** — fora dele a linha começa
  livre e as verdes já descobertas aparecem só como lembrete apagado (ghost,
  calculado em `renderCurrent()`); travar sempre deixava o modo normal igual ao
  difícil. Desligar o modo difícil no meio da partida limpa `state.locked`. Casinhas travadas nunca devem ser editáveis nem
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
- **Qualquer palpite vale**: `submit()` não checa se o palpite existe no
  dicionário — qualquer combinação de letras com o tamanho certo é aceita (a
  antiga opção "Aceitar qualquer palavra" / `state.free` foi removida).
- **Modo difícil** (`state.hard`): reaproveita `hardModeError()` para exigir
  que pistas já reveladas (verde/azul) apareçam nos palpites seguintes.
- **Sem framework de build**: `index.html` carrega `style.css` e `script.js`
  com um parâmetro `?v=N` para cache-busting. **Sempre que `script.js` ou
  `style.css` mudar, incrementar esse `N` em `index.html`** — sem isso,
  navegadores (principalmente no celular) continuam servindo a versão antiga
  em cache.
- **`state.mode`** (`'free'` infinito ou `'daily'` diário) e **`newGame(forcedKey)`**:
  `newGame()` sorteia normalmente pelo tema/tamanho (`themePool()`), mas aceita
  uma chave normalizada `forcedKey` pra pular o sorteio — usado pela palavra do
  dia (`dailyAnswerKey()`, PRNG determinístico com seed = data local, sempre
  5 letras/6 tentativas) e por um desafio recebido via link (`decodeChallenge()`
  no hash `#w=`). Todo `onclick` que chama `newGame` direto precisa ser
  `()=>newGame()`, nunca `newGame` cru — o handler de clique passaria o
  `MouseEvent` como `forcedKey` e corromperia a resposta.
- **Modo diário**: só uma partida por dia; o resultado (`rows`, `status`,
  `answerRaw`) fica em `store['dito:daily']` chaveado pela data local
  (`todayStr()`). Reabrir o jogo no mesmo dia cai em `showDailyDone()`, que
  reconstrói o tabuleiro a partir do resultado salvo sem permitir novo palpite
  (sem botão "Jogar de novo"). `setMode()` troca entre diário/infinito e guarda
  o tamanho/tentativas/tema anteriores em `state.savedLen/savedTries/savedTheme`
  pra restaurar ao voltar pro infinito.
- **Tema (`GROUPS`) também filtra o sorteio**, não só a dica: `themePool(len,
  theme)` filtra `DICT[len].keys` por `wordGroup`. `buildSegments()` desativa
  os tamanhos sem palavra suficiente no tema escolhido; ao trocar de tema
  (`selTheme`), se o tamanho atual ficar sem palavra, pula pro tamanho válido
  mais próximo — nunca deixar o jogador escolher uma combinação vazia.
- **Duelo por link**: `challenge()` codifica `len.tries.answer.hints.hard`
  em base64 no hash da URL (`#w=...`); `decodeChallenge()` valida e devolve
  `{len,tries,key,hints,hard}` (com padrões pra links antigos de 3 campos;
  o 6º campo de links antigos, "aceitar qualquer palavra", é ignorado)
  ou `null`. Só é honra entre amigos — o hash é trivial de decodificar, não é
  pensado como anti-cheat. O botão "🔗 Desafiar" (`btnChallenge`) fica nas
  hint-actions, disponível a qualquer momento da partida (não só no fim) —
  de propósito: assim quem cria o link também não viu a resposta ainda, e o
  duelo é justo pros dois lados. Desativado (`updatePlayActionBtns()`) no
  modo diário e durante um duelo já em andamento, junto com "🔄 Novo jogo".
- **`state.duel`**: ao entrar num desafio recebido por link, `hints`/`hard`
  são travados no valor de quem criou o link (senão os dois lados não
  jogam a mesma prova), guardando as preferências próprias do jogador em
  `state.savedHints/savedHard` pra restaurar depois. `buildSegments()`
  desabilita/esmaece os checkboxes (`.check.locked`) e os campos de
  tamanho/tentativas/tema enquanto `state.duel` for `true`. `endDuel()`
  restaura as preferências salvas e destrava tudo; é chamado automaticamente
  ao terminar a partida do duelo (`finish()`), ao trocar de modo (`setMode()`)
  e ao abandonar a partida pelo botão "novo jogo" (`startFresh()`).
- **Botão "🔄 Novo jogo"** (`startFresh()`): abandona a partida atual e sorteia
  outra a qualquer momento, reaproveitando a invalidação de `setTimeout`s via
  `state.gameId` (não precisa esperar o fim da rodada). Desabilitado no modo
  diário (`updateNewGameBtn()`, chamado de dentro de `buildSegments()`) porque
  só há uma tentativa por dia; se a partida abandonada era um duelo, chama
  `endDuel()` antes de sortear a próxima.

- **Modo relâmpago** (`state.mode==='blitz'`, aba "⚡ Relâmpago"): contra o
  relógio, sempre 5 letras/6 tentativas, travado como o diário em
  `buildSegments()` (`lockPlay = state.mode!=='free' || state.duel`). Estado em
  `state.blitz` (`score`, `deadline`, `timer`, `over`); o relógio só começa na
  1ª letra digitada (`blitzStartClock()` em `press()`). `finish()` desvia pra
  `blitzRoundEnd()` (não conta no placar normal), que põe o status de volta em
  `'ending'` e sorteia a próxima palavra num `setTimeout` com checagem de
  `gameId`. `endBlitz()` incrementa `gameId` pra cancelar revelações pendentes.
  "🔄 Novo jogo" vira "⏭️ Pular" (custa `BLITZ_SKIP` segundos). Ao sair do modo,
  `setMode()` para o `setInterval` e zera `state.blitz`. O modo não é restaurado
  ao recarregar (`saveSettings()` grava `'free'`).
- **Pontos e recordes do relâmpago**: cada acerto vale `blitzPoints()` (100 +
  20 por tentativa que sobrou + 25 por acerto seguido; `state.blitz.combo` zera
  ao errar ou pular). `endBlitz()` grava a rodada (se fez ponto) em
  `blitzRecords` — top `BLITZ_KEEP` por pontos, salvo em `store['dito:blitz']`
  como `{records:[{id,pts,words,date}]}` e carregado no `init`.
  `blitzRecordsHTML()` monta a tabela usada no fim da rodada (destacando a
  rodada atual) e no Placar. `stats.blitzBest` é um campo antigo, não usado.
- **`setMode()`** guarda tamanho/tentativas/tema em `state.savedLen/...` sempre
  que **sai** do infinito e restaura ao voltar; `saveSettings()` grava esses
  valores salvos (não os 5/6 fixos) quando o modo atual não é o infinito.
- **Análise do fim de jogo**: `possibilitiesTrail(rows, pool)` conta quantas
  palavras de `answerPool()` ainda batiam com todas as pistas depois de cada
  palpite (`state.trail`), mostrado no fim (`renderEndbar({trail})`) e no texto
  de "Copiar resultado". Só no fim, de propósito — durante o jogo viraria muleta.
- **Conquistas**: `ACHIEVEMENTS` (id/ícone/nome/descrição) + `ach` salvo em
  `store['dito:ach']`; `checkAchievements()` roda em `finish()` **antes** de
  `endDuel()` (precisa ver dicas/modo difícil/duelo da partida). As do
  relâmpago são liberadas em `endBlitz()`. Listadas no Placar (`renderAch()`).
  `stats.groups` (grupos já acertados) vive em `dito:stats`.
- **Contagem regressiva** do diário (`startCountdown()`): um `setInterval`
  global (`countdownTimer`), parado em `newGame()` e a cada `renderEndbar()`;
  se o dia virar com a tela aberta, chama `enterDaily()`.
- No diário, Enter depois do fim **não** sorteia outra palavra (`press()` só
  reinicia no infinito/relâmpago).
- **Tooltips** (`.tip`, botão "i" com `data-tip`): explicam "Dicas no fim" e
  "Modo difícil" nos Ajustes (CSS puro, hover/foco; no celular o balão sai
  abaixo do painel, largura toda). Ficam dentro do `<label>` mas, por serem
  `<button>`, clicar neles não marca/desmarca a caixa. Abas de modo e botões
  de ação usam `title` simples.
- **Visual doido 3D**: as regras de `.tile.correct/.present/.absent` no tema
  precisam vir **depois** de `.tile.filled` (mesma especificidade), senão a
  borda rosa da letra digitada fica por cima da cor do resultado.
- **Confete** (`confetti()`): canvas temporário na vitória e no recorde do
  relâmpago; ignorado com `prefers-reduced-motion`.

- **Layout no celular**: `#keyboard` é `position:sticky; bottom:0` (por isso
  `html,body` usam `min-height`, não `height`), então fica sempre visível
  mesmo quando o conteúdo passa da altura da tela. Como ele pode cobrir a
  linha atual ou a tela de fim, `submit()` e `renderEndbar()` chamam
  `scrollAboveKeyboard()`. Em telas ≤600px o painel de Ajustes começa fechado.

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
- No desktop, clicar em qualquer botão (`.btn`/`.icon-btn`, ex.: "🔄 Novo
  jogo") deixava o foco nele; como Enter é a tecla nativa de ativação de
  `<button>`, o próximo Enter — pensado pra enviar o palpite — reativava o
  próprio botão em vez disso (ex.: clicar em "Novo jogo" e depois teclar Enter
  reiniciava o jogo de novo, sem digitar nada). Corrigido com um listener
  global de `click` em `document` que dá `blur()` em qualquer `.btn`/
  `.icon-btn` logo após o clique.
