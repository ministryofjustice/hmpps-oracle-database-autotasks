def eng_account = "895523100917"

def project = [:]
project.config      = 'hmpps-env-configs'
project.autotasks   = 'hmpps-oracle-database-autotasks'

def environments = [
  'delius-core-sandpit',
  'delius-core-dev',
  'delius-int',
  'delius-test',
  'delius-perf',
  'delius-stage',
  'delius-mis-dev',
  'delius-mis-test',
  'delius-po-test1',
  'delius-training',
  'delius-training-test',
  'delius-perf',
  'delius-pre-prod',
  'delius-prod'
]


def prepare_env() {
    sh '''
        docker pull mojdigitalstudio/hmpps-ansible-builder:latest
    '''
}

def run_ansible(environment_name) {

    sshagent (credentials: ['hmpps_integration_test-key']) {
        sh """
        #!/usr/env/bin bash
        set +x
        docker run --rm \
        -v `pwd`:/home/tools/data \
        -v ~/.ssh:/home/tools/.ssh \
        -v $SSH_AUTH_SOCK:/ssh-agent \
        -e SSH_AUTH_SOCK=/ssh-agent \
        mojdigitalstudio/hmpps-ansible-builder \
            bash -c \"export ANSIBLE_CONFIG=/home/tools/data/hmpps-env-configs/ansible/ansible.cfg && \
                  ansible-playbook -u hmpps_integration_test \
                  -i "/home/tools/data/hmpps-env-configs/${environment_name}/ansible" \
                  ./hmpps-oracle-database-autotasks/configure_oracle_autotasks.yml \
                  --extra-vars "target_hosts=delius_primarydb,mis_primarydb,misboe_primarydb,misdsd_primarydb" \
                  -v \"
        set -x
        """
    }
}

pipeline {

  agent { label "oracle_ops" }

  parameters {
    choice(name: 'environment_name', choices: environments, description: 'Select environment')
   }

  stages {
    stage('Setup') {
      steps {
        dir( project.config ) {
          git url: 'git@github.com:ministryofjustice/' + project.config, branch: env.GIT_BRANCH.split(/\//)[1], credentialsId: 'f44bc5f1-30bd-4ab9-ad61-cc32caf1562a'
        }
        dir( project.autotasks ) {
          git url: 'git@github.com:ministryofjustice/' + project.autotasks,  branch: env.GIT_BRANCH.split(/\//)[1], credentialsId: 'f44bc5f1-30bd-4ab9-ad61-cc32caf1562a'
        }
        prepare_env()
      }
    }

    stage('Configure Oracle Autotasks') {
      steps {
        run_ansible(environment_name)
      }
    }
  }

  post {
    always {
      deleteDir()
      }
    failure {
      slackSend(message: "Failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}", color: 'danger')
    }
  }
}
