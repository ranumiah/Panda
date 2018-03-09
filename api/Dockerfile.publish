FROM microsoft/aspnetcore:2

WORKDIR /app

COPY bin/Debug/netcoreapp2.0/publish .

# Because we have WORKDIR set we don't need to do explict path: "/app/api.dll"
ENTRYPOINT [ "dotnet", "api.dll" ]

#docker build -t firsthelpline/swagger-ui .

#docker run --rm -it -p 8080:80 firsthelpline/swagger-ui
#docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'

#http://172.25.37.27/swagger/

# firsthelpline ==> is the name of your DockerHub RepoName
# *** Docker Hub Repository Push ***
#docker login
#docker push firsthelpline/swagger-ui