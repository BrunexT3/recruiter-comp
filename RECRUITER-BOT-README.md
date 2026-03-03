# Comp Recruiter Bot — Guia de Operação

## Visão Geral

Bot automatizado de sourcing e outreach para as vagas de crescimento da Comp, usando Claude Cowork + LinkedIn via Chrome.

---

## Fluxo de 6 Etapas

### 1. SCAN (Automático — Seg e Qui, 9h)
**Task:** `recruiter-scan`
O bot lê as JDs, busca candidatos no LinkedIn, avalia fit e registra na planilha.

**O que faz:**
- Lê as 3 JDs (BD, Growth Generalist, GTM Engineer)
- Monta queries de busca por vaga
- Acessa LinkedIn via Chrome, extrai dados dos perfis
- Avalia score de fit (1-10) com justificativa
- Registra na planilha com Status="Encontrado"

### 2. REVIEW (Humano)
**Você abre a planilha** `recruiter-pipeline.xlsx` e:
- Filtra por Status="Encontrado"
- Revisa scores e justificativas
- Muda Status para "Aprovado" ou "Rejeitado"

### 3. CRAFT (Semi-automático)
**Na aba "Templates Mensagens"** da planilha:
- Já existem templates iniciais para cada vaga e tipo de mensagem
- Revise/ajuste conforme seu tom desejado
- Mude Status Aprovação para "Aprovado" nos que estiverem ok

### 4. CONNECT (Manual trigger)
**Task:** `recruiter-connect`
Rode manualmente após aprovar candidatos e templates.
- Envia pedidos de conexão com nota personalizada
- Max 25/dia, cooldown 45s entre ações

### 5. NURTURE (Automático — Seg, Qua, Sex, 10h)
**Task:** `recruiter-nurture`
- Verifica conexões aceitas
- Envia follow-ups na cadência 3-5-7 dias
- Monitora respostas (sem responder automaticamente)

### 6. SCHEDULE (Humano)
Quando candidatos respondem com interesse:
- Você vê na planilha (Status="Respondeu - Interesse")
- Você agenda a entrevista manualmente ou pede ao bot para sugerir horários

---

## Planilha: recruiter-pipeline.xlsx

| Aba | Função |
|-----|--------|
| Pipeline | Todos os candidatos, status, datas, scores |
| Templates Mensagens | Templates de approach por vaga e tipo |
| Dashboard | Métricas consolidadas (atualizar manualmente) |
| Log Ações | Registro de todas as ações do bot |
| Config | Parâmetros de limites e cadências |

---

## Guardrails de Segurança

| Regra | Valor |
|-------|-------|
| Max conexões/dia | 25 |
| Max mensagens/dia | 25 |
| Cooldown entre ações | 45 segundos mínimo |
| Cadência Follow-up 1 | 3 dias após conexão |
| Cadência Follow-up 2 | 5 dias após FU1 |
| Cadência Follow-up 3 | 7 dias após FU2 |
| Horário de operação | 9h-11h |
| CAPTCHA/Restrição | PARAR IMEDIATAMENTE |

### O bot NUNCA deve:
- Enviar mais de 25 ações/dia
- Enviar mensagem sem template aprovado
- Responder candidatos automaticamente
- Alterar dados do perfil da conta
- Ignorar CAPTCHAs ou restrições
- Enviar follow-up fora da cadência
- Operar fora do horário configurado

---

## Vagas Ativas

1. **Business Development** — Junior/Mid, outbound B2B, hunter profile
2. **Growth Generalist** — Mid/Senior, experimentação e product marketing
3. **GTM Engineer** — Revenue Ops, automação (Clay, Make, n8n, HubSpot)

---

## Como Operar

### Dia a dia:
1. **Seg/Qui 9h** — Bot roda SCAN automaticamente
2. **Após SCAN** — Abra a planilha, revise candidatos, aprove/rejeite
3. **Após aprovação** — Rode `recruiter-connect` manualmente
4. **Seg/Qua/Sex 10h** — Bot roda NURTURE automaticamente
5. **Diariamente** — Verifique respostas na planilha

### Ajustes:
- Para mudar limites: edite a aba "Config" da planilha
- Para mudar templates: edite a aba "Templates Mensagens"
- Para adicionar vagas: adicione JDs na pasta `jd-growth/` e templates na planilha
