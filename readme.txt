Instalação do Home Assistant no docker
Desenvolvido por Marcello Favinha
www.sejahome.one
www.intagram.com/sejahomeone

Observações Importantes:
1) Esse script feito em um raspberry Pi4 de 4gb. Ele pode funcionar em outras versões de raspberry, porém pode ter adaptações necessárias.
2) Foi usado o Raspberry OS (32-bit) Lite, porém também pode funcionar em outras versões de linux com adaptações.
3) Esse script instala (portainer, homeassistant, MQTT e node-red) já configurado e funcionando.
4) Comandos de putty as vezes pedem confirmação. Sempre responda Sim (S) ou Yes (Y)

- Baixar a última versão do Raspberry OS (32-bit) Lite (ZIP) e descompactar
  https://www.raspberrypi.org/downloads/raspberry-pi-os/

- Formate um cartão de 32gb usando SD Card Formatter
  https://www.sdcard.org/
  
- Use o Balena Etcher para queimar a ISO no cartão
  https://www.balena.io/

- Coloque o cabo de rede, teclado, cc2531 e o cartão no PI
  
- Ligue o PI
  usuário: pi
  senha: raspberry

- Para configurar a rede execute o comando abaixo:
  sudo nano /etc/dhcpcd.conf

- Descomente as opções removendo o # e configure conforme abaixo:

interface eth0
static ip_address=<IP_FIXO_DO_RASPBERY>/24
static ip6_address=<DEIXE_O_VALOR_DO_ARQUIVO>
static routers=<IP_FIXO_DO_SEU_ROTEADOR>
static domain_name_servers=<IP_FIXO_DO_SEU_ROTEADOR> <DEIXE_O_VALOR_DO_ARQUIVO_PARA_IP6>

Ctrl+X - Comando para sair do Nano
Aperte Y para salvar
Depois é só apertar enter para confirmar

- Verique se o arquivo foi ajustado e qualquer problema refaça o passo anterior
cat /etc/dhcpcd.conf

- Habilite o SSH
sudo systemctl enable ssh

- Reinicie o PI (Pode remover o teclado e monitor)
sudo systemctl reboot

- Baixar o putty
  https://www.putty.org/
  
- Execute o putty, coloque o IP escolhido na configuração, porta 22 e acesse o raspberry (Se perguntar alguma coisa, responda SIM)

- Faça update do SO
sudo apt update
sudo apt full-upgrade
sudo apt clean

- Reinicie mais uma vez
sudo systemctl reboot

- Instalando o docker

sudo apt install raspberrypi-kernel raspberrypi-kernel-headers 

curl -sSL https://get.docker.com | sh

sudo usermod -aG docker pi

sudo systemctl reboot

- Verificando se o docker está corretamente instalado
docker version

- Instalando o docker-compose
sudo apt install docker-compose

- Instale o FileZilla ou outro client FTP de preferência
  https://filezilla-project.org/
  
- Copie os arquivos do github para a pasta /home/pi
Host: <IP_FIXO_DO_RASPBERY>
Nome de usuário: pi
Senha: raspberry
Porta: 22

- Instalando todos os containers
cd /home/pi
ls #confirme se todos os arquivos estão na pasta

# Edite aqui qualquer configuração necessária para o MQTT. Por enquanto deixe o arquivo do jeito que está.
sudo nano /home/pi/homeassistant/addons/mqtt/config/mosquitto.conf

docker-compose up -d
Obs: Este comando acima irá demorar mais na primeira vez, pois o docker vai fazer download da imagem de todos os containers que estão discriminados no arquivo docker-compose.yaml. Nas próximas execuções será extremamente rápido subir ou baixar.

- Após instalado tudo é para retornar:
Creating node-red      ... done
Creating mqtt          ... done
Creating homeassistant ... done
Creating portainer     ... done

- Criando o arquivo de senha do mosquito
docker ps

- Pegue o <ID> do eclipse-mosquitto e execute o comando abaixo substituindo o ID
docker exec -it <ID> sh
Ex: docker exec -it 546040b5f50c sh
Obs: Esse comando acima entra no container do mosquito

- Ao abrir o prompt sem erro, execute o comando abaixo para criar o arquivo com a senha criptografada do mosquitto
mosquitto_passwd -c /mosquitto/config/mosquitto.passwd pi
# Coloque a senha escolhida para a senha <SENHA_DO_MQTT>

exit

- Ajuste o arquivo de configuração para não permitir mais acesso sem senha

sudo nano /home/pi/homeassistant/addons/mqtt/config/mosquitto.conf

# Remova o # das linhas abaixo:
allow_anonymous false
password_file /mosquitto/config/mosquitto.passwd

Ctrl+X - Comando para sair do Nano
Aperte Y para salvar
Depois é só apertar enter para confirmar

- Para finalizar, vamos reiniciar apenas o MQTT

# Pegue o <ID> do container do eclipse-mosquitto
docker ps

docker restart <ID>

PRONTO !! Tudo instalado !

Acessos:

- Acesso ao portainer
<IP_FIXO_DO_RASPBERY>:9000

- Acesso ao HA
<IP_FIXO_DO_RASPBERY>:8123

- Node-red
<IP_FIXO_DO_RASPBERY>:1880
Obs: Para instalar a paletta de componentes do HA no node-red. Abra Settings, Palette, Install. Procure por "home assistant" (Sem aspas). Clique em install na opção "node-red-contrib-home-assistant-websocket".


- Alguns comandos docker, caso prefira não usar o portainer:

# lista containers rodando. Muito usado para pegar o <ID> do container, necessário em outros comandos
docker ps

# lista todos os containers, inclusive os parados
docker ps -a

# inicia container
docker start <ID>

# para container
docker stop <ID>

# restart do container
docker restart <ID>

# remove container
docker rm <ID>

# remove imagem
docker rmi <ID>

# lista volumes
docker volume ls

# sobe um arquivo de docker-compose.yaml
docker-compose up -d

# para e exclui os containers
docker-compose down

# para docker-compose em um arquivo específico
docker-compose -f arquivo.yaml

# entra no docker para acessar o container
docker exec -it <ID> sh


Considerações Finais:

- Como colocar o File Editor dentro do Menu do HA ? Se descobrir me avise como.
- Cade o menu Supervisor ? Não tem. A versão docker não possui Supervisor. Isso é particularidade do HassOS. Existe uma versão do docker para isso mas não é oficial.
- Como instalo os Addons ? Acesse a página do github do addon que você deseja instalar, verifique se ele ensina os comandos para docker. Caso contrário, google. Será necessário incluir no arquivo docker-compose.yaml as configurações necessárias para o Addon e executar o comando "docker-compose down" para baixar e destruir tudo e "docker-compose up -d" para recriar tudo novamente com o addon inserido.
- Cade o Node-red no HA ? Da mesma forma do File Editor, ele não aparece no menu do HA. Será preciso acessar de acordo como mostrado nos Acessos. Se descobrir também como colocar no HA por favor me ajude.

