# HelpCore — Arquitetura de implantação com custo zero

**Versão:** 0.2 — proposta  
**Status:** proposta para validação; não libera implementação nem produção  
**Decisões do Product Owner:** acesso pela internet; orçamento mensal de infraestrutura = R$ 0  
**Repositório:** HelpCore-v-1

## 1. Objetivo e limite da decisão

A arquitetura deve permitir acesso pela internet sem depender de computador ligado na Futura 7 e sem contratação de hospedagem mensal.

“Custo zero” significa usar exclusivamente cotas gratuitas, sem assumir que elas oferecem disponibilidade, suporte, backup ou desempenho de produção. O sistema deverá monitorar consumo e falhar de forma segura ao se aproximar dos limites, sem gerar cobrança automática. Não adicionar cartão ou habilitar cobrança por uso sem aprovação explícita do Product Owner.

Esta é uma arquitetura para protótipo/piloto controlado. Não se deve prometer SLA de disponibilidade ou recuperação enquanto depender de planos gratuitos.

## 2. Topologia proposta

| Camada | Serviço candidato | Função |
|---|---|---|
| Front-end | Cloudflare Workers Static Assets (ou Pages, após validação) | Servir a aplicação React compilada |
| API | Cloudflare Workers | Executar API HTTP e regras de negócio |
| Runtime | Node.js compatibility do Workers | Avaliar execução do back-end Node/Express |
| Banco | Neon Free PostgreSQL | Dados transacionais |
| Anexos | Cloudflare R2 Standard | Object storage privado |
| Tempo real | Cloudflare Durable Objects + WebSocket | Coordenar conexões e difundir eventos |
| E-mail | Resend Free | E-mails transacionais, sujeito a domínio remetente verificado |
| Código/CI | GitHub | Repositório, revisão e automações de build/teste |

Todos os serviços acima são candidatos e precisam de contas, configurações e testes. Não há contratação ou conexão realizada por esta decisão.

## 3. Limites gratuitos conhecidos — fotografia de 08/11/2026

Os limites mudam. Conferir os painéis e páginas oficiais antes de criar os recursos.

### Cloudflare Workers
A documentação informa, no plano gratuito, até 100.000 requisições por dia, 10 ms de CPU por invocação e 128 MB de memória. O runtime oferece compatibilidade parcial com APIs Node.js; não é um servidor Node tradicional.

Referências:
- https://developers.cloudflare.com/workers/platform/limits/
- https://developers.cloudflare.com/workers/runtime-apis/nodejs/

### Cloudflare Durable Objects
Disponível no plano gratuito para armazenamento SQLite, com cotas gratuitas diárias e 5 GB de armazenamento SQLite por conta. A documentação informa 100.000 requisições/dia e 13.000 GB-s/dia para compute no plano gratuito. O uso de WebSocket Hibernation deve ser avaliado para reduzir consumo ocioso.

Referências:
- https://developers.cloudflare.com/durable-objects/platform/limits/
- https://developers.cloudflare.com/durable-objects/platform/pricing/

### Neon Free PostgreSQL
A documentação consultada informa 1 GB de armazenamento por projeto, 100 CU-hours de compute por projeto/mês e 5 GB de transferência de rede pública por projeto/mês. O compute escala para zero após inatividade. Ao atingir cotas, o compute pode ficar suspenso até a próxima janela mensal ou upgrade.

Referência:
- https://neon.com/blog/neon-free-plan-1-gb-per-project

### Cloudflare R2 Standard
A cota gratuita publicada inclui 10 GB-month de armazenamento, 1 milhão de operações Class A e 10 milhões de operações Class B por mês; egress para internet é gratuito. O uso acima das cotas pode gerar cobrança, portanto alertas e limites internos são obrigatórios.

Referência:
- https://developers.cloudflare.com/r2/pricing/

### Resend Free
A página de preços consultada informa 3.000 e-mails por mês e limite de 100 por dia no plano gratuito. O envio depende de configuração e validação do domínio remetente. A equipe precisa ter autorização para configurar DNS do domínio corporativo; sem isso, e-mail transacional de produção não pode ser presumido.

Referência:
- https://resend.com/pricing

## 4. Compatibilidade do back-end aprovado

A stack funcional aprovada inclui Node.js + Express. Cloudflare Workers oferece compatibilidade parcial com APIs Node.js e suporte a APIs HTTP Node em datas de compatibilidade recentes, mas isso não prova que todas as dependências do Express e do projeto funcionarão corretamente.

Antes de escolher o runtime:
1. Criar uma prova de conceito isolada, sem funcionalidades do produto.
2. Executar uma rota Express simples no runtime Workers usando a compatibilidade Node suportada.
3. Testar cookies/sessões, crypto, streams, uploads, bibliotecas PostgreSQL, transações, WebSocket e middleware.
4. Testar localmente e em ambiente remoto Cloudflare.
5. Registrar bibliotecas incompatíveis e limitações.

Se Express não passar, não substituir silenciosamente por Hono ou outro framework. Apresentar a mudança ao Product Owner para aprovação. Alternativamente, avaliar outro runtime gratuito que execute Node tradicional, aceitando suas limitações de disponibilidade e suspensão.

## 5. Banco PostgreSQL

Neon Free é o candidato inicial porque preserva PostgreSQL, alinhado à stack aprovada.

Requisitos:
- Acesso por driver compatível com ambiente serverless.
- Transações reais para transferências de funcionários, autorização de edição, mudanças de status e operações críticas.
- Pool/conexões configurados para ambiente serverless.
- Timeouts e retries controlados; não repetir automaticamente operações não idempotentes.
- Índices e paginação para evitar consumo excessivo.
- Métricas de armazenamento, CU-hours e transferência.
- Bloquear crescimento de dados quando houver risco de exceder cotas, sem descartar histórico silenciosamente.

O driver definitivo só será escolhido após prova de conceito de transações, conexões concorrentes, migrações e testes de falha.

## 6. Anexos e limites internos

R2 Standard é o candidato para anexos privados. A cota publicada de 10 GB-month não comporta crescimento ilimitado, especialmente vídeos.

Proposta inicial de política operacional, sujeita à aprovação:
- Reservar no máximo 7 GB para anexos do HelpCore.
- Reservar o restante para margem operacional, versões e metadados.
- Exibir espaço utilizado e disponível aos administradores.
- Alertar em 70%, 85% e 95%.
- Bloquear novos uploads antes de exceder o limite, sem apagar anexos existentes.
- Não aceitar vídeo em massa; manter os limites individuais da especificação funcional, mas aplicar quota total do sistema.
- URLs de download nunca públicas; autorização no servidor e URLs assinadas de curta duração.

A política de retenção dos anexos continua pendente. Não excluir anexos automaticamente sem decisão do Product Owner.

## 7. WebSockets e notificações em tempo real

Durable Objects é o candidato para conexões WebSocket e distribuição de eventos em tempo real, evitando manter um servidor permanentemente ligado.

Requisitos:
- Usar WebSocket Hibernation quando compatível.
- Não guardar dados oficiais de chamados somente no Durable Object; PostgreSQL permanece fonte de verdade.
- Ao reconectar, o cliente consulta a API e recupera estado.
- Eventos devem carregar identificadores, não conteúdo privado desnecessário.
- Autenticar a conexão e validar autorização para cada canal/escopo.
- Definir estratégia para conexões encerradas e mensagens perdidas.

As cotas gratuitas de Durable Objects são finitas. Implementar monitoramento e limites de conexão/mensagens; ao exceder cotas, a atualização em tempo real pode falhar, mas a API e o banco devem preservar a integridade.

## 8. E-mail transacional

Resend Free é candidato inicial, sujeito a:
- domínio remetente verificado;
- acesso autorizado ao DNS;
- teste de SPF, DKIM e DMARC;
- limites de 3.000/mês e 100/dia conforme plano consultado;
- retry com backoff, idempotência e fila persistida;
- registro de entrega/falha sem registrar conteúdo sensível indevido.

**Risco de produto:** o alerta de chamado crítico por e-mail foi requisito aprovado. Um plano gratuito sem SLA não garante entrega imediata. Se a empresa exigir garantia operacional para alertas críticos, custo zero pode ser incompatível com esse requisito. A UI deve informar falha de envio e manter o alerta dentro do sistema.

Não usar SMTP direto a partir de Workers; a arquitetura deve usar API HTTPS do provedor.

## 9. Domínio e HTTPS

Para custo zero inicial, usar subdomínio gratuito fornecido pela plataforma (por exemplo, workers.dev). Não comprar domínio nem alterar DNS corporativo sem autorização.

Se a Futura 7 disponibilizar um domínio já registrado e autorizar as alterações DNS, poderá ser conectado sem comprar outro domínio, sujeito às condições do provedor.

HTTPS obrigatório.

## 10. Backup e recuperação — bloqueador de produção

O plano gratuito do banco não deve ser tratado como estratégia completa de backup de longo prazo. Antes de colocar dados operacionais reais em produção, implementar e testar:
- exportação diária do PostgreSQL;
- criptografia antes do envio ao destino;
- cópia em armazenamento separado do banco;
- retenção de 30 dias conforme requisito funcional;
- backup de metadados e anexos;
- alertas de falha;
- teste de restauração documentado;
- RPO/RTO aprovados.

GitHub Actions agendado pode ser avaliado como executor de exportação, desde que as cotas gratuitas, confiabilidade de agendamento, segurança dos segredos e upload ao R2 sejam validados. Não considerar backup implantado até concluir um teste de restauração ponta a ponta.

**Não liberar produção com dados reais enquanto o backup diário e a restauração não estiverem comprovados.**

## 11. Segurança operacional e controle de custos

- Não cadastrar cartão ou habilitar faturamento por uso sem autorização expressa.
- Configurar limites internos e alertas de consumo.
- Monitorar Workers requests/CPU, Durable Objects, Neon CU-hours/storage/transfer, R2 storage/operations e e-mails enviados.
- Definir comportamento de degradação: preservar dados, bloquear operações perigosas e informar indisponibilidade.
- Nunca sacrificar autorização, auditoria ou integridade para economizar cota.
- Segredos em configurações secretas do provedor, nunca no GitHub.
- Separar ambiente de testes e produção; verificar se a cota gratuita suporta os dois. Se não, usar branches/fixtures cuidadosamente ou ambiente local para testes.
- Não usar serviços de “ping” para impedir suspensão/scale-to-zero; isso viola a premissa técnica e pode consumir cotas sem benefício.

## 12. Riscos aceitos somente para piloto

1. Cold start ou escala para zero no banco.
2. Limites diários/mensais podem suspender partes do serviço.
3. Ausência de SLA de disponibilidade e suporte nos planos gratuitos.
4. Limite de armazenamento e transferência.
5. Limites de envio e entregabilidade de e-mail.
6. Compatibilidade parcial de Node/Express no runtime serverless.
7. Backup/restore depende de automação externa validada.
8. Recursos gratuitos e preços podem mudar sem controle da Futura 7.

O sistema deve ser apresentado como piloto enquanto esses riscos permanecerem.

## 13. Estratégia de migração futura

Manter o código independente de fornecedor:
- camada de armazenamento de arquivos por interface;
- camada de e-mail por interface;
- camada de publicação de eventos por interface;
- configuração por variáveis de ambiente;
- PostgreSQL padrão, sem dependência de extensões proprietárias desnecessárias;
- migrações versionadas;
- exportação de dados e anexos documentada.

Assim, uma futura migração para VPS, servidor Node tradicional ou serviços pagos não exigirá reescrever as regras de negócio.

## 14. Próximos testes técnicos

Antes de fechar a arquitetura:
1. Prova de conceito Express no Cloudflare Workers.
2. Prova de conexão e transação PostgreSQL com Neon.
3. Prova de upload privado e download autorizado no R2.
4. Prova de WebSocket autenticado com Durable Objects e hibernação.
5. Prova de e-mail Resend com domínio remetente autorizado.
6. Prova de backup diário e restauração.
7. Teste de limites e comportamento quando as cotas se esgotam.

## 15. Gate de aprovação

Este documento é uma proposta técnica de implantação gratuita. Não autoriza implementação de funcionalidades nem produção.

O Product Owner deve aprovar:
- arquitetura serverless proposta;
- provedores candidatos;
- limites internos de armazenamento;
- riscos de disponibilidade;
- estratégia de e-mail;
- estratégia de backup;
- resultado das provas de conceito.

A regra de orçamento permanece: nenhum serviço pago, upgrade, domínio novo ou cobrança por uso sem autorização explícita de Kléber.
