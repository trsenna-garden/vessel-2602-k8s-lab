# Kubernetes Labs

Repositório dedicado ao estudo e aplicação prática de Kubernetes.

## Estrutura

- `01-persistence-init/`: Laboratório sobre persistência de dados e Init Containers.

## Setup do Ambiente

Para a correta execução dos laboratórios, certifique-se de que o ambiente (ex: Minikube ou Kind) esteja com os seguintes componentes ativos:

1. **Ingress Controller**: Necessário para exposição de serviços.
2. **Metrics Server**: Necessário para monitoramento de recursos e HPA.

### Comandos úteis (Minikube)

```bash
minikube start

minikube addons enable ingress
minikube addons enable metrics-server
```
