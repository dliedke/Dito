# Dito

Um jogo de adivinhar palavras em português, no estilo Wordle/Termo — 100% front-end, sem build, sem dependências de servidor.

🔗 **Repositório:** https://github.com/dliedke/Dito
🔗 **Jogar (após ativar o GitHub Pages):** https://dliedke.github.io/Dito/

## Como jogar

Descubra a palavra secreta antes que acabem as tentativas. Cada palpite precisa ter o número de letras escolhido.

- 🟩 **Verde** — a letra está na palavra e na posição certa.
- 🟨 **Amarelo** — a letra está na palavra, mas em outra posição.
- ⬛ **Cinza** — a letra não aparece na palavra.

Acentos são preenchidos automaticamente: digitar `avo` conta como `avô`. O cedilha entra como `C`.

## Funcionalidades

- **Tamanho da palavra configurável** — de 3 a 8 letras.
- **Número de tentativas configurável** — de 3 a 8.
- **Dicionário próprio em português**, com listas de palavras separadas por tamanho (`script.js`), sem depender de API externa.
- **Modo difícil** — todas as dicas já reveladas precisam ser reaproveitadas no palpite seguinte.
- **Aceitar qualquer palavra** — desliga a validação contra o dicionário, permitindo digitar qualquer combinação de letras.
- **Dicas ao final** — revela progressivamente letras da resposta quando as tentativas acabam (se habilitado).
- **Tema claro/escuro** com alternância pelo botão ◐.
- **Placar da sessão** — jogos, vitórias, aproveitamento e sequência.
- **Compartilhar resultado** — copia um resumo do jogo para a área de transferência.
- **Teclado virtual** com teclado físico funcionando em paralelo, e totalmente acessível (`aria-live`, `role="grid"`, etc.).
- **Responsivo**, pensado para mobile e desktop.

## Estrutura do projeto

```
index.html  → estrutura da página (HTML)
style.css   → visual, temas e responsividade
script.js   → lógica do jogo, dicionário e estado
```

Sem dependências externas além das fontes do Google Fonts (`Bricolage Grotesque` e `Space Grotesk`).

## Rodando localmente

Como é um site 100% estático, basta abrir o `index.html` diretamente no navegador, ou servir a pasta com qualquer servidor simples:

```bash
npx serve .
# ou
python -m http.server
```

Depois acesse `http://localhost:<porta>/`.

## Publicando no GitHub Pages

1. Suba o projeto para o repositório [github.com/dliedke/Dito](https://github.com/dliedke/Dito) (branch `main`).
2. No GitHub, vá em **Settings → Pages**.
3. Em **Source**, selecione a branch `main` e a pasta `/ (root)`.
4. Salve e aguarde o deploy (leva cerca de 1 minuto).
5. O jogo ficará disponível em `https://dliedke.github.io/Dito/`.

Como o arquivo principal se chama `index.html`, o GitHub Pages já serve o jogo direto na raiz do site, sem precisar de nenhum caminho adicional.

## Persistência de dados

Configurações (tema, tamanho da palavra, tentativas, modos) e o placar da sessão são salvos no `localStorage` do navegador, então ficam guardados entre visitas no mesmo dispositivo/navegador. Se o `localStorage` estiver indisponível (ex.: modo privado com restrições), o jogo detecta isso e continua funcionando normalmente, só sem persistir os dados.

## Licença

Defina aqui a licença do projeto (por exemplo, MIT), se desejar deixá-lo aberto.
