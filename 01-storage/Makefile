# Configurações
NAMESPACE   := lab-storage
PROFILE     := minikube-k8s-lab
APP_LABEL   := app=web-server
DEPLOY_NAME := app-nginx
KUBECTL     := kubectl -n $(NAMESPACE)

.PHONY: help up down apply ctx status logs shell describe port-forward wait restart diff

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
	@echo ""
	@echo "Comandos de Interação:"
	@echo "  shell          Acessa o terminal do container"
	@echo "  port-forward   Acesso local via localhost:8080"
	@echo "  restart        Reinicia o deployment (testa persistência)"
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

## status: Exibe o estado de todos os recursos (Pods, Services, PVCs, Ingress)
status:
	@echo "--- Recursos no Namespace: $(NAMESPACE) ---"
	$(KUBECTL) get all,pvc,ingress

## logs: Acompanha os logs do Deployment
logs:
	$(KUBECTL) logs -f deploy/$(DEPLOY_NAME)

## shell: Abre um terminal interativo dentro do container do Nginx
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

## restart: Reinicia o Deployment (útil para testar initContainers e persistência)
restart:
	$(KUBECTL) rollout restart deployment/$(DEPLOY_NAME)

## diff: Mostra a diferença entre os manifestos locais e o cluster
diff:
	$(KUBECTL) diff -f k8s/
