# play.identity
Play Economy Identity microservice.

## Create and publish package
```powershell
$version="1.0.2"
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
```powershell
$adminPass="[ADMIN PASSWORD HERE]"
docker run -it --rm -p 5002:5002 --name identity -e MongoDbSettings__Host=mongo -e RabbitMQSettings__Host=rabbitmq -e IdentitySettings__AdminUserPassword=$adminPass --network playinfra_default play.identity:$version
```