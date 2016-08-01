Documentation
1. The ansible server (v2.1) was created on Amazon Web Services on an ec2 t2.micro instance running Amazon Linux AMI 2016.03.3. It utilizes the dynamic inventory/hosts script ec2.py (http://docs.ansible.com/ansible/intro_dynamic_inventory.html).

2. The application server is created and configured using ansible playbooks on Amazon Web Services on an ec2 t2.micro instance running Amazon Linux AMI 2016.03.3 (See attached playbooks create_inst.yaml & config_inst.yaml).

3. The application has been configured to listen on port 8080 and run in "production" to allow access to eth0.  It is started via a simple service script that has been added to runlevels 3,4 & 5.  The service references a launcher which sets up the environment and launches the app.

4. A user has been created for the app to run as (sinatra).  In order to access a privileged port (tcp/80),  iptables rules have been set up for filtering and forwarding from port 80 to the application listening on port 8080.

5. Security is provided by several factors:
	1. The AWS Linux footprint is very small and contains few if any unnecessary packages.
	2. Only tcp/22 and tcp/80 are visible to the internet.
	3. Authentication is by private key only.
	4. The application running under as an unprivileged user.
	5. fail2ban has been installed to proactively ban failed ssh attempts (this is actually redundant due to point 3, however I left it in anyway).

6. Stringing it all together, the order is: 
	1. Create app server
	2. Add keys to ssh-agent
	3. Refresh ec2 cache
	4. Configure server
	5. Get instance name and public IP address
	6. Reboot server
	7. Test

	ansible-playbook create_inst.yml && . ~/.anrc && /etc/ansible/ec2.py --refresh-cache && ansible-playbook config_inst.yml && ansible -m ec2_facts rea_simple_sinatra_app | egrep 'instance_id|public_ipv4s'| awk -F: '{print $2}'

	Reboot of the server is via ec2 cli utility command: ec2-reboot-instances instance-id