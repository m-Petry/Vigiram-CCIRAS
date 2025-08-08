### Guia rápido – Pipeline **Metabase → CSV → XLSX (BR-GLASS)**

Estrutura de pastas:

```
Vigiram/
├─ CSVtoXLSX/              ← notebook que gera os XLSX finais
├─ metabaseToCSV/          ← notebook que gera os vigiram-*.csv intermediários
├─ data/                   ← **todos** os .csv (entrada + intermediários)
├─ outputs/                ← apenas os .xlsx finais
├─ requirements.txt
└─ README.md
```

---

## 1 ▪ Extrair os CSVs do Metabase (salvar todos em `data/`)

* Abra a coleção **Vigiram/BR GLASS** do Metabase:
[Coleção Vigiram/BR GLASS](http://relatorios.hc-ufpe.ebserh/collection/591-vigiram-br-glass "Coleção Vigiram/BR GLASS")

**ITS – Perfis de Resistência (Vigiram)**

* Ajuste o filtro **`dthr_entrada`** para o mês (ex.: *jul 1–31, 2025*).
* Baixe em **CSV** como `its-[mmm][aa].csv` (ex.: `its-jul25.csv`).
* Repita para cada mês necessário e salve em `data/`.

**RSI (Vigiram)**

* Ajuste o filtro **“Entrada Amostra”** para o mês.
* Baixe em **CSV** como `rsi-[mmm][aa].csv` (ex.: `rsi-jul25.csv`).
* Salve em `data/`.

**AGHU (Vigiram)**

* Abra o **editor SQL** do modelo.
* Atualize a linha do período, por exemplo:

  ```sql
  AND amo.dthr_entrada >= '2025-07-01'
  AND amo.dthr_entrada <= '2025-07-31'  -- ajuste para o mês
  ```
* Execute e baixe em **CSV** como `aghu-[mmm][aa].csv` (ex.: `aghu-jul25.csv`).
* Salve em `data/`.

**Contagem pacientes amostra positiva (Vigiram)**

* No **SQL**, ajuste o período:

  ```sql
  WHERE
    "source"."dthr_entrada" >= timestamp '2025-01-01 00:00:00.000'
    AND "source"."dthr_entrada" <  timestamp '2025-07-01 00:00:00.000'
  ```
  *(exemplo: janeiro a junho de 2025)*
* Execute e baixe como **`Contagem pacientes amostra positiva.csv`** em `data/`.

**Contagem pacientes amostra negativa (Vigiram)**

* Repita o ajuste de período (mesmo intervalo).
* Execute e baixe como **`Contagem pacientes amostras negativas.csv`** em `data/`.

---

## 2 ▪ Gerar os `vigiram-*.csv` com **MetabaseToCSV.ipynb**

1. **Instale as dependências** (uma vez por máquina):

```bash
cd Vigiram
python -m venv .venv
.\.venv\Scripts\activate          # Linux/macOS: source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

2. Abra o **VS Code** (instale as extensões **Python** e **Jupyter**).
3. Abra `metabaseToCSV/MetabaseToCSV.ipynb`.
4. Na **terceira célula**, ajuste o ano (ex.: `ANO = "25"`).
5. **Run All**. Os arquivos `vigiram-[mmm][aa].csv` serão gerados em `data/`.

---

## 3 ▪ Converter para BR-GLASS com **CSVtoXLSX.ipynb**

1. Abra `CSVtoXLSX/CSVtoXLSX.ipynb`.
2. Na **última célula**, ajuste o ano (ex.: `processar_todos_arquivos_ano("25")`)

3. **Run All**. O notebook:

   * lê todos os `vigiram-*.csv` da pasta `data/`;
   * lê também `Contagem pacientes amostra positiva.csv` e `… amostras negativas.csv` de `data/`;
   * grava, em `outputs/`, os arquivos finais:
     `Copia_de_Modelo_BR-GLASS_vigiram-[mmm][aa].xlsx`.

4. Envie os `.xlsx` de `outputs/` para a **UACAP** validar e submeter.

---

## 4 ▪ Gerar planilhas CCIRAS com **CSVtoCCIRAS.ipynb**

1. Abra `CSVtoCCIRAS/CSVtoCCIRAS.ipynb`.
2. Ajuste os caminhos na última célula para os arquivos desejados ou use `processar_todos_arquivos_ano`.
3. **Run All**. O notebook lê os mesmos `vigiram-*.csv` e contagens de pacientes de `data/` e gera em `outputs/` o arquivo `CCIRAS_vigiram-[mmm][aa].xlsx` com quatro abas:
   * **Isolados Detalhados**
   * **Sensibilidade**
   * **Tendência Temporal**
   * **Indicadores**

---

### Dicas

* Para um mês específico:

  ```python
  gerar_excel_brglass(
      IN_DIR / "vigiram-jul25.csv",
      OUT_DIR / "Copia_de_Modelo_BR-GLASS_vigiram-jul25.xlsx"
  )
  ```
