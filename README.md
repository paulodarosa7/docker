# 👀 Monitoramento de Container com Docker!
Este projeto tem como objetivo a criação de um **sistema de monitoramento de containers** utilizando as ferramentas **cAdvisor**, **Prometheus** e **Grafana**.

---

## 🔨 Ferramentas Utilizadas
- <img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/38611b3e-5266-46de-acdd-eaf68c0f811b" />  **[Docker](https://www.docker.com/)** → Plataforma de gerenciamento de containers.
- <img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/5df421c8-f594-422b-bd13-75809f7ea5fd" /> **[cAdvisor](https://github.com/google/cadvisor)** → Coleta métricas dos containers.
- <img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/21eba712-c10d-4f77-b892-c01020578a7e" /> **[Prometheus](https://prometheus.io/)** → Organiza e armazena as métricas.
- <img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/3471f2c6-75bb-4264-ad6e-7182348705c2" /> **[Grafana](https://grafana.com/)** → Criação de dashboards de monitoramento.
- <img width="18" height="18" alt="image" src="https://github.com/user-attachments/assets/12a920c5-7fd3-462a-95b4-83b993f4650e" /> **[Prometheus Node Exporter](https://github.com/prometheus/node_exporter/releases)** → Coleta dados do host.

---

## 🌎 Preparação de um Ambiente Virtual

Criar e ativar o ambiente virtual:
```bash
virtualenv ambiente
source ambiente/bin/activate
```
Instalação de algumas dependências da ferramenta.
```bash
pip3 install docker
```
---
## ⚙️ Configuração do Prometheus

Criação de uma pasta chamada "prometheus" e um arquivo de texto chamado de "prometheus.yml"

```bash
mkdir prometheus ; cd prometheus
nano prometheus.yml
```

Dentro do arquivo "prometheus.yml" adicione o conteúdo:
```bash
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```
Estes são os parametros que serão passados para o prometheus para que a ferramenta consiga coletar os dados dos containers. 
---
## 📟 Manipulação de Containers
Há diversos comandos que utilizamos para "cuidar" dos nossos containers. 

Para listar containers ativos:
```bash
docker ps
```
Para listar containers ativos e inativos:
```bash
docker ps -a
```
Para listar containers em tempo real:
```bash
docker stats
```
Para dar start, stop e restart em algum container:
```bash
docker stop nome_ou_id_do_container
docker start nome_ou_id_do_container
docker restart nome_ou_id_do_container
```
Para remover:
```bash
docker stop container
docker remove container
```
---
⚠️ **Atenção às portas**

Antes de iniciar a pull dos containers, verifique se as portas 8080, 3000 ou 9090 já não estão em uso em sua máquina.

Se já estiverem em uso por outro container, será necessário parar esse container (docker stop nome_ou_id).

Caso não queira parar o container existente, você pode alterar a porta do host ao rodar o comando com -p.

Por exemplo: (não compilar)
```bash
docker run -d -p 9091:9090 prom/prometheus
```

Nesse caso, o Prometheus continua rodando dentro do container na porta 9090, mas estará acessível pelo host em 9091.

---
## 🌐 Criação de uma Docker Network
Vamos criar uma network chamada de mynet
```bash
docker network create -d bridge mynet
```
A rede em docker é para facilitar a comunicação entre containers. 
---
## 🐳 Pull de Containers
pull do Prometheus:
```bash
docker pull prom/prometheus:main
docker run -d --name prometheus \
  -p 9090:9090 --network=mynet \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```
pull do cAdvisor:
```bash
docker run -d --name=cadvisor \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --network=mynet \
  --publish=8080:8080 \
  --detach=true \
  --device=/dev/kmsg \
  gcr.io/cadvisor/cadvisor:latest
```
pull do Grafana:
```bash
docker pull grafana/grafana:main-ubuntu
docker run -d --network=mynet --name=grafana -p 3000:3000 grafana/grafana
```

pull do Node Exporter
```bash
docker pull prom/node-exporter:master
docker run -d \
  --network=mynet \
  -p 9100:9100 \
  --pid="host" \
  -v "/:/host:ro" \
  --name=node-exporter \
  quay.io/prometheus/node-exporter:latest \
  --path.rootfs=/host
```

Acesse na web:
http://localhost:3000 → Grafana, credenciais (admin / admin)
<img width="1055" height="730" alt="image" src="https://github.com/user-attachments/assets/acfe0210-b5bc-4a13-a19a-fc2f199c8eeb" />

http://localhost:8080/metrics → métricas do cAdvisor
<img width="1880" height="189" alt="image" src="https://github.com/user-attachments/assets/62b3868a-90fe-42a5-8a3e-0791a0e94563" />

http://localhost:9090/targets → Prometheus > O alvo cAdvisor deve estar como UP!
<img width="1898" height="291" alt="image" src="https://github.com/user-attachments/assets/4ce34ac0-c54f-4b41-8cd9-815697d83ef1" />

Estando desta forma, nossos serviços estão funcionando. Prosseguiremos com a configuração de nossas dashboards!

---

## 📊 Configurando Dashboards com Grafana!

### Adicionando o Prometheus API como data source
Acesse na web: http://localhost:3000

Coloque as credenciais admin / admin (altere a senha, se desejar)

Na tela inicial do Grafana, acesse o menu lateral
<img width="1901" height="284" alt="image" src="https://github.com/user-attachments/assets/51ffc7a6-c890-48b7-a289-0bd580f7c9a6" />

Encontre a opção "Connections" e depois clique em Data sources
<img width="1892" height="570" alt="image" src="https://github.com/user-attachments/assets/d9943c98-a3ab-49d8-affc-d1cf40ac26fa" />

Clique em Add data source e encontre o data source Prometheus.
<img width="1898" height="498" alt="image" src="https://github.com/user-attachments/assets/4f71f991-e977-4402-9156-700fc250b499" />

<img width="1877" height="280" alt="image" src="https://github.com/user-attachments/assets/0d9e9c9b-a359-4db1-a50d-40612ab8c4ec" />

Nas configurações do prometheus, em Connection, coloque o ip da sua máquina ou do servidor, onde o prometheus está rodando, digite ```ip a``` em nosso terminal e dê um enter. Para ver o ip da máquina geralmente aparece como eth0 ou enp0s0, se o serviço roda em localhost, coloque o ip mesmo assim, o grafana não encontra se colocar http://localhost:9090

http://ip_servidor:9090 → (porta utilizada pelo prometheus) ou http://prometheus:9090/ 

<img width="1865" height="706" alt="image" src="https://github.com/user-attachments/assets/f5ed747a-b2e5-4eb1-8266-f1363effcfd1" />

Role até o fim da tela e clique em Save & Test, deverá retornar sucesso.
<img width="1875" height="349" alt="image" src="https://github.com/user-attachments/assets/09796e06-09e8-4c6a-80fd-e780e2866805" />

### Criando a dashboard
Ao retornar sucesso com o data source, volte ao menu lateral e entre na opção Dashboards
<img width="1892" height="530" alt="image" src="https://github.com/user-attachments/assets/95717236-7382-403b-a15d-27ebb494253c" />

Em Dashboards, iremos importar um modelo de dashboard pronto.
<img width="1904" height="548" alt="image" src="https://github.com/user-attachments/assets/06a3bfa9-7daa-4562-9be2-199bb23a547f" />

Iremos importar uma Dashboard criada pela Docker e pelo Grafana Labs Solutions, com id 893, clique em Load.
<img width="1906" height="736" alt="image" src="https://github.com/user-attachments/assets/97c07aa1-0ca9-4466-92d0-3e522f6a487e" />
Selecione a Prometheus API, e clique em Import.
<img width="1906" height="736" alt="image" src="https://github.com/user-attachments/assets/ba9f2079-c3d2-4343-adea-4e54e3869cc3" />

Pronto! A dashboard está funcional! Altere apenas a hora de atualização, por padrão vem em 24 horas, altere para os últimos 5 minutos.
<img width="1866" height="886" alt="image" src="https://github.com/user-attachments/assets/907ae949-967e-4055-81f2-efee5a4ff9b6" />

---
### 📖 Dicionário de Argumentos Docker

| Argumento / Comando | Significado |
|----------------------|-------------|
| `docker pull <imagem>` | Realiza o download da imagem do Docker Hub para sua máquina. |
| `docker run <imagem>` | Cria e executa um container a partir de uma imagem. |
| `-d` | A flag -d (abreviação de --detach) executa o contêiner em segundo plano. Isso significa que o Docker inicia o contêiner e retorna ao prompt do terminal. Além disso, ele não exibe logs no terminal. |
| `--name nome` | Define um **nome personalizado** para o container. |
| `-p host:container` | O sinalizador -p (abreviação de --publish) cria um mapeamento de portas entre o host e o contêiner. O sinalizador -p recebe um valor de string no formato HOST:CONTAINER, onde HOST é o endereço no host e CONTAINER é a porta no contêiner |
| `-v host:container` | Monta um **volume** (pasta/arquivo do host disponível dentro do container). |
| `--rm` | Remove o container automaticamente após ser finalizado. |
| `--detach=true` | Similar ao `-d`, roda o container em segundo plano. |
| `--device=/dev/kmsg` | Dá acesso a um dispositivo do host dentro do container. |
| `docker ps` | Lista containers ativos. |
| `docker ps -a` | Lista containers ativos **e inativos**. |
| `docker stop <id/nome>` | Para um container em execução. |
| `docker start <id/nome>` | Inicia um container parado. |
| `docker restart <id/nome>` | Reinicia um container. |
| `docker [remove ou rm] <id/nome>` | Remove um container. |
| `docker rmi <imagem>` | Remove uma imagem do host. |
| `docker logs <id/nome>` | Mostra os **logs de saída** de um container. |
| `docker stats` | Exibe estatísticas em tempo real de CPU, memória e rede dos containers. |

---

### 📝 Referências
[Docker and system monitoring](https://grafana.com/grafana/dashboards/?search=893)

[Containerize an application](https://docs.docker.com/get-started/workshop/02_our_app/)

[Docker Hub](https://hub.docker.com/)

[cAdvisor](https://github.com/google/cadvisor)

[Grafana](https://grafana.com/)

[Prometheus](https://prometheus.io/)



















