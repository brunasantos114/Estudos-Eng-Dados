**Qual a diferença entre row context e filter context?**

Row context é quando o DAX avalia uma expressão considerando uma linha específica, geralmente criado por colunas calculadas ou iteradores. Já o filter context define quais linhas da tabela estão visíveis naquele momento, vindo de filtros, slicers ou da própria fórmula. O ponto chave é que o DAX sempre calcula dentro de um conjunto de dados filtrado, e o row context por si só não filtra a tabela — ele só “aponta” para uma linha. Para que essa linha influencie o restante do cálculo, é necessário transformá-la em filtro.

Exemplo:
Dentro de um SUMX você está na linha de Produto A. Isso é row context. Se você usar CALCULATE, esse Produto A passa a ser um filtro, e aí você consegue calcular o total daquele produto na tabela inteira.

**Quando usar SUMX ao invés de SUM?**

SUMX deve ser usado quando o valor que você quer agregar não está pronto em uma coluna e precisa ser calculado linha a linha antes da soma. Isso acontece porque o DAX é orientado a colunas e não combina valores de diferentes colunas automaticamente sem um iterador.

Exemplo:
Para calcular receita (Quantidade × Preço), o SUM não resolve porque não existe uma coluna pronta. O SUMX cria um contexto de linha temporário, calcula a multiplicação em cada linha e depois agrega.

**O que o CALCULATE faz de fato?**

O CALCULATE altera o contexto de filtro antes de avaliar uma expressão. Ele pode adicionar, remover ou modificar filtros e também é o responsável por transformar row context em filter context (context transition). Em termos práticos, ele redefine “quais dados” entram no cálculo.

Exemplo:
Se você está vendo uma linha de Produto A e usa CALCULATE, ele pega esse valor e aplica como filtro na tabela inteira antes de calcular o total.

**Por que os totais às vezes estão errados no Power BI?**

Porque o total não soma os resultados das linhas, ele reexecuta a fórmula em um contexto mais amplo. Como o DAX sempre recalcula a expressão com base no contexto atual, qualquer lógica que dependa de granularidade pode produzir resultados diferentes no total.

Exemplo:
Uma média por produto funciona linha a linha, mas no total ela é recalculada considerando todos os produtos juntos, e não somando as médias individuais.

**Explique context transition com um exemplo**

Context transition acontece quando um valor de uma linha é transformado em filtro para calcular algo na tabela inteira. Isso ocorre quando existe row context e você usa CALCULATE (explícito ou implícito).

Exemplo:
Você está em uma linha com Cliente João. Ao usar CALCULATE, o valor “João” vira um filtro e permite calcular o total de vendas desse cliente em toda a tabela.

**O que acontece ao usar uma medida dentro de SUMX?**

A medida é avaliada para cada linha do iterador, e como toda medida tem um CALCULATE implícito, ocorre context transition automaticamente. Isso significa que cada linha influencia o contexto em que a medida é calculada.

Exemplo:
Se você usa uma medida de total de vendas dentro de um SUMX, ela será recalculada para cada linha com base no contexto daquela linha.

**Como fazer uma média ponderada?**

Você precisa respeitar o peso de cada linha no cálculo. Isso é feito somando os valores multiplicados pelos pesos e dividindo pela soma dos pesos.

Exemplo:
Se um produto vende 100 unidades e outro vende 1, a média ponderada considera esse volume, diferente de uma média simples que trataria ambos igual.

**O que é star schema e por que usar?**

É um modelo onde uma tabela fato central se conecta a dimensões. Esse formato facilita a propagação de filtros e permite que o engine trabalhe com tabelas menores primeiro, reduzindo o volume processado.

Exemplo:
Quando você filtra um produto, o filtro vem da dimensão (pequena) e é aplicado na tabela fato (grande), evitando varrer dados desnecessários.

**O que é cardinalidade e como impacta o modelo?**

Cardinalidade é a quantidade de valores únicos em uma coluna. O engine usa compressão por dicionário, então colunas com muitos valores únicos ocupam mais espaço e são mais caras para processar.

Exemplo:
Uma coluna de categoria repete valores e comprime bem. Já uma coluna de ID único praticamente não comprime.

**O que é query folding e por que é importante?**

Query folding é quando o Power Query consegue traduzir transformações em consultas nativas da fonte. Isso faz com que o processamento aconteça antes dos dados chegarem ao Power BI, reduzindo volume e custo.

Exemplo:
Filtrar o ano 2024 pode virar um WHERE no SQL, evitando trazer dados de outros anos.

**O que faz o query folding quebrar?**

Qualquer operação que não tenha equivalente na linguagem da fonte. Quando isso acontece, o Power BI precisa materializar os dados e continuar localmente.

Exemplo:
Criar uma coluna com lógica complexa pode impedir a tradução para SQL e interromper o folding.

**Seu relatório está lento. O que você analisa primeiro?**

Primeiro o modelo: cardinalidade, colunas desnecessárias, relacionamentos e estrutura. Isso porque o custo de leitura e compressão vem antes do cálculo em si.

Exemplo:
Uma tabela com muitas colunas de texto únicas pode ser o principal gargalo, mesmo com DAX simples.

**Como modelar uma base com muitos dados (100M+)?**

É necessário reduzir o volume processado em cada etapa: modelo em estrela, agregações para consultas frequentes, particionamento para dividir dados e incremental refresh para evitar reprocessamento total.

Exemplo:
Consultas mensais usam tabela agregada, enquanto análises detalhadas acessam a tabela completa apenas quando necessário.

**O que são agregações?**

São tabelas resumidas que o engine usa automaticamente quando a consulta não precisa do nível detalhado. Ele escolhe a menor tabela capaz de responder à pergunta.

Exemplo:
Um gráfico por mês pode ser respondido por uma tabela mensal, sem acessar dados diários.

**O que é particionamento?**

É dividir uma tabela grande em partes menores que podem ser processadas independentemente.

Exemplo:
Dados separados por mês permitem atualizar apenas o mês atual sem tocar no histórico.

**O que é incremental refresh?**

É uma estratégia que combina particionamento com atualização seletiva, mantendo dados históricos fixos e recalculando apenas os mais recentes.

Exemplo:
Atualizar apenas os últimos dias enquanto mantém anos anteriores intactos.

**Quando usar Import vs DirectQuery?**

Import é ideal quando você quer performance e pode trabalhar com dados atualizados periodicamente, pois o cálculo acontece em memória. DirectQuery é usado quando há necessidade de acessar dados atualizados diretamente na fonte, mas cada interação depende da execução de queries externas.

Exemplo:
Relatório analítico → Import.
Dashboard operacional → DirectQuery.

**Os dados estão diferentes do Excel. O que fazer?**

Validar todo o pipeline: origem dos dados, transformações no Power Query, relacionamentos e medidas. O problema geralmente está em como os dados estão sendo modelados ou filtrados.

Exemplo:
Excel pode estar somando tudo sem relacionamento, enquanto o Power BI respeita chaves e evita duplicidade.