pipeline {
    agent any

    parameters {
        choice(
            name: 'ENV',
            choices: ['dev', 'prod'],
            description: 'Environnement cible'
        )
    }

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        ANSIBLE_FORCE_COLOR        = 'true'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                sh 'ls -la'
            }
        }

        stage('Lint Ansible') {
            steps {
                dir('ansible') {
                    sh 'ansible-playbook site.yml --syntax-check'
                }
            }
        }

        stage('Dry Run') {
            steps {
                dir('ansible') {
                    sh 'ansible all -m ping -i inventories/${ENV}/hosts.ini'
                }
            }
        }

        stage('Approval (prod)') {
            when { expression { params.ENV == 'prod' } }
            steps {
                input message: 'Déployer en PROD ?', ok: 'GO'
            }
        }

        stage('Deploy') {
            steps {
                dir('ansible') {
                    sh "ansible-playbook -i inventories/${params.ENV}/hosts.ini site.yml"
                }
            }
        }

        stage('Smoke Test') {
            steps {
                sh """
                    echo "Test du site sur ${params.ENV}..."
                    docker exec ${params.ENV == 'prod' ? 'target2' : 'target1'} curl -f -s -o /dev/null -w "HTTP %{http_code}\n" http://localhost
                """
            }
        }
    }

    post {
        always {
            echo "Job ${currentBuild.fullDisplayName} terminé avec statut: ${currentBuild.currentResult}"
        }
        success {
            echo '✅ Déploiement réussi'
        }
        failure {
            echo '❌ Déploiement échoué — vérifier les logs'
        }
    }
}
