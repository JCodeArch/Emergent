# Correções do Dashboard Grafana - F5 Elasticsearch

## Problemas Identificados e Correções

### 1. KPI: Taxa de Erro (%) - ID 3
**Arquivo:** `01_KPI_TaxaErro_id3.json`

**Problema:** 
- As queries sem `bucketAggs` não retornam dados no formato esperado pelo Grafana 12 para expressões matemáticas
- Query de range `f5.status:[400 TO *]` pode não funcionar bem em todas as versões

**Correção:**
- Adicionado `date_histogram` com `interval: "auto"` em bucketAggs para garantir formato de série temporal
- Alterada query para `f5.status:>=400` (sintaxe mais confiável)
- Adicionado `hide: true` nas queries A e B para mostrar apenas o resultado C

---

### 2. KPI: Disponibilidade (%) - ID 4
**Arquivo:** `02_KPI_Disponibilidade_id4.json`

**Problema:**
- Mesmo problema do KPI de Taxa de Erro
- Query `f5.status:[0 TO 399]` pode falhar

**Correção:**
- Adicionado `date_histogram` em bucketAggs
- Alterada query para `f5.status:<400`
- Adicionado `hide: true` nas queries auxiliares

---

### 3. Top 10 Endpoints Mais Lentos - ID 10
**Arquivo:** `03_Top10EndpointsLentos_id10.json`

**Problema:**
- `orderBy: "1"` com métricas de `percentiles` não funciona corretamente pois percentiles retorna um objeto
- O Elasticsearch não consegue ordenar pelo valor interno do percentil

**Correção:**
- Alterado `orderBy` para `"_term"` (ordenação alfabética inicial)
- Adicionado `transformations` com `sortBy` para ordenar pelo valor p95 após a consulta
- Adicionado `limit` para garantir apenas 10 resultados

---

### 4. Endpoints Instáveis - ID 12
**Arquivo:** `04_EndpointsInstaveis_id12.json`

**Problema:**
- `extended_stats` retorna múltiplos campos (count, min, max, avg, sum, variance, std_deviation)
- `orderBy: "1"` não funciona porque não há um campo único para ordenar

**Correção:**
- Separadas as métricas em `avg`, `min`, `max` individuais + `extended_stats`
- Alterado `orderBy` para `"_term"`
- Adicionado `transformations` para renomear campos e ordenar por `Std Deviation`
- Melhorada a visualização com overrides de unidades

---

### 5. Taxa de Erro por Endpoint - ID 15
**Arquivo:** `05_TaxaErroPorEndpoint_id15.json`

**Problema:**
- `transformations` com `joinByField` usando campo `"Term"` que pode não existir
- Cálculo de `%Erro` com `calculateField` usando sintaxe incorreta para Grafana 12
- A sintaxe de `fields: { "A": "Erros", "B": "Total" }` está obsoleta

**Correção:**
- Alterado `joinByField` para usar `"f5.url.keyword"` diretamente
- Corrigida sintaxe do `calculateField` para usar `binary: { left, right, operator }`
- Adicionado campo intermediário `%Erro_raw` e depois multiplicação por 100
- Query alterada para `f5.status:>=400`
- Melhorada organização final com ordenação por `%Erro`

---

### 6. 4xx vs 5xx (Pie Chart) - ID 19
**Arquivo:** `06_4xxVs5xx_id19.json`

**Problema:**
- `bucketAggs: []` vazio não funciona para pie charts
- Duas queries separadas sem aggregation não geram labels distintos para o gráfico

**Correção:**
- Consolidado em uma única query usando `range` aggregation
- Definidos ranges `400-499` (4xx) e `500-599` (5xx) com labels
- Filtro base `f5.status:>=400` para pegar apenas erros
- Alterado `reduceOptions.calcs` para `["sum"]` ao invés de `["lastNotNull"]`
- Adicionados overrides de cor (laranja para 4xx, vermelho para 5xx)

---

## Como Usar

1. Abra o dashboard no Grafana
2. Edite cada painel problemático
3. Vá em "Panel JSON" ou exporte o painel
4. Substitua pelo conteúdo do arquivo correspondente
5. Salve e teste

## Alternativa: Substituição Completa

Se preferir substituir o dashboard inteiro, use o arquivo `dashboard-f5-elastic-grafana12-FIXED.json` que será gerado com todas as correções integradas.
