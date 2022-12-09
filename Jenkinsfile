pipeline() {
  agent any
  parameters {

        string(name: 'CLOUD_NAME', defaultValue: '', description: '')
        string(name: 'ANSIBLE_SSH_PASSWORD', defaultValue: '', description: '')
        string(name: 'OSP_PUDDLE', defaultValue: 'passed_phase2', description: '')
        string(name: 'OSP_REGISTRY_MIRROR', defaultValue: '',description: '')
        string(name: 'OSP_REGISTRY_NAMESPACE', defaultValue: '', description: '')
        string(name: 'OSP_INSECURE_REGISTRIES', defaultValue: '', description: '')
        string(name: 'CIDR', defaultValue: '192.168.24.0/22', description: '')
        string(name: 'DHCP_START', defaultValue: '192.168.24.5', description: '')
        string(name: 'DHCP_END', defaultValue: '192.168.25.10', description: '')
        string(name: 'INSPECTION_IPRANGE', defaultValue: '192.168.25.20,192.168.26.20', description: '')
        choice(name: 'LAB_NAME', choices: ['scale', 'alias'], description: '')
        choice(name: 'OSP_VERSION', choices: ['17.0', '16.1', '16.2', '16.0', '13.0'], description: '')
        choice(name: 'INTROSPECTION_TIMEOUT', choices: ['2400', '5000'], description: '')
        choice(name: 'OVERCLOUD_TIMEOUT', choices: ['560', '800'], description: '')
        choice(name: 'FOREMAN_URL', choices: ['', ''], description: '')
        string(name: 'CEPH_MACHINE_TYPE', defaultValue: '', description: '')
        booleanParam(name: 'CEPH_ENABLED', defaultValue: 'true', description: '')
        booleanParam(name: 'FORCE_REPROVISION', defaultValue: '', description: '')
        booleanParam(name: 'COMPOSABLE_ROLES', defaultValue: '', description: '')
        booleanParam(name: 'SETUP_JETPACK', defaultValue: '', description: '')
        booleanParam(name: 'UNDERCLOUD_DEPLOY', defaultValue: '', description: '')
        booleanParam(name: 'INTROSPECTION', defaultValue: '', description: '')
        booleanParam(name: 'TAGGING', defaultValue: '', description: '')
        booleanParam(name: 'SET_BOOT_MODE_DIRECTOR', defaultValue: '', description: '')
        booleanParam(name: 'OVERCLOUD_DEPLOY', defaultValue: '', description: '')
        booleanParam(name: 'BROWBEAT_INSTALL', defaultValue: '', description: '')
        booleanParam(name: 'CLEANUP_DISK', defaultValue: '', description: '')
        booleanParam(name: 'RUN_WORKLOAD', defaultValue: '', description: '')
        choice(name: 'WORKLOAD', choices: ['api', 'nova-boot-in-batches-with-delay', 'dynamic-workloads', 'nova', 'neutron-trunk', 'heat', 'octavia'], description: '')

  }

  stages {
    stage('setup jetpack') {
        when {
            expression { params.SETUP_JETPACK }
        }
        steps {
            sh 'rm -rf osp-ci'
            sh 'git clone https://github.com/asyedham/osp-ci.git  '
            sh 'rm -rf jetpack'
            sh 'git clone https://github.com/asyedham/jetpack.git'
            sh 'cd jetpack && git checkout root-hints'
            sh 'cp osp-ci/group_vars/ceph.yaml jetpack/group_vars/all.yml'
            sh 'cp osp-ci/browbeat/automate_workload_parameters.yml jetpack/'
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
            expression { params.OVERCLOUD_DEPLOY}
        }
        steps {
            sh "cd jetpack && ansible-playbook -vvv main.yml -t overcloud"
        }
    }

    stage('Setup Browbeat') {
        when {
            expression { params.BROWBEAT_INSTALL}
        }
        steps {
            sh "cd jetpack && ansible-playbook -vvv browbeat.yml"
        }
    }

    stage('Run Workloads') {
        when {
            expression { params.RUN_WORKLOAD}
        }
        steps {
            sh "rm /var/lib/jenkins/osp-ci"
            sh "ln -s $WORKSPACE/osp-ci /var/lib/jenkins/osp-ci"
            sh "cd jetpack && ansible-playbook -vvv automate_workload_parameters.yml"
        }
    }

  }
}
