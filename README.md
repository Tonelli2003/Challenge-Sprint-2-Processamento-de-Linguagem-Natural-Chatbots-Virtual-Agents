# 🏭 Sprint 2 — Visualização Operacional e Representação do Ativo
### FIAP × Forzy | Processamento de Linguagem Natural, Chatbots & Virtual Agents

---

## 👥 Grupo

| Nome | RM |
|---|---|
| Augusto Oliveira Codo de Sousa | RM562080 |
| Felipe de Oliveira Cabral | RM561720 |
| Gabriel Tonelli Avelino Dos Santos | RM564705 |
| Vinícius Adrian Siqueira de Oliveira | RM564962 |
| Sofia Bueris Netto de Souza | RM565818 |

---

## 📋 Sobre o Projeto

Solução de **Digital Twin** para monitoramento e manutenção preditiva de motores elétricos industriais. Este repositório contém a **Sprint 2**, que adiciona sobre os artefatos da Sprint 1:

- **Representação vetorial** dos ativos do Digital Twin via *sentence-transformers*
- **Módulo de busca semântica** com normalização terminológica integrada
- **Geração de linguagem natural (NLG)** para enriquecimento da interface operacional

---

## 🗂️ Estrutura do Repositório

```
sprint2/
│
├── sprint2_pln_consulta.ipynb   ← Notebook principal (execute este)
│
├── inputs/                      ← Artefatos do glossário (Sprint 1)
│   ├── glossario_tecnico_motores_v5.csv
│   └── glossario_tecnico_motores_v5.json
│
├── data/                        ← Corpus textual (Sprint 1)
│   ├── corpus_consolidado_v1.csv
│   ├── corpus_motores_v1.json
│   └── corpus_motores_v1.txt
│
├── tabelas/                     ← Tabelas de suporte (Sprint 1)
│   ├── tabela_normalizacao.csv      ← 790 variantes → termos canônicos
│   └── stopwords_tecnicas.txt
│
├── relatorios/                  ← Gerado em runtime (não versionado)
│   ├── corpus_stats_sprint2.json
│   ├── metricas_avaliacao_sprint2.csv
│   ├── templates_nlg_sprint2.md
│   ├── relatorio_MT-042.txt
│   ├── relatorio_MT-088.txt
│   └── ...
│
├── .gitignore
└── README.md
```

> **Nota:** A pasta `relatorios/` é gerada automaticamente ao executar o notebook. Seus arquivos estão no `.gitignore` por serem outputs de runtime.

---

## 🚀 Como Executar

### Opção 1 — Google Colab (recomendado)

```bash
# 1. Clone o repositório no Colab
!git clone https://github.com/SEU_USER/SEU_REPO.git

# 2. Abra o notebook
# File > Open > sprint2/sprint2_pln_consulta.ipynb

# 3. Execute todas as células em ordem (Runtime > Run all)
```

A **Célula 0** detecta automaticamente o ambiente e configura os caminhos. Nenhuma edição manual é necessária.

### Opção 2 — Execução Local

```bash
# Pré-requisitos: Python 3.10+

# 1. Clone o repositório
git clone https://github.com/SEU_USER/SEU_REPO.git
cd sprint2

# 2. Instale as dependências
pip install sentence-transformers faiss-cpu pandas numpy jupyter

# 3. Abra o notebook
jupyter notebook sprint2_pln_consulta.ipynb

# 4. Execute todas as células em ordem
```

---

## 🧰 Dependências

| Pacote | Versão mínima | Papel |
|---|---|---|
| `sentence-transformers` | 2.2.0 | Geração de embeddings semânticos |
| `faiss-cpu` | 1.7.0 | Indexação e busca por similaridade vetorial |
| `pandas` | 1.5.0 | Manipulação do corpus e dos artefatos |
| `numpy` | 1.23.0 | Operações matriciais e conversão float32 |

Instalação automática na **Célula 1** do notebook via `%pip install`.

---

## 📦 Entregáveis da Sprint 2

### Entregável 1 — Representação Vetorial dos Ativos

**Modelo escolhido:** `paraphrase-multilingual-MiniLM-L12-v2`

| Critério | Justificativa |
|---|---|
| Bilinguismo PT/EN | O corpus tem distribuição 75% PT / 25% EN (321/107 docs). O modelo multilingual (50+ idiomas) representa ambos no mesmo espaço vetorial sem pipeline paralelo |
| Domínio técnico | Fine-tuned para similaridade semântica de frases — captura siglas (RPM, IP55, IEC 60034), termos compostos e variantes abreviadas |
| Eficiência | 384 dimensões, ~117M parâmetros. Executa em CPU no Colab em < 2s para o corpus atual |
| Cobertura | Vocabulário lematizado: 2.914 tipos (Sprint 1). Cobertura glossário v5: 100% (107/107 termos) |

**Arquitetura de indexação:** `FAISS IndexFlatL2` — busca exata por distância Euclidiana, determinístico e auditável. Para escala > 100k vetores, migrar para `IndexIVFFlat`.

---

### Entregável 2 — Módulo de Consulta Semântica

**Pipeline de recuperação:**

```
query bruta (jargão do operador)
      |
      v  normalizar_query() — tabela_normalizacao.csv (790 variantes, Sprint 1)
          backoff: trigrama -> bigrama -> unigrama
query normalizada (termos canônicos)
      |
      v  SentenceTransformer.encode()
vetor float32 [1 x 384]
      |
      v  faiss.IndexFlatL2.search(top_k)
[(tag, equipamento, distância_L2, score_similaridade), ...]
```

**Avaliação — 30 consultas-teste em 6 categorias de dificuldade:**

| Categoria | Nível | Foco |
|---|---|---|
| A | Fácil | Localização coloquial de área/setor |
| B | Médio | Especificação elétrica/mecânica com variações numéricas |
| C | Médio | Variante terminológica EN → PT via `tabela_normalizacao.csv` |
| D | Difícil | Siglas técnicas, acrônimos e códigos de falha (DOL, VSD, MCSA, oc fault) |
| E | Difícil | Múltiplos atributos potencialmente conflitantes |
| F | Muito Difícil | Code-switching PT + EN misturado na mesma query |

**Métricas calculadas:** Precisão@2 (P), Recall@2 (R), F1, MRR (Mean Reciprocal Rank)

O arquivo `relatorios/metricas_avaliacao_sprint2.csv` contém os resultados completos por consulta após a execução do notebook.

---

### Entregável 3 — Enriquecimento Textual (NLG)

Três templates de linguagem natural documentados em `relatorios/templates_nlg_sprint2.md`:

**Template 1 — Relatório Operacional Completo**
Destino: OS de manutenção, notificação ao gestor.
```
[STATUS] RELATÓRIO OPERACIONAL AUTOMÁTICO | Ativo: {TAG}
O equipamento {NOME}, localizado em '{LOCAL}', encontra-se {ESTADO}.
Última leitura: {V}V · {A}A · {RPM}RPM · {°C}°C [· {VIB} mm/s]
Diretriz: {DIRETRIZ}
```

**Template 2 — Linha de Dashboard**
Destino: HMI / SCADA / interface web em tempo real.
```
{TAG} | {ÍCONE} {STATUS} | {V}V · {A}A · {RPM}RPM · {°C}°C
```

**Template 3 — Resposta Conversacional**
Destino: Chatbot / assistente virtual integrado ao Digital Twin.
```
Ativo identificado: {TAG} (similaridade: {SCORE:.3f})
-> {NOME} | {LOCAL}
-> Estado: {ÍCONE} {STATUS} | {leituras}
-> Diretriz: {DIRETRIZ}
```

**Regras de composição por status:**

| Status | Ícone | Diretriz |
|---|---|---|
| Normal | ✅ | Nenhuma ação corretiva necessária. |
| Alarme de Temperatura | 🚨 | Abrir OS e realizar inspeção imediata. |
| Sobrecorrente | ⚠️ | Verificar carga acoplada e condições de acionamento. |
| Vibração Elevada | ⚠️ | Agendar análise espectral de vibração. |
| Offline | ⚫ | Verificar conectividade IIoT. |

---

## 🔗 Integração com a Sprint 1

| Artefato Sprint 1 | Localização | Papel na Sprint 2 |
|---|---|---|
| `tabela_normalizacao.csv` | `tabelas/` | Backbone do pipeline de normalização da query (790 variantes) |
| `glossario_tecnico_motores_v5.csv` | `inputs/` | Referência vocabular para design das 30 consultas-teste |
| `glossario_tecnico_motores_v5.json` | `inputs/` | Alternativa estruturada para leitura programática |
| `corpus_consolidado_v1.csv` | `data/` | Base textual; distribuição PT/EN justifica escolha do modelo |
| `corpus_stats.json` | `data/` | Métricas de vocabulário (2.914 tipos, cobertura 100%) usadas na justificativa |
| `stopwords_tecnicas.txt` | `tabelas/` | Referência de termos a preservar no pré-processamento |

---

## 📊 Resultados Esperados

Após executar todas as células, a pasta `relatorios/` conterá:

| Arquivo | Conteúdo |
|---|---|
| `corpus_stats_sprint2.json` | Metadados do índice FAISS (total vetores, dimensão, modelo, data) |
| `metricas_avaliacao_sprint2.csv` | Resultados das 30 consultas-teste com P@2, R@2, F1, MRR por linha |
| `templates_nlg_sprint2.md` | Documentação completa dos templates e regras de composição |
| `relatorio_MT-042.txt` | Relatório operacional gerado para o ativo MT-042 |
| `relatorio_MT-088.txt` | Relatório operacional gerado para o ativo MT-088 (alarme) |
| `relatorio_MT-205.txt` | Relatório operacional gerado para o ativo MT-205 (vibração) |
| `relatorio_MT-102.txt` | Relatório para MT-102 (offline — sem telemetria) |

---

## 📚 Referências

- Reimers, N. & Gurevych, I. (2019). *Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks*. EMNLP 2019.
- Johnson, J. et al. (2019). *Billion-scale similarity search with GPUs*. IEEE TPAMI. [[FAISS]](https://github.com/facebookresearch/faiss)
- IEC 60034 — *Rotating Electrical Machines*
- NBR 17094 — *Máquinas Elétricas Girantes — Motores de Indução*
- ISO 20816-3 — *Mechanical vibration — Measurement and evaluation of machine vibration*

---

*Sprint 2 | PLN Forzy — FIAP 2026*
