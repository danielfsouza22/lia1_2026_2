# Tarefa 2 — Prever o esquecimento: uma rede Keras contra a curva que o meu painel usa hoje

**Autor:** Daniel · **Disciplina:** Tópicos em Engenharia de Computação 1 (EMC0467)

Projeto sobre o **Painel PRF 2027**, o sistema que o senhor aprovou na 1ª entrega.

---

## O que abrir

`Tarefa2_Prever_Esquecimento_Keras.ipynb` — está com as saídas salvas, dá para ler inteiro sem
executar. Se quiser rodar: **abre e roda**, não pede upload nem GPU. O notebook baixa sozinho o
pedaço do dataset público que usa, e a rede treina em menos de um minuto na CPU.

## Em uma frase

Meu painel decide qual cartão eu reviso primeiro a partir de uma curva de esquecimento que **eu
escrevi à mão, sem nunca ter medido**. Esta entrega põe essa curva à prova contra uma rede Keras
treinada em 12,9 milhões de revisões reais — e termina perguntando se o modelo melhor muda alguma
decisão.

## Dataset

**Duolingo Spaced Repetition Data** — Settles & Meeder, *A Trainable Spaced Repetition Model for
Language Learning*, ACL 2016. Público, no Harvard Dataverse. 12,9 milhões de revisões de 115 mil
pessoas. Escolhido porque mede exatamente a grandeza que o painel precisa prever.

## Os quatro resultados

1. **A métrica óbvia premiou o modelo inútil.** No MAE, "responda sempre que vai lembrar" ganhou de
   todos (0,099) — inclusive da rede treinada. É o modelo que nunca aponta cartão nenhum. Sem linha
   de base, esse número teria passado como bom.

2. **A métrica certa saiu do produto, não da estatística.** O painel não precisa saber *se* eu
   lembro; precisa *ordenar* cartões. Trocando MAE por AUC o placar inverte:

   | Modelo | MAE | RMSE | AUC |
   |---|---|---|---|
   | Sempre 1,0 (otimista) | **0,099** | 0,283 | — |
   | Constante = média | 0,173 | 0,265 | — |
   | Curva do painel (hoje) | 0,118 | 0,284 | 0,548 |
   | Rede Keras | 0,166 | **0,263** | **0,600** |

3. **A curva que eu uso hoje está perto de uma moeda** (AUC 0,548). No corte que o painel de fato
   usa, ela encontra 23,8% das revisões em que eu realmente esqueci; a rede encontra 29,2%.

4. **A rede é melhor e mesmo assim não entra em produção.** Rodada sobre os meus 628 cartões reais,
   ela troca **14 dos 20** cartões da rodada — e a AUC foi medida em revisões de idioma, não nas
   minhas de Legislação de Trânsito. Trocar sem medir no domínio certo seria repetir, ao contrário,
   o erro da sua reflexão de 10/09.

**O que entra no painel:** gravar a previsão antes da resposta. Três campos a mais por revisão, e
em um mês a comparação deixa de depender de dado emprestado.

## O que tem na pasta

| Arquivo | O que é |
|---|---|
| `Tarefa2_Prever_Esquecimento_Keras.ipynb` | a entrega |
| `cartoes.csv` | os 628 cartões reais do painel, usados no teste de produto (também embutidos no notebook) |
| `extra/` | resposta escrita à reflexão de 10/09, medindo as duas versões do painel com o dado dele. Fora do escopo da Tarefa 2 — está aqui só como complemento |

## Técnicas aplicadas

Keras 3 / TensorFlow: rede densa 5→64→32→1 com dropout e saída sigmoid, `EarlyStopping`, curvas de
treino e validação, curva de calibração em decis, ROC/AUC e matriz de confusão no ponto de corte de
operação. Divisão treino/teste **por pessoa** (não por linha) e normalização com estatística só do
treino, para evitar vazamento; `session_correct` descartada pelo mesmo motivo.
