pipeline {
    agent any
    environment{
        DOCKERHUB_CREDENTIALS = credentials('docker_hub')
    }
    stages {
        stage('Build Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {
                    app = docker.build("$DOCKER_HUB/demolog4")
                }
            }
        }
        stage('Login'){
            steps{
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'main'
            }
            steps {
                script {

                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")

                }
            }
        }
        stage('Lacework Vulnerability Scan') {
            environment {
                LW_API_SECRET = credentials('lacework_api_secret')
            }
            agent {
                docker { image 'lacework/lacework-cli:latest' }
            }
            when {
                branch 'main'
            }
            steps {
                echo 'Running Lacework vulnerability scan'
                sh "lacework vulnerability container scan index.docker.io $DOCKER_HUB/lacework-cli latest --poll --noninteractive --details " //--fail_on_severity critical --fail_on_severity high --fail_on_severity medium"

            }

        }


        stage('Prisma Scan'){




            steps {


                // Scan the image


                //prismaCloudScanImage  dockerAddress: "unix:///var/run/docker.sock", image: "$DOCKER_HUB/lacework-cli", logLevel: 'debug', resultsFile: 'prisma-cloud-scan-results.json'
                // Scan the image


                prismaCloudScanImage ca: '/certs/ca/cert.pem',
                                     cert: '/certs/client/cert.pem',
                                     dockerAddress: 'https://docker:2376',
                                     image: 'ivanorte/demolog4',
                                     key: '/certs/client/key.pem',
                                     logLevel: 'info',
                                     resultsFile: 'prisma-cloud-scan-results.json',
                                     ignoreImageBuildTime: true
                 }
        }
    }


     post {
        always {
            // The post section lets you run the publish step regardless of the scan results
            prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'

        }
    }
}
