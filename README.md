# Hunt Vendors

Página estática para mapear fabricantes que podem virar parceiros. Ela reúne perfil, números financeiros, contatos, produtos, vídeos e avaliação Gartner de cada empresa, organizados em três categorias:

1. **Automação, Intralogística e Robótica**
2. **Barcode e Automação Comercial**
3. **Storage, Data Centers e Infraestrutura Tecnológica**

Não há build, framework nem dependências: são só um HTML e um arquivo de dados.

## Telas

| Tela | Endereço | O que mostra |
|---|---|---|
| Categorias | `#home` | Cards das três categorias e busca rápida por fabricante no topo |
| Categoria | `#cat1`, `#cat2`, `#cat3` | Lista lateral de fabricantes e o perfil completo do fabricante selecionado |
| Comparativo Geral | `#compare` | Filtros, ranking (receita, market cap, headcount), mapa de países e tabela comparativa ordenável |

## Arquivos

```
hunt-vendors.html   página: HTML, CSS e JavaScript
data.js             dados: fabricantes, logos e fotos (em base64)
```

O `data.js` define a constante global `D` e **precisa ser carregado antes** do script principal. O `hunt-vendors.html` já faz isso:

```html
<script src="data.js"></script>
<script>
  // lógica da página, que lê D
</script>
```

Os dois arquivos precisam ficar na mesma pasta.

## Rodando localmente

Sirva a pasta por HTTP em vez de abrir o arquivo com duplo clique:

```bash
python3 -m http.server 8000
```

E acesse <http://localhost:8000/hunt-vendors.html>.

> **Por que não abrir direto?** Com o arquivo aberto como `file://`, o navegador não envia o cabeçalho `Referer`, e o YouTube recusa o vídeo incorporado com **"Erro 153 – Erro de configuração do player de vídeo"**. Por HTTP, local ou no GitHub Pages, os vídeos funcionam.

## Publicando no GitHub Pages

1. Envie `hunt-vendors.html` e `data.js` para o repositório.
2. Em **Settings → Pages**, escolha **Deploy from a branch**, selecione a branch (ex.: `main`) e a pasta `/ (root)`.
3. A página fica disponível em `https://<usuario>.github.io/<repositorio>/hunt-vendors.html`.

Para a página abrir direto na raiz do site (`https://<usuario>.github.io/<repositorio>/`), renomeie `hunt-vendors.html` para `index.html`.

## Editando os dados

Todo o conteúdo fica em `data.js`, no formato:

```js
const D = {
  "vendors": [ /* um objeto por fabricante */ ],
  "bg": ""     // imagem de fundo da página (data URI ou URL); vazio = sem imagem
};
```

### Campos de um fabricante

Campos sem informação devem ficar como `null`. Listas vazias devem ser `[]`. Nesses casos a página mostra "Informação não disponível".

| Campo | Tipo | Descrição |
|---|---|---|
| `id` | número | Identificador único e sequencial. Também é usado na navegação |
| `name` | texto | Nome do fabricante |
| `cat` | `1` \| `2` \| `3` | Categoria (veja a lista no topo) |
| `country` | texto | País de origem. Para mais de um país, separe com `/` (ex.: `"China / Estados Unidos"`) |
| `sector` | texto | Segmento (ex.: `"AMR"`, `"Kiosks"`) |
| `summary` | texto | Resumo executivo |
| `mcap` / `mcapTxt` | número / texto | Market cap em US$ (usado nos cálculos) e texto exibido |
| `rev` / `revTxt` | número / texto | Receita anual em US$ e texto exibido |
| `emp` / `empTxt` | número / texto | Número de funcionários e texto exibido |
| `listed` | `true` \| `false` | Se tem capital aberto |
| `exchange` | texto | Bolsa e ticker, quando listada |
| `ipo` | texto | Informação sobre IPO, quando não listada |
| `contacts` | lista | Contatos: `[{ "n": "Nome", "c": "Cargo" }]` |
| `phone`, `linkedin`, `site` | texto | Telefone, LinkedIn e site |
| `eligible` | texto | Se é elegível sem presença no Brasil |
| `obs` | texto | Observações estratégicas |
| `extras` | lista de textos | Pontos adicionais exibidos abaixo das observações |
| `potential` | texto | Diferenciais competitivos e certificações |
| `case` | texto | Nome do case do vídeo. Use `"Institucional"` para vídeo institucional |
| `video` | texto | Link do vídeo (veja **Vídeos** abaixo) |
| `gscore` | número de 0 a 5 | Score Gartner |
| `gpot` | `"ALTO"` \| `"MODERADO"` | Potencial de crescimento Gartner |
| `gsrc` | texto | Link da fonte Gartner |
| `imp` | objeto | Diagnóstico de entrada no Brasil: `{ "clients", "br", "cls", "next" }` (clientes de referência, diagnóstico, classificação, próximo passo) |
| `dq` | lista de textos | Notas sobre a qualidade dos dados, exibidas em destaque no fim do perfil |
| `hunter` | texto | Agente responsável pelo mapeamento |
| `logo` | texto | Logo como data URI (`data:image/webp;base64,...`) ou URL |
| `logoDark` | `true` \| `false` | Ajusta a exibição de logos escuros |
| `logoBg` | texto | Cor de fundo atrás do logo (ex.: `"#fff"`), ou `null` |
| `prods` | lista | Imagens de produtos (veja abaixo) |

### Produtos (`prods`)

```js
{
  "src": "data:image/webp;base64,...",  // imagem (data URI ou URL)
  "kind": "photo",        // "photo" (foto), "studio" (fundo neutro) ou "cut" (recorte)
  "w": 850, "h": 1100,    // dimensões originais da imagem, em pixels
  "name": "Nome do produto",
  "cat": "Tipo do produto",
  "desc": "Descrição curta",
  "mode": "fit",          // fotos: "fit" (inteira) ou "cover" (preenche o quadro)
  "pos": "50% 50%"        // ponto de foco da imagem (CSS object-position)
}
```

A primeira foto do tipo `"photo"` também serve de capa do vídeo. A imagem de fundo de cada card de categoria vem de um produto do fabricante-destaque daquela categoria (`CATS[n].k` e `CATS[n].i` em `hunt-vendors.html`). Por isso esse fabricante precisa ter `prods`.

### Vídeos

O player incorporado funciona com links nos formatos:

- `https://www.youtube.com/watch?v=ID`
- `https://vimeo.com/ID`

Qualquer outro link aparece como um botão "Assistir no site do fabricante". Alguns vídeos têm a incorporação desativada pelo dono e não tocam dentro da página. Para esses casos sempre aparece o link "assistir em nova aba".

## Organização do código

O JavaScript em `hunt-vendors.html` está dividido em seções comentadas:

- **Constantes:** categorias (`CATS`), países com bandeira e coordenadas (`FL`), pontos do mapa-múndi (`LAND`), ícones (`IC`)
- **Helpers:** escape de HTML, logos, formatação de valores e faixas de faturamento e market cap
- **Estado:** objeto `S` com a tela atual, a categoria, o fabricante selecionado, a busca, os filtros e a ordenação
- **Componentes:** funções que devolvem HTML como string (`home()`, `vendor()`, `compare()`, ...)
- **Renderização e navegação:** `render()`, `go()` e um roteador de cliques baseado em `data-a="ação"` / `data-v="valor"`
- **Interações:** pré-visualização de produto no hover, lightbox de imagens e brilho do cursor
- **Eventos e inicialização:** busca, teclado, filtros e abertura da tela indicada no `#hash` da URL

No topo do script há uma legenda dos nomes curtos usados no código (`D`, `V`, `S`, `F`, `LB`...).

Para adicionar um país novo, inclua-o em `FL` com bandeira, longitude e latitude, para que ele apareça no mapa.

## Formatação

O CSS, o JavaScript e os dados foram formatados com Prettier, usando estas opções:

```bash
npx prettier@3.6.2 --print-width 100 --single-quote --quote-props preserve --write data.js
```

O CSS e o JS do `hunt-vendors.html` foram formatados separadamente, com as mesmas opções. Rodar o Prettier no `.html` inteiro também reformata a marcação HTML. Isso não foi testado, então confira a página depois se fizer isso.

> Evite o Prettier **3.3.3**: nessa versão, a formatação deste código troca `[1,2,3]` por `[3]` dentro de template strings, e a página passa a mostrar só uma categoria.
