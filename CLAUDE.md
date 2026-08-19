# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## O que é este projeto

Vitrine digital de uma revendedora de beleza (pt-BR, WhatsApp como checkout — sem pagamento; a sacola existe mas só MONTA a mensagem do pedido, não é e-commerce) que é TAMBÉM uma aula de HTML/CSS/JS para uma não-programadora. **O arquivo único, sem build e cheio de comentários didáticos É o entregável**: introduzir framework, build, minificação, arquivos separados ou código "enxuto demais" destrói o propósito do projeto, mesmo que o resultado pareça melhor.

- Site no ar: https://luigidevcode.github.io/vitrine/ (GitHub Pages, branch `main`, raiz — push em `main` = deploy, com cache de ~10 min).
- Tudo mora em `index.html` (~3000 linhas), organizado em "PARTES" numeradas por cabeçalhos-comentário (PARTE 1 = cores … PARTE 13 = o JavaScript, "o cérebro do catálogo").
- Trabalhe em português: comentários no código, mensagens de commit (frases curtas e humanas dizendo o RESULTADO — veja o `git log`) e conversa com o usuário.

## Regras da casa (invioláveis)

1. **Arquivo único, zero dependências** além do link do Google Fonts. Código novo ganha comentários em prosa calorosa em pt-BR explicando o PORQUÊ (imite o estilo que já está no arquivo). Pontos seguros de edição levam o marcador `TROQUE AQUI`.
2. **O site abre como `file://`** além do GitHub Pages. `history.pushState/replaceState` ESTOURAM em `file://` — sempre dentro de `try`, e sempre DEPOIS da ação principal (a janela abre primeiro, o endereço muda depois). Qualquer dependência de rede nova segue o mesmo espírito: falhou → o site fica exatamente como era ("Plano A / Plano B"). Módulo ES precisa ficar INLINE no HTML (`file://` bloqueia módulo em arquivo separado).
3. **Sem JavaScript o site inteiro continua funcionando**: o catálogo é lido do próprio HTML (os dados ficam nos `data-*` dos cards, sem array duplicado), há filtro reserva em CSS puro (`:target`) e o conteúdo da janela de produto aparece dentro do card. O gatilho do progressive enhancement é a classe `com-js` no `<html>`; controles que só fazem sentido com JS usam `.so-com-js`.
4. **Mudanças valem no computador E no celular**, salvo pedido explícito em contrário. Decisão de produto vai no CSS base; media query é só para o que muda de verdade com o tamanho da tela (colunas, o que gruda no topo).
5. **Disciplina de contraste é levada a sério** — os números medidos vivem nos comentários do próprio arquivo. `--rosa` NUNCA é cor de texto (2.57:1). Títulos usam `--cacau` (o que se LÊ) e ações usam `--marrom-marca` (o que se CLICA) — são dois marrons com trabalhos diferentes, não misturar. As opacidades do rodapé e dos cards são contas refeitas contra o fundo real — não baixar sem recalcular. Objeto/animação claro atrás de texto ainda reprova pela SOMBRA dele; se precisar cruzar texto, use material sem luz.
6. **Links compartilháveis vêm de `STORE_CONFIG.site`, nunca de `window.location`** (senão o arquivo aberto localmente geraria links `file:///` inúteis). Se o endereço do site mudar, são três lugares: `STORE_CONFIG.site`, `og:url` e `og:image`.
7. **Conteúdo pendente é da dona do site — não inventar**: preços (`data-preco=""` → o card mostra "Preço sob consulta"), recomendações em primeira pessoa (`.recomendacao`, o diferencial declarado do site), queridinhos (`data-destaque`), Instagram, nome definitivo da marca e a foto dela na seção Sobre.

## Arquitetura que exige ler mais de um lugar

- **Barras grudadas**: as alturas são variáveis CSS (`--barra-topo`, `--barra-fita`) porque o `scroll-margin-top` das âncoras depende delas. No celular o cabeçalho NÃO gruda (`--barra-topo: 0`) e só a fita de categorias fica; no desktop os dois grudam. O `sticky` da fita só se solta na hora certa porque o pai dela é a `div.area-catalogo`.
- **Fotos**: `fotos/<id>.jpg` (+ `<id>-2.jpg`, `-3`… para a galeria). O JS tenta carregar em segredo (`procurarFoto`) — sem foto não aparece ícone quebrado — e a galeria para no primeiro número que faltar, então a numeração não pode ter buraco. Só as fotos principais carregam com a vitrine; a galeria é buscada no clique.
- **"Página" de produto**: um único `<dialog>` reaproveitado, endereçável por `?produto=<id>` (o mesmo link do botão "Copiar link"); Voltar/Esc/X fecham sem sujar o histórico.
- **Cartão OG do WhatsApp**: `preview.jpg` é gerado por `preview-fonte.html` (o comando Chrome headless + `sips` está no rodapé desse arquivo). Ele tem paleta própria hardcoded — não herda a do site; mudou a cara do hero, regenere.
- **Sacola** (PARTE 11b no CSS, seção "A SACOLA" no JS): guarda só `{id, tamanho}` em localStorage (`vitrine-sacola`); nome, preço e foto são resolvidos na hora, lendo os cards. Todos os botões dela nascem no JS (sem JS, nada sobra na tela). São DOIS `<dialog>` compartilhando o CSS `.janela` e DOIS ouvintes de `popstate` — cada um cuida do próprio degrau ({janela: id} vs {sacola: true}), nunca dos dois. No card, produto com vários tamanhos NÃO adiciona direto: abre a página do produto pra cliente escolher o pote (nunca chutar tamanho). O toast não aparece por cima de `<dialog>` aberto (top layer) — dentro da janela, quem confirma é o próprio botão virando "Está na sacola ✓".

## Comandos e verificação

Não existe lint, teste nem build — a "suíte" é manual e cobre os modos de falha reais:

- Servidor local: `python3 -m http.server 8642` na raiz. Para testar no celular físico (obrigatório antes de publicar mudança visual): `--bind 0.0.0.0` + o IP de `ipconfig getifaddr en0`.
- Testar SEMPRE nos dois modos: via `http://` E abrindo `index.html` direto (`file://`), com e sem internet.
- Playwright: não há instalação global nem browsers do ms-playwright baixados; instale `playwright` via npm numa pasta temporária e use `chromium.launch({ channel: 'chrome' })` (o Chrome real está instalado). Emular viewport de celular, `prefers-reduced-motion` e bloquear domínios de CDN cobre os caminhos de degradação.
- Checklist mínimo por mudança: os dois modos acima + contraste dos textos afetados + zero erros de console + os dois formatos (celular e desktop).

## Referência visual

`~/Documents/lume-beleza (1).html` é a referência de design original — **nunca alterar esse arquivo**.
