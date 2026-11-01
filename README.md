Dell HomeLab | Network Engineering & Security
Estudar. Compreender. Praticar. Documentar. Compartilhar.

Laboratório pessoal de engenharia de redes, virtualização, troubleshooting e segurança defensiva, desenvolvido para consolidar conhecimentos práticos de infraestrutura corporativa, NOC/SOC e Blue Team.
Visão geral
O laboratório utiliza um servidor físico Dell, com Proxmox VE como hipervisor. Uma das máquinas virtuais executa o EVE-NG, no qual são criadas topologias com roteadores, switches e hosts virtuais.
Servidor Dell (hardware físico)
└── Proxmox VE (hipervisor)
    ├── VM 101 — EVE-NG (laboratórios de redes)
    │   └── Cisco IOL L3 / Cisco IOL L2 / VPCS
    └── Outras VMs (fora do escopo do Lab 01)

Estação de documentação: VS Code → Remote-SSH → Ubuntu de estudos
Repositório remoto: GitHub
Importante: o Ubuntu de estudos usado para editar e publicar esta documentação não é a VM EVE-NG. O repositório é mantido separadamente do ambiente de laboratório.
Laboratórios
Lab	Tema	Situação
[01 — Cisco IOL L2/L3 no EVE-NG](labs/01-cisco-iol-eve-ng/README.md)	Inicialização, interfaces, CDP, tabela MAC, duplex e diagnóstico	Documentado; testes IPv4 pendentes


Infraestrutura conhecida
Componente	Papel	Observações
Servidor Dell	Hardware de virtualização	Modelo não registrado nesta documentação
Proxmox VE	Hipervisor	Hospeda a VM 101 do EVE-NG
VM 101 — EVE-NG	Emulação de redes	EVE-NG Community 2.0.3-95; Ubuntu 16.04.5 LTS; 2 vCPU, 8 GB RAM, 40 GB de disco (dados observados no estudo)
Cisco IOL L3	Roteador virtual	IOS 15.4(2)T
Cisco IOL L2	Switch virtual	IOS 15.2, build CML Nightly de maio de 2018
VPCS	Hosts virtuais	Ainda sem testes IPv4 concluídos
Cisco Catalyst 3560G	Switch físico disponível	Não utilizado no Lab 01
Modem/roteador Claro	Rede residencial	Topologia e política de acesso externo ainda não verificadas


Projetos futuros — ainda não implementados
- Configurar IPv4, ARP e ICMP na topologia do Lab 01.
- Avaliar acesso remoto seguro ao HomeLab com OPNsense + WireGuard, após levantamento da rede Proxmox e da conexão residencial (inclusive possível CGNAT).
- Evoluir para segmentação, regras de firewall, logs, monitoramento e práticas Blue Team.
Status da VPN: apenas ideia/planejamento. Não há instalação ou túnel validado neste projeto.

Como documentamos
Cada laboratório registra objetivo → topologia → comandos → evidências → problemas → hipóteses → resultados → pendências → revisão. Resultados não comprovados permanecem explicitamente marcados como não confirmados.
Segurança e publicação
Este repositório é público. Não publicar senhas, chaves SSH ou WireGuard, tokens, arquivos de licença/autorização, backups de configuração com segredos, dados pessoais ou capturas que revelem acessos administrativos desnecessários. Fabricante e tecnologia, isoladamente, não equivalem a uma vulnerabilidade, mas versões antigas e detalhes de exposição exigem atenção.
Fluxo de edição pelo Ubuntu de estudos
# Executar na VM Ubuntu de estudos, no diretório do repositório
pwd
git status
# Editar e salvar os arquivos com VS Code
# Conferir as alterações antes de publicar
git diff
git add README.md labs/01-cisco-iol-eve-ng/README.md
git diff --cached
git commit -m "docs: documenta laboratorio Cisco IOL no EVE-NG"
git push origin main
Atenção: os comandos acima são um roteiro para execução manual, não um registro de comandos já executados neste repositório.
Referências de estudo
- Git — documentação oficial
- Proxmox VE — documentação
- EVE-NG — documentação
- Cisco — documentação e suporte
