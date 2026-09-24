# TP1 — G3 — RSNA 2021 Brain Tumor (MGMT Methylation)

**Disciplina:** Tópicos Especiais em Sistemas de Informação
**Professor:** Prof. Me. Décio Gonçalves de Aguiar Neto
**Grupo G3:** Alex Oliveira Silva, Kaike Ferreira Alves, João Pedro Pereira Rabelo
**Entrega final:** 04/10/2026

## Objetivo

Construir, avaliar e documentar uma solução **baseline clássica** (sem redes
neurais profundas) para o desafio RSNA-MICCAI Brain Tumor Radiogenomic
Classification: prever, a partir de exames de ressonância magnética, se o
status de metilação do promotor MGMT do tumor é positivo ou negativo.

Restrição do trabalho: proibido o uso de deep learning, tanto treinado de ponta
a ponta quanto como extrator de características pré-treinado. Todos os
descritores deste repositório são manuais (GLCM, Hu/forma, Sobel/Canny) e os
modelos são clássicos (regressão logística, Random Forest, SVM).

## Dados

Dataset hospedado no Kaggle, na página do desafio RSNA-MICCAI Brain Tumor
Radiogenomic Classification (RSNA AI Challenges,
https://www.rsna.org/artificial-intelligence/ai-image-challenge).
Requer cadastro e aceite dos termos de uso na página da competição. Os dados
brutos (imagens DICOM) **não** estão neste repositório — apenas o caminho para
obtê-los.

O conjunto de teste oficial não tem rótulos públicos; todo o protocolo foi
construído sobre o conjunto de treino (`train/` + `train_labels.csv`).

### Amostra utilizada

Amostragem estratificada por classe, **30 pacientes por classe (60 no total)**,
semente fixa = 42 (universo: 585 pacientes, 278 não metilados / 307 metilados).
Critério e contagem documentados em `notebooks/01_eda.ipynb` (seção 6). Lista de
IDs em `outputs/eda/amostra_g3.csv`.

Partição por paciente (5 dobras, `StratifiedGroupKFold`) definida e **congelada**
no Marco 2, em `outputs/marco2/particao_pacientes_congelada.csv`. Ela não foi
alterada nos marcos seguintes.

## Estrutura do repositório

```
TP1_G3_brain-tumor-mgmt/
├── README.md
├── requirements.txt
├── formulacao_problema.md            # formulação do problema, decisões e achados
├── fichamento.md                     # fichamento das 10 referências + referências metodológicas
├── introducao_trabalhos_relacionados.md  # rascunho das seções 1 e 2 do artigo
├── notebooks/
│   ├── 01_eda.ipynb                  # Marco 1: análise exploratória
│   ├── 02_preprocessing_baseline_G3.ipynb   # Marco 2: pipeline, partição, baseline, 1ª família
│   └── 03_feature_extraction_modeling_G3.ipynb  # Marco 3: grade descritor x modelo, ablações
└── outputs/
    ├── eda/                          # Marco 1
    │   ├── amostra_g3.csv            # <- entrada dos marcos 2 e 3
    │   ├── distribuicao_classes.csv / .png
    │   ├── contagem_cortes_por_paciente.csv
    │   ├── estatisticas_cortes.csv
    │   ├── metadados_exemplo.csv
    │   ├── orientacao_por_modalidade_exemplo.csv
    │   └── exemplos_visuais_por_classe.png
    ├── marco2/                       # Marco 2
    │   ├── particao_pacientes_congelada.csv   # <- entrada do marco 3
    │   ├── exemplo_pre_processamento.png
    │   ├── baseline_trivial_metricas.csv / _resumo.csv
    │   ├── features_glcm_t1wce_g3.csv
    │   ├── experimentos_parametros_glcm.csv
    │   └── logreg_glcm_metricas.csv / _resumo.csv, comparativo_baseline_vs_logreg.csv
    └── marco3/                       # Marco 3 (25 arquivos)
        ├── features_todas_familias_g3.csv         # features das 3 famílias (corte central, T1wCE)
        ├── features_multicorte_g3.csv             # 9 cortes centrais por paciente
        ├── features_ablacao_nonorm_crop.csv / features_ablacao_norm_nocrop.csv
        ├── plano_t1wce_por_paciente.csv           # plano de aquisição (cabeçalho DICOM)
        ├── baseline_trivial_prior_metricas.csv
        ├── grade_comparativa_descritor_modelo.csv # tabela principal (12 combinações)
        ├── grade_metricas_por_dobra.csv
        ├── selecao_aninhada_por_dobra.csv
        ├── comparativo_baseline_vs_melhor.csv
        ├── incerteza_melhor_combinacao.csv        # IC bootstrap + teste de permutação
        ├── estudo_ablacao_preprocessamento.csv
        ├── estudo_ablacao_agregacao_cortes.csv
        ├── extra_selecao_features_combinado.csv
        ├── predicoes_oof_todas_combinacoes.csv / predicoes_oof_melhor_combinacao.csv
        ├── predicoes_com_plano.csv / erros_melhor_combinacao.csv
        ├── metadados_execucao.json                # sementes e versões das bibliotecas
        └── grafico_auc_descritor_modelo.png, roc_melhor_combinacao.png,
            pr_melhor_combinacao.png, matriz_confusao_melhor_combinacao.png,
            permutacao_auc_nula.png, exemplos_erros_melhor_combinacao.png
```

## Como executar

**Dependências:** `pip install -r requirements.txt` (versões registradas em
`outputs/marco3/metadados_execucao.json`; Python 3.12). Nos notebooks do Kaggle
a primeira célula já instala o necessário.

**Dados:** aceite os termos na página da competição e adicione o dataset em
"Add Input" (Kaggle) ou baixe-o localmente e aponte a variável de ambiente
`RSNA_DATA_DIR` para a pasta que contém `train/` e `train_labels.csv`. Não há
caminhos absolutos de máquina local no código de leitura dos dados.

**Ordem de execução** (a partir da raiz do repositório):

1. `notebooks/01_eda.ipynb` → gera `outputs_eda_g3/` (inclui `amostra_g3.csv`).
2. `notebooks/02_preprocessing_baseline_G3.ipynb` → gera `outputs_marco2_g3/`
   (inclui a partição congelada). Depende de `amostra_g3.csv`: ajuste `AMOSTRA_CSV`
   na primeira célula para o local do arquivo, se necessário.
3. `notebooks/03_feature_extraction_modeling_G3.ipynb` → gera `outputs_marco3_g3/`.
   Procura `amostra_g3.csv` e `particao_pacientes_congelada.csv` em `outputs/eda`,
   `outputs/marco2`, nas pastas `outputs_*_g3/` ou em `/kaggle/input/...`.

O notebook 03 usa **cache de features**: se `features_todas_familias_g3.csv`
(e os demais CSVs de features) já existirem, não relê os DICOM. Para reproduzir
tudo do zero, a partir só dos dados brutos, ajuste na primeira célula
`USE_CACHE = False`. Sementes: `SEED = 42` em todos os notebooks e nas
permutações/bootstrap (`N_PERM = 200`, `N_BOOT = 2000`). Variação esperada entre
execuções: nenhuma nas métricas da grade (mesma partição e mesma semente);
diferenças mínimas de arredondamento podem ocorrer entre versões de
bibliotecas.

## Protocolo experimental (resumo)

- **Unidade de amostra:** paciente (uma linha por paciente; nunca por corte).
- **Modalidade:** T1wCE, corte central de cada paciente (agregação testada em ablação).
- **Pré-processamento:** normalização por percentil (1–99%), recorte da região
  não nula, redimensionamento 128×128.
- **Descritores (3 famílias):** textura GLCM/Haralick (12 features), forma/contorno
  (Hu + propriedades de região, 12), gradiente/bordas (histograma de orientações
  Sobel + Canny, 12); mais o conjunto Combinado (36).
- **Modelos:** regressão logística L2, Random Forest, SVM RBF, além do baseline
  trivial (classe majoritária).
- **Validação:** 5 dobras externas por paciente (congeladas) + validação cruzada
  aninhada (3 dobras internas, `GridSearchCV`) para os hiperparâmetros; padronização
  dentro do `Pipeline`, ajustada só no treino de cada dobra.
- **Métricas:** AUC-ROC, AUC-PR, acurácia balanceada, sensibilidade, especificidade,
  F1 (acurácia isolada não é usada como critério); média ± desvio entre dobras e
  AUC agrupado (fora-da-dobra) com IC 95% por bootstrap; teste de permutação.

## Principais resultados (Marco 3)

AUC médio ± desvio entre as 5 dobras:

| Descritor | Regressão logística | Random Forest | SVM (RBF) |
|---|---|---|---|
| GLCM | **0,738 ± 0,113** | 0,574 ± 0,151 | 0,460 ± 0,232 |
| Forma | 0,610 ± 0,083 | 0,568 ± 0,073 | 0,504 ± 0,124 |
| Gradiente | 0,598 ± 0,128 | 0,625 ± 0,095 | 0,552 ± 0,103 |
| Combinado | 0,691 ± 0,077 | 0,621 ± 0,090 | 0,396 ± 0,131 |

Comparação com o baseline trivial e estimativas mais conservadoras:

| Abordagem | AUC | Acurácia balanceada |
|---|---|---|
| Baseline trivial (classe majoritária) | 0,500 ± 0,000 | 0,500 |
| Melhor combinação (GLCM + LogReg), escolhida nas mesmas dobras | 0,738 ± 0,113 (agrupado 0,624; IC 95% 0,48–0,77) | 0,645 ± 0,080 |
| Seleção aninhada do par descritor × modelo (sem viés de seleção) | 0,542 ± 0,138 (agrupado 0,510) | 0,557 ± 0,058 |
| GLCM + LogReg, apenas pacientes com T1wCE axial (n = 52) | agrupado 0,523 (IC 95% 0,36–0,68) | — |

Ablações: normalização por percentil e recorte da região não nula mantiveram o
melhor AUC (0,738) contra 0,643 sem normalização e 0,693 sem recorte; o
*pooling* de múltiplos cortes centrais **não** superou o corte central (0,692–0,734
contra 0,738), que foi mantido.

**Leitura dos resultados.** O melhor par isolado tem sinal fraco (teste de
permutação p ≈ 0,005 para essa combinação, sem correção pela escolha entre as 12),
mas o IC do AUC agrupado inclui 0,5, a estimativa com seleção aninhada fica próxima
do acaso e, ao restringir aos exames axiais, o sinal desaparece. Na amostra, os 8
pacientes com T1wCE não axial (3 coronais, 5 sagitais) são todos metilados, e o modelo
tende a classificá-los como metilados: o **plano de aquisição é um confundidor**
(ver `formulacao_problema.md`). O resultado é coerente com Kim et al. (2022) e
Doniselli et al. (2024).

## Limitações

- Amostra pequena (60 pacientes): grande variância entre dobras; AUC de cada dobra
  calculado com 12 pacientes.
- O dataset da tarefa não traz máscara do tumor: a família "forma" descreve o contorno
  do tecido no corte, não do tumor; os descritores são calculados sobre o corte inteiro.
- Modalidade e parâmetros do GLCM (Marco 2) e o par descritor × modelo foram
  escolhidos olhando as mesmas dobras; por isso os números do "melhor par" são
  otimistas e a estimativa com seleção aninhada é a que deve ser tomada como desempenho
  do procedimento.
- Rótulo binário de MGMT com incerteza de medição (Brandner et al., 2021; Poon et al.,
  2021) e sem controle de subtipo molecular (Teske et al., 2022).
- Plano de aquisição da T1wCE varia entre pacientes e se associou ao rótulo na amostra.

## Status

- [x] Marco 1 — EDA, distribuição de classes, metadados DICOM, orientação por modalidade, exemplos visuais
- [x] Marco 2 — pré-processamento, partição por paciente congelada, baseline trivial, 1ª família de descritores, 1º classificador, experimentação de parâmetros do GLCM
- [x] Marco 3 — 3 famílias de descritores, 3 modelos, grade descritor × modelo, CV aninhada, ablações (pré-processamento e agregação de cortes), seleção aninhada, IC e permutação, análise de sensibilidade por plano
- [ ] Marco 4 — análise de erro qualitativa, artigo final (template SBC, 4 páginas, link do repositório no texto), teste de reprodutibilidade em ambiente limpo

**Antes da entrega:** (i) regenerar os CSVs de resultado do Marco 2 com a modalidade T1wCE
(`logreg_glcm_metricas.csv`, `logreg_glcm_resumo.csv`, `comparativo_baseline_vs_logreg.csv`);
(ii) testar a execução em ambiente limpo, com `USE_CACHE = False`, seguindo apenas este README.

## Uso de IA generativa

O assistente de IA **Claude (Anthropic)** foi utilizado como apoio nas atividades abaixo.
Em todos os casos o grupo executou o código no Kaggle, conferiu as saídas e validou as
decisões e as referências antes de incorporá-las ao projeto. O Claude não teve acesso aos
dados brutos DICOM: trabalhou com o código, os CSVs e as figuras gerados pelo grupo.

- **Marcos 1 e 2:** estruturação e rascunho de código dos notebooks de EDA e de
  pré-processamento/baseline, incluindo a leitura DICOM com ordenação por posição física,
  o pipeline de pré-processamento, a extração de descritores GLCM e o protocolo de
  validação cruzada por paciente.
- **Marco 3:** rascunho e revisão do notebook de experimentação comparativa (extração
  das famílias de forma e gradiente, grade descritor × modelo com validação cruzada
  aninhada, ablações, cache de features), e adição de métricas (AUC-PR, F1), bootstrap,
  teste de permutação, seleção aninhada do par descritor × modelo e análise de
  sensibilidade por plano de aquisição.
- **Análise de resultados e do enunciado:** apoio na interpretação dos números (viés de
  seleção, intervalos de confiança, confundidor de plano de aquisição) e na verificação de
  aderência ao enunciado do TP1, incluindo a detecção de inconsistências entre arquivos
  do repositório.
- **Documentação:** rascunho do fichamento, da formulação do problema, das seções de
  Introdução e Trabalhos Relacionados e deste README.
- **Referências:** a lista de 10 referências sobre MGMT foi organizada com apoio da IA;
  as referências metodológicas listadas em `fichamento.md` só entram no artigo depois de
  lidas pelo grupo. Nenhuma referência deve ser citada sem leitura e conferência do DOI.

Esta declaração também deve constar em nota ao final do artigo, indicando onde e para quê a IA foi usada.
