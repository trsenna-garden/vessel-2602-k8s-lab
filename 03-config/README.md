# Lab 03 - Explorando Health Probes no Kubernetes

Este laboratório demonstra o funcionamento prático das **Startup**, **Readiness** e **Liveness** Probes utilizando um servidor Nginx. O cenário foi configurado para simular uma aplicação que demora a iniciar e permite a manipulação manual do seu estado de saúde.

## O Cenário do Laboratório

O container Nginx foi configurado para:
1.  Aguardar **20 segundos** (`sleep 20`) antes de iniciar o processo do Nginx.
2.  Criar três arquivos de "check" no diretório `/tmp`: `startup_ok`, `ready_ok` e `alive_ok`.
3.  As Probes do Kubernetes monitoram a existência desses arquivos via comando `cat`.

## Como Iniciar

1.  **Prepare o ambiente e suba os recursos:**
    ```bash
    make up
    ```
2.  **Acompanhe o estado do Pod em tempo real:**
    ```bash
    watch kubectl get pods
    ```
    *(Você notará que o Pod levará cerca de 25-30 segundos para ficar `1/1 Ready` devido ao delay inicial).*

---

## Roteiro de Testes

### 1. Validando a Startup Probe
A **Startup Probe** bloqueia as outras probes até que ela passe.
- **Observação:** Durante os primeiros 20 segundos, o container estará no estado `Running`, mas o status do Pod não mudará para `Ready` porque o arquivo `/tmp/startup_ok` ainda não existe.
- **O que testar:** Tente rodar `make status` repetidamente logo após o `make up` e veja o campo `STATUS` e `READY`.

### 2. Validando a Readiness Probe
A **Readiness Probe** determina se o Pod pode receber tráfego do Service/Ingress.
- **Teste:** Remova o arquivo de prontidão:
  ```bash
  make shell
  # Dentro do container:
  rm /tmp/ready_ok
  exit
  ```
- **Resultado:** Execute `make status`. O Pod aparecerá como `0/1 Ready`. O container **não** é reiniciado, mas o Ingress parará de enviar requisições para este Pod.
- **Restauração:** Entre no shell novamente e execute `touch /tmp/ready_ok` para ver o Pod voltar a ficar pronto.

### 3. Validando a Liveness Probe
A **Liveness Probe** determina se o container deve ser reiniciado.
- **Teste:** Remova o arquivo de vitalidade:
  ```bash
  make shell
  # Dentro do container:
  rm /tmp/alive_ok
  exit
  ```
- **Resultado:** Após alguns segundos (baseado no `failureThreshold`), o Kubernetes detectará a falha e **reiniciará o container**.
- **Observação:** O comando `kubectl get pods` mostrará o contador na coluna `RESTARTS` subindo para 1.

---

## Comandos Úteis (Makefile)

- `make up`: Aplica os manifestos e configura o contexto.
- `make status`: Mostra o estado atual dos recursos.
- `make logs`: Acompanha a saída do Nginx.
- `make shell`: Acessa o container para manipular os arquivos de teste.
- `make down`: Limpa todos os recursos do laboratório.
