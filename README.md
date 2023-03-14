# DevOps Portfolio

This is part of my devops portfolio!

The goal of this project is to demonstrate all the Technologies and methodologies I learned as a devops engineer.

Tools & Technologies:
* App: python flask, HTML, Bootstrap, Javascript.
* Database: Mongodb
* CI/CD: Jenkins, Kubernetes, Argocd, Helm
* Infrastructure: Terraform & AWS
* Logging: EFK stack
* Monitoring: Prometheus & Grafana

The full project consists of 4 github repositories:
* The app source code - https://github.com/RachelNaane/linkMe
* The Jenkinsfile repo - you are here:)
* The infrastructure source code - https://github.com/RachelNaane/linkMe-terraform
* The kubernetes source code - https://github.com/RachelNaane/linkMe-gitops

## Architecture

To see the full architecture of the whole project - https://miro.com/app/board/uXjVPpJXqes=/?share_link_id=803191704280

## Deployment

This repo holds the CI part of the project.
To deploy this part of the project run

```bash
  docker-compose up -d
```
Access jenkins in localhost:8080.

## Plugins

In my project, I installed on my jenkins the 'Remote Jenkins Provider' plugin. This allows me to pull the jenkinsfile from a different repo than my source code.
To learn more about this plugin see the documentation - https://plugins.jenkins.io/remote-file/ 
