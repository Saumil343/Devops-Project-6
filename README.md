# Devops-Project-6
In this project i made a ci/cd pipeline using jenkins and deployed a web app to apache server using ansible and docker.
Tools Used :

- Git

- Jenkins

- Ansible

- Aws Ec2

- Docker Container 

- AWS RDS

- <b> Work Flow </b>
- ![178779349-933a6fab-73a2-43e8-b6fb-1970e3990677 (1) drawio](https://user-images.githubusercontent.com/53990452/186216844-8589464f-3c8f-460b-b517-ea0354c2074e.png)

- <b> Steps Involved </b>

1) Creating Simple webhook CI on jenkins for github
2) Using a test repo with node hello world program which also includes ansible playbook,ansible host file and Dockerfile , Link to repo used in project : https://github.com/Saumil343/test5.git (master branch)
3) Creating a ansible playbook which runs on webserver ec2 instance and creates a docker image and runs it.
4) AWS RDS Configure and Connect



1) Creating simple webhook CI
-  Go to Dashboard
- create new item and give it a name and select Free Style Project
- Go to general section and scroll to Source Code Management Section and select git and add your git repo link of the project to be deployed.
- EXAMPLE
- ![image](https://user-images.githubusercontent.com/53990452/178780219-04f794b8-3200-47f4-b398-980b1c7cf199.png)
- Scroll and add build triggers and select GitHub hook trigger for GITScm polling.
- EXAMPLE
- ![image](https://user-images.githubusercontent.com/53990452/178780361-ef6fe25c-e0d2-40cb-841e-e7e7c14db480.png)
- Scroll and add POST BUILD ACTION and give a name of Continious deployment job[yet to be created]
- Example
![image](https://user-images.githubusercontent.com/53990452/178780466-c1961ce5-dd09-4277-89b1-da0232611e66.png)

2) Creating a deployment job
- For this example to run we need to have a git repo with the required files that are 1)ansible Playbook 2) Inverntory to run 3) Dockerfile in the git repo of our project to be deployed

- Now go to jenkins dashboard

- and create a new item -> give it a name and select pipeline

- scroll to bottom and create a pipeline

- for the ease we can use the syntax generator like:

- first step in the pipeline would be accessing the git repo 

- commands to which can be generated by the syntax generator like:

- ![image](https://user-images.githubusercontent.com/53990452/178781312-d968016e-2110-4bb9-8971-605b429969e0.png)

- once this is done we can copy that to our pipeline code

- second stage in the pipeline would be to select and run the ansible playbook on our host machine

- Again go to syntax generator and select ansible invoke playbook to generate it's script

- Fill the details there, like the username,password of the machine, name of our playbook file (whcih is in git repo),Name/path of our inventory file(whcih is in our git repo) which can be seen as follows:

 - ![image](https://user-images.githubusercontent.com/53990452/178781535-3bf9d44d-119e-476d-8e73-1ca47e20b742.png)

- add ssh credentials as per you system (give the credential of the system which is specified in ansible inverntory/hosts file):

- EXAMPLE

- ![image](https://user-images.githubusercontent.com/53990452/178781655-f8f8c39a-f88e-439b-85d9-da7605caaf62.png)

- once all this is done generate the script and copy it :

- EXAMPLE:

- ![image](https://user-images.githubusercontent.com/53990452/178781751-ed9de2bd-90b7-49c0-8a4c-798d83523bcb.png)

- Now get back to the pipeline we created and we will add steps we need it to follow :

- Complete Pipeline Code : (change the generated scripts in your case as per the req)

```
pipeline{
   agent any
   stages{
       stage("git checks"){
           steps{
               git branch: 'main', url: 'https://github.com/Saumil343/test5.git'
           }
       }
       stage("Ansible playbook"){
           steps{
               ansiblePlaybook credentialsId: '91fda46f-c50f-40ce-bab7-8a965994c32b', disableHostKeyChecking: true, installation: 'ansible1', inventory: 'hosts', playbook: 'aws-php.yml'
           }
       }
   }
}

```

3) Trigger a build
now as soon as someting has changed in our git repo the change will be detected by our first pipeline which would trigger the ansible conitinious deployment pipeline which would run the playbooks

4) Docker file :
```
FROM amazonlinux
RUN yum install httpd php-5.4.16 php-mysqlnd -y
RUN systemctl enable httpd
COPY . /var/www/html
CMD ["/usr/sbin/httpd","-D","FOREGROUND"]
EXPOSE 80
```

5) Ansible playbook which pulls the latest code and kills, creates the container 
```
---
- hosts: 172.31.36.60
  tasks:
    - name: cleaning files
      shell: rm -rf ~/test5

    - name: pull
      shell: git clone https://github.com/Saumil343/test5.git

    - name: docker kill
      shell: docker stop phpapp4 || true && docker rm phpapp4 || true
    
    - name: docker build
      shell: docker build -t saumil343/demoapp5 ~/test5/
      
    - name: docker run 
      shell: docker run -it -d --name phpapp4 -p 3200:80 saumil343/demoapp5


```

6) AWS RDS INSTANCE 
- i used mysql database and created a t3.micro rds instande along with mysql versin 5.6
 ![image](https://user-images.githubusercontent.com/53990452/186218602-bb7d1a8a-2769-4a03-b4a9-0682ca69e19b.png)

