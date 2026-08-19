# ARBITRAJUS | Visão estratégica e roadmap técnico

**Versão:** 2026-06-v1  
**Autor:** Fundação ARBITRAJUS  
**Status:** Documento vivo (produto + negócio + integração institucional)

---

## 1. Propósito

A ARBITRAJUS existe para **democratizar a resolução de conflitos em escala nacional**: conciliação, mediação e arbitragem 100% digitais, com operação enxuta, plataforma automatizada e corpo técnico credenciado.

Diferente do modelo boutique (curadoria manual, equipe grande), a ARBITRAJUS aposta em:

- **Volume** (massa, PME, consumer, contratos)
- **Automação** (motor de resolução, prazos, notificações)
- **Parceiros externos** (mediadores, conciliadores, árbitros)
- **Integração futura** com fóruns e tribunais (justiça multiportas)

> *Somos movidos por propósito. Os números abaixo são estimativas para planejamento, não promessas.*

---

## 2. Posicionamento vs mercado

| Player | Modelo | Equipe interna (ref.) | Foco |
|--------|--------|----------------------|------|
| **Arbtrato** | Câmara digital boutique | ~28 + ~159 externos | Causas patrimoniais, ticket alto |
| **Jusfy** | Legaltech SaaS | Centenas (empresa) | Ferramentas para advogados |
| **ARBITRAJUS** | Infraestrutura ADR massa | 1,5 + automação + rede | Volume, self-service, integração TJ |

**UX / produto:** a vitrine institucional da ARBITRAJUS já compete visualmente com referências do setor. Próximas iterações: completar fluxos internos, painel do procedimento e integrações.

---

## 3. Motor de resolução (núcleo técnico)

O **motor de resolução** é o orquestrador que roda procedimentos sem intervenção humana na rotina. Não substitui mediador/árbitro na decisão; **automatiza operação**.

### 3.1 Módulos

| Módulo | Responsabilidade | Automação |
|--------|------------------|-----------|
| **M1. Triagem** | Classifica conflito (tipo, área, valor) | Regras + templates por vertical |
| **M2. Cadastro** | Coleta partes, descrição, documentos | Formulário guiado (wizard) |
| **M3. Protocolo** | Gera ARB/MED/CON-AAAA-NNNN | Automático na abertura |
| **M4. Pagamento** | Taxa plataforma + repasse honorários | Gateway (Asaas/Stripe) |
| **M5. Notificação** | E-mail/WhatsApp às partes | Resend + templates |
| **M6. Prazos (SLA)** | Lembretes, suspensão, arquivo | Cron + regras por tipo |
| **M7. Designação** | Roteia para mediador/árbitro credenciado | Área + disponibilidade |
| **M8. Sessões** | Agenda videoconferência | Integração Daily/Zoom |
| **M9. Documentos** | Upload, hash, registro de atos | Storage Supabase |
| **M10. Encerramento** | Termo, arquivo, NPS | Fluxo automático |
| **M11. Exceções** | Fila suporte (e-mail) | Escalação humana |
| **M12. Métricas** | Taxa acordo, tempo médio, CAC | Dashboard admin |

### 3.2 Máquina de estados (fluxo v1)

```text
rascunho
    │ submit
    ▼
pagamento_pendente
    │ confirmar_pagamento
    ▼
aguardando_partes
    │ partes_confirmadas
    ▼
em_andamento
    │ concluir          │ suspender
    ▼                   ▼
concluido            suspenso → em_andamento | arquivado
    │
    └── arquivado (timeout / desistência)
```

**Implementado na plataforma (v1):** API `/api/procedimentos`, wizard `abrir-procedimento.html`, painel `caso.html`, timeline de eventos.

### 3.3 Treinamento do sistema (escala, estilo Jusfy)

| Fase | O que o sistema aprende |
|------|-------------------------|
| Piloto | Templates de acordo por vertical, SLAs reais |
| Escala | Roteamento por taxa de sucesso do mediador |
| Maturidade | Previsão de acordo, sugestão de tipo de procedimento |
| Institucional | Padrões exigidos por convênio TJ/OAB |

IA no suporte interno: triagem L1, FAQ, rascunho de comunicações. **Nunca** decide mérito do conflito.

---

## 4. API para tribunais (visão + v1 stub)

Integração para CEJUSCs, justiça multiportas e convênios estaduais.

### 4.1 Endpoints planejados

| Método | Rota | Descrição |
|--------|------|-----------|
| GET | `/api/tribunais/health` | Status da integração |
| POST | `/api/tribunais/encaminhar` | TJ encaminha demanda para mediação/conciliação |
| GET | `/api/tribunais/procedimentos/:protocolo` | Consulta status (read-only) |
| POST | `/api/tribunais/retorno` | Devolve termo/acordo ao fórum |
| GET | `/api/tribunais/metricas` | Relatório agregado (LGPD) |

**Autenticação:** `X-Tribunal-Key` + IP allowlist + certificado mTLS (fase 2).

**v1 implementada:** health + encaminhar (stub) em `backend/src/routes/tribunais.js`.

### 4.2 Fluxo institucional

```text
Parte distribui / CEJUSC encaminha
        ↓
API tribunal → ARBITRAJUS (encaminhar)
        ↓
Motor abre procedimento + notifica partes
        ↓
Mediação/conciliação na plataforma
        ↓
API retorno → TJ (termo, status, métricas)
        ↓
Split de receita (convênio)
```

---

## 5. Modelo de split de receita

### 5.1 Mercado privado (fase 1,2)

| Componente | Destinatário | Faixa sugerida |
|------------|--------------|----------------|
| Taxa de administração (plataforma) | ARBITRAJUS | 100% |
| Honorários do mediador/árbitro | Profissional credenciado | 70,85% |
| Comissão de intermediação | ARBITRAJUS | 15,30% sobre honorários |
| Gateway pagamento | Asaas/Stripe | ~2,4% (custo) |

**Exemplo mediação (causa R$ 50k):**

| Item | Valor ref. |
|------|------------|
| Taxa plataforma | R$ 565 , R$ 1.200 |
| Honorários mediador | R$ 800 , R$ 2.500 |
| ARBITRAJUS (taxa + comissão) | R$ 700 , R$ 1.800/caso |

### 5.2 Parceria B2B (escritório, associação, fintech)

| Modelo | Split |
|--------|-------|
| Indicação | 10,20% da taxa plataforma por caso |
| White label leve | Fee mensal R$ 2k,15k + R$ X/caso |
| API embed | R$ 0,50,2,00 por transação + mínimo mensal |

### 5.3 Parceria estadual (TJ / CEJUSC / OAB), fase 3+

| Componente | Split sugerido (negociável) |
|------------|----------------------------|
| Taxa paga pela parte | 100% |
| **Tribunal / fundo CEJUSC** | 15,25% |
| **ARBITRAJUS** (tech + operação) | 40,55% |
| **Profissional** (mediador/conciliador) | 25,40% |

**Premissa:** valor menor que litígio pleno; tribunal ganha redução de fila sem investir em desenvolvimento.

### 5.4 Planos SaaS (opcional)

| Plano | Público | Preço ref. |
|-------|---------|------------|
| Básico | PF / micro | R$ 99/mês + taxa/caso |
| Profissional | Advogado / PME | R$ 299/mês + casos inclusos |
| Empresa | Volume | R$ 999+/mês custom |

---

## 6. Estimativa de faturamento por escala

Premissas: taxa média plataforma **R$ 1.000/caso** (mix conciliação/mediação/arbitragem leve) + **15% comissão** sobre honorários médios **R$ 1.500** = **~R$ 225** → **~R$ 1.225 receita bruta ARBITRAJUS/caso**.

| Escala | Casos/ano | Receita bruta anual (est.) | Equipe interna | Observação |
|--------|-----------|----------------------------|----------------|------------|
| **Piloto** | 200,500 | R$ 250k , R$ 600k | 1 (fundador) | Validação fluxo |
| **Tração** | 1.000,3.000 | R$ 1,2M , R$ 3,7M | 1,3 | B2B inicial |
| **Escala** | 5.000,10.000 | R$ 6M , R$ 12M | 3,8 + terceirizado | Competitivo vs Arbtrato |
| **Liderança massa** | 15.000,30.000 | R$ 18M , R$ 37M | 8,15 + rede | + convênios estaduais |
| **Infraestrutura nacional** | 50.000+ | R$ 60M+ | Estrutura govtech | Integração multi-TJ |

**Custos fixos enxutos (escala inicial):** R$ 3k,15k/mês (infra, Resend, Supabase, VPS) + suporte terceirizado sob demanda.

**Margem bruta alvo:** > 60% (software + procedimento digital).

---

## 7. Roadmap técnico por fases

### Fase 0 | Concluído / em curso
- [x] Site institucional
- [x] Credenciamento + outreach LGPD
- [x] E-mail transacional (Resend)
- [x] Motor de resolução v1 + wizard abertura
- [x] API tribunais (stub)

### Fase 1 | 0,6 meses (engrenagem massa)
- [ ] Pagamento real (Asaas)
- [ ] Supabase produção + RLS
- [ ] Designação automática de mediador
- [ ] Notificações e-mail transacionais por etapa
- [ ] 500 casos reais piloto
- [ ] 30,50 profissionais credenciados

### Fase 2 | 6,18 meses (B2B + dados)
- [ ] API parceiros B2B
- [ ] White label leve
- [ ] Métricas e relatórios
- [ ] Vertical consumer (e-commerce, condomínio)
- [ ] 3.000,8.000 casos/ano

### Fase 3 | 18,36 meses (institucional)
- [ ] Piloto 1 TJ/CEJUSC
- [ ] API tribunal produção (mTLS)
- [ ] Split automático de receita
- [ ] Convênio OAB seccional
- [ ] 10.000+ casos/ano

### Fase 4 | 36+ meses
- [ ] Multi-estado
- [ ] Investimento / escala nacional
- [ ] Posicionamento infraestrutura ADR do Brasil

---

## 8. Estrutura operacional enxuta

```text
Fundador: tecnologia + jurídico estratégico
IA: suporte L1, triagem, prazos, comunicações padrão
Plataforma: motor de resolução (automação)
Parceiros: mediadores, conciliadores, árbitros (credenciados)
Terceirizado (quando escalar): call center, admin financeiro
Freelancer tech: apenas se necessário
```

---

## 9. Riscos e mitigação

| Risco | Mitigação |
|-------|-----------|
| Baixa conversão inicial | 1 vertical, 1 parceiro âncora |
| TJ lento para convênio | Métricas privadas primeiro |
| Qualidade da rede | Credenciamento rigoroso |
| LGPD | Política, descadastro, exclusão (implementado) |
| Dependência do fundador | Documentação + motor automatizado |

---

## 10. Próximos passos imediatos

1. Rodar fluxo ponta a ponta em produção (wizard → pagamento demo → timeline)
2. Credenciar 10,20 profissionais piloto
3. Fechar 1 parceiro B2B com conflitos recorrentes
4. Publicar estudo de impacto (tempo, custo, taxa acordo)
5. Iterar UX do painel do procedimento

---

*ARBITRAJUS © 2026 | Documento interno estratégico*
