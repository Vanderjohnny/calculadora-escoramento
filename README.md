# Calculadora de Escoramento — NBR 15696

App web estático. Projeto e verificação de fôrmas e escoramentos conforme ABNT NBR 15696:
lê a planta de formas em DWG ou DXF, ou o modelo estrutural em IFC, lança a grade de escoramento
sobre o desenho, mostra a montagem em 3D e exporta o jogo de pranchas cotadas em PDF e a grade de
volta em DXF. Com o IFC o cálculo é do prédio inteiro, pavimento a pavimento.

## A tela

Em cima, o que se precisa para começar: o arquivo, a estrutura de apoio, o pavimento e um exemplo
pronto. Depois os números do pavimento, o desenho com as setas de subir e descer, e a tira de
custo do que está na tela.

Embaixo, três abas:

- **Orçamento** — preços, prazos, equipamento a mobilizar e o custo discriminado por pavimento.
- **Conferência** — a cadeia de cálculo de lajes e vigas, as fôrmas verticais do Anexo D, a
  estabilidade horizontal, a desforma e o memorial em texto corrido.
- **Avançado** — os elementos lidos do arquivo, o aproveitamento das chapas, os ajustes de leitura
  do desenho (escala, níveis, camadas, pano que dita a grade) e o catálogo de materiais e ações.

O interruptor **explicações**, à direita das abas, mostra o porquê de cada conta: de onde vem o
espaçamento, por que o equipamento sai do pico e não da soma, o que a norma manda. Fica desligado
para quem só quer o número. Os avisos — preço não é cotação, prazo de desforma é do projetista da
estrutura — aparecem sempre.

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

## Subir e descer o prédio

Acima do desenho ficam duas setas, **▲** e **▼**. A importação abre no pavimento mais baixo que
leva escora — por onde a obra começa — e a seta de subir percorre o prédio na ordem em que ele é
construído. A planta 2D e o 3D mostram sempre o mesmo pavimento, com as mesmas caixas de peça
(laje, vigas, pilares, placas, barrotes, longarinas, escoras).

No 3D o prédio aparece **como ele está no dia em que aquele pavimento é concretado**: o que já
subiu fica de pé, em concreto, sem escoramento — que já foi desmontado e levado para cima — e só
o pavimento da vez aparece escorado. Subindo com a seta, o prédio cresce. A planta 2D mostra um
pavimento de cada vez. A caixa **escorar todos os pavimentos** desenha o escoramento em todos de
uma vez, para ver o total de equipamento montado; é pesado de girar.

Logo abaixo do desenho, uma tira mostra o que está na tela: área de fôrma, metros de viga, apoios,
placas, dias de montagem e desmontagem, e o custo — serviço, equipamento e consumível. Na gaveta
do orçamento, o mesmo custo vem **discriminado linha a linha por pavimento**, com o R$/m² de cada
um: é onde se vê que o pavimento de transição custa o dobro do tipo, coisa que a média esconde.

O desenho é leve porque a laje sai do contorno do pavimento, não de um retângulo do tamanho da
janela de desenho, e porque no modo prédio inteiro a escora vira um retângulo de frente para a
câmera e o que fica debaixo da laje (placas, barrotes, longarinas) sai do desenho. Um pavimento
tipo dá 2 mil faces; a torre de 20 pavimentos do exemplo, 8,7 mil.

## O que não se escora

Fundação não é pavimento: radier, blocos, sapatas, baldrames e tubulões são concretados contra o
solo e contra a fôrma lateral, nunca sobre escora — o app os reconhece pelo nome e os tira da lista.

Do que sobra, **o piso mais baixo também não leva escora**: é o térreo, concretado sobre o terreno.
O primeiro pavimento escorado é o que vem acima dele. Quando há subsolo, o subsolo é que assenta no
solo e o térreo passa a ser escorado — a regra sai sozinha, porque é sempre o mais baixo que fica
de fora.

Se esse piso mais baixo for laje sobre subsolo ou sobre vazio, e não sobre terreno, a caixa
**escorar também o piso mais baixo**, na gaveta do projeto, o traz para o cálculo. Vale conferir
quando o modelo não traz laje no térreo: aí o app toma o pavimento seguinte como o do solo.

O nível mais baixo do modelo fica de fora: ele apoia no solo, não em escora.

## Exportar

**Gerar PDF** — jogo de sete pranchas em A4 paisagem, ocupando a folha inteira: uma geral, o plano
de corte das chapas e cinco cotadas, uma por elemento (placas, barrotes, longarinas, escoras de
laje, escoras de viga). Ao lado do desenho, cada folha traz o seu próprio quantitativo, a legenda
dos traços que aparecem nela e o custo do pavimento — montagem, desmontagem, serviço, equipamento
e consumível. A folha se explica sozinha no canteiro.

Toda cota sai de dois eixos de referência, o **A** e o **1**, e os dois se cruzam no centro de
um pilar — o mesmo pilar em todos os pavimentos, escolhido entre os que atravessam o prédio
inteiro. É isso que deixa a cota de um andar comparável com a do outro e com a planta de formas.

**Gerar memoriais** — o memorial de cálculo em texto corrido, com o quadro de pavimentos e a linha
do tempo da obra, em arquivo separado do jogo de pranchas.

**DXF** — a grade de volta para o CAD, em metros, no mesmo sistema de coordenadas da planta
importada, separada em camadas: `ESC-PLACAS`, `ESC-BARROTES`, `ESC-LONGARINAS`, `ESC-ESCORAS`,
`ESC-ESCORAS-VIGA`, `ESC-COTAS`, `ESC-EIXOS`, `ESC-MARCAS` e `PLANTA`.

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
