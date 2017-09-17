node {
    dir('demo') {
        // Mark the code checkout 'stage'....
        stage('Checkout from GitHub') {
            git url: 'https://github.com/karthik-nagaraj/jenkinsacr'
        }

        // Build and Deploy to ACR 'stage'... 
        stage('Build and Push to Azure Container Registry') {
                app = docker.build('acrjdtest.azurecr.io/node-demo')
                docker.withRegistry('https://acrjdtest.azurecr.io', 'acr_credentials') {
                app.push("${env.BUILD_NUMBER}")
                app.push('latest')
                }
        }

        stage('Open SSH Tunnel to Azure Swarm Cluster') {
                // Open SSH Tunnel to ACS Cluster
                sshagent(['acs_key']) {
                    sh 'ssh -fNL 2375:localhost:2375 -p 2200 jldeen@dnsprefix.location.cloudapp.azure.com -o StrictHostKeyChecking=no -o ServerAliveInterval=240 && echo "ACS SSH Tunnel successfully opened..."'
                }
                // Check tunnel is open and set DOCKER_HOST env var
           env.DOCKER_HOST=':2375'
           sh 'echo "DOCKER_HOST is $DOCKER_HOST"'
           sh 'docker info'
        }

        // Pull, Run, and Test on ACS 'stage'... 
        stage('ACS Docker Pull and Run') {
           app = docker.image('acrjdtest.azurecr.io/node-demo:latest')
           docker.withRegistry('https://acrjdtest.azurecr.io', 'acr_credentials') {
           app.pull()
           app.run('--name node-demo -p 80:8000')
            }
        }
    }
}
