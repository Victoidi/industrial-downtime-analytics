# Pipeline de Tratamento de Dados de Paradas Operacionais

## Visão geral

Este projeto consiste em um pipeline desenvolvido em Python para automatizar o tratamento de bases de dados  fictícia relacionadas a **paradas operacionais**. O objetivo principal é transformar arquivos brutos em planilhas organizadas, consolidadas e prontas para análise.

O notebook realiza etapas como:

- leitura de arquivos em diferentes formatos;
- padronização dos nomes das colunas;
- correção de classificações de paradas;
- agrupamento de horas por dia, mês, tipo de parada e justificativa;
- geração de relatórios em Excel;
- compactação dos arquivos processados para download.

A proposta do projeto é reduzir o trabalho manual no tratamento dos dados e criar uma estrutura reutilizável para análises futuras em ferramentas como Excel, Power BI ou Python.

---

## Objetivo do projeto

O objetivo do pipeline é preparar uma base de dados operacional para análise, separando e consolidando as informações de parada em relatórios mais fáceis de interpretar.

A partir da base original, o código gera saídas como:

- tempo de paradas programadas por dia;
- tempo de paradas não programadas por dia;
- total de horas por tipo de parada;
- ranking de justificativas com maior tempo acumulado;
- base processada consolidada;
- relatórios separados por mês ou por período completo.

---

## Ferramentas utilizadas

| Ferramenta | Como foi usada no projeto | Por que foi usada |
|---|---|---|
| **Google Colab** | Ambiente utilizado para executar o notebook e conectar com o Google Drive. | Facilita a execução do código sem configuração local e permite acessar arquivos diretamente do Drive. |
| **Google Drive** | Usado como local de armazenamento das bases brutas e dos arquivos processados. | Permite organizar os arquivos de entrada e saída em pastas, mantendo o fluxo mais prático no Colab. |
| **Excel** | Usado como formato de entrada e saída dos dados. O pipeline lê planilhas e exporta relatórios `.xlsx` com múltiplas abas. | É um formato comum em ambientes operacionais e facilita o compartilhamento dos resultados com áreas de negócio. |
| **Python** | Linguagem principal usada para construir o pipeline de tratamento. | Permite automatizar processos repetitivos, criar funções reutilizáveis e manipular dados com flexibilidade. |
| **Pandas** | Biblioteca usada para leitura, tratamento, agrupamento e exportação dos dados. | É uma das principais bibliotecas para manipulação de dados tabulares em Python. |
| **os** | Usado para percorrer diretórios, montar caminhos de arquivos e criar pastas de destino. | Ajuda a automatizar a busca por arquivos e a organização das saídas. |
| **datetime/timedelta** | Usado para converter segundos em formato de tempo legível. | Facilita a apresentação dos tempos acumulados em formato de horas, minutos e segundos. |
| **shutil** | Usado para compactar a pasta de arquivos processados em `.zip`. | Facilita o download e o compartilhamento dos relatórios gerados. |
| **google.colab.drive/files** | Usado para montar o Drive no Colab e baixar arquivos compactados. | Integra o notebook com o ambiente do Google Colab. |
| **openpyxl** | Utilizado indiretamente pelo Pandas como mecanismo de escrita de arquivos Excel. | Permite salvar planilhas `.xlsx` com múltiplas abas. |

---

## Bibliotecas Python utilizadas

```python
import pandas as pd
import os
from datetime import timedelta
from genericpath import exists
import shutil
from google.colab import drive, files
```

### Pandas

O `pandas` é a biblioteca central do projeto. Ele foi usado para:

- ler arquivos `.csv`, `.xlsx`, `.xlsm` e `.ods`;
- renomear colunas;
- corrigir valores em colunas específicas;
- converter colunas de data;
- agrupar dados por dia, mês, tipo de parada e justificativa;
- somar tempos em segundos;
- criar DataFrames finais;
- exportar relatórios em Excel.

Exemplos de uso no projeto:

```python
pd.read_excel(origem_entrada)
df.rename(columns=mapeamento)
df.groupby('Justificativa')['Tempo_segundos'].sum()
df.to_excel(writer, sheet_name='programada', index=False)
```

### os

A biblioteca `os` foi usada para lidar com arquivos e diretórios.

Ela aparece principalmente em tarefas como:

- extrair nome do arquivo;
- identificar extensão;
- percorrer pastas;
- criar diretórios de saída;
- montar caminhos de arquivos.

Exemplos de uso:

```python
os.path.splitext(origem_entrada)
os.path.basename(origem_entrada)
os.walk(caminho_pasta)
os.makedirs(os.path.dirname(sheet_destino), exist_ok=True)
```

### datetime/timedelta

O `timedelta` foi usado para transformar segundos acumulados em tempo legível.

Exemplo:

```python
str(timedelta(seconds=float(s)))
```

Além disso, o notebook possui uma função personalizada chamada `formatar_horas`, que converte segundos para o formato `HH:MM:SS`, inclusive quando o total passa de 24 horas.

### shutil

A biblioteca `shutil` foi usada no final do notebook para compactar a pasta de arquivos processados.

Exemplo:

```python
shutil.make_archive(nome_arquivo_zip, 'zip', caminho_pasta)
```

### google.colab.drive e google.colab.files

Essas ferramentas foram usadas para:

- conectar o notebook ao Google Drive;
- acessar arquivos armazenados no Drive;
- baixar o arquivo `.zip` com os relatórios processados.

Exemplo:

```python
drive.mount('/content/drive')
files.download(f'{nome_arquivo_zip}.zip')
```

### genericpath.exists

A função `exists` foi importada no notebook, mas não é utilizada diretamente no fluxo atual. Uma melhoria futura seria remover essa importação ou usá-la para validar se os arquivos e diretórios existem antes da execução.

---

## Estrutura esperada dos dados

A base original precisa conter colunas relacionadas a:

| Coluna original | Coluna padronizada |
|---|---|
| Área | Area |
| Estação | Estacao |
| Produto ativo | Produto_ativo |
| Justificativa | Justificativa |
| Tipo Justificativa | Tipo_justificativa |
| Data Inicial | Data_inicial |
| Data Final | Data_final |
| Segundos | Tempo_segundos |
| HH:MM:SS | Tempo_total |

A padronização dos nomes das colunas facilita consultas futuras, uso em bancos de dados e integração com ferramentas de análise.

---

## Etapas do pipeline

### 1. Importação das bibliotecas

O notebook começa importando as bibliotecas necessárias para manipulação de dados, acesso ao sistema de arquivos, conexão com o Google Drive e compactação dos resultados.

---

### 2. Conexão com o Google Drive

O projeto foi desenvolvido no Google Colab e utiliza o Google Drive como local de armazenamento dos arquivos.

```python
from google.colab import drive, files
drive.mount('/content/drive')
```

Essa etapa permite que o notebook acesse as pastas com os arquivos brutos e salve os relatórios processados automaticamente.

---

### 3. Importação e validação dos arquivos

A função `importar_e_validar_arquivo` identifica a extensão do arquivo e realiza a leitura adequada.

Formatos aceitos:

- `.csv`
- `.xlsx`
- `.xlsm`
- `.ods`

Essa etapa é importante porque evita que o pipeline tente processar arquivos em formatos inválidos.

---

### 4. Padronização dos nomes das colunas

A função `renomear_colunas` transforma nomes de colunas com espaços, acentos ou caracteres menos práticos em nomes mais simples.

Exemplo:

```text
Tipo Justificativa -> Tipo_justificativa
Data Inicial -> Data_inicial
Segundos -> Tempo_segundos
```

Essa padronização facilita o tratamento no Python e também deixa os dados mais adequados para uso posterior em bancos de dados ou ferramentas de BI.

---

### 5. Correção da classificação das justificativas

A função `corrigir_tipo_justificativa` corrige registros em que a justificativa é `"Parada   não declarada"`.

Nesse caso, o campo `Tipo_justificativa` é atualizado para `"Não Programada"`.

Essa regra de negócio é importante porque melhora a consistência dos dados antes da geração dos relatórios.

---

### 6. Geração de tabelas por mês

A função `tabela_tempo` agrupa os dados por dia e tipo de justificativa.

Ela separa os registros em:

- paradas programadas;
- paradas não programadas.

Depois, soma o tempo em segundos por dia e gera tabelas mensais com o tempo total.

---

### 7. Relatório mensal por justificativa

A função `gerar_relatorio_mensal_justificativas` calcula o tempo total por justificativa dentro de cada mês.

O resultado permite identificar quais motivos geraram maior tempo de parada no período analisado.

---

### 8. Exportação mensal dos arquivos

A função `exportar_arquivos_processados` cria arquivos Excel separados por mês.

Cada arquivo pode conter abas como:

- `programada`;
- `nao_programada`;
- `Horas totais`;
- `Tempo justificativa`.

Essa estrutura facilita a análise mensal e permite visualizar rapidamente o comportamento das paradas.

---

### 9. Exportação consolidada por período

Além da separação mensal, o notebook também possui uma opção para gerar um relatório do período completo.

A função `gerar_tabelas_periodo` cria quatro tabelas principais:

- programadas por dia;
- não programadas por dia;
- horas totais do período;
- tempo por justificativa no período.

A função `exportar_arquivo_periodo` salva essas tabelas em um único arquivo Excel.

Esse modo é útil quando o objetivo é analisar o período inteiro sem separar mês a mês.

---

### 10. Escolha do modo de execução

O notebook permite escolher entre três formas de execução:

| Opção | Função |
|---|---|
| `M` | Gera relatórios separados por mês. |
| `P` | Gera relatório consolidado por período. |
| `T` | Executa os dois processos: mensal e por período. |

Essa escolha torna o pipeline mais flexível para diferentes necessidades de análise.

---

### 11. Compactação dos resultados

Ao final, o notebook pode compactar a pasta de arquivos processados em um arquivo `.zip`.

Isso facilita o download dos relatórios gerados no Google Colab.

---

## Saídas geradas

O pipeline gera relatórios em Excel com diferentes abas, dependendo do modo escolhido.

### Saída mensal

Arquivos organizados por ano, origem e mês:

```text
files/processed/mes/{ano}/{nome}/{nome}_{mes}.xlsx
```

Abas possíveis:

- `programada`
- `nao_programada`
- `Horas totais`
- `Tempo justificativa`

### Saída por período

Arquivo consolidado por período:

files/processed/periodo/{ano}/{nome}/{nome}_periodo.xlsx


Abas possíveis:

- `programada`
- `nao_programada`
- `Horas totais`
- `Tempo justificativa`
- `Base processada`

---

## Principais aprendizados do projeto

Durante o desenvolvimento deste pipeline, foram praticados conceitos importantes para projetos de dados, como:

- leitura e validação de arquivos;
- tratamento e padronização de dados;
- criação de funções reutilizáveis;
- aplicação de regras de negócio;
- agrupamentos com Pandas;
- conversão de tempo em segundos para formato legível;
- exportação de relatórios automatizados em Excel;
- organização de arquivos processados;
- preparação de bases para análises futuras.

---

## Conclusão

Este projeto demonstra a construção de um pipeline em Python para tratamento de dados operacionais, com foco na transformação de bases brutas em relatórios consolidados e prontos para análise.

O notebook mostra uma aplicação prática de Pandas, Google Colab e Excel para resolver um problema comum em dados: organizar informações dispersas, padronizar campos, aplicar regras de correção e gerar arquivos finais de forma automatizada.

A solução permite analisar paradas programadas e não programadas, acompanhar o tempo acumulado por dia, mês, período e justificativa, além de facilitar a preparação dos dados para visualizações futuras em Excel, Power BI ou outras ferramentas de análise.
