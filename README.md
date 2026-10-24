# k8s-dash

Painel interativo em nginx que mostra, ao vivo, **qual pod e qual nó** do Kubernetes respondeu cada requisição. Com ele dá para ver na prática o balanceamento de carga, o self-healing, a escala e o rolling update.
  ![Painel k8s-dash](dashboard.png)

Feito para o vídeo **Kubernetes na Prática** no YouTube, rodando num cluster local com **k3d**.

## O que tem no manifesto

Tudo está em um único arquivo, o `k8s-dash.yaml`:

| Objeto | Para que serve |
|---|---|
| ConfigMap `k8s-dash-nginx` | Configuração do nginx com o endpoint `/api/info` (pod, IP e nó) |
| ConfigMap `k8s-dash-html` | A página do painel (HTML, CSS e JS) |
| Deployment `k8s-dash` | 3 réplicas do `nginx:alpine`, com Downward API e readinessProbe |
| Service `k8s-dash` | Endereço fixo que distribui o tráfego entre os pods |
| Ingress `k8s-dash` | Expõe o painel pelo Traefik (ingress padrão do k3s) |

Nenhuma imagem customizada: o conteúdo entra pelos ConfigMaps.

## Pré-requisitos

- Docker
- [k3d](https://github.com/k3d-io/k3d)
- kubectl

## Como rodar

Crie o cluster mapeando a porta 8081 para o load balancer:

```bash
k3d cluster create lab --servers 1 --agents 2 -p "8081:80@loadbalancer"
```

Aplique o manifesto direto do GitHub:

```bash
kubectl apply -f https://raw.githubusercontent.com/HenriqueLorente/k8s-dash/main/k8s-dash.yaml
kubectl get pods -l app=k8s-dash -o wide
```

Abra **http://localhost:8081** no navegador.

Teste pelo terminal:

```bash
for i in $(seq 6); do curl -s localhost:8081/api/info; echo; done
```

## Experimentos

```bash
# Self-healing: apague um pod e veja outro nascer no painel
kubectl delete pod <nome-do-pod>

# Escala: novas barras aparecem no gráfico
kubectl scale deploy k8s-dash --replicas=6
kubectl scale deploy k8s-dash --replicas=2

# Rolling update sem downtime: o contador de falhas fica em zero
kubectl rollout restart deploy k8s-dash
kubectl rollout status deploy k8s-dash
```

Dica: clique em **Zerar** no painel antes de cada experimento.

## Remover

```bash
kubectl delete -f https://raw.githubusercontent.com/HenriqueLorente/k8s-dash/main/k8s-dash.yaml
```
