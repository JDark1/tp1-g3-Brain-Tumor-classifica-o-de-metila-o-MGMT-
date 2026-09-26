# Formulação e solução do problema — G3
### RSNA 2021 — Brain Tumor (classificação de metilação MGMT)

## Formulação do problema

Dado um exame de ressonância magnética multimodal (FLAIR, T1w, T1wCE, T2w) de um
paciente com glioma, o objetivo é classificar, a partir de características
extraídas das imagens, se o status de metilação do promotor MGMT do tumor é
positivo ou negativo. A **unidade de amostra é o paciente** e a saída é uma
classificação binária (0 = não metilado, 1 = metilado). Nesta solução baseline,
cada paciente é representado por um vetor de descritores manuais extraído de uma
modalidade (T1wCE) e do corte central da série.

## Contexto clínico

A metilação do promotor MGMT é o principal fator preditivo de resposta à
temozolomida em glioblastoma: no ensaio clínico randomizado de referência da
área, apenas pacientes com promotor metilado obtiveram ganho de sobrevida com o
tratamento (Hegi et al., 2005). Determinar esse status influencia diretamente a
decisão terapêutica. Hoje essa determinação é invasiva (biópsia + análise
molecular), o que motiva o interesse em prevê-la de forma não invasiva a partir
de imagem (radiogenômica) — o problema deste TP1.

## Por que este é um problema de sinal fraco (embasamento na literatura)

A dificuldade deste desafio não está (apenas) na engenharia de características,
mas em limitações estruturais do problema, documentadas na literatura:

1. **O próprio rótulo carrega incerteza.** Não há consenso sobre qual técnica de
   medição (pirosequenciamento, MSP, imuno-histoquímica), nem sobre quais sítios
   CpG ou ponto de corte usar para definir "metilado" (Brandner et al., 2021;
   Gibson et al., 2024). Instituições diferentes que compuseram o dataset BraTS
   podem ter usado protocolos distintos.
2. **Reduzir a metilação a um binário descarta informação.** A extensão da
   metilação, tratada como variável contínua, tem valor prognóstico que se perde
   ao dicotomizar (Poon et al., 2021) — limitação herdada do desenho do dataset.
3. **A validação externa no próprio desafio mostrou desempenho próximo do acaso.**
   Kim et al. (2022) validaram externamente, em larga escala (420 experimentos, 2
   centros), os modelos do desafio BraTS 2021 de radiogenômica e concluíram que a
   maioria não se distinguiu do acaso, mesmo com deep learning.
4. **O padrão da área é desempenho inflado por validação inadequada.** Doniselli et
   al. (2024) mostraram que estudos de radiômica para MGMT com validação externa
   têm desempenho sistematicamente menor que os sem validação externa.

**Consequência prática:** o baseline é avaliado com expectativa realista, e o
rigor do protocolo (partição por paciente, validação cruzada aninhada, desvio-padrão
reportado, estimativas sem viés de seleção) importa tanto quanto o desempenho
bruto, justamente onde a maioria dos estudos da área falha.

## Decisões metodológicas

- **Amostragem:** estratificada por classe, 30 pacientes por classe (60 no total),
  semente fixa = 42 (`outputs/eda/amostra_g3.csv`).
- **Partição e validação:** partição por paciente com `StratifiedGroupKFold` (5
  dobras), definida e **congelada no Marco 2** (`outputs/marco2/particao_pacientes_congelada.csv`).
  Hiperparâmetros ajustados por validação cruzada aninhada (3 dobras internas), com
  padronização dentro do `Pipeline`. Relato de média ± desvio-padrão entre dobras,
  AUC agrupado (fora-da-dobra) e IC 95% por bootstrap.
- **Rótulo binário aceito como dado do desafio**, com a ressalva de que a informação
  de grau se perde (Poon et al., 2021); registrada como limitação.
- **Baseline trivial:** classe majoritária (`DummyClassifier`). Como a amostra é
  pequena e quase balanceada, a acurácia dele fica abaixo de 0,5 (0,40) por artefato
  das dobras; o critério de comparação é a acurácia balanceada (0,50) e o AUC (0,50).
- **Modalidade: T1wCE (não T1w).** Decisão do Marco 2 por evidência empírica do grupo
  (grade modalidade × distância GLCM × níveis de cinza, na partição congelada): T1wCE
  superou T1w em AUC nas 4 combinações (melhor T1wCE: 0,751 ± 0,114 com 32 níveis;
  T1w: 0,584 ± 0,219). Nenhum dos 10 artigos fichados compara T1w × T1wCE
  isoladamente; a escolha é do grupo e não deve ser atribuída à literatura.
  *Ressalva:* a escolha foi feita nas mesmas dobras usadas na avaliação (seleção
  otimista). Com 16 níveis de cinza o AUC foi maior (0,782), mas com desvio maior
  (0,133); mantivemos 32 níveis por estabilidade.
- **Parâmetros do GLCM:** distância 1 pixel, 32 níveis de cinza, 4 ângulos, média e
  desvio (12 features).
- **Estratégia de agregação: corte central por paciente.** No Marco 3, a ablação
  testou *pooling* (média) de 3, 5 e 9 cortes centrais e média + desvio de 5 cortes:
  AUC 0,696 / 0,692 / 0,734 / 0,650, contra 0,738 do corte central. Sem ganho, mantivemos
  o corte central (decisão sustentada por evidência do grupo).
- **Registro entre modalidades: não realizado.** A EDA (Marco 1) mostrou que as séries
  de um mesmo paciente são armazenadas em planos diferentes (axial, sagital, coronal).
  Como a solução usa uma única modalidade por paciente, o registro entre modalidades
  não é necessário; fica como limitação e como trabalho futuro para abordagens
  multimodais.
- **Sem máscara do tumor.** O dataset da tarefa não traz segmentação; os descritores de
  forma são calculados sobre a maior região de tecido após limiarização de Otsu (contorno
  do cérebro no corte), não do tumor. Limitação registrada.
- **Sem deep learning.** Todos os descritores são manuais (GLCM, momentos de Hu e
  propriedades de região, histograma de gradiente Sobel e densidade de bordas Canny) e os
  modelos são clássicos.

## Observações da EDA (Marco 1)

- Distribuição de classes: 278 (47,5%) não metilado vs. 307 (52,5%) metilado —
  aproximadamente balanceado. A dificuldade dominante não é desbalanceamento, e sim a
  fraqueza intrínseca do sinal.
- As 4 modalidades do mesmo paciente aparecem em planos de aquisição diferentes,
  identificados por `ImageOrientationPatient`.
- Grande variação no número de cortes por modalidade entre pacientes.

## Resultados do Marco 3 (experimento congelado)

Grade de 12 combinações (4 conjuntos de descritores × 3 modelos), AUC médio ± desvio entre
as 5 dobras externas:

| Descritor | LogReg | Random Forest | SVM (RBF) |
|---|---|---|---|
| GLCM | **0,738 ± 0,113** | 0,574 ± 0,151 | 0,460 ± 0,232 |
| Forma | 0,610 ± 0,083 | 0,568 ± 0,073 | 0,504 ± 0,124 |
| Gradiente | 0,598 ± 0,128 | 0,625 ± 0,095 | 0,552 ± 0,103 |
| Combinado | 0,691 ± 0,077 | 0,621 ± 0,090 | 0,396 ± 0,131 |

- **Melhor combinação (GLCM + regressão logística):** AUC 0,738 ± 0,113 por dobra;
  AUC agrupado 0,624 (IC 95% 0,48–0,77); AUC-PR agrupado 0,695; acurácia balanceada
  0,645; matriz de confusão: 19 VN, 11 FP, 13 FN, 17 VP. Teste de permutação (200
  permutações, só essa combinação): p ≈ 0,005.
- **Dois níveis de seleção — só o segundo tem viés.** (1) *Hiperparâmetros dentro de cada
  uma das 12 combinações:* escolhidos estritamente pela validação cruzada interna
  (`GridSearchCV`, 3 dobras, só no treino de cada dobra externa); nunca olham o teste.
  (2) *Qual das 12 combinações é "a melhor":* foi escolhida olhando o `auc_mean`
  calculado nas próprias dobras de teste externas — isso É seleção de modelo pelo
  teste (erro nº 4 do §10 do TP1), e por isso o AUC 0,738 do GLCM + LogReg é otimista.
- **Seleção aninhada do par (corrige o nível 2, sem viés de seleção):** em cada dobra
  externa, o par descritor×modelo é escolhido só pelo `score_interno` (nota da CV
  interna no treino), sem olhar o teste daquela dobra — só depois esse par é aplicado
  ao teste. Resultado: AUC 0,542 ± 0,138 (agrupado 0,510). O par escolhido mudou entre
  as dobras. **Este é o número a reportar como desempenho do procedimento**, ao lado do
  0,738 (rotulado como escolhido nas mesmas dobras) e do 0,523 (só exames axiais,
  abaixo).
- **Baseline trivial:** AUC 0,50; acurácia balanceada 0,50.
- **Modelos:** o SVM ficou abaixo do acaso em duas combinações (0,396 e 0,460), sinal de
  sobreajuste com amostra pequena. No conjunto Combinado (36 features para ~48 pacientes por
  dobra) a regressão logística piorou em relação ao GLCM isolado; `SelectKBest` recuperou
  parte (AUC 0,696), sem superar o GLCM.
- **Ablação de pré-processamento (GLCM + LogReg):** completo 0,738; sem normalização por
  percentil 0,643; sem recorte 0,693. Diferenças da ordem do desvio entre dobras: indício,
  não efeito comprovado.

## Achado: o plano de aquisição é um confundidor

O plano da T1wCE varia entre pacientes (52 axiais, 3 coronais, 5 sagitais na amostra).
**Todos os 30 pacientes não metilados são axiais**; os 8 não axiais são todos metilados
(qui-quadrado plano × rótulo, p ≈ 0,01; se 8 pacientes fossem sorteados ao acaso, a
probabilidade de caírem todos na classe metilada seria ≈ 0,002). O modelo atribui P(metilado)
média de 0,91 aos sagitais e 0,70 aos coronais, e 7 dos 17 verdadeiros positivos são exames
não axiais. Restrito aos 52 exames axiais, o AUC agrupado do melhor par cai de 0,624 para
**0,523** (IC 95% 0,36–0,68). Interpretação: parte do desempenho aparente reflete a
aquisição (e não a biologia do tumor). Se a associação é do dataset inteiro ou acaso da
amostra de 60 pacientes não foi verificado; ler o plano da T1wCE dos 585 pacientes
responderia isso.

## Limitações

- Amostra pequena (60 pacientes; dobras de 12): intervalos de confiança largos.
- Seleção de modalidade, parâmetros do GLCM e do par descritor × modelo nas mesmas dobras
  (otimista); a seleção aninhada corrige a última.
- Descritores sobre o corte inteiro, sem máscara do tumor; o corte central pode não conter
  a região relevante.
- Confundidor de plano de aquisição (acima) e possível heterogeneidade de aquisição entre
  centros (brilho e contraste variam entre pacientes).
- Incerteza do rótulo e ausência de controle de subtipo molecular (Brandner et al., 2021;
  Poon et al., 2021; Teske et al., 2022).

## Status dos marcos

**Marco 2 (concluído):**
- [x] Pipeline de pré-processamento; partição por paciente congelada; baseline trivial;
      1ª família de descritores + 1º classificador; experimentação de parâmetros (T1wCE)

**Marco 3 (concluído):**
- [x] 3 famílias de descritores (textura, forma, gradiente)
- [x] 3 modelos clássicos + baseline; grade descritor × modelo; validação cruzada aninhada
- [x] Ablação de pré-processamento e de agregação de cortes
- [x] Seleção aninhada, IC por bootstrap, teste de permutação, análise por plano
- [x] Decisão sobre agregação (corte central) e registro (não necessário com uma modalidade)

**Marco 2 — reexecução corrigida:** notebook sem caminho pessoal, com a partição
reaproveitada explicitamente do arquivo já congelado (não recalculada — importante
porque `StratifiedGroupKFold` pode gerar dobras diferentes entre versões do
scikit-learn mesmo com a mesma semente). CSVs de resultado regenerados com T1wCE.
*Conferir se os números batem com os já citados acima (0,751 ± 0,114); se divergirem,
atualizar este documento e o artigo com os números atuais.*

**Pendências do Marco 4:**
- [ ] Análise de erro qualitativa (`outputs/marco3/erros_melhor_combinacao.csv`,
      `exemplos_erros_melhor_combinacao.png`): casos difíceis e hipóteses de causa
- [ ] (opcional) Verificar a associação plano × MGMT nos 585 pacientes
- [ ] Redigir Resultados, Conclusão, resumo e abstract; ajustar às 4 páginas do template SBC
- [ ] Referências metodológicas lidas e conferidas (ver `fichamento.md`)
- [ ] Regenerar os CSVs do Marco 2 com T1wCE; testar reprodutibilidade em ambiente limpo
- [ ] Declaração de uso de IA no artigo; tabela de contribuição individual (Anexo A)

## Referências citadas

1. HEGI, M. E. et al. MGMT gene silencing and benefit from temozolomide in
   glioblastoma. *N Engl J Med*, 352(10):997-1003, 2005.
2. LOUIS, D. N. et al. The 2021 WHO Classification of Tumors of the Central
   Nervous System: a summary. *Neuro-Oncology*, 23(8):1231-1251, 2021.
3. BRANDNER, S. et al. MGMT promoter methylation testing to predict overall
   survival in people with glioblastoma treated with temozolomide: a
   comprehensive meta-analysis based on a Cochrane Systematic Review.
   *Neuro-Oncology*, 23(9):1457-1469, 2021.
4. GIBSON, D. et al. A systematic review of high impact CpG sites and
   regions for MGMT methylation in glioblastoma. *BMC Neurology*,
   24(1):103, 2024.
5. TESKE, N. et al. Extent, pattern, and prognostic value of MGMT promotor
   methylation: does it differ between glioblastoma and
   IDH-wildtype/TERT-mutated astrocytoma? *J Neuro-Oncol*, 156(2):317-327,
   2022.
6. POON, M. T. C. et al. Extent of MGMT promoter methylation modifies the
   effect of temozolomide on overall survival in patients with
   glioblastoma: a regional cohort study. *Neuro-Oncology Advances*,
   3(1):vdab171, 2021.
7. CHOI, H. J. et al. MGMT promoter methylation status in initial and
   recurrent glioblastoma: correlation study with DWI and DSC PWI features.
   *AJNR*, 42(5):853-860, 2021.
8. YOGANANDA, C. G. B. et al. MRI-based deep-learning method for
   determining glioma MGMT promoter methylation status. *AJNR*,
   42(5):845-852, 2021.
9. KIM, B.-H. et al. Validation of MRI-based models to predict MGMT
   promoter methylation in gliomas: BraTS 2021 radiogenomics challenge.
   *Cancers*, 14(19):4827, 2022.
10. DONISELLI, F. M. et al. Quality assessment of the MRI-radiomics studies
    for MGMT promoter methylation prediction in glioma: a systematic review
    and meta-analysis. *European Radiology*, 34(9):5802-5815, 2024.

*As referências metodológicas (Haralick, Hu, Dalal & Triggs, Otsu, Canny, Varma & Simon)
estão em `fichamento.md` e só devem ser citadas no artigo depois de lidas pelo grupo.*
