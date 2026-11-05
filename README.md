# Análise dos boletins de urna — 2º turno das Eleições 2026

Análise exploratória dos boletins de urna do 2º turno das eleições gerais de 2026,
a partir dos dados abertos do Tribunal Superior Eleitoral (TSE).

O conjunto cobre **2.088.107 linhas** e **46 colunas**, equivalentes a **499.205 seções
eleitorais** em 27 UFs mais o exterior (ZZ), com **158.744.977 eleitores aptos**.

---

## Fonte dos dados

| | |
|---|---|
| Origem | [TSE — Portal de Dados Abertos](https://dadosabertos.tse.jus.br/dataset/resultados-2026-boletim-de-urna) |
| Dataset | Resultados - 2026 - Boletim de Urna |
| Licença | Creative Commons Atribuição (CC BY) |
| Turno | 2º turno, 25/10/2026 (arquivos gerados em 26/10/2026) |

Os CSVs originais **não estão versionados** neste repositório: somavam 882 MB e um
deles passava do limite de 100 MB por arquivo do GitHub. Em `raw/FONTE_DOS_DADOS.txt`
estão o link e o passo a passo para baixá-los de novo.

Para rodar a análise os CSVs **não são necessários**: `bu_2t.parquet` já contém as
2.088.107 linhas e as 46 colunas originais, e é dele que o notebook parte.

---

## Estrutura

```
analise_urnas/
├── main_refinado.ipynb   # notebook da análise (24 gráficos)
├── main.ipynb            # versão anterior, mantida como histórico
├── bu_2t.parquet         # dados consolidados das 28 UFs (95 MB)
├── dados/
│   └── br_ufs.geojson    # malha das UFs, usada nos mapas coropléticos
├── raw/
│   ├── FONTE_DOS_DADOS.txt        # link do TSE e como rebaixar os CSVs
│   └── _leiame-boletimurnaweb.pdf # dicionário de dados oficial do TSE
└── plots/                # os 24 gráficos em PNG
```

---

## Como rodar

```bash
python -m venv .venv
.venv\Scripts\activate          # Windows;  no Linux/Mac: source .venv/bin/activate
pip install pandas numpy matplotlib pyarrow jupyter
jupyter notebook
```

Abra `main_refinado.ipynb` e ajuste a constante `RAIZ` na célula de configuração para
o caminho da pasta do projeto na sua máquina.

**Dependências:** apenas pandas, numpy, matplotlib e pyarrow. O código evita scipy e
seaborn de propósito — a correlação de Spearman sai por ranks e o KDE é gaussiano em
numpy puro, para o notebook rodar sem instalações extras. Os mapas são desenhados
direto em matplotlib, sem geopandas.

---

## Os gráficos

Todos saem em `plots/`, com fundo escuro e numerados por ordem de criação.

| # | Gráfico |
|---|---|
| 01 | Total de votos por opção |
| 02 | Percentual de votos válidos por UF |
| 03 | Taxa de abstenção por UF |
| 04 | Boletins recebidos por hora |
| 05 | Votos por atraso no encerramento da seção (prorrogação) |
| 06 | Apuração em 3D: Lula × Flávio × tempo |
| 07 | Mapa de comparecimento por UF |
| 08 | Comparecimento × abstenção por UF |
| 09 | Distribuição do comparecimento por seção |
| 10 | Eleitorado × comparecimento |
| 11 | Ranking de candidatos mais votados |
| 12 | Composição do voto por cargo |
| 13 | Concentração de votos (curva de Lorenz) |
| 14 | Heatmap município × partido |
| 15 | Boxplot de comparecimento por UF |
| 16 | Bolhas: eleitorado × abstenção por município |
| 17 | Municípios com mais abstenções absolutas |
| 18 | Duração de funcionamento das urnas |
| 19 | Tempo até a emissão do boletim |
| 20 | Biometria sem habilitação |
| 21 | Heatmap de horário de encerramento por UF |
| 22 | Fragmentação partidária |
| 23 | Votos inválidos por UF |
| 24 | Desigualdade de participação entre seções |

---

## Notas metodológicas

**Granularidade.** Cada seção aparece em várias linhas, uma por votável. Somar
`QT_APTOS`, `QT_COMPARECIMENTO` ou `QT_ABSTENCOES` direto sobre as linhas multiplica
esses valores. O notebook monta a tabela `sec` com uma linha por seção, usando a chave
`SG_UF + NM_MUNICIPIO + NR_ZONA + NR_SECAO`.

**Validações feitas antes de qualquer agregação:**

- A chave de 4 campos já é única — nenhuma seção tem mais de uma urna, então incluir
  `NR_URNA_EFETIVADA` não muda a contagem (499.205 nos dois casos).
- `QT_APTOS = QT_COMPARECIMENTO + QT_ABSTENCOES` em **100%** das seções.
- A soma das linhas por UF bate com o total de 2.088.107.

**Fuso horário.** Os horários vêm no fuso local de cada urna. A votação vai das 8h às
17h de Brasília, o que é 7h-16h em AM, MS, MT, RO e RR, e 6h-15h no AC. Sem converter,
essas UFs pareceriam adiantadas. A hora nominal de abertura sai do padrão da UF,
corrigida pela moda do município, porque o Amazonas tem municípios em dois fusos.
Depois da conversão, nenhuma seção encerra antes do horário previsto — o que indica
que a correção está certa.

**Votos inválidos.** `#NULO#` é marcador de campo vazio do TSE, diferente de voto
nulo. Linhas com `DS_TIPO_VOTAVEL = "#NULO#"` não são votos: registram seções sem
apuração (2 casos, ambos no exterior).

---

## Limitações do conjunto

Três itens de análise comuns não cabem num 2º turno, e estão no notebook com a
limitação anotada:

- **Voto de legenda não existe** em 2º turno. Só há candidato, branco e nulo.
- **Concentração entre candidatos é trivial:** cada disputa tem 2 candidatos, então o
  Gini entre eles fica próximo de zero. A curva de Lorenz foi feita entre os 5.715
  municípios, onde há resolução de verdade.
- **Heatmap por partido é esparso:** só 8 partidos disputam, e apenas as 6 UFs que
  também elegeram governador (AC, AM, DF, ES, RN, TO) têm mais de dois.

Os dados são do boletim de urna e descrevem **o que foi registrado**, não por quê.
Correlações encontradas aqui não sustentam conclusões causais.
