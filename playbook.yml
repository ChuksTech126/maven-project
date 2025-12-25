- hosts: httpd
  become: yes

  tasks:
    - name: Ensure Python3 is installed
      yum:
        name: python3
        state: present

    - name: Install Docker SDK for Python (RPM-safe)
      yum:
        name: python3-docker
        state: present

    - name: Set up Docker yum repository
      yum_repository:
        name: docker
        description: Docker CE Repository
        baseurl: https://download.docker.com/linux/centos/7/$basearch/stable
        gpgcheck: yes
        gpgkey: https://download.docker.com/linux/centos/gpg
        enabled: yes

    - name: Install Docker packages
      yum:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
        state: present

    - name: Start and enable Docker service
      service:
        name: docker
        state: started
        enabled: yes

    - name: Pull Docker image from Docker Hub
      community.docker.docker_image:
        name: bloomytech/my-webpage:2.0
        source: pull

    - name: Run webserver container
      community.docker.docker_container:
        name: webserver
        image: bloomytech/my-webpage:2.0
        state: started
        restart_policy: always
        ports:
          - "8080:80"
