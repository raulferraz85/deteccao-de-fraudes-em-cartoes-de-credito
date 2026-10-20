Detecção de fraudes em cartões de crédito
Projeto educacional em Python que compara regressão logística, Random Forest e XGBoost em um problema com fraudes raras.
Problema
O dataset contém 284.807 transações e 492 fraudes (cerca de 0,17%). Responder sempre “normal” dá aproximadamente 99,83% de acurácia, mas recall de fraude igual a zero. Por isso, o projeto prioriza recall e acompanha precisão, F1, falsos positivos, falsos negativos e Average Precision (AP).
Como executar
1. Abra deteccao_fraudes.ipynb no Google Colab (Arquivo → Fazer upload de notebook).
2. Execute as células em ordem, incluindo a instalação de dependências.
3. Aguarde o treinamento. As saídas já salvas permitem inspecionar a execução entregue.
Para executar localmente, use Python 3.12, instale requirements.txt com python -m pip install -r requirements.txt e abra o notebook em Jupyter ou no VS Code com suporte a notebooks. O editor/Jupyter deve ser instalado separadamente. A primeira célula instala as bibliotecas sem fixar versões para facilitar o Colab; requirements.txt registra as versões usadas nesta entrega. Versões diferentes podem gerar pequenas diferenças de resultado.

O CSV é lido diretamente de https://storage.googleapis.com/download.tensorflow.org/data/creditcard.csv e não está incluído no repositório.
Preparação
- Mantém o DataFrame original e cria uma cópia tratada.
- Confere tipos, dados ausentes, valores, classes e duplicatas.
- Remove 1.081 duplicatas exatas, restando 283.726 linhas. Sem identificador, essa decisão não comprova que toda repetição seja erro; evita cópias idênticas entre conjuntos.
- Substitui Amount por log1p(Amount) e preserva Time e V1–V28.
- Divide os dados com estratificação em 60% treino, 20% validação e 20% teste.
- Ajusta imputação e, na regressão logística, StandardScaler exclusivamente no treino por meio de Pipeline.
- Trata desbalanceamento com pesos de classe. Reamostragem fica como extensão opcional.
Seleção de modelo e limiar
Cada modelo é treinado no treino. Na validação, selecionamos o limiar de maior recall com precisão mínima de 50%, preferindo precisão maior em empate. Se o piso for impossível, usamos maior F2 e registramos a exceção. O piso é didático: em produção seria definido conforme custo e capacidade de revisão de alertas.

A escolha do modelo usa a mesma regra na validação. O teste só é usado depois que modelo e limiares estão definidos; não se troca o vencedor usando o resultado de teste. Mostramos os três modelos com limiares padrão e ajustado para comparação. Escores de modelos com pesos de classe não são necessariamente probabilidades calibradas.
Arquivos
- deteccao_fraudes.ipynb: código, explicações, tabelas e gráficos executados.
- requirements.txt: versões utilizadas.
- resultados.csv: avaliação de teste nos dois limiares.
- validacao.csv: avaliação usada na seleção.
- resumo.json: resumo calculado pelo notebook.
Resultados reais da execução
Modelo	Limiar ajustado	Recall	Precisão	F1	FP	FN
Regressão logística	0.979828	80.00%	55.07%	0.6524	62	19
Random Forest	0.245969	78.95%	78.95%	0.7895	20	20
XGBoost	0.368799	82.11%	50.65%	0.6265	76	17


O XGBoost foi escolhido na validação: recall de 89,36% com precisão de 50,30%, contra recall de 88,30% dos demais. No teste, encontrou 78 das 95 fraudes, perdeu 17 e gerou 76 falsos alertas. Precisão de 50,65% significa que aproximadamente metade dos alertas era fraude.

O limiar escolhido foi 0,368799. Comparado a 0,5, ele encontrou uma fraude adicional no teste, mas produziu 27 falsos alertas adicionais (76 em vez de 49). Esse ganho pode não compensar operacionalmente: o objetivo didático priorizou recall com piso de precisão. O Random Forest teve F1 maior e menos falsos alertas; isso não invalida a regra prévia, mas mostra por que a decisão depende dos custos do negócio. Não alteramos a escolha depois de consultar o teste.

No SHAP global da amostra, V14, V4, V12, V11 e V8 tiveram as maiores contribuições absolutas médias. No exemplo local, uma fraude real, V14 (+3,31), V12 (+1,42) e V10 (+1,21) elevaram a margem de fraude, enquanto V8 (−0,75) a reduziu. Esses valores são contribuições ao escore do modelo, não causas comprovadas nem porcentagens.
Explicabilidade
O notebook explica o XGBoost como referência de árvores, independentemente do vencedor. Inclui importância interna, SHAP global em 300 exemplos aleatórios e waterfall local do exemplo com maior escore de fraude. Essa escolha local é intencional e não representa uma transação típica. Valores SHAP positivos aumentam a margem do modelo; negativos a reduzem. A escala é log-odds, não pontos percentuais. V1–V28 são componentes de PCA: não é possível traduzir uma delas diretamente como “renda”, “loja” ou “localização”. SHAP explica o modelo, não a causa real da fraude.
O que foi acrescentado ao roteiro
Com base no enunciado disponibilizado: divisão explícita de validação, cuidado com duplicatas, regra documentada de limiar com piso de precisão, comparação dos limiares e contagem de FP/FN. O notebook original completo da professora não foi disponibilizado, então não é possível declarar diferenças exatas em relação ao código dela.
Limitações e próximos passos
A divisão aleatória não representa previsão de transações futuras. Há poucas fraudes nos conjuntos de avaliação, e diferenças pequenas podem ser instáveis. Próximos passos: validação temporal, comparação de undersampling e oversampling aplicados somente ao treino, busca de hiperparâmetros usando validação cruzada, calibração e análise de custos. O projeto não foi validado para uso em produção.
Publicação e entrega
Crie um repositório público na sua conta do GitHub, com nome como deteccao-fraudes-cartao, e envie o conteúdo desta pasta (não o ZIP nem o CSV original). Confira se o notebook e o README aparecem. Entregue o link da página inicial do repositório. Esta entrega de arquivos não significa que um repositório já foi publicado.
Referências
- TensorFlow: dataset e classificação desbalanceada
- Scikit-learn: prevenção de vazamento de dados
- SHAP: TreeExplainer
- Enunciado fornecido nas imagens da atividade.
