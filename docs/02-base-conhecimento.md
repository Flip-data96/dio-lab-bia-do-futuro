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
      "id": "tesouro_selic",
      "nome": "Tesouro Selic",
      "categoria": "titulo_publico",
      "rentabilidade": {
        "tipo_taxa": "pos_fixada",
        "indexador": {
          "nome": "SELIC",
          "valor_atual_anual": 0.15,
          "percentual_aplicado": 1.0
        },
        "taxa_fixa_anual": 0.0
      },
      "prazo": {
        "vencimento_data": "2029-03-01",
        "liquidez_dias": 1
      },
      "aporte_minimo": 30.00,
      "tributacao": {
        "tipo": "ir_regressivo",
        "tabela": [
          { "dias_max": 180, "aliquota": 0.225 },
          { "dias_max": 360, "aliquota": 0.20 },
          { "dias_max": 720, "aliquota": 0.175 },
          { "dias_max": null, "aliquota": 0.15 }
        ]
      }
    },
    {
      "id": "tesouro_ipca",
      "nome": "Tesouro IPCA+",
      "categoria": "titulo_publico",
      "rentabilidade": {
        "tipo_taxa": "hibrida",
        "indexador": {
          "nome": "IPCA",
          "valor_atual_anual": 0.045,
          "percentual_aplicado": 1.0
        },
        "taxa_fixa_anual": 0.0743
      },
      "prazo": {
        "vencimento_data": "2035-05-15",
        "liquidez_dias": 1
      },
      "aporte_minimo": 30.00,
      "tributacao": {
        "tipo": "ir_regressivo",
        "tabela": [
          { "dias_max": 180, "aliquota": 0.225 },
          { "dias_max": 360, "aliquota": 0.20 },
          { "dias_max": 720, "aliquota": 0.175 },
          { "dias_max": null, "aliquota": 0.15 }
        ]
      }
    },
    {
      "id": "tesouro_prefixado",
      "nome": "Tesouro Prefixado",
      "categoria": "titulo_publico",
      "rentabilidade": {
        "tipo_taxa": "prefixada",
        "indexador": null,
        "taxa_fixa_anual": 0.1262
      },
      "prazo": {
        "vencimento_data": "2029-01-01",
        "liquidez_dias": 1
      },
      "aporte_minimo": 30.00,
      "tributacao": {
        "tipo": "ir_regressivo",
        "tabela": [
          { "dias_max": 180, "aliquota": 0.225 },
          { "dias_max": 360, "aliquota": 0.20 },
          { "dias_max": 720, "aliquota": 0.175 },
          { "dias_max": null, "aliquota": 0.15 }
        ]
      }
    },
    {
      "id": "cdb_cdi",
      "nome": "CDB CDI",
      "categoria": "renda_fixa_bancaria",
      "rentabilidade": {
        "tipo_taxa": "pos_fixada",
        "indexador": {
          "nome": "CDI",
          "valor_atual_anual": 0.15,
          "percentual_aplicado": 1.0
        },
        "taxa_fixa_anual": 0.0
      },
      "prazo": {
        "vencimento_anos_min": 0.5,
        "vencimento_anos_max": 5,
        "liquidez_variavel": true
      },
      "aporte_minimo": 1000.00,
      "tributacao": {
        "tipo": "ir_regressivo",
        "tabela": [
          { "dias_max": 180, "aliquota": 0.225 },
          { "dias_max": 360, "aliquota": 0.20 },
          { "dias_max": 720, "aliquota": 0.175 },
          { "dias_max": null, "aliquota": 0.15 }
        ]
      }
    },
    {
      "id": "lci_cdi",
      "nome": "LCI CDI",
      "categoria": "renda_fixa_isenta",
      "rentabilidade": {
        "tipo_taxa": "pos_fixada",
        "indexador": {
          "nome": "CDI",
          "valor_atual_anual": 0.1505,
          "percentual_aplicado": 1.0
        },
        "taxa_fixa_anual": 0.0
      },
      "prazo": {
        "vencimento_anos_min": 1,
        "vencimento_anos_max": 5,
        "carencia_dias": 90
      },
      "aporte_minimo": 5000.00,
      "tributacao": {
        "tipo": "isento"
      }
    },
    {
      "id": "lca_cdi",
      "nome": "LCA CDI",
      "categoria": "renda_fixa_isenta",
      "rentabilidade": {
        "tipo_taxa": "pos_fixada",
        "indexador": {
          "nome": "CDI",
          "valor_atual_anual": 0.146,
          "percentual_aplicado": 1.0
        },
        "taxa_fixa_anual": 0.0
      },
      "prazo": {
        "vencimento_anos_min": 1,
        "vencimento_anos_max": 5,
        "carencia_dias": 90
      },
      "aporte_minimo": 5000.00,
      "tributacao": {
        "tipo": "isento"
      }
    },
    {
      "id": "cri_ipca",
      "nome": "CRI IPCA+6%",
      "categoria": "renda_fixa_estruturada",
      "rentabilidade": {
        "tipo_taxa": "hibrida",
        "indexador": {
          "nome": "IPCA",
          "valor_atual_anual": 0.045,
          "percentual_aplicado": 1.0
        },
        "taxa_fixa_anual": 0.06
      },
      "prazo": {
        "vencimento_anos_min": 2,
        "vencimento_anos_max": 10
      },
      "aporte_minimo": 10000.00,
      "tributacao": {
        "tipo": "isento"
      }
    },
    {
      "id": "cra_prefixado",
      "nome": "CRA 12% a.a.",
      "categoria": "renda_fixa_estruturada",
      "rentabilidade": {
        "tipo_taxa": "prefixada",
        "indexador": null,
        "taxa_fixa_anual": 0.12
      },
      "prazo": {
        "vencimento_anos_min": 2,
        "vencimento_anos_max": 10
      },
      "aporte_minimo": 10000.00,
      "tributacao": {
        "tipo": "isento"
      }
    }
  ],

  "simulacao_padrao": {
    "parametros_entrada": {
      "valor_inicial": 10000,
      "prazo_anos": 2,
      "regime_juros": "composto"
    },
    "motor_calculo": {
      "formula_valor_futuro": "VF = VP * (1 + i)^t",
      "calculo_taxa_efetiva": "i = (indexador.valor_atual_anual * percentual_aplicado) + taxa_fixa_anual",
      "calculo_ir": "IR incide apenas sobre rendimento conforme tabela regressiva",
      "calculo_valor_liquido": "valor_liquido = valor_bruto - imposto"
    }
  }
}
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

...
```
