# JurisTCU-RAG — o que temos que fazer

Guia de trabalho do Projeto Final da disciplina de IA Generativa / LLMs (Prof. Marcelo Pita).
Resume as exigências da disciplina e o que a nossa proposta ([Proposta_Projeto_RAG_TCU.pdf](Proposta_Projeto_RAG_TCU.pdf)) nos compromete a entregar.

## 1. O projeto em uma frase

Construir um **benchmark aberto em português** com acórdãos do TCU sobre licitações e contratos (Lei nº 14.133/2021) e usá-lo para **comparar arquiteturas de RAG** — em especial modelos abertos locais contra modelos proprietários via API — medindo precisão jurídica, alucinação e custo.

## 2. Entregas da disciplina

| Etapa | Entrega | Pontos | Data limite | Situação |
|---|---|---|---|---|
| 1 | Definição das equipes (3–4 pessoas) | 0 | 15/11/2026 | Prazo encerrado |
| 2 | Proposta de projeto (PDF ou Word) | 1,0 | **21/11/2026, 23:59** | PDF pronto, falta enviar |
| 3 | Apresentação do projeto/sistema | 3,0 | 19/12/2026 | A fazer |
| 4 | Relatório de projeto (ensaio ou artigo) | 4,0 | 08/01/2027 | A fazer |
| 5 | Códigos e arquivos em GitHub público | 2,0 | 08/01/2027 | A fazer |

### Pendências imediatas da Etapa 2

- [ ] Substituir os marcadores `[Nome do integrante N]` na seção 2 do PDF pelos nomes reais.
- [ ] Conferir que o PDF tem os 5 elementos exigidos: título, membros, contexto e objetivo, descrição dos dados, abordagem técnica (todos já constam).
- [ ] Enviar a tarefa na plataforma até 23:59 de 21/11/2026 (tentativas ilimitadas, dá para reenviar).

## 3. O que a proposta promete entregar

1. **Benchmark JurisTCU-RAG** — 150 a 250 pares pergunta–resposta validados, com acórdãos de referência, publicado no Hugging Face.
2. **Pipeline reprodutível** — código aberto de ingestão, indexação, recuperação, geração e avaliação (GitHub).
3. **Comparativo empírico** — tabela de qualidade, latência e custo das configurações testadas.
4. **Taxonomia de alucinações jurídicas** — catálogo dos tipos de erro observados.
5. **Guia prático** — recomendações para adoção segura de RAG na Administração Pública.

## 4. Plano de trabalho

### Fase 1 — Coleta e corpus

- [ ] Baixar os CSVs do Portal de Dados Abertos do TCU (seção Jurisprudência): acórdãos de inteiro teor, Jurisprudência Selecionada, súmulas e respostas a consultas.
- [ ] Obter o texto da Lei nº 14.133/2021.
- [ ] Fechar o recorte: licitações e contratos sob a Lei 14.133, 2021–2026, decisões de mérito do Plenário e das Câmaras.
- [ ] Limpar e normalizar o texto; separar Relatório, Voto e Acórdão; preservar metadados (número/ano, colegiado, relator, data da sessão, tipo de processo, assunto).
- [ ] Confirmar os volumes reais contra a estimativa (3–5 mil acórdãos, 1–2 mil enunciados, 40–80 milhões de tokens).
- [ ] Versionar o corpus limpo.

### Fase 2 — Golden Dataset

- [ ] Gerar perguntas a partir dos enunciados da Jurisprudência Selecionada (LLM + revisão humana); o acórdão de origem é o gabarito de recuperação.
- [ ] Respeitar a composição prevista:

  | Tipo | O que testa | Participação |
  |---|---|---|
  | Busca direta | Enunciado em um único acórdão | ≈ 35% |
  | Síntese multi-hop | Cruzamento de dois ou mais acórdãos | ≈ 25% |
  | Evolução jurisprudencial | Mudança de entendimento; acórdão mais recente prevalece | ≈ 20% |
  | Fora de escopo | Pergunta sem respaldo; mede abstenção | ≈ 20% |

- [ ] Registrar para cada item: pergunta, resposta de referência, acórdãos de suporte, tipo de pergunta.
- [ ] Validar uma amostra com 2–3 especialistas e calcular a concordância (kappa de Cohen ou Fleiss).

### Fase 3 — Pipeline RAG

Cada etapa precisa ser intercambiável, para compor a matriz experimental.

- [ ] **Chunking**: tamanho fixo com sobreposição, estrutural (seção e parágrafo numerado) e semântico. Todo chunk carrega número do acórdão, colegiado, data e seção.
- [ ] **Indexação**: embeddings densos (BGE-M3, multilingual-E5-large) em base vetorial (Qdrant ou FAISS) e índice lexical BM25.
- [ ] **Recuperação**, quatro variantes:
  - vetorial pura;
  - híbrida BM25 + densa com Reciprocal Rank Fusion;
  - híbrida com reranking por cross-encoder (bge-reranker-v2-m3);
  - filtros por metadados (data, colegiado) para perguntas de evolução jurisprudencial.
- [ ] **Geração**: prompt com citação obrigatória do número do acórdão, uso exclusivo do contexto recuperado e abstenção explícita quando não houver respaldo.
- [ ] **Modelos geradores**:
  - abertos locais: famílias Llama, Qwen e Gemma, 7B–14B, quantizados (4/8 bits), via Ollama ou vLLM;
  - proprietários: um ou dois modelos de ponta (GPT, Claude ou Gemini) e o Sabiá.
- [ ] Fixar as versões exatas dos modelos antes de rodar os experimentos.
- [ ] Decidir a orquestração (LangChain ou LlamaIndex).

### Fase 4 — Avaliação e experimentos

- [ ] Rodar a matriz chunking × recuperação × modelo gerador sobre o mesmo Golden Dataset.
- [ ] Calcular as métricas:

  | Dimensão | Métricas |
  |---|---|
  | Recuperação | Recall@K, MRR |
  | Geração | Fidelidade ao contexto, correção factual, exatidão da citação |
  | Segurança | Taxa de alucinação, taxa de abstenção correta (fora de escopo) |
  | Viabilidade | Latência por consulta, custo por mil consultas (API vs. local) |
  | Confiabilidade | Concordância entre LLM-juiz e avaliadores humanos |

- [ ] Configurar o LLM-as-a-Judge: modelo de maior porte, diferente dos avaliados, calibrado com rótulos humanos (RAGAS como apoio).
- [ ] Analisar os resultados por tipo de pergunta, para localizar onde cada arquitetura falha.
- [ ] Classificar os erros e montar a taxonomia de alucinações (número de acórdão inexistente, voto vencido tratado como decisão, entendimento superado apresentado como vigente etc.).

### Fase 5 — Entregas finais

- [ ] **Apresentação (19/12)**: slides e demonstração do sistema.
- [ ] **Relatório (08/01)**: ensaio ou artigo com método, resultados, taxonomia e guia de recomendações.
- [ ] **GitHub público (08/01)**: código, README com instruções de reprodução, configurações dos experimentos.
- [ ] **Hugging Face**: publicação do Golden Dataset.

## 5. Calendário sugerido

Sugestão de distribuição das fases nos prazos da disciplina; ajustar conforme a disponibilidade da equipe.

| Período | Foco |
|---|---|
| 21/11 | Envio da proposta |
| 22/11 – 29/11 | Fase 1: coleta, recorte e limpeza do corpus |
| 30/11 – 06/12 | Fase 2 (primeira versão do Golden Dataset) e início da Fase 3 (chunking e indexação) |
| 07/12 – 13/12 | Fase 3 completa e primeira rodada de experimentos |
| 14/12 – 18/12 | Resultados preliminares e preparação da apresentação |
| **19/12** | **Apresentação** |
| 20/12 – 03/01 | Matriz completa, validação humana, análise de erros |
| 04/01 – 07/01 | Redação final do relatório, limpeza do repositório, publicação do dataset |
| **08/01** | **Relatório e GitHub** |

São só quatro semanas até a apresentação. Para 19/12 o mínimo demonstrável é um pipeline de ponta a ponta com um subconjunto do Golden Dataset e resultados parciais; a matriz completa pode ficar para o relatório.

## 6. Divisão de papéis

Conforme a proposta (nomes a preencher):

| Integrante | Responsabilidade |
|---|---|
| 1 | Coordenação, desenho do benchmark e curadoria do Golden Dataset |
| 2 | Coleta, limpeza e chunking do corpus |
| 3 | Pipeline de recuperação (indexação vetorial, busca híbrida, reranking) |
| 4 (opcional) | Geração com LLMs, avaliação (métricas e LLM-as-a-Judge) e análise de erros |

## 7. Riscos e como lidar

| Risco | Mitigação |
|---|---|
| Falta de especialistas para validar | Reduzir a amostra de validação humana e usar o enunciado da Jurisprudência Selecionada como gabarito |
| Hardware limitado | Modelos quantizados de até 14B; GPU em nuvem acadêmica se necessário |
| Viés do LLM-juiz | Calibrar com rótulos humanos; juiz diferente dos modelos avaliados |
| LGPD e ética | Só decisões públicas; acórdãos sigilosos não estão na base aberta |
| Prazo curto até a apresentação | Reduzir a matriz (menos modelos e estratégias de chunking) antes de reduzir a qualidade do Golden Dataset |

## 8. Decisões em aberto

- [ ] Nomes e papéis definitivos da equipe.
- [ ] Qdrant ou FAISS.
- [ ] LangChain ou LlamaIndex.
- [ ] Quais modelos abertos e proprietários, e em que versões.
- [ ] Qual modelo será o juiz.
- [ ] Hardware disponível para os modelos locais.
- [ ] Quem serão os especialistas validadores.
- [ ] Formato do relatório: ensaio ou artigo.
