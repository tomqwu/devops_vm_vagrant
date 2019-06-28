- name: system
  hosts: all
  tasks:
    - name: Set timezone
      timezone:
        name: America/Toronto
  roles:
    - ./ansible-role-ntp

- name: Common packages
  hosts: all
  tasks:
    - name: upgrade all
      apt:
        upgrade: dist

    - name: install common packages
      package:
        name: "{{ item }}"
        state: present
      with_items:
        - apt-transport-https
        - ca-certificates
        - curl
        - software-properties-common
        - python-pip
        - python3-pip
        - openjdk-8-jdk
        - wget
        - unzip
        - jq
        - zip
        - dos2unix
        - golang-go

- name: install pip packages
  hosts: all
  tasks:
    - name: install pip packages
      pip:
        name: ["databricks-cli"]

- name: hashicorp
  hosts: all
  tasks:
    - name: install packer
      unarchive:
        src: https://releases.hashicorp.com/packer/1.4.1/packer_1.4.1_linux_amd64.zip
        dest: /usr/local/bin
        mode: 0755
        remote_src: yes

    - name: install terraform
      unarchive:
        src: https://releases.hashicorp.com/terraform/0.11.13/terraform_0.11.13_linux_amd64.zip
        dest: /usr/local/bin
        mode: 0755
        remote_src: yes

    - name: install vault
      unarchive:
        src: https://releases.hashicorp.com/vault/1.1.3/vault_1.1.3_linux_amd64.zip
        dest: /usr/local/bin
        mode: 0755
        remote_src: yes
    
    - name: get tfe-cli
      git:
        repo: https://github.com/hashicorp/tfe-cli.git
        dest: /opt/tfe-cli
    
    - name: link
      file:
        src: /opt/tfe-cli/bin/tfe
        dest: /usr/local/bin/tfe
        state: link

- name: Install Docker
  hosts: all
  tasks:
    - name: add docker gpg
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: docker apt repository
      apt_repository:
        repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable # bionic -> ubuntu 18.04
        state: present

    - name: install docker-ce
      package:
        name: "docker-ce"
        state: present

    - name: Add {{ lookup('env','USER') }} to docker group
      user:
        name: "{{ lookup('env','USER') }}"
        groups: docker
        append: yes

    - name: install compose
      get_url:
        url: https://github.com/docker/compose/releases/download/1.24.1/docker-compose-Linux-x86_64
        dest: /usr/local/bin/docker-compose
        mode: 0755

    - name: docker
      pip:
        name: ['docker', 'docker-compose']

- name: aws
  hosts: all
  tasks:
    - name: install pip packages
      pip:
        name: awscli
        state: latest

    - name: create .aws
      file:
        path: "{{ item }}"
        state: directory
        mode: 0755
      with_items:
        - /root/.aws
        - /home/{{ lookup('env','USER') }}/.aws
        
    - name: get amazon-ecr-credential-helper
      git:
        repo: "https://github.com/awslabs/amazon-ecr-credential-helper.git"
        dest: /opt/amazon-ecr-credential-helper

    - name: Check folder exists
      stat:
        path: /usr/bin/docker-credential-ecr-login
      register: stat_result

    - name: build helper
      shell: |
        cd /opt/amazon-ecr-credential-helper
        make docker
        mv bin/local/docker-credential-ecr-login /usr/bin
        chmod +x /usr/bin/docker-credential-ecr-login
      when: stat_result.stat.exists == False

    - name: create .docker
      file:
        path: "{{ item }}"
        state: directory
        mode: 0755
      with_items:
        - /root/.docker
        - /home/{{ lookup('env','USER') }}/.docker

    - name: cp config.json
      copy:
        src: files/config.json
        dest: "{{ item }}/config.json"
      with_items:
        - /root/.docker
        - /home/{{ lookup('env','USER') }}/.docker

- name: kubernetes
  hosts: all
  tasks:
    - name: Add an Apt signing key
      apt_key:
        url: https://packages.cloud.google.com/apt/doc/apt-key.gpg
        state: present

    - name: Add kubernetes repo
      apt_repository:
        repo: deb https://apt.kubernetes.io/ kubernetes-xenial main
        state: present

    - name: install kubectl
      package:
        name: kubectl
        state: present

    - name: Download aws-iam-authenticator
      get_url:
        url: https://amazon-eks.s3-us-west-2.amazonaws.com/1.13.7/2019-06-11/bin/linux/amd64/aws-iam-authenticator
        dest: /usr/bin/aws-iam-authenticator
        mode: 0755

    - name: get helm
      unarchive:
        src: https://get.helm.sh/helm-v2.14.1-linux-amd64.tar.gz
        dest: /opt
        remote_src: yes

    - name: link helm
      file:
        src: /opt/linux-amd64/helm
        dest: /usr/local/bin/helm
        state: link
