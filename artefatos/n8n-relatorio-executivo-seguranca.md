# Relatório Executivo: Segurança do n8n Self-Hosted em Produção

**Documento de Governança, Arquitetura e Baseline de Segurança Tecnológica**  
*Data de emissão: Outubro de 2026 | Versão: 2.0 (Self-Hosted Production)*  
*Classificação: Documento Técnico / Uso Interno de Engenharia e Segurança*

---

## 1. Executive Summary

A automação de processos corporativos via **n8n self-hosted** traz eficiência operacional expressiva, mas introduz riscos críticos de segurança cibernética quando implantada sem controles rigorosos. Por atuar como um orquestrador central com acesso a bancos de dados relacionais, CRMs, ERPs, barramentos de e-mail, APIs internas e modelos de inteligência artificial, uma instância do n8n não endurecida torna-se um alvo prioritário para vetores de ataque complexos.

Este relatório executivo estabelece a estratégia abrangente de proteção para ambientes n8n self-hosted em produção. A postura recomendada fundamenta-se nos princípios de **Defesa em Profundidade** e **Zero Trust Architecture (NIST SP 800-207)**, garantindo que a execução de código do usuário, o gerenciamento de credenciais e a orquestração de Agentes de IA sejam estritamente contidos.

### Principais Diretrizes Estratégicas:

1. **Isolamento Rígido de Runtime**: O desacoplamento do motor de execução de código (*Task Runners*) do processo orquestrador principal é indispensável. A execução em contêineres *sidecar* `distroless`, com privilégios reduzidos e sistema de arquivos somente leitura, anula a capacidade de elevação de privilégios.

2. **Cofre e Gestão de Segredos**: O armazenamento de credenciais deve utilizar criptografia AES-256 de duas camadas, proibindo a exposição de chaves mestras e variáveis de ambiente no contexto dos nós de código (*Code Nodes*).

3. **Perímetro e Egress Limitado**: Restrição do tráfego de saída (*Egress Filtering*) e ativação obrigatoria de proteção contra *Server-Side Request Forgery* (SSRF) para impedir a exfiltração de dados e o acesso a serviços de metadados da nuvem (`169.254.169.254`).

4. **Governança de Agentes de IA**: Validação determinística de ferramentas (*Tool Call Validation*) e mecanismos de aprovação humana (*Human-in-the-Loop - HITL*) para mitigar injeções de prompt diretas e indiretas (*OWASP Top 10 for LLM*).

5. **Conformidade à LGPD**: Aplicação de recursos de redação de dados de execução (*Execution Data Redaction*) e expurgamento automático de histórico para proteção de dados pessoais.

---

## 2. Modelo de Ameaça

O modelo de ameaça da plataforma n8n self-hosted analisa a interação entre os componentes da aplicação, a infraestrutura e os atores maliciosos.

### Atores de Ameaça (*Threat Actors*):
* **Atacante Externo não Autenticado**: Explora webhooks expostos, endpoints de formulários ou falhas em dependências web públicas.
* **Usuário Interno Malicioso (*Insider Threat*)**: Usuário com permissão legítima de edição de fluxos que tenta exfiltrar credenciais corporativas ou executar comandos no sistema operacional.
* **Atacante de Cadeia de Suprimentos (*Supply Chain Attacker*)**: Compromete pacotes npm da comunidade (*Community Nodes*) ou imagens de contêiner.
* **Injeção Indireta de IA (*Prompt Injector*)**: Atacante que insere instruções maliciosas em fontes lidas por Agentes de IA (e-mails, chamadas HTTP, documentos).

### Superfícies de Ataque Críticas:
* **Interface do Editor e API REST (`/rest/`, `/api/v1`)**: Alvo de ataques de força bruta, sequestro de sessão e falsificação de requisições.
* **Webhooks Públicos (`/webhook/*`)**: Alvo de inundações de Negação de Serviço (DDoS), envenenamento de cargas úteis e injeção de parâmetros.
* **Code Nodes (JavaScript e Python)**: Vetor para tentativa de escape de sandbox (*Sandbox Escape*) e execução remota de código (RCE).
* **Cofre de Credenciais e Banco de Dados**: Alvo para extração de tokens OAuth2, chaves de API e senhas de sistemas integrados.

---

## 3. Principais Riscos Técnicos

Os riscos operacionais de maior severidade identificados nas análises e boletins de segurança incluem:

1. **Execução Remota de Código (RCE) via Escape de Sandbox**:
   * *Mecanismo*: Vulnerabilidades no ambiente de execução de scripts em Python/JS (como demonstrado na CVE-2025-68668 / N8Scape no Pyodide e CVE-2026-42234 no Python Task Runner) permitem que um atacante contorne a sandbox da linguagem e execute comandos diretamente no contêiner.
   * *Impacto*: Leitura do sistema de arquivos, acesso à chave mestra de criptografia e comprometimento do servidor.
   * *Fonte*: N8Scape Advisory; CodeAnt AI Report.

2. **Vazamento Massivo de Credenciais por Acesso a Variáveis de Ambiente**:
   * *Mecanismo*: Scripts executados dentro de *Code Nodes* acessam a memória do processo Node.js (`process.env`), extraindo a variável `N8N_ENCRYPTION_KEY` e tokens do sistema.
   * *Impacto*: Permite descriptografar todo o banco de dados de credenciais do n8n.
   * *Fonte*: n8n Docs (*v2.0 Breaking changes*); Arquitetura e Endurecimento.

3. **Server-Side Request Forgery (SSRF) e Movimentação Lateral**:
   * *Mecanismo*: O nó de *HTTP Request* ou Agentes de IA são induzidos a realizar requisições para portas de redes privadas internas (`10.0.0.0/8`, `192.168.0.0/16`) ou para a interface de metadados de instâncias em nuvem (`169.254.169.254`).
   * *Impacto*: Roubo de credenciais IAM do provedor de nuvem e varredura de vulnerabilidades na rede interna.
   * *Fonte*: NIST SP 800-207; OWASP Cheat Sheet Series.

4. **Injeção de Prompt Direta e Indireta em AI Agents**:
   * *Mecanismo*: Dados não confiáveis lidos por um Agente de IA alteram o comportamento do modelo LLM, forçando a invocação não autorizada de ferramentas (*Tool Calls*) como envio de e-mails, exclusão de dados em SQL ou chamadas de API.
   * *Impacto*: Alteração não autorizada de registros de negócios e exfiltração de dados confidenciais.
   * *Fonte*: OWASP Top 10 for LLM Applications (LLM01 / LLM02).

5. **Ataques de Cadeia de Suprimentos via Community Nodes**:
   * *Mecanismo*: Instalação de pacotes npm da comunidade contendo código malicioso que intercepta credenciais ou estabelece conexões de comando e controle (C2).
   * *Impacto*: Comprometimento da integridade da aplicação e exfiltração contínua de segredos.
   * *Fonte*: OWASP Software Supply Chain Security Cheat Sheet.

---

## 4. Arquitetura de Referência Endurecida

A arquitetura de referência para n8n self-hosted em produção organiza a infraestrutura em três zonas de segurança isoladas, garantindo que o plano de controle, o plano de execução e a camada de dados estejam segregados.

### Diagrama Conceitual da Arquitetura:

```
[ INTERNET PÚBLICA ]
       │
       ▼
[ WAF / CDN (Cloudflare / Coraza) ] ── (Filtragem DDoS, TLS 1.3, Rate Limiting)
       │
       ▼
[ PROXY REVERSO / INGRESS CONTROLLER ]
       ├── Rota Pública (/webhook/*, /form/*) ────────┐
       └── Rota Privada (/editor, /rest/*) [VPN/SSO]  │
                                                     │
┌────────────────────────────────────────────────────┘
│  [ ZONA DE APLICAÇÃO - PRIVATE APP SUBNET ]
▼
[ n8n Main (Orquestrador) ] ──WebSocket──► [ Task Runner Sidecar (distroless) ]
       │                                        (Execução de Código Isolada)
       ├── (Queue Mode)
       ▼
[ Redis Queue Cluster ] ◄───────────────► [ n8n Workers ]
                                                │
                                                ▼
                                         [ Task Runner Sidecar ]
       │
       ├─────────────────────────────────────────────┐
       ▼                                             ▼
[ ZONA DE DADOS - DATA SUBNET ]            [ EGRESS PROXY / FIREWALL ]
[ PostgreSQL (TLS / AES-256) ]                   │
                                                 ├──► APIs Externas / SaaS
                                                 ├──► Sistemas Internos
                                                 └──► Provedores de IA / LLM
```

### Componentes e Responsabilidades:
* **Ingress / Edge Proxy**: Realiza o encerramento TLS 1.3, aplica proteção contra inundações no WAF e restringe o acesso à interface administrativa do editor `/` apenas a usuários autenticados via VPN corporativa ou Identity-Aware Proxy (IAP com SSO/MFA).
* **n8n Main e Workers**: Processos de orquestração isolados em sub-rede privada, sem IP público. Comunicam-se apenas com a base de dados e com a fila Redis.
* **Task Runners em Modo Externo (`N8N_RUNNERS_MODE=external`)**: Contêineres *sidecar* descartáveis rodando a imagem `n8nio/runners:<ver>-distroless`, sob o usuário não-privilegiado `nobody` (UID/GID 65532), com sistema de arquivos somente leitura e perfil AppArmor restritivo.
* **PostgreSQL & Redis**: Alocados em sub-rede privada de dados sem acesso à internet, exigindo comunicação criptografada por TLS e autenticação forte.

---

## 5. Security Baseline de Produção

O Security Baseline organiza as diretrizes operacionais em **18 categorias funcionais**, classificando a relevância técnica de cada controle em três níveis de prioridade:
* **P0 (Obrigatório)**: Controles indispensáveis. A ausência de qualquer controle P0 impede a entrada em produção.
* **P1 (Recomendado)**: Controles de alta prioridade para mitigar riscos avançados.
* **P2 (Maturidade)**: Controles para elevado grau de automação e governança.

---

## 6. Controles P0 (Obrigatórios para Entrada em Produção)

Os seguintes controles devem estar 100% validados antes do go-live da instância:

* **SEC-01: Chave Mestra de Criptografia de Alta Entropia**: Definir a variável `N8N_ENCRYPTION_KEY` com string aleatória de pelo menos 32 bytes gerada via `openssl rand -base64 32`.
  * *Fonte*: Set a custom encryption key - n8n Docs; LumaDock.
* **SEC-03: Bloqueio de Acesso a Variáveis no Code Node**: Configurar `N8N_BLOCK_ENV_ACCESS_IN_NODE=true` para impedir a exfiltração de chaves mestras e tokens da memória via scripts em workflows.
  * *Fonte*: n8n Docs (*v2.0 Breaking changes*).
* **APP-01: Desacoplamento de Task Runners em Modo Externo**: Definir `N8N_RUNNERS_MODE=external` e executar a imagem `n8nio/runners:<ver>-distroless` em contêineres *sidecar* dedicados.
  * *Fonte*: Harden task runners - n8n Docs; CodeAnt AI.
* **NET-03: Ativação da Proteção SSRF Nativa**: Configurar `N8N_ENABLE_SSRF_PROTECTION=true` para rejeitar requisições de nós HTTP direcionadas a redes privadas e metadados de nuvem (`169.254.169.254`).
  * *Fonte*: n8n Docs (*Release Notes 1.121+*); OWASP Cheat Sheet.
* **CTR-01: Execução com Usuário Não-Root**: Configurar o contêiner principal do n8n para rodar como usuário `node` (UID 1000) e os Task Runners estritamente sob o usuário `nobody` (UID/GID 65532).
  * *Fonte*: Harden task runners - n8n Docs; Docker Engine Security.
* **K8S-01: Desativação do Token Automount do Kubernetes**: Configurar `automountServiceAccountToken: false` nos pods e ServiceAccounts do n8n para impedir o roubo de tokens do K8s API Server.
  * *Fonte*: Pod Security Standards - Kubernetes; [INFERÊNCIA].
* **NET-05: Políticas de Rede Default-Deny**: Aplicar `NetworkPolicies` no Kubernetes ou Security Groups no Docker bloqueando todo o tráfego de entrada e saída por padrão, liberando apenas conexões estritamente autorizadas.
  * *Fonte*: Network Policies - Kubernetes; NIST SP 800-207.
* **NET-01: Encerramento TLS 1.3 no Proxy Reverso**: Exigir conexões encriptadas via HTTPS em todas as rotas públicas e privadas no nível do Ingress/Proxy.
  * *Fonte*: Secure Your n8n Instance (VPS US); OWASP Authentication Cheat Sheet.
* **BKP-01: Armazenamento Segregado de Backup e Chave Mestra**: O backup do banco de dados relacional e o valor da chave `N8N_ENCRYPTION_KEY` nunca devem ser gravados no mesmo diretório, servidor ou bucket.
  * *Fonte*: LumaDock; OWASP Secrets Management Cheat Sheet.

---

## 7. Controles P1 (Recomendados para Alta Segurança)

Controles direcionados à proteção da identidade, observabilidade e governança de produção:

* **IAM-02: Single Sign-On (SSO) com MFA Obrigatório**: Integrar a autenticação do n8n ao IdP corporativo via SAML 2.0 ou OIDC (disponível na edição Enterprise) com imposição de MFA.
  * *Fonte*: Configure SSO - n8n Docs; LumaDock.
* **IAM-03: Controle de Acesso Baseado em Papéis (RBAC)**: Atribuir papeis restritos por projeto (Admin, Editor, Viewer), garantindo que usuários comuns possuam papel de leitura.
  * *Fonte*: Set permissions and roles (RBAC) - n8n Docs.
* **NET-04: Inspeção e Filtragem de Egress Proxy**: Canalizar o tráfego de saída do n8n através de um proxy de saída (Squid/Egress Gateway) para registro e filtragem de domínios por *allowlist*.
  * *Fonte*: NIST SP 800-207; [INFERÊNCIA].
* **LOG-01: Log Streaming em Tempo Real para SIEM**: Transmitir os eventos do barramento de auditoria (`n8n.audit.*`) via Syslog TLS ou Webhook assinado para coletor imutável (OpenObserve, Datadog, Splunk).
  * *Fonte*: Stream logs to external systems - n8n Docs; OpenTelemetry Guide.
* **APP-02: Bloqueio de Nós de Alto Risco**: Inserir os nós de execução de comandos do SO na lista de bloqueio: `N8N_NODES_DENYLIST=["n8n-nodes-base.executeCommand", "n8n-nodes-base.ssh"]`.
  * *Fonte*: n8n Docs (*v2.0 Breaking changes*); VPS US.
* **SUP-01: Política Estrita de Restrição a Community Nodes**: Desativar ou proibir a instalação direta de pacotes npm não auditados pela interface.
  * *Fonte*: OWASP Software Supply Chain Security; Run security audits - n8n Docs.

---

## 8. Controles P2 (Maturidade Avançada e Automação)

Controles de governança contínua e automação de DevSecOps:

* **SEC-04: Integração com External Secrets Managers**: Sincronizar credenciais de produção diretamente com cofres corporativos (HashiCorp Vault, AWS Secrets Manager) via External Secrets Operator.
  * *Fonte*: n8n Docs (*Release Notes 2.12/2.13*); OWASP Secrets Management.
* **SUP-03: Assinatura e Atestação de Imagens (Cosign/Sigstore)**: Validar a assinatura criptográfica das imagens do n8n no controle de admissão (Kyverno/OPA) antes de autorizar o deploy no cluster.
  * *Fonte*: OWASP Software Supply Chain Security; [INFERÊNCIA].
* **WFK-03: Análise Estática de Workflows (Linters/SAST)**: Integrar verificações automatizadas de segurança no pipeline de CI/CD para detectar o uso de nós HTTP sem autenticação ou expressões inseguras em Pull Requests.
  * *Fonte*: OWASP CI/CD Security Cheat Sheet; [INFERÊNCIA].

---

## 9. AI/Agent Security (Segurança em Fluxos de IA)

O uso de **Agentes de IA e Modelos de Linguagem (LLMs)** no n8n exige a mitigação dos riscos previstos no *OWASP Top 10 for Large Language Model Applications*:

### Matriz de Proteção para AI Agents:

1. **Mitigação contra Prompt Injection (LLM01/LLM02)**:
   * Separação estrita entre instruções do sistema (*System Prompts*) e dados de entrada fornecidos por usuários ou fontes externas (e-mails, webhooks).
   * Uso de modelos de filtragem prévia (*Guardrails*) para analisar payloads de entrada antes de repassá-los ao nó do agente.
   * *Fonte*: OWASP LLM Prompt Injection Prevention Cheat Sheet.

2. **Aprovação Humana Obrigatória (Human-in-the-Loop - HITL)**:
   * Imposição de confirmação humana explícita (via nó de formulário ou aprovação por e-mail/chat) para qualquer ferramenta (*Tool*) que execute ações com impacto no mundo real (ex: exclusão em banco SQL, alteração no ERP, envio de e-mail externo ou transação financeira).
   * *Fonte*: OWASP Top 10 for LLM Applications; n8n Docs (*AI Agents & Form Nodes*).

3. **Validação Determinística de Ferramentas (Tool Call Validation)**:
   * Os parâmetros gerados pelo LLM para invocação de uma ferramenta não devem ser repassados diretamente aos sistemas de destino. O fluxo deve validar tipos de dados, limites numéricos e esquemas JSON antes da execução.
   * *Fonte*: OWASP Top 10 for LLM Applications; [INFERÊNCIA].

4. **Isolamento de Memória e Acesso a Contextos**:
   * Garantir que a memória da conversa (*Window Buffer Memory / Vector Store*) seja segregada estritamente por ID de usuário e sessão, impedindo o vazamento de contexto entre interações de usuários distintos.
   * *Fonte*: OWASP Top 10 for LLM Applications.

---

## 10. Privacidade e Conformidade à LGPD

A execução de workflows que manipulam Dados Pessoais (PII) sujeita a organização às disposições da **Lei Geral de Proteção de Dados (LGPD - Lei 13.709/2018)**.

### Papéis Regulatórios e Arquitetura:
* **Controlador dos Dados**: A empresa que contrata/implanta a instância self-hosted do n8n e define as finalidades do tratamento.
* **Operador dos Dados**: A infraestrutura/equipe responsável pela manutenção do servidor.
* **Subprocessadores**: Provedores externos cujas APIs são acionadas pelos nós do n8n (ex: OpenAI, Anthropic, Google, Salesforce).

### Requisitos Técnicos de Conformidade:

1. **Minimização e Finalidade (Art. 6º, I e III)**:
   * Filtrar e remover campos de PII não essenciais no início do fluxo utilizando nós de transformação (`Edit Fields / Set`) antes de transmitir os dados para APIs de terceiros.
   * *Fonte*: Lei Geral de Proteção de Dados (LGPD).

2. **Redação de Dados de Execução (Execution Data Redaction)**:
   * Ativar o recurso de redação de dados de execução para ocultar cargas úteis sensíveis na interface gráfica do n8n, garantindo que operadores não visualizem PII desnecessariamente.
   * *Fonte*: n8n Docs (*Redact execution data*).

3. **Retenção e Expurgamento Automático (Art. 16)**:
   * Configurar a retenção curta de logs de execução no banco de dados via variáveis de expurgo automático (`EXECUTIONS_DATA_MAX_AGE=7` a `30` dias) para garantir que PII não permaneça armazenada indefinidamente no histórico da aplicação.
   * *Fonte*: Lei Geral de Proteção de Dados (LGPD); n8n Docs (*Execution data pruning*).

4. **Transferência Internacional de Dados (Art. 33)**:
   * Mapear chamadas feitas para APIs com servidores fora do território nacional (ex: provedores de LLM nos EUA) e garantir a existência de cláusulas contratuais padrão (*Standard Contractual Clauses*) com os fornecedores.
   * *Fonte*: Lei Geral de Proteção de Dados (LGPD); n8n Privacy Policy.

---

## 11. Incident Response (Resposta a Incidentes)

Em conformidade com as diretrizes da **NIST SP 800-61 Rev. 3 (Incident Response Recommendations)**, a organização deve manter um Procedimento Operacional Padrão (POP) para resposta a incidentes de segurança no n8n.

### Fases do Plano de Resposta:

```
[ 1. PREPARAÇÃO ] ──► [ 2. DETECÇÃO E ANÁLISE ] ──► [ 3. CONTENÇÃO E ERRADICAÇÃO ] ──► [ 4. RECUPERAÇÃO E PÓS-INCIDENTE ]
```

1. **Preparação**:
   * Manter contêineres e imagens atualizados, logs centralizados no SIEM e backups testados.
2. **Detecção e Análise**:
   * Identificar alertas anômalos no SIEM (ex: múltiplos eventos `n8n.audit.user.login.failed`, execuções suspeitas de `Code Node` ou pico imprevisto no consumo de tokens de IA).
   * *Fonte*: NIST SP 800-61 Rev. 3; Stream logs to external systems - n8n Docs.
3. **Contenção Imediata**:
   * **Isolamento de Rede**: Aplicar `NetworkPolicy` emergencial para cortar o tráfego de saída do namespace do n8n.
   * **Revogação de Sessões**: Executar a revogação de tokens de usuários via CLI ou reiniciar as instâncias de aplicação.
   * **Revogação de Credenciais Upstream**: Revogar imediatamente no provedor SaaS/AWS qualquer chave de API que esteve associada à instância comprometida.
   * *Fonte*: NIST SP 800-61 Rev. 3; [INFERÊNCIA].
4. **Erradicação e Recuperação**:
   * Destruir os contêineres comprometidos e reimplantá-los a partir de imagens verificadas no pipeline de CI/CD.
   * Executar a rotação da chave mestra `N8N_ENCRYPTION_KEY` e das DEKs no banco de dados.
   * *Fonte*: Rotate encryption keys - n8n Docs; NIST SP 800-61 Rev. 3.

---

## 12. Backup & Disaster Recovery (DR)

A estratégia de continuidade de negócios para o n8n deve assegurar a capacidade de restauração completa da aplicação contra cenários de corrupção de dados ou ataques de ransomware.

### Requisitos de Disaster Recovery:

* **Separação Obrigatória de Componentes**: O arquivo de dump do banco de dados relacional (PostgreSQL) e o valor da chave `N8N_ENCRYPTION_KEY` **devem ser armazenados em locais fisicamente e logicamente separados**. O comprometimento do repositório de backup do banco de dados não deve expor a chave mestra de descriptografia.
  * *Fonte*: LumaDock; OWASP Secrets Management Cheat Sheet.
* **Criptografia em Repouso**: Todos os arquivos de backup enviados para armazenamento externo ou nuvem devem ser encriptados com algoritmo AES-256 antes da transmissão.
  * *Fonte*: NIST Cybersecurity Framework; OWASP Secrets Management Cheat Sheet.
* **Imutabilidade (Object Lock / WORM)**: Armazenar cópias de backup em buckets com política de retenção imutável para proteção contra exclusão maliciosa por ransomware.
  * *Fonte*: NIST Cybersecurity Framework; [INFERÊNCIA].
* **Teste de Restauração Periódico**: Realizar simulação trimestral de restauração completa (*Disaster Recovery Restore Test*) em ambiente de homologação isolado para validar a integridade dos dados e das chaves de criptografia.
  * *Fonte*: NIST Cybersecurity Framework.

---

## 13. Security Audit & Observabilidade

A auditoria da postura de segurança do n8n combina verificações diagnósticas internas e monitoramento contínuo de eventos em tempo real.

### Ferramentas e Rotinas de Auditoria:

1. **Ferramenta Diagnóstica Nativa (`n8n audit`)**:
   * Executar o comando `n8n audit` via CLI, REST API (`POST /audit`) ou fluxo agendado.
   * O relatório gerado identifica credenciais cadastradas mas não utilizadas, workflows inativos com acesso a segredos e o uso de nós de risco.
   * *Fonte*: Run security audits - n8n Docs; VPS US.

2. **Monitoramento via OpenTelemetry e Prometheus**:
   * Coletar métricas técnicas do endpoint `/metrics` para monitorar taxa de erros, tempo de execução de fluxos e consumo de recursos.
   * Integrar a propagação de contextos W3C (`traceparent`) no nó HTTP Request para rastreamento distribuído de chamadas externas.
   * *Fonte*: n8n Monitoring with OpenTelemetry and OpenObserve.

3. **Event Bus Log Streaming**:
   * Habilitar a transmissão automatizada de logs estruturados em JSON cobrindo eventos de login, alteração de permissões, criação/atualização de credenciais e execução de workflows.
   * *Fonte*: Stream logs to external systems - n8n Docs.

---

## 14. Lacunas Técnicas Identificadas

A análise crítica do acervo documental e da arquitetura do n8n revelou as seguintes lacunas técnicas que devem ser objeto de investigação futura e acompanhamento junto ao fabricante:

1. **Ausência de Manifestos de Referência Oficiais para Kubernetes / Helm Hardened**:
   * *Lacuna*: A documentação oficial do n8n fornece diretrizes detalhadas de hardening para Docker Compose, mas não disponibiliza um gráfico Helm oficial pré-configurado com Pod Security Standards *Restricted*, `NetworkPolicies` e regras de admissão.
   * *Status*: `[LACUNA DE INFRAESTRUTURA]`

2. **Parâmetros de Conexão SSL/TLS e Validação de CA no PostgreSQL**:
   * *Lacuna*: Embora a documentação determine o uso do PostgreSQL, faltam especificações explícitas sobre como passar os parâmetros de validação estrita de certificado CA (`sslmode=verify-full` e `sslrootcert`) via variáveis de ambiente da aplicação.
   * *Status*: `[LACUNA DE DOCUMENTAÇÃO]`

3. **Validação Determinística Nativa de Chamadas de Ferramentas em Agentes de IA**:
   * *Lacuna*: O n8n disponibiliza os nós de AI Agents e ferramentas de integração, mas a validação de parâmetros gerados pelo modelo antes do disparo da ferramenta precisa ser construída manualmente via lógica de sub-workflows, sem uma camada de schema-validation nativa e automática no nó de IA.
   * *Status*: `[LACUNA DE FUNCIONALIDADE]`

4. **Procedimento de Teste Automatizado para Rotação de Chave Mestra**:
   * *Lacuna*: O procedimento de rotação da `N8N_ENCRYPTION_KEY` exige exportação/importação manual via CLI (`n8n export:credentials --all --decrypted`), o que introduz risco de erro humano e janela de exposição de arquivo temporário sem criptografia durante a operação.
   * *Status*: `[LACUNA OPERACIONAL]`

---

## 15. Roadmap de Implementação Priorizado

O roadmap de implementação estabelece a sequência cronológica para adequação da postura de segurança em instâncias self-hosted, dividido em três fases executivas:

```
[ FASE 1: CONTENÇÃO E P0 ] ──► [ FASE 2: ESTRUTURAÇÃO E P1 ] ──► [ FASE 3: MATURIDADE E P2 ]
      (Semanas 1 a 2)                  (Semanas 3 a 6)                  (Semanas 7 a 12)
```

### Fase 1: Contenção Imediata e Controles P0 (Semanas 1 a 2)
* Configuração da chave mestra `N8N_ENCRYPTION_KEY` de 32 bytes gerada via OpenSSL.
* Ativação de `N8N_BLOCK_ENV_ACCESS_IN_NODE=true` e `N8N_ENABLE_SSRF_PROTECTION=true`.
* Desacoplamento de *Task Runners* em modo externo (`N8N_RUNNERS_MODE=external`) com a imagem `distroless` sob o usuário `nobody` (65532).
* Aplicação de `automountServiceAccountToken: false` e execução sob `runAsNonRoot: true`.
* Isolamento de rede perimetral e aplicação de `NetworkPolicies` com bloqueio do IP do API Server e metadados (`169.254.169.254`).
* Separação física do backup do PostgreSQL e da chave de criptografia.

### Fase 2: Estruturação e Governança P1 (Semanas 3 a 6)
* Habilitação de SSO corporativo (SAML/OIDC) com MFA obrigatório no IdP.
* Configuração de papéis RBAC restritos por projeto.
* Implementação do Log Streaming de auditoria para o SIEM.
* Configuração de Egress Proxy para inspeção e filtragem de saída.
* Bloqueio de nós de risco em `N8N_NODES_DENYLIST`.
* Implementação de aprovação humana (*HITL*) em workflows com impacto crítico e Agentes de IA.

### Fase 3: Maturidade e Automação P2 (Semanas 7 a 12)
* Integração com External Secrets Operator para sincronização com Vault/AWS Secrets Manager.
* Automação de varredura estática de workflows (SAST/Linters) e auditorias periódicas via `n8n audit`.
* Atestação e assinatura de imagens via Cosign no controle de admissão (Kyverno/OPA).
* Realização do primeiro teste simulado completo de Disaster Recovery e Resposta a Incidentes.

---

**Conclusão**: A implementação integral deste Security Baseline transforma a instância do n8n self-hosted em uma plataforma de automação robusta, imune a vetores triviais de exploração e pronta para orquestrar processos críticos de negócio com plena conformidade regulatória.
