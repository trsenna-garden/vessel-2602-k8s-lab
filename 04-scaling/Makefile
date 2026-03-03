# Configurações
NAMESPACE   := lab-scaling
PROFILE     := minikube-k8s-lab
APP_LABEL   := app=web-server
DEPLOY_NAME := app-scaling
KUBECTL     := kubectl -n $(NAMESPACE)

.PHONY: help up down apply ctx status logs shell describe port-forward wait restart diff load-test hpa-status

## help: Exibe esta mensagem de ajuda
help:
	@echo "Uso: make [comando]"
	@echo ""
	@echo "Comandos de Ciclo de Vida:"
	@echo "  up             Prepara o ambiente completo (ctx, apply, wait)"
	@echo "  down           Remove todos os recursos do projeto"
	@echo "  apply          Aplica alterações nos manifestos YAML"
	@echo ""
	@echo "Comandos de Inspeção:"
	@echo "  status         Estado geral dos recursos"
	@echo "  logs           Logs em tempo real do deployment"
	@echo "  describe       Detalhes técnicos dos pods"
	@echo "  diff           Diferença entre local e cluster"
	@echo "  hpa-status     Monitora o escalonamento em tempo real"
	@echo ""
	@echo "Comandos de Interação:"
	@echo "  shell          Acessa o terminal do container"
	@echo "  port-forward   Acesso local via localhost:8080"
	@echo "  restart        Reinicia o deployment"
	@echo "  load-test      Gera carga para testar o HPA"
	@echo ""
	@echo "Configuração (Utilitários):"
	@echo "  ctx            Define o perfil do minikube e o contexto para o namespace do lab"

## up: Prepara o ambiente completo (ctx, apply, wait)
up: ctx apply wait

## down: Remove todos os recursos definidos no diretório k8s/
down:
	kubectl delete -f k8s/ --ignore-not-found=true

## apply: Aplica todos os manifestos do diretório k8s/
apply:
	$(KUBECTL) apply -f k8s/

## ctx: Define o perfil do minikube e o namespace do projeto como padrão no contexto atual
ctx:
	@echo "Configurando perfil Minikube: $(PROFILE)"
	minikube profile $(PROFILE)
	@echo "Configurando namespace: $(NAMESPACE)"
	kubectl config set-context --current --namespace=$(NAMESPACE)

## status: Exibe o estado de todos os recursos (Pods, Services, Ingress, HPA)
status:
	@echo "--- Recursos no Namespace: $(NAMESPACE) ---"
	$(KUBECTL) get all,ingress,hpa

## logs: Acompanha os logs do Deployment
logs:
	$(KUBECTL) logs -f deploy/$(DEPLOY_NAME)

## shell: Abre um terminal interativo dentro do container
shell:
	$(KUBECTL) exec -it deploy/$(DEPLOY_NAME) -- sh

## describe: Exibe detalhes técnicos dos Pods
describe:
	$(KUBECTL) describe pod -l $(APP_LABEL)

## port-forward: Encaminha a porta local 8080 para a porta 80 do Deployment
port-forward:
	$(KUBECTL) port-forward deploy/$(DEPLOY_NAME) 8080:80

## wait: Aguarda até que os Pods estejam prontos
wait:
	$(KUBECTL) wait --for=condition=ready pod -l $(APP_LABEL) --timeout=60s

## restart: Reinicia o Deployment
restart:
	$(KUBECTL) rollout restart deployment/$(DEPLOY_NAME)

## diff: Mostra a diferença entre os manifestos locais e o cluster
diff:
	$(KUBECTL) diff -f k8s/

## load-test: Dispara carga infinita contra o serviço para forçar o HPA (use Ctrl+C para parar)
load-test:
	@echo "Iniciando teste de carga... (Pressione Ctrl+C para parar)"
	$(KUBECTL) run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://app-scaling-service; done"

## hpa-status: Monitora o estado do HPA e o número de réplicas em tempo real
hpa-status:
	$(KUBECTL) get hpa app-hpa -w
