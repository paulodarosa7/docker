# 👀 Monitoramento de Container com Docker!
Este projeto tem como objetivo a criação de um **sistema de monitoramento de containers** utilizando as ferramentas **cAdvisor**, **Prometheus** e **Grafana**.

---

## 📦 Ferramentas Utilizadas

- <img width="15" height="15" alt="prometheus" src="https://github.com/user-attachments/assets/ddf05e78-12f5-48ab-ba2c-bc35857eacc6" /> **[cAdvisor](https://github.com/google/cadvisor)** → Coleta métricas dos containers.
- **[Prometheus](https://prometheus.io/)** → Organiza e armazena as métricas.
- **[Grafana](https://grafana.com/)** → Criação de dashboards de monitoramento.

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







