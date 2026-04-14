# Claude Code Security Agents: Red Team + Blue Team Automatizados

> **50+ vulnerabilidades encontradas e corrigidas** usando dois arquivos Markdown que transformam o Claude Code em especialistas de segurança ofensiva e defensiva.

Coloque dois arquivos `.md` no seu projeto. Ganhe pentesting e remediação automatizados — sem frameworks, sem infra, sem experiência em segurança.

---

## Sumário

1. [O Problema](#o-problema)
2. [A Solução](#a-solução)
3. [Como Funciona](#como-funciona)
4. [Setup (2 minutos)](#setup-2-minutos)
5. [Red Team — Agente Ofensivo](#red-team--agente-ofensivo)
6. [Blue Team — Agente Defensivo](#blue-team--agente-defensivo)
7. [Executando os Agentes](#executando-os-agentes)
8. [Vulnerabilidades Reais Encontradas](#vulnerabilidades-reais-encontradas)
9. [Customizando para Seu Stack](#customizando-para-seu-stack)
10. [FAQ](#faq)

---

## O Problema

Você está buildando rápido. Entregando features. Mas segurança fica sempre pro "depois."

Enquanto isso, seu codebase acumula vulnerabilidades silenciosamente: HTML renderizado sem sanitização, endpoints de API públicos, rotas admin sem proteção, webhooks que não verificam assinatura. Você não vai achar isso num code review — elas se escondem na interação entre componentes, nas suposições que você fez às 2 da manhã, na integração de terceiros que você copiou da documentação.

Contratar um pentester custa milhares de reais. Rodar ferramentas SAST te dá 200 avisos genéricos, a maioria falsos positivos. E você ainda tem features pra entregar.

---

## A Solução

Dois agentes especializados para o Claude Code — cada um é um único arquivo Markdown (~120 linhas):

| Agente | Papel | Modo | O que faz |
|--------|-------|------|-----------|
| **Red Team** | Atacante | Somente leitura | Escaneia seu codebase em busca de vulnerabilidades. Pensa como um pentester. Gera relatório estruturado com severidade, cenários de ataque e recomendações de correção |
| **Blue Team** | Defensor | Modo edição | Pega o relatório do Red Team e corrige cada vulnerabilidade. Escreve código, adiciona testes, commita as mudanças |

Eles funcionam como um loop de feedback:

```
Red Team escaneia → encontra 14 vulnerabilidades → gera relatório
                                                        ↓
Blue Team lê relatório → corrige 12/14 → commita fixes
                                                        ↓
Red Team re-escaneia → encontra 3 novos issues → gera relatório
                                                        ↓
Blue Team corrige → hardening completo
```

Cada ciclo leva minutos, não dias.

---

## Como Funciona

O Claude Code suporta [agentes customizados](https://docs.anthropic.com/en/docs/claude-code/agents) — arquivos Markdown em `.claude/agents/` que dão ao Claude uma persona especializada, metodologia e acesso restrito a ferramentas.

O agente Red Team recebe **ferramentas somente leitura** (Read, Grep, Glob, npm audit). Pode analisar tudo mas não muda nada.

O agente Blue Team recebe **ferramentas de edição** (Read, Write, Edit, git). Pode modificar código, instalar pacotes, rodar testes e commitar.

Ambos herdam o contexto do seu projeto via `CLAUDE.md`, então entendem seu stack, arquitetura e restrições sem nenhuma configuração extra.

---

## Setup (2 minutos)

### 1. Crie o diretório de agentes

```bash
mkdir -p .claude/agents
```

### 2. Copie os arquivos dos agentes

```bash
# Opção A: Clone este repo e copie
git clone https://github.com/lucasrosati/claude-security-agents.git /tmp/security-agents
cp /tmp/security-agents/agents/red-team-scanner.md .claude/agents/
cp /tmp/security-agents/agents/blue-team-defender.md .claude/agents/

# Opção B: Download direto
curl -o .claude/agents/red-team-scanner.md https://raw.githubusercontent.com/lucasrosati/claude-security-agents/main/agents/red-team-scanner.md
curl -o .claude/agents/blue-team-defender.md https://raw.githubusercontent.com/lucasrosati/claude-security-agents/main/agents/blue-team-defender.md
```

### 3. Rode sua primeira auditoria

Abra o Claude Code no seu projeto e digite:

```
Run the red-team-scanner agent on this codebase
```

Só isso. Sem arquivos de config, sem API keys, sem containers Docker.

---

## Red Team — Agente Ofensivo

**Arquivo:** [`agents/red-team-scanner.md`](./agents/red-team-scanner.md)

**Modelo:** Sonnet (rápido, custo-benefício para scanning)
**Modo de permissão:** `plan` (somente leitura — não pode modificar seu código)

### O que analisa

| Categoria | Exemplos |
|-----------|---------|
| Autenticação e Autorização | Auth quebrada, validação JWT fraca, rate limiting ausente, IDOR, escalação de privilégio |
| Segurança de API | Endpoints públicos, autorização a nível de objeto ausente, mass assignment, enumeração |
| Validação de Input e Injeção | SQL/NoSQL injection, XSS (stored/reflected/DOM), SSRF, path traversal |
| Upload de Arquivos | MIME spoofing, validação de magic bytes ausente, storage público exposto |
| Segurança de Dados e Compliance | Secrets hardcoded, PII em logs, armazenamento inseguro de tokens, gaps LGPD/HIPAA |
| Infraestrutura | CORS mal configurado, security headers ausentes (CSP, HSTS), problemas de container |
| Painel Admin | Rotas admin desprotegidas, endpoints escondidos, session hijacking |
| Pagamento e Billing | Verificação de webhook ausente, manipulação de preço, bypass de assinatura |
| Banco de Dados (Supabase/Firebase) | Políticas RLS quebradas, service_role key exposta, acesso cross-tenant |

### Formato de saída

Para cada vulnerabilidade encontrada:

```
[VULN-001] XSS via conteúdo de IA renderizado sem sanitização
Severidade: HIGH
Localização: src/pages/StudySession.tsx:267
Descrição: HTML gerado por IA renderizado via dangerouslySetInnerHTML sem sanitização
Cenário de ataque: Atacante injeta <script> via conteúdo manipulado → rouba tokens de sessão
Impacto: Account takeover, exfiltração de dados
Correção: Sanitizar com DOMPurify antes de renderizar
```

---

## Blue Team — Agente Defensivo

**Arquivo:** [`agents/blue-team-defender.md`](./agents/blue-team-defender.md)

**Modelo:** Sonnet
**Modo de permissão:** `acceptEdits` (pode modificar código com sua aprovação)

### O que corrige

| Categoria | Ações |
|-----------|-------|
| Autenticação | Rate limiting, hardening de JWT, invalidação de sessão, validação de roles |
| Segurança de API | Middleware de auth, validação de input (Zod/Joi), autorização a nível de objeto |
| Prevenção de Injeção | DOMPurify para XSS, queries parametrizadas, checks de path traversal, whitelists SSRF |
| Upload de Arquivos | Validação por magic bytes, limites de tamanho, renomeação de arquivos, signed URLs |
| Segurança de Dados | Secrets para env vars, remoção de PII dos logs, criptografia at rest, audit logging |
| Infraestrutura | Security headers (CSP, HSTS, X-Frame-Options), CORS restritivo, HTTPS forçado |
| Painel Admin | Middleware de auth + role, logging de ações admin, rate limiting |
| Pagamento | Verificação de assinatura de webhook, validação server-side de preços, idempotency keys |
| Banco de Dados | Auditoria e correção de RLS, validação de input em Edge Functions, prevenção cross-tenant |

### Formato de saída

Para cada correção aplicada:

```
[FIX para VULN-001] Sanitizar conteúdo de IA com DOMPurify
Status: FIXED
O que foi feito: Adicionado wrapper sanitizeAIContent() usando DOMPurify com whitelist restritiva de tags
Arquivos modificados: src/lib/sanitize.ts (novo), src/pages/StudySession.tsx, + 3 outras páginas
Teste: Injetar <script>alert(1)</script> no conteúdo agora renderiza como texto puro
Prevenção: Todo conteúdo de IA deve passar por sanitizeAIContent() antes de dangerouslySetInnerHTML
```

### Checklist de hardening

Após corrigir todas as vulnerabilidades, o Blue Team gera um checklist final:

```
- [x] Security headers implementados
- [x] Rate limiting ativo em auth e API
- [x] CORS restrito a origens conhecidas
- [x] Logging e audit trail funcionando
- [x] Dados sensíveis criptografados at rest e in transit
- [x] Secrets em env vars (nenhum hardcoded)
- [x] Dependências atualizadas (0 CVEs critical/high)
- [x] Políticas RLS auditadas
- [x] Validação de input em todos os endpoints
- [x] Upload de arquivos seguro
- [x] Painel admin protegido
- [x] Webhooks verificados
- [ ] Compliance LGPD (consentimento, retenção de dados, audit log)
```

---

## Executando os Agentes

### Primeira auditoria (Red Team)

```bash
# No Claude Code
> Run the red-team-scanner agent on this codebase
```

O Red Team vai:
1. Mapear seu stack, endpoints, rotas e middleware
2. Rodar análise estática buscando padrões vulneráveis
3. Rodar `npm audit` para CVEs conhecidas nas dependências
4. Revisar configuração (env vars, CORS, headers)
5. Rastrear fluxos de dados sensíveis (PII, tokens, credenciais)
6. Gerar relatório numerado (VULN-001, VULN-002, ...)

### Aplicar correções (Blue Team)

```bash
# No Claude Code, após receber o relatório do Red Team
> Run the blue-team-defender agent. Here is the Red Team report: [cole o relatório]
```

O Blue Team vai:
1. Triar por severidade (CRITICAL e HIGH primeiro)
2. Engenharia reversa de cada vetor de ataque
3. Implementar a correção no código
4. Rodar testes para verificar que nada quebrou
5. Commitar cada grupo de fix: `fix(security): [VULN-XXX] descrição`
6. Gerar checklist de hardening

### Re-scan (iterar)

```bash
# Rode o Red Team novamente para capturar regressões ou novos issues
> Run the red-team-scanner agent again. Previous audit fixed VULN-001 through VULN-012.
```

Repita até o relatório do Red Team voltar limpo.

### Comandos rápidos

| Comando | O que faz |
|---------|-----------|
| `Run red-team-scanner` | Auditoria completa de segurança |
| `Run blue-team-defender` | Corrigir vulnerabilidades do relatório Red Team |
| `Run red-team-scanner on src/api/` | Escanear diretório específico |
| `Run red-team-scanner focusing on authentication` | Scan direcionado |

---

## Vulnerabilidades Reais Encontradas

Estas são vulnerabilidades reais que os agentes encontraram em uma aplicação SaaS em produção (React + Supabase + Edge Functions):

### Críticas

| ID | Vulnerabilidade | Impacto |
|----|----------------|---------|
| VULN-002 | Self-referral no sistema de afiliados | Usuários podiam gerar comissões para si mesmos |
| VULN-010 | Processamento duplicado de comissão | Mesmo pagamento podia disparar comissão dupla |

### Alta severidade

| ID | Vulnerabilidade | Impacto |
|----|----------------|---------|
| VULN-003 | XSS em 4 páginas via `dangerouslySetInnerHTML` | Injeção de scripts em conteúdo de estudo, roubo de sessão |
| VULN-004 | Endpoints de API com zero autenticação | Qualquer pessoa podia consumir a cota de IA |
| VULN-005 | Rotas admin acessíveis via race condition | Usuários comuns podiam acessar o painel admin |
| VULN-006 | Campo de referral podia ser sobrescrito | Manipulação de atribuição de afiliado |
| VULN-007 | Templates de email vulneráveis a HTML injection | Phishing via nomes de usuário manipulados |
| VULN-008 | JWT parseado sem verificação de assinatura | Bypass de autenticação |

### Média severidade

| ID | Vulnerabilidade | Impacto |
|----|----------------|---------|
| VULN-009 | Sem limite de payload em endpoint de import em massa | DoS via request oversized |
| VULN-011 | CORS wildcard em webhook server-to-server | Abuso de requisições cross-origin |
| VULN-012 | Enumeração de CPF sem autenticação | Violação de LGPD, data harvesting |
| VULN-013 | Funções SQL com volatilidade incorreta | Cache poisoning no query planner |

Todas corrigidas automaticamente pelo agente Blue Team.

---

## Customizando para Seu Stack

Os agentes funcionam direto com qualquer stack que o Claude Code suporta. Para otimizar pro seu projeto:

### 1. Mude a linha de identidade

Substitua a descrição do projeto em ambos os arquivos:

```markdown
# Default (genérico)
You are the **Red Team Scanner** for this project.

# Customizado (exemplo)
You are the **Red Team Scanner** for Acme Corp — a B2B fintech SaaS handling payment processing.
```

### 2. Adicione seções específicas do seu stack

Se você usa Firebase em vez de Supabase, substitua a seção 9:

```markdown
### 9. Firebase-Specific
- Auditoria de security rules do Firestore
- Configuração do Firebase Auth
- Validação de input em Cloud Functions
- Storage rules (público vs privado)
```

### 3. Adicione requisitos de compliance

```markdown
### 10. SOC 2 Compliance
- Logging de acesso para todas operações sensíveis
- Criptografia de dados at rest e in transit
- Procedimentos de resposta a incidentes documentados
- Avaliação de segurança de fornecedores
```

### 4. Mude o modelo

Para análise mais profunda, use Opus:

```yaml
model: opus  # mais detalhado, custo maior
```

Para scans mais rápidos em codebases grandes:

```yaml
model: haiku  # mais rápido, mais barato, menos detalhado
```

### 5. Mude o idioma

Os agentes funcionam em qualquer idioma que o Claude suporta. Basta reescrever as instruções:

```markdown
# Versão em português
Você é o **Red Team Scanner** deste projeto.
## Identidade
- Senior Security Engineer especializado em SaaS, OWASP Top 10, API Security
```

---

## FAQ

**Preciso ter experiência em segurança para usar esses agentes?**
Não. O Red Team explica cada vulnerabilidade com cenários de ataque em linguagem clara. O Blue Team corrige automaticamente. Você só revisa as mudanças.

**O Red Team vai modificar meu código?**
Não. Ele roda em modo `plan` (somente leitura). Pode ler arquivos, buscar código e rodar `npm audit`, mas não pode escrever, editar ou executar comandos arbitrários.

**O Blue Team vai quebrar meu app?**
Ele roda testes após cada correção e commita mudanças separadamente. Se algo quebrar, você pode reverter commits individuais. Além disso, roda em modo `acceptEdits`, então você aprova cada mudança.

**Quanto custa?**
Os agentes usam sua assinatura existente do Claude Code ou créditos de API. Uma auditoria completa de um projeto médio (~100 arquivos) tipicamente leva 5-15 minutos com Sonnet.

**Posso rodar no CI/CD?**
Não diretamente — agentes do Claude Code são interativos. Mas você pode agendar auditorias periódicas como parte do seu workflow de segurança.

**Os agentes são específicos de alguma linguagem?**
Não. Os agentes funcionam com qualquer linguagem de programação. O Claude Code entende o codebase independente da linguagem.

**Qual a diferença de ferramentas como Snyk ou SonarQube?**
Ferramentas SAST encontram issues baseados em padrões (CVEs conhecidas, code smells simples). Esses agentes entendem sua arquitetura, rastreiam fluxos de dados entre componentes e encontram vulnerabilidades de lógica (self-referral, race conditions, escalação de privilégio) que ferramentas estáticas não pegam.

---

## Arquitetura

```
.claude/
└── agents/
    ├── red-team-scanner.md      ← ~120 linhas, somente leitura
    └── blue-team-defender.md    ← ~140 linhas, modo edição

Workflow:
┌──────────────┐   relatório  ┌──────────────┐    fixes    ┌──────────┐
│  Red Team    │ ──────────→ │  Blue Team   │ ─────────→ │ Codebase │
│  (scanner)   │             │  (defender)  │            │ (seguro) │
└──────────────┘             └──────────────┘            └──────────┘
       ↑                                                       │
       └───────────────── re-scan ─────────────────────────────┘
```

---

## Créditos e Links

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — agente de código da Anthropic
- [Claude Code Agents](https://docs.anthropic.com/en/docs/claude-code/agents) — documentação de agentes customizados
- [OWASP Top 10](https://owasp.org/www-project-top-ten/) — referência de vulnerabilidades de segurança

---

**Se este guia te ajudou a proteger seu app, dê uma ⭐ no repo e compartilhe com outros devs que estão shippando rápido.**
