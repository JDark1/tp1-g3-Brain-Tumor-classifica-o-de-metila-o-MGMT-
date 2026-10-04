# TP1 — Baseline de Projeto de Pesquisa (G3)

**Disciplina:** Tópicos Especiais em Sistemas de Informação
**Professor:** Décio Gonçalves de Aguiar Neto
**Grupo G3:** Alex Oliveira Silva · Kaike Ferreira Alves · João Pedro Pereira Rabelo
**Desafio RSNA atribuído:** 2021 — Brain Tumor (classificação de metilação MGMT)

## Objetivo

Construir um baseline com métodos clássicos de visão computacional e aprendizado de
máquina (sem redes neurais profundas) para prever, a partir de exames de ressonância
magnética, se o status de metilação do promotor MGMT de um glioma é positivo ou
negativo — usando o dataset do desafio RSNA-MICCAI Brain Tumor Radiogenomic
Classification (2021).

O baseline final cobre **3 famílias de descritores** (textura GLCM/Haralick,
forma/momentos de Hu, gradiente/bordas Sobel+Canny — 12 atributos cada, 36 no
conjunto Combinado) e **3 modelos clássicos** (regressão logística L2, Random Forest,
SVM RBF), conforme exigido pelo §4.2/§4.3 do enunciado, com validação cruzada
aninhada (3 dobras internas) para hiperparâmetros.

**Sobre seleção de modelo — tratamento deliberadamente transparente, não só
"nunca olha o teste":** dentro de cada uma das 12 combinações, os hiperparâmetros são
escolhidos estritamente pela validação cruzada interna, nunca pelo conjunto de teste.
Mas a escolha de **qual das 12 combinações reportar como "a melhor"** foi feita
olhando as mesmas dobras de teste externas — isso é seleção de modelo pelo teste, um
viés real (§10 do enunciado). Em vez de esconder isso, o projeto reporta os dois
números lado a lado: o par otimista escolhido assim, e uma **seleção aninhada**, em
que o par é escolhido em cada dobra só pelo escore da validação interna, sem consultar
o teste — essa é a estimativa honesta do procedimento. Ver `formulacao_problema.md`.

## Estrutura do repositório

```
.
├── README.md
├── requirements.txt
├── fichamento.md                  # fichamento das 10 referências de MGMT + referências metodológicas
├── formulacao_problema.md         # definição da tarefa (entrada/saída/unidade de amostra), decisões e achados
├── notebooks/
│   ├── 01_eda.ipynb                            # Marco 1: EDA, metadados DICOM, amostragem estratificada
│   ├── 02_preprocessing_baseline_G3.ipynb      # Marco 2: pipeline, partição congelada, baseline, 1ª família
│   └── 03_feature_extraction_modeling_G3.ipynb # Marco 3: 3 famílias, grade descritor×modelo, ablações, IC
└── outputs/
    ├── eda/                        # Marco 1
    │   ├── amostra_g3.csv                      # <- entrada dos marcos 2 e 3
    │   ├── distribuicao_classes.csv / .png
    │   ├── contagem_cortes_por_paciente.csv
    │   ├── estatisticas_cortes.csv
    │   ├── metadados_exemplo.csv
    │   ├── orientacao_por_modalidade_exemplo.csv
    │   └── exemplos_visuais_por_classe.png
    ├── marco2/                     # Marco 2
    │   ├── particao_pacientes_congelada.csv    # <- entrada do marco 3; congelada, nunca recalculada
    │   ├── exemplo_pre_processamento.png
    │   ├── baseline_trivial_metricas.csv / _resumo.csv
    │   ├── features_glcm_t1wce_g3.csv
    │   ├── experimentos_parametros_glcm.csv
    │   ├── logreg_glcm_metricas.csv / _resumo.csv, comparativo_baseline_vs_logreg.csv
    │   └── metadados_execucao.json             # sementes e versões das bibliotecas (Kaggle)
    └── marco3/                     # Marco 3 (25 arquivos)
        ├── features_todas_familias_g3.csv      # features das 3 famílias (corte central, T1wCE) — cache
        ├── features_multicorte_g3.csv          # 9 cortes centrais por paciente (ablação de agregação)
        ├── features_ablacao_nonorm_crop.csv / features_ablacao_norm_nocrop.csv
        ├── plano_t1wce_por_paciente.csv        # plano de aquisição (cabeçalho DICOM) — confundidor
        ├── baseline_trivial_prior_metricas.csv
        ├── grade_comparativa_descritor_modelo.csv  # tabela principal: 12 combinações, média ± desvio
        ├── grade_metricas_por_dobra.csv
        ├── selecao_aninhada_por_dobra.csv          # estimativa honesta (sem viés de seleção)
        ├── comparativo_baseline_vs_melhor.csv
        ├── incerteza_melhor_combinacao.csv         # IC bootstrap + teste de permutação
        ├── estudo_ablacao_preprocessamento.csv
        ├── estudo_ablacao_agregacao_cortes.csv
        ├── extra_selecao_features_combinado.csv
        ├── predicoes_oof_todas_combinacoes.csv / predicoes_oof_melhor_combinacao.csv
        ├── predicoes_com_plano.csv / erros_melhor_combinacao.csv  # análise de erro qualitativa
        ├── metadados_execucao.json
        └── grafico_auc_descritor_modelo.png, roc_melhor_combinacao.png, pr_melhor_combinacao.png,
            matriz_confusao_melhor_combinacao.png, permutacao_auc_nula.png,
            exemplos_erros_melhor_combinacao.png
```

## Dados

Os dados não estão neste repositório (política do próprio Kaggle e do enunciado do
TP1 — dados brutos não vão para o repositório).

Fonte: RSNA-MICCAI Brain Tumor Radiogenomic Classification (2021) — Kaggle
(https://www.rsna.org/artificial-intelligence/ai-image-challenge). É necessário
aceitar os termos da competição no Kaggle antes de acessar os dados.

A amostra usada no baseline (lista de `patient_id`, estratificada por `MGMT_value`,
semente fixa `SEED = 42`) está documentada em `outputs/eda/amostra_g3.csv`: **30
pacientes por classe (60 no total)**, sorteados do universo de 585 pacientes rotulados
(278 não metilados / 307 metilados).

A partição por paciente (`outputs/marco2/particao_pacientes_congelada.csv`,
`StratifiedGroupKFold`, 5 dobras) está **congelada desde o Marco 2** — não deve ser
recalculada nos marcos seguintes. Isso é reforçado no próprio código: o notebook do
Marco 2, se encontrar essa partição já salva, **reaproveita os folds em vez de
recalculá-los** (ver "Reprodutibilidade" abaixo, sobre por que isso é necessário).

## Como rodar

1. Acesse o Kaggle e crie um Notebook vinculado à competição
   `rsna-miccai-brain-tumor-radiogenomic-classification` (o dataset fica disponível em
   `/kaggle/input/competitions/rsna-miccai-brain-tumor-radiogenomic-classification`
   depois de adicionado em "Add Input").
2. Rode em sequência:
   - `01_eda.ipynb` (EDA + amostragem estratificada);
   - `02_preprocessing_baseline_G3.ipynb` (pipeline de pré-processamento, partição por
     paciente, baseline trivial, 1ª família de descritores — GLCM em T1wCE — e 1º
     classificador);
   - `03_feature_extraction_modeling_G3.ipynb` (3 famílias de descritores, grade de 12
     combinações descritor×modelo com validação cruzada aninhada, seleção aninhada,
     ablações, IC por bootstrap, teste de permutação, análise de sensibilidade por
     plano de aquisição).
3. Confirme, na primeira célula de cada notebook, que os dados são encontrados (a
   variável `DATA_DIR` é localizada automaticamente, com `RSNA_DATA_DIR` como opção de
   sobrescrita — não há caminho absoluto de máquina pessoal no código).
4. Se cada marco rodar em uma sessão separada do Kaggle (não a mesma sessão contínua
   desde o Marco 1), é preciso encadear os notebooks manualmente. Após executar cada
   marco:
   1. Clique em **Save Version**, com a opção **"Save & Run All (Commit)"** — isso
      anexa os arquivos de `outputs_*_g3/` gerados à versão salva.
   2. No notebook do marco seguinte, clique em **+ Add Input** e selecione a versão
      salva do marco anterior.
   3. Rode o próximo notebook normalmente.

   Isso é necessário porque arquivos "congelados" de um marco (`amostra_g3.csv`,
   `particao_pacientes_congelada.csv`, `features_todas_familias_g3.csv` etc.) só
   aparecem como Input do marco seguinte se a versão que os gerou tiver sido salva com
   os outputs anexados. Os notebooks já têm uma função (`find_file`/`load_or_build`)
   que procura esses arquivos tanto na pasta de trabalho local quanto em
   `/kaggle/input/...`, então não é necessário descobrir o caminho exato manualmente —
   só salvar a versão e anexar o Input correto antes de rodar.

**Nota sobre tempo de execução:** a extração das 3 famílias de descritores e a grade
de 12 combinações (Marco 3) são a parte mais demorada, por envolverem validação
cruzada aninhada (`GridSearchCV` em 3 dobras internas) para cada combinação. As
ablações de pré-processamento e de agregação de cortes, e o cálculo do plano de
aquisição, leem os DICOM apenas uma vez e ficam em cache (`features_ablacao_*.csv`,
`features_multicorte_g3.csv`, `plano_t1wce_por_paciente.csv`); reexecuções seguintes
reaproveitam esse cache (`USE_CACHE = True`, o padrão) e são rápidas. Para reproduzir
tudo do zero a partir só dos dados brutos, ajuste `USE_CACHE = False` na primeira
célula do Marco 3.

## Reprodutibilidade

- **Semente fixa:** `SEED = 42` (amostragem estratificada, partição por paciente,
  validação cruzada, bootstrap e teste de permutação).
- **Nenhum caminho absoluto pessoal é usado** — apenas busca automática de arquivos
  (`find_file`) e o caminho padrão de input do Kaggle (ou a variável de ambiente
  `RSNA_DATA_DIR`).
- **Partição congelada, não apenas com a mesma semente.** `StratifiedGroupKFold` com
  `shuffle=True` pode produzir uma divisão diferente entre versões do scikit-learn,
  mesmo com `random_state` idêntico (verificado neste projeto: a mesma semente, em
  outra versão da biblioteca, mudou a dobra de 45 dos 60 pacientes). Por isso o
  notebook do Marco 2, ao encontrar `particao_pacientes_congelada.csv` já salvo,
  **reaproveita os folds existentes em vez de recalculá-los** — só calcula do zero na
  primeiríssima execução do projeto.
- **Desbalanceamento de classes não é o obstáculo dominante aqui.** A distribuição é
  278/307 (47,5%/52,5%), aproximadamente balanceada — por isso não há pesos de classe
  nem reamostragem no pipeline; a dificuldade do desafio é a fraqueza intrínseca do
  sinal radiogenômico (ver `formulacao_problema.md`), não desbalanceamento.
  Mesmo assim, as métricas incluem AUC-ROC, AUC-PR, sensibilidade, especificidade, F1
  e acurácia balanceada — nunca só acurácia isolada.
- **Seleção de modelo com dois níveis, ambos reportados.** Hiperparâmetros: só por
  validação cruzada interna. Escolha de qual das 12 combinações é "a melhor": pelo
  teste externo (otimista, declarado como tal) — com a seleção aninhada reportada ao
  lado como a estimativa sem esse viés (ver Objetivo, acima, e "Principais resultados").
- Dependências declaradas em `requirements.txt`, com as versões usadas na execução
  real no Kaggle registradas em `outputs/marco2/metadados_execucao.json` e
  `outputs/marco3/metadados_execucao.json`.

## Status do projeto

- [x] Marco 1 — EDA, leitura DICOM com orientação por modalidade, amostragem
      estratificada, fichamento das 10 referências, formulação do problema
- [x] Marco 2 — pipeline de pré-processamento completo, partição por paciente
      congelada, baseline trivial, primeira família de descritores (GLCM em T1wCE) e
      primeiro classificador, experimentação de parâmetros do GLCM
- [x] Marco 3 — extração das 3 famílias de descritores, grade de experimentos
      descritor × modelo (LogReg, Random Forest, SVM) com validação cruzada aninhada,
      seleção aninhada sem viés, IC por bootstrap e teste de permutação, estudo de
      ablação (pré-processamento e agregação de cortes), análise de sensibilidade por
      plano de aquisição
- [x] Marco 4 — análise de erro qualitativa (casos específicos por paciente, matriz de
      confusão como figura), escrita final do artigo (template SBC), link do
      repositório no corpo do texto, tabela de contribuição individual (Anexo A)
- [ ] Confirmar, no PDF renderizado do template SBC, que o artigo não excede 4 páginas
- [ ] Teste de reprodutibilidade em ambiente limpo (Kaggle/Colab novo), seguindo
      apenas este README
- [ ] (opcional) Verificar a associação entre plano de aquisição e MGMT nos 585
      pacientes do dataset completo, não só na amostra de 60

## Principais resultados (grade completa com validação cruzada aninhada)

| Protocolo | AUC-ROC (méd ± dp) | AUC agrupada | Ac. balanceada |
|---|---|---|---|
| Baseline trivial | 0,500 ± 0,000 | — | 0,500 |
| Melhor combinação (GLCM + LogReg) — **otimista**, escolhida nas mesmas dobras de teste | 0,738 ± 0,113 | 0,624 (IC95% [0,484; 0,769]) | 0,645 ± 0,080 |
| **Seleção aninhada** — par escolhido só pela CV interna, sem olhar o teste | **0,542 ± 0,138** | **0,510** | 0,557 ± 0,058 |
| Só pacientes com T1wCE axial (n = 52) — controla o confundidor de plano | — | 0,523 (IC95% [0,359; 0,684]) | — |

Sob o protocolo honesto (seleção aninhada), o sinal é fraco e compatível com o acaso —
resultado coerente com Kim et al. (2022) e Doniselli et al. (2024), que documentam
desempenho próximo do acaso para este mesmo desafio sob validação rigorosa.

**Ablação:** combinar as 3 famílias de descritores nem sempre é o melhor caminho — o
conjunto Combinado (36 atributos) teve AUC pior que o GLCM isolado com regressão
logística (0,691 vs. 0,738), sinal de alta dimensionalidade para a amostra pequena
(`SelectKBest` recupera parte da perda, sem superar o GLCM isolado). O corte central
também superou o *pooling* de múltiplos cortes (3/5/9 cortes: 0,696/0,692/0,734, contra
0,738 do corte central) — ver `outputs/marco3/estudo_ablacao_agregacao_cortes.csv`.

**Análise de erro (Marco 3/4):** o plano de aquisição da T1wCE é um **confundidor**.
Dos 60 pacientes, os 8 com T1wCE não axial (3 coronais, 5 sagitais) são todos
metilados e respondem por 7 dos 17 verdadeiros positivos; restringir as predições aos
52 pacientes axiais derruba a AUC agrupada de 0,624 para 0,523. Casos específicos
(paciente 00538, falso positivo com P(metilado) = 0,784; paciente 00331, falso
negativo com P(metilado) = 0,179, entre outros) estão em
`outputs/marco3/erros_melhor_combinacao.csv` e ilustrados em
`exemplos_erros_melhor_combinacao.png`.

## Uso de IA generativa

Declarado conforme exigido pelo enunciado (§6): utilizado apoio de IA generativa
— **Claude (Anthropic)** para estruturação e revisão do código dos três marcos
(EDA, pipeline de pré-processamento, extração de descritores, grade de experimentos,
validação cruzada aninhada), depuração de vieses metodológicos (seleção de modelo
pelo conjunto de teste, risco de a partição mudar entre versões do scikit-learn),
cálculo de incerteza (bootstrap, teste de permutação), identificação e investigação do
confundidor de plano de aquisição, verificação de referências bibliográficas e apoio
na redação deste README, do fichamento e da formulação do problema; e **Codex
(OpenAI)** para apoio na redação, tradução, conferência e formatação em LaTeX do
artigo final — conforme declarado também na nota de uso de IA ao final do artigo. Em
todos os casos o grupo executou o código no Kaggle, conferiu as saídas e validou as
decisões e as referências antes de incorporá-las ao projeto; a IA não teve acesso aos
dados brutos DICOM, apenas ao código, aos CSVs e às figuras gerados pelo grupo.
