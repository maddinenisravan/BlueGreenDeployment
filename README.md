Cluster:

* main.tf: it's the main configuration file. It typically defines the primary infrastructure resources, such as virtual machines, networks, or databases, to be deployed

output.tf:  This is Terraform file, this one is used to define output variables(like an IP address or a DNS)
variable.tf:  Terraform file. It's where all the input variables for the Terraform configuration .

2. Config:  It contains application configuration information

Dockerfile: This is a text file that contains instructions for building a Docker image.
Jenkins:  Jenkinsfile: This file defines a Jenkins pipeline, which is a series of automated steps for building, testing, and deploying software. By storing this file in your project's code.
mvnw :  They allow a project to be built with a specific version of Apache Maven without requiring the user to install it on their system 
mvn.cmd: the equivalent batch file for Windows.
A pom.xml : file is the Project Object Model configuration file used in Maven projects. It manages project build, dependencies, and plugins.

3. Deployment: It contain .yml files 
app-deployment-blue.yml → Deploys the Blue version of the app (old/stable release).
app-deployment-green.yml → Deploys the Green version of the app (new release).
bankapp-service.yml → Service that routes traffic → can point to Blue or Green.
mysql-ds.yml → Deploys MySQL database with storage for the app.

4.SRC:

config       → app settings (security, swagger, db configs)
controller   → REST APIs (handles requests)
model        → entities / DTOs (data objects)
repository   → database layer (CRUD using JPA)
service      → business logic (called by controllers)

5. test/java/com/example/bankapp

 To write automated tests for your application.







