# HelpCore

Sistema interno de gestão de serviços de TI e suporte da Futura 7.

> **Status:** documentação e especificação. A implementação ainda não foi autorizada.

## Objetivo
Centralizar chamados de TI, comunicação de incidentes, disponibilidade de serviços, acompanhamento por supervisores, gestão operacional e indicadores.

## Documentos oficiais
- [Especificação funcional v1.0](docs/SPECIFICATION-v1.0.md)
- [Diretrizes de engenharia](ENGINEERING.md)
- [Registro de decisões](docs/DECISIONS.md)

## Stack aprovada
- Front-end: React + TypeScript
- Back-end: Node.js + Express
- Banco de dados: PostgreSQL
- API: REST
- Estilos: Tailwind CSS
- Tempo real: WebSockets
- Anexos: object storage privado

## Governança
- Product Owner e validador: Kléber.
- Arquitetura, especificação e revisão técnica: ChatGPT (Tom).
- Implementação: Claude, somente após aprovação explícita dos documentos técnicos e do plano de execução.
- Administradores do sistema: Kléber e Felipe, com poderes equivalentes.

## Regra de trabalho
Não iniciar implementação, gerar telas ou alterar escopo com base em suposições. Primeiro, especificar; depois, obter aprovação do Product Owner; então implementar em etapas pequenas, testar e revisar.

## Repositório
Este repositório é a nova linha de desenvolvimento. O repositório HelpCore anterior permanece legado e não deve ser copiado ou mesclado sem decisão explícita.
