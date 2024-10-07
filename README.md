# <a name="_nyhw11cmhoti"></a>**Deployment of a Web Application with Apache, Nginx Load Balancing, and Jenkins Automation**
##
## <a name="_tqqsla9sfxx5"></a><a name="_6lg9atd1j1nm"></a>**Objective**
Task given to deploy a web application using Apache, running 4 instances of the application, with Nginx as a load balancer. The entire process is containerized using Docker and automated through Jenkins.
##
## <a name="_ephyvtsrplal"></a><a name="_h07etirn2jua"></a>**Tools Used:**
**Docker:** Used for containerisation from the GitHub repo.

**Jenkins:** Automation Tool for creating and maintaining pipeline.

**Nginx:** It’s a load-balancer used to resolve heavy traffic onto the server.

**Git Bash:** Git bash is CLI terminal which can Pull, Fetch, and Clone repos from GitHub.

**GitHub**: It’s an Distributed Version Controlling System (DVCS) where the commits and present in remote location.

**WSL:** The Windows Subsystem for Linux (WSL) is a feature of the Windows operating system that enables you to run a Linux file system, along with Linux command-line tools and GUI apps.

## <a name="_fdjitguwr48a"></a>**Steps**
### <a name="_tbdb5a5wpxrm"></a>**Step 1: Setup the Web Application with Apache and Docker**
**Clone the Web App Repository**: Open your terminal and run:

|git clone https://github.com/Sid1545/food.git|
| :- |

![](Aspose.Words.30c6633f-e347-40c4-ae3c-c28802dc4b3e.001.png)

**Create Dockerfile**: Inside the cloned repository, create a file named Dockerfile with the following content:
***dockerfile***

|FROM httpd:2.4<br>COPY . /usr/local/apache2/htdocs/|
| :- |

**Build Docker Image**: In the project folder, run the following command to build the Docker image:

|docker build -t my-apache-app .|
| :- |

![](Aspose.Words.30c6633f-e347-40c4-ae3c-c28802dc4b3e.002.png)

**Run 4 Apache Instances**: Start 4 containers, each mapped to a different port:

|docker run -dit --name app1 -p 8081:80 my-apache-app<br>docker run -dit --name app2 -p 8082:80 my-apache-app<br>docker run -dit --name app3 -p 8083:80 my-apache-app<br>docker run -dit --name app4 -p 8084:80 my-apache-app|
| :- |

![](Aspose.Words.30c6633f-e347-40c4-ae3c-c28802dc4b3e.003.png)
### <a name="_pvdex4l39hwu"></a>**Step 2: Configure Load Balancing with Nginx**
**Create Nginx Configuration**: Create a file named nginx.conf with the following content:
*nginx*

|upstream apache\_servers {<br>`    `server app1:80;<br>`    `server app2:81;<br>`    `server app3:82;<br>`    `server app4:83;<br>}<br><br>server {<br>`    `listen 80;<br>`    `location / {<br>`        `proxy\_pass http://apache\_servers;<br>`    `}<br>}|
| :- |

![](Aspose.Words.30c6633f-e347-40c4-ae3c-c28802dc4b3e.004.png)

**Run Nginx in Docker**: To avoid port conflicts, first stop any existing Nginx container:

|docker stop nginx-loadbalancer<br>docker rm nginx-loadbalancer|
| :- |

Then, run Nginx as a container:




|docker run -d -p 80:80 --name nginx-loadbalancer -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf nginx|
| :- |

### <a name="_mo4up1eu3hr6"></a>**Step 3: Automate the Setup with Jenkins**
1. **Install Docker Plugin in Jenkins**:
   1. Navigate to Manage Jenkins > Manage Plugins > Available.
   1. Search for "Docker" and install it.

**Create Jenkins Pipeline**: Create a Jenkinsfile in the project root with the following content:
Groovy -

|pipeline {<br>`    `agent any<br>`    `stages {<br>`        `stage('Build Docker Images') {<br>`            `steps {<br>`                `sh 'docker build -t my-apache-app .'<br>`            `}<br>`        `}<br>`        `stage('Deploy Apache Containers') {<br>`            `steps {<br>`                `sh '''<br>`                `docker run -dit --name app1 -p 8081:80 my-apache-app<br>`                `docker run -dit --name app2 -p 8082:80 my-apache-app<br>`                `docker run -dit --name app3 -p 8083:80 my-apache-app<br>`                `docker run -dit --name app4 -p 8084:80 my-apache-app<br>`                `'''<br>`            `}<br>`        `}<br>`        `stage('Deploy Nginx Load Balancer') {<br>`            `steps {<br>`                `sh 'docker run -d -p 80:80 --name nginx-loadbalancer -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf nginx'<br>`            `}<br>`        `}<br>`    `}<br>}|
| :- |

![](Aspose.Words.30c6633f-e347-40c4-ae3c-c28802dc4b3e.005.png)![](Aspose.Words.30c6633f-e347-40c4-ae3c-c28802dc4b3e.006.png)![](Aspose.Words.30c6633f-e347-40c4-ae3c-c28802dc4b3e.007.png)![](Aspose.Words.30c6633f-e347-40c4-ae3c-c28802dc4b3e.008.png)

The website is running on 4 different ports i.e (.80,.81,82,83)

1. **Create a Pipeline Job in Jenkins**:
   1. In Jenkins, create a new pipeline job.
   1. Link it to your Git repository.
   1. The pipeline will trigger automatically on changes. (add webhook plugin)

![](Aspose.Words.30c6633f-e347-40c4-ae3c-c28802dc4b3e.009.png)
## <a name="_2gzpuuh6ib41"></a>**Difficulties Faced**
1. **Port Conflicts**:
   1. During the setup, multiple instances of Nginx attempted to bind to port 80, leading to errors. This required careful management of existing containers and their associated ports.
1. **Container Name Conflicts**:
   1. Reusing the same container names resulted in conflicts. Removing or renaming existing containers was necessary to resolve this issue.
1. **Docker Volume Bind Issues**:
   1. Issues with binding the nginx.conf file to the Nginx container were encountered. Ensuring the configuration file was in the correct path and accessible helped mitigate this problem.
1. **Jenkins Pipeline Errors**:
   1. Initial attempts to set up the Jenkins pipeline faced errors related to Docker commands. Validating the pipeline syntax and ensuring Docker was properly installed on Jenkins resolved these issues.
1. **Accessing Services**:
   1. There were initial problems accessing the services through the browser, which required checking if the containers were running correctly and if the ports were correctly mapped.
##
## <a name="_87db2iovu41t"></a><a name="_ksguf5u8kwcc"></a>**Suggestion:**
Personally, using AWS services can ease the process of scaling with automatic responsive load balancer. Kubernetes orchestration controls the docker container and jenkins pipeline, ensuring safe migration.
## <a name="_wa2tamu75rie"></a>**Conclusion**
This documentation outlines the complete setup and configuration to deploy a web application using Apache, with Nginx as a load balancer and Jenkins for automation. Adjust configurations as needed based on your specific requirements.

**Task performed by Faizan Quazi** (<quazi.faizan14@gmail.com>)

07/10/2024
