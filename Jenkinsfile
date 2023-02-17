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

        stage ("build") {
            steps {
                sh "docker build -t app ."
            }
        }

        stage ("unit test") {
            steps {
                sh """
                    docker run --name app -d -p 3000:3000 app
                    sleep 4
                    ip=\$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
                    chmod +x ./tests/unit-test.sh
                    ./tests/unit-test.sh \$ip:3000
                """
            }
            post { always { sh " docker stop app && docker rm app" } }
        }
    }
}