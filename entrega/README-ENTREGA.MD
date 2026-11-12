# Entrega — Desafio Técnico Onfly

## Consolidador de Reservas de Hotel

### 1. Objetivo

O objetivo deste desafio foi identificar e corrigir defeitos existentes em um CLI Python responsável por consolidar reservas de hotéis em um relatório JSON.

A abordagem adotada foi investigar os testes que apresentavam falhas, identificar suas causas e realizar correções pontuais, preservando a estrutura original do projeto, o contrato do CLI e os testes fornecidos.

### 2. Defeitos identificados e corrigidos

#### 2.1. Conversão de moedas

**Problema:** O relatório apresentava valores incorretos após a conversão para reais.

**Causa raiz:** A implementação dividia o valor da reserva pela taxa de câmbio, quando a regra de negócio exigia multiplicação.

**Correção:** Substituí a divisão pela multiplicação entre o valor original e a taxa de conversão para BRL.

**Validação:** O teste `test_currency_conversion_to_brl` passou após a correção.

#### 2.2. Cálculo de noites

**Problema:** Reservas que atravessavam meses apresentavam quantidades negativas de noites.

**Causa raiz:** O cálculo utilizava `checkout.day - checkin.day`, considerando apenas o dia do mês e ignorando a diferença completa entre as datas.

**Correção:** Substituí o cálculo pela diferença entre as datas, utilizando `(checkout - checkin).days * rooms`.

**Validação:** O teste `test_nights_across_month_boundary` passou após a alteração.

#### 2.3. Regras de status

**Problema:** Reservas não faturáveis eram incluídas nos valores consolidados.

**Causa raiz:** A condição utilizada para filtrar reservas permitia a inclusão de status diferentes de `CONFIRMED`.

**Correção:** Ajustei a condição para considerar exclusivamente reservas confirmadas nos cálculos de noites, faturamento e agrupamentos, preservando as contagens dos demais status.

**Validação:** O teste `test_status_rules_exclude_cancelled_and_no_show` passou após a correção.

#### 2.4. Ordenação dos hotéis

**Problema:** O ranking apresentava hotéis em ordem crescente de faturamento.

**Causa raiz:** A função `sorted()` utilizava o valor monetário como chave de ordenação crescente.

**Correção:** Ajustei a chave para ordenar o faturamento de forma decrescente, mantendo o identificador do hotel em ordem crescente como critério de desempate.

**Validação:** O teste `test_top_hotels_are_ranked_descending` passou após a alteração.

### 3. Metodologia de investigação

Iniciei a investigação executando os testes fornecidos com `pytest`, que inicialmente apresentaram cinco falhas.

Analisei as mensagens de erro e os valores esperados, relacionando cada falha às regras de negócio descritas no README do desafio.

Em seguida, localizei os trechos responsáveis pelos cálculos e filtros, realizando alterações pontuais e executando novamente os testes para verificar os resultados.

A correção da conversão cambial também resolveu a falha do teste de regressão, indicando que esse defeito afetava o relatório consolidado.

Durante a investigação, utilizei o ChatGPT para interpretar mensagens de erro, compreender as regras de negócio e discutir possíveis correções. A ferramenta também forneceu exemplos pontuais de código para ilustrar as soluções, como a utilização de operações com datas, condições de filtragem por status e critérios de ordenação.

Não compartilhei o código-fonte completo nem disponibilizei acesso ao repositório. A identificação dos trechos com defeitos, a aplicação das alterações e a execução dos testes foram realizadas por mim, diretamente no ambiente local.

As sugestões da IA foram utilizadas como referência para compreender e corrigir os problemas identificados, sem geração ou modificação automática dos arquivos do projeto.

### 4. Validação

**Testes automatizados**

Comando utilizado:

```bash
python -m pytest -q
```

Resultado informado após as correções:

```text
5 passed
```

Os cinco testes fornecidos passaram sem alterações nos arquivos de teste.

**Validação funcional**

Executei o CLI utilizando o arquivo `data/exemplo-validacao.csv` e comparei o relatório gerado com os valores esperados no enunciado.

| Indicador | Resultado |
|---|---:|
| Total de reservas | 12 |
| Reservas confirmadas | 9 |
| Reservas canceladas | 2 |
| No-show | 1 |
| Total de noites | 28 |
| Faturamento em BRL | 7566.00 |
| Diária média em BRL | 270.21 |

Os agrupamentos por mês e o ranking dos cinco hotéis também corresponderam à amostra de validação.

Verifiquei ainda a leitura do nome `Hotel Madrid, Gran Vía` em UTF-8, confirmando que o caractere acentuado estava preservado.

### 5. Como reproduzir a validação

Requisitos: Python 3.11 ou superior e dependências de `requirements-dev.txt`.

Instalar dependências:

```bash
python -m pip install -r requirements-dev.txt
```

Executar os testes:

```bash
python -m pytest -q
```

Executar a amostra de validação em Linux/macOS:

```bash
PYTHONPATH=src python -m onfly_report \
  --input data/exemplo-validacao.csv \
  --rates data/taxas_cambio.csv \
  --output output/exemplo.json
```

No PowerShell:

```powershell
$env:PYTHONPATH = "src"
python -m onfly_report --input data/exemplo-validacao.csv --rates data/taxas_cambio.csv --output output/exemplo.json
```

### 6. Considerações finais

Foram corrigidos quatro defeitos relacionados à conversão monetária, ao cálculo de noites, ao filtro de reservas faturáveis e à ordenação dos hotéis.

Após as alterações, os cinco testes fornecidos passaram, e os resultados da amostra de validação corresponderam aos valores especificados.

As correções foram realizadas de forma pontual, preservando o comportamento esperado e a estrutura geral do projeto.