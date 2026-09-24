# Fichamento — G3 (RSNA 2021 Brain Tumor MGMT Methylation)

Baseado na lista padronizada de 10 referências do grupo
(`Referencias_MGMT_padronizado.docx`), organizada em 3 eixos temáticos:
1) Fundamento clínico, 2) Métodos/CpG (como a metilação é medida), 3) Imagem
(prever o marcador por MRI). Ao final, a seção "Referências metodológicas"
lista os trabalhos clássicos dos descritores usados no Marco 3.

> **Aviso de integridade (TP1, §6 e §9):** referência inexistente ou citada sem
> leitura zera o critério de embasamento. Cada fichamento abaixo deve ser
> conferido pelo grupo contra o artigo original (DOI na lista padronizada) antes
> de qualquer citação. Números específicos só devem ir ao artigo depois de
> conferidos no texto original.

---

## Eixo 1 — Fundamento clínico

### 1. Hegi et al. (2005) — *N Engl J Med*
Ensaio clínico randomizado que mostrou que apenas pacientes com promotor
MGMT metilado obtiveram ganho de sobrevida com temozolomida. É o estudo que
justifica, clinicamente, por que prever o status de MGMT importa.
**Relação com o TP1:** Introdução, como justificativa clínica da predição de MGMT
(contexto clínico exigido no §6 do TP1).

### 2. Louis et al. (2021) — *Neuro-Oncology* (Classificação OMS 2021)
Resumo da 5ª edição da classificação da OMS para tumores do SNC, que
consolidou marcadores moleculares (IDH, TERT etc.) como critério diagnóstico.
**Relação com o TP1:** referência de terminologia e definições ao descrever o
tipo de tumor do desafio.

---

## Eixo 2 — Métodos de medição / sítios CpG

### 3. Brandner et al. (2021) — *Neuro-Oncology* (meta-análise Cochrane)
Compara técnicas de medição de metilação (pirosequenciamento, MSP,
imuno-histoquímica) quanto ao poder prognóstico. Conclui que
pirosequenciamento e MSP superam IHC, e **não há consenso sobre pontos de
corte nem sobre quais sítios CpG usar**.
**Relação com o TP1:** Limitações — o próprio rótulo (`MGMT_value`) carrega
incerteza metodológica de origem, pois diferentes instituições do BraTS podem ter
usado protocolos e cortes distintos. Parte do "ruído" pode estar no rótulo, e não
só na imagem.

### 4. Gibson et al. (2024) — *BMC Neurology* (revisão sistemática)
Reúne quais sítios CpG do promotor têm associação significativa com sobrevida e
expressão de MGMT. Complementa Brandner et al. na lacuna de quais sítios importam.
**Relação com o TP1:** reforça a discussão sobre incerteza na variável-alvo
(Metodologia ou Limitações); não motiva decisão técnica de pré-processamento.

### 5. Teske et al. (2022) — *J Neuro-Oncol* (coorte retrospectiva)
Compara extensão e padrão de sítios CpG metilados entre glioblastoma e
astrocitoma IDH-selvagem/TERT-mutado. Mais sítios metilados associou-se a
desfecho mais favorável nos dois grupos.
**Relação com o TP1:** base para dizer, em Limitações, que o grupo não controla
subtipo molecular (IDH, TERT) ao tratar MGMT isoladamente.

### 6. Poon et al. (2021) — *Neuro-Oncology Advances* (coorte regional, n = 414)
Trata a metilação como variável **quantitativa** em vez de binária, e mostra que
o ponto de corte binário pode esconder informação prognóstica relevante.
**Relação com o TP1:** ressalva na Metodologia — o dataset fornece `MGMT_value`
binário (0/1), o que descarta informação de grau. Limitação herdada do dataset.

---

## Eixo 3 — Predição por imagem (radiogenômica)

### 7. Choi et al. (2021) — *AJNR* (correlação DWI/DSC-PWI)
Correlaciona mudanças no status de metilação (tumor inicial vs. recorrente) com
parâmetros de difusão e perfusão por MRI, sem aprendizado de máquina.
**Relação com o TP1:** exemplo de abordagem por imagem convencional, útil como
contraste em Trabalhos Relacionados (a relação MGMT × imagem já era estudada antes
da radiômica/ML).

### 8. Yogananda et al. (2021) — *AJNR* (deep learning, T2 apenas)
Rede neural 3D treinada só com T2 para classificar o status de metilação, citada
na literatura com acurácia alta em validação interna. *(Conferir no artigo original
os valores exatos antes de citar números.)*
**Relação com o TP1:** não aplicável como método (o TP1 proíbe deep learning), mas
é o contraponto histórico que Kim et al. (2022) desmonta com validação externa —
ilustra o problema de generalização em Trabalhos Relacionados.

### 9. Kim et al. (2022) — *Cancers* (BraTS 2021 radiogenomics challenge)
**Artigo mais diretamente relevante ao G3:** validação externa em larga escala
(420 experimentos, 2 centros) dos modelos submetidos ao desafio BraTS 2021 de
radiogenômica — o desafio que o G3 resolve. Conclusão: a maioria dos modelos não
se distinguiu do acaso; os autores concluem que o status de MGMT pode não ser
previsível por MRI pré-operatória, mesmo com deep learning.
**Relação com o TP1:** calibra a expectativa de desempenho do baseline. Entra na
Introdução (propósito do baseline) e na Conclusão (limitações). **Coerência com o
Marco 3:** nossa estimativa sem viés de seleção (AUC 0,54) e o resultado restrito a
exames axiais (AUC agrupado 0,52) são compatíveis com essa conclusão.

### 10. Doniselli et al. (2024) — *European Radiology* (revisão + meta-análise)
Avalia a qualidade metodológica de estudos de radiômica para MGMT com as escalas
RQS e TRIPOD. Aderência geralmente baixa às diretrizes; estudos **com validação
externa tiveram desempenho significativamente menor** que os sem validação externa.
**Relação com o TP1:** embasa a Metodologia (partição por paciente, validação
cruzada aninhada, ausência de vazamento) e explica por que o G3 reporta o AUC
agrupado, o IC e a seleção aninhada, e não só o melhor par.

---

## Síntese — como os 10 artigos se conectam ao TP1

1. **Fundamento (1–2):** justificam clinicamente a predição de MGMT e fixam a
   terminologia — Introdução.
2. **Medição do rótulo (3–6):** o próprio "gabarito" carrega incerteza (técnicas
   diferentes, sem consenso de corte, binário descarta informação) — outra
   explicação, além da fraqueza do sinal de imagem, para o teto de desempenho.
3. **Imagem (7–10):** de correlação simples (Choi) a deep learning com acurácia
   aparentemente alta (Yogananda), à sua desmontagem em validação externa no
   desafio do G3 (Kim) e à confirmação sistemática de que a validação externa
   reduz o desempenho na área (Doniselli).
4. **Achado mais acionável:** Kim et al. (2022) e Doniselli et al. (2024) justificam
   a meta realista de desempenho e o rigor do protocolo (§4.4 do TP1).

---

## Referências metodológicas (Marco 3) — **a ler antes de citar**

Trabalhos clássicos dos descritores e do protocolo usados no Marco 3. Estão aqui para
sustentar as decisões metodológicas (o TP1 exige que cada decisão não trivial seja
citada), mas **não contam como lidas até que o grupo as leia e confira**.

| Uso no projeto | Referência | Lida? |
|---|---|---|
| Textura GLCM / descritores de Haralick | HARALICK, R. M.; SHANMUGAM, K.; DINSTEIN, I. Textural features for image classification. *IEEE Transactions on Systems, Man, and Cybernetics*, SMC-3(6):610-621, 1973. | [ ] |
| Momentos invariantes de Hu (forma) | HU, M.-K. Visual pattern recognition by moment invariants. *IRE Transactions on Information Theory*, 8(2):179-187, 1962. | [ ] |
| Histograma de orientação de gradiente | DALAL, N.; TRIGGS, B. Histograms of oriented gradients for human detection. *IEEE CVPR*, 2005, p. 886-893. | [ ] |
| Limiarização de Otsu (região de tecido) | OTSU, N. A threshold selection method from gray-level histograms. *IEEE Transactions on Systems, Man, and Cybernetics*, 9(1):62-66, 1979. | [ ] |
| Detector de bordas de Canny | CANNY, J. A computational approach to edge detection. *IEEE Transactions on Pattern Analysis and Machine Intelligence*, PAMI-8(6):679-698, 1986. | [ ] |
| Viés de seleção com validação cruzada (justifica CV aninhada) | VARMA, S.; SIMON, R. Bias in error estimation when using cross-validation for model selection. *BMC Bioinformatics*, 7:91, 2006. | [ ] |

**Lacunas a preencher com leitura (sem referência ainda):**
- Normalização de intensidade em RM (justificativa da normalização por percentil):
  buscar um trabalho de radiômica ou de padronização de intensidade em RM, ler e
  conferir antes de citar.
- Variabilidade de aquisição e plano de corte como confundidor em radiômica
  (Discussão do achado de plano de aquisição).

**Contagem para o requisito do TP1 (§6.7):** mínimo de 10 referências, sendo 6
revisadas por pares. As 10 referências de MGMT são todas de periódicos revisados
por pares; as metodológicas acima acrescentam mais referências, se lidas.

---

## Notas de atualização

### Marco 2
A escolha da modalidade T1wCE em vez de T1w foi feita por **experimentação do
próprio grupo**, não por citação direta dos 10 artigos — nenhum compara T1w × T1wCE
isoladamente. Amparada pelo §4.1 do TP1 ("evidência empírica do próprio grupo") e
apresentada no artigo como tal.

### Marco 3
- **Sem literatura direta para os resultados do G3:** as conclusões (seleção aninhada
  AUC 0,54; sinal desaparece nos exames axiais) são achados do grupo; a literatura fichada
  (Kim; Doniselli) serve para contextualizá-los, não para prová-los.
- **Confundidor de plano de aquisição:** nenhum dos 10 artigos trata desse ponto
  especificamente. É achado do grupo, a discutir em Limitações/Análise de erro, com
  referência de apoio a ser buscada e lida (ver lacunas acima).
- **Uso de IA:** o rascunho deste fichamento contou com apoio de IA generativa
  (declarado no `README.md`); o grupo deve conferir cada resumo contra o artigo original.
