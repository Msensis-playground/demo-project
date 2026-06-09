pipeline {
    agent { label 'spring-boot-agent' }

    stages {
        stage('Build') {
            when { branch 'master' }
            steps {
                sh './gradlew clean bootJar'
            }
        }

        stage('Deploy') {
            when { branch 'master' }
            steps {
                script {
                    def jarPath = sh(script: 'find build/libs -maxdepth 1 -name "*.jar" ! -name "*-plain.jar" | head -n 1', returnStdout: true).trim()
                    def jarName = sh(script: "basename ${jarPath}", returnStdout: true).trim()

                    echo "Deploying artifact: ${jarName} to 10.0.0.186..."

                    sh """
                        scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i ~/.ssh/id_ed25519 \
                        ${jarPath} root@10.0.0.186:/var/demo-app/
                    """

                    sh """
                        ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -i ~/.ssh/id_ed25519 root@10.0.0.186 << EOF
                        cd /var/demo-app
                        ln -sf ${jarName} app.jar
                        systemctl restart myapp.service
EOF
                    """
                }
            }
        }
    }
}