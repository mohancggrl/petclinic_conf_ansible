pipeline {
    agent {
        label 'ansible-agent'
    }
    environment {
        workspace = "${WORKSPACE}"
    }
    parameters {
        string(
            name: 'app_version', 
            defaultValue: '1.0-1', 
            description: 'enter the application version to deploy'
        )
        extendedChoice(
            name: 'hosts',
            type: 'PT_MULTI_SELECT',
            value: '10.0.0.0,10.0.0.1',
            description: '',
            multiSelectDelimiter: ',',
            visibleItemCount: 2,
            quotedValue: false,
            saveJSONParameterToFile: falsess
        )
    }
    stages {
        stage('Install Dependencies via Ansible Galaxy') {
            steps {
                sh """
                    ansible-galaxy install -r requirements.yml -p ${workspace}/roles
                """
            }
        }
        stage('Run Playbook') {
            steps {
                sshagent(['SSH_CREDENTIALS_ID']) {
                    withCredentials([
                        string(credentialsId: 'ARTIFACTORY_TOKEN_ID', variable: 'ARTI_TOKEN'),
                        file(credentialsId: 'ANSIBLE_VAULT_FILE_ID', variable: 'VAULT_PASS_FILE')
                    ]) {
                        sh """
                            ansible-playbook -i "${hosts}," playbook.yml \
                                --vault-password-file "$VAULT_PASS_FILE" \
                                -e app_version="${app_version}" \
                                -e arti_token="${ARTI_TOKEN}"
                        """
                    }
                }
            }
        }
    }
}