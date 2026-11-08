# HelpCore — Modelo lógico de dados v0.2

**Status:** proposta atualizada com decisões do Product Owner; aguardando revisão final  
**Projeto:** HelpCore / ITDESK  
**Banco previsto:** PostgreSQL  
**Arquitetura:** monólito modular  
**Importante:** este documento define o modelo conceitual. Não contém migrations nem autoriza implementação.

## 1. Princípios do modelo

1. PostgreSQL é a fonte oficial dos dados transacionais.
2. O modelo preserva histórico; não sobrescrever eventos relevantes.
3. Identidade de funcionário é independente de sua equipe atual.
4. Chamado, ciclo de atendimento, incidente e comunicado são conceitos distintos.
5. Autorização considera o vínculo atual do funcionário, conforme HC-067.
6. Operações críticas usam transações.
7. Datas são armazenadas em UTC (timestamptz); regras de expediente usam America/Sao_Paulo.
8. Entidades principais devem usar UUID como identificador interno, sujeito à aprovação técnica final. Protocolos legíveis de chamados são identificadores públicos separados.
9. Exclusão lógica/arquivamento quando necessário; chamados e auditoria não são apagados definitivamente pela interface.
10. Anexos binários ficam em object storage privado; PostgreSQL guarda metadados e chave do objeto.

## 2. Domínios e entidades

### 2.1 Identidade e autorização

- users: conta de acesso, username único para autenticação, nome de exibição, e-mail de contato obrigatório para supervisores ativos, hash de senha, estado, troca obrigatória de senha e timestamps. O username é o identificador de login dos supervisores; não exigir e-mail como nome de usuário.
- roles: perfis funcionais (administrador, supervisor, técnico).
- permissions: permissões atômicas.
- user_roles: associação de usuários e perfis.
- role_permissions: permissões por perfil.
- sessions: sessões ativas/revogadas, hash do identificador, última atividade, expiração e revogação.
- login_attempts ou eventos equivalentes: tentativas e bloqueios auditáveis.

Regras:
- Kléber e Felipe são administradores equivalentes.
- Supervisores não fazem auto cadastro.
- Não armazenar senha em texto puro.
- Funcionários afetados não precisam de conta em users.
- O modelo permite múltiplos técnicos futuros.

### 2.2 Organização e funcionários

- teams: equipe vinculada a exatamente um supervisor responsável na V1.
- employees: cadastro independente do vínculo atual; somente nome completo, situação ativo/inativo, data do cadastro e responsável pelo cadastro. Não haverá identificador interno. A equipe atual é derivada do vínculo vigente em employee_team_assignments.
- employee_team_assignments: histórico de vínculo, com employee_id, team_id, valid_from, valid_to, autor da mudança e motivo.

Invariante: cada supervisor possui exatamente uma equipe na V1; cada equipe possui exatamente um supervisor responsável. Todo funcionário ativo deve possuir exatamente um vínculo atual. Cadastro ou ativação sem equipe é bloqueado. Transferência fecha o vínculo anterior e abre o novo na mesma transação. Não pode haver janela persistida sem equipe ou vínculos atuais duplicados.

A autorização de chamados de funcionário é derivada do vínculo atual. O chamado continua guardando quem abriu originalmente e qual era o supervisor responsável à época.

### 2.3 Catálogo e serviços

- service_categories: categorias dos chamados.
- services: catálogo editável (Internet e rede, Telefonia/MicroSIP, Bitrix24, Sistemas internos, Computadores/notebooks, Impressoras/periféricos, E-mail corporativo, Outros).
- service_status_history: status operacional (Operacional, Instável, Indisponível), autor, início/fim e observação.

A categoria do chamado e o serviço afetado são dimensões diferentes. A primeira classifica a natureza da solicitação; o segundo identifica o serviço da empresa impactado.

### 2.4 Chamados e atendimento

- tickets: protocolo principal, título, descrição, categoria, serviço opcional, funcionário afetado, solicitante, prioridade atual, estado atual, datas principais e arquivamento.
- ticket_cycles: ciclo inicial e ciclos de reabertura (R1, R2...), prioridade do ciclo, estado, abertura e encerramento.
- ticket_assignments: períodos de atribuição a técnicos, com início/fim, autor e motivo.
- ticket_status_events: transições de estado, autor, timestamp e motivo.
- ticket_priority_events: prioridade anterior/nova, autor, timestamp e motivo.
- ticket_requests: solicitações administrativas tipadas (edição, cancelamento, reabertura), solicitante, motivo, estado da decisão, decisor, timestamps.
- ticket_edit_grants: autorização de edição específica por chamado, usuário e validade de 15 minutos.
- ticket_time_events: eventos temporais necessários para reconstruir abertura, primeira resposta humana, início de atendimento, espera, resolução e retomada.
- ticket_incident_links: relação entre chamados e incidentes, com autor, data e motivo.

O protocolo principal é estável. Os ciclos não criam novos chamados independentes. Histórico do ciclo anterior permanece preservado.

### 2.5 Mensagens e anexos

- ticket_messages: chamado/ciclo, autor, conteúdo, data/hora e tipo de mensagem (resposta pública ou anotação interna).

Mensagens internas/anotações da TI são permitidas e visíveis exclusivamente a administradores e técnicos autorizados. Supervisores não podem acessá-las por interface, API, pesquisa, notificações, exportações ou anexos associados. Respostas públicas seguem a autorização de acesso do chamado conforme HC-067.
- attachments: chave privada do objeto, nome original seguro, MIME detectado, tamanho, hash, estado de verificação, autor e timestamp.
- message_attachments: associação de anexos a mensagens.
- ticket_attachments: associação de anexos diretamente ao chamado, quando enviados na abertura.

Validação do arquivo é feita no servidor. O banco não armazena o binário. Download exige autorização atual e passa pelo back-end. Funcionário afetado não tem acesso.

### 2.6 Incidentes e comunicados

- incidents: protocolo próprio com prefixo INC (ex.: INC-0001), título, descrição, serviço relacionado, severidade, estado, início, normalização e encerramento.
- incident_ticket_links: relação N:N entre incidentes e chamados; um incidente pode agrupar vários chamados e um chamado pode ser relacionado a vários incidentes quando houver justificativa operacional. Cada vínculo registra autor, data/hora e motivo.
- announcements: tipo, conteúdo, estado, publicação, previsão opcional, autor e encerramento.
- announcement_acknowledgements: comunicado, supervisor, data/hora de ciência.

Resolver incidente não encerra automaticamente os chamados associados. Comunicado e incidente são entidades distintas.

### 2.7 Calendário, SLA e métricas

- business_calendars: calendário e timezone configurados.
- calendar_exceptions: feriados, dias excepcionais e expediente especial.
- sla_policies: metas por prioridade e versão da política.
- ticket_sla_instances: política aplicada ao chamado/ciclo, início do relógio, vencimento calculado, estado e conclusão.
- ticket_time_events: trilha de eventos para calcular tempos por ciclo e técnico.
- ticket_assignment_periods ou ticket_assignments: períodos por técnico, incluindo início/fim e transferências.

Metas aprovadas de primeira resposta: Baixa 2 dias úteis; Média 1 dia útil; Alta 4 horas úteis; Crítica imediata.

Regra HC-076: o relógio total não para durante espera do supervisor. A espera é medida separadamente. As fórmulas finais de tempo total, espera, atendimento e produtividade precisam ser especificadas e aprovadas antes da implementação. Não inferir que tempo de espera será subtraído do total.

A mudança para Em atendimento registra o início automaticamente (HC-078). Primeira resposta exige ação humana do técnico (HC-077).

### 2.8 Notificações e auditoria

- notifications: destinatário, canal, tipo de evento, referência, estado de entrega, tentativas, timestamps e chave de idempotência.
- notification_preferences (se aplicável): preferências de canal, sem desabilitar alertas obrigatórios.
- audit_events: ator, ação, entidade, identificador, timestamp, correlação, origem e payload de diferenças minimizado.
- scheduled_jobs ou fila persistida equivalente: tarefas agendadas com estado, tentativas, próxima execução, lock e chave idempotente.

Auditoria é append-only. A role da aplicação não recebe UPDATE/DELETE na tabela de auditoria. O banco deve proteger isso além da lógica de interface.

## 3. Relacionamentos principais

- Cada supervisor administra exatamente uma equipe na V1; uma equipe possui exatamente um supervisor responsável.
- Uma equipe possui muitos funcionários ao longo do tempo.
- Um funcionário possui histórico de vínculos; no máximo um vínculo atual.
- Um funcionário pode estar associado a muitos chamados.
- Um chamado possui um protocolo principal e um ou mais ciclos.
- Um ciclo possui eventos, mensagens, atribuições e métricas próprias.
- Um chamado pode ter vários técnicos em períodos distintos.
- Um chamado pode estar relacionado a vários incidentes, conforme decisão aprovada; cada associação é explícita e auditada.
- Um incidente pode agrupar vários chamados.
- Um comunicado possui confirmações de leitura de vários supervisores.
- Um serviço possui histórico de disponibilidade.
- Um chamado pode conter vários anexos; um anexo pertence ao chamado ou mensagem conforme origem.
- Um usuário pode receber muitas notificações e produzir muitos eventos de auditoria.

## 4. Invariantes que precisam ser protegidas

1. Funcionário ativo não pode ter dois vínculos atuais simultâneos.
2. Transferência de equipe deve ser atômica.
3. Chamado não pode ser excluído definitivamente pela interface.
4. Protocolo principal não muda em reabertura.
5. Cada reabertura cria ciclo próprio.
6. Ciclos encerrados não têm histórico sobrescrito.
7. Somente administradores aprovam edição, cancelamento e reabertura.
8. Concessão de edição expira em 15 minutos e vale para um chamado e usuário específicos.
9. Supervisor só acessa chamados conforme equipe atual do funcionário.
10. Administradores possuem visão global.
11. Anexos não são públicos e cada download revalida autorização.
12. Evento de auditoria não pode ser alterado ou apagado pela aplicação.
13. Job de fechamento automático é idempotente e só fecha após o prazo útil e sem contestação.
14. Resolver incidente não fecha chamados individuais.
15. Notificações duplicadas devem ser evitadas por chave idempotente.
16. Dados históricos necessários a métricas não podem depender apenas de campos atuais mutáveis.

## 5. Índices e desempenho — proposta inicial

Índices candidatos:
- tickets(protocol) único.
- tickets(employee_id, created_at).
- tickets(status, priority, updated_at).
- tickets(requester_user_id, created_at).
- ticket_cycles(ticket_id, cycle_number) único.
- employee_team_assignments(employee_id, valid_to).
- ticket_assignments(ticket_id, technician_user_id, started_at).
- ticket_messages(ticket_id, created_at).
- notifications(recipient_user_id, status, created_at).
- audit_events(entity_type, entity_id, occurred_at).
- service_status_history(service_id, started_at).

Índices finais serão definidos após consultas reais e plano de execução. Evitar índices redundantes.

## 6. Exclusão, retenção e arquivamento

- Chamados são arquivados, não apagados definitivamente pela interface.
- Arquivamento automático após 30 dias do fechamento.
- Chamados arquivados permanecem consultáveis por administradores e pelo supervisor atual do funcionário, respeitando HC-067.
- Anexos têm política de retenção total ainda pendente; não excluir automaticamente sem aprovação.
- Auditoria possui política própria de retenção e proteção, a definir antes da produção.

## 7. Decisões já confirmadas pelo Product Owner

1. Login dos supervisores por username e senha. E-mail é dado de contato/notificação, não identificador de login; será obrigatório para supervisores ativos.
2. Cada supervisor terá uma única equipe na V1.
3. Um chamado poderá ser relacionado a vários incidentes; cada associação deve ser explícita, justificada e auditada.
4. Todo funcionário ativo deve pertencer a exatamente uma equipe; cadastro/ativação sem equipe é bloqueado.
5. Mensagens internas/anotações da TI são permitidas e restritas a administradores e técnicos autorizados; supervisores não podem acessá-las por nenhum canal.
6. Incidentes terão protocolo próprio com prefixo INC (ex.: INC-0001).\n7. Cadastro de funcionário terá somente nome completo, equipe atual (derivada do vínculo vigente), situação ativo/inativo, data do cadastro e responsável pelo cadastro; não haverá identificador interno.

## 8. Pendências antes do esquema físico

1. O chamado guarda categoria e serviço afetado separadamente? Recomendação: sim.
2. Quais campos de funcionário são realmente necessários além do nome e estado?
3. Retenção de mensagens, anexos, notificações e auditoria.
4. Fórmulas exatas de tempo de atendimento, espera e produtividade.
5. Regras para usuários desativados e reativados, preservando histórico.

## 9. Próxima entrega

O diagrama ER e o dicionário de dados foram publicados em `docs/ER-DIAGRAM-AND-DATA-DICTIONARY-v0.1.md`. Após revisão desse documento:
1. Fechar as pendências remanescentes.
2. Produzir esquema físico final.
3. Definir estratégia de migrations e seed inicial.
4. Definir testes de integridade e autorização.

**Este documento não autoriza criação de tabelas, migrations ou implementação.**
