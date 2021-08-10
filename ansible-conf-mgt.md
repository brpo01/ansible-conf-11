# Ansible Configuration Management

This projects covers how to manage the configurations of your servers/virtual Machines and automate tasks using ansible. Before we begin, we have to understand what ansible is and what it is capable of. Ansible is an automation used in automating deployments with the help of configuration scripts written in YAML that run the tasks. To understand how powerful ansible is, you have to think on a larger scale how it'll be possible to run deployments on multiple servers, let's say a thousand(1000) servers. Will it be done manually? well if that's the case it's going to take ages. Ansible is a tool that has been created to make this process easier. It is more efficient, it saves time, it is also very sensible from the business perspective as profit will be maximized and value served to the customers.

# Ansible Client as a Jump Server (Bastion Host)

To be able to configure our servers, we'll need a Jump Server that will run all of the configurations. A jump server is an intermediary server through which access to an internal network is provided, so basically what this means is that we do not have direct access to the servers we'll be making configurations on, but we can access them through a bastion host. To access the servers through a jump server can use the `ssh-agent` forwarding method, that will be covered in this project. This method of communicating though an intermediary provides better security and also reduces attack surface

![2](https://user-images.githubusercontent.com/47898882/128895391-f802244e-a5cf-41c0-bab6-7e7c41a10a3c.JPG)

This project will execute the following tasks
- Install and configure Ansible client to act as a Jump Server/Bastion Host
- Create an Ansible playbook to automate servers configuration.  

The reference architecture we'll be working with is shown below.

![7](https://user-images.githubusercontent.com/47898882/128916051-abd1f8ec-05ea-461d-95ab-4b3fca3ea485.JPG)




# INSTALL AND CONFIGURE ANSIBLE ON EC2 INSTANCE TO ACT AS JUMP SERVER/BASTION HOST
- Spin up an Ubuntu EC2 instance and install ansible.

```
$ sudo apt update
$ sudo apt install ansible
```
- Give it a name tag of `Bastion Host` or `Jump Server`. This will help for better refrencing.

- Create a Repository on Github to handle all of your files. You can call it `ansible-config-mgt`.

- When your repo is ready, configure a webhook that will trigger a build on Jenkins. If you do not understand how to configure jenkins, use this [github-project](https://github.com/brpo01/jenkinsci-9/blob/master/jenkins-ci-9.md) as a reference. This is a continuation of that project 

![3](https://user-images.githubusercontent.com/47898882/128914091-ca6ea409-c290-4e3a-8b8f-29d21b7de6b2.JPG)

- To Confirm the webhook works, set the build to trigger on *github hook trigger for GITScm polling* and add a post build action that copies all the files(**) after the build has been run. 

![4](https://user-images.githubusercontent.com/47898882/128914976-bbf7fb83-2411-49a6-be60-d04357ec4362.JPG)
![5](https://user-images.githubusercontent.com/47898882/128914724-9d05bb58-65ee-4639-b72c-65298025e1f2.JPG)

- After you have made these configurations, make a chnage in you repo and commit the changes. The build should run successfully without any manual intervention.

![6](https://user-images.githubusercontent.com/47898882/128915864-47f6ce3d-f13f-40ac-90c5-c541efe9a19d.JPG)


# Create an Ansible playbook to automate servers configuration.
- Within your repository, create a new branch `feature/ansible-conf` and checkout into that branch.

- Create two new directories and name them `inventories` & `playbooks`. Within the inventories folder, we keep details/information that are stored in a variable about the target servers such as `ansible_user`, `ansible_port`, `ansible_ssh_pass`. The playbooks directory is where the YAML configuration files that'll automate the tasks will be stored.

```
$ sudo mkdir playbooks inventories
```
- Create a common.yml file within the playbooks folder. The .yml is the extension for a YAML file.

```
$ sudo touch common.yml
```

- Within the inventories, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) `dev`, `staging`, `uat`, and `prod` respectively.

```
$ sudo touch dev staging uat prod
```

- Update your `dev.yml` file within the inventories file with this snippet of code 

![8](https://user-images.githubusercontent.com/47898882/128918664-e2fefa4c-5bab-4366-8cbf-40ff5d663e88.JPG)

- Write a playbook configuration in the common.yml file. You will write configuration for repeatable, re-usable, and multi-machine tasks. In the configuration, you will be automatint the installation of wireshark on the servers that have been grouped into nfs, webservers, db(database) & lb(load-balancer)

![9](https://user-images.githubusercontent.com/47898882/128919033-9477a7b8-e903-47b2-822e-d92966c16a20.JPG)

- Commit your code on the `feature/ansible-conf` branch to the repository.

```
$ git add .
$ git commit -m "added new files"
$ git push
```
- When you push, there'll be a request on your repository to pull the changes and merge with the master branch. If there are no conflicts, the merge should work out successfully.

- When the code has been merged to the master branch, a build will be triggered by the webhook and the files from the build will be saved as artifacts.

![14](https://user-images.githubusercontent.com/47898882/128929887-005ef639-9975-4743-bbfb-4a166bf272a4.JPG)


- Now that we have the configuration file ready, we need to run the file using the `ansible-playbook` command. But before we get ahead of ourselves, there is still one missing piece in the puzzle. How will the Jump Server/Bastion Host be able to communicate with the target servers?? because without an established connection between this intermediary server and the target servers, we cannot run the ansible playbook successfully. This is where the concept of ssh-agent forwarding comes into play, basically the ssh-agent helps to forward the private key of Bastion Host to the target servers & enable connectivity. The very good part about this is that your private key will not get stored on the the target servers, ssh-agent just forwards it to be used for connectivity.

- To achieve ssh-agent forwarding, use the `ssh-add` utility to add your public key to your local agent.

```
$ ssh-add .ssh/id_rsa
```

- Go to the /etc/ssh/ssh_config directory and make sure agent forwarding is enabled. Also list the ip addresses of your target servers in the Host section.

![10](https://user-images.githubusercontent.com/47898882/128922821-f66caa80-0224-4297-8637-07e2b1954659.JPG)

- Test that you can ssh into your target servers using ssh username@ipaddress, an example would be ssh ec2-user@172.31.14.214. If you connect successfully, then you have established a connection between your Bastion Host & Target Servers. 

![11](https://user-images.githubusercontent.com/47898882/128924367-6209508f-3d74-47a8-9fa1-893d9430fca3.JPG)

- You can now run your ansible playbook. If your syntax and configuration is accurate, it should run successfully. It should be run in this format
`ansible-playbook -i inventory_file yaml_config_file`

![1](https://user-images.githubusercontent.com/47898882/128925005-29f9fe35-7331-45b3-80b2-1645a2cf922b.JPG)

- You can check that wireshark installed successfully on the target servers using `wireshark --version`

![13](https://user-images.githubusercontent.com/47898882/128927117-cb0b5091-b6a0-4008-9e23-f7c8c48d9abd.JPG)

- The updated with Ansible architecture now looks like this:

![12](https://user-images.githubusercontent.com/47898882/128927296-d5d4a7dd-a913-4ea6-9f08-8fb4028319ab.JPG)

### Congratulations!! You have successfully used ansible to automate the configuration of your target servers.







