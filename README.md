# Calculadora de Escoramento — NBR 15696

App web estático. Projeto e verificação de fôrmas e escoramentos conforme ABNT NBR 15696,
com desenho da grade sobre a planta de formas importada e vista 3D da montagem.

## Usar

Qualquer servidor estático serve. Localmente:

    python -m http.server 8777

e abrir http://localhost:8777

Abrir o `index.html` direto do disco **não** carrega as pranchas de exemplo: o navegador
bloqueia a leitura da pasta `pranchas/`. O resto do app funciona, e a importação de DXF também.

## Importar uma prancha

Exportar a planta de formas em DXF (comando `DXFOUT` no AutoCAD) e soltar no campo de importação.
O app lê as marcas (`L3 h=18`, `V4 14x88`, `P3 22x175`), detecta a escala do desenho comparando a
seção dos pilares com o contorno deles, delimita o pavimento e lança a grade de escoramento.

As camadas são configuráveis: o padrão é o do AltoQi Eberick (`FO-Vigas`, `FO-Pilares`,
`FO-Aberturas`), e ao importar o app lista o que achou e deixa remapear.

## Limites

Dimensiona a cadeia fôrma → travessa → longarina → apoio vertical → base e posiciona a grade.
Não substitui o projeto executivo de escoramento. Contraventamento, análise global de estabilidade,
peças de madeira complementares, base de apoio e a ART de projeto e montagem são do responsável
técnico pela execução (NBR 15696 6.3 d e B.2.2). O prazo de retirada do escoramento e do
reescoramento é do projetista da estrutura de concreto. As cargas de utilização de catálogo devem
ser confirmadas com o fornecedor dos equipamentos. Os preços do orçamento são valores de partida,
não cotação.
