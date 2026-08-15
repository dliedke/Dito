# Dito

Um jogo de adivinhar palavras em português, no estilo Wordle/Termo — 100% front-end, sem build, sem dependências de servidor.

🔗 **Jogar** https://dliedke.github.io/Dito/

## Como jogar 

Descubra a palavra secreta antes que acabem as tentativas. Cada palpite precisa ter o número de letras escolhido.

- 🟩 **Verde** — a letra está na palavra e na posição certa.
- 🟦 **Azul** — a letra está na palavra, mas em outra posição.
- ⬛ **Cinza** — a letra não aparece na palavra.

Acentos são preenchidos automaticamente: digitar `avo` conta como `avô`. O cedilha entra como `C`.

## Funcionalidades

- **Tamanho da palavra configurável** — de 3 a 8 letras.
- **Número de tentativas configurável** — de 3 a 8.
- **Dicionário próprio em português**, com listas de palavras separadas por tamanho (`wwwroot/script.js`), sem depender de API externa.
- **Modo difícil** — todas as dicas já reveladas precisam ser reaproveitadas no palpite seguinte.
- **Aceitar qualquer palavra** — desliga a validação contra o dicionário, permitindo digitar qualquer combinação de letras.
- **Dicas ao final** — revela progressivamente letras da resposta quando as tentativas acabam (se habilitado).
- **Tema claro/escuro** com alternância pelo botão ◐, refletido inclusive na cor da barra do navegador no mobile.
- **Placar** — jogos, vitórias, aproveitamento e sequência, salvo entre visitas.
- **Efeitos sonoros** sintetizados via Web Audio (tecla, cores do palpite, vitória/derrota, erro), com botão de mudo 🔊/🔇.
- **Compartilhar resultado** — copia um resumo do jogo para a área de transferência.
- **Teclado virtual** com teclado físico funcionando em paralelo, e totalmente acessível (`aria-live`, `role="grid"`, etc.).
- **Responsivo e otimizado para mobile** — respeita as áreas seguras (notch/home indicator) em iOS, alvos de toque maiores, sem "elástico" de scroll, ajustes para telas estreitas e para paisagem.

## Estrutura do projeto

```
Dito.slnx           → solution do Visual Studio
Dito.csproj          → projeto ASP.NET Core (só hospeda os arquivos estáticos)
Program.cs           → host mínimo, serve wwwroot/ com UseStaticFiles
Properties/
  launchSettings.json → perfis de execução (F5 abre o navegador)
wwwroot/
  index.html          → estrutura da página (HTML)
  style.css           → visual, temas e responsividade
  script.js           → lógica do jogo, dicionário e estado
```

O jogo em si (`wwwroot/`) é 100% front-end — HTML/CSS/JS puro, sem build e sem
chamadas a servidor. O projeto .NET existe só como um jeito prático de abrir o
repositório com solution/projeto no Visual Studio e rodar localmente com F5;
ele não tem nenhuma lógica de jogo, API nem estado no servidor.

Sem dependências externas além das fontes do Google Fonts (`Bricolage Grotesque` e `Space Grotesk`).

## Rodando localmente

**Com o Visual Studio:** abra `Dito.slnx` e aperte F5 (ou Ctrl+F5). O navegador abre
sozinho em `http://localhost:5080/`.

**Com a CLI do .NET** (SDK 8 ou mais novo):

```bash
dotnet run
```

**Sem o .NET**, como é um site 100% estático, também dá pra abrir `wwwroot/index.html`
direto no navegador, ou servir a pasta com qualquer servidor simples:

```bash
npx serve wwwroot
# ou
python -m http.server --directory wwwroot
```

Depois acesse `http://localhost:<porta>/`.

## Publicando no GitHub Pages

O deploy é automático via GitHub Actions (`.github/workflows/pages.yml`): a cada
push na branch `main`, o workflow publica o conteúdo de `wwwroot/` no GitHub Pages
(sem build, é só uma cópia dos arquivos estáticos).

1. Suba o projeto para o repositório [github.com/dliedke/Dito](https://github.com/dliedke/Dito) (branch `main`).
2. No GitHub, vá em **Settings → Pages**.
3. Em **Source**, selecione **GitHub Actions** (em vez de "Deploy from a branch").
4. Dê um push na `main` (ou rode o workflow manualmente em **Actions**) e aguarde o deploy (leva cerca de 1 minuto).
5. O jogo ficará disponível em `https://dliedke.github.io/Dito/`.

## Persistência de dados

Configurações (tema, tamanho da palavra, tentativas, modos) e o placar da sessão são salvos no `localStorage` do navegador, então ficam guardados entre visitas no mesmo dispositivo/navegador. Se o `localStorage` estiver indisponível (ex.: modo privado com restrições), o jogo detecta isso e continua funcionando normalmente, só sem persistir os dados.

## Licença

Defina aqui a licença do projeto (por exemplo, MIT), se desejar deixá-lo aberto.
