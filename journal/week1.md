# Week 1 — App Containerization

## Run the flask backend
Go to the backend directory `backend-flask`
```sh
cd backend-flask
```

Set the backend and frontend url environment variables by entering the following in the terminal
```sh
export FRONTEND_URL="*"
export BACKEND_URL="*"
```

Install the backend dependencies by running this command in the terminal 
```sh
pip3 install -r requirements.txt
```

Run the backend python app
```sh
python3 -m flask run --host=0.0.0.0 --port=4567
```

Go the ports tab( by the terminal tab) and make sure the port defined above is unlocked

![image](../_docs/assets/ports-tab.png)

Click the circle/globe/internet icon (2 icons to the right of unlock), to get the URL of the backend. It should open in a new tab.

In the new tab in the browser, append `/api/activities/home` to the url and press enter. You should now get a response. 


![image](../_docs/assets/response-example.png)

## Containerize the backend
We are going to put the previous steps in a Dockerfile. In the backend-flask folder, create a new file called Dockerfile and enter the following contents
```dockerfile
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```
Build the backend container using the Dockerfile

```sh
docker build -t  backend-flask ./backend-flask
```

Use this command to run the backend in the background(-d flag)
```sh
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' -d backend-flask
```

Use the command `docker ps` to see the docker process.

To stop the backgrounded container you can get the container id from the `docker ps` command and run
```sh
docker stop <container id>
```

You can also delete the docker image with the following command

```sh
docker image rm backend-flask --force
```
## Run the frontend
Go to the frontend-react-js directory
```sh
cd frontend-react-js
```

Install the node packages by running this command
```sh
npm i
```

Run the frontend
```sh
npm start
```

You can get the URL for the frontend from the ports tab. 
![image](../_docs/assets/frontend-port.png)


The frontend should look like this
![image](../_docs/assets/frontend-example.png)

## Containerize the frontend
Create a file in frontend-react-js called Dockerfile with the following contents

```dockerfile
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```

Build the container
```sh
docker build -t frontend-react-js ./frontend-react-js
```

Run the container
```sh
docker run --rm -p 3000:3000 -it -d frontend-react-js
```

Use the command `docker ps` to see the docker process.

To stop the backgrounded container you can get the container id from the `docker ps` command and run
```sh
docker stop <container id>
```

You can also delete the docker image with the following command

```sh
docker image rm backend-flask --force
```