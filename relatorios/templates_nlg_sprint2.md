# Templates NLG — Sprint 2 | PLN Forzy
## FIAP x Forzy | Processamento de Linguagem Natural
Data: 2026-06-20

---

## Visão Geral da Arquitetura NLG

A geração de linguagem natural (NLG) da Sprint 2 segue uma arquitetura
**Template-Based com lógica condicional**, que cruza:

- **Dados estáticos** (Digital Twin): cadastro do ativo (TAG, equipamento, especificação, localização)
- **Dados dinâmicos** (Telemetria IIoT): leituras de sensores (V, A, RPM, °C, status)

---

## Template 1: Relatório Operacional Completo

**Destino:** Documento de manutenção, notificação ao operador, ordem de serviço.

```
{ÍCONE} RELATÓRIO OPERACIONAL AUTOMÁTICO | Ativo: {TAG}
{LINHA_SEPARADORA}
O equipamento {NOME_EQUIPAMENTO}, localizado em '{LOCALIZAÇÃO}',
encontra-se atualmente {ESTADO_OPERACIONAL}.

Última leitura dos sensores industriais:
  • Tensão       : {TENSÃO} V
  • Corrente     : {CORRENTE} A
  • Rotação      : {RPM} RPM
  • Temp. Carcaça: {TEMPERATURA} °C

Diretriz: {DIRETRIZ}
```

### Regras de Composição — Campo {ESTADO_OPERACIONAL}

| Status Sensor | {ÍCONE} | {ESTADO_OPERACIONAL} | {DIRETRIZ} |
|---|---|---|---|
| Normal | ✅ | operando dentro dos parâmetros normais estabelecidos | Nenhuma ação corretiva é necessária no momento. |
| Alarme de Temperatura | 🚨 | apresentando um desvio operacional crítico (Alarme de Temperatura) | Recomenda-se a abertura de uma ordem de serviço e inspeção imediata. |
| Sobrecorrente | ⚠️ | apresentando sobrecorrente acima do valor nominal | Verificar carga acoplada, couplings e condições de acionamento. |
| Offline | ⚫ | — | Verificar conectividade do sensor IIoT. |

---

## Template 2: Linha de Dashboard (Formato Compacto)

**Destino:** Painéis de supervisão HMI / SCADA / interface web do Digital Twin.

```
{TAG} | {EMOJI} {STATUS_LABEL} | {TENSÃO}V · {CORRENTE}A · {RPM}RPM · {TEMPERATURA}°C
```

**Exemplos de saída:**
- `MT-042 | ✅ Normal                 |  379V ·  14.5A · 1745RPM · 45°C`
- `MT-088 | 🔴 Alarme de Temperatura  |  440V ·  85.0A · 3450RPM · 92°C`
- `MT-102 | ⚫ OFFLINE               | Sem telemetria`

**Princípio de design:** substituir dados brutos por linguagem contextualizada.
O operador não precisa memorizar limiares — o sistema traduz o estado para uma
leitura imediata.

---

## Template 3: Resposta Conversacional (NLP → NLG)

**Destino:** Interface de chatbot / assistente virtual do Digital Twin.

```
Ativo identificado: {TAG} (similaridade: {SCORE})
→ {NOME_EQUIPAMENTO} | {LOCALIZAÇÃO}
→ Estado atual: {ÍCONE} {STATUS} | {TENSÃO}V · {CORRENTE}A · {RPM}RPM · {TEMPERATURA}°C
→ Diretriz: {DIRETRIZ}
```

**Exemplo de uso:**
- Operador: "motor da área 3"
- Sistema: "Ativo identificado: MT-042 (similaridade: 0.812)
  → Motor de Indução Trifásico WEG W22 | Área 3 - Bomba de Resfriamento Principal
  → Estado atual: ✅ Normal | 379V · 14.5A · 1745RPM · 45°C
  → Diretriz: Nenhuma ação corretiva é necessária no momento."

---

## Integração com a Sprint 1

| Artefato Sprint 1 | Uso na Sprint 2 |
|---|---|
| `glossario_tecnico_motores_v5.csv` | Vocabulário técnico de referência para avaliação |
| `tabela_normalizacao.csv` (790 variantes) | Pipeline de normalização terminológica da query |
| `corpus_consolidado_v1.csv` | Base textual para validação do corpus |
| `corpus_stats.json` | Métricas de vocabulário usadas para justificar o modelo |
| `stopwords_tecnicas.txt` | Referência de termos a preservar no pré-processamento |

---

*Gerado automaticamente pelo pipeline Sprint 2 | PLN Forzy*
