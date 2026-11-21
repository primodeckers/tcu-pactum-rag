# TCU-Pactum-RAG — o que temos que fazer

Guia de trabalho do Projeto Final da disciplina de IA Generativa / LLMs (Prof. Marcelo Pita).
Resume as exigências da disciplina e o que a nossa proposta nos compromete a entregar.

Versão de referência da proposta: [Proposta_Projeto_RAG_TCU_v2.pdf](Proposta_Projeto_RAG_TCU_v2.pdf) (editável em [.docx](Proposta_Projeto_RAG_TCU_v2.docx)). A [versão original](Proposta_Projeto_RAG_TCU.pdf) fica no repositório só como histórico.

## 1. O projeto em uma frase

Construir um **benchmark aberto em português** com acórdãos do TCU sobre licitações e contratos e usá-lo para **comparar arquiteturas de RAG** — modelos abertos rodando em uma única GPU local contra modelos proprietários via API — medindo precisão jurídica, alucinação e custo. O mesmo pipeline vira um **assistente de consulta** demonstrável.

## 2. Entregas da disciplina

| Etapa | Entrega | Pontos | Data limite | Situação |
|---|---|---|---|---|
| 1 | Definição das equipes (3–4 pessoas) | 0 | 15/11/2026 | Concluída |
| 2 | Proposta de projeto (PDF ou Word) | 1,0 | **21/11/2026, 23:59** | v2 pronta; enviar na plataforma |
| 3 | Apresentação do projeto/sistema | 3,0 | 19/12/2026 | A fazer |
| 4 | Relatório de projeto (ensaio ou artigo) | 4,0 | 08/01/2027 | A fazer |
| 5 | Códigos e arquivos em GitHub público | 2,0 | 08/01/2027 | Repositório criado; código a fazer |

### Pendências da Etapa 2

- [ ] Confirmar com o grupo a divisão de papéis da seção 2 da proposta.
- [ ] Enviar o PDF da v2 na plataforma (tentativas ilimitadas, dá para reenviar).

## 3. O que mudou da proposta original para a v2

| Tema | Original | v2 |
|---|---|---|
| Equipe | Marcadores sem nome, 4 papéis | 3 integrantes nomeados, papéis redistribuídos |
| Sistema | Só benchmark e pipeline | Inclui assistente de consulta demonstrável |
| Golden Dataset | 150 a 250 pares | Mínimo de 100, com expansão até 250 |
| Recorte | Acórdãos "regidos pela Lei 14.133" | Licitações e contratos de 2021 a 2026, cobrindo a transição da Lei 8.666 para a 14.133 |
| Hardware | "Hardware da equipe", sem definição | RTX 4090 de 24 GB, com seção própria de infraestrutura |
| Modelos locais | 7B a 14B | 7B a 14B, mais um de 27B a 32B como teto |
| Juiz | Sem definição de onde roda | Via API |
| Experimentos | Matriz completa | Dois estágios: recuperação primeiro, geração depois |
| Validação por especialistas | Parte do escopo | Extensão opcional; a equipe revisa todos os pares |

## 4. O que a proposta promete entregar

1. **Assistente de consulta demonstrável** — pergunta em linguagem natural, resposta com os acórdãos citados; apresentado em funcionamento em 19/12.
2. **Benchmark TCU-Pactum-RAG** — no mínimo 100 pares pergunta–resposta revisados pela equipe (meta de até 250), com acórdãos de referência, publicado no Hugging Face.
3. **Pipeline reprodutível** — código aberto de ingestão, indexação, recuperação, geração e avaliação.
4. **Comparativo empírico** — qualidade, latência e custo das configurações, confrontando APIs com modelos abertos em uma RTX 4090.
5. **Taxonomia de alucinações jurídicas** — catálogo dos tipos de erro observados.
6. **Guia prático** — recomendações para adoção segura de RAG na Administração Pública.

Os itens 5 e 6 podem ser seções do relatório, sem virar documentos separados.

## 5. Infraestrutura

Toda a parte local roda na estação do Renê, com **GPU NVIDIA RTX 4090 de 24 GB**: embeddings, reranking e inferência dos LLMs abertos. Modelos proprietários e o LLM-juiz ficam via API e qualquer integrante pode rodá-los.

Regras de trabalho que decorrem disso:

- **Tudo por script.** Cada experimento recebe um arquivo de configuração, roda do começo ao fim sem supervisão e salva resultados parciais e finais.
- **Desenvolver com amostra.** Cada etapa é testada com algumas dezenas de acórdãos e perguntas, na própria máquina ou via API; a versão completa roda na GPU depois de subir para o GitHub.
- **Resultados voltam para o repositório**, junto com a configuração usada.
- **Janela de contexto explícita.** Definir `num_ctx` no Ollama, igual para todos os modelos locais; o padrão é pequeno e trunca o contexto recuperado sem avisar.
- **Teste-piloto antes das rodadas completas**, com cerca de 20 perguntas, para medir o tempo por modelo.
- **Plano B:** RTX 4090 alugada (RunPod ou Vast.ai), na faixa de US$ 0,30 a 0,90 por hora em novembro de 2026.

## 6. Plano de trabalho

### Fase 1 — Coleta e corpus

- [ ] Baixar os CSVs do Portal de Dados Abertos do TCU (seção Jurisprudência): acórdãos de inteiro teor, Jurisprudência Selecionada, súmulas e respostas a consultas.
- [ ] Obter o texto das Leis nº 14.133/2021 e nº 8.666/1993.
- [ ] Fechar o recorte: licitações e contratos, 2021–2026, decisões de mérito do Plenário e das Câmaras.
- [ ] Marcar cada acórdão com o regime legal que aplica (8.666, 14.133 ou ambos) e conferir uma amostra da marcação.
- [ ] Limpar e normalizar o texto; separar Relatório, Voto e Acórdão; preservar metadados (número/ano, colegiado, relator, data da sessão, tipo de processo, assunto).
- [ ] Confirmar os volumes reais contra a estimativa (3–5 mil acórdãos, 1–2 mil enunciados, 40–80 milhões de tokens).
- [ ] Versionar o corpus limpo.

### Fase 2 — Golden Dataset

- [ ] Gerar perguntas a partir dos enunciados da Jurisprudência Selecionada (LLM + revisão humana); o acórdão de origem é o gabarito de recuperação.
- [ ] Chegar ao mínimo de 100 pares revisados, respeitando a composição:

  | Tipo | O que testa | Participação |
  |---|---|---|
  | Busca direta | Enunciado em um único acórdão | ≈ 35% |
  | Síntese multi-hop | Cruzamento de dois ou mais acórdãos | ≈ 25% |
  | Evolução jurisprudencial | Mudança de entendimento, inclusive na transição da 8.666 para a 14.133 | ≈ 20% |
  | Fora de escopo | Pergunta sem respaldo; mede abstenção | ≈ 20% |

- [ ] Registrar para cada item: pergunta, resposta de referência, acórdãos de suporte, tipo de pergunta.
- [ ] Revisar todos os pares em equipe.
- [ ] **Plus, se houver tempo e gente:** validar uma amostra com 2–3 especialistas e calcular a concordância (kappa de Cohen ou Fleiss).
- [ ] Expandir até 250 pares se houver tempo.

### Fase 3 — Pipeline RAG e assistente

Cada etapa precisa ser intercambiável por configuração.

- [ ] **Chunking**: tamanho fixo com sobreposição, estrutural (seção e parágrafo numerado) e semântico. Todo chunk carrega número do acórdão, colegiado, data e seção.
- [ ] **Indexação**: embeddings densos (BGE-M3, multilingual-E5-large) em base vetorial (Qdrant ou FAISS) e índice lexical BM25.
- [ ] **Recuperação**, quatro variantes:
  - vetorial pura;
  - híbrida BM25 + densa com Reciprocal Rank Fusion;
  - híbrida com reranking por cross-encoder (bge-reranker-v2-m3);
  - filtros por metadados (data, colegiado) para perguntas de evolução.
- [ ] **Geração**: prompt com citação obrigatória do número do acórdão, uso exclusivo do contexto recuperado e abstenção explícita quando não houver respaldo.
- [ ] **Modelos geradores**:
  - abertos locais: Llama, Qwen e Gemma, 7B–14B, em 4 ou 8 bits, e um modelo de 27B–32B em 4 bits, via Ollama;
  - proprietários: um ou dois modelos de ponta (GPT, Claude ou Gemini) e o Sabiá.
- [ ] Fixar as versões exatas dos modelos antes de rodar os experimentos.
- [ ] **Assistente de consulta**: interface simples sobre o pipeline, para a demonstração.

### Fase 4 — Experimentos em dois estágios

- [ ] **Teste-piloto** na GPU com cerca de 20 perguntas, para medir tempos.
- [ ] **Estágio 1 — recuperação:** rodar todas as combinações de chunking, embeddings e recuperação e medir só Recall@K e MRR, sem gerar respostas.
- [ ] **Estágio 2 — geração:** fixar as uma ou duas melhores configurações de recuperação e comparar os modelos geradores.
- [ ] Calcular as métricas:

  | Dimensão | Métricas |
  |---|---|
  | Recuperação | Recall@K, MRR |
  | Geração | Fidelidade ao contexto, correção factual, exatidão da citação |
  | Segurança | Taxa de alucinação, taxa de abstenção correta (fora de escopo) |
  | Viabilidade | Latência por consulta; custo por mil consultas (API vs. aquisição e energia da RTX 4090) |
  | Confiabilidade | Conferência manual de uma amostra das notas do LLM-juiz |

- [ ] Configurar o LLM-as-a-Judge: modelo de maior porte, diferente dos avaliados, via API, com uma amostra das notas conferida pela equipe (RAGAS como apoio).
- [ ] Analisar os resultados por tipo de pergunta.
- [ ] Classificar os erros e montar a taxonomia de alucinações (número de acórdão inexistente, voto vencido tratado como decisão, entendimento superado apresentado como vigente etc.).

### Fase 5 — Entregas finais

- [ ] **Apresentação (19/12)**: slides e demonstração do assistente.
- [ ] **Relatório (08/01)**: ensaio ou artigo com método, resultados, taxonomia e guia de recomendações.
- [ ] **GitHub público (08/01)**: código, README com instruções de reprodução, configurações dos experimentos.
- [ ] **Hugging Face**: publicação do Golden Dataset.

## 7. Calendário sugerido

Sugestão de distribuição das fases nos prazos da disciplina; não consta da proposta e deve ser ajustada pelo grupo.

| Período | Foco |
|---|---|
| 21/11 | Envio da proposta |
| 22/11 – 29/11 | Fase 1: coleta, recorte, marcação do regime legal e limpeza |
| 30/11 – 06/12 | Fase 2 (primeira versão do Golden Dataset) e início da Fase 3 (chunking e indexação) |
| 07/12 – 13/12 | Pipeline completo, teste-piloto e estágio 1 |
| 14/12 – 18/12 | Estágio 2 parcial, assistente e preparação da apresentação |
| **19/12** | **Apresentação** |
| 20/12 – 03/01 | Estágio 2 completo, revisão do Golden Dataset, análise de erros |
| 04/01 – 07/01 | Redação final do relatório, limpeza do repositório, publicação do dataset |
| **08/01** | **Relatório e GitHub** |

Para 19/12, o mínimo demonstrável é o assistente funcionando de ponta a ponta, o estágio 1 concluído e resultados parciais do estágio 2. O que for rodar na GPU para a apresentação precisa estar com o Renê alguns dias antes.

## 8. Equipe e papéis

| Integrante | Responsabilidade |
|---|---|
| Alexandre Ferreira Ponte | Coordenação, desenho do benchmark e curadoria do Golden Dataset |
| Elisangela Osório | Coleta, limpeza e chunking do corpus; avaliação (métricas e LLM-as-a-Judge) e análise de erros |
| Renê Estevam Deckers | Pipeline de recuperação (indexação, busca híbrida, reranking), geração com LLMs e execução dos experimentos na GPU |

O assistente de consulta ainda não tem responsável definido.

## 9. Riscos e como lidar

| Risco | Mitigação |
|---|---|
| Sem validação externa | A validação por especialistas é extensão opcional; o gabarito parte dos enunciados do TCU e a equipe revisa todos os pares |
| Infraestrutura concentrada em uma única GPU | Execução em lote com salvamento parcial; modelos limitados a 24 GB; GPU em nuvem como contingência |
| Viés do LLM-juiz | Conferir manualmente uma amostra das notas; juiz diferente dos modelos avaliados |
| LGPD e ética | Só decisões públicas; acórdãos sigilosos não estão na base aberta |
| Prazo curto até a apresentação | Priorizar o assistente e o estágio 1; reduzir o número de modelos antes de reduzir a qualidade do Golden Dataset |
| Carga concentrada em uma pessoa | Recuperação, geração e GPU estão com o Renê; redistribuir parte da geração ou do assistente |

## 10. Decisões em aberto

- [ ] Confirmação da divisão de papéis pelo grupo.
- [ ] Responsável pelo assistente de consulta e tecnologia da interface.
- [ ] Qdrant ou FAISS.
- [ ] LangChain ou LlamaIndex.
- [ ] Quais modelos abertos e proprietários, e em que versões.
- [ ] Qual modelo será o juiz.
- [ ] Se haverá validação por especialistas (plus) e quem seriam.
- [ ] Formato do relatório: ensaio ou artigo.
- [ ] Data interna de corte para enviar experimentos à GPU antes de 19/12.
