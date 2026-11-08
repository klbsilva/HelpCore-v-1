# HelpCore — Registro de decisões

Este registro consolida decisões aprovadas para evitar regressões e interpretações divergentes. A especificação funcional base está em [SPECIFICATION-v1.0.md](SPECIFICATION-v1.0.md). Decisões posteriores registradas aqui prevalecem quando explicitamente identificadas como atualização.

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
| HC-076 (regra atualizada) | Tempo total de resolução e SLAs avançam somente nos intervalos úteis configurados; espera do supervisor continua contando dentro desses intervalos e é mensurada separadamente. Esta decisão posterior substitui a interpretação anterior de relógio corrido. |
| HC-077 | Primeira resposta exige ação humana do técnico; automações não contam. |
| HC-078 | Entrada em “Em atendimento” registra início automaticamente. |
| HC-079 | Períodos individuais por técnico são preservados, inclusive transferências. |
| HC-080 | Produtividade usa múltiplos indicadores, não apenas volume resolvido. |
| HC-081 | Filtros por técnico, período e categoria; histórico preservado. |
| Resolução | Supervisor tem 12 horas úteis para confirmar/contestar; ausência de resposta fecha automaticamente. |
| Expediente contabilizável | Segunda a sexta: 09:00–12:00, 13:30–15:00 e 15:20–16:50 (America/Sao_Paulo). Total de 6h00/360 minutos por dia. Sábados, domingos e feriados/dias fechados não contam; horários especiais são configuráveis. A pausa corrigida é 15:00–15:20 (20 minutos). |
| Abertura de chamados | Supervisores podem abrir chamados 24 horas por dia; fora dos intervalos úteis, o relógio de SLA/resolução fica pausado e retoma no próximo intervalo. |
| Crítico | E-mail imediato para Kléber; aviso genérico a supervisores, sem dados privados. |
| Arquivamento | Automático após 30 dias do fechamento; sem exclusão definitiva pela interface. |
| Anexos | Até 5 por envio; imagens 10 MB, documentos 20 MB, vídeos 100 MB; tipos permitidos conforme especificação. |
| Stack | React/TypeScript, Node/Express, PostgreSQL, REST, Tailwind, WebSockets e object storage privado. |
| Arquitetura | Monólito modular. |
| Repositório | HelpCore-v-1 é a nova linha; repositório HelpCore antigo permanece legado. |
| Liberação | Não implementar até aprovação da especificação técnica e do plano de execução. |

## Observação de governança
Se uma implementação futura parecer contradizer qualquer decisão acima, interromper a etapa, apresentar a divergência e solicitar decisão do Product Owner. Não alterar silenciosamente regras aprovadas.
