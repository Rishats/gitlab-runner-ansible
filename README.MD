# Ansible playbooks for GitLab runner

* Docker
* GitLab Runner

## Getting started

### Supported distributions

* Ubuntu Trusty
* Debian (Not tested)

Other versions of the distributions listed above might work, but no guarantees given.

### Installing Ansible
You can install Ansible by following the [official installation documentation](http://docs.ansible.com/ansible/intro_installation.html). For Windows 10 users, you can use the instructions for Ubuntu after [installing the Linux subsystem](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide). For older Windows version, please [check this tutorial on running Ansible on cygwin](https://www.jeffgeerling.com/project/running-ansible-within-windows).

### Clone the repository
```shell
git clone git@github.com:Rishats/ansible_gitlab_runner.git
```

### Configure your inventory files
The inventory files are located in the inventory directory. Check the example for development and read the [Ansible documentation](http://docs.ansible.com/ansible/intro_inventory.html).

### Configure your host variables
Host variables control some of the aspects how the machine is configured. Store the host variable file with the same name as the hostname in the inventory file. Below is an example for the production development machine. 

```yaml
---
# Change this variable value to 'gitlab-runner' for versions >= 10.x
gitlab_runner_package_version: '11.8.0'
gitlab_runner_package_name: 'gitlab-runner'

# Maximum number of global jobs to run concurrently
gitlab_runner_concurrent: '{{ ansible_processor_vcpus }}'

# GitLab coordinator URL
gitlab_runner_coordinator_url: 'https://gitlab.example.com/ci'
# GitLab registration token
gitlab_runner_registration_token: 'token-admin-panel'

gitlab_runner_sentry_dsn: ''

# A list of runners to register and configure
gitlab_runner_runners:
    # The identifier of the runner.
  - name: '{{ ansible_hostname }}'
    # set to 'absent' if you want to delete the runner. Defaults to 'present'.
    state: present
    # The executor used by the runner.
    executor: 'docker'
    # Maximum number of jobs to run concurrently on this specific runner.
    # Defaults to 0, simply means don't limit.
    concurrent_specific: '1'
    # The default Docker image to use. Required when executor is `docker`.
    docker_image: 'php:7.3'
    # The tags assigned to the runner.
    tags: ['sonarqube']
    # Indicates whether this runner can pick jobs without tags.
    run_untagged: false
    # Docker privileged mode
    docker_privileged: false
    # Runner Locked. When a runner is locked, it cannot be assigned to other projects
    locked: 'true'
    # Custom environment variables injected to registration command
    env_vars: []
    #
    # Runner SSH user
    # ssh_user: ''
    #
    # Runner SSH host
    # ssh_host: ''
    #
    # Runner SSH port
    # ssh_port: ''
    #
    # Runner SSH password
    # ssh_password: ''
    #
    # Runner SSH identity file
    # ssh_identity_file: ''
    #
    # Cache type
    # cache_type: 's3'
    #
    # Cache path
    # cache_path: prefix/key
    #
    # Cache shared
    # cache_shared: false
    #
    # Cache S3 server address
    # cache_s3_server_address: "s3.amazonaws.com"
    #
    # Cache S3 access key
    # cache_s3_access_key: "AMAZON_S3_ACCESS_KEY"
    #
    # Cache S3 secret key
    # cache_s3_secret_key: "AMAZON_S3_SECRET_KEY"
    #
    # Cache S3 bucket name
    # cache_s3_bucket_name: "my-bucket"
    #
    # Cache S3 bucket location
    # cache_s3_bucket_location: "eu-west-1"
    #
    # Cache S3 insecure
    # cache_s3_insecure: false
    #
    # Builds directory
    # builds_dir: '/builds_dir'
    #
    # Cache directory
    # cache_dir: '/cache'
    #
    # Extra registration option
    # extra_registration_option: '--maximum-timeout=3600'
    #
    # Extra configuration options to change in the config.toml file
    # This parameter is a dictionnary where the first level keys are TOML section names
    # Full list of configuration are available on Gitlab Runner documentation:
    # See https://docs.gitlab.com/runner/configuration/advanced-configuration.html
    #
    # extra_configs:
    #   runners.docker:
    #     memory: 512m
    #     allowed_images: ["ruby:*", "python:*", "php:*"]
    #   runners.docker.sysctls:
    #     net.ipv4.ip_forward: "1"

gitlab_runner_temp_dir: /tmp

docker__edition: "ce"
docker__channel: ["stable"]
docker__version: ""
docker__state: "present"
docker__users: ["{{ ansible_env.SUDO_USER | d('root') }}"]
```

### Install and run
1) Git clone project.
```shell
git clone git@github.com:Rishats/gitlab-runner-ansible.git
```

2) Create file production.yml in host_vars folder and paste data below:
```yaml
---
# Change this variable value to 'gitlab-runner' for versions >= 10.x
gitlab_runner_package_version: '11.8.0'
gitlab_runner_package_name: 'gitlab-runner'

# Maximum number of global jobs to run concurrently
gitlab_runner_concurrent: '{{ ansible_processor_vcpus }}'

# GitLab coordinator URL
gitlab_runner_coordinator_url: 'https://gitlab.example.com/ci'
# GitLab registration token
gitlab_runner_registration_token: 'token-admin-panel'

gitlab_runner_sentry_dsn: ''

# A list of runners to register and configure
gitlab_runner_runners:
    # The identifier of the runner.
  - name: '{{ ansible_hostname }}'
    # set to 'absent' if you want to delete the runner. Defaults to 'present'.
    state: present
    # The executor used by the runner.
    executor: 'docker'
    # Maximum number of jobs to run concurrently on this specific runner.
    # Defaults to 0, simply means don't limit.
    concurrent_specific: '1'
    # The default Docker image to use. Required when executor is `docker`.
    docker_image: 'php:7.3'
    # The tags assigned to the runner.
    tags: ['sonarqube']
    # Indicates whether this runner can pick jobs without tags.
    run_untagged: false
    # Docker privileged mode
    docker_privileged: false
    # Runner Locked. When a runner is locked, it cannot be assigned to other projects
    locked: 'true'
    # Custom environment variables injected to registration command
    env_vars: []
gitlab_runner_temp_dir: /tmp
```

3) Install python 3.6.
https://www.python.org/downloads/

4) Install virtualenv for python.
https://docs.python-guide.org/dev/virtualenvs/#lower-level-virtualenv

5) Create virtualenv for python.
```shell 
python3 -m venv env
```

6) Use virtualenv
```shell
source env/bin/activate
```

7) Install requirements for Python via pip
```shell
pip install -r requirements.txt
```

8) Open file [production](inventory/production) and edit ansible_ssh_host=ansible_ssh_host with your IP.

9) Run Check connection via Ansible for Appnode.
```shell
ansible -i inventory/production servers -m ping
```

10) Start playbook for Appnode via Ansible.
```shell
python3 $(which ansible-playbook) -i inventory site.yml
```

### Securing your host variables
Since you storing highly confidential information like production database passwords in the host variables and .env templates it is highly recommend that encrypt them with a symmetric AES key. Luckily Ansible has built-in tool for this called Vault. Please [read the Vault documentation](http://docs.ansible.com/ansible/playbooks_vault.html) on how to set it up.

#### Supported variables

## Running

### Checking SSH access
After you have done with the configuration make sure you have SSH access to the servers that listed on the inventory file. You can test this by using the ping module. Troubleshooting the SSH connection is the out of scope of this repository. Please note that you can set the SSH user and port in the inventory file, see the development for example.

```shell
ansible -i inventory/production servers -m ping
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
