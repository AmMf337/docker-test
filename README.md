# Getting Started with Create React App

This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).
hi

## Available Scripts

In the project directory, you can run:

### `npm start`

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in your browser.

The page will reload when you make changes.\
You may also see any lint errors in the console.

### `npm test`

Launches the test runner in the interactive watch mode.\
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `npm run build`

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

### `npm run eject`

**Note: this is a one-way operation. Once you `eject`, you can't go back!**

If you aren't satisfied with the build tool and configuration choices, you can `eject` at any time. This command will remove the single build dependency from your project.

Instead, it will copy all the configuration files and the transitive dependencies (webpack, Babel, ESLint, etc) right into your project so you have full control over them. All of the commands except `eject` will still work, but they will point to the copied scripts so you can tweak them. At this point you're on your own.

You don't have to ever use `eject`. The curated feature set is suitable for small and middle deployments, and you shouldn't feel obligated to use this feature. However we understand that this tool wouldn't be useful if you couldn't customize it when you are ready for it.

## Learn More

You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started).

To learn React, check out the [React documentation](https://reactjs.org/).

### Code Splitting

This section has moved here: [https://facebook.github.io/create-react-app/docs/code-splitting](https://facebook.github.io/create-react-app/docs/code-splitting)

### Analyzing the Bundle Size

This section has moved here: [https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size](https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size)

### Making a Progressive Web App

This section has moved here: [https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app](https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app)

### Advanced Configuration

This section has moved here: [https://facebook.github.io/create-react-app/docs/advanced-configuration](https://facebook.github.io/create-react-app/docs/advanced-configuration)

### Deployment

This section has moved here: [https://facebook.github.io/create-react-app/docs/deployment](https://facebook.github.io/create-react-app/docs/deployment)

### `npm run build` fails to minify

This section has moved here: [https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify](https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify)

# Solución
### Dockerfile
Se indica la imagen adecuada para la aplicación y se marca a la etapa con "AS build":

<pre> 
FROM node:latest AS build
 </pre>

Se indica el directorio de la app y se copia el contenido del json con la configuración y las dependencias:

<pre> dockerfile WORKDIR /app COPY package*.json ./ </pre>

Se instalan las dependencias establecidas en el archivo package.json ,se copia el código de la app y se ejecuta el script en el json:

<pre>
RUN npm install
COPY . .
RUN npm run build
  </pre>

Se llama una imagen sw servidor web en la que se publicará el html estático:

<pre>
FROM nginx:alpine
 </pre>

Se elimina cualqiuier archivo html previo, se copia el archivo de html de la página, se expone el puerto 80 de http y finalmente se agrega un comando en cmd para correr nginx en primer plano en el contenedor:

<pre> 
RUN rm -rf /usr/share/nginx/html/*

COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
</pre>


### Github Worflow

Nombre del workflow:

<pre>
name: Build and Push Docker Image
</pre>
 
Descencadenador del workflow,en este caso se le indica que el workflow se dispara cuando se hace push a la rama main:

<pre>
on:
  push:
    branches:
      - main
</pre>

El campo **jobs** son las acciones que se realizaran en el workflow:

Aqui se hace el build de la imagen y se elige en que sistema operativo se realizara dicho build:

<pre>
jobs:
  build:
    runs-on: ubuntu-latest
</pre>

Se da comienzo a los pasos del workflow en el campo **steps**, cada paso tiene un nombre y las acciones a ejecutar, el primero llamado "checkout repository" hace uso de un action de github que indica al runner de github que copie el código del repositorio y usara la versión 4 de dicha acción:

<pre>
    steps:
   
      - name: Checkout repository
        uses: actions/checkout@v4
</pre>

Llama a otra action oficial de github que configura el build de la imagen en el runner:

<pre>  
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
</pre>

LLama a una acción que se encarga de realizar el login en dockerhub con las variables establecidas para el repositorio, para establecer estas variables se debe entrar a configuraciones, entrar a la pestaña "secretos y variables" y crear nuevas variables, estas ultimas deben ser nombradas exactamente de la manera en la que se referencian aqui:

<pre>     
      - name: Log in to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
</pre>

Finalmente se realiza la subida de la imagen a dockerhub a la cuenta con la que se inicio sesión con las credenciales:

<pre>   
      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/rick-morty:latest
 </pre>
