# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.hostname = "precise64"
  config.vm.box = "precise64"

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"

  # Foward of common ports
  config.vm.network :forwarded_port, guest: 7070, host: 7070
  config.vm.network :forwarded_port, guest: 4848, host: 4848

  # SSH Foward Agent (for github private keys)
  config.ssh.forward_agent = true

  # Enable VPN 
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end

  # Chef provisioning 
  config.vm.provision :shell, :inline => 'apt-get update'
  config.vm.provision :shell, :inline => 'apt-get install build-essential ruby1.9.1-dev --no-upgrade --yes'
  config.vm.provision :shell, :inline => "gem install chef --version 11.6.0 --no-rdoc --no-ri --conservative"
  config.vm.provision :chef_solo do |chef|
    chef.cookbooks_path = "cookbooks"
    chef.json = {
      "mysql" => {
        "port" => 3306,
        "allow_remote_root" => true,
        "server_root_password" => "root",
        "server_repl_password" => "root",
        "server_debian_password" => "root",
        "client" => {
          "packages" => ["mysql-client", "libmysqlclient-dev"]
        }
      },
      "java" => {
        "install_flavor" => "oracle",
        "jdk_version" => 7,
        "oracle" => {
          "accept_oracle_download_terms" => true
        }
      },
      "glassfish" => {
        "version" => "3",
        "package_url" => "http://dlc.sun.com.edgesuite.net/glassfish/3.1.2.2/release/glassfish-3.1.2.2.zip",
        "base_dir" => "/usr/local/glassfish",
        "domains_dir" => "/usr/local/glassfish/glassfish/domains",
        "domains" => {
          "domain1" => {
            "config" => {
              "port" => 7070,
              "admin_port" => 4848,
              "username" => "admin",
              "password" => "admin",
              "remote_access" => true,
              "secure" => false
            },
            'extra_libraries' => {
              'jdbcdriver' => {
                'type' => 'common',
                'url' => 'http://repo1.maven.org/maven2/mysql/mysql-connector-java/5.1.29/mysql-connector-java-5.1.29.jar'
              }
            },
            'jdbc_connection_pools' => {
              'AppPool' => {
                'config' => {
                  'datasourceclassname' => 'com.mysql.jdbc.jdbc2.optional.MysqlDataSource',
                  'restype' => 'javax.sql.DataSource',
                  'isconnectvalidatereq' => 'true',
                  'validationmethod' => 'auto-commit',
                  'ping' => 'true',
                  'description' => 'App Pool',
                  'properties' => {
                    'Instance' => "jdbc:mysql://localhost:3306/appdb",
                    'ServerName' => "localhost",
                    'User' => 'root',
                    'Password' => 'root',
                    'PortNumber' => '3306',
                    'DatabaseName' => 'appdb'
                  }
                },
                'resources' => {
                  'jdbc/App' => {
                    'description' => 'Resource for App Pool',
                  }
                }
              }
            }
          }
        }
      }
    }
    chef.add_recipe "apt"
    chef.add_recipe "java::default"
    chef.add_recipe "glassfish::attribute_driven_domain"
    chef.add_recipe "mysql::server"
    chef.add_recipe "cassandra::datastax"
  end

end