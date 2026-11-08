# HelpCore — Especificação Funcional v1.0

**Produto:** HelpCore — Futura 7  
**Projeto:** ITDESK  
**Versão:** 1.0  
**Status:** APROVADA pelo Product Owner  
**Escopo:** requisitos funcionais aprovados para a primeira versão. Esta aprovação não libera automaticamente a implementação; a especificação técnica e o plano de execução ainda precisam de aprovação.

## 1. Objetivo e contexto

Centralizar o suporte de TI da Futura 7, com abertura e acompanhamento de chamados por supervisores, operação da TI, gestão de incidentes e comunicados, disponibilidade de serviços, indicadores e auditoria.

A primeira versão será construída em repositório novo. O repositório antigo HelpCore é legado e não será mesclado sem decisão explícita.

## 2. Papéis, usuários e governança

- Administradores: Kléber e Felipe, equivalentes e com poderes administrativos completos.
- Técnico inicial: Kléber. Todo chamado novo é atribuído automaticamente a ele.
- A arquitetura deve permitir múltiplos técnicos no futuro.
- Supervisores autorizados a abrir chamados: Henrique, Carol, Elis, Jennifer, Lorraine, Bruno, Camila e Nicole.
- Cadastro restrito; não existe auto cadastro.
- Funcionários afetados não possuem login nem acesso ao sistema.
- Somente administradores gerenciam usuários, equipes e funcionários.
- Cada supervisor possui uma equipe cadastrada; seleção de funcionário ocorre em lista, sem nome livre. Supervisor pode selecionar a si próprio como afetado.

## 3. Funcionários e transferência de equipe

Funcionário tem identidade independente do vínculo com supervisor. Transferência preserva todo o histórico.

**Regra HC-067 — visibilidade pelo vínculo atual:** o histórico de chamados do funcionário acompanha seu supervisor/equipe atual, não o supervisor que abriu o chamado. Ex.: ao transferir Rafaela de Jennifer para Carol, Carol passa a visualizar todo o histórico; Jennifer perde acesso. Kléber e Felipe veem tudo. Chamados antigos preservam solicitante original e supervisor responsável à época. Chamados em andamento também acompanham o vínculo atual; notificações futuras vão para a supervisora atual.

A transferência deve ser transacional: não pode deixar funcionário sem equipe nem com vínculos duplicados em caso de falha.

## 4. Abertura e conteúdo de chamados

Campos:
- título;
- categoria;
- descrição;
- funcionário afetado;
- prioridade solicitada;
- anexos opcionais.

Categorias:
1. Hardware (PC, notebook, memória)
2. Software (Windows, programas, sistemas)
3. Rede e Internet (Wi-Fi, VPN, conexão)
4. Telefonia (MicroSIP, chamadas)
5. Acessos (senhas, contas, permissões)
6. Impressoras e periféricos
7. Solicitação de equipamento
8. Outros

Prioridades (HC-058):
- Baixa: fila normal.
- Média: fila normal.
- Alta: destaque na fila da TI.
- Crítica: atendimento imediato e alertas especiais.
- “Altíssima” foi descartada.

Supervisor escolhe prioridade inicial. Kléber pode revisar. Toda alteração registra valor anterior, novo valor, autor e timestamp.

Chamados podem ser abertos 24/7. Fora do expediente, o registro é aceito e o sistema informa início do atendimento no próximo expediente. Críticos continuam gerando e-mail imediato; não há obrigação de plantão.

## 5. Prioridade crítica e alertas

Chamado crítico gera:
- e-mail imediato para Kléber em kleber.silva@futura7.com.br;
- aviso genérico no HelpCore e e-mail a todos os supervisores ativos, sem dados privados e sem nome do solicitante.

Prioridade crítica não significa necessariamente incidente coletivo. Kléber decide se relaciona o chamado a incidente compartilhado. Se for individual, pode encerrar o aviso geral sem criar incidente. Alerta inicial crítico e incidente compartilhado são conceitos independentes.

## 6. Status e ciclo de vida

Estados: Aberto, Em atendimento, Aguardando supervisor, Resolvido, Fechado, Reaberto, Cancelado e Arquivado.

Fluxo de resolução:
1. Kléber marca como Resolvido.
2. Supervisor atual recebe notificação.
3. Tem 12 horas úteis para confirmar ou contestar.
4. Se confirmar, fecha.
5. Se não responder no prazo, fecha automaticamente.
6. Se contestar, retorna ao atendimento.

O histórico deve distinguir fechamento confirmado de fechamento automático.

Calendário de 12 horas úteis: segunda a sexta, 09:00–16:50, fuso Brasília UTC−3; fins de semana pausam o contador; feriados e dias excepcionais configuráveis pelos administradores. Exemplo: resolução sexta às 16:40; o contador retoma segunda às 09:00, descontando apenas o tempo útil decorrido antes da pausa.

## 7. SLA de primeira resposta (HC-059)

Metas:
- Baixa: até 2 dias úteis.
- Média: até 1 dia útil.
- Alta: até 4 horas úteis.
- Crítica: imediatamente.

São metas de primeira resposta, não de resolução. Expediente: segunda a sexta, 09:00–16:50. Se aberto fora do expediente, o prazo começa no próximo expediente.

Primeira resposta só conta ação humana do técnico, como mensagem no chat ou atualização formal (HC-077). Confirmação automática de abertura e notificações automáticas não contam.

## 8. Chat por chamado

Chat entre supervisor solicitante e TI:
- histórico permanente;
- mensagens com autor, data e hora;
- notificação interna e e-mail para nova mensagem;
- acesso de supervisores respeita a regra de equipe atual do funcionário;
- administradores veem tudo.

Mensagens não substituem o histórico formal de status.

## 9. Acompanhamento do supervisor

Tela “Meus Chamados” permite consultar status, prioridade, datas, histórico, solução, confirmar/contestar resolução e conversar com TI.

Tela inicial simples e direta:
- botão “Abrir chamado”;
- botão “Acompanhar chamados”;
- resumo pequeno dos chamados em andamento/aguardando.

## 10. Solicitação de edição, cancelamento e reabertura

**HC-073 — edição:** supervisor solicita edição e informa motivo. Kléber ou Felipe aprova/recusa. Se aprovada, autorização específica para aquele chamado dura 15 minutos; depois o supervisor pode editar. Valores antigos/novos são auditados. Não se aplica a chamados fechados/arquivados. Não criar permissão genérica ou persistente.

**HC-074 — cancelamento:** supervisor solicita cancelamento com motivo. Administrador aprova/recusa. Se aprovado, status Cancelado; não é resolução, não conta produtividade como resolvido, histórico preservado e pesquisável, operação auditada.

**HC-070/071 — reabertura:** supervisor solicita reabertura de chamado fechado/resolvido com motivo. Kléber/Felipe aprova ou recusa. Se aprovada, cria novo ciclo de atendimento, prioridade inicial Alta, ajustável pelo administrador. Protocolo principal permanece, ciclos recebem sufixos HC-0012-R1, R2 etc. Histórico original preservado e indicadores de recorrência disponíveis. Pode solicitar reabertura mesmo após dois dias ou arquivamento, mediante aprovação.

## 11. Fila da TI (HC-075)

Quatro áreas:
- Novos;
- Em atendimento;
- Aguardando supervisor;
- Reaberturas.

Críticos destacados no topo independentemente da área. Cancelados e arquivados não poluem a fila operacional. Chamados novos são atribuídos automaticamente a Kléber.

## 12. Chamados, incidentes e comunicados

Conceitos separados:
- Chamado: solicitação individual.
- Incidente: problema geral que afeta vários funcionários/equipes.
- Comunicado: informação geral de TI.

Um incidente pode agrupar múltiplos chamados, preservando protocolos e históricos individuais. Ex.: telefonia indisponível, com chamados de diferentes equipes relacionados ao mesmo incidente. Resolver incidente não fecha automaticamente chamados individuais; cada chamado pode exigir conclusão própria.

Ao normalizar incidente: remover aviso de indisponibilidade, informar supervisores, manter histórico e preservar chamados individuais ainda pendentes.

**HC-062:** ao selecionar serviço indisponível, HelpCore sugere incidentes ativos relacionados. Supervisor pode vincular ou abrir chamado separado; não há associação automática sem confirmação/análise da TI.

Administradores podem criar, editar, publicar e encerrar comunicados. Supervisores recebem e consultam. Comunicados gerais são enviados dentro do HelpCore e por e-mail a todos os supervisores ativos. Tipos: indisponibilidade, instabilidade, manutenção programada e normalização. Podem ser publicados antecipadamente.

Status: Identificado, Em análise, Em tratamento, Normalizado, Encerrado. Previsão de retorno é opcional e só deve ser exibida quando confiável.

**HC-049 — ciência:** botão “Estou ciente” registra supervisor e data/hora. Entrega de e-mail não equivale a confirmação de leitura. Administradores consultam pendências.

## 13. Catálogo e disponibilidade

Catálogo editável por administradores. Itens iniciais:
- Internet e rede;
- Telefonia/MicroSIP;
- Bitrix24;
- Sistemas internos;
- Computadores/notebooks;
- Impressoras/periféricos;
- E-mail corporativo;
- Outros.

Painel de disponibilidade: Operacional, Instável ou Indisponível; visível no dashboard e NOC, com atualização em tempo real e histórico. V1: atualização manual por administradores. Monitoramento automático fica para fase 2.

Histórico deve permitir analisar número de incidentes por serviço, frequência, tempo médio de normalização, histórico mensal, serviços recorrentes e incidentes críticos anteriores.

## 14. Dashboard e NOC

Dashboard de Kléber e Felipe idêntico:
- volume de chamados;
- resolvidos e pendentes;
- tempo médio de resolução;
- categorias;
- funcionários com mais chamados;
- prioridades;
- incidentes por serviço;
- tempo de normalização;
- comparações;
- filtros dia, semana, mês e período personalizado.

Painel NOC para tablet, tela cheia e sem menus administrativos:
- incidentes ativos;
- serviços indisponíveis/instáveis;
- comunicados;
- chamados críticos;
- indicadores;
- relógio;
- última atualização.
Atualiza em tempo real e reconecta após perda de internet.

Painel supervisor deve permanecer simples, rápido e direto.

## 15. Pesquisa, exportação e relatórios

**HC-040:** pesquisa avançada exclusiva de administradores, por número, funcionário, supervisor, categoria, período, status e prioridade. Filtros combináveis, inclui arquivados, ordenação por data/prioridade/atualização e exportação CSV/Excel.

Pesquisa de satisfação/estrelas foi descartada (HC-072).

## 16. Métricas de tempo e produtividade

**HC-076:** registrar abertura, primeira resposta, início de atendimento, resolução, tempo aguardando supervisor e tempo efetivo/em atendimento. Regra aprovada: o relógio não para, inclusive aguardando supervisor. Períodos de espera são separados, sem descontar do tempo total nem do tempo acumulado de atendimento. Tempos separados por ciclo.

**HC-078:** mudar para “Em atendimento” registra automaticamente início do atendimento, mesmo sem mensagem do técnico.

**HC-079:** registrar tempo individual por técnico, incluindo atribuição, início/fim e transferências; preservar períodos anteriores.

**HC-080:** painel de produtividade por técnico com chamados resolvidos, primeira resposta média, tempo médio de atendimento, tempo médio de resolução, em andamento, reabertos e volume por período. Não comparar técnicos apenas por quantidade resolvida; indicadores separados.

**HC-081:** filtros combináveis por técnico, período (diário, semanal, mensal, personalizado) e categoria; comparar técnicos e evolução individual; preservar histórico se técnico sair.

As fórmulas exatas de cada métrica devem ser formalizadas na especificação técnica sem alterar a regra aprovada de relógio contínuo e espera separada.

## 17. Anexos

Formatos aceitos: JPG/JPEG/PNG, PDF, DOC/DOCX, XLS/XLSX, TXT, MP4/MOV.
- Até 5 arquivos por envio, na abertura e nas mensagens.
- Imagens: até 10 MB cada.
- Documentos: até 20 MB cada.
- Vídeos: até 100 MB cada.

Validar tipo real (magic bytes), tamanho e armazenar privadamente. Download controlado pelo back-end, com URLs assinadas curtas quando adequado. Registrar autor e timestamp. Funcionário afetado não tem acesso. Supervisor atual do funcionário e administradores podem visualizar/baixar; administradores têm acesso global.

## 18. Autenticação e segurança

- Login individual para supervisores, Kléber e Felipe.
- Sem auto cadastro; contas e senhas criadas pelos administradores.
- Senha provisória e troca obrigatória no primeiro acesso.
- Somente Kléber/Felipe redefinem senhas.
- Supervisor solicita ajuda à TI; admin gera senha provisória e entrega por canal seguro.
- Sistema nunca exibe nem armazena senha em texto puro.
- Sessão expira após 3 horas de inatividade (HC-063); atividade válida renova; logout encerra; bloqueio revoga sessões; administradores podem encerrar remotamente; auditoria de login/logout/expiração.
- 5 tentativas incorretas consecutivas bloqueiam temporariamente por 15 minutos (HC-065). Tentativas auditadas. Proteção adicional por IP com limites próprios para evitar denial-of-service induzido.
- HTTPS obrigatório; controles de acesso no servidor; proteção contra acesso indevido; sessões seguras; backup.

Sistema acessível pela internet, não restrito à rede local. Não depende de computador da TI ligado. Tablet e desktops podem acessar de qualquer local.

## 19. Auditoria e retenção

**HC-066:** auditoria imutável pela interface; nenhum administrador pode editar/apagar eventos pelo HelpCore. Banco deve impor proteção: usuário da aplicação apenas insere/consulta. Reconhece-se que superusuário de banco/infra pode contornar; devem existir controles administrativos e backups.

Auditar login, abertura/edição/encerramento, prioridade, status, mensagens, uploads, cadastros/transferências, usuários, comunicados, serviços, calendário, consulta/exportação sensível.

Chamados não podem ser excluídos definitivamente pela interface. Arquivamento permitido, preservando dados, histórico e anexos. Arquivar automaticamente após 30 dias do fechamento; administradores podem arquivar manualmente. Reabertura após fechamento/arquivamento exige solicitação e aprovação. Arquivados continuam visíveis ao supervisor atual do funcionário e administradores.

## 20. Backup e operação

- Backup diário criptografado.
- Retenção dos últimos 30 dias.
- Armazenamento separado do servidor principal.
- Incluir banco e arquivos.
- Monitorar falhas e alertar.
- Testar restauração.
- Atualizações remotas, controle de versão, ambiente de testes, backup antes de mudanças críticas e rollback.

## 21. Tecnologias e experiência visual

Stack aprovada:
- React + TypeScript;
- Node.js + Express;
- PostgreSQL;
- API REST;
- Tailwind CSS;
- WebSockets;
- object storage privado para anexos.

Arquitetura: monólito modular, com separação interna por módulos.

Tema neutro azul escuro/preto, azul para ações, tema escuro padrão. Responsivo; desktop prioritário, tablet/celular desde a primeira versão.

## 22. Integrações futuras

Integrações futuras previstas com DASH IT, Automação em Calls e inventário de computadores. Não bloquear HelpCore pela conclusão do inventário. Contratos e detalhes de integração serão especificados em etapa futura.

## 23. Pendências técnicas que não alteram a aprovação funcional

Hospedagem/provedor, domínio, SMTP, limites totais de armazenamento, RPO/RTO, retenção de anexos, expiração de tokens e detalhes de sessão ainda precisam de decisão técnica. Também precisam ser detalhados modelo lógico de dados, contratos de API, wireframes, CI/CD, observabilidade, estratégia de deploy e fórmulas formais de métricas.

## 24. Critério de liberação

A especificação funcional v1.0 está aprovada. Antes de iniciar desenvolvimento, deve haver especificação técnica detalhada e plano incremental de implementação aprovados explicitamente pelo Product Owner.
