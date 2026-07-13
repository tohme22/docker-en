---
title: "Exercise"
description: "docker"
draft: false
weight: 3
---
### Exercise

1. Create the folder `www`

2. Move into the folder `www` and create a `index.html`. 

3. Edit the file `index.html` with some texts.

4. Download and run the **httpd (apache)** image. Name the container `my-web-site`, use the port `8080` and use the variable `"$PWD":/usr/local/apache2/htdocs` to create a **Bindmount volume** on the container local folder `httpd:2.4`.

5. Open a browser and test the website using: **http://127.0.0.1:8080**

6. Create another **httpd** container using the same Bindmount volume `"$PWD”` and the port `8090`.

7. Open a browser and test the website using: **http://127.0.0.1:8090**

8. Edit the file `index.html` and check if the content of both websites is changed automatically.




