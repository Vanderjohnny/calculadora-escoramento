# Calculadora de Escoramento — NBR 15696

App web estático. Projeto e verificação de fôrmas e escoramentos conforme ABNT NBR 15696:
lê a planta de formas em DWG ou DXF, ou o modelo estrutural em IFC, lança a grade de escoramento
sobre o desenho, mostra a montagem em 3D e exporta o jogo de pranchas cotadas em PDF e a grade de
volta em DXF. Com o IFC o cálculo é do prédio inteiro, pavimento a pavimento.

## Usar

Qualquer servidor estático serve. Localmente:

    python -m http.server 8777

e abrir http://localhost:8777

Abrir o `index.html` direto do disco **não** carrega as pranchas de exemplo: o navegador bloqueia
a leitura da pasta `pranchas/`. O resto do app funciona, e a importação de DXF também.

## Importar uma prancha

Soltar a planta de formas no campo de importação, em **DWG** ou **DXF**. O app lê as marcas
(`L3 h=18`, `V4 14x88`, `P3 22x175`), detecta a escala do desenho comparando a seção dos pilares
com o contorno deles, delimita o pavimento e lança a grade.

O DWG é binário e fechado: o app converte com a [libredwg](https://www.gnu.org/software/libredwg/)
compilada para WebAssembly, buscada no CDN (jsdelivr) só quando chega um DWG — são 11 MB que não
fazem falta a quem importa DXF ou IFC. Convertido, o desenho segue pelo mesmo caminho do DXF. As
duas pranchas de exemplo dão resultado idêntico pelos dois caminhos. Sem internet, ou se a página
bloquear o download, o app avisa e pede o DXF (`DXFOUT` no AutoCAD, Exportar › CAD no Revit,
Arquivo › Exportar › DXF no Eberick).

As camadas são configuráveis: o padrão é o do AltoQi Eberick (`FO-Vigas`, `FO-Pilares`,
`FO-Aberturas`), e ao importar o app lista o que achou e deixa remapear.

## Importar o modelo em IFC

Soltar o arquivo `.ifc` no mesmo campo. O app lê a estrutura espacial (`IfcBuildingStorey`) e
separa o cálculo por pavimento, com a cota de cada um. Daí saem, medidos no próprio sólido:

- **laje** — contorno, área e espessura, com o furo de shaft e caixa de escada descontado;
  o tipo (maciça, nervurada, treliçada) vem da propriedade do modelo quando ela existe;
- **viga** — eixo, largura e altura do perfil da extrusão, sem pareamento de arestas;
- **pilar e parede** — seção e altura, para a fôrma vertical do Anexo D;
- **pé-direito** — diferença de cota entre o pavimento e o de baixo.

Testado contra 53 modelos de 12 obras, exportados pelo AltoQi Eberick e pelo Revit, de 20 kB a
388 MB (7,5 milhões de entidades). Esquema IFC4 e IFC2x3, em milímetro, centímetro, metro ou pé.
Geometria por extrusão (`IfcExtrudedAreaSolid`), por casca de faces (`IfcFacetedBrep`), por malha
(`IfcPolygonalFaceSet`, `IfcTriangulatedFaceSet`), por item mapeado (`IfcMappedItem`) e com recorte
booleano (`IfcBooleanClippingResult`).

O Revit costuma exportar dois níveis por pavimento, o de osso e o acabado, a poucos centímetros um
do outro; o app junta níveis a menos de 1 m, senão o pé-direito de um deles viraria 6 cm e a área
contaria duas vezes. Modelo de arquitetura não tem viga, e o app avisa: o cálculo sai só com a laje.

O projeto inteiro é calculado **na importação**, não sob demanda: a gaveta **Projeto completo**
já abre com os números. Ela mostra, por pavimento, área de fôrma, metros de viga, grade, apoios,
placas, dias de montagem e de desmontagem, e aproveitamento. Para o projeto, o equipamento a
mobilizar sai do **pico** entre os pavimentos multiplicado pelos jogos simultâneos, e não da soma:
o jogo que sobe é o mesmo, e tem de atender o pavimento mais exigente. O consumível vem da soma
dividida pelo reaproveitamento.

Montagem e desmontagem saem da produtividade por m² de fôrma com a equipe informada (valores de
partida, editáveis na gaveta do orçamento). Desforma parcial e total e o reescoramento do Anexo C
entram numa linha do tempo pavimento a pavimento, em dias corridos do início da estrutura — e daí
sai quantos jogos a obra precisa de verdade: se o informado for menor, o app avisa que o
equipamento está subestimado. Os prazos de desforma são do projeto estrutural e prevalecem.

O **3D mostra o prédio inteiro** com o escoramento de cada pavimento, filtrável por faixa de
pavimentos e pelas mesmas caixas de peça da vista de um andar (laje, vigas, pilares, placas,
barrotes, longarinas, escoras). Acima de dois pavimentos a escora vira um retângulo de frente para
a câmera, uma face no lugar de doze, para a tela aguentar: a torre de 21 pavimentos do exemplo dá
24 mil faces, ou 11 mil sem placas e barrotes.

O nível mais baixo do modelo fica de fora: ele apoia no solo, não em escora.

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
