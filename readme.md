# Projeto de Automação de Relatórios Laboratoriais (Vigiram)

Este projeto automatiza a extração, consolidação e transformação de dados laboratoriais do Hospital Universitário para a geração de relatórios específicos para o sistema BR-GLASS e para a Comissão de Controle de Infecções Relacionadas à Saúde (CCIRAS).

## 📁 Estrutura de Pastas

Para o correto funcionamento dos scripts, o projeto **deve** seguir a seguinte estrutura de diretórios:

```
Vigiram/
   |
   |-- 📁 data/                 <-- Armazena TODOS os arquivos .csv (entrada + intermediários)
   |
   |-- 📁 outputs/              <-- Armazena APENAS os relatórios .xlsx finais
   |
   |-- 📁 notebooks/            <-- Contém os notebooks Jupyter do projeto
   |    |-- MetabaseToCSV.ipynb
   |    |-- CSVtoXLSX.ipynb
   |    +-- GeradorRelatorioCCIRAS.ipynb
   |
   +-- 📜 requirements.txt     
   +-- 📜 readme.md            
```

## ⚙️ Fluxo de Trabalho

O pipeline de dados é executado em fases distintas, utilizando três notebooks principais localizados na pasta `notebooks/`:

1.  **Fase 1: Consolidação de Dados**
    * **`notebooks/MetabaseToCSV.ipynb`**: Ponto de partida do fluxo. Ele lê múltiplos arquivos CSV brutos (exportados do Metabase), limpa, une e consolida-os em um único CSV padronizado, o `vigiram-{mes}{ano}.csv`, que é salvo na pasta `data/`.

2.  **Fase 2: Geração de Relatórios**
    * O arquivo `vigiram-*.csv` gerado serve de entrada para dois processos paralelos:
        * **`notebooks/CSVtoXLSX.ipynb`**: Converte os dados para o formato exigido pelo sistema **BR-GLASS**. Realiza mapeamentos complexos para códigos numéricos e gera a planilha `Copia_de_Modelo_BR-GLASS_*.xlsx` na pasta `outputs/`.
        * **`notebooks/GeradorRelatorioCCIRAS.ipynb`**: Gera um relatório analítico detalhado para a **CCIRAS**. Cria a planilha `Relatorio_CCIRAS_HC-UFPE_*.xlsx` focada em vigilância epidemiológica, mantendo os nomes originais dos dados, na pasta `outputs/`.

## 🚀 Como Usar

### Passo 1: Configuração do Ambiente

1.  **Pré-requisitos:**
    * [Python](https://www.python.org/downloads/) (versão 3.9 ou superior)
    * [Visual Studio Code](https://code.visualstudio.com/) com as extensões **Python** e **Jupyter**.

2.  **Instalação das Dependências:**
    * Abra um terminal na pasta raiz do projeto (`Vigiram/`).
    * Crie e ative um ambiente virtual (altamente recomendado):
        ```bash
        # Criar ambiente virtual
        python -m venv .venv

        # Ativar no Windows (PowerShell)
        .\.venv\Scripts\activate

        # Ativar no Linux/macOS
        source .venv/bin/activate
        ```
    * Instale as bibliotecas necessárias usando o `requirements.txt`:
        ```bash
        pip install -r requirements.txt
        ```

### Passo 2: Extração dos CSVs do Metabase

**Importante:** Todos os arquivos extraídos devem ser salvos diretamente na pasta `data/`.

* Acesse a coleção **Vigiram/BR GLASS** no Metabase através do link:
    * **[Coleção Vigiram/BR GLASS](http://relatorios.hc-ufpe.ebserh/collection/591-vigiram-br-glass)**

Para cada mês desejado, extraia os seguintes relatórios:

* **ITS – Perfis de Resistência (Vigiram)**
    * Ajuste o filtro **`dthr_entrada`** para o mês (ex: *1 a 31 de julho de 2025*).
    * Baixe em **CSV** e salve como `its-jul25.csv`.

* **RSI (Vigiram)**
    * Ajuste o filtro **“Entrada Amostra”** para o mês.
    * Baixe em **CSV** e salve como `rsi-jul25.csv`.

* **AGHU (Vigiram)**
    * Abra o **editor SQL** do modelo.
    * Atualize as linhas que definem o período da consulta:
        ```sql
        AND amo.dthr_entrada >= '2025-07-01'
        AND amo.dthr_entrada <= '2025-07-31'  -- ajuste para o mês desejado
        ```
    * Execute a consulta e baixe em **CSV**, salvando como `aghu-jul25.csv`.

* **Contagem pacientes amostra positiva (Vigiram)**
    * No editor **SQL**, ajuste o período desejado:
        ```sql
        WHERE
          "source"."dthr_entrada" >= timestamp '2025-01-01 00:00:00.000'
          AND "source"."dthr_entrada" <  timestamp '2025-07-01 00:00:00.000'
        ```
        *(Este exemplo cobre de Janeiro a Junho de 2025)*
    * Execute e baixe como **`Contagem pacientes amostra positiva.csv`**.

* **Contagem pacientes amostra negativa (Vigiram)**
    * Repita o mesmo ajuste de período do item anterior.
    * Execute e baixe como **`Contagem pacientes amostras negativas.csv`**.

### Passo 3: Geração do CSV Consolidado

1.  No VS Code, abra o notebook `notebooks/MetabaseToCSV.ipynb`.
2.  Na **terceira célula**, ajuste a variável `ANO` para o ano desejado (ex: `ANO = "25"`).
3.  Execute todas as células (`Run All`). Os arquivos `vigiram-[mmm][aa].csv` serão gerados e salvos na pasta `data/`.

### Passo 4: Geração dos Relatórios Finais

* **Para o relatório BR-GLASS:**
    1.  Abra o notebook `notebooks/CSVtoXLSX.ipynb`.
    2.  Na **última célula**, ajuste o ano na chamada da função (ex: `processar_todos_arquivos_ano("25")`).
    3.  Execute todas as células. Os arquivos `.xlsx` serão salvos na pasta `outputs/`.
    4.  Envie os relatórios gerados para a **UACAP** para validação e submissão.

* **Para o relatório da CCIRAS:**
    1.  Abra o notebook `notebooks/GeradorRelatorioCCIRAS.ipynb`.
    2.  Na **última célula**, ajuste o ano na variável `ANO_A_PROCESSAR`.
    3.  Execute todas as células. O arquivo `Relatorio_CCIRAS_HC-UFPE_*.xlsx` será salvo na pasta `outputs/`.