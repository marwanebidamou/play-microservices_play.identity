# play.identity
Play Economy Identity microservice.

## Create and publish package
```powershell
$version="1.0.5"
$owner="play-microservice"
$gh_pat="[GITHUB ACCESS TOKEN HERE]"
$nuget_src_name="Play Github"
$package_name="Play.Identity.Contracts"

dotnet pack src\$package_name\ --configuration Release -p:PackageVersion=$version -p:RepositoryUrl=https://github.com/$owner/play.identity -o ..\packages

dotnet nuget push ..\packages\$package_name.$version.nupkg --api-key $gh_pat --source $nuget_src_name
```

## Build the docker image
```powershell
$env:GH_OWNER="play-microservice"
$env:GH_PAT="[GITHUB ACCESS TOKEN HERE]"
docker build --secret id=GH_OWNER --secret id=GH_PAT -t play.identity:$version .
```

## Run the docker image
### run it using local mongo & rabbitmq
```powershell
$adminPass="[ADMIN PASSWORD HERE]"
docker run -it --rm -p 5002:5002 --name identity -e MongoDbSettings__Host=mongo -e RabbitMQSettings__Host=rabbitmq -e IdentitySettings__AdminUserPassword=$adminPass --network playinfra_default play.identity:$version
```
### run it using azure cosmos db and azure service bus
```powershell
$adminPass="[ADMIN PASSWORD HERE]"
$cosmosDbConnString="[CONN STRING HERE]"
$serviceBusConnString="[CONN STRING HERE]"
docker run -it --rm -p 5002:5002 --name identity -e MongoDbSettings__ConnectionString=$cosmosDbConnString -e ServiceBusSettings__ConnectionString=$serviceBusConnString -e ServiceSettings__MessageBroker="SERVICEBUS" -e IdentitySettings__AdminUserPassword=$adminPass play.identity:$version
```

## Publishing the Docker image
```powershell
$acrname="azacreconomy"
az acr login --name $acrname
docker tag play.identity:$version "$acrname.azurecr.io/play.identity:$version"
docker push "$acrname.azurecr.io/play.identity:$version"
```

## Create the Kubernetes namespace
```powershell
$namespace="identity"
kubectl create namespace $namespace
```

## Create the Kubernetes secrets
```powershell
kubectl create secret generic identity-secrets --from-literal=cosmosdb-connectionstring=$cosmosDbConnString --from-literal=servicebus-connectionstring=$serviceBusConnString --from-literal=admin-password=$adminPass -n $namespace
```

## Create the Kubernetes pod
```powershell
kubectl apply -f .\kubernetes\identity.yaml -n $namespace


#to check the pods run the command:
kubectl get pods -n $namespace

#to check the pod log
kubectl logs identity-deployment-xxxxx -n $namespace

#to get pod description
 kubectl describe pod identity-deployment-xxxx -n $namespace
kubectl

#to list services
kubectl get services -n $namespace

#get all events in a namespace
kubectl get events -n $namespace
```