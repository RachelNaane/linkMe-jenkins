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

        stage ("E2E test") {
            steps {
                sh """
                    export APP_IMAGE=app
                    docker compose up -d

                    ip=\$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
                    chmod +x ./tests/e2e-test.sh
                    ./tests/e2e-test.sh \$ip:80
                """
            }
            post { always { sh "export APP_IMAGE=app && docker compose down" } }
        }

        stage ("version") {
            when {  branch "main"  }
            steps {
                script {
                    sh 'git fetch https://${REPO_CRED_USR}:${REPO_CRED_PSW}@${SCM_REPO_URL} --tags'
                    
                    lastTag = sh(script: "git describe --tags --abbrev=0 || echo ''", returnStdout: true ).trim()
                    
                    if (lastTag) { 
                        versionParts = lastTag.tokenize(".")
                        nextVersion = "${versionParts[0]}.${versionParts[1]}.${(versionParts[2].toInteger() + 1)}"
                    } else {
                        nextVersion = "1.0.0"
                    }

                    env.NEXT_VERSION = nextVersion
                    println(env.NEXT_VERSION)
                }
            }
        }

        stage ("publish") {
            when { branch "main" }
            steps {
                sh """
                    aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin 644435390668.dkr.ecr.eu-west-2.amazonaws.com
                    docker tag app:latest 644435390668.dkr.ecr.eu-west-2.amazonaws.com/rachel-naane-linkme:${env.NEXT_VERSION}
                    docker push 644435390668.dkr.ecr.eu-west-2.amazonaws.com/rachel-naane-linkme:${env.NEXT_VERSION}
                """
            }
        }

        stage ("tag") {
            when { branch "main" }
            steps {
                withCredentials([gitUsernamePassword(credentialsId: 'linkMe-repos-access', gitToolName: 'Default')]) { 
                    sh """
                        git tag -a ${env.NEXT_VERSION} -m "new release - version ${env.NEXT_VERSION}"
                        git push origin ${env.NEXT_VERSION}
                    """
                }
            }
        }

        stage ("deploy") {
            when { branch "main" }
            steps {
                withCredentials([gitUsernamePassword(credentialsId: 'linkMe-repos-access', gitToolName: 'Default')]) { 
                    sh """
                        git clone ${GITOPS_REPO_URL}
                        cd linkMe-gitops/linkMe/
                        sed -i "s/tag: .*/tag: ${env.NEXT_VERSION}/" values.yaml
                        git add values.yaml
                        git commit -m "by terraform - updated image tag to ${env.NEXT_VERSION}"
                        git push origin main
                    """
                }
            }
        }
    }

    post {
        always {
            sh "docker image rm app || echo 'no image found'"
            sh "docker image rm 644435390668.dkr.ecr.eu-west-2.amazonaws.com/rachel-naane-linkme:${env.NEXT_VERSION} || echo 'no image found'"
            sh "docker image rm app_main-nginx || echo 'no image found'"    
        }
        failure {
            emailext recipientProviders: [culprits()],
            subject: 'build failure',
            body: 'something in your last commit broke the code!', 
            attachLog: true, 
            compressLog: true                    
        }
        success {
            emailext recipientProviders: [culprits()],
            subject: 'build success!',
            body: 'good job!',
            attachLog: true,  
            compressLog: true            
        }
    }
}