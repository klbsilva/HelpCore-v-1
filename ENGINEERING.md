# ENGINEERING.md — HelpCore

**Versão:** 0.1 — proposta técnica para validação  
**Status:** não autoriza implementação  
**Fonte funcional:** `docs/SPECIFICATION-v1.0.md`

## 1. Autoridade e protocolo

Este documento orienta agentes e desenvolvedores. A especificação funcional v1.0 registra decisões aprovadas pelo Product Owner. Quando houver conflito, não presumir: registrar a divergência e solicitar decisão.

Papéis:
- Kléber: Product Owner, validação funcional e aprovação de escopo.
- ChatGPT/Tom: arquitetura, engenharia, especificações, segurança, testes e revisão.
- Claude: implementação estritamente dentro do escopo aprovado.
- Kléber e Felipe: administradores funcionais do HelpCore.

**Portão obrigatório:** Claude não deve implementar código, telas, migrations ou infraestrutura até que Kléber aprove expressamente a especificação técnica detalhada e o plano de execução. A aprovação da especificação funcional v1.0, isoladamente, não libera codificação.

Fluxo: especificar → revisar → aprovar → implementar uma etapa → testar → revisar → aprovar próxima etapa.

## 2. Produto e escopo arquitetural

HelpCore é um sistema interno de gestão de serviços de TI para a Futura 7. A primeira versão é um monólito modular, com fronteiras claras entre domínios. Não iniciar com microserviços.

Domínios previstos:
- identidade, autenticação, sessões e autorização;
- usuários, supervisores, equipes e funcionários afetados;
- catálogo de serviços;
- chamados, ciclos de atendimento, atribuições e histórico;
- mensagens e anexos;
- incidentes compartilhados;
- comunicados e confirmações de leitura;
- disponibilidade de serviços;
- calendário e expediente;
- SLA, relógios e jobs;
- dashboards, relatórios e exportações;
- auditoria;
- notificações e preferências/canais.

## 3. Stack aprovada

- Front-end: React + TypeScript.
- Back-end: Node.js + Express.
- Persistência relacional: PostgreSQL.
- Interface de integração: API REST.
- Estilização: Tailwind CSS.
- Atualizações em tempo real: WebSockets.
- Arquivos: object storage privado; PostgreSQL guarda metadados e referência de armazenamento.

Não introduzir framework, ORM, biblioteca de autenticação, provedor cloud ou serviço externo sem registrar justificativa, impacto e aprovação técnica.

## 4. Princípios de arquitetura

1. Monólito modular; módulos não acessam tabelas ou regras internas de outros módulos sem contratos definidos.
2. Regras de negócio e autorização vivem no servidor, nunca apenas na interface.
3. Rotas/controllers devem ser finos; validação, casos de uso e persistência separados.
4. Operações que alteram múltiplas entidades devem usar transações.
5. Eventos de domínio devem ser usados para desacoplar notificações, auditoria e atualizações em tempo real.
6. Jobs agendados devem ser idempotentes, observáveis e seguros para reexecução.
7. Datas persistidas em UTC; apresentação e regras de expediente em `America/Sao_Paulo` (UTC−3 conforme regra funcional aprovada, sujeito à validação técnica de timezone).
8. Toda regra temporal deve ser testável com relógio injetável, sem depender do horário real do servidor.
9. Não armazenar anexos no filesystem efêmero do servidor de aplicação.
10. Não criar dependência de um computador ligado na Futura 7.

## 5. Autorização e visibilidade

Perfis iniciais: administrador e supervisor. O modelo deve permitir perfil técnico adicional no futuro. Kléber e Felipe são administradores equivalentes. Na V1, Kléber é o técnico responsável padrão; novos chamados são atribuídos a ele.

A autorização deve ser aplicada em cada consulta e operação do servidor. Não confiar em IDs enviados pelo cliente.

Regra crítica de visibilidade: a visibilidade dos chamados relacionados a um funcionário acompanha o vínculo atual desse funcionário com a equipe/supervisor. Transferir funcionário deve preservar histórico e solicitante original, mas recalcular autorização com base no vínculo atual. Kléber e Felipe têm visão global. Implementar transferência de forma transacional e auditar antes/depois.

Funcionários afetados não possuem login. Supervisores não têm auto cadastro. Somente administradores gerenciam contas, equipes e funcionários.

## 6. Dados e integridade

O modelo de dados deverá ser especificado antes das migrations. Entidades conceituais mínimas:
- User, Role/Permission ou mecanismo equivalente;
- SupervisorTeam e Employee;
- EmployeeTeamAssignment com vigência/histórico;
- ServiceCatalogItem;
- Ticket e TicketCycle;
- TicketAssignmentPeriod;
- TicketStatusHistory e TicketPriorityHistory;
- TicketMessage e Attachment;
- TicketEditRequest, TicketCancellationRequest e ReopenRequest;
- Incident e IncidentTicket;
- Announcement e AnnouncementAcknowledgement;
- ServiceStatusHistory;
- BusinessCalendar, Holiday/Exception;
- Notification;
- AuditEvent;
- SLA/metric snapshots ou eventos suficientes para cálculo reproduzível.

Não fixar prematuramente nomes/colunas ou ORM antes do diagrama lógico e das decisões técnicas. IDs devem ser não enumeráveis externamente (UUID recomendado, sujeito à aprovação). Integridade referencial e restrições devem proteger invariantes críticas.

## 7. Chamados, ciclos e histórico

- Protocolo principal permanece após reabertura.
- Cada reabertura cria novo ciclo numerado (ex.: HC-0012-R1).
- Eventos e métricas devem identificar o ciclo.
- Não apagar chamados definitivamente pela interface.
- Arquivamento não elimina histórico nem anexos.
- Alterações de prioridade/status/atribuição e operações administrativas devem gerar histórico e auditoria.
- A primeira resposta exige ação humana do técnico; confirmações automáticas não contam.
- Entrar em “Em atendimento” registra início do atendimento.
- Atribuições e períodos individuais de técnicos devem ser preservados para métricas futuras.

## 8. Relógios, SLA e calendário

Expediente aprovado: segunda a sexta, 09:00–16:50, fuso de Brasília; feriados e exceções configuráveis pelos administradores.

SLA de primeira resposta:
- Baixa: 2 dias úteis.
- Média: 1 dia útil.
- Alta: 4 horas úteis.
- Crítica: imediata.

SLA começa no próximo expediente se o chamado for aberto fora do horário. A meta é de primeira resposta, não de resolução.

A janela de confirmação após resolução é de 12 horas úteis. Sem resposta no prazo, o sistema fecha automaticamente e registra fechamento automático. Confirmação e contestação devem ser registradas separadamente.

O relógio total não para enquanto aguarda supervisor. A espera deve ser mensurada separadamente, sem apagar ou descontar o tempo decorrido. Métricas de tempo total, espera e períodos de atendimento devem permanecer distinguíveis e calculáveis por ciclo. A definição exata das fórmulas e dos campos será detalhada na especificação técnica antes da implementação.

Jobs de fechamento automático, SLA e notificações devem ser idempotentes, tolerar reinício e recuperar atrasos após indisponibilidade.

## 9. Segurança

- HTTPS obrigatório em produção.
- Senhas armazenadas somente com hash moderno e parâmetros adequados; nunca texto puro.
- Senha provisória com troca obrigatória no primeiro acesso.
- Sessão expira após 3 horas de inatividade.
- 5 tentativas consecutivas incorretas: bloqueio temporário de 15 minutos; limites por IP devem evitar abuso/negação de serviço.
- Cookies de sessão seguros (HttpOnly, Secure, SameSite adequado) se a estratégia escolhida usar cookies.
- Proteção contra CSRF quando aplicável, XSS, SQL injection, brute force, IDOR, upload malicioso e abuso de endpoints.
- Validação de entrada no servidor e respostas sem stack traces em produção.
- Segredos apenas em secret manager/variáveis de ambiente seguras; nunca no Git.
- Princípio do menor privilégio para aplicação, banco, storage e deploy.
- Logs não devem conter senhas, tokens, conteúdo privado desnecessário ou URLs assinadas.

## 10. Auditoria

Auditoria append-only pela aplicação. A conta de banco usada pelo app deve poder inserir e consultar eventos, mas não editar ou apagar registros de auditoria. Proteger também por privilégios e mecanismos no banco, não apenas por convenção no código.

Registrar, no mínimo: autenticação, sessões, abertura e alterações de chamados, status, prioridade, atribuições, mensagens, anexos, usuários, transferências, incidentes, comunicados, catálogo, disponibilidade, calendário, exportações e consultas sensíveis.

A especificação técnica deverá definir estrutura do evento, correlação, retenção, integridade, acesso e procedimento de auditoria administrativa. Reconhecer que superusuário da infraestrutura/banco pode contornar controles; mitigar com separação de funções, backups e monitoramento.

## 11. Anexos

Tipos aceitos: JPG/JPEG/PNG, PDF, DOC/DOCX, XLS/XLSX, TXT, MP4/MOV. Até 5 arquivos por envio. Limites: imagem 10 MB, documento 20 MB, vídeo 100 MB.

Validar extensão, MIME e assinatura real (magic bytes), limites, nomes e conteúdo. Armazenar em object storage privado. Download sempre autorizado pelo back-end; usar URLs assinadas de curta duração quando adequado. Nunca publicar bucket/objeto. Guardar autor, data/hora, tamanho, tipo e vínculo. A política de retenção e limite total de storage ainda precisa de decisão.

## 12. Notificações e tempo real

E-mail e notificações internas para eventos definidos na especificação funcional. E-mail de crítico imediato para o endereço aprovado na especificação. Comunicados gerais enviados a supervisores ativos sem revelar dados privados de chamados.

WebSockets servem para atualização de UI, não como fonte de verdade. Reconexão deve recuperar estado via API. Notificações devem ser persistidas e despachadas de modo resiliente; evitar duplicidade por chave de idempotência.

Provedor SMTP/e-mail, domínio, credenciais, política de retry e templates permanecem pendentes.

## 13. API

API REST versionada (prefixo recomendado `/api/v1`). Definir contratos OpenAPI antes da implementação. Padrões obrigatórios:
- validação de payload e parâmetros;
- paginação e ordenação explícitas;
- filtros com allowlist;
- erros consistentes e códigos HTTP corretos;
- autorização por recurso;
- idempotência em operações sensíveis quando aplicável;
- limites de taxa;
- documentação de endpoints e exemplos sem dados reais.

## 14. Front-end e experiência

React + TypeScript. Tema escuro como padrão, paleta neutra azul escuro/preto e azul para ações. Desktop prioritário; tablet e celular responsivos desde V1.

Experiências distintas:
- supervisor: início simples com “Abrir chamado” e “Acompanhar chamados”;
- TI: fila segmentada em Novos, Em atendimento, Aguardando supervisor e Reaberturas; críticos destacados;
- administradores: dashboard global e gestão;
- NOC: tela cheia para tablet, sem menus administrativos.

Não construir telas antes de aprovação de fluxos, wireframes e contratos de API.

## 15. Testes obrigatórios

A especificação técnica deverá definir ferramentas e cobertura. No mínimo:
- testes unitários de regras de negócio;
- testes de integração com PostgreSQL real ou ambiente equivalente;
- testes de autorização e isolamento entre supervisores;
- teste de transferência de funcionário e visibilidade de histórico;
- testes de transação e concorrência;
- testes de ciclos/reabertura;
- testes de SLA e calendário (sexta-feira, feriados, fora do expediente, mudança de horário);
- testes dos 12h úteis e fechamento automático;
- testes de upload, magic bytes, limites e autorização de download;
- testes de auditoria append-only;
- testes de notificações/idempotência e reconexão WebSocket;
- testes ponta a ponta dos fluxos principais;
- testes de backup e restauração.

## 16. Deploy, operação e recuperação

Hospedagem externa, domínio, provedor de e-mail, RPO/RTO, capacidade e limites de storage ainda não foram definidos. Não escolher fornecedor por suposição.

Requisitos aprovados:
- acesso pela internet com HTTPS;
- backup diário criptografado;
- retenção dos últimos 30 dias;
- cópia em armazenamento separado do servidor principal;
- backup de banco e arquivos;
- monitoramento de falhas e alertas;
- teste periódico de restauração;
- ambiente de testes;
- atualização remota, versionamento, backup antes de mudanças críticas e rollback.

A especificação técnica deve propor opções e explicitar custos, riscos, RPO/RTO e responsabilidades antes da escolha.

## 17. Integrações futuras

DASH IT, Automação em Calls e inventário de computadores são integrações futuras. Não acoplar V1 a APIs ou estruturas ainda não especificadas. Projetar interfaces internas limpas e documentar eventos/contratos quando esses projetos forem detalhados.

## 18. Pendências para fechar a especificação técnica

1. Hospedagem/provedor e região.
2. Domínio e estratégia de DNS/TLS.
3. SMTP/provedor de e-mail e remetente.
4. Estratégia de sessão (cookie server-side ou token) e revogação.
5. ORM/query layer e estratégia de migrations.
6. Storage privado e limites totais/retenção de anexos.
7. RPO/RTO e política de backup/restore.
8. Observabilidade, logs, métricas e alertas.
9. Estratégia de deploy, CI/CD e rollback.
10. Definição formal das fórmulas de tempo por técnico/ciclo.
11. Modelo lógico do banco e contratos OpenAPI.
12. Wireframes e critérios de aceite por fluxo.

## 19. Instrução ao agente de implementação

Antes de qualquer código:
1. Ler integralmente este arquivo, a especificação funcional e o registro de decisões.
2. Apontar inconsistências, ambiguidades e riscos; não resolver decisões de produto por conta própria.
3. Propor arquitetura detalhada, modelo de dados, contratos API, estratégia de segurança, testes, deploy e plano incremental.
4. Aguardar aprovação explícita de Kléber.
5. Implementar somente a etapa aprovada, sem antecipar etapas.
