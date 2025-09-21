# 👀 Monitoramento de Container com Docker!
Este projeto tem como objetivo a criação de um **sistema de monitoramento de containers** utilizando as ferramentas **cAdvisor**, **Prometheus** e **Grafana**.

---

## 📦 Ferramentas Utilizadas

- <img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/5df421c8-f594-422b-bd13-75809f7ea5fd" /> **[cAdvisor](https://github.com/google/cadvisor)** → Coleta métricas dos containers.
- <img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/21eba712-c10d-4f77-b892-c01020578a7e" /> **[Prometheus](https://prometheus.io/)** → Organiza e armazena as métricas.
- <img width="15" height="15" alt="image" src="https://github.com/user-attachments/assets/3471f2c6-75bb-4264-ad6e-7182348705c2" /> **[Grafana](https://grafana.com/)** → Criação de dashboards de monitoramento.

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
## 🪛 Configuração do Prometheus

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
      - targets: ['ip_local:8080']
```
Estes são os parametros que serão passados para o prometheus para que a ferramenta consiga coletar os dados dos containers. 
Porém precisamos colocar o endereço ip do host na configuração.

No terminal, rode:
```bash
ip a
```
Copie o IP do host (geralmente a placa de rede será eno1 ou enops1) e cole no - targets do nosso arquivo de configuração prometheus.yml
```bash
nano prometheus.yml
```
```nano
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['192.168.x.x:8080'] <--------------------------------------- coloque o seu endereço ip
```
---
## 💡 Manipulação de Containers
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
É interessante verificarmos se não há containers que utilizam as portas 8080, 3000 ou 9090 da sua máquina, pois haverá conflito. Quando realizarmos a pull das imagens precisaremos alterar a porta que o container irá rodar, ```docker run -p nova_porta_host:porta_container``` Se é um container que não está em uso, será preciso "stopar" o container.

---
## 🐳 Pull de Containers
pull do Prometheus:
```bash
docker pull prom/prometheus:main
docker run -d --name prometheus \
  -p 9090:9090 \
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
  --publish=8080:8080 \
  --detach=true \
  --device=/dev/kmsg \
  gcr.io/cadvisor/cadvisor:latest
```
pull do Grafana:
```bash
docker pull grafana/grafana:main-ubuntu
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```
Acesse na web:
http://localhost:3000 -> Grafana, credenciais (admin / admin)
<img width="1055" height="730" alt="image" src="https://github.com/user-attachments/assets/acfe0210-b5bc-4a13-a19a-fc2f199c8eeb" />

http://localhost:8080/metrics -> métricas do cAdvisor
<img width="1880" height="189" alt="image" src="https://github.com/user-attachments/assets/62b3868a-90fe-42a5-8a3e-0791a0e94563" />

http://localhost:9090/targets -> Prometheus > O alvo cAdvisor deve estar como UP!
<img width="1898" height="291" alt="image" src="https://github.com/user-attachments/assets/4ce34ac0-c54f-4b41-8cd9-815697d83ef1" />

Estando desta forma, nossos serviços estão funcionando. Prosseguiremos com a configuração de nossas dashboards!

## 📊 Configurando Dashboards com Grafana!







