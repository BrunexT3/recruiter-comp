---
name: recruiter-scan
description: Busca candidatos no LinkedIn para vagas da Comp e monta shortlist na planilha
---

## Objetivo
Buscar candidatos no LinkedIn para as vagas abertas da Comp e registrar na planilha de pipeline.

## Contexto
A Comp é uma scale-up de Total Rewards (compensação) backed by Kaszek. Há 3 vagas abertas com JDs na pasta `jd-growth/`:
- Business Development
- Growth Generalist  
- GTM Engineer

## Passos

### 1. Ler as Job Descriptions
Leia os 3 arquivos .md na pasta `jd-growth/` para entender os requisitos de cada vaga.

### 2. Acessar o LinkedIn via Chrome
Use o conector do Chrome (Claude in Chrome) para:
- Abrir o LinkedIn na conta do bot
- Navegar para a busca de pessoas
- Para cada vaga, montar queries de busca relevantes baseadas nos requisitos da JD

### 3. Queries de Busca por Vaga

**Business Development:**
- Buscar: "Business Development" OR "SDR" OR "BDR" + startup + São Paulo
- Filtros: 1-3 anos experiência, Brasil
- Keywords: outbound, vendas B2B, HubSpot, SaaS

**Growth Generalist:**
- Buscar: "Growth" OR "Growth Marketing" OR "Product Marketing" + startup
- Filtros: mid-to-senior, Brasil  
- Keywords: growth hacking, experimentação, demand gen, product marketing

**GTM Engineer:**
- Buscar: "Revenue Operations" OR "GTM Engineer" OR "Growth Engineer" + automação
- Filtros: Brasil
- Keywords: Clay, Make, n8n, Zapier, HubSpot, Python, automação

### 4. Para cada candidato encontrado, extrair (OBRIGATÓRIO visitar o perfil):
**REGRA CRÍTICA: Só registrar candidatos cujo perfil individual foi visitado.**
Nunca registrar candidatos apenas com dados da página de busca. Para cada candidato promissor:
1. Clicar no nome para abrir o perfil completo
2. Aguardar 14-21 segundos (rate limit)
3. Extrair os dados obrigatórios:
   - Nome completo
   - Headline/Cargo atual
   - Empresa atual
   - Localização
   - **URL do perfil LinkedIn** (extrair da barra de endereço — campo obrigatório, nunca deixar como N/A)
   - Experiência estimada (anos) — scrollar até a seção "Experiência"
   - Formação — scrollar até a seção "Formação acadêmica"
   - Skills relevantes mencionadas no perfil
4. Voltar para a página de resultados

Se não for possível visitar o perfil (CAPTCHA, erro, etc.), NÃO adicionar o candidato à planilha.

### 5. Avaliar fit (Score 1-10)
Para cada candidato, avaliar aderência à vaga com base em:
- Match de experiência com o que a JD pede
- Tipo de empresa (startup/tech é preferido)
- Skills relevantes
- Localização (São Paulo é preferido)
- Escrever justificativa curta do score

### 6. Deduplicação (OBRIGATÓRIO antes de registrar)
Antes de adicionar qualquer candidato à planilha:
1. Leia a aba "Pipeline" do `recruiter-pipeline.xlsx` com pandas
2. Extraia a lista de LinkedIn URLs já existentes (coluna "LinkedIn URL")
3. Para cada candidato encontrado, verifique se o LinkedIn URL já está na planilha
4. Se o URL já existe → PULAR o candidato e registrar no log "Candidato duplicado — já existe no Pipeline"
5. Se o URL não existe → prosseguir com o registro normalmente
6. Também respeitar candidatos com Decisão Gestor = "Descartado" — nunca re-adicionar

Isso garante que execuções repetidas não criem duplicatas.

### 7. Registrar na planilha
Abra o arquivo `recruiter-pipeline.xlsx` na pasta do workspace.
Na aba "Pipeline", adicione cada candidato NOVO (que passou pela deduplicação).
Preencha: ID sequencial (continuando do último ID existente), Vaga, dados do perfil, Score, Justificativa, Status="Encontrado", Decisão Gestor="Pendente", Data Encontrado=hoje.

### 8. Registrar no Log
Na aba "Log Ações", registre cada ação realizada com data/hora.
Incluir: buscas realizadas, perfis analisados, candidatos adicionados, candidatos pulados (duplicatas), erros encontrados.

### 9. Gerar/Atualizar candidates.json (OBRIGATÓRIO após registrar na planilha)
Após atualizar a planilha, gerar o arquivo `candidates.json` na raiz do workspace com TODOS os candidatos da aba Pipeline.

Estrutura do JSON:
```json
{
  "meta": {
    "ultima_atualizacao": "YYYY-MM-DDTHH:MM",
    "total_candidatos": N,
    "vagas": ["Business Development", "Growth Generalist", "GTM Engineer"]
  },
  "candidatos": [
    {
      "id": 1,
      "vaga": "...",
      "nome": "...",
      "headline": "...",
      "empresa": "...",
      "localizacao": "...",
      "linkedin_url": "...",
      "experiencia_anos": N,
      "formacao": "...",
      "skills": "...",
      "score": N,
      "justificativa": "...",
      "status": "Encontrado",
      "decisao_gestor": "Pendente",
      "notas_gestor": "",
      "data_encontrado": "YYYY-MM-DD"
    }
  ]
}
```

**Passos:**
1. Ler TODOS os candidatos da aba Pipeline com pandas
2. Preservar os valores de `decisao_gestor` e `notas_gestor` existentes (não sobrescrever decisões já tomadas)
3. Gerar o JSON completo e salvar em `candidates.json`
4. Este arquivo alimenta o dashboard HTML (`index.html`) usado pelo gestor para revisar candidatos

## Limites de Segurança (CRÍTICO)
- Máximo 25 perfis analisados por execução
- Esperar 14-21 segundos entre cada busca/ação no LinkedIn
- Se detectar CAPTCHA ou qualquer restrição, PARAR IMEDIATAMENTE e registrar no log
- NÃO enviar conexões ou mensagens nesta etapa — apenas busca e registro
- NÃO alterar nenhum dado do perfil da conta
- NÃO interagir com posts, comentários ou qualquer conteúdo

## Output esperado
- Planilha atualizada com novos candidatos
- `candidates.json` atualizado com todos os candidatos (alimenta o dashboard HTML)
- Log de ações registrado
- Resumo: quantos candidatos encontrados por vaga, scores médios