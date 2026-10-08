# N8N SELF-HOSTED PRODUCTION SECURITY BASELINE

Este documento estabelece o **Security Baseline de Produção para Instâncias Self-Hosted do n8n**, consolidando requisitos normativos, arquitetura de endurecimento, matrizes de controle e diretrizes operacionais para mitigação de riscos em ambientes corporativos de alta criticidade.

O baseline está estruturado em **18 categorias de controle**, classificando as medidas segundo sua prioridade de implementação:
* **P0**: Controle obrigatório e indispensável para operação em ambiente de produção.
* **P1**: Controle de alta relevância técnica e recomendação prioritária.
* **P2**: Controle de maturidade avançada de segurança e governança.

---

## 1. Governance (GOV)

### GOV-01: Política de Uso Aceitável de Automação e Conformidade Contratual
* **ID**: GOV-01
* **Controle**: Conformidade com Termos de Uso (AUP/EULA) e Política de Uso Aceitável de Automação
* **Descrição**: Estabelecer formalmente as diretrizes corporativas sobre os tipos de processos que podem ser automatizados via n8n, garantindo o cumprimento dos termos do n8n Customer Acceptable Use Policy (AUP) e do End User License Agreement (EULA), proibindo automações para atividades ilegais, extração não autorizada de dados, mineração ou bypass de controles de terceiros.
* **Risco mitigado**: Sanções legais, rescisão unilateral de licença pelo fornecedor, violações regulatórias e responsabilidade civil/penal por automações abusivas.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Equipe de Governança de TI / Jurídico / CISO
* **Como implementar**: Criar a *Política Corporativa de Automação de Processos*, incorporando as proibições do n8n AUP. Exigir que todo desenvolvedor ou editor de fluxos assine o termo antes de receber acesso de edição no n8n.
* **Como verificar**: Auditoria semestral dos tipos de fluxos implantados em produção em relação à lista de casos de uso permitidos na política.
* **Evidência esperada**: Documento de política assinado, registros de aceite dos usuários e relatórios de auditoria de conformidade.
* **Frequência de revisão**: Anual
* **Fonte**: n8n Customer Acceptable Use Policy; n8n End User License Agreement.

### GOV-02: Classificação de Risco e Inventário de Automações Críticas
* **ID**: GOV-02
* **Controle**: Matriz de Classificação de Risco de Workflows e Mapeamento de Impacto
* **Descrição**: Classificar cada workflow do n8n em níveis de criticidade (Baixo, Médio, Alto, Crítico) com base na sensibilidade dos dados manipulados (PII, credenciais, dados financeiros) e nos privilégios dos sistemas integrados (bancos de dados, ERPs, CRMs, APIs com acesso de escrita).
* **Risco mitigado**: Falta de visibilidade sobre automações de alto risco, ausência de controles proporcionais e incapacidade de priorizar incidentes.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Arquitetura de Segurança / Donos de Processo / Admin n8n
* **Como implementar**: Criar um inventário centralizado contendo: ID do workflow, nome, proprietário, nível de criticidade, sistemas de destino e se manipula PII ou executa ações irreversíveis.
* **Como verificar**: Checagem do inventário de workflows contra a lista de fluxos ativos obtida via API do n8n (`GET /api/v1/workflows`).
* **Evidência esperada**: Planilha/Database de inventário de workflows atualizado e vinculado aos donos de negócio.
* **Frequência de revisão**: Trimestral
* **Fonte**: NIST Cybersecurity Framework (CSF 2.0); Arquitetura e Endurecimento Markdown. [INFERÊNCIA]

### GOV-03: Processo Formal de Gestão de Mudanças em Workflows Corporativos
* **ID**: GOV-03
* **Controle**: Esteira de Aprovação e Gestão de Mudanças em Automações de Produção
* **Descrição**: Proibir a criação e edição direta de workflows diretamente na instância de produção. Exigir que todo workflow passe por desenvolvimento em ambiente isolado, revisão de código por pares (*Code Review*), testes de segurança e aprovação formal antes da promoção.
* **Risco mitigado**: Alterações não autorizadas, erros de lógica produtivos, injeção de scripts maliciosos e indisponibilidade de processos de negócio.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: Engenharia de Software / DevOps / SecOps
* **Como implementar**: Implementar fluxo de promoção via repositório Git corporativo utilizando a integração nativa de Git (Enterprise) ou exportação de pacotes `.n8np` via CI/CD, exigindo Pull Request com pelo menos um aprovador.
* **Como verificar**: Comparar a data e os hashes dos fluxos em produção contra os commits aprovados no branch `main` do Git.
* **Evidência esperada**: Histórico de Pull Requests no Git com aprovações registradas e logs do pipeline de deploy.
* **Frequência de revisão**: A cada alteração / Mensal
* **Fonte**: OWASP CI/CD Security Cheat Sheet; Set permissions and roles (RBAC) - n8n Docs. [INFERÊNCIA]

---

## 2. Asset Management (ASM)

### ASM-01: Inventário Automatizado de Instâncias, Workers e Task Runners
* **ID**: ASM-01
* **Controle**: Mapeamento de Ativos e Componentes da Topologia n8n
* **Descrição**: Manter registro atualizado de todas as instâncias do n8n (Main, Workers, Task Runners, contêineres sidecar, bancos PostgreSQL e clusters Redis) operantes na infraestrutura.
* **Risco mitigado**: Instâncias "sombra" (*Shadow IT*), Workers desatualizados executando versões vulneráveis e falta de contenção em caso de incidentes.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DevOps / Cloud Infrastructure / SecOps
* **Como implementar**: Utilizar tags/labels padronizadas nos contêineres e pods (`app.kubernetes.io/name=n8n`, `role=main`, `role=worker`, `role=runner`), e integrar o discovery de ativos ao CMDB/Prometheus.
* **Como verificar**: Executar rotinas de varredura no orquestrador (Docker/Kubernetes) comparando os contêineres ativos com a lista do CMDB.
* **Evidência esperada**: Dashboard de infraestrutura/CMDB listando todas as instâncias n8n e seus respectivos hashes de imagem.
* **Frequência de revisão**: Mensal
* **Fonte**: NIST Cybersecurity Framework (CSF 2.0); Set up task runners - n8n Docs. [INFERÊNCIA]

### ASM-02: Mapeamento de Conexões e Integrações de Terceiros
* **ID**: ASM-02
* **Controle**: Inventário de Módulos, Nós e APIs Conectadas
* **Descrição**: Mapear todas as APIs externas, webhooks de entrada e conexões com sistemas legados ativas nos workflows da organização, identificando os protocolos e o nível de acesso concedido.
* **Risco mitigado**: Conexões órfãs mantendo acesso a sistemas críticos e vazamento de dados por integrações esquecidas.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / Segurança de Aplicações
* **Como implementar**: Utilizar o comando `n8n audit` via CLI/API para extrair a lista completa de nós em uso e mapear as credenciais associadas.
* **Como verificar**: Análise do relatório JSON gerado pelo comando `n8n audit`.
* **Evidência esperada**: Relatório oficial do `n8n audit` arquivado no repositório de segurança.
* **Frequência de revisão**: Trimestral
* **Fonte**: Run security audits - n8n Docs; Secure Your n8n Instance (VPS US).

---

## 3. Identity & Access Management (IAM)

### IAM-01: Habilitação Obrigatória de Multiusuário e Desativação do Modo Anônimo
* **ID**: IAM-01
* **Controle**: Habilitação do Motor de Gestão de Usuários
* **Descrição**: Garantir que o modo de usuário único/anônimo esteja desativado em produção, exigindo autenticação individualizada para qualquer acesso à interface gráfica ou API do n8n.
* **Risco mitigado**: Acesso não autenticado ao canvas do editor, capacidade de visualização e alteração de fluxos por atacantes anônimos.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / SecOps
* **Como implementar**: Configurar a variável de ambiente `N8N_USER_MANAGEMENT_DISABLED=false` e garantir que a conta de proprietário (*Owner*) inicial seja provisionada com senha forte.
* **Como verificar**: Tentar acessar a URL base do n8n em uma janela anônima e verificar o redirecionamento obrigatório para a tela de login.
* **Evidência esperada**: Configuração de variáveis de ambiente do contêiner e teste de acesso bloqueado sem autenticação.
* **Frequência de revisão**: A cada deploy / Contínuo
* **Fonte**: Secure Your n8n Instance (VPS US); n8n Security Guide.

### IAM-02: SSO Obrigatório com MFA via SAML 2.0 / OIDC / LDAP
* **ID**: IAM-02
* **Controle**: Autenticação Centralizada com Single Sign-On e MFA no IdP
* **Descrição**: Exigir que a autenticação de todos os usuários do n8n seja delegada a um Provedor de Identidade corporativo (Okta, Entra ID, Keycloak, Ping Identity) via SAML 2.0, OIDC ou LDAP, impondo Autenticação Multi-Fator (MFA) obrigatória no IdP.
* **Risco mitigado**: Ataques de força bruta, roubo de credenciais locais, uso de senhas fracas e persistência de acessos pós-desligamento de funcionários (*offboarding* tardio).
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: IAM Team / Admin n8n
* **Como implementar**: Configurar SSO nas opções da instância (Enterprise), preenchendo as variáveis `N8N_SSO_SAML_*` ou `N8N_SSO_OIDC_*`, desativando o login local por e-mail/senha caso suportado.
* **Como verificar**: Testar o fluxo de login confirmando o redirecionamento para o IdP corporativo e a exigência do segundo fator de autenticação.
* **Evidência esperada**: Logs de autenticação do IdP registrando logins bem-sucedidos com MFA e tela de configurações SSO ativa no n8n.
* **Frequência de revisão**: Trimestral
* **Fonte**: Configure SSO - n8n Docs; n8n SSO options (LumaDock); OWASP Authentication Cheat Sheet.

### IAM-03: Controle de Acesso Baseado em Papéis (RBAC) em Nível de Projeto e Instância
* **ID**: IAM-03
* **Controle**: Aplicação do Princípio do Menor Privilégio via RBAC
* **Descrição**: Atribuir papéis e permissões estritas aos usuários da plataforma (Instance Admin, Global Member, Project Admin, Project Editor, Project Viewer), restringindo quem pode criar, editar, executar ou apenas visualizar workflows.
* **Risco mitigado**: Escalação de privilégios interna, alteração acidental de fluxos críticos por usuários não qualificados e leitura não autorizada de execuções.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / Governança de TI
* **Como implementar**: Mapear os grupos do IdP corporativo para os papéis do RBAC do n8n via provisionamento automático de papéis (`N8N_SSO_USER_ROLE_PROVISIONING`). Atribuir o papel 'Viewer' por padrão a novos usuários.
* **Como verificar**: Consultar a lista de usuários e suas permissões na interface administrativa (*Settings > Users*) ou via API REST.
* **Evidência esperada**: Relatório de exportação de usuários e papéis atribuídos no n8n.
* **Frequência de revisão**: Mensal
* **Fonte**: Set permissions and roles (RBAC) - n8n Docs; OWASP Authorization Cheat Sheet.

### IAM-04: Aplicação de Políticas de Espaço Pessoal (Personal Space Policies)
* **ID**: IAM-04
* **Controle**: Restrição de Criação e Execução em Espaços Pessoais
* **Descrição**: Restringir a capacidade dos usuários de criarem e executarem workflows com credenciais corporativas dentro de seus espaços pessoais (*Personal Spaces*) sem supervisão.
* **Risco mitigado**: *Shadow Automation*, desvio de governança, exfiltração de dados para contas pessoais e criação de fluxos não auditados.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / SecOps
* **Como implementar**: Ativar as políticas de gerenciamento de espaço pessoal nas configurações globais de governança da instância (Enterprise), proibindo o compartilhamento desregrado e a ativação de fluxos pessoais de alta criticidade.
* **Como verificar**: Tentar ativar um workflow de teste em um espaço pessoal sem permissão administrativa e verificar o bloqueio.
* **Evidência esperada**: Captura de tela das configurações de *Personal Space Policy* ativas na instância.
* **Frequência de revisão**: Semestral
* **Fonte**: Set permissions and roles (RBAC) - n8n Docs; v2.0 Breaking changes - n8n Docs.

### IAM-05: Configuração Segura de Cookies de Sessão e Tokens API
* **ID**: IAM-05
* **Controle**: Proteção de Tokens de Sessão HTTP e Chaves API
* **Descrição**: Impor atributos de segurança estritos nos cookies de sessão emitidos pelo n8n, garantindo que não sejam transmitidos em texto claro nem acessíveis via scripts no navegador (XSS).
* **Risco mitigado**: Sequestro de sessão (*Session Hijacking*), ataques de Man-in-the-Middle (MitM) e falsificação de requisições (*CSRF*).
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DevOps / Admin n8n
* **Como implementar**: Definir as variáveis de ambiente `N8N_SECURE_COOKIE=true` (exige HTTPS) e `N8N_SAMESITE_COOKIE=lax` (ou `strict`).
* **Como verificar**: Inspecionar os cabeçalhos HTTP da resposta do login (`Set-Cookie`) no navegador e confirmar a presença das flags `Secure`, `HttpOnly` e `SameSite=Lax`.
* **Evidência esperada**: Análise de cabeçalhos HTTP capturados via cURL ou ferramentas de inspeção Web.
* **Frequência de revisão**: A cada deploy / Trimestral
* **Fonte**: Secure Your n8n Instance (VPS US); OWASP Session Management Cheat Sheet.

---

## 4. Secrets & Credentials (SEC)

### SEC-01: Configuração de Chave Mestra de Alta Entropia (`N8N_ENCRYPTION_KEY`)
* **ID**: SEC-01
* **Controle**: Definição Manual da Chave Mestra de Criptografia do Cofre
* **Descrição**: Definir manualmente uma chave de criptografia de alta entropia (string aleatória de 32 bytes em base64) para a variável `N8N_ENCRYPTION_KEY` antes da primeira inicialização do n8n, impedindo a geração automática de chaves não versionadas no sistema de arquivos local.
* **Risco mitigado**: Perda permanente do cofre de credenciais ao recriar o contêiner, uso de chaves fracas e exposição de credenciais em backups do arquivo `config`.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DevOps / SecOps
* **Como implementar**: Gerar a chave via `openssl rand -base64 32` e injetá-la como variável de ambiente `N8N_ENCRYPTION_KEY` ou arquivo montado `N8N_ENCRYPTION_KEY_FILE`.
* **Como verificar**: Verificar se a variável está definida no processo e se os logs de inicialização não apresentam a mensagem "Mismatching encryption keys".
* **Evidência esperada**: Arquivo `.env` ou manifesto Kubernetes Secret auditado (com o valor oculto) e logs de inicialização sem erros.
* **Frequência de revisão**: A cada implantação / Anual
* **Fonte**: Set a custom encryption key - n8n Docs; Rotate the n8n encryption key (LumaDock).

### SEC-02: Ativação da Rotação de Data Encryption Keys (DEKs)
* **ID**: SEC-02
* **Controle**: Rotação Ativa de Chaves de Criptografia de Dados
* **Descrição**: Ativar a funcionalidade de rotação de chaves DEK no n8n, permitindo que novas credenciais sejam encriptadas com uma nova chave ativa sem quebrar as credenciais antigas, que são re-encriptadas progressivamente.
* **Risco mitigado**: Comprometimento de longo prazo por uso continuado da mesma chave criptográfica e não conformidade com padrões de gestão de segredos.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / SecOps
* **Como implementar**: Configurar a variável `N8N_ENV_FEAT_ENCRYPTION_KEY_ROTATION=true` em todas as instâncias (Main e Workers) e acionar a rotação pela UI em *Settings > Data Encryption Keys* ou via API `POST /encryption/keys`.
* **Como verificar**: Consultar o status das chaves na interface de gestão de chaves do n8n e verificar se a nova DEK consta como ativa.
* **Evidência esperada**: Captura de tela da UI de gestão de chaves ou resposta JSON do endpoint `/encryption/keys`.
* **Frequência de revisão**: Semestral
* **Fonte**: Rotate encryption keys - n8n Docs; OWASP Secrets Management Cheat Sheet.

### SEC-03: Bloqueio de Acesso a Variáveis de Ambiente no Code Node
* **ID**: SEC-03
* **Controle**: Desativação do Acesso ao `process.env` por Scripts de Usuários
* **Descrição**: Bloquear rigorosamente a capacidade de scripts em JavaScript e Python executados dentro dos nós de código (*Code Nodes*) lerem as variáveis de ambiente do processo principal do n8n.
* **Risco mitigado**: Exfiltração da chave mestra `N8N_ENCRYPTION_KEY`, senhas do banco de dados e tokens de sistema por código malicioso ou injeção de scripts em workflows.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / SecOps
* **Como implementar**: Definir a variável de ambiente `N8N_BLOCK_ENV_ACCESS_IN_NODE=true` no orquestrador do n8n (padrão a partir da v2.0).
* **Como verificar**: Criar um workflow de teste com um *Code Node* contendo `return process.env;` e confirmar que a execução retorna um objeto vazio ou erro de acesso.
* **Evidência esperada**: Log de execução do workflow de teste comprovando a negação de acesso ao `process.env`.
* **Frequência de revisão**: A cada atualização / Mensal
* **Fonte**: v2.0 Breaking changes - n8n Docs; Arquitetura e Endurecimento Markdown; CodeAnt AI.

### SEC-04: Integração com Cofres de Segredos Externos (External Secrets Stores)
* **ID**: SEC-04
* **Controle**: Armazenamento e Resolução Dinâmica de Segredos em Cofres Corporativos
* **Descrição**: Integrar o n8n a gerenciadores de segredos externos (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager, 1Password), evitando a gravação de segredos brutos no banco de dados relacional da aplicação.
* **Risco mitigado**: Vazamento do cofre de credenciais em caso de comprometimento do banco de dados relacional e falta de rotação centralizada de segredos.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: Arquitetura de Segurança / DevOps
* **Como implementar**: Configurar a conexão com o provedor de segredos externo nas configurações de *External Secrets* (Enterprise) e referenciar as chaves nos workflows no formato `{{ $secrets.vault.MY_SECRET }}`.
* **Como verificar**: Verificar se as credenciais cadastradas no n8n utilizam referências externas em vez de valores estáticos em texto claro.
* **Evidência esperada**: Configuração ativa visível em *Settings > External Secrets* e logs de auditoria do Vault/AWS registrando acessos pelo n8n.
* **Frequência de revisão**: Trimestral
* **Fonte**: n8n-docs/docs/changelog/README.md; OWASP Secrets Management Cheat Sheet.

### SEC-05: Padrão de Contas de Serviço com Menor Privilégio e Nomenclatura Padronizada
* **ID**: SEC-05
* **Controle**: Restrição de Escopos de Credenciais e Padronização de Nomes
* **Descrição**: Proibir o uso de contas pessoais de funcionários ou credenciais de administradores globais nos conectores do n8n. Exigir a criação de Contas de Serviço (*Service Accounts* / *App-Only Auth*) com o menor privilégio necessário para a tarefa específica.
* **Risco mitigado**: Ampliação do raio de impacto (*Blast Radius*) em caso de vazamento de credencial, alteração indevida de recursos em sistemas de destino e perda de rastreabilidade.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Donos de Workflows / Segurança de Aplicações
* **Como implementar**: Exigir que cada credencial siga a convenção de nome `[SISTEMA]-[PERMISSÃO]-[EQUIPE]-[PROPÓSITO]` (ex: `stripe-write-ops-invoices`) e limitar os escopos OAuth2 estritamente às APIs necessárias.
* **Como verificar**: Auditoria visual do cadastro de credenciais no n8n e verificação dos escopos configurados no provedor do SaaS.
* **Evidência esperada**: Inventário de credenciais cadastradas com papéis e escopos mapeados.
* **Frequência de revisão**: Trimestral
* **Fonte**: API Authentication and Security for Business Automation (Serenichron); OWASP Authorization Cheat Sheet. [INFERÊNCIA]

---

## 5. Network Security (NET)

### NET-01: Encerramento TLS 1.3 Obrigatório em Proxy Reverso / Ingress
* **ID**: NET-01
* **Controle**: Criptografia de Dados em Trânsito Perimetral
* **Descrição**: Exigir que todo o tráfego de entrada para o n8n passe por um Proxy Reverso (Nginx, Traefik, Caddy) ou Ingress Controller no Kubernetes configurado com TLS 1.3/1.2 e certificados digitais válidos, proibindo conexões HTTP não criptografadas.
* **Risco mitigado**: Interceptação de tráfego (*Eavesdropping*), roubo de tokens de sessão em redes abertas e ataques de Man-in-the-Middle (MitM).
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DevOps / Cloud Infrastructure
* **Como implementar**: Configurar o Proxy Reverso para realizar o encerramento TLS, redirecionar todo o tráfego HTTP (porta 80) para HTTPS (porta 443) e aplicar o cabeçalho HSTS (`Strict-Transport-Security`).
* **Como verificar**: Executar varredura cURL ou SSL Labs na URL do n8n confirmando a rejeição de conexões HTTP e suporte exclusivo a ciphers fortes TLS 1.2/1.3.
* **Evidência esperada**: Configuração do Proxy/Ingress auditada e relatório de teste de TLS com nota A+.
* **Frequência de revisão**: Semestral
* **Fonte**: Secure Your n8n Instance (VPS US); Security | Kubernetes.

### NET-02: Segmentação de Rede em Zonas (DMZ, App, Data Subnet)
* **ID**: NET-02
* **Controle**: Isolamento de Arquitetura de Rede em Três Camadas
* **Descrição**: Dividir a infraestrutura de hospedagem do n8n em três sub-redes isoladas por firewall: Zona de Borda (DMZ para o Proxy/Ingress), Zona de Aplicação (privada para o n8n Main, Workers e Runners) e Zona de Dados (privada restrita para o PostgreSQL e Redis).
* **Risco mitigado**: Acesso direto da internet ao banco de dados ou ao Redis, movimentação lateral de atacantes e comprometimento da infraestrutura.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Segurança de Rede / Cloud Infrastructure
* **Como implementar**: Criar Security Groups / tabelas de roteamento na nuvem permitindo que apenas a DMZ acesse a porta do n8n, e que o n8n acesse apenas as portas 5432 (PostgreSQL) e 6379 (Redis) na Zona de Dados.
* **Como verificar**: Tentar conectar diretamente aos endereços IP do PostgreSQL e Redis a partir de uma origem externa à rede privada.
* **Evidência esperada**: Diagrama de topologia de rede e regras de Security Group / Firewall exportadas da nuvem.
* **Frequência de revisão**: Semestral
* **Fonte**: SP 800-207, Zero Trust Architecture | CSRC; Arquitetura e Endurecimento Markdown. [INFERÊNCIA]

### NET-03: Proteção Nativa contra SSRF (`N8N_ENABLE_SSRF_PROTECTION=true`)
* **ID**: NET-03
* **Controle**: Habilitação do Filtro de Requisições Server-Side Request Forgery
* **Descrição**: Ativar o mecanismo interno do n8n que valida e bloqueia qualquer tentativa de requisição efetuada por nós de integração (como *HTTP Request*) direcionada a faixas de IP privadas locais, endereços de loopback ou serviços de metadados da nuvem.
* **Risco mitigado**: Leitura não autorizada da interface de metadados da nuvem (`169.254.169.254`), exfiltração de chaves IAM do nó e acesso a portas internas da infraestrutura corporativa.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / SecOps
* **Como implementar**: Definir a variável de ambiente `N8N_ENABLE_SSRF_PROTECTION=true` no orquestrador do n8n.
* **Como verificar**: Criar um workflow de teste com o nó *HTTP Request* fazendo requisição para `http://169.254.169.254` ou `http://127.0.0.1:5432` e confirmar a rejeição do disparo pelo n8n.
* **Evidência esperada**: Log de execução do workflow de teste exibindo o bloqueio por política de SSRF.
* **Frequência de revisão**: A cada atualização / Trimestral
* **Fonte**: n8n Security Vulnerabilities Whitepaper; Secure Your n8n Instance (VPS US).

### NET-04: Restrição de Egress e Firewall de Saída
* **ID**: NET-04
* **Controle**: Filtragem de Conexões de Saída da Instância do n8n
* **Descrição**: Aplicar regras de firewall de saída (*Egress Rules*) no nível do sistema operacional, contêiner ou Egress Gateway, bloqueando todas as conexões iniciadas pelo n8n para a internet, exceto para as portas HTTPS e FQDNs dos SaaS explicitamente autorizados.
* **Risco mitigado**: Comunicação do contêiner com servidores de Comando e Controle (C2) de atacantes, exfiltração não autorizada de dados e ataques de scanning de rede a partir do n8n.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: Segurança de Rede / DevOps
* **Como implementar**: Configurar regras de iptables, AWS Security Groups de saída ou Egress Proxy (Squid/Istio) permitindo tráfego de saída apenas para a lista de domínios das APIs corporativas e SaaS utilizados.
* **Como verificar**: Executar comando de teste a partir do contêiner tentando conectar a um IP externo arbitrário na porta 80/443 não autorizada.
* **Evidência esperada**: Regras do Firewall de Egress/Proxy exportadas e ativas.
* **Frequência de revisão**: Trimestral
* **Fonte**: SP 800-207, Zero Trust Architecture | CSRC; Arquitetura e Endurecimento Markdown. [INFERÊNCIA]

### NET-05: Isolamento de Rede no Kubernetes via NetworkPolicies (Default-Deny)
* **ID**: NET-05
* **Controle**: Aplicação de Políticas de Microsegmentação no Namespace do K8s
* **Descrição**: Implementar um objeto `NetworkPolicy` no Kubernetes para o namespace do n8n aplicando uma política padrão de negação total (*Default-Deny-All*) para Ingress e Egress, liberando apenas o tráfego estritamente necessário.
* **Risco mitigado**: Movimentação lateral de um pod do n8n comprometido para outros pods e serviços no cluster Kubernetes.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DevOps / Kubernetes Admin / SecOps
* **Como implementar**: Aplicar manifestos de `NetworkPolicy` liberando Ingress apenas do Ingress Controller (porta 5678) e comunicação interna n8n-Runner (porta 5679), e Egress apenas para o CoreDNS (porta 53), PostgreSQL (5432) e Redis (6379).
* **Como verificar**: Executar `kubectl exec` no pod do n8n e tentar dar `ping` ou `curl` no IP de um pod em outro namespace.
* **Evidência esperada**: Manifesto `NetworkPolicy` aplicado e confirmado via `kubectl get netpol -n n8n`.
* **Frequência de revisão**: Semestral
* **Fonte**: Network Policies | Kubernetes; Pod Security Standards | Kubernetes. [INFERÊNCIA]

---

## 6. Application Security (APP)

### APP-01: Desativação do Modo Inseguro de Avaliação de Expressões
* **ID**: APP-01
* **Controle**: Execução Segura do Motor de Expressões Javascript
* **Descrição**: Garantir que o motor de avaliação de expressões do n8n opere em modo seguro, impedindo o uso de contextos não isolados na interpretação de código dentro de chaves `{{ }}`.
* **Risco mitigado**: Injeção de código e execução remota de comandos via expressões maliciosas (*CVE-2025-68613*).
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / SecOps
* **Como implementar**: Garantir que a variável `N8N_RUNNERS_INSECURE_MODE=false` esteja configurada em todas as instâncias e Workers.
* **Como verificar**: Verificar o valor da variável de ambiente no contêiner do n8n orquestrador.
* **Evidência esperada**: Tabela de variáveis de ambiente do processo confirmando `N8N_RUNNERS_INSECURE_MODE=false`.
* **Frequência de revisão**: A cada atualização / Mensal
* **Fonte**: v2.0 Breaking changes - n8n Docs; CodeAnt AI.

### APP-02: Bloqueio do Acesso a Arquivos Internos do n8n (`N8N_BLOCK_FILE_ACCESS_TO_N8N_FILES=true`)
* **ID**: APP-02
* **Controle**: Restrição de Leitura de Arquivos de Configuração pelo Engine
* **Descrição**: Impedir que nós de leitura de arquivos ou scripts tenham acesso aos arquivos internos da própria aplicação n8n (como a pasta `~/.n8n/`, banco SQLite interno ou arquivos `.env`).
* **Risco mitigado**: Leitura não autorizada do arquivo de configuração `config`, extração da chave mestra do disco local e acesso ao banco de dados SQLite residual.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / SecOps
* **Como implementar**: Definir a variável de ambiente `N8N_BLOCK_FILE_ACCESS_TO_N8N_FILES=true`.
* **Como verificar**: Criar um workflow de teste com o nó *Read/Write Files from Disk* tentando ler o arquivo `/home/node/.n8n/config` e verificar a mensagem de acesso negado.
* **Evidência esperada**: Log de execução do workflow comprovando a rejeição do acesso ao arquivo interno.
* **Frequência de revisão**: Semestral
* **Fonte**: Secure Your n8n Instance (VPS US); v2.0 Breaking changes - n8n Docs.

### APP-03: Restrição do Sistema de Arquivos ao Diretório Permitido (`N8N_RESTRICT_FILE_ACCESS_TO`)
* **ID**: APP-03
* **Controle**: Limitando o Acesso de Escrita e Leitura de Arquivos Locais (Chroot Lógico)
* **Descrição**: Restringir a operação de todos os nós que manipulam o sistema de arquivos local estritamente a um diretório específico e isolado (ex: `/home/node/.n8n-files`).
* **Risco mitigado**: Vulnerabilidades de *Path Traversal* (como a CVE-2026-21877 no nó Git), leitura arbitrária do sistema de arquivos e sobrescrita de binários do sistema operacional.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / SecOps
* **Como implementar**: Configurar a variável `N8N_RESTRICT_FILE_ACCESS_TO=/home/node/.n8n-files` e garantir que o diretório possua as permissões de pasta adequadas.
* **Como verificar**: Tentar ler o arquivo `/etc/passwd` via nó de arquivo no n8n e confirmar que a operação é bloqueada pelo filtro de diretório.
* **Evidência esperada**: Log de teste comprovando o bloqueio de acesso fora da pasta permitida.
* **Frequência de revisão**: Semestral
* **Fonte**: v2.0 Breaking changes - n8n Docs; Security Advisory (n8n Blog).

### APP-04: Bloqueio de Nós de Alto Risco (`N8N_NODES_DENYLIST`)
* **ID**: APP-04
* **Controle**: Remoção de Nós de Execução de Comandos do Sistema e Acesso Shell
* **Descrição**: Desativar e proibir o carregamento de nós nativos que concedem acesso direto ao shell do sistema operacional ou execução de comandos arbitrários no servidor hospedeiro.
* **Risco mitigado**: Execução remota de comandos (RCE) direta por usuários com permissão de edição de workflows sem necessidade de explorar qualquer falha de software.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / SecOps
* **Como implementar**: Configurar a variável `N8N_NODES_DENYLIST=["n8n-nodes-base.executeCommand", "n8n-nodes-base.ssh"]` ou utilizar a variável legada `NODES_EXCLUDE`.
* **Como verificar**: Abrir o editor do n8n e buscar pelos nós "Execute Command" e "SSH", confirmando que não aparecem no menu de adição de nós.
* **Evidência esperada**: Relatório do `n8n audit` confirmando a ausência dos nós da denylist na instância.
* **Frequência de revisão**: Trimestral
* **Fonte**: Run security audits - n8n Docs; Secure Your n8n Instance (VPS US).

### APP-05: Desativação da API Pública Não Utilizada
* **ID**: APP-05
* **Controle**: Desativação do Endpoint da API REST Externa
* **Descrição**: Desativar a API pública do n8n (`/api/v1`) caso a organização não utilize automações externas para criação e gestão programática de workflows e usuários via API.
* **Risco mitigado**: Redução da superfície de ataque, mitigação de tentativas de força bruta em chaves API e prevenção de enumeração de dados da instância.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / SecOps
* **Como implementar**: Configurar a variável de ambiente `N8N_PUBLIC_API_DISABLED=true`.
* **Como verificar**: Realizar requisição HTTP GET para a URL `https://n8n.empresa.com/api/v1/workflows` e confirmar o retorno de erro 404/403.
* **Evidência esperada**: Resposta cURL comprovando o bloqueio do endpoint da API pública.
* **Frequência de revisão**: Semestral
* **Fonte**: Secure Your n8n Instance (VPS US); v2.0 Breaking changes - n8n Docs.

---

## 7. Workflow Security (WKF)

### WKF-01: Separação de Papéis de Criação e Publicação de Workflows
* **ID**: WKF-01
* **Controle**: Segregação de Funções no Ciclo de Vida do Workflow
* **Descrição**: Separar as permissões de usuários que podem criar/editar rascunhos de workflows daqueles que possuem autorização para ativar e publicar workflows em ambiente de produção.
* **Risco mitigado**: Implantação não autorizada de automações maliciosas ou não testadas, desvio de governança e alterações em produção por desenvolvedores júniores.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / Governança de TI
* **Como implementar**: Configurar papéis no RBAC do n8n onde desenvolvedores possuem permissão de edição em projetos de Staging, mas apenas administradores/líderes técnicos possuem permissão de publicação em produção.
* **Como verificar**: Tentar ativar um workflow com um usuário com papel de 'Editor' básico e verificar se o botão de ativação permanece desabilitado.
* **Evidência esperada**: Matriz de papéis e permissões do n8n validada.
* **Frequência de revisão**: Semestral
* **Fonte**: Set permissions and roles (RBAC) - n8n Docs; n8n Security Guide.

### WKF-02: Inspeção Visual de Alterações (Visual Diff) e Historização
* **ID**: WKF-02
* **Controle**: Rastreabilidade e Auditoria Visual de Modificações no Canvas
* **Descrição**: Utilizar o recurso de histórico de versões e comparação visual de alterações (*Visual Diff*) para revisar todas as modificações realizadas nos nós e conexões de um workflow antes de aprovar a publicação.
* **Risco mitigado**: Inserção silenciosa de nós maliciosos de exfiltração de dados, alteração oculta de parâmetros de destino e erros de lógica imperceptíveis.
* **Prioridade**: P1
* **Tipo**: Detectivo
* **Responsável**: Liderança Técnica / Revisores de Código
* **Como implementar**: Exigir o uso da ferramenta de *Visual Diff* (Enterprise) durante o processo de revisão de alterações entre a versão salva e a versão em publicação.
* **Como verificar**: Inspecionar o histórico de versões do workflow na interface confirmando os registros de comparação e o nome do autor da alteração.
* **Evidência esperada**: Captura de tela do histórico de versões com as alterações destacadas e aprovadas.
* **Frequência de revisão**: A cada alteração
* **Fonte**: n8n-docs/docs/changelog/README.md; Set permissions and roles - n8n Docs.

### WKF-03: Parametrização Estrita de Consultas SQL (SQL Injection Prevention)
* **ID**: WKF-03
* **Controle**: Prevenção de Injeção de SQL em Nós de Banco de Dados
* **Descrição**: Proibir categoricamente a concatenação direta de strings ou interpolação de variáveis na construção de consultas SQL dentro dos nós de banco de dados (PostgreSQL, MySQL, MS SQL), exigindo o uso exclusivo de parâmetros e consultas preparadas (*Prepared Statements*).
* **Risco mitigado**: Injeção de SQL (SQLi), destruição de tabelas, exfiltração de dados e alteração não autorizada de registros nos bancos de dados corporativos.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Desenvolvedores de Workflows / SecOps
* **Como implementar**: Utilizar os campos de parâmetros nativos dos nós SQL do n8n (sintaxe `$1`, `$2` ou objetos de parâmetros do nó) em vez de concatenar expressões `{{ $json.user_input }}` diretamente na query SQL.
* **Como verificar**: Executar o comando `n8n audit` que varre automaticamente a instância buscando consultas SQL não parametrizadas.
* **Evidência esperada**: Relatório do `n8n audit` confirmando zero alertas de SQLi não parametrizado.
* **Frequência de revisão**: A cada alteração / Trimestral
* **Fonte**: Run security audits - n8n Docs; Secure Your n8n Instance (VPS US).

### WKF-04: Validação de Entrada e Regra "Only Run If" no Canvas
* **ID**: WKF-04
* **Controle**: Filtragem e Validação de Carga Útil na Entrada do Workflow
* **Descrição**: Inserir nós de validação de dados (`If`, `Switch` ou `Edit Fields`) imediatamente após os nós de gatilho (Webhook, Form, e-mail), aplicando a regra "Only run if" para descartar dados malformados ou requisições suspeitas antes de iniciar o processamento pesado.
* **Risco mitigado**: Processamento de cargas maliciosas, envenenamento de dados, estouro de cotas de APIs e ataques de Negação de Serviço Lógica.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: Desenvolvedores de Workflows
* **Como implementar**: Configurar a opção `Only run if` nas configurações do nó de gatilho ou inserir um nó `If` no início do fluxo validando tipos de dados, tamanhos e presença de campos obrigatórios.
* **Como verificar**: Disparar o workflow com cargas inválidas ou incompletas e verificar no histórico que a execução é encerrada na primeira etapa sem acionar nós subsequentes.
* **Evidência esperada**: Estrutura do workflow no editor comprovando o nó de validação inicial.
* **Frequência de revisão**: A cada criação de workflow
* **Fonte**: n8n Security Guide; API Authentication and Security (Serenichron). [INFERÊNCIA]

---

## 8. Webhook Security (WHK)

### WHK-01: Autenticação Obrigatória em Endpoints de Webhook
* **ID**: WHK-01
* **Controle**: Proteção de Acesso aos Endpoints de Gatilho HTTP
* **Descrição**: Configurar obrigatoriamente um método de autenticação (Header Auth, Basic Auth ou Bearer Token) em todos os nós de Webhook expostos publicamente, proibindo o uso da opção "Authentication: None" em produção.
* **Risco mitigado**: Disparo não autorizado de automações, execução indevida de processos de negócio, exfiltração de dados por chamadas não autenticadas e flooding.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Desenvolvedores de Workflows / SecOps
* **Como implementar**: Alterar a propriedade "Authentication" do nó de Webhook para "Header Auth" ou "Basic Auth", vinculando-o a uma credencial com chave/token de alta entropia.
* **Como verificar**: Executar `n8n audit` para listar todos os webhooks desprotegidos ativas na instância.
* **Evidência esperada**: Relatório do `n8n audit` sem registros de webhooks não autenticadas em produção.
* **Frequência de revisão**: Trimestral
* **Fonte**: Run security audits - n8n Docs; Secure Your n8n Instance (VPS US).

### WHK-02: Validação de Assinaturas HMAC e Controle de Replay Timestamps
* **ID**: WHK-02
* **Controle**: Autenticidade e Integridade de Chamadas de Webhook de Terceiros
* **Descrição**: Validar a assinatura criptográfica HMAC-SHA256 presente no cabeçalho dos webhooks enviados por provedores SaaS (Stripe, GitHub, Shopify, Typeform) e verificar a janela do timestamp do evento (< 300 segundos).
* **Risco mitigado**: Falsificação de chamadas de webhook por atacantes, adulteração de payloads em trânsito e ataques de repetição (*Replay Attacks*).
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Desenvolvedores de Workflows / SecOps
* **Como implementar**: Utilizar a validação de assinatura nativa do nó de webhook ou inserir um nó de código no início do fluxo calculando `crypto.createHmac('sha256', secret).update(rawBody).digest('hex')` e comparando com o cabeçalho recebido.
* **Como verificar**: Enviar um payload de teste com assinatura HMAC alterada e confirmar que o workflow rejeita o processamento com erro 401/403.
* **Evidência esperada**: Código do workflow demonstrando a etapa de validação HMAC e logs de erro para assinaturas inválidas.
* **Frequência de revisão**: Semestral
* **Fonte**: API Authentication and Security (Serenichron); n8n Security Guide.

### WHK-03: Filtragem de IP (Allowlist CIDR) e Rate Limiting Perimetral no WAF
* **ID**: WHK-03
* **Controle**: Restrição de Origem e Limitação de Taxa para Webhooks
* **Descrição**: Aplicar regras de filtragem de IP no Proxy Reverso/WAF restringindo o acesso aos URLs de webhooks apenas aos blocos de IP oficiais dos fornecedores do SaaS (quando fixos) e impor limites rígidos de taxa de requisições por IP (*Rate Limiting*).
* **Risco mitigado**: Ataques de Negação de Serviço (DDoS), inundações por bots e varreduras automatizadas na porta de webhooks.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: Segurança de Rede / DevOps
* **Como implementar**: Configurar módulos de Rate Limiting no Nginx (`limit_req_zone`) ou regras de WAF/Cloudflare limitando requisições na rota `/webhook/*` a valores compatíveis com a operação normal (ex: 10 req/s por IP).
* **Como verificar**: Executar testes de estresse com a ferramenta `ab` ou `k6` disparando requisições em massa contra a URL do webhook e confirmando o bloqueio com código HTTP 429 (Too Many Requests).
* **Evidência esperada**: Configuração do WAF/Proxy e relatório de teste de carga confirmando o bloqueio por Rate Limiting.
* **Frequência de revisão**: Semestral
* **Fonte**: Security Advisory (n8n Blog); Secure Your n8n Instance (VPS US). [INFERÊNCIA]

---

## 9. Data Protection (DTP)

### DTP-01: Ativação do Redator de Dados de Execução (Execution Data Redaction)
* **ID**: DTP-01
* **Controle**: Ocultação de Dados Sensíveis na Interface de Histórico de Execuções
* **Descrição**: Ativar a funcionalidade de *Execution Data Redaction* para workflows que processam Dados Pessoais (PII), informações financeiras ou segredos corporativos, omitindo os payloads da visualização padrão na UI.
* **Risco mitigado**: Exposição indevida de dados pessoais e segredos para operadores e desenvolvedores que visualizam o histórico de execuções na interface gráfica do n8n.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / DPO
* **Como implementar**: Configurar a opção de redação de dados de execução nas propriedades globais da instância ou nas configurações individuais do workflow (Enterprise).
* **Como verificar**: Abrir o histórico de execuções de um workflow com dados sensíveis com uma conta de usuário padrão e verificar que os dados aparecem ocultos com asteriscos/mascarados.
* **Evidência esperada**: Captura de tela do histórico de execuções demonstrando o mascaramento de dados ativado.
* **Frequência de revisão**: Trimestral
* **Fonte**: n8n-docs/docs/changelog/README.md; Security at n8n.

### DTP-02: Política Agressiva de Expurgamento de Execuções (`EXECUTIONS_DATA_MAX_AGE`)
* **ID**: DTP-02
* **Controle**: Limpeza Automática do Histórico de Dados de Execução no Banco
* **Descrição**: Configurar a retenção do histórico de execuções no banco de dados para o menor período necessário à operação (ex: 7 a 14 dias ou 168 horas), executando a rotina de expurgamento automático (*Pruning*).
* **Risco mitigado**: Acúmulo desnecessário de dados pessoais (violação do princípio de limitação do armazenamento da LGPD), crescimento excessivo do banco de dados e vazamento massivo de histórico em caso de invasão.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / Database Admin
* **Como implementar**: Configurar as variáveis de ambiente `EXECUTIONS_DATA_PRUNE=true`, `EXECUTIONS_DATA_MAX_AGE=168` (em horas) e `EXECUTIONS_DATA_PRUNE_MAX_COUNT=50000`.
* **Como verificar**: Consultar a tabela `execution_entity` no banco de dados e verificar se não existem registros com data de criação superior ao limite de dias configurado.
* **Evidência esperada**: Query SQL executada no PostgreSQL confirmando a ausência de registros antigos.
* **Frequência de revisão**: Mensal
* **Fonte**: Secure Your n8n Instance (VPS US); LGPD — Lei Geral de Proteção de Dados Pessoais.

### DTP-03: Criptografia de Disco em Repouso (AES-256) no SGBD e Storage
* **ID**: DTP-03
* **Controle**: Proteção de Dados em Repouso no Armazenamento Físico
* **Descrição**: Garantir que as partições de disco que hospedam o banco de dados PostgreSQL, os volumes de dados do n8n e os backups estejam criptografados com algoritmo AES-256 no nível de infraestrutura/nuvem.
* **Risco mitigado**: Furto físico de discos, acesso não autorizado a snapshots de volumes na nuvem e descarte inadequado de mídias de armazenamento.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Cloud Infrastructure / Database Admin
* **Como implementar**: Ativar a opção de criptografia de volume (ex: AWS EBS Encryption, GCP Persistent Disk Encryption, LUKS em bare-metal) com chaves gerenciadas por KMS corporativo.
* **Como verificar**: Consultar as propriedades do volume de armazenamento no painel do provedor de nuvem e confirmar o status "Encrypted: True".
* **Evidência esperada**: Relatório de configuração do provedor de nuvem comprovando a criptografia ativa nos volumes de dados.
* **Frequência de revisão**: Semestral
* **Fonte**: SP 800-207, Zero Trust Architecture | CSRC; LGPD — Lei Geral de Proteção de Dados Pessoais.

### DTP-04: Sanitização e Ocultação de PII no Canvas de Automação
* **ID**: DTP-04
* **Controle**: Minimização e Remoção Contínua de Dados Pessoais nos Fluxos
* **Descrição**: Inserir nós de transformação de dados (`Edit Fields / Set` ou `Remove Keys`) nos workflows para descartar ou anonimizar campos com dados pessoais (CPF, cartões de crédito, senhas) imediatamente após o uso, evitando que trafeguem para nós subsequentes.
* **Risco mitigado**: Mapeamento e gravação desnecessária de PII em logs de sistemas secundários, descumprimento do princípio da minimização da LGPD e vazamentos indesejados.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: Desenvolvedores de Workflows / DPO
* **Como implementar**: Configurar os nós de transformação utilizando a opção "Keep Only Set" para manter estritamente os atributos necessários para os passos seguintes do fluxo.
* **Como verificar**: Inspecionar os payloads do fluxo no editor e confirmar que campos não essenciais são removidos na primeira etapa.
* **Evidência esperada**: Estrutura do workflow revisada e aprovada pelo DPO/SecOps.
* **Frequência de revisão**: Semestral
* **Fonte**: Lei Geral de Proteção de Dados Pessoais (LGPD); Privacy Policy (n8n). [INFERÊNCIA]

---

## 10. Container / Infrastructure Security (INF)

### INF-01: Execução com Usuários Não-Root (`node` UID 1000 e `nobody` UID 65532)
* **ID**: INF-01
* **Controle**: Desqualificação de Privilégios do Usuário do Contêiner
* **Descrição**: Garantir que nenhum contêiner do n8n ou dos Task Runners seja executado sob o usuário `root` (UID 0), forçando a execução com usuários do sistema sem privilégios administrativos.
* **Risco mitigado**: Escalação de privilégios para o hospedeiro, alteração de configurações do sistema operacional e mitigação de vulnerabilidades de escape de contêiner.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DevOps / SecOps
* **Como implementar**: Definir `user: "1000:1000"` no Docker Compose/K8s para a imagem `n8nio/n8n` e `user: "65532:65532"` (usuário `nobody`) para a imagem `n8nio/runners`.
* **Como verificar**: Executar `docker exec <container_id> id` ou `kubectl exec` e verificar que o UID retornado é diferente de 0.
* **Evidência esperada**: Saída do comando `id` dentro do contêiner confirmando UID 1000 ou 65532.
* **Frequência de revisão**: A cada deploy / Trimestral
* **Fonte**: Harden task runners - n8n Docs; Docker Engine security | Docker Docs.

### INF-02: Execução de Task Runners em Modo Externo (`N8N_RUNNERS_MODE=external`) com Imagem Distroless
* **ID**: INF-02
* **Controle**: Desacoplamento do Processo de Execução de Código do Orquestrador
* **Descrição**: Isolar completamente a execução de nós de código (JavaScript e Python) em contêineres *sidecar* dedicados rodando em modo externo, utilizando a variante de imagem `n8nio/runners:<ver>-distroless`.
* **Risco mitigado**: Vulnerabilidades críticas de RCE e sandbox escape (CVE-2025-68668 N8Scape e CVE-2026-42234), impedindo que um código malicioso acesse o processo orquestrador do n8n.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DevOps / SecOps
* **Como implementar**: Configurar `N8N_RUNNERS_MODE=external` no orquestrador n8n e implantar o contêiner sidecar com a imagem `n8nio/runners:<ver>-distroless`, comunicando-se via porta interna 5679 com token de autenticação.
* **Como verificar**: Verificar se o contêiner do runner está rodando separadamente e testar se a imagem não possui binários de shell (`docker exec` no runner tentando rodar `/bin/sh` deve falhar).
* **Evidência esperada**: Configuração do manifesto Docker/K8s e teste de falha ao tentar invocar shell no runner.
* **Frequência de revisão**: A cada deploy / Mensal
* **Fonte**: Set up task runners - n8n Docs; Harden task runners - n8n Docs; N8Scape (CVE-2025-68668).

### INF-03: Sistema de Arquivos Raiz Somente Leitura (`readOnlyRootFilesystem: true`)
* **ID**: INF-03
* **Controle**: Imutabilidade do Sistema de Arquivos do Contêiner
* **Descrição**: Configurar o sistema de arquivos raiz do contêiner do *Task Runner* para ser montado em modo estritamente somente leitura, fornecendo armazenamento temporário exclusivo via volume em memória `emptyDir` em `/tmp`.
* **Risco mitigado**: Gravação de malware, criação de scripts de persistência no sistema de arquivos do contêiner e alteração de bibliotecas de execução por atacantes.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DevOps / SecOps
* **Como implementar**: Configurar `readOnlyRootFilesystem: true` na especificação de segurança do contêiner do runner no Kubernetes ou `--read-only` no Docker.
* **Como verificar**: Tentar criar um arquivo no diretório raiz do contêiner runner (`touch /test.txt`) e verificar a mensagem "Read-only file system".
* **Evidência esperada**: Saída do teste de tentativa de escrita e manifesto de configuração do contêiner.
* **Frequência de revisão**: A cada deploy / Semestral
* **Fonte**: Harden task runners - n8n Docs; Pod Security Standards | Kubernetes.

### INF-04: Confinamento por Perfil AppArmor e Descarte Total de Capabilities
* **ID**: INF-04
* **Controle**: Restrição de Chamadas de Sistema do Kernel
* **Descrição**: Descartar todas as capabilities do Linux (`--cap-drop=ALL`) nos contêineres e aplicar um perfil AppArmor restritivo bloqueando a leitura dos diretórios de memória `/proc/*/environ` e `/proc/*/mounts`.
* **Risco mitigado**: Leitura de variáveis de ambiente do processo diretamente da memória do kernel, bypass de sandbox e tentativas de exploração do kernel do hospedeiro.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DevOps / SecOps
* **Como implementar**: Aplicar o perfil AppArmor recomendado na documentação oficial do n8n contendo a regra `audit deny @{PROC}/*/{environ,mounts} rwl,` e adicionar `capabilities: { drop: ["ALL"] }` no manifesto do contêiner.
* **Como verificar**: Inspecionar as propriedades do pod/contêiner confirmando a remoção de capabilities e verificar o status do AppArmor via `aa-status` no host.
* **Evidência esperada**: Saída do comando `aa-status` exibindo o perfil ativo para o contêiner do n8n runner.
* **Frequência de revisão**: Semestral
* **Fonte**: Harden task runners - n8n Docs; Docker Engine security | Docker Docs.

### INF-05: Proibição Absoluta da Montagem do Socket do Docker (`/var/run/docker.sock`)
* **ID**: INF-05
* **Controle**: Bloqueio de Acesso ao Daemon de Contêineres do Hospedeiro
* **Descrição**: Proibir terminantemente o mapeamento do arquivo de socket do Docker (`/var/run/docker.sock`) para dentro de qualquer contêiner do n8n ou dos Task Runners.
* **Risco mitigado**: Escape imediato de contêiner para o hospedeiro com privilégios equivalentes a `root`, criação de contêineres maliciosos no servidor e comprometimento total do nó de infraestrutura.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DevOps / SecOps
* **Como implementar**: Auditar todos os arquivos `docker-compose.yml` e manifestos do Kubernetes garantindo a ausência de montagens do tipo `hostPath` apontando para `/var/run/docker.sock`.
* **Como verificar**: Executar verificação estática nos manifestos de implantação via linter/Kyverno buscando por regras que bloqueiem `docker.sock`.
* **Evidência esperada**: Relatório do linter/controle de admissão confirmando a ausência da montagem do socket.
* **Frequência de revisão**: A cada deploy / Contínuo
* **Fonte**: Harden task runners - n8n Docs; Docker Engine security | Docker Docs.

### INF-06: Desativação do Automount de Token de ServiceAccount no Kubernetes
* **ID**: INF-06
* **Controle**: Remoção de Credenciais de Acesso à API Server do K8s
* **Descrição**: Desativar a montagem automática de tokens da `ServiceAccount` dentro dos pods do n8n e dos Task Runners no Kubernetes.
* **Risco mitigado**: Roubo do token JWT do Kubernetes por um invasor com RCE no pod, prevenindo tentativas de autenticação e tomada de controle do API Server do cluster.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Kubernetes Admin / SecOps
* **Como implementar**: Configurar `automountServiceAccountToken: false` no manifesto do objeto `ServiceAccount` e na especificação do Pod do n8n.
* **Como verificar**: Verificar que o diretório `/var/run/secrets/kubernetes.io/serviceaccount/` não existe dentro do pod do n8n.
* **Evidência esperada**: Saída do comando `kubectl exec` confirmando que o caminho de segredos do K8s não está montado no pod.
* **Frequência de revisão**: A cada deploy / Semestral
* **Fonte**: Security | Kubernetes; Pod Security Standards | Kubernetes. [INFERÊNCIA]

### INF-07: Definição Rígida de Limites de Recursos (CPU e Memória)
* **ID**: INF-07
* **Controle**: Alocação Controlada de Recursos para Prevenção de DoS
* **Descrição**: Definir limites máximos e solicitações mínimas de CPU e Memória RAM (`requests` e `limits`) para todos os contêineres do n8n, Workers e Task Runners.
* **Risco mitigado**: Esgotamento de memória do servidor (*Node OOM*), travamento do servidor por laços infinitos em scripts e ataques de Negação de Serviço por consumo excessivo de recursos.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DevOps / Kubernetes Admin
* **Como implementar**: Configurar o bloco `resources: requests: {cpu: "500m", memory: "1Gi"}, limits: {cpu: "2000m", memory: "2Gi"}` no manifesto do contêiner.
* **Como verificar**: Executar `kubectl top pods` ou `docker stats` para verificar se os contêineres estão operando dentro das cotas estabelecidas.
* **Evidência esperada**: Dashboard do Prometheus/Grafana monitorando o consumo de recursos contra os limites definidos.
* **Frequência de revisão**: Trimestral
* **Fonte**: Pod Security Standards | Kubernetes; Secure Your n8n Instance (VPS US).

---

## 11. Supply Chain Security (SCM)

### SCM-01: Uso Exclusivo de Tags de Imagem Imutáveis e Fixadas por Versão Exata
* **ID**: SCM-01
* **Controle**: Banimento de Tags Flutuantes em Imagens de Produção
* **Descrição**: Proibir o uso da tag flutuante `:latest` ou tags de versão minoritária alteráveis na implantação do n8n, exigindo a fixação da versão exata e idêntica para o n8n e o Task Runner (ex: `n8nio/n8n:2.38.5` e `n8nio/runners:2.38.5-distroless`).
* **Risco mitigado**: Introdução não testada de atualizações de software com quebras de compatibilidade, alterações não auditadas na imagem base e comportamentos imprevisíveis.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DevOps / SecOps
* **Como implementar**: Configurar a tag exata no arquivo de implantação e utilizar verificadores de política (como Kyverno/Conftest) no CI/CD para rejeitar manifestos com a tag `:latest`.
* **Como verificar**: Inspecionar a lista de imagens em execução no cluster via `kubectl get pods -o jsonpath='{.items[*].spec.containers[*].image}'`.
* **Evidência esperada**: Relatório do controle de admissão aprovando as tags de versão fixadas.
* **Frequência de revisão**: A cada deploy / Mensal
* **Fonte**: Software Supply Chain Security - OWASP; Set up task runners - n8n Docs.

### SCM-02: Restrição e Governança Estrita para Instalação de Community Nodes
* **ID**: SCM-02
* **Controle**: Política de Aprovação e Isolamento de Nós da Comunidade
* **Descrição**: Proibir a instalação direta de *Community Nodes* vindos do registro público do npm por usuários comuns, estabelecendo uma esteira de avaliação de segurança, análise estática do código-fonte e teste em sandbox antes da autorização.
* **Risco mitigado**: Injeção de código malicioso por pacotes npm comprometidos (*Supply Chain Attacks*), trojans em dependências de terceiros e exfiltração silenciosa de credenciais.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Segurança de Aplicações / Admin n8n
* **Como implementar**: Restringir a permissão de instalação de pacotes no RBAC do n8n e estabelecer a *Política de Governança de Community Nodes*, exigindo a compilação prévia e verificação do pacote em registro npm privado corporativo.
* **Como verificar**: Executar `n8n audit` para extrair a lista de todos os *Community Nodes* instalados e verificar se possuem aprovação formal no inventário de segurança.
* **Evidência esperada**: Relatório do `n8n audit` com zero pacotes não autorizados e fichas de avaliação de código dos pacotes aprovados.
* **Frequência de revisão**: Trimestral
* **Fonte**: Software Supply Chain Security - OWASP; Run security audits - n8n Docs.

### SCM-03: Análise Estática (SAST) e Revisão de Código para Custom Nodes
* **ID**: SCM-03
* **Controle**: Homologação de Código Próprio Desenvolvido para o n8n
* **Descrição**: Submeter todo nó customizado (*Custom Node*) desenvolvido internamente pela organização a ferramentas de análise estática de segurança (SAST) e revisão de código por especialista em segurança antes de disponibilizá-lo no diretório de extensões do n8n.
* **Risco mitigado**: Introdução involuntária de vulnerabilidades de SQLi, Command Injection, gravação insegura de arquivos ou vazamentos de memória em nós customizados.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: Segurança de Aplicações / Engenharia de Software
* **Como implementar**: Incluir etapas automáticas de varredura com SonarQube/Semgrep na esteira de CI/CD do repositório de custom nodes e configurar o diretório isolado `N8N_CUSTOM_EXTENSIONS`.
* **Como verificar**: Consultar os relatórios da ferramenta SAST no repositório do nó customizado confirmando a ausência de vulnerabilidades de severidade Alta ou Crítica.
* **Evidência esperada**: Dashboard do SonarQube/Semgrep com aprovação técnica e historico de aprovações de Pull Request.
* **Frequência de revisão**: A cada alteração de código
* **Fonte**: Software Supply Chain Security - OWASP; Set up task runners - n8n Docs. [INFERÊNCIA]

### SCM-04: Verificação de Assinatura Criptográfica de Imagens (Cosign/Sigstore)
* **ID**: SCM-04
* **Controle**: Validação de Integridade e Autenticidade da Imagem do Contêiner
* **Descrição**: Configurar o orquestrador de contêineres para validar a assinatura criptográfica e a procedência das imagens do n8n e dos Task Runners via Cosign/Sigstore antes de autorizar a execução no ambiente produtivo.
* **Risco mitigado**: Substituição maliciosa da imagem do n8n por imagens adulteradas em registros de contêineres intermediários (*Image Spoofing/Tampering*).
* **Prioridade**: P2
* **Tipo**: Preventivo
* **Responsável**: DevOps / SecOps
* **Como implementar**: Configurar a política do Kyverno / Sigstore Policy Controller no Kubernetes para verificar a chave pública de assinatura do repositório oficial da n8n GmbH.
* **Como verificar**: Simular o deploy de uma imagem não assinada no cluster e verificar a rejeição automática pelo controlador de admissão.
* **Evidência esperada**: Logs do Policy Controller no Kubernetes confirmando a verificação de assinatura bem-sucedida.
* **Frequência de revisão**: Semestral
* **Fonte**: Software Supply Chain Security - OWASP; Security | Kubernetes. [INFERÊNCIA]

---

## 12. CI/CD Security (CICD)

### CICD-01: Exportação Segura de Workflows em Pacotes `.n8np` sem Segredos (Stubs)
* **ID**: CICD-01
* **Controle**: Sanitização Automática de Segredos na Exportação de Workflows
* **Descrição**: Garantir que todo processo de exportação programática de workflows para versionamento em repositórios Git utilize o formato oficial de pacotes do n8n (`.n8np`) ou comandos CLI que removem dados sensíveis, mantendo apenas referências (*stubs*) das credenciais.
* **Risco mitigado**: Inclusão acidental de chaves de API, senhas e tokens OAuth2 em texto claro nos arquivos JSON de workflows comitados em repositórios de código.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DevOps / Desenvolvedores de Workflows
* **Como implementar**: Utilizar a API oficial do n8n ou a CLI `n8n export:workflow` (sem a flag `--decrypted`), garantindo que o arquivo exportado contenha apenas o campo `credentials: { id: "123", name: "stripe-account" }` sem segredos brutos.
* **Como verificar**: Executar script de inspeção em busca de strings no formato de chaves de API conhecidas nos arquivos `.json`/`.n8np` do repositório Git.
* **Evidência esperada**: Arquivos de workflows no repositório Git auditados e aprovados pela ferramenta de verificação.
* **Frequência de revisão**: A cada commit / A cada deploy
* **Fonte**: n8n-docs/docs/changelog/README.md; CI CD Security - OWASP Cheat Sheet.

### CICD-02: Varredura Automática de Segredos em Repositórios (Gitleaks)
* **ID**: CICD-02
* **Controle**: Detecção de Vazamento de Credenciais na Esteira de Código
* **Descrição**: Integrar ferramentas de detecção automatizada de segredos (*Gitleaks*, *Git-Secrets*, *TruffleHog*) nos pipelines de CI/CD e em ganchos de pré-commit (*Pre-commit Hooks*) do repositório de infraestrutura e workflows.
* **Risco mitigado**: Exposição de credenciais da empresa, chaves `N8N_ENCRYPTION_KEY` ou tokens corporativos no histórico de commits do repositório Git.
* **Prioridade**: P0
* **Tipo**: Detectivo
* **Responsável**: SecOps / DevOps
* **Como implementar**: Adicionar uma etapa obrigatória no pipeline do GitHub Actions / GitLab CI executando `gitleaks detect --source . --verbose`, bloqueando o *merge* do Pull Request em caso de identificação de segredos.
* **Como verificar**: Criar um branch de teste com um token fictício e validar o bloqueio do Pull Request pela ferramenta de varredura.
* **Evidência esperada**: Logs de execução do pipeline do CI/CD comprovando a passagem da varredura sem alertas.
* **Frequência de revisão**: A cada Pull Request / Mensal
* **Fonte**: CI CD Security - OWASP Cheat Sheet; Secrets Management - OWASP Cheat Sheet.

### CICD-03: Implantação Automatizada com Aprovações Obrigatórias e PR Reviews
* **ID**: CICD-03
* **Controle**: Governança da Esteira de Deploy de Automações
* **Descrição**: Automatizar completamente a implantação de alterações no n8n via pipelines de CI/CD gerenciados (GitOps / ArgoCD / GitLab CI), proibindo deploys manuais e exigindo a aprovação de pelo menos dois revisores qualificados em todo Pull Request.
* **Risco mitigado**: Alterações não auditadas em ambiente de produção, injeção de pipelines maliciosos e contaminação de ambientes por deploys diretos.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: DevOps / SecOps
* **Como implementar**: Configurar proteção de branches principais (`main`/`production`) no repositório Git exigindo *PR Review* obrigatório e passar a chave de implantação via Service Account do pipeline.
* **Como verificar**: Tentar realizar um `git push` direto para o branch principal e confirmar a rejeição pelo servidor Git.
* **Evidência esperada**: Configuração de proteção de branch do repositório Git exportada.
* **Frequência de revisão**: Semestral
* **Fonte**: CI CD Security - OWASP Cheat Sheet; Set permissions and roles - n8n Docs. [INFERÊNCIA]

---

## 13. Logging & Monitoring (MON)

### MON-01: Streaming de Eventos de Auditoria em Tempo Real (`n8nEventLog.log`)
* **ID**: MON-01
* **Controle**: Envio Contínuo do Barramento de Logs de Auditoria para SIEM
* **Descrição**: Configurar o envio contínuo dos eventos do barramento de auditoria do n8n (*Event Bus*) para um sistema centralizado de SIEM/Gerenciamento de Logs (OpenObserve, Datadog, Splunk, Elastic) através do arquivo `n8nEventLog.log` ou Syslog TLS.
* **Risco mitigado**: Perda de trilhas de auditoria em caso de destruição do contêiner, cegueira operacional sobre ações de usuários e incapacidade de realizar perícia pós-incidente.
* **Prioridade**: P0
* **Tipo**: Detectivo
* **Responsável**: SecOps / DevOps
* **Como implementar**: Configurar a escrita de logs de auditoria em arquivo dedicado (`N8N_EVENTBUS_LOGWRITER_LOGFULLPATH=/var/log/n8n/n8nEventLog.log`) e integrar um agente de transporte (FluentBit/Logstash) enviando os dados via protocolo criptografado para o SIEM.
* **Como verificar**: Realizar uma ação auditada na interface (ex: login ou alteração de credencial) e consultar o evento correspondente no painel do SIEM dentro de 60 segundos.
* **Evidência esperada**: Dashboard do SIEM exibindo os eventos estruturados `n8n.audit.user.login`, `n8n.audit.workflow.updated`, etc.
* **Frequência de revisão**: Trimestral
* **Fonte**: Stream logs to external systems - n8n Docs; n8n Monitoring with OpenTelemetry.

### MON-02: Gerenciamento Centralizado de Log Level (`N8N_LOG_LEVEL=info`)
* **ID**: MON-02
* **Controle**: Padronização do Nível de Detalhamento dos Logs Operacionais
* **Descrição**: Definir e travar o nível de log operacional da aplicação como `info` ou `warn` em ambiente de produção, evitando o uso prolongado do nível `debug`.
* **Risco mitigado**: Gravação acidental de dados de cargas de trabalho, payloads completos de requisições e potenciais segredos em arquivos de log operacionais em disco.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DevOps / SecOps
* **Como implementar**: Configurar a variável de ambiente `N8N_LOG_LEVEL=info` e definir `N8N_LOG_STREAMING_MANAGED_BY_ENV=true` para impedir a alteração não autorizada do nível de log pela interface gráfica.
* **Como verificar**: Inspecionar os logs de execução e confirmar que não constam mensagens de nível `debug` no tráfego operacional comum.
* **Evidência esperada**: Tabela de variáveis do processo e amostra de logs de produção auditada.
* **Frequência de revisão**: Semestral
* **Fonte**: Stream logs to external systems - n8n Docs; Secure Your n8n Instance (VPS US).

### MON-03: Anonimização de Mensagens de Auditoria (`anonymizeAuditMessages=true`)
* **ID**: MON-03
* **Controle**: Mascaramento de Identificadores Pessoais nas Trilhas de Auditoria
* **Descrição**: Ativar a opção de anonimização nas mensagens do barramento de auditoria, substituindo nomes e e-mails de usuários por identificadores hashes/IDs de usuário nos logs enviados para coletores externos.
* **Risco mitigado**: Vazamento de dados pessoais de funcionários/operadores para sistemas de monitoramento e desconformidade com o princípio da minimização da LGPD nos logs.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: Admin n8n / DPO / SecOps
* **Como implementar**: Configurar o parâmetro `anonymizeAuditMessages=true` nas configurações do coletor de logs do n8n.
* **Como verificar**: Inspecionar uma mensagem do evento `n8n.audit.user.login` no SIEM e verificar que o campo de e-mail aparece anonimizado/hasheado.
* **Evidência esperada**: Registro de log formatado no SIEM comprovando o mascaramento de identificadores.
* **Frequência de revisão**: Semestral
* **Fonte**: Stream logs to external systems - n8n Docs; Privacy Policy (n8n).

### MON-04: Coleta de Métricas Prometheus e Rastreamento OpenTelemetry (W3C Traceparent)
* **ID**: MON-04
* **Controle**: Observabilidade Técnica de Infraestrutura e Propagação de Contexto
* **Descrição**: Habilitar a raspagem de métricas operacionais via endpoint do Prometheus (`/metrics`) e configurar a injeção do cabeçalho de rastreamento distribuído W3C (`traceparent`) nas chamadas HTTP realizadas pelo nó de integração.
* **Risco mitigado**: Falta de visibilidade sobre gargalos de desempenho, picos de erros e falta de rastreabilidade de requisições que cruzam múltiplos microserviços corporativos.
* **Prioridade**: P1
* **Tipo**: Detectivo
* **Responsável**: DevOps / SRE
* **Como implementar**: Habilitar as variáveis `N8N_METRICS=true`, `N8N_METRICS_INCLUDE_QUEUE_METRICS=true` e proteger a rota `/metrics` com restrição de rede para consumo exclusivo do servidor Prometheus.
* **Como verificar**: Acessar o endpoint interno `/metrics` e verificar a presença de métricas do n8n como `n8n_workflows_execution_time_seconds`.
* **Evidência esperada**: Dashboard do Grafana ativo exibindo métricas de execuções do n8n e rastros no Jaeger/OpenTelemetry.
* **Frequência de revisão**: Trimestral
* **Fonte**: n8n Monitoring with OpenTelemetry and OpenObserve; Stream logs to external systems - n8n Docs.

---

## 14. Vulnerability Management (VUL)

### VUL-01: Execução Automática do Módulo de Auditoria (`n8n audit`)
* **ID**: VUL-01
* **Controle**: Verificação Automática de Configurações Inseguras da Instância
* **Descrição**: Agendar a execução periódica do comando interno `n8n audit` via linha de comando ou acionamento programado da API REST (`POST /audit`), gerando relatórios de diagnósticos sobre erros de configuração e nós de risco.
* **Risco mitigado**: Permanência de configurações inseguras ativas por tempo indeterminado, presença de webhooks não autenticados e uso de credenciais não utilizadas.
* **Prioridade**: P0
* **Tipo**: Detectivo
* **Responsável**: SecOps / Admin n8n
* **Como implementar**: Criar uma tarefa cron na infraestrutura ou um workflow administrativo de segurança que executa `n8n audit` semanalmente e envia o resultado em formato JSON para análise da equipe de SecOps.
* **Como verificar**: Consultar o histórico de execução da tarefa de auditoria e validar o conteúdo do relatório retornado.
* **Evidência esperada**: Arquivo JSON do relatório do `n8n audit` gerado na última semana e arquivado na pasta de conformidade.
* **Frequência de revisão**: Semanal
* **Fonte**: Run security audits - n8n Docs; Secure Your n8n Instance (VPS US).

### VUL-02: Varredura Contínua de Imagens e Dependências (Trivy/Grype)
* **ID**: VUL-02
* **Controle**: Análise Automática de Vulnerabilidades em Software e Contêineres
* **Descrição**: Executar ferramentas de varredura de vulnerabilidades de infraestrutura e componentes (*Trivy*, *Grype*, *Clair*, *Snyk*) em todas as imagens de contêineres do n8n e dos Task Runners mantidas no repositório local.
* **Risco mitigado**: Execução de binários ou bibliotecas com vulnerabilidades conhecidas e exploráveis (CVEs públicas registradas na NVD/GHSA).
* **Prioridade**: P0
* **Tipo**: Detectivo
* **Responsável**: SecOps / DevOps
* **Como implementar**: Configurar tarefas agendadas no registro de contêineres ou no cluster (Trivy Operator) para varrer as imagens ativas do n8n e alertar em caso de CVEs Críticas.
* **Como verificar**: Inspecionar o dashboard da ferramenta de varredura confirmando zero vulnerabilidades de severidade Crítica sem mitigação registrada.
* **Evidência esperada**: Relatório PDF/JSON emitido pelo Trivy/Grype atestando o status das imagens do n8n.
* **Frequência de revisão**: Semanal
* **Fonte**: Software Supply Chain Security - OWASP; Docker Engine security | Docker Docs. [INFERÊNCIA]

### VUL-03: Gerenciamento de Patches e Ciclo de Atualização do n8n
* **ID**: VUL-03
* **Controle**: Política de Atualização e Aplicação de Correções de Segurança
* **Descrição**: Estabelecer uma janela quinzenal ou mensal de atualização da versão da imagem do n8n e do Task Runner para aplicar correções de segurança lançadas pela n8n GmbH, com fluxo de atualização emergencial em até 48 horas para CVEs de severidade Crítica.
* **Risco mitigado**: Exploração de vulnerabilidades de software conhecidas já corrigidas pelo fornecedor em versões recentes.
* **Prioridade**: P0
* **Tipo**: Corretivo
* **Responsável**: DevOps / Admin n8n
* **Como implementar**: Acompanhar o canal oficial de notas de versão (*Changelog*) do n8n e avisos de segurança no blog oficial, executando a atualização em ambiente de Staging antes da promoção.
* **Como verificar**: Consultar a versão atual da imagem em execução no cluster e comparar com a última versão estável disponibilizada pela n8n GmbH.
* **Evidência esperada**: Histórico de deploys do cluster comprovando a atualização regular da imagem do n8n.
* **Frequência de revisão**: Quinzena / A cada boletim de segurança
* **Fonte**: n8n-docs/docs/changelog/README.md; Security Advisory (n8n Blog).

---

## 15. Backup & Disaster Recovery (BDR)

### BDR-01: Separação Física entre Dump do Banco de Dados e a Chave `N8N_ENCRYPTION_KEY`
* **ID**: BDR-01
* **Controle**: Isolamento de Armazenamento do Cofre Criptografado e da Chave Mestra
* **Descrição**: Proibir terminantemente o armazenamento do arquivo de dump do banco de dados relacional (PostgreSQL) e do arquivo contendo o valor da variável `N8N_ENCRYPTION_KEY` no mesmo bucket de armazenamento, diretório de disco ou servidor de backup.
* **Risco mitigado**: Vazamento massivo de todas as credenciais descriptografadas da organização caso o arquivo de backup do banco de dados seja interceptado ou acessado indevidamente.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Cloud Infrastructure / Database Admin / SecOps
* **Como implementar**: Armazenar os backups do PostgreSQL em um bucket de backup dedicado com controle de acesso restrito, e armazenar o registro da `N8N_ENCRYPTION_KEY` exclusivamente no Vault/KMS corporativo mantido por equipe distinta.
* **Como verificar**: Auditar as permissões e o conteúdo do bucket de backups do banco de dados confirmando a ausência da chave `N8N_ENCRYPTION_KEY` ou arquivos `.env`.
* **Evidência esperada**: Relatório de configuração de permissões dos buckets de backup e diretrizes do cofre de chaves.
* **Frequência de revisão**: Semestral
* **Fonte**: Rotate the n8n encryption key (LumaDock); Secrets Management - OWASP Cheat Sheet.

### BDR-02: Backup Criptografado (AES-256) e Imutável (WORM/Object Lock)
* **ID**: BDR-02
* **Controle**: Proteção e Retenção Segura de Cópias de Segurança
* **Descrição**: Criptografar todos os arquivos de backup do banco de dados e do sistema de arquivos do n8n antes do envio para a nuvem secundaria, ativando políticas de imutabilidade (*Object Lock / WORM*) nas cópias de segurança.
* **Risco mitigado**: Exposição do conteúdo do banco em caso de vazamento do arquivo de backup e destruição ou criptografia dos backups por ataques de ransomware.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Cloud Infrastructure / Backup Admin
* **Como implementar**: Configurar a ferramenta de backup (ex: pg_dump via pipeline criptografado com GPG/AES-256) enviando o arquivo para um bucket S3 com a opção "Object Lock" ativada com retenção mínima de 30 dias.
* **Como verificar**: Tentar alterar ou excluir um arquivo de backup armazenado no bucket de retenção e verificar a mensagem de bloqueio por política de imutabilidade.
* **Evidência esperada**: Logs da ferramenta de backup atestando a criptografia e status de Object Lock no bucket S3.
* **Frequência de revisão**: Trimestral
* **Fonte**: Secure Your n8n Instance (VPS US); Incident Response | CSRC. [INFERÊNCIA]

### BDR-03: Procedimento Homologado de Teste de Restauração (Disaster Recovery Test)
* **ID**: BDR-03
* **Controle**: Validação Periódica da Capacidade de Recuperação do n8n
* **Descrição**: Executar um procedimento formal de teste de restauração completa (*DR Restore Test*) em ambiente isolado a partir dos backups do banco de dados e da chave `N8N_ENCRYPTION_KEY`, validando a integridade das credenciais e a execução de workflows.
* **Risco mitigado**: Descobrir que os backups do banco de dados estão corrompidos, incompletos ou incompatíveis com a chave de criptografia no momento de um incidente real.
* **Prioridade**: P1
* **Tipo**: Corretivo
* **Responsável**: DevOps / Database Admin / SecOps
* **Como implementar**: Criar uma automação que lê o último backup do banco, sobre uma instância de testes limpa do n8n, injeta a chave de criptografia e executa um workflow de teste validando que as credenciais são descriptografadas com sucesso.
* **Como verificar**: Verificar a execução do workflow de teste na instância restaurada e o relatório de validação da recuperação.
* **Evidência esperada**: Relatório assinado de teste de Disaster Recovery homologado com o tempo total de recuperação (RTO/RPO mensurado).
* **Frequência de revisão**: Semestral
* **Fonte**: Set a custom encryption key - n8n Docs; SP 800-61 Rev. 3 | NIST. [INFERÊNCIA]

---

## 16. Incident Response (IR)

### IR-01: Plano de Resposta a Incidentes Alinhado ao NIST SP 800-61 Rev. 3
* **ID**: IR-01
* **Controle**: Procedimentos Formais de Contenção, Erradicação e Recuperação
* **Descrição**: Desenvolver e homologar o *Plano de Resposta a Incidentes para a Plataforma n8n*, definindo os passos para contenção de comprometimentos, análise forense, notificação de partes interessadas e recuperação do serviço.
* **Risco mitigado**: Ações desordenadas durante um incidente, prolongamento do tempo de exposição e incapacidade de conter a movimentação lateral de um atacante.
* **Prioridade**: P0
* **Tipo**: Corretivo
* **Responsável**: Equipe de Resposta a Incidentes (CSIRT) / SecOps
* **Como implementar**: Documentar os playbooks de resposta específicos para os cenários: Comprometimento de Credencial, RCE em Task Runner, Injeção de Código em Workflow e Vazamento de Dados Pessoais.
* **Como verificar**: Executar um exercício de simulação de incidente (*Tabletop Exercise*) testando a capacidade da equipe de seguir os passos documentados.
* **Evidência esperada**: Documento do Plano de Resposta a Incidentes atualizado e ata do exercício de simulação realizado.
* **Frequência de revisão**: Anual
* **Fonte**: SP 800-61 Rev. 3 | NIST; Cybersecurity Framework | NIST.

### IR-02: Procedimento de Revogação Imediata de Credenciais Comprometidas
* **ID**: IR-02
* **Controle**: Ação de Emergência para Invalidação de Segredos Expostos
* **Descrição**: Estabelecer o procedimento de emergência de duas fases para revogação de credenciais em caso de suspeita de comprometimento: primeiro a invalidação direta no provedor do SaaS/API de origem, seguida da substituição no cofre do n8n.
* **Risco mitigado**: Uso continuado de tokens roubados a partir do n8n para acesso e exfiltração de dados diretamente nos sistemas corporativos de destino.
* **Prioridade**: P0
* **Tipo**: Corretivo
* **Responsável**: CSIRT / Admin n8n / Donos de Sistemas
* **Como implementar**: Documentar a lista de contatos dos administradores de cada sistema integrado e manter scripts de emergência para revogação rápida de tokens via API do provedor (ex: AWS CLI, GitHub API).
* **Como verificar**: Simular a revogação de uma credencial de testes e confirmar que a chamada correspondente passa a retornar erro de autenticação 401 imediatamente.
* **Evidência esperada**: Playbook de revogação de emergência testado e validado.
* **Frequência de revisão**: Semestral
* **Fonte**: Secrets Management - OWASP Cheat Sheet; Secure Your n8n Instance (VPS US).

### IR-03: Procedimento de Isolação de Rede e Contenção de Worker/Runner
* **ID**: IR-03
* **Controle**: Contenção Imediata de Nós de Infraestrutura Comprometidos
* **Descrição**: Definir os comandos de infraestrutura e automações para isolamento imediato de rede de um pod/contêiner de Worker ou Task Runner afetado por exploração de código, preservando a memória para análise forense.
* **Risco mitigado**: Continuidade da exfiltração de dados, movimentação lateral no cluster e destruição de evidências pelo atacante.
* **Prioridade**: P1
* **Tipo**: Corretivo
* **Responsável**: CSIRT / DevOps / Kubernetes Admin
* **Como implementar**: Criar manifestos de `NetworkPolicy` de emergência (Isolamento Total) e scripts de remoção de nó do cluster de produção mantendo o pod em estado pausado para extração de snapshot de memória.
* **Como verificar**: Executar teste de isolamento em ambiente de homologação verificando a perda imediata de conectividade de rede do pod afetado.
* **Evidência esperada**: Script de isolamento emergencial de pod e playbook de contenção atested.
* **Frequência de revisão**: Semestral
* **Fonte**: Incident Response | CSRC; SP 800-61 Rev. 3 | NIST. [INFERÊNCIA]

---

## 17. AI / LLM / Agent Security (AI)

### AI-01: Aprovação Humana Obrigatória (Human-in-the-Loop) para Ferramentas Críticas
* **ID**: AI-01
* **Controle**: Intervenção Humana Decisória na Execução de Ações por Agentes de IA
* **Descrição**: Exigir a configuração de nós de aprovação humana (*Human-in-the-Loop* / HITL) antes que um agente de IA execute ferramentas de escrita, alteração, exclusão de dados ou envio de comunicações externas (e-mail, alteração em banco SQL, ações de ERP/CRM).
* **Risco mitigado**: Execução não intencional de ações destrutivas ou indesejadas causadas por alucinação do modelo, injeção de prompt ou manipulação de parâmetros por usuários.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Desenvolvedores de AI Agents / SecOps
* **Como implementar**: Inserir o nó de aprovação por formulário/e-mail (HITL) nas ramificações de ferramentas de alta criticidade do agente, interrompendo a execução do fluxo até o clique de confirmação do operador responsável.
* **Como verificar**: Disparar uma requisição para o agente solicitando uma alteração no banco e verificar se o fluxo trava na etapa de aprovação aguardando a interação do operador.
* **Evidência esperada**: Estrutura do workflow com Agente de IA demonstrando o nó HITL e logs de auditoria registrando o e-mail do aprovador humano.
* **Frequência de revisão**: A cada criação de Agente de IA
* **Fonte**: OWASP Top 10 for LLM Applications; Set permissions and roles - n8n Docs.

### AI-02: Validação Determinística de Chamadas de Ferramentas (Tool Call Validation)
* **ID**: AI-02
* **Controle**: Filtragem Rígida dos Parâmetros Gerados por Modelos de IA
* **Descrição**: Interceptador e validar programaticamente todos os argumentos e parâmetros em JSON gerados pelo modelo de linguagem (LLM) antes de repassá-los para os nós de execução das ferramentas (*Tool Nodes*).
* **Risco mitigado**: *Tool Manipulation*, injeção de comandos SQL/Shell via argumentos gerados pelo LLM e alteração de parâmetros de negócios.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Desenvolvedores de AI Agents / Segurança de Aplicações
* **Como implementar**: Criar sub-workflows como ferramentas que recebem a chamada do LLM e executam nós de validação de esquema JSON (*JSON Schema Validator*) e sanitização antes de disparar a ação externa.
* **Como verificar**: Testar a ferramenta enviando um parâmetro contendo caracteres de injeção e verificar a rejeição pelo nó de validação do esquema.
* **Evidência esperada**: Esquema JSON de validação configurado no sub-workflow da ferramenta.
* **Frequência de revisão**: Semestral
* **Fonte**: LLM Prompt Injection Prevention - OWASP Cheat Sheet; OWASP Top 10 for LLM Applications. [INFERÊNCIA]

### AI-03: Delimitação Estrita de Prompts e Defesas contra Prompt Injection Indireto
* **ID**: AI-03
* **Controle**: Isolamento entre Instruções do Sistema e Dados Não Confiáveis
* **Descrição**: Estruturar os prompts do sistema (*System Prompts*) utilizando delimitadores claros e aplicando modelos de filtragem prévia (*Guardrail Models*) para analisar dados recebidos de fontes externas (e-mails, documentos RAG, webhooks) antes de inseri-los no contexto do agente.
* **Risco mitigado**: Injeção de Prompt Direta e Indireta (*Indirect Prompt Injection*), sequestro de raciocínio do agente e vazamento do System Prompt para usuários externos.
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: Engenharia de IA / SecOps
* **Como implementar**: Utilizar sintaxe de delimitação explícita (ex: `<user_input> {{ $json.body }} </user_input>`), instruir o LLM a tratar o conteúdo como texto passivo e utilizar nós de integração com modelos de guarda (Llama Guard / ShieldGemma).
* **Como verificar**: Enviar e-mails e payloads contendo instruções maliciosas de override de contexto e verificar se o agente ignora o comando malicioso mantendo sua instrução original.
* **Evidência esperada**: Ficha técnica do System Prompt auditado e logs de testes de injeção efetuados.
* **Frequência de revisão**: Semestral
* **Fonte**: LLM Prompt Injection Prevention - OWASP Cheat Sheet; OWASP Top 10 for LLM Applications. [INFERÊNCIA]

### AI-04: Auditoria Runtime de Chamadas MCP e Eventos de LLM (`n8n.audit.mcp.tool.called`)
* **ID**: AI-04
* **Controle**: Monitoramento e Registro Detalhado da Operação de Agentes de IA
* **Descrição**: Ativar o streaming de eventos de auditoria específicos para nós de IA e Model Context Protocol (MCP), registrando todas as ferramentas acionadas, modelos chamados, tokens consumidos e erros de geração.
* **Risco mitigado**: Frequente opacidade sobre as ações decididas autonomamente pelos Agentes de IA e ausência de rastreabilidade para investigação de danos causados por LLMs.
* **Prioridade**: P1
* **Tipo**: Detectivo
* **Responsável**: SecOps / AI Engineering
* **Como implementar**: Garantir que o barramento de logs de auditoria do n8n esteja coletando os eventos `n8n.audit.mcp.tool.called`, `n8n.audit.llm.generated` e direcionando-os ao SIEM centralizado.
* **Como verificar**: Consultar no SIEM o histórico do evento `n8n.audit.mcp.tool.called` confirmando o registro dos parâmetros do usuário e do status da execução.
* **Evidência esperada**: Dashboard no SIEM monitorando o uso de ferramentas por Agentes de IA e alertas de erro em ferramentas.
* **Frequência de revisão**: Mensal
* **Fonte**: Stream logs to external systems - n8n Docs; OWASP Top 10 for LLM Applications.

---

## 18. Privacy / LGPD (PRV)

### PRV-01: Mapeamento de Operações de Tratamento e Registro de Atividades (ROPA)
* **ID**: PRV-01
* **Controle**: Mapeamento do Fluxo de Dados Pessoais nos Workflows
* **Descrição**: Manter registro detalhado e atualizado das operações de tratamento de dados pessoais realizadas pelos workflows do n8n, documentando a base legal, a finalidade, a lista de dados coletados e os sistemas de destino.
* **Risco mitigado**: Desconformidade com o Artigo 37 da LGPD (Registro das Operações de Tratamento) e incapacidade de responder a auditorias da Autoridade Nacional de Proteção de Dados (ANPD).
* **Prioridade**: P0
* **Tipo**: Preventivo
* **Responsável**: DPO / Jurídico / Donos de Workflows
* **Como implementar**: Incorporar os campos de adequação à LGPD no inventário corporativo de workflows, mapeando se a automação trata dados pessoais comuns ou sensíveis e identificando a base legal correspondente.
* **Como verificar**: Análise anual do inventário de workflows em relação aos relatórios de impacto à proteção de dados pessoais (RIPD/DPIA).
* **Evidência esperada**: Documento de Registro das Operações de Tratamento de Dados (ROPA) cobrindo as automações do n8n.
* **Frequência de revisão**: Anual
* **Fonte**: Lei Geral de Proteção de Dados Pessoais (LGPD) — Art. 37; Privacy Policy (n8n).

### PRV-02: Mecanismo de Atendimento aos Direitos dos Titulares (Exclusão/Exportação)
* **ID**: PRV-02
* **Controle**: Atendimento a Solicitações de Exclusão, Acesso e Portabilidade
* **Descrição**: Estabelecer workflows administrativos padronizados para localização, exportação e exclusão de dados pessoais de um titular em todos os bancos de dados, CRMs e plataformas conectadas via n8n.
* **Risco mitigado**: Descumprimento do Artigo 18 da LGPD (Direitos do Titular) e imposição de multas administrativas pela ANPD.
* **Prioridade**: P0
* **Tipo**: Corretivo
* **Responsável**: DPO / Equipe de Operações de Privacidade
* **Como implementar**: Criar um workflow mestre de privacidade no n8n que recebe o identificador do titular (ex: CPF/e-mail) e dispara chamadas de expurgamento/exportação em todos os sistemas integrados.
* **Como verificar**: Simular uma solicitação de exclusão de titular em ambiente de homologação e verificar que os dados do titular são totalmente removidos dos sistemas de destino.
* **Evidência esperada**: Relatório de execução do workflow de atendimento ao titular comprovando o expurgamento.
* **Frequência de revisão**: Semestral
* **Fonte**: Lei Geral de Proteção de Dados Pessoais (LGPD) — Art. 18; Privacy Policy (n8n). [INFERÊNCIA]

### PRV-03: Cláusulas Contratuais de Transferência Internacional e Governança de Subprocessadores
* **ID**: PRV-03
* **Controle**: Gestão do Fluxo Transfronteiriço de Dados Pessoais
* **Descrição**: Mapear e formalizar os instrumentos jurídicos adequados (Cláusulas-Padrão Contratuais / SCCs) e verificações de segurança para qualquer workflow do n8n que envie dados pessoais para servidores ou APIs hospedadas fora do território brasileiro (ex: LLMs nos EUA).
* **Risco mitigado**: Transferência internacional de dados desalinhada do Artigo 33 da LGPD e uso de subprocessadores sem garantias de privacidade equivalentes.
* **Prioridade**: P1
* **Tipo**: Preventivo
* **Responsável**: DPO / Jurídico
* **Como implementar**: Auditar os destinos de saída dos nós de integração (HTTP Request, OpenAI, Anthropic, Google) e exigir que os fornecedores internacionais assinem os Termos de Processamento de Dados (DPA) com Cláusulas-Padrão.
* **Como verificar**: Análise contratual dos DPAs firmados com os fornecedores de APIs externas e SaaS utilizados nos workflows do n8n.
* **Evidência esperada**: DPAs e Cláusulas-Padrão Contratuais assinadas com os provedores internacionais de SaaS/LLM.
* **Frequência de revisão**: Anual
* **Fonte**: Lei Geral de Proteção de Dados Pessoais (LGPD) — Art. 33; n8n Customer Acceptable Use Policy.

---

## Lacunas de Segurança e Tópicos para Investigação Futura

Apesar do grau de maturidade técnica das diretrizes estabelecidas neste baseline, a análise do ecossistema do n8n self-hosted em produção revelou as seguintes lacunas técnicas e operacionais que demandam investigação contínua:

1. **Manifestos de Referência Hardened para Kubernetes Helm Charts**:
   * *Descrição da Lacuna*: A documentação oficial do n8n e as fontes públicas focam fortemente na implantação via Docker Compose. Falta um projeto oficial de referência de *Helm Chart* pré-configurado contendo todos os manifestos de `PodSecurityStandards` (Restricted), `NetworkPolicies` por camada e regras de `AppArmor` nativas para Kubernetes. [NÃO CONFIRMADO]
2. **Framework de Testes e Sandbox In-Memory para Validar Task Runners**:
   * *Descrição da Lacuna*: Ausência de um utilitário oficial de testes de estresse e penetração específico para validar a eficácia da sandbox do *Task Runner* em ambiente de homologação antes de promover uma nova versão para produção. [NÃO CONFIRMADO]
3. **Mecanismos Determinísticos de Guardrails no Canvas para AI Agents**:
   * *Descrição da Lacuna*: Embora o OWASP Top 10 para LLMs forneça a teoria de mitigação contra Prompt Injection, o n8n ainda não possui um nó nativo dedicado a atuar como um *Guardrail Firewall* determinístico (sanitização automática de entrada e saída de LLMs sem depender de um segundo modelo de linguagem). [NÃO CONFIRMADO]
4. **Governança Automatizada de Rotação de Credenciais OAuth2 Refresh Tokens**:
   * *Descrição da Lacuna*: A rotação de chaves mestras e DEKs está documentada no n8n, contudo a expiração e renovação automática de *Refresh Tokens OAuth2* de longa duração mantidos no cofre em caso de revogação em massa no provedor de origem carece de automação via CLI. [NÃO CONFIRMADO]
5. **Procedimento de Perícia Forense e Extração de Memória em Task Runners**:
   * *Descrição da Lacuna*: Falta um guia operacional detalhado para extração e preservação de evidências de memória (*Memory Dump*) de contêineres *distroless* em execução sem interromper a coleta de logs do orquestrador em caso de incidente de RCE ativado. [NÃO CONFIRMADO]

---
