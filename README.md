# daes-helm-servicemesh

Este projeto de Service Mesh utiliza o projeto `spring-app-daes` como aplicação Java Spring Boot.
A imagem da aplicação é gerada a partir do `Dockerfile` existente em `spring-app-daes`.

## Build e Push da Imagem (Docker Hub)

```bash
cd /home/danielgomes/workspace/service_mesh_exemplo/spring-app-daes
mvn clean package -DskipTests
docker build -t danieldggomes/springboot-app-daes:1.0.0 .
docker push danieldggomes/springboot-app-daes:1.0.0
```

## Validar, Empacotar e Publicar Helm Chart

```bash
cd /home/danielgomes/workspace/service_mesh_exemplo/spring-boot

helm lint .
helm template spring-boot . -n apps \
| kubeconform -strict -summary \
  -schema-location default \
  -schema-location 'https://raw.githubusercontent.com/datreeio/CRDs-catalog/main/{{.Group}}/{{.ResourceKind}}_{{.ResourceAPIVersion}}.json' \
  -skip Route

helm package .
helm push spring-boot-helm-chart-1.0.31.tgz oci://registry-1.docker.io/danieldggomes
```

## Relacao Entre os Charts e a Imagem da Aplicacao

- `spring-app-daes`:
  projeto Java Spring Boot que gera a imagem Docker `danieldggomes/springboot-app-daes:1.0.0`.
- `spring-boot-helm-chart`:
  chart Helm de template Kubernetes (Deployment, Service, Route, etc). Nao contem o codigo da aplicacao.
- `daes-helm-servicemesh-blueprint`:
  chart pai que compoe multiplas instancias do `spring-boot-helm-chart` e injeta via `values.yaml` qual imagem da aplicacao cada instancia deve usar.

No estado atual deste projeto:

- `Chart.yaml` define 3 dependencias do `spring-boot-helm-chart` (aliases `spring-boot-1`, `spring-boot-2`, `spring-boot-3`) via OCI no Docker Hub.
- `values.yaml` define para cada alias:
  - `image.repository: danieldggomes/springboot-app-daes`
  - `image.tag: 1.0.0`
  - `image.pullSecrets: []` (imagem publica)

## Deploy no OpenShift (oc + Helm)

```bash
cd /home/danielgomes/workspace/service_mesh_exemplo/daes-helm-servicemesh

helm registry login registry-1.docker.io -u danieldggomes
helm dependency update .
helm lint .

oc login <API_DO_OPENSHIFT>
oc project <SEU_NAMESPACE>

helm upgrade --install daes-helm-servicemesh . -n <SEU_NAMESPACE>
```
