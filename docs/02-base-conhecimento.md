# Base de Conhecimento

## Dados Utilizados


| Arquivo | Formato | Utilização no Agente |
|---------|---------|---------------------|
| `historico_atendimento.csv` | CSV | Contextualizar interações anteriores |
| `perfil_investidor` | JSON | Obter informações sobre aporte e tempo que o usuário deve deixar o valor investido |
| `produtos_financeiros.json` | JSON | Sugerir produtos adequados ao valor e prazo do investimento do cliente |

---

## Adaptações nos Dados

> Você modificou ou expandiu os dados mockados? Descreva aqui.

Como o prejeto é focado em recomendar a melhor opção de investimento em renda fixa, alterei o "perfil_investidor" para captar informações relevantes para sugerir investimentos em renda fixa, como data que o usuário deseja efetuar o resgate e o valor do aporte inicial,  incrementei mais opções de renda fixa e a dedução do imposto de renda(quando cabível)ao JSON "produtos_financeiros" e excluí as opções que não eram relacionadas a elas. Neste projeto não é necessário o uso de histórico de transações.

---

## Estratégia de Integração

### Como os dados são carregados?
> Existem duas possibilidades, injetar os dados diretamente no código(CTRL+C, CTRL+V), ou carregar os arquivos via código, como no exemplo abaixo:


```python


import pandas as pd
import json

histórico = pd.read_csv('data/historico_atendimento.csv')

with open('data/perfil_investidor.json', 'r', encoding='utf-8') as f:
    perfil_investidor = json.load(f)

with open('data/produtos_financeiros.json', 'r', encoding='utf-8') as f:
    produtos_financeiros = json.load(f)

```

Os JSON/CSV são carregados no início da sessão e incluídos no contexto do prompt

### Como os dados são usados no prompt?
> Os dados vão no system prompt? São consultados dinamicamente?

Para simplificar, podemos apenas "injetar" os dados em no prompt, garantindo que o agente tenha o melhor contexto possível. 
Lembrando que, em soluções mais robustas, o ideal é que essas informações sejam carregadas de maneira dinâmica, ganhando mais flexibilidade.

``` text

Dados do cliente:

nome, valor do aporte inicial, tempo que o aporte ficará investido e tipo de aporteúnico, semanal, mensal, semestral, anual.

data/perfil_investidor.json

```

``` json
{
  "perfil_investidor": {
    "nome": "João Silva",
    "valor_aporte_inicial": 20000.00,
    "tempo_investimento_meses": 36,
    "tipo_aporte": "Único"
  }
}

```

``` text
Histórico de atendimento:

data/historico_atendimento.csv

```
``` csv
data,canal,tema,resumo,resolvido
2025-09-15,chat,CDB,Cliente perguntou sobre rentabilidade e prazos,sim
2025-09-22,telefone,Problema no app,Erro ao visualizar extrato foi corrigido,sim
2025-10-01,chat,Tesouro Selic,Cliente pediu explicação sobre o funcionamento do Tesouro Direto,sim
2025-10-12,chat,Metas financeiras,Cliente acompanhou o progresso da reserva de emergência,sim
2025-10-25,email,Atualização cadastral,Cliente atualizou e-mail e telefone,sim
```

```text

Produtos de renda fixa:
Rentabilidade, vencimento, liquidez, IRPF

data/produtos_financeiros.json

```
``` json
[
  {
    "titulo": "Tesouro Selic",
    "rentabilidade": "Taxa Selic atual (15,00% a.a.)",
    "prazo_resgate": "D+1",
    "data_vencimento": "2029-03-01",
    "aporte_minimo": "R$ 30,00",
    "liquidez": "Diária",
    "cobra_irpf": true,
    "tabela_irpf_regressiva": {
      "até_180_dias": "22,5%",
      "181_a_360_dias": "20%",
      "361_a_720_dias": "17,5%",
      "acima_720_dias": "15%"
    }
  },
  {
    "titulo": "Tesouro IPCA+",
    "rentabilidade": "IPCA + 5% a.a. média",
    "prazo_resgate": "D+1",
    "data_vencimento": "2035-05-15",
    "aporte_minimo": "R$ 30,00",
    "liquidez": "Diária",
    "cobra_irpf": true,
    "tabela_irpf_regressiva": {
      "até_180_dias": "22,5%",
      "181_a_360_dias": "20%",
      "361_a_720_dias": "17,5%",
      "acima_720_dias": "15%"
    }
  },
  {
    "titulo": "Tesouro Prefixado",
    "rentabilidade": "Taxa prefixada (~11% a.a. média)",
    "prazo_resgate": "D+1",
    "data_vencimento": "2029-01-01",
    "aporte_minimo": "R$ 30,00",
    "liquidez": "Diária",
    "cobra_irpf": true,
    "tabela_irpf_regressiva": {
      "até_180_dias": "22,5%",
      "181_a_360_dias": "20%",
      "361_a_720_dias": "17,5%",
      "acima_720_dias": "15%"
    }
  },
  {
    "titulo": "CDB (Certificado de Depósito Bancário)",
    "rentabilidade": "CDI pós-fixado (atrelado à Selic atual de 15,00% a.a.), Prefixado ou IPCA+",
    "prazo_resgate": "Variável (D+0 até vencimento)",
    "data_vencimento": "6 meses a 5 anos",
    "aporte_minimo": "R$ 1.000,00 (varia por banco)",
    "liquidez": "Diária ou apenas no vencimento",
    "cobra_irpf": true,
    "tabela_irpf_regressiva": {
      "até_180_dias": "22,5%",
      "181_a_360_dias": "20%",
      "361_a_720_dias": "17,5%",
      "acima_720_dias": "15%"
    }
  },
  {
    "titulo": "LCI (Letra de Crédito Imobiliário)",
    "rentabilidade": "CDI (~11% a.a. média)",
    "prazo_resgate": "Mínimo 90 dias",
    "data_vencimento": "1 a 5 anos",
    "aporte_minimo": "R$ 5.000,00",
    "liquidez": "Somente no vencimento",
    "cobra_irpf": false,
    "aliquota_irpf": "Isento"
  },
  {
    "titulo": "LCA (Letra de Crédito do Agronegócio)",
    "rentabilidade": "CDI (~11% a.a. média)",
    "prazo_resgate": "Mínimo 90 dias",
    "data_vencimento": "1 a 5 anos",
    "aporte_minimo": "R$ 5.000,00",
    "liquidez": "Somente no vencimento",
    "cobra_irpf": false,
    "aliquota_irpf": "Isento"
  },
  {
    "titulo": "CRI (Certificado de Recebíveis Imobiliários)",
    "rentabilidade": "IPCA + 6% a.a. média",
    "prazo_resgate": "Até vencimento",
    "data_vencimento": "2 a 10 anos",
    "aporte_minimo": "R$ 10.000,00",
    "liquidez": "Somente no vencimento",
    "cobra_irpf": false,
    "aliquota_irpf": "Isento para pessoa física"
  },
  {
    "titulo": "CRA (Certificado de Recebíveis do Agronegócio)",
    "rentabilidade": "IPCA + 6% a.a. média",
    "prazo_resgate": "Até vencimento",
    "data_vencimento": "2 a 10 anos",
    "aporte_minimo": "R$ 10.000,00",
    "liquidez": "Somente no vencimento",
    "cobra_irpf": false,
    "aliquota_irpf": "Isento para pessoa física"
  },
  {
    "titulo": "Debêntures Incentivadas",
    "rentabilidade": "IPCA + 5,5% a.a. média",
    "prazo_resgate": "Até vencimento",
    "data_vencimento": "3 a 15 anos",
    "aporte_minimo": "R$ 1.000,00",
    "liquidez": "Somente no vencimento",
    "cobra_irpf": false,
    "aliquota_irpf": "Isento para pessoa física"
  },
  {
    "titulo": "Debêntures Comuns",
    "rentabilidade": "IPCA + 6% a.a. média",
    "prazo_resgate": "Até vencimento",
    "data_vencimento": "3 a 10 anos",
    "aporte_minimo": "R$ 1.000,00",
    "liquidez": "Somente no vencimento",
    "cobra_irpf": true,
    "tabela_irpf_regressiva": {
      "até_180_dias": "22,5%",
      "181_a_360_dias": "20%",
      "361_a_720_dias": "17,5%",
      "acima_720_dias": "15%"
    }
  }
]
```

---

## Exemplo de Contexto Montado

> Mostre um exemplo de como os dados são formatados para o agente.

O exemplo de contexto montado abaixo se baseia nos dados contidos na base de conhecimento, mas os sintetiza, deixando apenas as informações mais relevantes para otimizar o consumo de tokens.

```
🔎 Interpretação do Sistema

Investidor: João Silva
Capital disponível: R$ 20.000
Horizonte de investimento: 36 meses (3 anos)
Tipo de aporte: Único

Perfil comportamental implícito:
 -Já demonstrou interesse em Tesouro Selic e CDB
 -Busca informações sobre segurança e prazos
 -Acompanha metas financeiras (indício de perfil moderado/conservador)

Produtos financeiros disponíveis para sugestão:

- Tesouro Selic — Tributado (IR regressivo: 22,5% → 15%)
- Tesouro IPCA+ — Tributado (IR regressivo: 22,5% → 15%)
- Tesouro Prefixado — Tributado (IR regressivo: 22,5% → 15%)
- CDB (Certificado de Depósito Bancário) — Tributado (IR regressivo: 22,5% → 15%)
- LCI (Letra de Crédito Imobiliário) — Isento de IR para pessoa física
- LCA (Letra de Crédito do Agronegócio) — Isento de IR para pessoa física
- CRI (Certificado de Recebíveis Imobiliários) — Isento de IR para pessoa física
- CRA (Certificado de Recebíveis do Agronegócio) — Isento de IR para pessoa física
- Debêntures Incentivadas — Isento de IR para pessoa física
- Debêntures Comuns — Tributado (IR regressivo: 22,5% → 15%)
...
```
