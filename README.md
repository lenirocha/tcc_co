# Estratégias de tratamento de dados ausentes e outliers na regressão da concentração de CO

A implementação foi feita no **KNIME Analytics Platform** e a apresentação dos resultados no **Power BI**.

## Estrutura do repositorio

```
.
├── dados/
│   └── AirQuality.csv                     # dataset original
├── knime/
│   └── tcc_co_com_vazamento.knwf          # workflow com vazamento de dados
│   └── tcc_co_sem_vazamento.knwf          # workflow sem vazamento de dados
├── powerbi/
│   └── dashboards_sem_vazamento.pbix      # dashboards
│   └── dashbords_com_vazamento.pbix       # dashboards
│   └── dashbords_comparacao.pbix          # dashboards
├── resultados/
│   └── resultados_tcc_com_vazamento.csv   # métricas da versão com vazamento
│   └── resultados_tcc_sem_vazamento.csv   # métricas da versão sem vazamento
└── README.md
```

## Dataset

- **Fonte:** Air Quality Dataset (UCI), versão disponibilizada no Kaggle por fedesoriano.
- **Instâncias:** 9.358 registros horários de sensores de qualidade do ar.
- **Formato:** CSV delimitado por ponto e vírgula, com vírgula como separador decimal (UTF-8).
- **Valores ausentes:** codificados como `-200`, e não como célula vazia.

Decisões de preparação aplicadas antes dos tratamentos:

- exclusão da coluna `NMHC(GT)`, com cerca de 90% de valores ausentes;
- exclusão das colunas `Date` e `Time`;
- remoção dos registros com `CO(GT)` ausente (as estratégias de dados ausentes se aplicam apenas às variáveis preditoras);
- remoção dos registros em que todas as variáveis preditoras estão ausentes.

## Desenho experimental

Fatorial 3 × 3 × 2, totalizando 18 combinações.

| Fator | Níveis |
|---|---|
| Dados ausentes | A – remoção de registros incompletos; B – imputação pela mediana; C – KNN Imputer |
| Outliers | Z-score; IQR; Winsorização |
| Algoritmo | Regressão Linear; Random Forest |

- **Divisão treino/teste:** 70/30, aleatória (Table Partitioner, semente 42).
- **Métricas:** RMSE, MAE e R², calculadas pelo nó Numeric Scorer.

## Versões do workflow

**Com vazamento:** os tratamentos são aplicados ao conjunto completo e a divisão treino/teste ocorre depois.

**Sem vazamento:** antes da divisão ocorrem apenas operações por linha. Os parâmetros dos tratamentos (medianas, média e desvio-padrão, quartis e limites da Winsorização) são calculados somente no treino e aplicados ao teste. No KNN Imputer, os vizinhos usados para preencher os valores ausentes, tanto no treino quanto no teste, são buscados apenas entre os registros de treino. O Z-score e o IQR removem linhas apenas do treino; a Winsorização é aplicada ao teste com os limites do treino, sem alterar o `CO(GT)` do teste. No caminho A, a remoção de registros incompletos é aplicada a treino e teste.

## Como reproduzir
**KNIME**

1. Baixar os arquivos do repositório
- Na página do repositório, clique em Code → Download ZIP.
- Extraia o arquivo .zip em uma pasta do computador (por exemplo, Documentos\tcc).
2. Instalar o KNIME
- Acesse https://www.knime.com/downloads e baixe o KNIME Analytics Platform para o seu sistema operacional. O cadastro solicitado na página é opcional.
- Execute o instalador e mantenha as opções padrão.

Ao abrir o KNIME pela primeira vez, ele solicita uma pasta de trabalho (workspace). Aceite a pasta sugerida e clique em Launch.

Este trabalho foi desenvolvido na versão 5.12. Versões posteriores da série 5 devem abrir o workflow sem alterações.

3. Instalar a extensão Python

Os nós Python Script (usados no KNN Imputer e no Z-score) dependem de uma extensão que não vem instalada por padrão.
 - No canto superior direito do KNIME, clique no ícone de menu (☰) e depois em Install extensions.
- Na caixa de busca, digite Python Integration.
- Marque KNIME Python Integration e clique em Next, aceite os termos e clique em Finish.
- Reinicie o KNIME quando solicitado.

A extensão já traz um ambiente Python próprio, sem necessidade de instalar o Python separadamente.

4. Importar o workflow

Os workflows estão na pasta knime/ no formato .knwf (workflow compactado).

- Na tela inicial do KNIME, localize o painel Space explorer à esquerda (área Local space).
- Arraste o arquivo .knwf da pasta do computador para dentro do Local space.
- Dê dois cliques no workflow importado para abri-lo.
- Repita o procedimento para o segundo arquivo .knwf (versões com e sem vazamento).

5. Entender a tela do workflow

Cada caixa na tela é um nó, que executa uma etapa do processamento. As setas indicam o caminho dos dados de um nó para o próximo, da esquerda para a direita. As caixas de texto sobre grupos de nós identificam cada etapa (leitura, preparação, caminhos A, B e C, tratamento de outliers, modelagem e avaliação).

O indicador abaixo de cada nó mostra o seu estado:

**Cor e Significado**
Vermelho: nó não configurado (precisa de ajuste)
Amarelo:	nó configurado, mas ainda não executado
Verde:	nó executado com sucesso

Para navegar:

Zoom: Ctrl + roda do mouse, ou em **-** e **+** na própria interface.
Mover a tela: clique com o botão direito do mouse em uma área vazia e arraste, ou use as barras de rolagem.
Ver a configuração de um nó: clique uma vez sobre o nó; as opções aparecem no painel à direita.
Ver o código dos nós Python Script: dê dois cliques sobre o nó para abrir o editor.

6. Ajustar o caminho do dataset

O caminho do arquivo de dados aponta para o computador onde o projeto foi desenvolvido e precisa ser atualizado.

- Clique uma vez no nó de leitura do CSV (CSV Reader), no início do workflow.
- No painel à direita, no campo do arquivo, clique em Browse e selecione dados/AirQuality.csv na pasta extraída no passo 1.
- Clique em Apply.

7. Executar o workflow
- Clique no botão Execute all (ícone de duas setas ▶▶), na barra superior da tela do workflow.
Aguarde até que os indicadores dos nós fiquem verdes. A execução completa pode levar alguns minutos, por causa do KNN Imputer e do Random Forest.
Para executar apenas uma parte, clique com o botão direito sobre um nó e escolha Execute; os nós anteriores a ele também são executados.
8. Visualizar os resultados
Clique uma vez sobre um nó com indicador verde.
A tabela de saída desse nó aparece no painel inferior da tela.
Os nós Numeric Scorer mostram as métricas (RMSE, MAE e R²) de cada combinação.
A tabela consolidada com as 18 combinações também está disponível na pasta resultados/, que pode ser aberta diretamente no Excel, sem o KNIME.
Problemas comuns
Nó vermelho com aviso de arquivo não encontrado: o caminho do dataset não foi ajustado (passo 6).
Erro nos nós Python Script: a extensão Python não foi instalada ou o KNIME não foi reiniciado após a instalação (passo 3). Caso o erro mencione sklearn, acesse ☰ → Preferences → KNIME → Python e confirme que a opção Bundled está selecionada.
Aviso sobre extensões ausentes ao abrir o workflow: clique na opção oferecida pelo próprio KNIME para buscar e instalar as extensões indicadas.
### Power BI

1. Abra o arquivo `.pbix` no Power BI Desktop.
2. Atualize o caminho dos CSVs em **Página Inicial → Transformar dados → Configurações da fonte de dados → Alterar Fonte...**.
3. Clique em **Atualizar** para recarregar os dados.

## Ferramentas

- KNIME Analytics Platform 5.12
- Python (pandas, scikit-learn) via KNIME Python Integration
- Power BI Desktop

## Autoria

Leni — curso de Sistemas de Informação, <nome da instituição>, <ano>.
