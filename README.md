# Example microservice app

This is an example of web application comprising of several components communicating to each other. In other words, this is an example of microservice app. Why is it better than many other examples? Well, because these microservices are written in different languages. This approach gives you flexibility for running experiments in polyglot environment.

The app itself is a simple TODO app that additionally authenticates users. I planned to add some admin functionality, but decided to cut the scope and add it later if needed.

## Components

1. [Frontend](/frontend) part is a Javascript application, provides UI. Created with [VueJS](http://vuejs.org)
2. [Auth API](/auth-api) is written in Go and provides authorization functionality. Generates JWT tokens to be used with other APIs.
3. [TODOs API](/todos-api) is written with NodeJS, provides CRUD functionality ove user's todo records. Also, it logs "create" and "delete" operations to Redis queue, so they can be later processed by [Log Message Processor](/log-message-processor).
4. [Users API](/users-api) is a Spring Boot project written in Java. Provides user profiles. Does not provide full CRUD for simplicity, just getting a single user and all users.
5. [Log Message Processor](/log-message-processor) is a very short queue processor written in Python. It's sole purpose is to read messages from Redis queue and print them to stdout
6. [Zipkin](https://zipkin.io). Optional 3rd party system that aggregates traces produced by other components.

Take a look at the components diagram that describes them and their interactions.
![microservice-app-example](https://user-images.githubusercontent.com/1905821/34918427-a931d84e-f952-11e7-85a0-ace34a2e8edb.png)

## Use cases

- Evaluate various instruments (monitoring, tracing, you name it): how easy they integrate, do they have any bugs with different languages, etc.


## Pipeline de Jenkins
Para que funcione compleamente el pipeline de Jenkins tenemos que extraer el token para poder subir imagenes al registry:
Debemos coger el token del secreto llamado pipeline-token-xxx  en el namespace donde se valla a subir la imagen, en este caso laboratorio.

```Bash
oc get secret -n laboratorio $(oc get secret -n laboratorio | grep pipeline-token | head -1 | awk '{print $1}') -o jsonpath="{.data.token}" | base64 -d
```

Guardamos el token como secreto en Jenkins