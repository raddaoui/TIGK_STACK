install TIGK stack to monitor your cloud
#########################################
:tags: telegraf, InfluxDB, Grafana, lxc, ansible


// install TIGK stackbout this repository
---------------------

This set of commands/playbooks will deploy TIGK stack inside lxc containers

Process
-------

Inside your monitring server/VM (ubuntu14.04) update you repo sources:

.. code-block:: bash

	cat  >  /etc/apt/sources.list << EOF
	deb http://archive.ubuntu.com/ubuntu trusty main restricted
	deb-src http://archive.ubuntu.com/ubuntu trusty main restricted
	deb http://archive.ubuntu.com/ubuntu trusty-updates main restricted
	deb-src http://archive.ubuntu.com/ubuntu trusty-updates main restricted
	deb http://archive.ubuntu.com/ubuntu trusty universe
	deb-src http://archive.ubuntu.com/ubuntu trusty universe
	deb http://archive.ubuntu.com/ubuntu trusty-updates universe
	deb-src http://archive.ubuntu.com/ubuntu trusty-updates universe
	deb http://archive.ubuntu.com/ubuntu trusty multiverse
	deb-src http://archive.ubuntu.com/ubuntu trusty multiverse
	deb http://archive.ubuntu.com/ubuntu trusty-updates multiverse
	deb-src http://archive.ubuntu.com/ubuntu trusty-updates multiverse
	deb http://archive.ubuntu.com/ubuntu trusty-backports main restricted universe multiverse
	deb-src http://archive.ubuntu.com/ubuntu trusty-backports main restricted universe multiverse
	deb http://security.ubuntu.com/ubuntu trusty-security main restricted
	deb-src http://security.ubuntu.com/ubuntu trusty-security main restricted
	deb http://security.ubuntu.com/ubuntu trusty-security universe
	deb-src http://security.ubuntu.com/ubuntu trusty-security universe
	deb http://security.ubuntu.com/ubuntu trusty-security multiverse
	deb-src http://security.ubuntu.com/ubuntu trusty-security multiverse
	EOF


Update your package manager and install the following packages

.. code-block:: bash

	apt-get update
	sudo apt-get install -y python-pip python-dev libmysqlclient-dev git lxc

Install ansible

.. code-block:: bash

	sudo apt-get install software-properties-common
	sudo apt-add-repository ppa:ansible/ansible
	sudo apt-get update
	sudo apt-get install ansible

clone current repo

.. code-block:: bash

	git clone https://github.com/raddaoui/TIGK_STACK.git /opt/TIGK_STACK


Create two lxc containers one for your monitoring stack and one for galera

.. code-block:: bash

	sudo lxc-create -t download -n TIGK1 -- --dist ubuntu --release trusty --arch amd64
	sudo lxc-create -t download -n galera -- --dist ubuntu --release trusty --arch amd64
	lxc-start -d -n TIGK1
	lxc-start -d -n galera

add your ssh key in your host  to authorized_keys in each container to enable password less sshing


.. code-block:: bash

	# login to first container 
	lxc-attach -n TIGK1
	# add your host/VM publick key to authorized_keys
	mkdir ~/.ssh
	vi ~/.ssh/authorized_keys
	# install openssh-server to enable sshing to the contaier
	apt-get install openssh-server

.. code-block:: bash

        # login to first container
        lxc-attach -n galera
        # add your host/VM publick key to authorized_keys
        mkdir ~/.ssh
        vi ~/.ssh/authorized_keys
        # install openssh-server to enable sshing to the contaier
        apt-get install openssh-server
	# install mysql
	sudo apt-get install mysql-server
	apt-get install mysql-server python-pip python-dev libmysqlclient-dev
	pip install MySQL-python


Install Influx in your TIGK container

.. code-block:: bash
	
	ansible-playbook playbook-influx-db.yml

Install telegraf in TIGK container

.. code-block:: bash
	
	ansible-playbook playbook-influx-telegraf.yml

Install grafan in TIGK container. Change Galera_container_ip and  root_password to what your galera container ip and your mysql root password

.. code-block:: bash

	ansible-playbook -i inv.yml playbook-grafana.yml -e galera_root_user=root -e galera_address='Galera_container_ip' -e galera_root_password='root_password'


Install telegraf everywhere where you want to monitor (if hosts your want to monitor are using a different key you can speciy that in command otherwise remove)

.. code-block:: bash

	ansible-playbook playbook-influx-telegraf.yml --private-key=path_to_private_key

