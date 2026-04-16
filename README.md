Open vscode -> folder rivera-events
Create index.html
<!DOCTYPE html>
<html>
<head>
    <title>Rivera Events</title>
    <style>
        body {
            font-family: Arial;
            text-align: center;
            background-color: #f2f2f2;
        }
        table {
            margin: auto;
            border-collapse: collapse;
            width: 70%;
        }
        th, td {
            border: 1px solid black;
            padding: 10px;
        }
        th {
            background-color: #2196F3;
            color: white;
        }
    </style>
</head>
<body>
<h1>Rivera Events</h1>
<table>
<tr>
<th>Event Name</th>
<th>Category</th>
<th>Date</th>
</tr>
<tr>
<td>Dance Battle</td>
<td>Cultural</td>
<td>10 March</td>
</tr>
<tr>
<td>Hackathon</td>
<td>Technical</td>
<td>11 March</td>
</tr>
<tr>
<td>Music Night</td>
<td>Entertainment</td>
<td>12 March</td>
</tr>
<tr>
<td>Photography Contest</td>
<td>Creative</td>
<td>13 March</td>
</tr>
<tr>
<td>Gaming Tournament</td>
<td>Fun</td>
<td>14 March</td>
</tr>
</table>
</body>
</html>
Open in browser and check locally
 
Create repo and push to it
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/Srilaya18/rivera.git
git push -u origin main
 
Create Dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80

Create deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rivera-events
spec:
  replicas: 2
  selector:
    matchLabels:
      app: rivera-events
  template:
    metadata:
      labels:
        app: rivera-events
    spec:
      containers:
      - name: rivera-events
        image: srilayam/rivera-events:latest   # ⚠️ change username
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: rivera-service
spec:
  type: NodePort
  selector:
    app: rivera-events
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30009

Push to git again
git add .
git commit -m "Added Dockerfile and Kubernetes config"
git push
 
Install Kubernetes CLI plugin, Docker and Docker Pipeline 
Steps 
Manage Jenkins - Credentials - (System) -Global credentials (unrestricted) - Add Credentials 
Fill like this: 
Kind: Username with password 
Scope: Global 
Username: srilayam
Password: sridocker@123
ID: dockerhub-creds 
Description: DockerHub Login

Create pipeline and add script
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "srilayam/rivera-events:latest"  //add docker username
    } 
    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/Srilaya18/rivera.git'   //repo link change
            }
        }
        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE% .'
            }
        }
        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    bat 'docker login -u %USER% -p %PASS%'
                    bat 'docker push %DOCKER_IMAGE%'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                bat 'kubectl apply -f deployment.yaml --validate=false || exit 0'
            }
        }
    }
    post {
        success { 
            echo 'CI/CD Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check logs.'
        }
    }
}
 
In terminal
Check pods- kubectl get pods        (must get 1/1 then only working)
Check service- kubectl get svc 
 
Website link is- http://localhost:30009
