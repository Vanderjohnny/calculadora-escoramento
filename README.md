# Calculadora de Escoramento — NBR 15696

App web estático. Projeto e verificação de fôrmas e escoramentos conforme ABNT NBR 15696:
lê a planta de formas em DXF, lança a grade de escoramento sobre o desenho, mostra a montagem
em 3D e exporta o jogo de pranchas cotadas em PDF e a grade de volta em DXF.

## Usar

Qualquer servidor estático serve. Localmente:

    python -m http.server 8777

e abrir http://localhost:8777

Abrir o `index.html` direto do disco **não** carrega as pranchas de exemplo: o navegador bloqueia
a leitura da pasta `pranchas/`. O resto do app funciona, e a importação de DXF também.

## Importar uma prancha

Exportar a planta de formas em DXF (comando `DXFOUT` no AutoCAD) e soltar no campo de importação.
O app lê as marcas (`L3 h=18`, `V4 14x88`, `P3 22x175`), detecta a escala do desenho comparando a
seção dos pilares com o contorno deles, delimita o pavimento e lança a grade.

As camadas são configuráveis: o padrão é o do AltoQi Eberick (`FO-Vigas`, `FO-Pilares`,
`FO-Aberturas`), e ao importar o app lista o que achou e deixa remapear.

## Exportar

**PDF** — jogo de seis pranchas em A3 paisagem: uma geral sem cotas e cinco cotadas, uma por
elemento (placas, barrotes, longarinas, escoras de laje, escoras de viga), todas com planta e
marcas. Mais o memorial de cálculo.

**DXF** — a grade de volta para o CAD, em metros, no mesmo sistema de coordenadas da planta
importada, separada em camadas: `ESC-PLACAS`, `ESC-BARROTES`, `ESC-LONGARINAS`, `ESC-ESCORAS`,
`ESC-ESCORAS-VIGA`, `ESC-COTAS`, `ESC-MARCAS` e `PLANTA`.

Dentro de um artefato publicado no claude.ai as duas exportações ficam indisponíveis, porque o
frame bloqueia impressão e download. Pelo app servido funcionam normalmente.

## Limites

Dimensiona a cadeia fôrma → travessa → longarina → apoio vertical → base e posiciona a grade.
Não substitui o projeto executivo de escoramento. Contraventamento, análise global de estabilidade,
peças de madeira complementares, base de apoio e a ART de projeto e montagem são do responsável
técnico pela execução (NBR 15696 6.3 d e B.2.2). O prazo de retirada do escoramento e do
reescoramento é do projetista da estrutura de concreto. As cargas de utilização de catálogo devem
ser confirmadas com o fornecedor dos equipamentos. Os preços do orçamento são valores de partida,
não cotação.
