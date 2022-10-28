pipeline() {
  agent any
  stages {
    stage('setup jetpack') {
        when {
            expression { params.SETUP_JETPACK }
        }
        steps {
            sh 'rm -rf osp-ci'
            sh 'git clone https://github.com/asyedham/osp-ci.git  '
            sh 'rm -rf jetpack'
            sh 'git clone https://github.com/redhat-performance/jetpack.git'
            sh 'cd jetpack'
            sh 'cp osp-ci/group_vars/all.yml jetpack/group_vars/all.yml'
            sh ' cd jetpack &&  ansible-playbook -vvv cleanup.yml'
            sh 'ansible-playbook -vvv osp-ci/create_jetpack_dir_symlink.yml'
        }
    }
    

    stage('Undercloud Install') {
        when {
            expression { params.UNDERCLOUD_DEPLOY}
        }
        steps {
            sh "cd jetpack && ansible-playbook -vvv main.yml -t undercloud"
        }
    }

    stage('Set Boot Mode to director') {
        when {
            expression { params.SET_BOOT_MODE_DIRECTOR}
        }
        steps {
            sh "cd jetpack && ansible-playbook -vvv main.yml -t boot_order"
        }
    }
    
    stage('Introspection') {
        when {
            expression { params.INTROSPECTION}
        }
        steps {
            sh "cd jetpack && ansible-playbook -vvv main.yml -t introspect "
        }
    }
        
    stage('Tagging') {
        when {
            expression { params.TAGGING}
        }
        steps {
            sh "cd jetpack && ansible-playbook -vvv main.yml -t tag "
        }
    }
    
    stage('clean nodes') {
        when {
            expression { params.CLEANUP_DISK}
        }
        steps {
            sh "cp osp-ci/cleanup_disk.yaml jetpack/"
            sh "cd jetpack && ansible-playbook -vvv cleanup_disk.yaml  "
        }
    }
    
    stage('overcloud deploy') {
        when {
            expression { params.OVERCLOUD_DEPLOY }
        }
        steps {
            script {
                try {
                    sh "cd jetpack && ansible-playbook -vvv main.yml -t overcloud"
                } catch(err) {
                    echo "Overcloud deployment failed, retry again"
                    retry(0) {
                        sh "cd jetpack && ansible-playbook -vvv main.yml -t overcloud"
                    }
                }
            }
        }
    }

    stage('Setup Browbeat') {
        when {
            expression { params.BROWBEAT_INSTALL}
        }
        steps {
            sh "cd jetpack && ansible-playbook -i hosts -vvv main.yml -t browbeat"
        }
    }
  }
}
