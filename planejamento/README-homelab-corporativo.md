# Projeto futuro — HomeLab Corporativo: Redes, Servidores e Segurança

> **Status:** Planejamento — nenhuma das novas integrações descritas aqui foi implementada ou validada.
>
> **Prioridade atual:** concluir a etapa em andamento de Linux Essentials 010-160. Retomar este projeto posteriormente.

## 1. Visão do projeto

Evoluir o HomeLab baseado em **Dell + Proxmox VE + EVE-NG** de topologias de roteadores e switches para uma **infraestrutura corporativa funcional**, na qual estações Windows/Linux e equipamentos Cisco acessem serviços reais de rede e passem por controles de segurança e autenticação.

O objetivo é estudar e documentar, na prática, a integração entre **redes, administração de sistemas Linux, serviços corporativos, identidade, firewall e monitoramento**, mantendo o foco defensivo (Blue Team).

## 2. Infraestrutura já conhecida

- Servidor físico Dell com **Proxmox VE**.
- Máquina virtual **EVE-NG**, utilizada em laboratórios de redes.
- Laboratório Cisco IOL L2/L3 com roteador, switch e hosts virtuais.
- Experiência prévia com Linux, virtualização, SSH, Git e GitHub.
- Equipamento físico **Cisco Catalyst 3560G** disponível, mas não integrado ao Lab 01.

**Importante:** a existência desses componentes não significa que os serviços propostos abaixo já estejam instalados.

## 3. Arquitetura conceitual desejada

```text
Servidor Dell (hardware físico)
└── Proxmox VE
    ├── EVE-NG — laboratório de redes
    │   ├── Roteadores e switches Cisco
    │   ├── VLANs e roteamento inter-VLAN
    │   ├── Firewall virtual (a definir)
    │   └── Clientes virtuais (Windows/Linux/VPCS)
    │
    └── VMs de serviços (localização e integração a definir)
        ├── DNS e DHCP
        ├── Servidor web / aplicações
        ├── Identidade e diretório (AD ou alternativa)
        ├── Autenticação AAA / RADIUS / TACACS+
        └── Logs e monitoramento
```

*Este diagrama é uma proposta, não uma topologia implementada. A posição dos servidores (dentro do EVE-NG ou em VMs do Proxmox) será decidida após avaliar recursos e conectividade.*

## 4. Serviços e tecnologias a estudar

| Área | Possibilidades | O que queremos compreender |
| --- | --- | --- |
| Redes Cisco | VLANs, trunks, roteamento, ACLs | Comunicação e segmentação entre redes |
| Firewall | OPNsense e/ou FortiGate-VM | Políticas entre VLANs, NAT, inspeção e logs |
| DNS | Servidor Linux | Resolução de nomes para os clientes |
| DHCP | Servidor Linux ou serviço dedicado | Distribuição de IP, gateway e DNS |
| Web | Nginx/Apache | Hospedar serviços acessados pelos clientes |
| Diretório | Active Directory ou alternativa compatível | Centralização de usuários, grupos e computadores |
| AAA | RADIUS, TACACS+ e, futuramente, Cisco ISE | Autenticação, autorização e contabilização |
| Observabilidade | Syslog, Zabbix, Grafana; SIEM em etapa futura | Coleta, correlação e investigação de eventos |

### Cisco ISE e Active Directory não são a mesma coisa

- **Cisco ISE (Identity Services Engine):** plataforma de identidade e controle de acesso, com funções AAA e políticas de acesso à rede.
- **Active Directory:** serviço de diretório para identidades, grupos, computadores e políticas em ambientes Windows.
- **RADIUS e TACACS+:** protocolos que podem participar da autenticação e administração de acessos.

A implantação do Cisco ISE dependerá de viabilidade técnica, recursos e licenciamento. É possível começar os estudos de AAA com alternativas adequadas ao laboratório, sem depender inicialmente do ISE.

## 5. Cenários práticos desejados

1. **Cliente recebe conectividade:** uma estação obtém IP via DHCP, resolve nomes via DNS e acessa uma página web interna.
2. **Segmentação de rede:** clientes de VLANs diferentes só alcançam os serviços permitidos pelas regras de firewall.
3. **Identidade centralizada:** usuários e grupos são cadastrados em um serviço de diretório, com autenticação em sistemas selecionados.
4. **Administração de dispositivos:** equipamentos Cisco utilizam AAA para validar acessos administrativos, com registros de eventos.
5. **Monitoramento e investigação:** logs de servidores, equipamentos e firewall são enviados a uma plataforma central para análise.

Cada cenário deverá ser implementado e validado separadamente. Não registrar resultados como concluídos antes de executar testes.

## 6. Etapas futuras sugeridas (não iniciadas)

- **Fase 0 — Inventário:** verificar CPU, RAM, armazenamento, redes virtuais, bridges, versões e capacidade disponível no Dell/Proxmox.
- **Fase 1 — Rede-base:** validar conectividade L2/L3, endereçamento IPv4, VLANs e roteamento no EVE-NG.
- **Fase 2 — Serviços fundamentais:** DNS, DHCP e servidor web; testar a partir de clientes.
- **Fase 3 — Firewall:** implementar segmentação e políticas, com testes de tráfego permitido e bloqueado.
- **Fase 4 — Identidade e AAA:** diretório, usuários, grupos, RADIUS/TACACS+ e eventual avaliação do Cisco ISE.
- **Fase 5 — Observabilidade:** centralização de logs, métricas, alertas e investigação defensiva.
- **Fase 6 — Integração:** cenários corporativos completos, troubleshooting e documentação de evidências.

A ordem poderá ser ajustada conforme os pré-requisitos e a capacidade real do hardware.

## 7. Critérios de documentação

Para cada laboratório, registrar:

- Objetivo e pré-requisitos.
- Topologia e endereçamento utilizados, sem expor segredos.
- Configurações e comandos realmente executados.
- Resultados esperados e resultados observados.
- Evidências, erros, hipóteses, correções e testes de validação.
- Conclusões próprias e pendências.

**Segurança:** não publicar senhas, tokens, chaves, licenças, configurações sensíveis ou detalhes de acesso administrativo à rede residencial.

## 8. Relação com os estudos atuais

Este documento é apenas um **registro de planejamento**. Não altera a prioridade atual: **Linux Essentials 010-160**, com leitura do material LPI e acompanhamento da CertBest.

Ao retornar aos laboratórios de redes, avaliar a continuidade do projeto **`dell-homelab-network-engineering`** e organizar as novas fases sem misturar os conteúdos com o repositório `linux-essentials`.

---

**Filosofia:** Estudar. Compreender. Praticar. Documentar. Compartilhar.
