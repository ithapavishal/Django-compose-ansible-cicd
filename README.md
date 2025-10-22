# django-compose-ansible-cicd

ansible/
├── ansible.cfg
├── inventory/
│   ├── dev.ini
│   └── prod.ini
├── group_vars/
│   ├── dev.yml
│   └── prod.yml
├── playbooks/
│   └── deploy.yml
└── roles/
    ├── docker_setup/
    │   ├── tasks/
    │   │   └── main.yml
    │   └── handlers/
    │       └── main.yml
    └── app_deploy/
        ├── tasks/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        └── templates/
            └── env.j2
