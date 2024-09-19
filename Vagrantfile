# vagrant plugin install vagrant-aws 
# vagrant up --provider=aws
# vagrant destroy -f && vagrant up --provider=aws
## optional:
# export COMMON_COLLECTION_PATH='~/git/inqwise/ansible/ansible-common-collection'
# export STACKTREK_COLLECTION_PATH='~/git/inqwise/ansible/ansible-stack-trek'

TOPIC_NAME = "errors"
ACCOUNT_ID = "992382682634"
AWS_REGION = "il-central-1"
MAIN_SH_ARGS = <<MARKER
-e "playbook_name=ansible-loki discord_message_owner_name=#{Etc.getpwuid(Process.uid).name}"
MARKER
NODE_COUNT = 1
Vagrant.configure("2") do |config|
  (1..NODE_COUNT).each do |i|
    config.vm.define "node#{i}" do |subconfig|
      subconfig.vm.provision "shell", inline: <<-SHELL
        set -euxo pipefail
        echo "start vagrant file"
        cd /vagrant
        python3 -m venv /tmp/ansibleenv
        source /tmp/ansibleenv/bin/activate
        aws s3 cp s3://resource-opinion-stg/get-pip.py - | python3
        echo $PWD
        export VAULT_PASSWORD=#{`op read "op://Security/ansible-vault inqwise-opinion-stg/password"`.strip!}
        echo "$VAULT_PASSWORD" > vault_password
        export ANSIBLE_VERBOSITY=0
        export ANSIBLE_DISPLAY_SKIPPED_HOSTS=false
        if [ -f "main.sh" ]; then
          echo "Local main.sh found. Run the local main.sh script..."
          bash main.sh #{MAIN_SH_ARGS}
        else
          echo "Local main.sh not found. running the main.sh script from the URL..."
          curl -s https://raw.githubusercontent.com/inqwise/ansible-automation-toolkit/default/main_amzn2023.sh | bash -s -- #{MAIN_SH_ARGS}
        fi
        rm vault_password
      SHELL

      subconfig.vm.provider :aws do |aws, override|
        override.vm.box = "dummy"
        override.ssh.username = "ec2-user"
        override.ssh.private_key_path = "~/.ssh/id_rsa"
        aws.access_key_id             = `op read "op://Security/aws inqwise-stg/Security/Access key ID"`.strip!
        aws.secret_access_key         = `op read "op://Security/aws inqwise-stg/Security/Secret access key"`.strip!
        #aws.aws_dir = ENV['HOME'] + "/.aws/"
        #aws.aws_profile = "opinion-stg"
        aws.keypair_name = Etc.getpwuid(Process.uid).name
        override.vm.allowed_synced_folder_types = [:rsync]
        override.vm.synced_folder ".", "/vagrant", type: :rsync, rsync__exclude: ['.git/','inqwise/'], disabled: false
        common_collection_path = ENV['COMMON_COLLECTION_PATH'] || '~/git/ansible-common-collection' 
        stacktrek_collection_path = ENV['STACKTREK_COLLECTION_PATH'] || '~/git/ansible-stack-trek'
        override.vm.synced_folder common_collection_path, '/vagrant/collections/ansible_collections/inqwise/common', type: :rsync, rsync__exclude: '.git/', disabled: false
        override.vm.synced_folder stacktrek_collection_path, '/vagrant/collections/ansible_collections/inqwise/stacktrek', type: :rsync, rsync__exclude: '.git/', disabled: false

        aws.region = AWS_REGION  
        aws.security_groups = ["sg-020afd8fd0fa9fd0b","sg-0fb6558f0dd742c1e", "sg-00cd6b533fef19ceb"]
        # public-ssh, loki, consul
        aws.ami = "ami-009b671c6592c55db"
        aws.instance_type = "t4g.small"
        aws.subnet_id = "subnet-0f46c97c53ea11e2e"
        aws.associate_public_ip = true
        aws.iam_instance_profile_name = "bootstrap-role"
        aws.tags = {
          Name: "loki-test-#{Etc.getpwuid(Process.uid).name}",
          private_dns: "loki-test-#{Etc.getpwuid(Process.uid).name}"
        }
      end
    end
  end
end








 #   aws.security_groups = ["sg-0e11a618872a5a387","sg-0fb6558f0dd742c1e"]
        # public-ssh, loki
 