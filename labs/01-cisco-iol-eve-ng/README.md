Lab 01 — Cisco IOL Layer 2 e Layer 3 no EVE-NG
Ambiente: servidor Dell → Proxmox VE → VM 101 (EVE-NG)
Tema: inicialização de imagens, interfaces Ethernet, CDP, tabela MAC e troubleshooting de duplex
Status: laboratório L2 operacional; correção de duplex não comprovada; IPv4 e ICMP pendentes.
1. Por que este laboratório existe?
O objetivo é entender o funcionamento de um laboratório de redes virtualizado, e não apenas fazer os nós aparecerem como ligados na interface gráfica. Registramos o que foi observado, o que foi testado, os erros e o que ainda não foi demonstrado.

Competências praticadas:
- Diferenciar hardware físico, hipervisor, máquina virtual e equipamento de rede emulado.
- Inicializar Cisco IOL L2/L3 e diagnosticar falhas de inicialização.
- Ler o estado administrativo e operacional de interfaces Ethernet.
- Interpretar descoberta CDP e aprendizado de MAC em switches.
- Investigar DUPLEX_MISMATCH sem confundir indício com prova.
- Usar Linux, SSH, VS Code e Git para documentar o trabalho.
2. Ambiente e topologia
Camada	Componente	Informação observada
Hardware	Servidor Dell	Hospeda Proxmox VE
Virtualização	Proxmox VE	VM 101 identificada como eve-ng
VM	EVE-NG Community	Versão 2.0.3-95, Ubuntu 16.04.5 LTS
Recursos da VM	VM 101	2 vCPU, 8 GB RAM, 40 GB disco
Router	Cisco IOL L3 (R4)	IOS 15.4(2)T, 4 interfaces Ethernet
Switch	Cisco IOL L2 (R2)	IOS 15.2, build CML Nightly maio/2018; 4 interfaces Ethernet
Hosts	VPC1, VPC2	Presentes na topologia; endereçamento ainda pendente


                 R4 — Router (Cisco IOL L3)
                         Ethernet0/0
                             |
                         Ethernet0/2
                 R2 — Switch (Cisco IOL L2)
                   Ethernet0/0  Ethernet0/1
                        |             |
                       VPC1          VPC2
Atenção: a figura representa conexões lógicas do EVE-NG. O switch físico Catalyst 3560G não participou destes testes.
3. Preparação e problemas na inicialização
As imagens IOL foram preparadas no EVE-NG, em diretório relacionado a /opt/unetlab/addons/iol/bin/. Durante os testes ocorreram:
1. Falhas para iniciar o nó IOL L3.
2. Mensagens envolvendo a localização do arquivo iourc.
3. Mensagens relacionadas a sockets AF_UNIX.
4. Tentativas repetidas de inicialização; posteriormente os consoles IOS dos nós L2 e L3 passaram a funcionar.
Resultado comprovado: ambos os equipamentos iniciaram e aceitaram comandos IOS.
Não comprovado: a causa-raiz das falhas anteriores. Não é correto afirmar que uma única alteração específica resolveu definitivamente o problema.

Segurança: não incluir arquivos de licença/autorização nem seu conteúdo no GitHub.
Erros reais de comando observados
No console Linux do EVE-NG houve tentativas com caminhos digitados incorretamente para o wrapper de permissões, produzindo command not found e No such file or directory. O caminho utilizado posteriormente foi:
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
A ausência de erro nessa execução não comprova, por si só, que ela tenha sido a causa definitiva da recuperação dos nós. Esse comando altera permissões e não deve ser executado indiscriminadamente em outros ambientes.
4. Interfaces: estado administrativo x operacional
A interface Ethernet0/0 do Router começou administratively down. Foi aplicada a habilitação:
configure terminal
interface Ethernet0/0
no shutdown
end
show ip interface brief
O enlace foi observado como up/up.
O que significa up/up?
- Status up: interface administrativamente habilitada e com enlace detectado.
- Protocol up: protocolo de linha operacional.
- administratively down: interface desabilitada por configuração (shutdown).
up/up indica enlace operacional, não prova que o IPv4 está configurado ou que um ping funcionará.
5. CDP — descoberta de vizinhos
Comandos utilizados:
show cdp neighbors
show interfaces Ethernet0/0
show interfaces Ethernet0/2
show ip interface brief
O CDP identificou o Router e o Switch como vizinhos diretamente conectados, permitindo associar:
- Router R4: Ethernet0/0.
- Switch R2: Ethernet0/2.
Conceito: CDP (Cisco Discovery Protocol) é um protocolo de descoberta de vizinhos diretamente conectados. Pode funcionar mesmo antes de configurarmos endereços IPv4 nos dispositivos.

Limite do teste: enxergar um vizinho via CDP não comprova que o enlace esteja livre de problemas de duplex ou que haja conectividade IP.
6. Switch L2 — aprendizado da tabela MAC
Comando executado no switch:
show mac address-table
Registro observado:
VLAN 1 — MAC aabb.cc00.4000 — Ethernet0/2
Interpretação: o switch associou o endereço MAC de origem observado à porta de entrada Ethernet0/2, na VLAN 1. Essa é a base do encaminhamento Ethernet por tabela MAC.

Não confundir:
- Tabela MAC: mapeia MAC → porta/VLAN no switch L2.
- Tabela ARP: associa IPv4 → MAC em um dispositivo IPv4.
- Tabela de rotas: seleciona caminhos para redes IP.
Ainda não realizamos os testes de ARP dos VPCS neste laboratório.
7. Troubleshooting — %CDP-4-DUPLEX_MISMATCH
7.1 Sintoma
O Router registrou mensagens semelhantes a:
%CDP-4-DUPLEX_MISMATCH: duplex mismatch discovered on Ethernet0/0 (not full duplex), with Switch Ethernet0/2 (full duplex).
A mensagem aponta uma incompatibilidade anunciada pelo CDP: o switch indicava full-duplex e o roteador era identificado como não full-duplex.
7.2 Comandos e evidências
No Switch, foi executado:
show running-config interface Ethernet0/2
show interfaces Ethernet0/2
A configuração apresentou:
interface Ethernet0/2
 duplex full
A saída da interface indicou Full-duplex, Auto-speed, sem erros CRC ou colisões nos contadores consultados.

No Router, foram executados:
show running-config interface Ethernet0/0
show interfaces Ethernet0/0
show running-config all | section interface Ethernet0/0
A saída do running-config mostrou apenas:
interface Ethernet0/0
 no ip address
Mesmo com show running-config all, não apareceu um campo explícito de duplex. A saída consultada de show interfaces também não forneceu a indicação esperada.

Foi testado:
configure terminal
interface Ethernet0/0
duplex full
end
show running-config interface Ethernet0/0
O IOS aceitou duplex full, mas a configuração não apareceu no running-config apresentado posteriormente.

Conclusão correta: aceitar um comando não basta para provar que a operação mudou. A ausência da linha no running-config também não permite afirmar, sozinha, qual era o duplex operacional efetivo nessa imagem IOL.
7.3 Logs e linha do tempo
Comando:
show logging | include DUPLEX_MISMATCH
Evento	Horário observado no log
Primeiro alerta apresentado	02:38:16
Último alerta apresentado	02:48:02
Última aplicação de duplex full	02:57:44


Não foram apresentados alertas posteriores à última tentativa de configuração na saída consultada. Isso não demonstra sozinho que a incompatibilidade foi eliminada, pois os registros exibidos podem ser anteriores e o CDP opera com atualizações periódicas.
7.4 O que significa duplex?
- Half-duplex: transmissão e recepção não ocorrem simultaneamente no mesmo enlace; tradicionalmente associado a Ethernet compartilhada.
- Full-duplex: transmissão e recepção simultâneas; padrão em enlaces Ethernet comutados modernos.
- Duplex mismatch: as duas pontas usam modos incompatíveis, o que pode causar degradação, erros e retransmissões.
7.5 O que ainda seria necessário para encerrar o diagnóstico?
Confirmar, em um teste futuro, o estado efetivo do enlace após a alteração, observar o comportamento do CDP ao longo de novas atualizações e verificar contadores de erros e tráfego. Caso a imagem IOL não exponha o estado operacional de duplex, registrar a limitação do emulador em vez de inventar uma confirmação.

Status do incidente: investigado, correção definitiva não confirmada.
8. Rede Linux do EVE-NG
No console da VM EVE-NG, e não no host Proxmox, foi executado:
ip -br addr
Resultado IPv4 relevante:
lo      UNKNOWN   127.0.0.1/8
eth0    UP
pnet0   UP        192.168.0.16/24
Também apareceram pnet1 a pnet9, interfaces vnet* e vunl*, além de endereços IPv6 em pnet0.

Aprendizado: a interface pnet0 é parte da conectividade virtual do EVE-NG. O endereço 192.168.0.16/24 é da VM EVE-NG, não deve ser confundido com o IP do host Proxmox.

A configuração das bridges físicas e virtuais do host Proxmox ainda não foi levantada neste laboratório.
9. Git e VS Code — como este laboratório foi documentado
A documentação foi preparada em uma VM Ubuntu de estudos separada, acessada pelo VS Code via Remote-SSH.
9.1 Identificar onde estamos
pwd
hostname
Foram observados o host ubuntu-lab e o projeto anterior em /home/aluno/linux-estudos. Antes de clonar o novo repositório, usamos cd ~ para ir ao diretório pessoal e evitar colocar um projeto dentro do outro.
9.2 Clonar o repositório
A primeira tentativa teve um erro de digitação (hhtps em vez de https), que produziu:
git: 'remote-hhtps' não é um comando git.
fatal: remote helper 'hhtps' aborted session
Após corrigir a URL, o clone concluiu com Receiving objects: 100%:
git clone https://github.com/dias0001/dell-homelab-network-engineering.git
cd ~/dell-homelab-network-engineering
Conceito: git clone cria uma cópia local de um repositório remoto. Como o repositório já existia no GitHub, não foi necessário executar git init.
9.3 Navegação e edição
pwd
ls -l
cat README.md
code .
code README.md
code labs/01-cisco-iol-eve-ng/README.md
- pwd: mostra o diretório atual.
- ls -l: lista com detalhes; - no início indica arquivo regular e d indica diretório.
- cat: mostra o conteúdo, sem abrir um editor.
- code .: abre o diretório atual no VS Code.
- code arquivo: abre um arquivo no VS Code (quando a integração está disponível).
- O Explorer do VS Code pode exibir Compact Folders: pastas aninhadas são mostradas em uma única linha, sem alterar a estrutura real.
9.4 Por que git status inicialmente estava limpo?
A pasta labs/01-cisco-iol-eve-ng havia sido criada, mas estava vazia. Foi observado:
ls -l labs/01-cisco-iol-eve-ng
# total 0
Git não rastreia diretórios vazios. Depois de criar o arquivo:
touch labs/01-cisco-iol-eve-ng/README.md
git status
O Git passou a mostrar labs/ como Arquivos não monitorados (untracked files).

Fluxo de publicação: editar/salvar → git status → git add → git commit → git push. git push envia commits, não arquivos avulsos.
10. Resultado consolidado
Verificação	Estado
VM EVE-NG inicializa	Confirmado
Nós Cisco IOL L2/L3 inicializam	Confirmado
Interface Router–Switch up/up	Confirmado
CDP identifica vizinhos	Confirmado
Switch aprende MAC na VLAN 1	Confirmado
Alerta DUPLEX_MISMATCH foi observado	Confirmado
Correção definitiva do duplex	Não confirmada
IP 192.168.10.1/24 configurado no Router	Não executado neste registro
IPv4 nos VPCS	Pendente
ARP e ping	Pendentes
VPN WireGuard / OPNsense	Não implementada


11. Próximo laboratório — planejamento, não execução
Proposta de endereçamento para continuar quando houver tempo:
Dispositivo	Interface	IPv4 proposto
Router R4	Ethernet0/0	192.168.10.1/24
VPC1	Interface VPCS	192.168.10.10/24, gateway 192.168.10.1
VPC2	Interface VPCS	192.168.10.20/24, gateway 192.168.10.1


A partir daí: validar IP → observar ARP → testar ICMP → analisar a tabela MAC → documentar evidências. Não registrar como executado antes de realizar os testes.
12. Revisão para estudar na praia 🏖️
Tente responder sem consultar as explicações anteriores:
1. Qual a diferença entre servidor físico, hipervisor, VM e roteador emulado?
2. Por que uma interface pode aparecer como administratively down?
3. O que up/up comprova? O que não comprova?
4. Para que serve o CDP? Ele exige endereços IPv4 configurados?
5. Como o switch aprende um endereço na tabela MAC?
6. Qual a diferença entre tabela MAC, ARP e tabela de rotas?
7. O que é full-duplex? Por que um DUPLEX_MISMATCH merece investigação?
8. Por que duplex full ser aceito no CLI não encerrou nosso diagnóstico?
9. O que ip -br addr mostrou sobre a VM EVE-NG? Isso é a configuração do host Proxmox?
10. Qual a diferença entre cat README.md, code README.md e code .?
11. Por que git status não mostrou a pasta vazia, mas passou a mostrar labs/ depois do touch?
12. Qual a diferença entre git clone, git add, git commit e git push?
Respostas resumidas para autocorreção
1. Hardware executa o hipervisor; hipervisor hospeda VMs; EVE-NG é uma VM; o Cisco IOL é um nó emulado dentro dela.
2. Pode estar desabilitada com shutdown.
3. Indica enlace e protocolo de linha operacionais; não prova endereçamento IP ou ping.
4. Descobre vizinhos Cisco diretamente conectados; não exige IPv4 na interface para a descoberta básica.
5. Observando o MAC de origem de quadros recebidos, por porta/VLAN.
6. MAC → porta/VLAN; ARP → associação IPv4/MAC; rotas → redes/próximo salto.
7. Full-duplex permite enviar e receber simultaneamente; mismatch pode degradar o enlace.
8. Não houve comprovação do duplex operacional nem verificação conclusiva posterior.
9. Mostrou pnet0 com 192.168.0.16/24 na VM EVE-NG; não mostrou a rede do host Proxmox.
10. cat exibe texto; code arquivo abre o arquivo; code . abre a pasta atual.
11. Git rastreia arquivos, não pastas vazias; touch criou um arquivo não monitorado.
12. Clonar cópia remota; preparar alterações; registrar commit local; publicar commits no remoto.
13. Segurança e limitações
- Não publicar arquivos iourc, credenciais, chaves privadas ou configurações sensíveis.
- Não expor a administração do EVE-NG ou do Proxmox diretamente à internet.
- As versões antigas do EVE-NG/Ubuntu observadas exigem cuidado adicional; não tratar esse ambiente como plataforma pública endurecida.
- Este README registra o que foi possível reconstruir dos comandos, logs e capturas compartilhados. Não substitui a coleta completa de evidências dos equipamentos.
Princípio do laboratório: o erro só vira aprendizado quando registramos o que aconteceu, por que aconteceu e o que realmente conseguimos demonstrar.
