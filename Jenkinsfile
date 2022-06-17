pipeline {
    agent none
    stages {
        stage('Check markdown syntax') {
            agent { docker { image 'ruby:alpine' } }
            steps {
                sh 'apk --no-cache add git'
                sh 'gem install mdl'
                sh 'mdl --version'
                sh 'mdl --style all --warnings --git-recurse \${WORKSPACE}'
            }
	}
        stage('Test and deploy the application') {
            environment {
                SUDOPASS = credentials('sudopass')
		USER = credentials('user-ansible')
            }
            agent { docker { image 'registry.gitlab.com/robconnolly/docker-ansible:latest' } }
            stages {
               stage("Deploy app in production") {
                    when {
                       expression { GIT_BRANCH == 'origin/master' }
                    }
                   steps {
                       sh '''
                       apt-get update
                       apt-get install -y sshpass
                       ansible-playbook  -i hosts.yml  --extra-vars "ansible_user="$USER --extra-vars "ansible_password="$SUDOPASS \
		       --extra-vars ansible_ssh_common_args='"-o StrictHostKeyChecking=no -o ServerAliveInterval=30"' \
		       playbook.yml
                       '''
                   }
               } 
            }
          }
      }
}