# 🛡️ Pesquisa & Relatório Executivo: Segurança do n8n Self-Hosted em Produção

> **Metodologia de Pesquisa Assistida por IA**: Este repositório apresenta a pesquisa técnica e o plano estratégico para o endurecimento e proteção de instâncias *self-hosted* do **n8n** em ambientes de produção. Todo o processo de mineração de fontes, análise de vulnerabilidades, modelagem de ameaças, síntese normativa e geração do relatório executivo e podcast foi conduzido com o suporte do **Gemini Notebook** (ferramenta de inteligência artificial do Google, anteriormente conhecida como **NotebookLM**).

---

## 📌 1. Tema e Objetivo

### O Tema
O **n8n** é uma das principais plataformas de automação de fluxos de trabalho (*workflow automation*). Quando implantado em modo *self-hosted* em ambientes corporativos, ele assume um papel crítico de orquestração: centraliza credenciais de alto privilégio (bancos de dados, ERPs, CRMs, serviços de e-mail, nuvem e APIs) e executa código customizado (JavaScript e Python), além de atuar como motor de integração para Agentes de IA e LLMs.

### O Objetivo Central: O Relatório Executivo
O objetivo principal desta pesquisa foi gerar um quadro normativo e operacional completo de segurança para o n8n self-hosted em produção, cujo resultado culminou no **Relatório Executivo de Segurança (`n8n-relatorio-executivo-seguranca.md`)**.

O Relatório Executivo foi projetado para responder às seguintes necessidades estratégicas:
1. Mapear a superfície de ataque completa (22 camadas) e os vetores de RCE (*Remote Code Execution*), SSRF e exfiltração de segredos.
2. Definir uma **Arquitetura de Referência em 3 Zonas** com isolamento estrito de *Task Runners* em contêineres *sidecar distroless*.
3. Estabelecer um **Security Baseline** categorizado com priorização clara (**P0** = Obrigatório para Produção, **P1** = Alta Recomendação, **P2** = Maturidade).
4. Garantir a conformidade com a **LGPD** (Lei Geral de Proteção de Dados) e frameworks internacionais (**OWASP Top 10 LLM**, **NIST CSF 2.0**, **NIST SP 800-207 Zero Trust**).
5. Fornecer um **Checklist Operacional de Auditoria** pronto para ser executado por administradores Linux e engenheiros DevOps.

---

## 📚 2. Fontes Utilizadas e Crivo de Confiabilidade

A pesquisa foi fundamentada em um acervo rigoroso de **46 fontes primárias e secundárias**, organizadas para garantir rastreabilidade e evitar alucinações:

| Categoria | Fontes Ingeridas | Por que são confiáveis? |
| :--- | :--- | :--- |
| **Documentação Oficial n8n** | Manual de Deploy, *Harden Task Runners*, *Rotate Encryption Keys*, *Run Security Audits*, *Set Permissions (RBAC)*, Changelog v2.0 Breaking Changes, Termos AUP e EULA. | Fontes primárias do próprio fabricante (n8n GmbH) detalhando parâmetros exatos da aplicação, flags de runtime e breaking changes. |
| **Relatórios de Vulnerabilidades (CVEs)** | Whitepapers N8Scape (CVE-2025-68668 / Pyodide), CodeAnt AI (CVE-2026-42234 / Python Runner), Advisories n8n Blog (v1.65-v1.120.4). | Análises técnicas detalhadas das falhas reais de RCE, sandbox bypass e traversals descobertas por pesquisadores de segurança. |
| **Frameworks de Segurança e Normas** | NIST CSF 2.0, NIST SP 800-207 (Zero Trust), NIST SP 800-61 Rev. 3 (Incident Response), Lei Geral de Proteção de Dados (LGPD - Lei 13.709/2018). | Padrões internacionais e regulamentação brasileira para governança, gestão de incidentes e privacidade. |
| **OWASP Cheat Sheets & Top 10** | OWASP Top 10 for LLM Applications, Cheat Sheets de Authentication, Authorization, Secrets Management, CI/CD, Supply Chain e Prompt Injection. | Diretrizes normativas da comunidade global de segurança para desenvolvimento e operações seguras. |
| **Infraestrutura e Conteinerização** | Docker Engine Security, Docker Rootless Mode, Kubernetes Pod Security Standards (PSS), K8s Network Policies. | Documentação primária de conteinerização e orquestração para contenção de workloads. |

---

## 🎯 3. Diretrizes de Comportamento dadas ao Gemini Notebook (NotebookLM)

Para assegurar a máxima precisão técnica e evitar que a inteligência artificial inventasse configurações inexistentes, foram estabelecidas as seguintes **regras estritas de conduta**:

1. **Groundedness Absoluto**: Responder estritamente com base nas fontes carregadas. Proibido preencher lacunas com conhecimento geral de treinamento sem sinalização explícita.
2. **Diferenciação Estrita de Camadas**:
   - Diferenciar a segurança do **Produto n8n** da segurança da **Infraestrutura** (Docker / Kubernetes / Proxy Reverso / Nuvem).
   - Diferenciar recomendações oficiais do **n8n**, diretrizes **OWASP/NIST** e exigências da **LGPD**.
3. **Marcação de Transparência**:
   - Identificar explicitamente deduções técnicas como `[INFERÊNCIA]`.
   - Declarar explicitamente `[NÃO CONFIRMADO NAS FONTES]` quando não houver evidência direta.
4. **Respeito à Taxonomia de Riscos e Prioridades**:
   - Classificação rígida em **P0** (obrigatório para entrada em produção), **P1** (alta recomendação) e **P2** (maturidade).
5. **Autocrítica e Auditoria Deliberada**:
   - Executar uma sessão final de auto-auditoria procurando ativamente por erros, desatualizações de versão (v1.x vs v2.x) e diferenciação entre os planos pago (Enterprise/Business) e gratuito (Community).

---

## 🗺️ 4. O Caminho Trilhado: Perguntas Fundamentais, Respostas e Fontes

A pesquisa seguiu um percurso metodológico estruturado em etapas até a consolidação do Relatório Executivo:

### Etapa 1: Mapeamento da Superfície de Ataque (22 Camadas)
* **Pergunta Fundamental**: *"Qual é a superfície de ataque de uma instalação n8n self-hosted em produção em todas as suas 22 camadas (Editor, RBAC, Code Nodes, Webhooks, DB, Redis, AI Agents, etc.)?"*
* **Síntese da Resposta**: Identificou-se que o maior vetor de risco reside nos **Code Nodes** (execução de código JS/Python) e no acesso ao cofre de credenciais. A solução exige o desacoplamento do orquestrador via *Task Runners* externos em modo `distroless` e uso do usuário `nobody`.
* **Fontes-Chave**: *Harden task runners | n8n Docs*, *N8Scape Whitepaper*, *CodeAnt AI Advisory*.

### Etapa 2: Criptografia, Cofre e Gestão de Segredos
* **Pergunta Fundamental**: *"Como o n8n armazena, encripta, rotaciona e protege segredos e chaves de criptografia (`N8N_ENCRYPTION_KEY` e DEK)?"*
* **Síntese da Resposta**: O n8n utiliza criptografia AES-256 de duas camadas. Para evitar vazamentos, é obrigatório definir `N8N_BLOCK_ENV_ACCESS_IN_NODE=true` e nunca armazenar o backup do banco relacional no mesmo local que a chave mestra.
* **Fontes-Chave**: *Rotate encryption keys | n8n Docs*, *Set a custom encryption key | n8n Docs*, *OWASP Secrets Management Cheat Sheet*.

### Etapa 3: Threat Modeling Detalhado (20 Cenários)
* **Pergunta Fundamental**: *"Construa um Threat Model completo cobrindo 20 cenários de ameaça (Atacante externo, RCE, SSRF, Container Escape, Prompt Injection, Compromecimento do DB/Backup, etc.)."*
* **Síntese da Resposta**: Mapeamento estruturado de cada ameaça com Threat Actor, Asset, Attack Path, Controles Preventivos/Detectivos/Corretivos e Risco Residual.
* **Fontes-Chave**: *NIST CSF 2.0*, *OWASP Top 10 LLM*, *Arquitetura e Endurecimento Markdown*.

### Etapa 4: Hardening de Infraestrutura (Docker vs. Kubernetes)
* **Pergunta Fundamental**: *"Quais configurações de Docker e Kubernetes impedem que um comprometimento do n8n resulte em Cluster Takeover?"*
* **Síntese da Resposta**:
  - **Docker**: Proibição de mapeamento do `/var/run/docker.sock`, uso de `readOnlyRootFilesystem`, `--cap-drop=ALL` e isolamento do runner.
  - **Kubernetes**: Imposição de Pod Security Standards *Restricted*, `automountServiceAccountToken: false`, `runAsNonRoot: true` (UID 65532), `NetworkPolicies` default-deny e limitação de Egress (SSRF).
* **Fontes-Chave**: *Pod Security Standards | K8s*, *Network Policies | K8s*, *Docker Engine Security*.

### Etapa 5: Segurança de Agentes de IA e Privacidade / LGPD
* **Pergunta Fundamental**: *"Como proteger AI Agents no n8n contra Prompt Injection / Tool Abuse e garantir conformidade com a LGPD?"*
* **Síntese da Resposta**: Implementação de aprovação humana (*Human-in-the-Loop - HITL*), validação determinística de parâmetros fora do LLM, política de retenção de dados via *Execution Data Redaction* e expurgamento automático (`EXECUTIONS_DATA_MAX_AGE`).
* **Fontes-Chave**: *OWASP Top 10 for LLM Applications*, *Lei Geral de Proteção de Dados (LGPD)*, *Stream logs to external systems | n8n Docs*.

### Etapa 6: Consolidação do Relatório Executivo e Auto-Auditoria
* **Pergunta Fundamental**: *"Consolide toda a pesquisa no Relatório Executivo final, no Security Baseline e no Checklist Operacional de Auditoria."*
* **Síntese da Resposta**: Geração dos artefatos conclusivos, seguida por uma auditoria crítica que apontou as diferenças entre a versão Community e Enterprise do n8n (ex: SSO e External Secrets são recursos Enterprise).

---

## 🎨 5. Artefatos e Links do Repositório

Neste repositório você encontrará os artefatos gerados ao longo do projeto:

1. **📄 Relatório Executivo Principal**: [`n8n-relatorio-executivo-seguranca.md`](./n8n-relatorio-executivo-seguranca.md)
   - O documento final consolidado (equivalente a 10 páginas) cobrindo as 15 seções estratégicas de segurança, baseline, controles P0/P1/P2, LGPD e roadmap.
2. **🎙️ Podcast em Áudio**: [`Como blindar o n8n contra ataques`](./Como%20blindar%20o%20n8n%20contra%20ataques)
   - Visão geral dinâmica e leve em formato de áudio (gerado via Studio do Gemini Notebook) discutindo os principais pontos do relatório executivo.
3. **📋 Checklist Operacional de Auditoria**: [`n8n-operational-audit-checklist.md`](./n8n-operational-audit-checklist.md)
   - Guia passo a passo para administradores SysAdmin/DevOps executarem a verificação prática dos 70 controles no servidor Linux/Kubernetes.
4. **🛡️ Security Baseline & Threat Model**: [`n8n-production-security-baseline.md`](./n8n-production-security-baseline.md) e [`n8n-threat-model-production.md`](./n8n-threat-model-production.md)
   - Matrizes técnicas completas de baseline e modelagem de ameaças.
5. **🖼️ Prints do Notebook**:
   - *(Adicione aqui suas capturas de tela mostrando a interface do Gemini Notebook / NotebookLM, a lista de fontes carregadas e o painel Studio)*

---

🔗 **Link para o Notebook Compartilhado**: [Acesse o Notebook no Gemini Notebook / NotebookLM](INSIRA_O_SEU_LINK_AQUI)
