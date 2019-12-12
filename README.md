# WebAPI com ASPNet Core 3.1 e Docker

Docker se tornou uma palavra muito repetida nas entrevistas para cargos de engenharia de software. Experiência profissional, ter um caso de estudo ou conhecer e entender do o que se trata sempre estará no escopo de perguntas que os entrevistadores farão.

Meu objetivo aqui é uma introdução neste assunto tão interessante. Para isto, utilizarei a última versão do framework ASPNet Core 3.1 juntamente com Docker e executaremos o projeto localmente.

Não vou abordar o desenvolvimento em .Net Core 3.1, para isto utilizarei o que o template vai nos gerar.  Espero também que já tenha instalado em sua máquina a última versão do SDK do .Net Core 3.1. Durante o tutorial, utilizarei a IDE Visual Studio Code.


## Criando um projecto .Net Core por CLI

Nosso projeto será criado a partir do CLI utilizando comandos dotnet. Para isso, navegue até um diretório onde deseja criar seu projeto. No meu caso fica em c:/github. Execute o comando:
```bash
 dotnet new webapi -n TesteComDocker
 ```

Você deve ter conseguido criar um projeto com o nome TesteComDocker, criando uma pasta na raiz. Poderá verificar que a versão utilizada é a 3.

![image 1](https://raw.githubusercontent.com/j-ew-s/docker-study-api-core/master/images/Terminal1.png)

Navegamos até a raiz do nosso projeto. Execute o comando :
```bash
 cd TesteComDocker
 ```

Abra-o no editor que preferir. No caso utilizarei o Visual Studio Code (VS Code). Execute o comando:
```bash
 code .
 ```

Legal, já estamos indo bem com pouco que avançamos. Porém, é bom que façam as instalações da extensão C# e Docker (esta má circulada na imagem).

![image 2](https://raw.githubusercontent.com/j-ew-s/docker-study-api-core/master/images/terminal2.png)


![image 3](https://raw.githubusercontent.com/j-ew-s/docker-study-api-core/master/images/Terminal3.png)


Agora execute o projeto : CTRL + F5

Navegue até https://localhost:5001/WeatherForecast  e desfrute do resultado :) Está tudo OK!


## Configurando Dockerfile

Agora que temos nosso projeto funcionando (não esqueça de pará-lo), vamos adicionar o Dockerfile, que vai ajudar a criar uma imagem do nosso projeto para depois executá-la. 

No lado direito, na Solution Explorer - para os habituados com visual studio - adicione um novo arquivo chamado Dockerfile. Apenas isto, sem extensão e com o D maiúsculo. 

Adicione as seguintes linhas :

```shell
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build
WORKDIR /app
 
COPY *.csproj ./
RUN dotnet restore 
 
COPY . ./
RUN dotnet publish  -c Release -o out
 
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1
WORKDIR /app
EXPOSE 80
COPY --from=build /app/out .
ENTRYPOINT ["dotnet", "TesteComDocker.dll"]

```

As palavras reservadas (instruções) do Dockerfile estão em azul, portanto evite criar coisas que tenham este nome.

Reparem que agora no projeto de vocês tem este arquivo com o símbolo do Docker.


![image 4](https://raw.githubusercontent.com/j-ew-s/docker-study-api-core/master/images/Terminal4.png)


## Dockerfile em partes

#### FROM
```bash
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build
```

Nosso arquivo Dockerfile inicia com esta instrução indicando que utilizaremos a imagem do Microsoft Dotnet Core SDK  na versão 3.1 para construirmos a nossa, ou seja é uma imagem base. 

Nós  utilizaremos o SDK neste primeiro momento e não o runtime. Isto se deve pelo motivo de que nós vamos compilar a aplicação no Docker durante o build da imagem. Isto nos dá mais versatilidade já que a máquina em que vamos subir o Docker não precisará ter o SDK instalado, ficando muito mais simples.

Você pode verificar mais versões, como as anteriores ou do Runtime neste link.


#### WORKDIR
```bash
WORKDIR /app
```

WORKDIR indica para onde nós vamos direcionar todo o resultado das ações em sequência. Ou seja, nossa pasta de saída, resultado. Caso não exista uma pasta app ela será criada.

#### COPY
```bash
COPY *.csproj ./
```

A instrução é bem explicativa mesmo. Neste passo copiamos todos (indicado por asterisco) os arquivos com extensão csproj para a indicada na instrução WORKDIR app.


#### RUN
```bash
RUN dotnet restore
```



Simplesmente executamos o restore do csproj que fora copiada para nossa raiz. Todo o resultado da execução do restore do csproj ficará na pasta app. Isto é o suficiente para termos todos os Nuget packages atualizados.

#### COPY & RUN
```bash
COPY . ./
RUN dotnet publish  -c Release -o out
```


Agora copiamos todos os arquivos restantes no nosso projeto (arquivos .cs, .json e etc)  para a pasta app. Isto garante que o que precisava do SDK, para executar foi passado e agora, nenhum outro conflito pode existir. 

Já na sequência executamos um publish ‘dotnet publish’. A instrução publish do dotnet vai simplesmente gerar um projeto web utilizando a configuração Release ‘-c Release’ e colocar o resultado de tudo isto na pasta out ´-o out’ ficando em ‘app/out’.

#### FROM e WORKDIR novamente.
```bash
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1
```

Para os olhares mais atentos, para além de termos mais uma vez o FROM temos uma mudança no parâmetro. Antes tínhamos o sdk:3:1 agora aspnet:3.1 e não tem o alias AS build.

Quando temos mais de um FROM os FROMs anteriores ou as imagens anteriores são temporárias e que foram utilizadas em apenas um momento durante o processo. Este processo chama Multi-Stage Builds.

No nosso caso, só utilizamos a primeira imagem, que é um SDK, para poder compilar nossa aplicação. Mais nada. Agora, vamos seguir com um runtime, o que já é o suficiente para nossa aplicação ser executada, o que diminui também o tamanho do resultado.

Com isto, vamos dizer que, para esta imagem também, teremos uma pasta principal que chama app por meio da instrução WORKDIR. Esta app não é a mesma da imagem anterior, lembre-se, são imagens diferentes.

#### EXPOSE

Neste comando estamos a dizer pro docker que nosso projeto será exposto na porta 80, assim por ela iremos consumir o serviço. Calma, vamos chegar no mapeamento desta porta.

#### COPY com --from

Nós chegamos a compilar utilizando a imagem que tem o alias build e o resultado dele está na pasta out dentro de app ‘app/out’. Nós podemos pegar o resultado do run desta imagem utilizando o parâmetro  --from=build.  E como endereço final utilizamos a pasta out dentro da nossa atual app (indicada no WORKDIR anteriormente).

#### ENTRYPOINT

O Entrypoint indica o comando que será executado (ou comandos) quando nosso container iniciar.  Assim, o que temos é dotnet e a DLL do nosso projeto, executando então nosso projeto.

## Build 

Uma vez que já criaste o Dockerfile, temos que fazer um build.

Antes disso, lembra que comentei sobre a instalação da extensão do Docker no Visual Code, pois então, fique de olho nele. 

As áreas IMAGES e CONTAINERS, vão mudar respectivamente conforme vamos executando comandos para build e run.  No caso de vocês deve ter menos itens em cada uma das áreas, mas isto não é importante. Agora, foque que em IMAGES não se encontra o TesteComDocker.

Vamos fazer o build da nossa imagem. Para isto no terminal, navegue até a raiz do seu projeto. No meu caso : C:\github\TesteComDocker>

execute o comando : docker build -t testecomdocker . 

Docker build indica que faremos um build de uma imagem. O parâmetro -t (--tag) serve para darmos uma tag para a imagem, assim, resolvi nomear testecomdocker (tem que ser lowercase), para facilitar. Mas pode ser qualquer coisa. O ponto final, depois de testecomdocker indica que o diretório que o comando está a ser executado é o nosso contexto de build, ou seja, onde temos nosso csproj (lembre-se, dos passos no Dockerfile).

Pode dar enter. :)

Você pode acompanhar pelo terminal que todos os passos foram executados e que o resultado deu sucesso, indicando o número do build e o tagged como latest. Pode ser que apareça algum warning, mas isto não será coberto por este tutorial.


![image 5](https://raw.githubusercontent.com/j-ew-s/docker-study-api-core/master/images/Terminal5.png)


Agora repare que nossa imagem já fora criada e pode ser identificada :

![image 6](https://raw.githubusercontent.com/j-ew-s/docker-study-api-core/master/images/Terminal6.png)

Você também pode  ver pela linha de comando:

```bash
docker images
```

![image 7](https://raw.githubusercontent.com/j-ew-s/docker-study-api-core/master/images/Terminal7.png)

Pode ser que existam uma série de outras imagens abaixo e algumas com <none>, mas trataremos disso depois. 

## Run

Vamos botar pra funcionar. No mesmo terminal (na pasta raiz) vamos fazer o RUN do docker:

``` bash
docker run -p 8086:80 testecomdocker
``` 

O Docker run vai executar nosso container. o -p 8086:80 vai expor a porta 8086 localmente representando a porta 80 do docker, aquela que indicamos logo acima. Por fim, dizemos qual imagem vamos executar, no caso a testecomdocker.

Espero que tenham recebido uma mensagem parecida com esta 

![image 8](https://raw.githubusercontent.com/j-ew-s/docker-study-api-core/master/images/Terminal8.png)

Com isto também podemos ver que no nossos containers agora temos um novo item o testecomdocker 


![image 9](https://raw.githubusercontent.com/j-ew-s/docker-study-api-core/master/images/Terminal9.png)


## Teste

Agora pode testar acessar sua aplicação no localhost utilizando a porta 8086

http://localhost:8086/weatherforecast



## Links 

inspirado em : 
https://www.softwaredeveloper.blog/multi-project-dotnet-core-solution-in-docker-image#dockerfile-from-instruction
https://www.youtube.com/watch?v=f0lMGPB10bM 
https://docs.docker.com/engine/reference/commandline/stop/
https://stackoverflow.com/questions/48813286/stop-all-docker-containers-at-once-on-windows
https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes

```bash
docker rmi -f $(docker images -f "dangling=true" -q)
```

https://docs.google.com/document/d/1xAHwOkNyDVfkH7eQR8612aWeaY-OnKNHAMZT06VzMwI/edit?usp=sharing








