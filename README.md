# Monitoramento de Container com Docker!
Este projeto tem como objetivo a criação de um **sistema de monitoramento de containers** utilizando as ferramentas **cAdvisor**, **Prometheus** e **Grafana**.

---

## 📦 Ferramentas Utilizadas

- **[cAdvisor](https://github.com/google/cadvisor)** → Coleta métricas dos containers.
- **[Prometheus](https://prometheus.io/)** → Organiza e armazena as métricas.
- **[Grafana](https://grafana.com/)** → Criação de dashboards de monitoramento.

---

## ⚙️ Preparação do Ambiente

Criar e ativar o ambiente virtual:
```bash
virtualenv ambiente
source ambiente/bin/activate
```
## Instalação de algumas dependências da ferramenta.
```bash
pip3 install docker
```
## Realizaremos a configuração do nosso Prometheus, primeiro crie uma pasta chamada prometheus e um arquivo prometheus.yml
```bash
mkdir prometheus ; cd prometheus
nano prometheus.yml
```
