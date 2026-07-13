---
title: "Exercise "
description: "$ docker"
draft: false
weight: 3
---
### Exercise  - Creating an image from a Dockerfile

#### Preparation

Create a folder named `dos` and create this `index.html` file in that folder.

```yaml
<html>
  <style type="text/css" media="screen">
      canvas {
          width: 800px; 
          height: 600px;
      }
  </style>
  <head>
    <title>DOS Game!</title>
    <script src="js-dos.js"></script>
  </head>
  <body>
    <canvas id="jsdos" width="800" height="600" ></canvas>
    <script>
      Dos(document.getElementById("jsdos"), {
      }).ready((fs, main) => {
        fs.extract("game.zip").then(() => {
          main(["-c", GAME_ARGS])
        });
      });
    </script>
  </body>
</html>
```

#### Dockerfile

Use the information provided below to create a Dockerfile with the appropriate Docker directives. For each step, a description and the value(s) to use are given. Identify the corresponding directive and complete the file.

1.	**Image Base:** Use the image **`node:16-alpine`** as the base image for your container.

2.	**Working Directory:** Set the main working directory of the container where all commands will be executed. Use **`site`** as the location of this directory.

3.	**Installing JS-DOS Files:** Run these three commands to download the files needed for JS-DOS using the following URLs:
    *	wget https://js-dos.com/6.22/current/js-dos.js
    *	wget https://js-dos.com/6.22/current/wdosbox.js
    *	wget https://js-dos.com/6.22/current/wdosbox.wasm.js

4.	**Installing serve**: Run this command **`npm install -g serve`** to globally install the package **`serve`**.

5.	**Set a variable**: Set the variable **`GAME_URL`** that will specify the URL of the game to download when building the image.

6.	**Adding a game**: Run this command: **`wget -O game.zip $GAME_URL`** to download the game using the previous variable and save it as **`game.zip`**.

7.	**Game Configuration**: 

    *  Set another variable **`GAME_ARGS`** to pass arguments to the game when building the image.

    *  Copy the **`index.html`** file from the local folder to the image local folder.

    *  Run this command **`sed -i s/GAME_ARGS/$GAME_ARGS/ index.html`** to replace the string in index.html **`GAME_ARGS`** with the value of the variable configured for the game arguments.

10.	**Launching the game server**: Run this command **`npx serve -l tcp://0.0.0.0:8000`** when running the container, so that the server launches the game using the **`8000`**. _Hint: You can use ENTRYPOINT ads a instruction._

#### Image creation

Build a **Docker image** using your Dockerfile with the following parameters:

*	**Game URL** : You need to specify the URL from where to download the game when building the Docker image. The URL to use is**https://archive.org/download/msdos_festival_SCORCH15/SCORCH15.ZIP**

*	**Game Arguments** : To run the game, some arguments need to be passed. In this case, you need to pass **`\"SCORCH.EXE\"`** as a game argument.

*	**Image Name**: The image you will build should be named **mycool:dosgame**.

#### Container creation

Start a Docker container based on an image you previously built with the following settings:
* **Automatic Cleanup**: Use an option that ensures that the container is automatically deleted after stopping its execution. 

* **Port Mapping** : In order to make the game accessible via a web browser on your host machine, you need to map a port on your host machine to a port in the container. The service inside the container listens on port **`8000`**. You need to map this port **`8000`** to the address **`127.0.0.1`** of your host machine.

#### Container test

Open a browser to test the new container.


