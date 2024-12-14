*Description*

This repository contains files for an Ansible role provisioning a forward proxy server on a Debian host.

__________
*Workflow*

The role utilises the Squid application to provision a forward proxy server. Squid was chosen as it inherently supports forward proxy functionality and is easily configurable.

1. The first playbook installas Squid on the Debian target node.
2. The second playbook installs the additional apache-utils2 dependency in order to enable basic authentication management using the htpasswd utility.
3. In the third playbook, Squid is configured as a forward proxy server with basic authentication and an IP allow-list:
     - First, an htpasswd file containing the basic authentication credentials is created, the credentials are sourced from the variables in /vars/main.yml.
     - You must ensure that you have defined your basic auth credentials in /vars/main.yml before executing the playbook. Create a relevant username and a strong password.
     - Then, Squid is configured as a forward proxy server, the configuration is sourced from the Jinja2 template in /templates/squid.conf.j2.
     - Before executing the playbook, you must define your IP allow-list in the Jinja2 template in the dedicated "acl allowed_ips" section.
     - The squid service is then restarted on the Debian target node to ensure the configuration comes into effect.
4. The fourth playbook ensures that the Squid service has been started and is running.

__________
*Applying the role*

Upload the role's directory (provision-forward-proxy-server-role) into the dedicated /roles directory in your main Ansible directory (/etc/ansible). Ensure your Ansible inventory has been configured correctly, i.e. that it contains the target node's address, the "become" password, the connection method, and any other details relevant to your setup.
Create a new playbook for the role or add the role into an existing playbook, then execute it:

ansible-playbook [your_playbook].yml

*add any additional attributes to your command, if relevant.

__________
*Testing*

Run the following curl command to test your forward proxy server:

curl -x [debian_target_node_address]:3128 -U [basic_auth_username]:[basic_auth_password] https://example.com/

The command should return the public IP address of your server.

___________
*Common "gotchas"*

- For improved security, it would be best to encrypt any secrets (i.e. sensitive information, such as usernames and passwords) using the ansible-vault utility.
- If you are testing this role on an Oracle VirtualBox Debian VM, make sure the openssh-server application has been installed (sudo apt-get install openssh-server).
- You can use any port number for your forward proxy server, 3128 is just among the most commonly used. Simply amend the Jinja2 config template with the port number of your choice.
- If you are getting 403 errors, the IP address of the machine from which you are making the request probably has not been listed in acl allowed_ips in the Jinja2 template.
- Troubleshooting tip: check the Squid access log for errors (sudo nano /var/logs/squid/access.log).
- If the Squid service appears to be inactive, run "sudo systemctl status squid" on the target Debian node. This will give you an overview of the service's status.
- If the Squid service is in "failed" status, run "journalctl -xeu squid" to get more information. The output might contain more exact reasons for the service's failure.
- Usually, the service's failure is caused by issues in its configuration file. Double-check the Jinja2 template and ensure that the syntax is correct, then re-run the Ansible playbook.
