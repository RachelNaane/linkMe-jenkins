pipeline {

    agent any
    options {
        timeout(time:5, unit:'MINUTES')
        buildDiscarder(logRotator(
            numToKeepStr: '10',
            daysToKeepStr: '7',
            artifactNumToKeepStr: '10'))
    }

    environment {
        REPO_CRED=credentials('linkMe-repos-access')
        SCM_REPO_URL = "github.com/RachelNaane/linkMe.git"
        GITOPS_REPO_URL = "https://github.com/RachelNaane/linkMe-gitops.git"
    }

    stages {
        stage ("clean & clone") {
            steps{
                deleteDir()
                checkout scm
            }           
        }
    }
}