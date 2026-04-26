[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/6QACnjim)
[![Open in Codespaces](https://classroom.github.com/assets/launch-codespace-2972f46106e565e64193e422d61a12cf1da4916b45550586e14ef0a7c637dd04.svg)](https://classroom.github.com/open-in-codespaces?assignment_repo_id=23667484)
# SESION DE LABORATORIO N° 02: Consumiendo datos de una base de datos Microsoft SQL Server

Nombre: Usher Damiron Halanocca Rojas

## OBJETIVOS
  * Comprender el funcionamiento de una aplicación que consume una base de datos relacional contenerizada.

## REQUERIMIENTOS
  * Conocimientos: 
    - Conocimientos básicos de administración de base de datos Microsoft SQL Server.
    - Conocimientos básicos de SQL.
    - Conocimientos shell y comandos en modo terminal.
  * Hardware:
    - Virtualization activada en el BIOS.
    - CPU SLAT-capable feature.
    - Al menos 4GB de RAM.
  * Software:
    - Windows 10 64bit: Pro, Enterprise o Education (1607 Anniversary Update, Build 14393 o Superior)
    - Docker Desktop 
    - Powershell versión 7.x
    - .Net 6 o superior

## CONSIDERACIONES INICIALES
  * Clonar el repositorio mediante git para tener los recursos necesaarios
  * Verificar que otro servicio no este utilizando el puerto 1433. Desactivarlo si existiese.

## DESARROLLO
1. Iniciar la aplicación Docker Desktop:  <img width="1276" height="615" alt="docker run" src="https://github.com/user-attachments/assets/0518ec37-662d-4e72-b8a8-89869364f793" />

2. Iniciar la aplicación Powershell o Windows Terminal en modo administrador  <img width="940" height="215" alt="powershell admin" src="https://github.com/user-attachments/assets/204af05c-a3f3-4fda-a29a-abfe3fb981eb" />

3. En el terminal, ubicarse en un ruta que no sea del sistema. Crear la carpeta "ServicioCliente" y dentro de esta crear la carpeta "db".  <img width="550" height="415" alt="ServicioCliente" src="https://github.com/user-attachments/assets/fb7c4aee-e562-464d-a928-dc779eed7845" />
```Bash 
mkdir -p ServicioCliente/db
cd ServicioCliente 


```
4. Iniciar Visual Studio Code desde el terminal.  <img width="981" height="465" alt="vscode " src="https://github.com/user-attachments/assets/10302bd6-70f7-4b01-b59b-fdb250cc91b7" />
```Bash
code .


```
5. En Visual Studio Code, dentro de la carpeta db, añadir el archivo clientes.sql, oon el siguiente contenido:  <img width="1135" height="830" alt="clientes" src="https://github.com/user-attachments/assets/60c5777b-3dec-47a7-9b3c-a0c28da2841b" />
```SQL
if (DB_ID(N'BD_CLIENTES') IS NOT NULL)
    DROP DATABASE BD_CLIENTES
GO
CREATE DATABASE BD_CLIENTES ON
  PRIMARY (
    NAME = N'BD_CLIENTES',
    FILENAME = N'/var/opt/mssql/data/BD_CLIENTES.mdf',
    SIZE = 50MB ,
    FILEGROWTH = 10MB
  ) LOG ON (
    NAME = N'BD_CLIENTES_log',
    FILENAME = N'/var/opt/mssql/data/BD_CLIENTES_log.ldf',
    SIZE = 10MB ,
    FILEGROWTH = 5MB
  )
GO
USE BD_CLIENTES
GO
CREATE TABLE CLIENTES (
    ID_CLIENTE INT IDENTITY NOT NULL CONSTRAINT PK_CLIENTES PRIMARY KEY,
    NOM_CLIENTE VARCHAR(100) NOT NULL
)
GO
CREATE TABLE TIPOS_DOCUMENTOS (
    ID_TIPO_DOCUMENTO TINYINT IDENTITY NOT NULL CONSTRAINT PK_TIPOS_DOCUMENTOS PRIMARY KEY,
    DES_TIPO_DOCUMENTO VARCHAR(50) NOT NULL
)
GO
CREATE TABLE CLIENTES_DOCUMENTOS (
    ID_CLIENTE INT NOT NULL,
    ID_TIPO_DOCUMENTO TINYINT NOT NULL,
    NUM_DOCUMENTO VARCHAR(15) NOT NULL,
    CONSTRAINT PK_CLIENTES_DOCUMENTOS PRIMARY KEY (ID_CLIENTE, ID_TIPO_DOCUMENTO),
    CONSTRAINT FK_CLIENTES_DOCUMENTOS_X_CLIENTE FOREIGN KEY (ID_CLIENTE) REFERENCES CLIENTES(ID_CLIENTE),
    CONSTRAINT FK_CLIENTES_DOCUMENTOS_X_TIPO_DOCUMENTO FOREIGN KEY (ID_TIPO_DOCUMENTO) REFERENCES TIPOS_DOCUMENTOS(ID_TIPO_DOCUMENTO)
)
GO
INSERT TIPOS_DOCUMENTOS(DES_TIPO_DOCUMENTO)
VALUES ('DNI'),('RUC')
GO


```
6. En Visual Studio Code, en en la raiz del proyecto, añadir un archivo docker-compose.yaml, con el siguiente contenido:  <img width="918" height="489" alt="docker compose" src="https://github.com/user-attachments/assets/101632d7-342a-43a0-84a1-cd110c0b6fe2" />
```YAML
version: "3.9"  # optional since v1.27.0
services:
  bd:
    image: "mcr.microsoft.com/mssql/server"
    container_name: bd_clientes
    ports:
      - "1433:1433" 
    environment:
      - ACCEPT_EULA=y
      - SA_PASSWORD=Upt.2022
    volumes:
      - ./db:/tmp

```
7. En el terminal anteriormente abierto o abrir uno en Visual Studio Code, y dentro de la carpeta ServicioCliente ejecutar el siguiente comando  <img width="969" height="210" alt="docker compose up" src="https://github.com/user-attachments/assets/9debe778-4130-42dc-b37f-b287ae4cd56c" />
```Bash  
docker-compose up -d


```
8. En el terminal, esperar un minuto que inicie correctamente el contenedor y ejecutar el siguiente comando:  <img width="957" height="319" alt="docker exe" src="https://github.com/user-attachments/assets/0dd20942-299f-4c2d-a85d-da41901e8b7b" />
```Bash
docker exec -it bd_clientes /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P Upt.2022 -i /tmp/clientes.sql

```
9. En el terminal, seguidamente proceder a instalar las siguientes herramientas de .Net:  <img width="793" height="227" alt="dotnet" src="https://github.com/user-attachments/assets/969983fa-54cc-4b27-bb1f-7fcb59a905fc" />
```Bash
dotnet tool install -g dotnet-ef
dotnet tool install -g dotnet-aspnet-codegenerator


```
10. En el terminal, proceder a crear la API para exponer la data ejecutada anteriormente:  <img width="818" height="353" alt="ClienteAPI" src="https://github.com/user-attachments/assets/4f1919fd-87af-4115-8180-5714df268814" />
```Bash
dotnet new webapi -o ClienteAPI
cd ClienteAPI


```
11. En el terminal, Ahora agregar las librerias que se necesitara en la aplicación con el siguiente comando  <img width="946" height="503" alt="librerias" src="https://github.com/user-attachments/assets/725f2c4c-27f1-4aa3-9125-825fa0a6d837" />
```Bash
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design


```
12. En Visual Studio Code, editar el archivo appsetting.json, que se encuentra en el proyecto ClienteAPI, y adicionar lo siguiente despues de la apertura de la primera llave.  <img width="1054" height="525" alt="json" src="https://github.com/user-attachments/assets/16e89b17-5c89-4e5e-a1e8-aa5a70849b33" />
```JSON
  "ConnectionStrings": {
    "ClienteDB": "Server=(local);Database=BD_CLIENTES;User Id=sa;Password=Upt.2022;TrustServerCertificate=true"
  },


```
13. En el Terminal, ubicarse en la carpeta ClienteAPI, ejecutar el siguiente comando para importar las estructuras de datos de la base de datos. (Tener en cuenta las consideraciones iniciales del laboratorio)  <img width="961" height="469" alt="BdClientesContext " src="https://github.com/user-attachments/assets/ce362bce-7485-49ce-a550-2157313fdf01" />

```Bash
dotnet ef dbcontext scaffold "Name=ConnectionStrings:ClienteDB" Microsoft.EntityFrameworkCore.SqlServer --context-dir Data --output-dir Models --force

```
14. En Visual Studio Code, modificar la clase creada BdClientesContext que se encuentra en el proyecto ClienteAPI en la carpeta Data. Buscar:  ![Uploading BdClientesContext .png…]()  <img width="1047" height="719" alt="modify" src="https://github.com/user-attachments/assets/0248ee66-88c0-476e-aef6-8d03d3b5ff19" />

`Name=ConnectionStrings:ClienteDB` y reemplazar por `ClienteDB`
    
15. En el terminal, ubicarse en la carpeta ClienteAPI, ejecutar el siguiente comando para generar el Controlador que expondra los datos mediante el API.  <img width="971" height="373" alt="controllers" src="https://github.com/user-attachments/assets/b977655d-f0a1-45fe-bf3c-a1563cabd134" />

```Bash
dotnet aspnet-codegenerator controller -name TiposDocumentosController -async -api -m TiposDocumento -dc BdClientesContext -outDir Controllers
```
16. En Visual Studio Code, modificar el archivo program.cs, al inicio del archivo agregar  <img width="1023" height="673" alt="import using" src="https://github.com/user-attachments/assets/135f0c8c-650b-4b29-b461-9ae98068296c" />

```C#
using ClienteAPI.Data;
using Microsoft.EntityFrameworkCore;
```
17. En Visual Studio Code, modificar el archivo program.cs, debajo de la linea que indica `// Add services to the container`. <img width="1042" height="366" alt="config dbcontext" src="https://github.com/user-attachments/assets/298113fe-9ca3-4da0-b2b8-34d4bf038d39" />

```C#
builder.Services.AddDbContext<BdClientesContext>(opt => opt.UseSqlServer(builder.Configuration.GetConnectionString("ClienteDB")));
```
18. Nuevamente en el terminal, ubicarse en la carpeta ClienteAPI. Ejecutar el comando `dotnet build` para compilar la aplicación y verificar que se esta construyendo correctamente.  <img width="954" height="309" alt="dotnet build" src="https://github.com/user-attachments/assets/83cb8dfe-ead9-4030-b7ef-2517c8aa751b" />


19. (Opcional) en el terminal, ubicarse en la carpeta ClienteAPI, ejecutar el comando `dotnet run` para iniciar la aplicación. Anotar el numero de puerto que aparecera: Now listening on: http://localhost:5280. Abrir un navegador de internet e ingresar la url: http://localhost:5280/swagger

20. En Visual Studio Code, para completar crear el archivo Dockerfile dentro del proyecto ClienteAPI, con el siguiente contenido.  <img width="1087" height="570" alt="dockerfile" src="https://github.com/user-attachments/assets/e5a66916-a126-45bc-86d8-09cb43f13828" />

```YAML
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY ["ClienteAPI.csproj", "."]
RUN dotnet restore "./ClienteAPI.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "ClienteAPI.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "ClienteAPI.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "ClienteAPI.dll"]
```
21. En Visual Studio Code, modificar y completar el archivo docker-compose.yaml con el siguiente contenido al final del archivo.   <img width="1077" height="582" alt="modify compose" src="https://github.com/user-attachments/assets/0eb94ee5-7154-48b1-b12c-2e6e1db36a7d" />

```YAML
  api:
    build: ClienteAPI/. # build the Docker image 
    container_name: api_clientes
    ports:
      - "5000:80"
    environment:
      - ConnectionStrings__ClienteDB=Server=bd_clientes;Database=BD_CLIENTES;User Id=sa;Password=Upt.2022;TrustServerCertificate=true
    depends_on:
      - bd
```
22. En Visual Studio Code, modificar el archivo program.cs, comentar las siguientes lineas.  <img width="1039" height="507" alt="modify program" src="https://github.com/user-attachments/assets/fb216ca1-90ce-4707-99d7-e2f9ee8a8eef" />

```C#
// if (app.Environment.IsDevelopment() )
// {
    app.UseSwagger();
    app.UseSwaggerUI();
//}
```
23. Nuevamente en el terminal, ubicarse en la carpeta ClienteAPI. Ejecutar el comando `dotnet build` para compilar la aplicación y verificar que se esta construyendo correctamente. <img width="957" height="270" alt="dotnet 2" src="https://github.com/user-attachments/assets/a2ac5e97-380f-4b5f-824d-aea63cde25d4" />
 

24. En el terminal, ubicarse en la carpeta ServicioCliente y ejecutar el siguiente comando  <img width="908" height="548" alt="levantar docker" src="https://github.com/user-attachments/assets/cac8037b-d0fa-4d55-a3b8-bc6fdf84bb2c" />

```Bash
docker-compose up -d
```
25. Abrir un navegador de internet e ingresar la url: http://localhost:5000/swagger, ejecutar una peticion GET para TiposDocumentos, la cual deberá devolver lo siguiente:

![image](https://github.com/UPT-FAING-EPIS/SI783_BDII/assets/10199939/be43e388-7c3b-4284-abed-e0efc83d4e5b)


---
## Actividades Encargadas
1. Escribir el código necesrio para completar los controladores para las clases Cliente y ClientesDocumento. <img width="1121" height="698" alt="activity" src="https://github.com/user-attachments/assets/c52aa7a6-e056-4947-b2bf-3a2fd7f0ccb6" />


