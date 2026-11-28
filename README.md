# Projeto de Previsão de Passageiros — ANAC

Notebook de análise exploratória, preparação de dados, treinamento de modelos de previsão e avaliação da movimentação diária de passageiros por aeroporto.

## 1. Visão geral

O projeto trabalha com dados consolidados de passageiros por aeroporto e data. A variável-alvo representa a quantidade diária de passageiros, conforme a fonte utilizada.

O notebook contempla:

- Importação dos dados por **Pandas (CSV)** ou **DuckDB**.
- Seleção interativa da fonte, do aeroporto e do arquivo CSV por caixas suspensas (`ipywidgets`).
- Padronização dos nomes de colunas e validação dos dados.
- Análise exploratória (EDA), estatísticas descritivas e visualizações.
- Consulta de informações de aeroportos para obter latitude e longitude.
- Consulta de dados climáticos históricos pela API do Open-Meteo.
- Construção de variáveis de calendário e indicadores relacionados à pandemia.
- Treinamento e previsão com três modelos: Regressão Linear, LightGBM e KNN.
- Comparação gráfica entre passageiros observados e previsões.
- Avaliação por métricas, incluindo MAE e MAPE, e backtesting com horizonte de 92 dias.
- Exibição de um dicionário de dados ao final do notebook.

> **Importante:** o notebook mantém 2020 e 2021 na série histórica. O choque de demanda da COVID-19 é tratado explicitamente na metodologia, em vez de excluir esses registros.

## 2. Estrutura de pastas

Organize os arquivos desta forma:

```text
Projeto_ANAC/
├── Previsao_ANAC_clean_code_widget_corrigido_csv.ipynb
├── requirements.txt
└── PAX/
    ├── GRU_AEROPORTO.csv
    ├── CGH_AEROPORTO.csv
    ├── VCP_AEROPORTO.csv
    └── ...
```

A pasta `PAX` deve ficar no mesmo diretório de onde o Jupyter será iniciado (diretório de trabalho atual do notebook). O código procura arquivos `.csv` diretamente dentro dessa pasta, sem percorrer subpastas.

O nome de cada arquivo deve começar com o código IATA de três letras do aeroporto, por exemplo:

- `GRU_AEROPORTO.csv`
- `CGH_AEROPORTO.csv`
- `VCP_AEROPORTO.csv`

O código usa esse prefixo para agrupar os arquivos e preencher o seletor de aeroportos.

## 3. Formato esperado dos CSVs

O leitor identifica automaticamente o separador mais comum entre vírgula (`,`) e ponto e vírgula (`;`). O arquivo deve conter cabeçalho e, no mínimo, as colunas equivalentes a:

| Informação | Nomes aceitos pelo notebook |
|---|---|
| Data | `data`, `date` ou `ds` |
| Passageiros | `pax`, `passageiros` ou `y` |
| Código IATA (quando houver mais de um aeroporto na mesma base) | `sg_iata_origem`, `aeroporto`, `iata` ou `unique_id` |

Exemplo de CSV com ponto e vírgula:

```csv
data;sg_iata_origem;pax
2025-01-01;VCP;12500
2025-01-02;VCP;13200
2025-01-03;VCP;12800
```

Exemplo com vírgula:

```csv
data,sg_iata_origem,pax
2025-01-01,VCP,12500
2025-01-02,VCP,13200
2025-01-03,VCP,12800
```

Se o CSV contiver apenas um aeroporto e não possuir coluna de identificação, o notebook utiliza o código escolhido no seletor. As colunas de data e passageiros são obrigatórias. Os valores de data devem ser interpretáveis pelo Pandas e os valores de passageiros devem ser numéricos.

A leitura utiliza `utf-8-sig`, para também lidar com arquivos CSV que contenham BOM UTF-8.

## 4. Fonte de dados DuckDB

Além de CSV, o notebook oferece a opção DuckDB.

Na versão atual do notebook, a conexão está configurada no código com:

```python
USUARIO = "luizh"
DUCK = "anac_prod.db"
DB_PATH = rf"c:\Users\{USUARIO}\Documents\DB\{DUCK}"
```

**Antes de usar DuckDB em outro computador, ajuste `USUARIO` e/ou `DB_PATH` para o local real do arquivo `anac_prod.db`.**

A consulta espera encontrar a tabela `main.anac` com as colunas:

- `dt_partida_real`
- `sg_iata_origem`
- `nr_passag_pagos`
- `nr_passag_gratis`

A consulta soma passageiros pagos e gratuitos por data e aeroporto. O intervalo da consulta está definido no próprio SQL do notebook: a partir de 2015 e até `2026-09-30`.

A opção DuckDB não usa o arquivo CSV selecionado. Ela consulta diretamente o banco configurado.

## 5. Instalação

### Pré-requisitos

- Python 3.10 ou superior recomendado.
- Acesso à internet para instalar os pacotes e consultar os serviços externos utilizados pelo notebook.
- Os arquivos CSV na pasta `PAX`, ou o banco DuckDB configurado, conforme a fonte escolhida.

### 5.1. Criar um ambiente virtual (venv)

O ambiente virtual isola as bibliotecas deste projeto das bibliotecas de outros projetos e da instalação global do Python. Assim, as versões instaladas pelo `requirements.txt` ficam associadas somente ao Projeto ANAC.

Abra o terminal na pasta raiz do projeto, onde estão o notebook e o `requirements.txt`.

**Windows (PowerShell):**

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**Windows (Prompt de Comando / CMD):**

```bat
py -m venv .venv
.venv\Scripts\activate.bat
```

**Linux/macOS:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Depois da ativação, o terminal normalmente mostra `(.venv)` no início da linha. Isso indica que os próximos comandos Python e pip serão executados dentro do ambiente virtual.

Para sair do ambiente virtual:

```bash
deactivate
```

### 5.2. Instalar o requirements dentro da venv

Com a venv ativada, atualize o instalador e instale as dependências:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

**Importante:** execute esses comandos somente depois de ativar a `.venv`. Não é necessário instalar cada biblioteca manualmente. O `pip` lê o `requirements.txt` e instala as dependências listadas, além das dependências transitivas exigidas por elas.

### 5.3. Registrar a venv como kernel do Jupyter

Ainda com a venv ativada:

```bash
python -m ipykernel install --user --name projeto-anac --display-name "Python (Projeto ANAC)"
```

Esse comando permite selecionar o ambiente virtual diretamente dentro do Jupyter.

## 6. Como executar

1. Coloque o notebook e a pasta `PAX` na estrutura indicada.
2. Ative o ambiente virtual.
3. Inicie o Jupyter na pasta raiz do projeto:

   ```bash
   jupyter lab
   ```

   ou:

   ```bash
   jupyter notebook
   ```

4. Abra `Previsao_ANAC_clean_code_widget_corrigido_csv.ipynb`.
5. Selecione o kernel **Python (Projeto ANAC)**.
6. Execute as células em ordem, de cima para baixo.
7. Na seção **Importando Dados**, os controles aparecem ao executar a célula:
   - Escolha **Pandas (CSV)** ou **DuckDB**.
   - Para CSV, escolha o aeroporto e o arquivo.
   - Para DuckDB, informe o código IATA e confirme se o caminho do banco foi configurado.
   - Clique em **Carregar dados**.
8. Após a importação bem-sucedida, continue executando as células seguintes em ordem.

O widget depende de um ambiente Jupyter com suporte a `ipywidgets`. Se os controles não aparecerem, confirme que o notebook está usando o kernel no qual o `requirements.txt` foi instalado e atualize/reinicie a sessão do Jupyter.

## 7. Bibliotecas utilizadas

As dependências estão listadas em `requirements.txt`. Em resumo:

| Biblioteca | Finalidade no projeto |
|---|---|
| pandas / numpy | Manipulação, limpeza e cálculos de dados |
| matplotlib / seaborn | Gráficos e visualização |
| ipywidgets / IPython | Interface interativa de seleção no notebook |
| duckdb | Consultas SQL e leitura da base local |
| requests | Requisições às APIs externas |
| holidays | Informações de feriados |
| scikit-learn | Regressão Linear, KNN e padronização |
| lightgbm | Modelo LightGBM |
| mlforecast | Treinamento e geração de previsões com séries temporais |
| statsforecast / statsmodels | Ferramentas de séries temporais, sazonalidade e autocorrelação |
| utilsforecast | Métricas, preenchimento e visualização de séries |

## 8. Serviços externos e acesso à internet

O notebook consulta serviços externos durante o processamento:

- **OurAirports:** baixa o arquivo público de aeroportos para relacionar o código IATA às coordenadas geográficas.
- **Open-Meteo Archive API:** consulta variáveis climáticas históricas para as coordenadas do aeroporto, no intervalo das datas presentes na base.

É necessário permitir acesso HTTPS a esses serviços. A disponibilidade e o conteúdo das APIs são externos ao projeto e podem mudar.

## 9. Modelos e avaliação

O notebook utiliza os seguintes modelos:

- **Linear:** `LinearRegression` do scikit-learn.
- **LGBM:** `LGBMRegressor` do LightGBM.
- **KNN:** `ScaledKNNRegressor`, uma implementação baseada em `KNeighborsRegressor` com padronização das variáveis explicativas.

A variável-alvo é preservada na escala original (`y_orig`) e também transformada com `log1p` para o treinamento (`y`). As previsões são convertidas de volta para a escala de passageiros com `expm1`.

O notebook apresenta comparações entre valores observados e previstos e calcula métricas de erro. O backtesting utiliza os últimos 92 dias disponíveis como período de teste, conforme a implementação atual.

## 10. Por que o modelo Linear pode acompanhar pior os dados reais?

O notebook compara três abordagens diferentes: Regressão Linear (`Linear`), LightGBM (`LGBM`) e KNN (`KNN`). É esperado que as linhas de previsão não tenham o mesmo formato, porque cada algoritmo aprende relações diferentes nos dados.

### Regressão Linear

A Regressão Linear estima uma combinação aditiva das variáveis explicativas. No notebook, ela recebe defasagens da série (`lags`), variáveis de calendário e variáveis exógenas, como indicadores de feriados, períodos da pandemia/recuperação e clima. O alvo é treinado na escala `log1p` e convertido novamente para passageiros com `expm1`.

Essa abordagem é simples e útil como referência, mas tem limitações importantes para uma série de passageiros:

- **Relações essencialmente lineares:** o modelo estima coeficientes fixos. Ele não aprende automaticamente regras complexas, mudanças de regime ou interações não lineares entre calendário, clima e demanda.
- **Previsão recursiva com defasagens:** a previsão usa lags como 1, 7, 14, 30, 90, 180 e 365 dias. Conforme o horizonte avança, valores previstos passam a alimentar as defasagens futuras. Um desvio inicial pode se propagar e fazer a curva se afastar do comportamento real.
- **Mudanças de padrão e picos:** feriados, férias, eventos, recuperação de demanda e alterações operacionais podem produzir variações que não são bem representadas por uma combinação linear fixa.
- **Extrapolação:** a regressão pode continuar uma tendência estimada de forma pouco realista quando o futuro se afasta das combinações observadas no treinamento.

O uso de `log1p` ajuda a reduzir a assimetria da variável-alvo, mas não transforma a Regressão Linear em um modelo capaz de capturar automaticamente relações não lineares.

### Por que LightGBM e KNN podem parecer mais próximos?

- **LightGBM:** utiliza árvores de decisão em conjunto. Consegue representar relações não lineares, limiares e interações entre variáveis. Por isso, pode acompanhar padrões complexos melhor que uma regressão linear em determinados conjuntos de dados.
- **KNN:** estima a resposta a partir de observações de treinamento com características semelhantes. No notebook, as variáveis explicativas são padronizadas dentro do `ScaledKNNRegressor`, pois a distância entre variáveis em escalas diferentes poderia distorcer a busca por vizinhos.

Isso não significa que LightGBM e KNN serão sempre melhores. O KNN pode ter dificuldade em situações futuras sem vizinhos históricos parecidos; o LightGBM também pode não extrapolar tendências além dos padrões aprendidos. O desempenho deve ser medido no período de teste, e não apenas pela aparência do gráfico.

### Como interpretar o gráfico

A linha vertical tracejada marca o início do horizonte de previsão. Antes dela, há dados observados e, conforme o gráfico/célula, previsões retrospectivas para comparação. Depois dela, a curva representa o horizonte futuro do modelo. Não se deve esperar que uma previsão futura reproduza exatamente oscilações diárias ainda desconhecidas.

Para decidir qual modelo é mais adequado, compare as métricas MAE e MAPE no backtesting, usando o mesmo intervalo de datas e os mesmos valores reais para todos os modelos. Considere também os gráficos de resíduos e a estabilidade das previsões. Uma curva visualmente mais próxima em um trecho isolado não garante melhor desempenho geral.

**Observação metodológica:** o notebook usa os últimos 92 dias disponíveis como teste no backtesting. Essa avaliação é mais apropriada para comparar os modelos do que comparar visualmente somente as curvas de previsão.

## 11. Pontos de configuração e solução de problemas

### O seletor não mostra aeroportos

- Confirme que a pasta `PAX` existe no diretório de trabalho atual do Jupyter.
- Confirme que há arquivos `.csv` diretamente dentro dela.
- Confirme que os nomes começam com um código IATA de três letras, como `GRU_...csv`.

### Erro de colunas obrigatórias ausentes

Verifique se o CSV possui cabeçalho e colunas de data e passageiros reconhecíveis. O notebook aceita os nomes descritos na seção 3. Confira também se o separador é vírgula ou ponto e vírgula.

### O widget não aparece

- Confirme que `ipywidgets` foi instalado no mesmo ambiente do kernel.
- Selecione o kernel correto no Jupyter.
- Reinicie o kernel e execute novamente a célula da seção **Importando Dados**.

### Erro ao conectar ao DuckDB

- Confirme que o arquivo `anac_prod.db` existe no caminho configurado.
- Ajuste `USUARIO`/`DB_PATH` no notebook.
- Confirme que a tabela `main.anac` e as colunas esperadas estão presentes.

### Erro ao consultar clima ou cadastro de aeroportos

- Verifique a conexão com a internet/proxy/firewall.
- Tente novamente mais tarde se o serviço externo estiver indisponível.
- Confira se o código IATA selecionado existe no cadastro consultado.

### Resultados ou gráficos diferentes

Os resultados dependem dos dados de entrada, do aeroporto, do intervalo disponível, das versões das bibliotecas e da execução das células. Para uma comparação reproduzível, mantenha a mesma base de dados, a mesma configuração e execute o notebook desde o início, em ordem.

## 12. Limites e observações

- O notebook não inclui os dados ANAC/CSV nem o banco `anac_prod.db`; esses arquivos devem ser fornecidos separadamente.
- O caminho do DuckDB está definido para uma máquina específica e precisa ser ajustado em outros ambientes.
- O período máximo da consulta SQL DuckDB está fixado no código em `2026-09-30`; altere-o somente se desejar mudar o recorte da consulta.
- O notebook faz chamadas de rede para obter o cadastro de aeroportos e o histórico climático.
- `requirements.txt` lista as dependências Python diretas identificadas nos imports do notebook. O `pip` instala automaticamente as dependências transitivas.

---
Projeto de previsão de passageiros com dados ANAC.
