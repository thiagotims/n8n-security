# N8N SELF-HOSTED PRODUCTION SECURITY: CHECKLIST OPERACIONAL DE AUDITORIA

Este checklist foi derivado do **N8N Self-Hosted Production Security Baseline** e foi estruturado para execução prática por administradores Linux, engenheiros DevOps e equipes de SecOps.

Para cada item de controle, o checklist fornece a pergunta de auditoria, o procedimento de verificação técnica, os comandos suportados pelas fontes oficiais, o resultado esperado, a evidência a ser coletada, o campo de validação (OK/NOK), o risco associado, a ação corretiva e a prioridade de implementação.


---

### Item Checklist: GOV-01 — Conformidade com Termos de Uso (AUP/EULA) e Política de Uso Aceitável de Automação

* **ID**: GOV-01
* **Pergunta**: Os workflows ativos em produção estão em conformidade formal com a Política de Uso Aceitável de Automação e os termos contratuais (AUP/EULA) do n8n?
* **Como verificar**: Auditoria semestral dos tipos de fluxos implantados em produção em relação à lista de casos de uso permitidos na política.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Política corporativa assinada e alinhada com o AUP/EULA do n8n.
* **Evidência**: Documento de política assinado, registros de aceite dos usuários e relatórios de auditoria de conformidade.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Sanções legais, rescisão unilateral de licença pelo fornecedor, violações regulatórias e responsabilidade civil/penal por automações abusivas.
* **Ação corretiva**: Criar a *Política Corporativa de Automação de Processos*, incorporando as proibições do n8n AUP. Exigir que todo desenvolvedor ou editor de fluxos assine o termo antes de receber acesso de edição no n8n.
* **Prioridade**: P0

### Item Checklist: GOV-02 — Matriz de Classificação de Risco de Workflows e Mapeamento de Impacto

* **ID**: GOV-02
* **Pergunta**: Existe um inventário atualizado com a classificação de risco de todos os workflows corporativos ativos no n8n?
* **Como verificar**: Checagem do inventário de workflows contra a lista de fluxos ativos obtida via API do n8n (`GET /api/v1/workflows`).
* **Comando/procedimento, quando suportado pelas fontes**: curl -s -H "X-N8N-API-KEY: $N8N_API_KEY" http://localhost:5678/api/v1/workflows | jq '.data[] | {id, name, active}'
* **Resultado esperado**: Lista de workflows ativos retornada em JSON sem exceções não mapeadas.
* **Evidência**: Planilha/Database de inventário de workflows atualizado e vinculado aos donos de negócio.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Falta de visibilidade sobre automações de alto risco, ausência de controles proporcionais e incapacidade de priorizar incidentes.
* **Ação corretiva**: Criar um inventário centralizado contendo: ID do workflow, nome, proprietário, nível de criticidade, sistemas de destino e se manipula PII ou executa ações irreversíveis.
* **Prioridade**: P0

### Item Checklist: GOV-03 — Esteira de Aprovação e Gestão de Mudanças em Automações de Produção

* **ID**: GOV-03
* **Pergunta**: Toda alteração em workflows de produção passa obrigatoriamente por esteira de aprovação e revisão de código antes da implantação?
* **Como verificar**: Comparar a data e os hashes dos fluxos em produção contra os commits aprovados no branch `main` do Git.
* **Comando/procedimento, quando suportado pelas fontes**: git log -n 5 --oneline (no repositório de workflows/IaC)
* **Resultado esperado**: Commits de alteração de workflow vinculados a Pull Requests aprovados no Git.
* **Evidência**: Histórico de Pull Requests no Git com aprovações registradas e logs do pipeline de deploy.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Alterações não autorizadas, erros de lógica produtivos, injeção de scripts maliciosos e indisponibilidade de processos de negócio.
* **Ação corretiva**: Implementar fluxo de promoção via repositório Git corporativo utilizando a integração nativa de Git (Enterprise) ou exportação de pacotes `.n8np` via CI/CD, exigindo Pull Request com pelo menos um aprovador.
* **Prioridade**: P1

### Item Checklist: ASM-01 — Mapeamento de Ativos e Componentes da Topologia n8n

* **ID**: ASM-01
* **Pergunta**: Todas as instâncias, Workers e Task Runners ativos do n8n estão devidamente registrados e mapeados no inventário de ativos/CMDB?
* **Como verificar**: Executar rotinas de varredura no orquestrador (Docker/Kubernetes) comparando os contêineres ativos com a lista do CMDB.
* **Comando/procedimento, quando suportado pelas fontes**: docker ps --filter "label=app.kubernetes.io/name=n8n"  # em Docker
kubectl get pods -n n8n-prod -l app.kubernetes.io/name=n8n  # em Kubernetes
* **Resultado esperado**: Todos os contêineres e pods ativos constam no inventário/CMDB oficial.
* **Evidência**: Dashboard de infraestrutura/CMDB listando todas as instâncias n8n e seus respectivos hashes de imagem.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Instâncias "sombra" (*Shadow IT*), Workers desatualizados executando versões vulneráveis e falta de contenção em caso de incidentes.
* **Ação corretiva**: Utilizar tags/labels padronizadas nos contêineres e pods (`app.kubernetes.io/name=n8n`, `role=main`, `role=worker`, `role=runner`), e integrar o discovery de ativos ao CMDB/Prometheus.
* **Prioridade**: P0

### Item Checklist: ASM-02 — Inventário de Módulos, Nós e APIs Conectadas

* **ID**: ASM-02
* **Pergunta**: Todas as conexões, webhooks e APIs de terceiros utilizadas pelos workflows do n8n estão catalogadas?
* **Como verificar**: Análise do relatório JSON gerado pelo comando `n8n audit`.
* **Comando/procedimento, quando suportado pelas fontes**: n8n audit  # via CLI no contêiner n8n
curl -X POST -H "X-N8N-API-KEY: $N8N_API_KEY" http://localhost:5678/rest/audit  # via API
* **Resultado esperado**: Relatório JSON do n8n audit gerado com lista de nós e integrações.
* **Evidência**: Relatório oficial do `n8n audit` arquivado no repositório de segurança.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Conexões órfãs mantendo acesso a sistemas críticos e vazamento de dados por integrações esquecidas.
* **Ação corretiva**: Utilizar o comando `n8n audit` via CLI/API para extrair a lista completa de nós em uso e mapear as credenciais associadas.
* **Prioridade**: P1

### Item Checklist: IAM-01 — Habilitação do Motor de Gestão de Usuários

* **ID**: IAM-01
* **Pergunta**: O motor de gestão de usuários está ativado e a autenticação individual é exigida para acessar o n8n?
* **Como verificar**: Tentar acessar a URL base do n8n em uma janela anônima e verificar o redirecionamento obrigatório para a tela de login.
* **Comando/procedimento, quando suportado pelas fontes**: docker exec n8n-main env | grep N8N_USER_MANAGEMENT_DISABLED
kubectl exec -n n8n-prod deploy/n8n-main -- env | grep N8N_USER_MANAGEMENT_DISABLED
* **Resultado esperado**: N8N_USER_MANAGEMENT_DISABLED=false
* **Evidência**: Configuração de variáveis de ambiente do contêiner e teste de acesso bloqueado sem autenticação.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Acesso não autenticado ao canvas do editor, capacidade de visualização e alteração de fluxos por atacantes anônimos.
* **Ação corretiva**: Configurar a variável de ambiente `N8N_USER_MANAGEMENT_DISABLED=false` e garantir que a conta de proprietário (*Owner*) inicial seja provisionada com senha forte.
* **Prioridade**: P0

### Item Checklist: IAM-02 — Autenticação Centralizada com Single Sign-On e MFA no IdP

* **ID**: IAM-02
* **Pergunta**: O Single Sign-On (SSO) via SAML/OIDC/LDAP com MFA está ativado para autenticação de todos os usuários no n8n?
* **Como verificar**: Testar o fluxo de login confirmando o redirecionamento para o IdP corporativo e a exigência do segundo fator de autenticação.
* **Comando/procedimento, quando suportado pelas fontes**: docker exec n8n-main env | grep -E "N8N_SSO_|N8N_SAML_|N8N_OIDC_"
* **Resultado esperado**: Variáveis de SSO configuradas e login redirecionado ao IdP.
* **Evidência**: Logs de autenticação do IdP registrando logins bem-sucedidos com MFA e tela de configurações SSO ativa no n8n.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Ataques de força bruta, roubo de credenciais locais, uso de senhas fracas e persistência de acessos pós-desligamento de funcionários (*offboarding* tardio).
* **Ação corretiva**: Configurar SSO nas opções da instância (Enterprise), preenchendo as variáveis `N8N_SSO_SAML_*` ou `N8N_SSO_OIDC_*`, desativando o login local por e-mail/senha caso suportado.
* **Prioridade**: P0

### Item Checklist: IAM-03 — Aplicação do Princípio do Menor Privilégio via RBAC

* **ID**: IAM-03
* **Pergunta**: As permissões de acesso dos usuários estão restritas conforme o modelo RBAC em nível de projeto e instância?
* **Como verificar**: Consultar a lista de usuários e suas permissões na interface administrativa (*Settings > Users*) ou via API REST.
* **Comando/procedimento, quando suportado pelas fontes**: curl -s -H "X-N8N-API-KEY: $N8N_API_KEY" http://localhost:5678/api/v1/users
* **Resultado esperado**: Lista de usuários retornada vinculada a papéis RBAC específicos por projeto.
* **Evidência**: Relatório de exportação de usuários e papéis atribuídos no n8n.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Escalação de privilégios interna, alteração acidental de fluxos críticos por usuários não qualificados e leitura não autorizada de execuções.
* **Ação corretiva**: Mapear os grupos do IdP corporativo para os papéis do RBAC do n8n via provisionamento automático de papéis (`N8N_SSO_USER_ROLE_PROVISIONING`). Atribuir o papel 'Viewer' por padrão a novos usuários.
* **Prioridade**: P0

### Item Checklist: IAM-04 — Restrição de Criação e Execução em Espaços Pessoais

* **ID**: IAM-04
* **Pergunta**: As políticas de espaço pessoal estão ativas impedindo o compartilhamento descontrolado de automações?
* **Como verificar**: Tentar ativar um workflow de teste em um espaço pessoal sem permissão administrativa e verificar o bloqueio.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Políticas de espaço pessoal ativas proibindo publicação não gerenciada.
* **Evidência**: Captura de tela das configurações de *Personal Space Policy* ativas na instância.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: *Shadow Automation*, desvio de governança, exfiltração de dados para contas pessoais e criação de fluxos não auditados.
* **Ação corretiva**: Ativar as políticas de gerenciamento de espaço pessoal nas configurações globais de governança da instância (Enterprise), proibindo o compartilhamento desregrado e a ativação de fluxos pessoais de alta criticidade.
* **Prioridade**: P1

### Item Checklist: IAM-05 — Proteção de Tokens de Sessão HTTP e Chaves API

* **ID**: IAM-05
* **Pergunta**: Os cookies de sessão estão configurados com as flags de segurança N8N_SECURE_COOKIE e N8N_SAMESITE_COOKIE?
* **Como verificar**: Inspecionar os cabeçalhos HTTP da resposta do login (`Set-Cookie`) no navegador e confirmar a presença das flags `Secure`, `HttpOnly` e `SameSite=Lax`.
* **Comando/procedimento, quando suportado pelas fontes**: docker exec n8n-main env | grep -E "N8N_SECURE_COOKIE|N8N_SAMESITE_COOKIE"
* **Resultado esperado**: N8N_SECURE_COOKIE=true e N8N_SAMESITE_COOKIE=lax (ou strict).
* **Evidência**: Análise de cabeçalhos HTTP capturados via cURL ou ferramentas de inspeção Web.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Sequestro de sessão (*Session Hijacking*), ataques de Man-in-the-Middle (MitM) e falsificação de requisições (*CSRF*).
* **Ação corretiva**: Definir as variáveis de ambiente `N8N_SECURE_COOKIE=true` (exige HTTPS) e `N8N_SAMESITE_COOKIE=lax` (ou `strict`).
* **Prioridade**: P0

### Item Checklist: SEC-01 — Definição Manual da Chave Mestra de Criptografia do Cofre

* **ID**: SEC-01
* **Pergunta**: A chave mestra N8N_ENCRYPTION_KEY possui alta entropia (32 bytes) e está protegida com permissão 600 no arquivo de configuração?
* **Como verificar**: Verificar se a variável está definida no processo e se os logs de inicialização não apresentam a mensagem "Mismatching encryption keys".
* **Comando/procedimento, quando suportado pelas fontes**: docker exec n8n-main env | grep N8N_ENCRYPTION_KEY
stat -c "%a %n" ~/.n8n/config
* **Resultado esperado**: N8N_ENCRYPTION_KEY definida com 32 bytes e permissão 600 no arquivo config.
* **Evidência**: Arquivo `.env` ou manifesto Kubernetes Secret auditado (com o valor oculto) e logs de inicialização sem erros.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Perda permanente do cofre de credenciais ao recriar o contêiner, uso de chaves fracas e exposição de credenciais em backups do arquivo `config`.
* **Ação corretiva**: Gerar a chave via `openssl rand -base64 32` e injetá-la como variável de ambiente `N8N_ENCRYPTION_KEY` ou arquivo montado `N8N_ENCRYPTION_KEY_FILE`.
* **Prioridade**: P0

### Item Checklist: SEC-02 — Rotação Ativa de Chaves de Criptografia de Dados

* **ID**: SEC-02
* **Pergunta**: A rotação periódica de Data Encryption Keys (DEKs) está ativada na instância do n8n?
* **Como verificar**: Consultar o status das chaves na interface de gestão de chaves do n8n e verificar se a nova DEK consta como ativa.
* **Comando/procedimento, quando suportado pelas fontes**: curl -s -H "X-N8N-API-KEY: $N8N_API_KEY" http://localhost:5678/encryption/keys
* **Resultado esperado**: Retorno de status indicando DEKs ativas e rotacionadas.
* **Evidência**: Captura de tela da UI de gestão de chaves ou resposta JSON do endpoint `/encryption/keys`.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Comprometimento de longo prazo por uso continuado da mesma chave criptográfica e não conformidade com padrões de gestão de segredos.
* **Ação corretiva**: Configurar a variável `N8N_ENV_FEAT_ENCRYPTION_KEY_ROTATION=true` em todas as instâncias (Main e Workers) e acionar a rotação pela UI em *Settings > Data Encryption Keys* ou via API `POST /encryption/keys`.
* **Prioridade**: P1

### Item Checklist: SEC-03 — Desativação do Acesso ao `process.env` por Scripts de Usuários

* **ID**: SEC-03
* **Pergunta**: A variável N8N_BLOCK_ENV_ACCESS_IN_NODE está configurada como true para impedir que Code Nodes leiam os segredos da aplicação?
* **Como verificar**: Criar um workflow de teste com um *Code Node* contendo `return process.env;` e confirmar que a execução retorna um objeto vazio ou erro de acesso.
* **Comando/procedimento, quando suportado pelas fontes**: docker exec n8n-main env | grep N8N_BLOCK_ENV_ACCESS_IN_NODE
* **Resultado esperado**: N8N_BLOCK_ENV_ACCESS_IN_NODE=true
* **Evidência**: Log de execução do workflow de teste comprovando a negação de acesso ao `process.env`.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Exfiltração da chave mestra `N8N_ENCRYPTION_KEY`, senhas do banco de dados e tokens de sistema por código malicioso ou injeção de scripts em workflows.
* **Ação corretiva**: Definir a variável de ambiente `N8N_BLOCK_ENV_ACCESS_IN_NODE=true` no orquestrador do n8n (padrão a partir da v2.0).
* **Prioridade**: P0

### Item Checklist: SEC-04 — Armazenamento e Resolução Dinâmica de Segredos em Cofres Corporativos

* **ID**: SEC-04
* **Pergunta**: O n8n está integrado a um cofre de segredos externo (External Secrets) para resolução de credenciais em tempo de execução?
* **Como verificar**: Verificar se as credenciais cadastradas no n8n utilizam referências externas em vez de valores estáticos em texto claro.
* **Comando/procedimento, quando suportado pelas fontes**: docker exec n8n-main env | grep -i EXTERNAL_SECRETS
* **Resultado esperado**: Variáveis ou configurações de External Secrets ativas apontando para o cofre.
* **Evidência**: Configuração ativa visível em *Settings > External Secrets* e logs de auditoria do Vault/AWS registrando acessos pelo n8n.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Vazamento do cofre de credenciais em caso de comprometimento do banco de dados relacional e falta de rotação centralizada de segredos.
* **Ação corretiva**: Configurar a conexão com o provedor de segredos externo nas configurações de *External Secrets* (Enterprise) e referenciar as chaves nos workflows no formato `{{ $secrets.vault.MY_SECRET }}`.
* **Prioridade**: P1

### Item Checklist: SEC-05 — Restrição de Escopos de Credenciais e Padronização de Nomes

* **ID**: SEC-05
* **Pergunta**: As credenciais cadastradas utilizam Contas de Serviço dedicadas sob padrão de nomenclatura estruturado?
* **Como verificar**: Auditoria visual do cadastro de credenciais no n8n e verificação dos escopos configurados no provedor do SaaS.
* **Comando/procedimento, quando suportado pelas fontes**: n8n audit
* **Resultado esperado**: Credenciais nomeadas no padrão [SISTEMA]-[PERMISSÃO]-[EQUIPE]-[PROPÓSITO].
* **Evidência**: Inventário de credenciais cadastradas com papéis e escopos mapeados.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Ampliação do raio de impacto (*Blast Radius*) em caso de vazamento de credencial, alteração indevida de recursos em sistemas de destino e perda de rastreabilidade.
* **Ação corretiva**: Exigir que cada credencial siga a convenção de nome `[SISTEMA]-[PERMISSÃO]-[EQUIPE]-[PROPÓSITO]` (ex: `stripe-write-ops-invoices`) e limitar os escopos OAuth2 estritamente às APIs necessárias.
* **Prioridade**: P0

### Item Checklist: NET-01 — Criptografia de Dados em Trânsito Perimetral

* **ID**: NET-01
* **Pergunta**: O acesso à interface do n8n e webhooks exige o protocolo TLS 1.3 no Proxy Reverso / Ingress Controller?
* **Como verificar**: Executar varredura cURL ou SSL Labs na URL do n8n confirmando a rejeição de conexões HTTP e suporte exclusivo a ciphers fortes TLS 1.2/1.3.
* **Comando/procedimento, quando suportado pelas fontes**: curl -vI https://n8n.empresa.com 2>&1 | grep "SSL connection"
openssl s_client -connect n8n.empresa.com:443 -tls1_3
* **Resultado esperado**: Conexão negociada sob TLS 1.3 com certificado válido.
* **Evidência**: Configuração do Proxy/Ingress auditada e relatório de teste de TLS com nota A+.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Interceptação de tráfego (*Eavesdropping*), roubo de tokens de sessão em redes abertas e ataques de Man-in-the-Middle (MitM).
* **Ação corretiva**: Configurar o Proxy Reverso para realizar o encerramento TLS, redirecionar todo o tráfego HTTP (porta 80) para HTTPS (porta 443) e aplicar o cabeçalho HSTS (`Strict-Transport-Security`).
* **Prioridade**: P0

### Item Checklist: NET-02 — Isolamento de Arquitetura de Rede em Três Camadas

* **ID**: NET-02
* **Pergunta**: A infraestrutura do n8n está segmentada em zonas de rede distintas (DMZ, Aplicação e Dados)?
* **Como verificar**: Tentar conectar diretamente aos endereços IP do PostgreSQL e Redis a partir de uma origem externa à rede privada.
* **Comando/procedimento, quando suportado pelas fontes**: kubectl get networkpolicy -n n8n-prod
docker network inspect n8n-bridge
* **Resultado esperado**: NetworkPolicies do Kubernetes ou Security Groups restringindo portas.
* **Evidência**: Diagrama de topologia de rede e regras de Security Group / Firewall exportadas da nuvem.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Acesso direto da internet ao banco de dados ou ao Redis, movimentação lateral de atacantes e comprometimento da infraestrutura.
* **Ação corretiva**: Criar Security Groups / tabelas de roteamento na nuvem permitindo que apenas a DMZ acesse a porta do n8n, e que o n8n acesse apenas as portas 5432 (PostgreSQL) e 6379 (Redis) na Zona de Dados.
* **Prioridade**: P0

### Item Checklist: NET-03 — Habilitação do Filtro de Requisições Server-Side Request Forgery

* **ID**: NET-03
* **Pergunta**: A proteção nativa contra SSRF (N8N_ENABLE_SSRF_PROTECTION) está ativada no n8n?
* **Como verificar**: Criar um workflow de teste com o nó *HTTP Request* fazendo requisição para `http://169.254.169.254` ou `http://127.0.0.1:5432` e confirmar a rejeição do disparo pelo n8n.
* **Comando/procedimento, quando suportado pelas fontes**: docker exec n8n-main env | grep N8N_ENABLE_SSRF_PROTECTION
* **Resultado esperado**: N8N_ENABLE_SSRF_PROTECTION=true
* **Evidência**: Log de execução do workflow de teste exibindo o bloqueio por política de SSRF.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Leitura não autorizada da interface de metadados da nuvem (`169.254.169.254`), exfiltração de chaves IAM do nó e acesso a portas internas da infraestrutura corporativa.
* **Ação corretiva**: Definir a variável de ambiente `N8N_ENABLE_SSRF_PROTECTION=true` no orquestrador do n8n.
* **Prioridade**: P0

### Item Checklist: NET-04 — Filtragem de Conexões de Saída da Instância do n8n

* **ID**: NET-04
* **Pergunta**: O firewall de saída (Egress Firewall) bloqueia acessos do n8n para a rede interna e endpoints de metadados da nuvem?
* **Como verificar**: Executar comando de teste a partir do contêiner tentando conectar a um IP externo arbitrário na porta 80/443 não autorizada.
* **Comando/procedimento, quando suportado pelas fontes**: kubectl exec -n n8n-prod deploy/n8n-main -- curl -m 5 -s https://169.254.169.254
* **Resultado esperado**: Requisição para 169.254.169.254 ou IPs privados locais rejeitada / timeout.
* **Evidência**: Regras do Firewall de Egress/Proxy exportadas e ativas.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Comunicação do contêiner com servidores de Comando e Controle (C2) de atacantes, exfiltração não autorizada de dados e ataques de scanning de rede a partir do n8n.
* **Ação corretiva**: Configurar regras de iptables, AWS Security Groups de saída ou Egress Proxy (Squid/Istio) permitindo tráfego de saída apenas para a lista de domínios das APIs corporativas e SaaS utilizados.
* **Prioridade**: P1

### Item Checklist: NET-05 — Aplicação de Políticas de Microsegmentação no Namespace do K8s

* **ID**: NET-05
* **Pergunta**: A política NetworkPolicy default-deny-all está aplicada no namespace do Kubernetes isolando o n8n?
* **Como verificar**: Executar `kubectl exec` no pod do n8n e tentar dar `ping` ou `curl` no IP de um pod em outro namespace.
* **Comando/procedimento, quando suportado pelas fontes**: kubectl get networkpolicy -n n8n-prod default-deny-all
* **Resultado esperado**: NetworkPolicy default-deny-all aplicada no namespace.
* **Evidência**: Manifesto `NetworkPolicy` aplicado e confirmado via `kubectl get netpol -n n8n`.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Movimentação lateral de um pod do n8n comprometido para outros pods e serviços no cluster Kubernetes.
* **Ação corretiva**: Aplicar manifestos de `NetworkPolicy` liberando Ingress apenas do Ingress Controller (porta 5678) e comunicação interna n8n-Runner (porta 5679), e Egress apenas para o CoreDNS (porta 53), PostgreSQL (5432) e Redis (6379).
* **Prioridade**: P0

### Item Checklist: APP-01 — Execução Segura do Motor de Expressões Javascript

* **ID**: APP-01
* **Pergunta**: Os nós de código (Code Nodes) executam desacoplados via Task Runners externos (N8N_RUNNERS_MODE=external)?
* **Como verificar**: Verificar o valor da variável de ambiente no contêiner do n8n orquestrador.
* **Comando/procedimento, quando suportado pelas fontes**: docker exec n8n-main env | grep N8N_RUNNERS_MODE
* **Resultado esperado**: N8N_RUNNERS_MODE=external
* **Evidência**: Tabela de variáveis de ambiente do processo confirmando `N8N_RUNNERS_INSECURE_MODE=false`.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Injeção de código e execução remota de comandos via expressões maliciosas (*CVE-2025-68613*).
* **Ação corretiva**: Garantir que a variável `N8N_RUNNERS_INSECURE_MODE=false` esteja configurada em todas as instâncias e Workers.
* **Prioridade**: P0

### Item Checklist: APP-02 — Restrição de Leitura de Arquivos de Configuração pelo Engine

* **ID**: APP-02
* **Pergunta**: A API pública do n8n está desativada caso não seja utilizada para automação de gerenciamento?
* **Como verificar**: Criar um workflow de teste com o nó *Read/Write Files from Disk* tentando ler o arquivo `/home/node/.n8n/config` e verificar a mensagem de acesso negado.
* **Comando/procedimento, quando suportado pelas fontes**: docker exec n8n-main env | grep N8N_PUBLIC_API_DISABLED
* **Resultado esperado**: N8N_PUBLIC_API_DISABLED=true (se a API pública não for necessária).
* **Evidência**: Log de execução do workflow comprovando a rejeição do acesso ao arquivo interno.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Leitura não autorizada do arquivo de configuração `config`, extração da chave mestra do disco local e acesso ao banco de dados SQLite residual.
* **Ação corretiva**: Definir a variável de ambiente `N8N_BLOCK_FILE_ACCESS_TO_N8N_FILES=true`.
* **Prioridade**: P0

### Item Checklist: APP-03 — Limitando o Acesso de Escrita e Leitura de Arquivos Locais (Chroot Lógico)

* **ID**: APP-03
* **Pergunta**: Os nós de alto risco executeCommand e ssh estão incluídos na lista de bloqueio N8N_NODES_DENYLIST?
* **Como verificar**: Tentar ler o arquivo `/etc/passwd` via nó de arquivo no n8n e confirmar que a operação é bloqueada pelo filtro de diretório.
* **Comando/procedimento, quando suportado pelas fontes**: docker exec n8n-main env | grep N8N_NODES_DENYLIST
* **Resultado esperado**: N8N_NODES_DENYLIST contendo executeCommand e ssh.
* **Evidência**: Log de teste comprovando o bloqueio de acesso fora da pasta permitida.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Vulnerabilidades de *Path Traversal* (como a CVE-2026-21877 no nó Git), leitura arbitrária do sistema de arquivos e sobrescrita de binários do sistema operacional.
* **Ação corretiva**: Configurar a variável `N8N_RESTRICT_FILE_ACCESS_TO=/home/node/.n8n-files` e garantir que o diretório possua as permissões de pasta adequadas.
* **Prioridade**: P0

### Item Checklist: APP-04 — Remoção de Nós de Execução de Comandos do Sistema e Acesso Shell

* **ID**: APP-04
* **Pergunta**: O contêiner do Task Runner executa sob perfil restritivo do AppArmor e Seccomp?
* **Como verificar**: Abrir o editor do n8n e buscar pelos nós "Execute Command" e "SSH", confirmando que não aparecem no menu de adição de nós.
* **Comando/procedimento, quando suportado pelas fontes**: docker inspect n8n-runner | grep -i AppArmor
kubectl get pod <runner-pod> -o jsonpath='{.spec.securityContext.seccompProfile}'
* **Resultado esperado**: Perfil AppArmor ativo impedindo acesso a /proc e Seccomp RuntimeDefault.
* **Evidência**: Relatório do `n8n audit` confirmando a ausência dos nós da denylist na instância.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Execução remota de comandos (RCE) direta por usuários com permissão de edição de workflows sem necessidade de explorar qualquer falha de software.
* **Ação corretiva**: Configurar a variável `N8N_NODES_DENYLIST=["n8n-nodes-base.executeCommand", "n8n-nodes-base.ssh"]` ou utilizar a variável legada `NODES_EXCLUDE`.
* **Prioridade**: P0

### Item Checklist: APP-05 — Desativação do Endpoint da API REST Externa

* **ID**: APP-05
* **Pergunta**: O controle Desativação do Endpoint da API REST Externa está implementado e operante na instância do n8n?
* **Como verificar**: Realizar requisição HTTP GET para a URL `https://n8n.empresa.com/api/v1/workflows` e confirmar o retorno de erro 404/403.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Resposta cURL comprovando o bloqueio do endpoint da API pública.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Redução da superfície de ataque, mitigação de tentativas de força bruta em chaves API e prevenção de enumeração de dados da instância.
* **Ação corretiva**: Configurar a variável de ambiente `N8N_PUBLIC_API_DISABLED=true`.
* **Prioridade**: P1

### Item Checklist: WKF-01 — Segregação de Funções no Ciclo de Vida do Workflow

* **ID**: WKF-01
* **Pergunta**: O controle Segregação de Funções no Ciclo de Vida do Workflow está implementado e operante na instância do n8n?
* **Como verificar**: Tentar ativar um workflow com um usuário com papel de 'Editor' básico e verificar se o botão de ativação permanece desabilitado.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Matriz de papéis e permissões do n8n validada.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Implantação não autorizada de automações maliciosas ou não testadas, desvio de governança e alterações em produção por desenvolvedores júniores.
* **Ação corretiva**: Configurar papéis no RBAC do n8n onde desenvolvedores possuem permissão de edição em projetos de Staging, mas apenas administradores/líderes técnicos possuem permissão de publicação em produção.
* **Prioridade**: P1

### Item Checklist: WKF-02 — Rastreabilidade e Auditoria Visual de Modificações no Canvas

* **ID**: WKF-02
* **Pergunta**: O controle Rastreabilidade e Auditoria Visual de Modificações no Canvas está implementado e operante na instância do n8n?
* **Como verificar**: Inspecionar o histórico de versões do workflow na interface confirmando os registros de comparação e o nome do autor da alteração.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Captura de tela do histórico de versões com as alterações destacadas e aprovadas.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Inserção silenciosa de nós maliciosos de exfiltração de dados, alteração oculta de parâmetros de destino e erros de lógica imperceptíveis.
* **Ação corretiva**: Exigir o uso da ferramenta de *Visual Diff* (Enterprise) durante o processo de revisão de alterações entre a versão salva e a versão em publicação.
* **Prioridade**: P1

### Item Checklist: WKF-03 — Prevenção de Injeção de SQL em Nós de Banco de Dados

* **ID**: WKF-03
* **Pergunta**: O controle Prevenção de Injeção de SQL em Nós de Banco de Dados está implementado e operante na instância do n8n?
* **Como verificar**: Executar o comando `n8n audit` que varre automaticamente a instância buscando consultas SQL não parametrizadas.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Relatório do `n8n audit` confirmando zero alertas de SQLi não parametrizado.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Injeção de SQL (SQLi), destruição de tabelas, exfiltração de dados e alteração não autorizada de registros nos bancos de dados corporativos.
* **Ação corretiva**: Utilizar os campos de parâmetros nativos dos nós SQL do n8n (sintaxe `$1`, `$2` ou objetos de parâmetros do nó) em vez de concatenar expressões `{{ $json.user_input }}` diretamente na query SQL.
* **Prioridade**: P0

### Item Checklist: WKF-04 — Filtragem e Validação de Carga Útil na Entrada do Workflow

* **ID**: WKF-04
* **Pergunta**: O controle Filtragem e Validação de Carga Útil na Entrada do Workflow está implementado e operante na instância do n8n?
* **Como verificar**: Disparar o workflow com cargas inválidas ou incompletas e verificar no histórico que a execução é encerrada na primeira etapa sem acionar nós subsequentes.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Estrutura do workflow no editor comprovando o nó de validação inicial.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Processamento de cargas maliciosas, envenenamento de dados, estouro de cotas de APIs e ataques de Negação de Serviço Lógica.
* **Ação corretiva**: Configurar a opção `Only run if` nas configurações do nó de gatilho ou inserir um nó `If` no início do fluxo validando tipos de dados, tamanhos e presença de campos obrigatórios.
* **Prioridade**: P1

### Item Checklist: WHK-01 — Proteção de Acesso aos Endpoints de Gatilho HTTP

* **ID**: WHK-01
* **Pergunta**: O controle Proteção de Acesso aos Endpoints de Gatilho HTTP está implementado e operante na instância do n8n?
* **Como verificar**: Executar `n8n audit` para listar todos os webhooks desprotegidos ativas na instância.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Relatório do `n8n audit` sem registros de webhooks não autenticadas em produção.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Disparo não autorizado de automações, execução indevida de processos de negócio, exfiltração de dados por chamadas não autenticadas e flooding.
* **Ação corretiva**: Alterar a propriedade "Authentication" do nó de Webhook para "Header Auth" ou "Basic Auth", vinculando-o a uma credencial com chave/token de alta entropia.
* **Prioridade**: P0

### Item Checklist: WHK-02 — Autenticidade e Integridade de Chamadas de Webhook de Terceiros

* **ID**: WHK-02
* **Pergunta**: O controle Autenticidade e Integridade de Chamadas de Webhook de Terceiros está implementado e operante na instância do n8n?
* **Como verificar**: Enviar um payload de teste com assinatura HMAC alterada e confirmar que o workflow rejeita o processamento com erro 401/403.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Código do workflow demonstrando a etapa de validação HMAC e logs de erro para assinaturas inválidas.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Falsificação de chamadas de webhook por atacantes, adulteração de payloads em trânsito e ataques de repetição (*Replay Attacks*).
* **Ação corretiva**: Utilizar a validação de assinatura nativa do nó de webhook ou inserir um nó de código no início do fluxo calculando `crypto.createHmac('sha256', secret).update(rawBody).digest('hex')` e comparando com o cabeçalho recebido.
* **Prioridade**: P0

### Item Checklist: WHK-03 — Restrição de Origem e Limitação de Taxa para Webhooks

* **ID**: WHK-03
* **Pergunta**: O controle Restrição de Origem e Limitação de Taxa para Webhooks está implementado e operante na instância do n8n?
* **Como verificar**: Executar testes de estresse com a ferramenta `ab` ou `k6` disparando requisições em massa contra a URL do webhook e confirmando o bloqueio com código HTTP 429 (Too Many Requests).
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Configuração do WAF/Proxy e relatório de teste de carga confirmando o bloqueio por Rate Limiting.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Ataques de Negação de Serviço (DDoS), inundações por bots e varreduras automatizadas na porta de webhooks.
* **Ação corretiva**: Configurar módulos de Rate Limiting no Nginx (`limit_req_zone`) ou regras de WAF/Cloudflare limitando requisições na rota `/webhook/*` a valores compatíveis com a operação normal (ex: 10 req/s por IP).
* **Prioridade**: P1

### Item Checklist: DTP-01 — Ocultação de Dados Sensíveis na Interface de Histórico de Execuções

* **ID**: DTP-01
* **Pergunta**: O recurso de Redação de Dados de Execução (Execution Data Redaction) está ativo para ocultação de PII na UI?
* **Como verificar**: Abrir o histórico de execuções de um workflow com dados sensíveis com uma conta de usuário padrão e verificar que os dados aparecem ocultos com asteriscos/mascarados.
* **Comando/procedimento, quando suportado pelas fontes**: curl -s -H "X-N8N-API-KEY: $N8N_API_KEY" http://localhost:5678/api/v1/settings
* **Resultado esperado**: Execution Data Redaction ativado nas configurações da instância.
* **Evidência**: Captura de tela do histórico de execuções demonstrando o mascaramento de dados ativado.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Exposição indevida de dados pessoais e segredos para operadores e desenvolvedores que visualizam o histórico de execuções na interface gráfica do n8n.
* **Ação corretiva**: Configurar a opção de redação de dados de execução nas propriedades globais da instância ou nas configurações individuais do workflow (Enterprise).
* **Prioridade**: P0

### Item Checklist: DTP-02 — Limpeza Automática do Histórico de Dados de Execução no Banco

* **ID**: DTP-02
* **Pergunta**: O expurgamento automático de histórico (EXECUTIONS_DATA_MAX_AGE) está configurado entre 7 e 30 dias?
* **Como verificar**: Consultar a tabela `execution_entity` no banco de dados e verificar se não existem registros com data de criação superior ao limite de dias configurado.
* **Comando/procedimento, quando suportado pelas fontes**: docker exec n8n-main env | grep EXECUTIONS_DATA_MAX_AGE
* **Resultado esperado**: EXECUTIONS_DATA_MAX_AGE configurado entre 168 e 720 horas (7 a 30 dias).
* **Evidência**: Query SQL executada no PostgreSQL confirmando a ausência de registros antigos.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Acúmulo desnecessário de dados pessoais (violação do princípio de limitação do armazenamento da LGPD), crescimento excessivo do banco de dados e vazamento massivo de histórico em caso de invasão.
* **Ação corretiva**: Configurar as variáveis de ambiente `EXECUTIONS_DATA_PRUNE=true`, `EXECUTIONS_DATA_MAX_AGE=168` (em horas) e `EXECUTIONS_DATA_PRUNE_MAX_COUNT=50000`.
* **Prioridade**: P0

### Item Checklist: DTP-03 — Proteção de Dados em Repouso no Armazenamento Físico

* **ID**: DTP-03
* **Pergunta**: A criptografia de disco em repouso (AES-256) está ativada na partição de dados do servidor e do PostgreSQL?
* **Como verificar**: Consultar as propriedades do volume de armazenamento no painel do provedor de nuvem e confirmar o status "Encrypted: True".
* **Comando/procedimento, quando suportado pelas fontes**: lsblk -f
* **Resultado esperado**: Volume de disco marcado com criptografia AES-256 (crypto_LUKS / Cloud Encrypted).
* **Evidência**: Relatório de configuração do provedor de nuvem comprovando a criptografia ativa nos volumes de dados.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Furto físico de discos, acesso não autorizado a snapshots de volumes na nuvem e descarte inadequado de mídias de armazenamento.
* **Ação corretiva**: Ativar a opção de criptografia de volume (ex: AWS EBS Encryption, GCP Persistent Disk Encryption, LUKS em bare-metal) com chaves gerenciadas por KMS corporativo.
* **Prioridade**: P0

### Item Checklist: DTP-04 — Minimização e Remoção Contínua de Dados Pessoais nos Fluxos

* **ID**: DTP-04
* **Pergunta**: A comunicação entre o n8n e o banco de dados PostgreSQL exige conexão criptografada via TLS/SSL?
* **Como verificar**: Inspecionar os payloads do fluxo no editor e confirmar que campos não essenciais são removidos na primeira etapa.
* **Comando/procedimento, quando suportado pelas fontes**: docker exec n8n-main env | grep DB_POSTGRESDB_SSL_ENABLED
* **Resultado esperado**: Conexão com o PostgreSQL exigindo SSL (sslmode=require).
* **Evidência**: Estrutura do workflow revisada e aprovada pelo DPO/SecOps.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Mapeamento e gravação desnecessária de PII em logs de sistemas secundários, descumprimento do princípio da minimização da LGPD e vazamentos indesejados.
* **Ação corretiva**: Configurar os nós de transformação utilizando a opção "Keep Only Set" para manter estritamente os atributos necessários para os passos seguintes do fluxo.
* **Prioridade**: P1

### Item Checklist: INF-01 — Desqualificação de Privilégios do Usuário do Contêiner

* **ID**: INF-01
* **Pergunta**: O contêiner do n8n executa sob o usuário node (1000) e o Task Runner sob o usuário nobody (65532)?
* **Como verificar**: Executar `docker exec <container_id> id` ou `kubectl exec` e verificar que o UID retornado é diferente de 0.
* **Comando/procedimento, quando suportado pelas fontes**: docker inspect --format '{{.Config.User}}' n8n-main n8n-runner
kubectl get pod -n n8n-prod <pod> -o jsonpath='{.spec.containers[*].securityContext.runAsUser}'
* **Resultado esperado**: Usuário node (1000) no orquestrador e nobody (65532) no Task Runner.
* **Evidência**: Saída do comando `id` dentro do contêiner confirmando UID 1000 ou 65532.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Escalação de privilégios para o hospedeiro, alteração de configurações do sistema operacional e mitigação de vulnerabilidades de escape de contêiner.
* **Ação corretiva**: Definir `user: "1000:1000"` no Docker Compose/K8s para a imagem `n8nio/n8n` e `user: "65532:65532"` (usuário `nobody`) para a imagem `n8nio/runners`.
* **Prioridade**: P0

### Item Checklist: INF-02 — Desacoplamento do Processo de Execução de Código do Orquestrador

* **ID**: INF-02
* **Pergunta**: O sistema de arquivos raiz do contêiner do Task Runner está montado como somente leitura (Read-Only)?
* **Como verificar**: Verificar se o contêiner do runner está rodando separadamente e testar se a imagem não possui binários de shell (`docker exec` no runner tentando rodar `/bin/sh` deve falhar).
* **Comando/procedimento, quando suportado pelas fontes**: docker inspect --format '{{.HostConfig.ReadonlyRootfs}}' n8n-runner
kubectl get pod -n n8n-prod <runner-pod> -o jsonpath='{.spec.containers[*].securityContext.readOnlyRootFilesystem}'
* **Resultado esperado**: ReadonlyRootfs=true no contêiner do Task Runner.
* **Evidência**: Configuração do manifesto Docker/K8s e teste de falha ao tentar invocar shell no runner.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Vulnerabilidades críticas de RCE e sandbox escape (CVE-2025-68668 N8Scape e CVE-2026-42234), impedindo que um código malicioso acesse o processo orquestrador do n8n.
* **Ação corretiva**: Configurar `N8N_RUNNERS_MODE=external` no orquestrador n8n e implantar o contêiner sidecar com a imagem `n8nio/runners:<ver>-distroless`, comunicando-se via porta interna 5679 com token de autenticação.
* **Prioridade**: P0

### Item Checklist: INF-03 — Imutabilidade do Sistema de Arquivos do Contêiner

* **ID**: INF-03
* **Pergunta**: O socket do Docker (/var/run/docker.sock) e diretórios do hospedeiro estão completamente ausentes dos contêineres/pods?
* **Como verificar**: Tentar criar um arquivo no diretório raiz do contêiner runner (`touch /test.txt`) e verificar a mensagem "Read-only file system".
* **Comando/procedimento, quando suportado pelas fontes**: docker inspect n8n-main n8n-runner | grep docker.sock
kubectl get pod -n n8n-prod <pod> -o jsonpath='{.spec.volumes[*].hostPath.path}'
* **Resultado esperado**: Zero montagens do docker.sock ou hostPath nos contêineres/pods.
* **Evidência**: Saída do teste de tentativa de escrita e manifesto de configuração do contêiner.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Gravação de malware, criação de scripts de persistência no sistema de arquivos do contêiner e alteração de bibliotecas de execução por atacantes.
* **Ação corretiva**: Configurar `readOnlyRootFilesystem: true` na especificação de segurança do contêiner do runner no Kubernetes ou `--read-only` no Docker.
* **Prioridade**: P0

### Item Checklist: INF-04 — Restrição de Chamadas de Sistema do Kernel

* **ID**: INF-04
* **Pergunta**: Limites de memória e CPU estão configurados em cgroups/resources.limits para todos os contêineres do n8n?
* **Como verificar**: Inspecionar as propriedades do pod/contêiner confirmando a remoção de capabilities e verificar o status do AppArmor via `aa-status` no host.
* **Comando/procedimento, quando suportado pelas fontes**: docker inspect --format '{{.HostConfig.Memory}} {{.HostConfig.NanoCpus}}' n8n-main
kubectl get pod -n n8n-prod <pod> -o jsonpath='{.spec.containers[*].resources}'
* **Resultado esperado**: Limites de memória e CPU configurados em cgroups / resources.limits.
* **Evidência**: Saída do comando `aa-status` exibindo o perfil ativo para o contêiner do n8n runner.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Leitura de variáveis de ambiente do processo diretamente da memória do kernel, bypass de sandbox e tentativas de exploração do kernel do hospedeiro.
* **Ação corretiva**: Aplicar o perfil AppArmor recomendado na documentação oficial do n8n contendo a regra `audit deny @{PROC}/*/{environ,mounts} rwl,` e adicionar `capabilities: { drop: ["ALL"] }` no manifesto do contêiner.
* **Prioridade**: P0

### Item Checklist: INF-05 — Bloqueio de Acesso ao Daemon de Contêineres do Hospedeiro

* **ID**: INF-05
* **Pergunta**: O banco de dados PostgreSQL está isolado em sub-rede privada e configurado com SSL obrigatório?
* **Como verificar**: Executar verificação estática nos manifestos de implantação via linter/Kyverno buscando por regras que bloqueiem `docker.sock`.
* **Comando/procedimento, quando suportado pelas fontes**: kubectl exec -it postgres-pod -- psql -U postgres -c "SHOW ssl;"
* **Resultado esperado**: PostgreSQL configurado com SSL ativo e usuário sem privilégio SUPERUSER.
* **Evidência**: Relatório do linter/controle de admissão confirmando a ausência da montagem do socket.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Escape imediato de contêiner para o hospedeiro com privilégios equivalentes a `root`, criação de contêineres maliciosos no servidor e comprometimento total do nó de infraestrutura.
* **Ação corretiva**: Auditar todos os arquivos `docker-compose.yml` e manifestos do Kubernetes garantindo a ausência de montagens do tipo `hostPath` apontando para `/var/run/docker.sock`.
* **Prioridade**: P0

### Item Checklist: INF-06 — Remoção de Credenciais de Acesso à API Server do K8s

* **ID**: INF-06
* **Pergunta**: O cluster Redis exige senha de autenticação (AUTH) e aceita conexões apenas da rede privada?
* **Como verificar**: Verificar que o diretório `/var/run/secrets/kubernetes.io/serviceaccount/` não existe dentro do pod do n8n.
* **Comando/procedimento, quando suportado pelas fontes**: redis-cli -h redis-host ping
* **Resultado esperado**: Redis respondendo com erro NOAUTH para conexões sem senha.
* **Evidência**: Saída do comando `kubectl exec` confirmando que o caminho de segredos do K8s não está montado no pod.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Roubo do token JWT do Kubernetes por um invasor com RCE no pod, prevenindo tentativas de autenticação e tomada de controle do API Server do cluster.
* **Ação corretiva**: Configurar `automountServiceAccountToken: false` no manifesto do objeto `ServiceAccount` e na especificação do Pod do n8n.
* **Prioridade**: P0

### Item Checklist: INF-07 — Alocação Controlada de Recursos para Prevenção de DoS

* **ID**: INF-07
* **Pergunta**: O controle Alocação Controlada de Recursos para Prevenção de DoS está implementado e operante na instância do n8n?
* **Como verificar**: Executar `kubectl top pods` ou `docker stats` para verificar se os contêineres estão operando dentro das cotas estabelecidas.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Dashboard do Prometheus/Grafana monitorando o consumo de recursos contra os limites definidos.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Esgotamento de memória do servidor (*Node OOM*), travamento do servidor por laços infinitos em scripts e ataques de Negação de Serviço por consumo excessivo de recursos.
* **Ação corretiva**: Configurar o bloco `resources: requests: {cpu: "500m", memory: "1Gi"}, limits: {cpu: "2000m", memory: "2Gi"}` no manifesto do contêiner.
* **Prioridade**: P0

### Item Checklist: SCM-01 — Banimento de Tags Flutuantes em Imagens de Produção

* **ID**: SCM-01
* **Pergunta**: O controle Banimento de Tags Flutuantes em Imagens de Produção está implementado e operante na instância do n8n?
* **Como verificar**: Inspecionar a lista de imagens em execução no cluster via `kubectl get pods -o jsonpath='{.items[*].spec.containers[*].image}'`.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Relatório do controle de admissão aprovando as tags de versão fixadas.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Introdução não testada de atualizações de software com quebras de compatibilidade, alterações não auditadas na imagem base e comportamentos imprevisíveis.
* **Ação corretiva**: Configurar a tag exata no arquivo de implantação e utilizar verificadores de política (como Kyverno/Conftest) no CI/CD para rejeitar manifestos com a tag `:latest`.
* **Prioridade**: P0

### Item Checklist: SCM-02 — Política de Aprovação e Isolamento de Nós da Comunidade

* **ID**: SCM-02
* **Pergunta**: O controle Política de Aprovação e Isolamento de Nós da Comunidade está implementado e operante na instância do n8n?
* **Como verificar**: Executar `n8n audit` para extrair a lista de todos os *Community Nodes* instalados e verificar se possuem aprovação formal no inventário de segurança.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Relatório do `n8n audit` com zero pacotes não autorizados e fichas de avaliação de código dos pacotes aprovados.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Injeção de código malicioso por pacotes npm comprometidos (*Supply Chain Attacks*), trojans em dependências de terceiros e exfiltração silenciosa de credenciais.
* **Ação corretiva**: Restringir a permissão de instalação de pacotes no RBAC do n8n e estabelecer a *Política de Governança de Community Nodes*, exigindo a compilação prévia e verificação do pacote em registro npm privado corporativo.
* **Prioridade**: P0

### Item Checklist: SCM-03 — Homologação de Código Próprio Desenvolvido para o n8n

* **ID**: SCM-03
* **Pergunta**: O controle Homologação de Código Próprio Desenvolvido para o n8n está implementado e operante na instância do n8n?
* **Como verificar**: Consultar os relatórios da ferramenta SAST no repositório do nó customizado confirmando a ausência de vulnerabilidades de severidade Alta ou Crítica.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Dashboard do SonarQube/Semgrep com aprovação técnica e historico de aprovações de Pull Request.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Introdução involuntária de vulnerabilidades de SQLi, Command Injection, gravação insegura de arquivos ou vazamentos de memória em nós customizados.
* **Ação corretiva**: Incluir etapas automáticas de varredura com SonarQube/Semgrep na esteira de CI/CD do repositório de custom nodes e configurar o diretório isolado `N8N_CUSTOM_EXTENSIONS`.
* **Prioridade**: P1

### Item Checklist: SCM-04 — Validação de Integridade e Autenticidade da Imagem do Contêiner

* **ID**: SCM-04
* **Pergunta**: O controle Validação de Integridade e Autenticidade da Imagem do Contêiner está implementado e operante na instância do n8n?
* **Como verificar**: Simular o deploy de uma imagem não assinada no cluster e verificar a rejeição automática pelo controlador de admissão.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Logs do Policy Controller no Kubernetes confirmando a verificação de assinatura bem-sucedida.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Substituição maliciosa da imagem do n8n por imagens adulteradas em registros de contêineres intermediários (*Image Spoofing/Tampering*).
* **Ação corretiva**: Configurar a política do Kyverno / Sigstore Policy Controller no Kubernetes para verificar a chave pública de assinatura do repositório oficial da n8n GmbH.
* **Prioridade**: P2

### Item Checklist: CICD-01 — Sanitização Automática de Segredos na Exportação de Workflows

* **ID**: CICD-01
* **Pergunta**: O controle Sanitização Automática de Segredos na Exportação de Workflows está implementado e operante na instância do n8n?
* **Como verificar**: Executar script de inspeção em busca de strings no formato de chaves de API conhecidas nos arquivos `.json`/`.n8np` do repositório Git.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Arquivos de workflows no repositório Git auditados e aprovados pela ferramenta de verificação.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Inclusão acidental de chaves de API, senhas e tokens OAuth2 em texto claro nos arquivos JSON de workflows comitados em repositórios de código.
* **Ação corretiva**: Utilizar a API oficial do n8n ou a CLI `n8n export:workflow` (sem a flag `--decrypted`), garantindo que o arquivo exportado contenha apenas o campo `credentials: { id: "123", name: "stripe-account" }` sem segredos brutos.
* **Prioridade**: P0

### Item Checklist: CICD-02 — Detecção de Vazamento de Credenciais na Esteira de Código

* **ID**: CICD-02
* **Pergunta**: O controle Detecção de Vazamento de Credenciais na Esteira de Código está implementado e operante na instância do n8n?
* **Como verificar**: Criar um branch de teste com um token fictício e validar o bloqueio do Pull Request pela ferramenta de varredura.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Logs de execução do pipeline do CI/CD comprovando a passagem da varredura sem alertas.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Exposição de credenciais da empresa, chaves `N8N_ENCRYPTION_KEY` ou tokens corporativos no histórico de commits do repositório Git.
* **Ação corretiva**: Adicionar uma etapa obrigatória no pipeline do GitHub Actions / GitLab CI executando `gitleaks detect --source . --verbose`, bloqueando o *merge* do Pull Request em caso de identificação de segredos.
* **Prioridade**: P0

### Item Checklist: CICD-03 — Governança da Esteira de Deploy de Automações

* **ID**: CICD-03
* **Pergunta**: O controle Governança da Esteira de Deploy de Automações está implementado e operante na instância do n8n?
* **Como verificar**: Tentar realizar um `git push` direto para o branch principal e confirmar a rejeição pelo servidor Git.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Configuração de proteção de branch do repositório Git exportada.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Alterações não auditadas em ambiente de produção, injeção de pipelines maliciosos e contaminação de ambientes por deploys diretos.
* **Ação corretiva**: Configurar proteção de branches principais (`main`/`production`) no repositório Git exigindo *PR Review* obrigatório e passar a chave de implantação via Service Account do pipeline.
* **Prioridade**: P1

### Item Checklist: MON-01 — Envio Contínuo do Barramento de Logs de Auditoria para SIEM

* **ID**: MON-01
* **Pergunta**: O controle Envio Contínuo do Barramento de Logs de Auditoria para SIEM está implementado e operante na instância do n8n?
* **Como verificar**: Realizar uma ação auditada na interface (ex: login ou alteração de credencial) e consultar o evento correspondente no painel do SIEM dentro de 60 segundos.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Dashboard do SIEM exibindo os eventos estruturados `n8n.audit.user.login`, `n8n.audit.workflow.updated`, etc.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Perda de trilhas de auditoria em caso de destruição do contêiner, cegueira operacional sobre ações de usuários e incapacidade de realizar perícia pós-incidente.
* **Ação corretiva**: Configurar a escrita de logs de auditoria em arquivo dedicado (`N8N_EVENTBUS_LOGWRITER_LOGFULLPATH=/var/log/n8n/n8nEventLog.log`) e integrar um agente de transporte (FluentBit/Logstash) enviando os dados via protocolo criptografado para o SIEM.
* **Prioridade**: P0

### Item Checklist: MON-02 — Padronização do Nível de Detalhamento dos Logs Operacionais

* **ID**: MON-02
* **Pergunta**: O controle Padronização do Nível de Detalhamento dos Logs Operacionais está implementado e operante na instância do n8n?
* **Como verificar**: Inspecionar os logs de execução e confirmar que não constam mensagens de nível `debug` no tráfego operacional comum.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Tabela de variáveis do processo e amostra de logs de produção auditada.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Gravação acidental de dados de cargas de trabalho, payloads completos de requisições e potenciais segredos em arquivos de log operacionais em disco.
* **Ação corretiva**: Configurar a variável de ambiente `N8N_LOG_LEVEL=info` e definir `N8N_LOG_STREAMING_MANAGED_BY_ENV=true` para impedir a alteração não autorizada do nível de log pela interface gráfica.
* **Prioridade**: P0

### Item Checklist: MON-03 — Mascaramento de Identificadores Pessoais nas Trilhas de Auditoria

* **ID**: MON-03
* **Pergunta**: O controle Mascaramento de Identificadores Pessoais nas Trilhas de Auditoria está implementado e operante na instância do n8n?
* **Como verificar**: Inspecionar uma mensagem do evento `n8n.audit.user.login` no SIEM e verificar que o campo de e-mail aparece anonimizado/hasheado.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Registro de log formatado no SIEM comprovando o mascaramento de identificadores.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Vazamento de dados pessoais de funcionários/operadores para sistemas de monitoramento e desconformidade com o princípio da minimização da LGPD nos logs.
* **Ação corretiva**: Configurar o parâmetro `anonymizeAuditMessages=true` nas configurações do coletor de logs do n8n.
* **Prioridade**: P1

### Item Checklist: MON-04 — Observabilidade Técnica de Infraestrutura e Propagação de Contexto

* **ID**: MON-04
* **Pergunta**: O controle Observabilidade Técnica de Infraestrutura e Propagação de Contexto está implementado e operante na instância do n8n?
* **Como verificar**: Acessar o endpoint interno `/metrics` e verificar a presença de métricas do n8n como `n8n_workflows_execution_time_seconds`.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Dashboard do Grafana ativo exibindo métricas de execuções do n8n e rastros no Jaeger/OpenTelemetry.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Falta de visibilidade sobre gargalos de desempenho, picos de erros e falta de rastreabilidade de requisições que cruzam múltiplos microserviços corporativos.
* **Ação corretiva**: Habilitar as variáveis `N8N_METRICS=true`, `N8N_METRICS_INCLUDE_QUEUE_METRICS=true` e proteger a rota `/metrics` com restrição de rede para consumo exclusivo do servidor Prometheus.
* **Prioridade**: P1

### Item Checklist: VUL-01 — Verificação Automática de Configurações Inseguras da Instância

* **ID**: VUL-01
* **Pergunta**: O comando n8n audit é executado periodicamente para diagnóstico de riscos e vulnerabilidades na instância?
* **Como verificar**: Consultar o histórico de execução da tarefa de auditoria e validar o conteúdo do relatório retornado.
* **Comando/procedimento, quando suportado pelas fontes**: n8n audit
* **Resultado esperado**: Comando n8n audit executado sem erros críticos reportados.
* **Evidência**: Arquivo JSON do relatório do `n8n audit` gerado na última semana e arquivado na pasta de conformidade.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Permanência de configurações inseguras ativas por tempo indeterminado, presença de webhooks não autenticados e uso de credenciais não utilizadas.
* **Ação corretiva**: Criar uma tarefa cron na infraestrutura ou um workflow administrativo de segurança que executa `n8n audit` semanalmente e envia o resultado em formato JSON para análise da equipe de SecOps.
* **Prioridade**: P0

### Item Checklist: VUL-02 — Análise Automática de Vulnerabilidades em Software e Contêineres

* **ID**: VUL-02
* **Pergunta**: Varreduras automatizadas de vulnerabilidade na imagem do contêiner são realizadas semanalmente?
* **Como verificar**: Inspecionar o dashboard da ferramenta de varredura confirmando zero vulnerabilidades de severidade Crítica sem mitigação registrada.
* **Comando/procedimento, quando suportado pelas fontes**: trivy image n8nio/n8n:2.38.5
* **Resultado esperado**: Varredura de vulnerabilidades de contêiner executada semanalmente.
* **Evidência**: Relatório PDF/JSON emitido pelo Trivy/Grype atestando o status das imagens do n8n.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Execução de binários ou bibliotecas com vulnerabilidades conhecidas e exploráveis (CVEs públicas registradas na NVD/GHSA).
* **Ação corretiva**: Configurar tarefas agendadas no registro de contêineres ou no cluster (Trivy Operator) para varrer as imagens ativas do n8n e alertar em caso de CVEs Críticas.
* **Prioridade**: P0

### Item Checklist: VUL-03 — Política de Atualização e Aplicação de Correções de Segurança

* **ID**: VUL-03
* **Pergunta**: A versão do n8n é mantida atualizada com os patches e notas de lançamento de segurança oficiais?
* **Como verificar**: Consultar a versão atual da imagem em execução no cluster e comparar com a última versão estável disponibilizada pela n8n GmbH.
* **Comando/procedimento, quando suportado pelas fontes**: curl -s https://api.github.com/repos/n8n-io/n8n/releases/latest | grep tag_name
* **Resultado esperado**: Versão do n8n alinhada com os últimos lançamentos de correção de segurança.
* **Evidência**: Histórico de deploys do cluster comprovando a atualização regular da imagem do n8n.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Exploração de vulnerabilidades de software conhecidas já corrigidas pelo fornecedor em versões recentes.
* **Ação corretiva**: Acompanhar o canal oficial de notas de versão (*Changelog*) do n8n e avisos de segurança no blog oficial, executando a atualização em ambiente de Staging antes da promoção.
* **Prioridade**: P0

### Item Checklist: BDR-01 — Isolamento de Armazenamento do Cofre Criptografado e da Chave Mestra

* **ID**: BDR-01
* **Pergunta**: O controle Isolamento de Armazenamento do Cofre Criptografado e da Chave Mestra está implementado e operante na instância do n8n?
* **Como verificar**: Auditar as permissões e o conteúdo do bucket de backups do banco de dados confirmando a ausência da chave `N8N_ENCRYPTION_KEY` ou arquivos `.env`.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Relatório de configuração de permissões dos buckets de backup e diretrizes do cofre de chaves.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Vazamento massivo de todas as credenciais descriptografadas da organização caso o arquivo de backup do banco de dados seja interceptado ou acessado indevidamente.
* **Ação corretiva**: Armazenar os backups do PostgreSQL em um bucket de backup dedicado com controle de acesso restrito, e armazenar o registro da `N8N_ENCRYPTION_KEY` exclusivamente no Vault/KMS corporativo mantido por equipe distinta.
* **Prioridade**: P0

### Item Checklist: BDR-02 — Proteção e Retenção Segura de Cópias de Segurança

* **ID**: BDR-02
* **Pergunta**: O controle Proteção e Retenção Segura de Cópias de Segurança está implementado e operante na instância do n8n?
* **Como verificar**: Tentar alterar ou excluir um arquivo de backup armazenado no bucket de retenção e verificar a mensagem de bloqueio por política de imutabilidade.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Logs da ferramenta de backup atestando a criptografia e status de Object Lock no bucket S3.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Exposição do conteúdo do banco em caso de vazamento do arquivo de backup e destruição ou criptografia dos backups por ataques de ransomware.
* **Ação corretiva**: Configurar a ferramenta de backup (ex: pg_dump via pipeline criptografado com GPG/AES-256) enviando o arquivo para um bucket S3 com a opção "Object Lock" ativada com retenção mínima de 30 dias.
* **Prioridade**: P0

### Item Checklist: BDR-03 — Validação Periódica da Capacidade de Recuperação do n8n

* **ID**: BDR-03
* **Pergunta**: O controle Validação Periódica da Capacidade de Recuperação do n8n está implementado e operante na instância do n8n?
* **Como verificar**: Verificar a execução do workflow de teste na instância restaurada e o relatório de validação da recuperação.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Relatório assinado de teste de Disaster Recovery homologado com o tempo total de recuperação (RTO/RPO mensurado).
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Descobrir que os backups do banco de dados estão corrompidos, incompletos ou incompatíveis com a chave de criptografia no momento de um incidente real.
* **Ação corretiva**: Criar uma automação que lê o último backup do banco, sobre uma instância de testes limpa do n8n, injeta a chave de criptografia e executa um workflow de teste validando que as credenciais são descriptografadas com sucesso.
* **Prioridade**: P1

### Item Checklist: IR-01 — Procedimentos Formais de Contenção, Erradicação e Recuperação

* **ID**: IR-01
* **Pergunta**: O controle Procedimentos Formais de Contenção, Erradicação e Recuperação está implementado e operante na instância do n8n?
* **Como verificar**: Executar um exercício de simulação de incidente (*Tabletop Exercise*) testando a capacidade da equipe de seguir os passos documentados.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Documento do Plano de Resposta a Incidentes atualizado e ata do exercício de simulação realizado.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Ações desordenadas durante um incidente, prolongamento do tempo de exposição e incapacidade de conter a movimentação lateral de um atacante.
* **Ação corretiva**: Documentar os playbooks de resposta específicos para os cenários: Comprometimento de Credencial, RCE em Task Runner, Injeção de Código em Workflow e Vazamento de Dados Pessoais.
* **Prioridade**: P0

### Item Checklist: IR-02 — Ação de Emergência para Invalidação de Segredos Expostos

* **ID**: IR-02
* **Pergunta**: O controle Ação de Emergência para Invalidação de Segredos Expostos está implementado e operante na instância do n8n?
* **Como verificar**: Simular a revogação de uma credencial de testes e confirmar que a chamada correspondente passa a retornar erro de autenticação 401 imediatamente.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Playbook de revogação de emergência testado e validado.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Uso continuado de tokens roubados a partir do n8n para acesso e exfiltração de dados diretamente nos sistemas corporativos de destino.
* **Ação corretiva**: Documentar a lista de contatos dos administradores de cada sistema integrado e manter scripts de emergência para revogação rápida de tokens via API do provedor (ex: AWS CLI, GitHub API).
* **Prioridade**: P0

### Item Checklist: IR-03 — Contenção Imediata de Nós de Infraestrutura Comprometidos

* **ID**: IR-03
* **Pergunta**: O controle Contenção Imediata de Nós de Infraestrutura Comprometidos está implementado e operante na instância do n8n?
* **Como verificar**: Executar teste de isolamento em ambiente de homologação verificando a perda imediata de conectividade de rede do pod afetado.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Script de isolamento emergencial de pod e playbook de contenção atested.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Continuidade da exfiltração de dados, movimentação lateral no cluster e destruição de evidências pelo atacante.
* **Ação corretiva**: Criar manifestos de `NetworkPolicy` de emergência (Isolamento Total) e scripts de remoção de nó do cluster de produção mantendo o pod em estado pausado para extração de snapshot de memória.
* **Prioridade**: P1

### Item Checklist: AI-01 — Intervenção Humana Decisória na Execução de Ações por Agentes de IA

* **ID**: AI-01
* **Pergunta**: O controle Intervenção Humana Decisória na Execução de Ações por Agentes de IA está implementado e operante na instância do n8n?
* **Como verificar**: Disparar uma requisição para o agente solicitando uma alteração no banco e verificar se o fluxo trava na etapa de aprovação aguardando a interação do operador.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Estrutura do workflow com Agente de IA demonstrando o nó HITL e logs de auditoria registrando o e-mail do aprovador humano.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Execução não intencional de ações destrutivas ou indesejadas causadas por alucinação do modelo, injeção de prompt ou manipulação de parâmetros por usuários.
* **Ação corretiva**: Inserir o nó de aprovação por formulário/e-mail (HITL) nas ramificações de ferramentas de alta criticidade do agente, interrompendo a execução do fluxo até o clique de confirmação do operador responsável.
* **Prioridade**: P0

### Item Checklist: AI-02 — Filtragem Rígida dos Parâmetros Gerados por Modelos de IA

* **ID**: AI-02
* **Pergunta**: O controle Filtragem Rígida dos Parâmetros Gerados por Modelos de IA está implementado e operante na instância do n8n?
* **Como verificar**: Testar a ferramenta enviando um parâmetro contendo caracteres de injeção e verificar a rejeição pelo nó de validação do esquema.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Esquema JSON de validação configurado no sub-workflow da ferramenta.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: *Tool Manipulation*, injeção de comandos SQL/Shell via argumentos gerados pelo LLM e alteração de parâmetros de negócios.
* **Ação corretiva**: Criar sub-workflows como ferramentas que recebem a chamada do LLM e executam nós de validação de esquema JSON (*JSON Schema Validator*) e sanitização antes de disparar a ação externa.
* **Prioridade**: P0

### Item Checklist: AI-03 — Isolamento entre Instruções do Sistema e Dados Não Confiáveis

* **ID**: AI-03
* **Pergunta**: O controle Isolamento entre Instruções do Sistema e Dados Não Confiáveis está implementado e operante na instância do n8n?
* **Como verificar**: Enviar e-mails e payloads contendo instruções maliciosas de override de contexto e verificar se o agente ignora o comando malicioso mantendo sua instrução original.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Ficha técnica do System Prompt auditado e logs de testes de injeção efetuados.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Injeção de Prompt Direta e Indireta (*Indirect Prompt Injection*), sequestro de raciocínio do agente e vazamento do System Prompt para usuários externos.
* **Ação corretiva**: Utilizar sintaxe de delimitação explícita (ex: `<user_input> {{ $json.body }} </user_input>`), instruir o LLM a tratar o conteúdo como texto passivo e utilizar nós de integração com modelos de guarda (Llama Guard / ShieldGemma).
* **Prioridade**: P0

### Item Checklist: AI-04 — Monitoramento e Registro Detalhado da Operação de Agentes de IA

* **ID**: AI-04
* **Pergunta**: O controle Monitoramento e Registro Detalhado da Operação de Agentes de IA está implementado e operante na instância do n8n?
* **Como verificar**: Consultar no SIEM o histórico do evento `n8n.audit.mcp.tool.called` confirmando o registro dos parâmetros do usuário e do status da execução.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Configuração ou parâmetro conforme esperado no baseline.
* **Evidência**: Dashboard no SIEM monitorando o uso de ferramentas por Agentes de IA e alertas de erro em ferramentas.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Frequente opacidade sobre as ações decididas autonomamente pelos Agentes de IA e ausência de rastreabilidade para investigação de danos causados por LLMs.
* **Ação corretiva**: Garantir que o barramento de logs de auditoria do n8n esteja coletando os eventos `n8n.audit.mcp.tool.called`, `n8n.audit.llm.generated` e direcionando-os ao SIEM centralizado.
* **Prioridade**: P1

### Item Checklist: PRV-01 — Mapeamento do Fluxo de Dados Pessoais nos Workflows

* **ID**: PRV-01
* **Pergunta**: Workflows que processam dados pessoais aplicam o princípio da minimização coletando apenas campos estritamente necessários?
* **Como verificar**: Análise anual do inventário de workflows em relação aos relatórios de impacto à proteção de dados pessoais (RIPD/DPIA).
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Workflows configurados para coletar apenas dados pessoais estritamente necessários.
* **Evidência**: Documento de Registro das Operações de Tratamento de Dados (ROPA) cobrindo as automações do n8n.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Desconformidade com o Artigo 37 da LGPD (Registro das Operações de Tratamento) e incapacidade de responder a auditorias da Autoridade Nacional de Proteção de Dados (ANPD).
* **Ação corretiva**: Incorporar os campos de adequação à LGPD no inventário corporativo de workflows, mapeando se a automação trata dados pessoais comuns ou sensíveis e identificando a base legal correspondente.
* **Prioridade**: P0

### Item Checklist: PRV-02 — Atendimento a Solicitações de Exclusão, Acesso e Portabilidade

* **ID**: PRV-02
* **Pergunta**: As políticas de retenção de dados e anonimização de logs estão ativas para conformidade com a LGPD?
* **Como verificar**: Simular uma solicitação de exclusão de titular em ambiente de homologação e verificar que os dados do titular são totalmente removidos dos sistemas de destino.
* **Comando/procedimento, quando suportado pelas fontes**: docker exec n8n-main env | grep -E "EXECUTIONS_DATA_MAX_AGE|anonymizeAuditMessages"
* **Resultado esperado**: Retenção de dados e anonimização de logs ativas.
* **Evidência**: Relatório de execução do workflow de atendimento ao titular comprovando o expurgamento.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Descumprimento do Artigo 18 da LGPD (Direitos do Titular) e imposição de multas administrativas pela ANPD.
* **Ação corretiva**: Criar um workflow mestre de privacidade no n8n que recebe o identificador do titular (ex: CPF/e-mail) e dispara chamadas de expurgamento/exportação em todos os sistemas integrados.
* **Prioridade**: P0

### Item Checklist: PRV-03 — Gestão do Fluxo Transfronteiriço de Dados Pessoais

* **ID**: PRV-03
* **Pergunta**: Existe procedimento formal para atendimento aos direitos do titular de dados e mapeamento de subprocessadores de IA?
* **Como verificar**: Análise contratual dos DPAs firmados com os fornecedores de APIs externas e SaaS utilizados nos workflows do n8n.
* **Comando/procedimento, quando suportado pelas fontes**: Procedimento de verificação não especificado nas fontes.
* **Resultado esperado**: Fluxo documentado para atendimento aos direitos do titular e mapeamento de subprocessadores.
* **Evidência**: DPAs e Cláusulas-Padrão Contratuais assinadas com os provedores internacionais de SaaS/LLM.
* **OK/NOK**: [ ] OK   [ ] NOK
* **Risco**: Transferência internacional de dados desalinhada do Artigo 33 da LGPD e uso de subprocessadores sem garantias de privacidade equivalentes.
* **Ação corretiva**: Auditar os destinos de saída dos nós de integração (HTTP Request, OpenAI, Anthropic, Google) e exigir que os fornecedores internacionais assinem os Termos de Processamento de Dados (DPA) com Cláusulas-Padrão.
* **Prioridade**: P1


---

## Instruções de Execução para o Auditor

1. **Ambiente de Execução**: Executar este checklist no servidor hospedeiro Linux (via SSH) ou na estação de gerenciamento DevOps com acesso ao cluster Kubernetes (`kubectl`).

2. **Coleta de Evidências**: Salvar a saída de cada comando executado em um arquivo de texto compactado (`audit-evidences-<data>.tar.gz`) acompanhado deste checklist preenchido.

3. **Itens Não Especificados**: Para itens cujo procedimento esteja indicado como *"Procedimento de verificação não especificado nas fontes."*, utilizar validação documental, entrevista com os responsáveis ou checagem manual na interface de administração do n8n.
