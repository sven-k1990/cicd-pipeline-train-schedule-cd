pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('DeployToStaging') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'webserver_login', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'staging', 
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$PASSWORD"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule',
                                        remoteDirectory: '/tmp', 
                                        remoteDirectorySDF: false, 
                                        removePrefix: 'dist/', 
                                        sourceFiles: 'dist/trainSchedule.zip')
                                ],
                                usePromotionTimestamp: false, 
                                useWorkspaceInPromotion: false,
                                verbose: false)
                        ]
                    )
                }
            }
        }
    }
}
