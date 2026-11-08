# HelpCore — Registro de decisões

Este registro consolida decisões aprovadas para evitar regressões e interpretações divergentes. A especificação funcional completa está em [SPECIFICATION-v1.0.md](SPECIFICATION-v1.0.md).

| ID / tema | Decisão aprovada |
|---|---|
| Governança | Kléber e Felipe são administradores equivalentes. |
| Técnico inicial | Kléber recebe automaticamente os novos chamados. |
| Supervisores | Apenas Henrique, Carol, Elis, Jennifer, Lorraine, Bruno, Camila e Nicole abrem chamados. |
| Funcionários | Sem login; selecionados de cadastro prévio. |
| HC-058 | Prioridades Baixa, Média, Alta e Crítica; “Altíssima” descartada. |
| HC-059 | SLA de primeira resposta: Baixa 2 dias úteis; Média 1 dia útil; Alta 4 horas úteis; Crítica imediata. |
| HC-062 | Sugerir incidentes ativos relacionados; vínculo depende de confirmação/análise. |
| HC-063 | Sessão expira após 3 horas de inatividade. |
| HC-065 | 5 tentativas incorretas consecutivas geram bloqueio temporário de 15 minutos. |
| HC-066 | Auditoria append-only; app não edita/apaga eventos. |
| HC-067 | Visibilidade do histórico acompanha o supervisor/equipe atual do funcionário. |
| HC-070/071 | Reabertura mediante solicitação e aprovação; cria novo ciclo e preserva protocolo principal. |
| HC-072 | Pesquisa de satisfação/estrelas descartada. |
| HC-073 | Edição por supervisor somente após aprovação; autorização específica de 15 minutos. |
| HC-074 | Cancelamento solicitado pelo supervisor e aprovado por administrador; não conta como resolvido. |
| HC-075 | Fila TI: Novos, Em atendimento, Aguardando supervisor e Reaberturas; críticos destacados. |
| HC-076 | Relógio não para enquanto aguarda supervisor; espera é mensurada separadamente. |
| HC-077 | Primeira resposta exige ação humana do técnico; automações não contam. |
| HC-078 | Entrada em “Em atendimento” registra início automaticamente. |
| HC-079 | Períodos individuais por técnico são preservados, inclusive transferências. |
| HC-080 | Produtividade usa múltiplos indicadores, não apenas volume resolvido. |
| HC-081 | Filtros por técnico, período e categoria; histórico preservado. |
| Resolução | Supervisor tem 12 horas úteis para confirmar/contestar; ausência de resposta fecha automaticamente. |
| Expediente | Segunda a sexta, 09:00–16:50, horário de Brasília; feriados/exceções configuráveis. |
| Crítico | E-mail imediato para Kléber; aviso genérico a supervisores, sem dados privados. |
| Arquivamento | Automático após 30 dias do fechamento; sem exclusão definitiva pela interface. |
| Anexos | Até 5 por envio; imagens 10 MB, documentos 20 MB, vídeos 100 MB; tipos permitidos conforme especificação. |
| Stack | React/TypeScript, Node/Express, PostgreSQL, REST, Tailwind, WebSockets e object storage privado. |
| Arquitetura | Monólito modular. |
| Repositório | HelpCore-v-1 é a nova linha; repositório HelpCore antigo permanece legado. |
| Liberação | Não implementar até aprovação da especificação técnica e do plano de execução. |

## Observação de governança
Se uma implementação futura parecer contradizer qualquer decisão acima, interromper a etapa, apresentar a divergência e solicitar decisão do Product Owner. Não alterar silenciosamente regras aprovadas.
