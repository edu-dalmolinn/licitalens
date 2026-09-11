# Escopo do MVP — Sinalização de riscos em contratações do PNCP

## 1. Visão do produto

Construir uma ferramenta local de triagem que extraia dados do Portal Nacional de Contratações Públicas (PNCP) e sinalize processos de contratação potencialmente problemáticos para posterior investigação humana.

A ferramenta não determinará ilegalidade, fraude ou descumprimento da Lei nº 14.133/2021. Ela apresentará indícios de risco, evidências, cálculos e links para as fontes utilizadas.

## 2. Pergunta orientadora

> Em que medida as contratações públicas registradas no PNCP apresentam evidências de vantajosidade, isonomia, competição e adequação dos preços, conforme os objetivos do art. 11 da Lei nº 14.133/2021?

Inovação e desenvolvimento sustentável não serão avaliados no MVP.

## 3. Recorte do MVP

| Elemento | Decisão |
| --- | --- |
| Finalidade | Detecção e priorização de riscos para investigação humana |
| Unidade principal | Processo de contratação |
| Unidade analítica complementar | Item da contratação |
| Modalidade | Pregão eletrônico |
| Objeto | Bens padronizados |
| Abrangência institucional | Órgãos federais em todo o Brasil |
| Período | Últimos 12 meses completos em relação à execução da coleta |
| Atualização | Semanal |
| Execução | MVP local |
| Fontes | API do PNCP e documentos disponibilizados no próprio PNCP |
| Fontes externas | Não serão utilizadas |

## 4. Princípios de análise

1. Não haverá nota geral de risco de 0 a 100.
2. Cada alerta será representado por uma tag independente.
3. As tags poderão ser aplicadas ao item e ao processo.
4. Cada tag deverá apresentar severidade, regra utilizada, valores calculados, evidências e fonte.
5. Ausência de dados não será tratada como ausência de risco.
6. Quando não houver dados suficientes, a ferramenta usará uma tag específica de insuficiência de dados.
7. Os resultados deverão usar linguagem de indício, e não de conclusão: “preço atípico”, “potencial restrição” e “baixa competição observada”.

## 5. Estrutura mínima de uma tag

Cada ocorrência deverá conter, no mínimo:

- código e nome da tag;
- nível: item ou processo;
- severidade: baixa, média ou alta;
- identificador PNCP da contratação;
- identificador do item, quando aplicável;
- descrição objetiva da regra acionada;
- valores e parâmetros usados no cálculo;
- quantidade e escopo das observações comparáveis;
- trecho e referência do documento, quando houver análise textual;
- link para a contratação ou documento no PNCP;
- data de geração do alerta.

## 6. Adequação dos preços

### 6.1 Objetivo

Identificar preços unitários homologados que estejam significativamente acima ou abaixo da distribuição observada em compras públicas comparáveis.

Um preço baixo atípico será tratado como sinal para investigar possível inexequibilidade, sem concluir que a proposta seja inexequível. Um preço alto atípico será tratado como sinal de possível inadequação ou sobrepreço, também sem conclusão automática.

### 6.2 Definição de item comparável

Dois registros serão comparáveis somente quando apresentarem:

- o mesmo código de catálogo;
- a mesma unidade de medida; e
- descrições compatíveis.

A regra objetiva para determinar “descrições compatíveis” ainda deverá ser definida e testada.

Antes da comparação, deverão ser tratados registros com unidade incompatível, quantidade inválida, valor nulo ou não positivo e possíveis duplicidades.

### 6.3 Grupo de referência

A comparação seguirá uma hierarquia:

1. usar compras da mesma região nos últimos 12 meses completos;
2. se não houver pelo menos 20 compras comparáveis, ampliar para todo o Brasil;
3. se ainda não houver 20 compras comparáveis, não classificar o preço e aplicar `DADOS_INSUFICIENTES_PRECO`.

O próprio registro avaliado não deverá compor sua amostra de referência.

### 6.4 Regra estatística padronizada

Será utilizado o intervalo interquartil (IQR), com a mesma fórmula para todos os itens:

```text
IQR = Q3 - Q1
limite_inferior = Q1 - 1,5 × IQR
limite_superior = Q3 + 1,5 × IQR
```

Regras:

- preço unitário acima do limite superior: `PRECO_ACIMA_DA_FAIXA`;
- preço unitário abaixo do limite inferior: `PRECO_ABAIXO_DA_FAIXA`;
- menos de 20 comparáveis após o fallback nacional: `DADOS_INSUFICIENTES_PRECO`.

A distribuição e os quartis serão calculados separadamente para cada grupo de itens comparáveis, mas a fórmula, o multiplicador 1,5 e a amostra mínima de 20 serão fixos em todo o MVP.

### 6.5 Evidências exibidas

- preço unitário analisado;
- Q1, mediana, Q3 e IQR;
- limites inferior e superior;
- distância percentual em relação à mediana e ao limite ultrapassado;
- número de compras comparáveis;
- abrangência usada: regional ou nacional;
- lista das compras que formaram a referência.

## 7. Competição

### 7.1 Métrica principal

Contar fornecedores distintos que apresentaram proposta válida para cada item.

### 7.2 Regra padronizada

| Fornecedores distintos com proposta válida | Severidade |
| ---: | --- |
| 1 | Alta |
| 2 | Média |
| 3 | Baixa |
| 4 ou mais | Sem alerta de baixa competição |

Tags previstas:

- `FORNECEDOR_UNICO_VALIDO`;
- `BAIXA_COMPETICAO`;
- `DADOS_DE_COMPETICAO_INDISPONIVEIS`.

### 7.3 Fonte dos participantes

A ferramenta tentará usar dados estruturados do PNCP. Quando eles não contiverem todos os participantes e propostas, poderá extrair essas informações das atas e de outros documentos disponibilizados dentro do próprio PNCP.

Não haverá integração com Compras.gov.br ou qualquer outro sistema ou API externa.

## 8. Isonomia

A isonomia não pode ser comprovada apenas com os dados disponíveis. O MVP buscará sinais objetivos de possível restrição ou tratamento desigual por duas frentes.

### 8.1 Indicadores estruturados

Indicadores candidatos:

- intervalo entre publicação e abertura das propostas;
- proporção de propostas ou fornecedores desclassificados;
- concentração de vitórias por fornecedor;
- itens com apenas um fornecedor válido;
- recorrência de um mesmo fornecedor em contratações semelhantes do mesmo órgão.

Os limites para prazo curto, taxa elevada de desclassificação e concentração ainda deverão ser definidos antes da implementação.

### 8.2 Análise textual híbrida

O fluxo será:

1. regras e dicionários localizam trechos potencialmente relevantes em editais, termos de referência e documentos relacionados;
2. um modelo de linguagem analisa o trecho dentro do seu contexto;
3. a ferramenta apresenta o trecho, a fonte, a justificativa e o grau de confiança;
4. nenhuma sinalização textual será exibida sem a evidência documental correspondente.

Padrões candidatos:

- menção a marca ou modelo sem indicação clara de equivalência;
- especificações excessivamente detalhadas ou potencialmente direcionadas;
- exigências técnicas ou de habilitação aparentemente desproporcionais;
- restrições geográficas;
- prazos ou condições de entrega potencialmente restritivos.

Tags candidatas:

- `POTENCIAL_MENCAO_RESTRITIVA_A_MARCA`;
- `POTENCIAL_ESPECIFICACAO_RESTRITIVA`;
- `POTENCIAL_EXIGENCIA_DESPROPORCIONAL`;
- `PRAZO_POTENCIALMENTE_RESTRITIVO`;
- `ALTA_DESCLASSIFICACAO`;
- `CONCENTRACAO_DE_VENCEDOR`.

## 9. Vantajosidade

### 9.1 Limite conceitual

No MVP, a vantajosidade será aproximada exclusivamente pela economia financeira em relação ao valor estimado. Essa métrica não captura qualidade, desempenho, risco, custo do ciclo de vida ou benefícios gerados.

Por isso, o alerta será denominado `BAIXA_ECONOMIA_SOBRE_ESTIMATIVA`, e não “falta de vantajosidade”.

### 9.2 Fórmula

```text
economia_percentual =
    ((valor_estimado - valor_homologado) / valor_estimado) × 100
```

### 9.3 Regra padronizada

| Economia calculada | Severidade |
| ---: | --- |
| Menor ou igual a 0% | Alta |
| Maior que 0% e menor ou igual a 5% | Média |
| Acima de 5% | Sem alerta de baixa economia |

Se o valor estimado estiver ausente, for sigiloso, nulo ou inválido, a métrica não será calculada e será aplicada `DADOS_INSUFICIENTES_VANTAJOSIDADE`.

O percentual de economia deverá sempre ser interpretado junto ao indicador de adequação de preços, pois uma estimativa inicial inflada pode produzir uma economia aparente elevada.

## 10. Propagação de tags do item para o processo

As tags serão calculadas inicialmente no nível mais específico possível. A página do processo exibirá todas as ocorrências dos seus itens.

Para evitar uma pontuação geral arbitrária, o processo poderá ser ordenado por:

1. maior severidade encontrada;
2. quantidade de tags de severidade alta;
3. quantidade total de itens sinalizados;
4. valor financeiro associado aos itens sinalizados.

A presença de uma tag em um item não significará que todo o processo apresenta o mesmo problema.

## 11. Dashboard do MVP

### 11.1 Visão geral

- total de processos e itens analisados;
- total e proporção de registros com dados suficientes;
- quantidade de alertas por tag e severidade;
- valor financeiro associado aos itens sinalizados;
- distribuição por órgão, região, fornecedor e categoria de bem.

### 11.2 Filtros

- período;
- órgão e unidade compradora;
- região e UF;
- código ou descrição do item;
- fornecedor vencedor;
- tag;
- severidade;
- disponibilidade ou insuficiência de dados.

### 11.3 Página do processo

- identificação e link original no PNCP;
- órgão, modalidade, datas e valores;
- itens e fornecedores vencedores;
- tags do processo e dos itens;
- explicação de cada regra acionada;
- cálculos e grupo de compras comparáveis;
- trechos dos documentos usados como evidência;
- links para os documentos de origem.

## 12. Fora do escopo

- modalidades diferentes de pregão eletrônico;
- serviços, obras e bens sem padronização suficiente;
- estados, municípios e entidades não classificadas como órgãos federais;
- integrações com sistemas ou APIs externas;
- análise de inovação e desenvolvimento sustentável;
- fiscalização da execução contratual;
- detecção de superfaturamento ocorrido durante a execução;
- emissão de parecer jurídico ou conclusão de irregularidade;
- publicação para o público geral;
- autenticação, gestão de usuários e fluxo formal de revisão humana;
- definição de arquitetura tecnológica definitiva.

## 13. Decisões ainda pendentes

1. Critério técnico para validar compatibilidade entre descrições de itens com o mesmo código e unidade.
2. Tratamento de embalagens, apresentações, conversões de unidade e escalas de quantidade.
3. Necessidade de ajuste temporal dos preços dentro da janela de 12 meses.
4. Definição exata de “mesma região” e comportamento quando a localização estiver ausente.
5. Regras de severidade para preços acima e abaixo dos limites do IQR.
6. Limites dos indicadores estruturados de isonomia.
7. Dicionários, padrões e modelo usados na análise textual híbrida.
8. Critério para elevar e consolidar tags de item no nível do processo.
9. Campos efetivamente disponíveis e qualidade dos dados nos endpoints e documentos do PNCP.
10. Métricas de validação do MVP, como precisão dos alertas em uma amostra revisada manualmente.

## 14. Critérios mínimos de conclusão do MVP

O MVP será considerado funcional quando conseguir:

1. coletar semanalmente os pregões eletrônicos federais da janela definida;
2. recuperar seus itens, resultados e documentos disponíveis no PNCP;
3. formar grupos válidos de compras comparáveis;
4. executar as regras de preço, competição e economia;
5. executar uma primeira versão da análise de isonomia;
6. gerar tags explicáveis no nível de item e processo;
7. distinguir claramente risco, ausência de risco observado e insuficiência de dados;
8. permitir filtrar, priorizar e abrir as evidências de cada contratação sinalizada.

## 15. Referências iniciais

- [Lei nº 14.133/2021 — texto atualizado](https://legis.senado.leg.br/norma/33382036)
- [Dados abertos do PNCP](https://www.gov.br/pncp/pt-br/acesso-a-informacao/copy_of_dados-abertos)
- [Manual da API de Consultas do PNCP](https://www.gov.br/pncp/pt-br/pncp/copy_of_manuais/ManualPNCPAPIConsultasVerso1.0.pdf/%40%40display-file/file)

