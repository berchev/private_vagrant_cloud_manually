# private_vagrant_cloud_manually

This repo is a guideline on HOW to create and setup your own Private Vagrant Cloud. 

We will start step by step explaining everything in detail.

Note that all of the steps are tested on Ubuntu 18.04 

## Requirements
- [Vagrant installed](https://www.vagrantup.com/intro/getting-started/install.html#installing-vagrant)
- [Packer installed](https://www.packer.io/intro/getting-started/install.html)
- [VirtualBox installed](https://www.virtualbox.org/)
- Some basic Linux knowledge 
- Some basic Vagrant knowledge 
- Some basic Packer knowledge

## Repo files description
| File                   | Description                      |
|         ---            |                ---               |
|    test/boxes          | These directories are intentionally empty. We are going to use them in the project explanation |
|    xenial64_0_1        | All needed configuration required to create clean Xenial64 box  |
|    xenial64_0_2        | All needed configuration required to create Xenial64 box with some tools (kitchen, pythoon 3.6) installed |
|          images        | All screenshtos needed for readme file |
|     README.md          | Instructions file |

## Repo content
- [**Part I** - Boxes preparation](https://github.com/berchev/private_vagrant_cloud_manually#part-i---boxes-preparation)
- [**Part II** - Add box locally to vagrant](https://github.com/berchev/private_vagrant_cloud_manually#part-ii---add-box-locally-to-vagrant)
- [**Part III** - Create catalog (versioning) for one box](https://github.com/berchev/private_vagrant_cloud_manually#part-iii---create-catalog-versioning-for-one-box)
- [**Part IV** - Adding the second box to the catalog (versioning)](https://github.com/berchev/private_vagrant_cloud_manually#part-iv---adding-the-second-box-to-the-catalog-versioning)
- [**Part V** - Setup private vagrant cloud](https://github.com/berchev/private_vagrant_cloud_manually#part-v---setup-private-vagrant-cloud)
- [**Part VI** - Test your private vagrant cloud](https://github.com/berchev/private_vagrant_cloud_manually#part-vi---test-your-private-vagrant-cloud)

## Let's start!
- Download this repo: `git clone https://github.com/berchev/private_vagrant_cloud_manually.git`
- Change to **private_vagrant_cloud_manually** directory: `cd private_vagrant_cloud_manually`

## Part I - Boxes preparation
- Change to **xenial64_0_1** directory: `cd xenial64_0_1`
- Run command: `packer build xenial64.json` and wait till execution finish
- Into current directory you will see result box created: `xenial64_0.1.box` (For short we will call this box - **first box**)
- Move one directory backwards: `cd ..` 
- Change to **xenial64_0_2** directory: `cd xenial64_0_2`
- Run command: `packer build xenial64_with_tools.json` and wait till execution finish
- Into current directory you will see result box created: `xenial64_0.2.box` (For short we will call this box - **second box**)
- Move one directory backwards: `cd ..`

## Part II - Add box locally to vagrant
- Copy **first box** to **test/boxes** directory: `cp xenial64_0_1/xenial64_0.1.box test/boxes`
- Change to **test** directory: `cd test`
- Add **first_box** locally to vagrant: `vagrant box add --name rr boxes/xenial64_0.1.box`, `rr` is the name of the box (can be any other name)
- Now place in the current directory minimal **Vagrantfile** template (no help comments): `vagrant init -m rr`
- You can test your box: `vagrant up`
- Connect to the box: `vagrant ssh`
- Type: `exit`
- Destroy the box: `vagrant destroy`
- Remove the box from vagrant: `vagrant box remove rr`

## Part III - Create catalog (versioning) for one box
- Determine sha1 of your **first_box**: `openssl sha1 boxes/xenial64_0.1.box`
- Create catalog file **xenial64.json** into **test** directory: `touch xenial64.json`
- With your favorite editor add following content to the file you just created - **xenial64.json**
```
{
  "name": "xenial64",
  "description": "This box contains Ubuntu 16.04 LTS 64-bit.",
  "versions": [
    {
      "version": "0.1",
      "providers": [
        {
          "name": "virtualbox",
          "url": "file:///home/gberchev/spreadsheet/vagrant_test/private_vagrant_cloud_manually/test/boxes/xenial64_0.1.box",
          "checksum_type": "sha1",
          "checksum": "da4bff540d96784dca9d504bd93e2dc95d0fa03d"
        }
      ]
    }
  ]
}
```
- Replace **url** with full path to **xenial64_0.1.box** (beware with **///** after **file:**)
- Replace **checksum** with the resulted sha1


Instead of add the box locally to vagrant with `vagrant box add --name <chosen_name> <path_to_your_box_file>`, you can link the catalog to vagrant. Below you will find all steps to do this:
- Edit current **Vagrantfile** as follows:
```
Vagrant.configure("2") do |config|
  config.vm.box = "xenial64"
  config.vm.box_url = "file:///home/gberchev/spreadsheet/vagrant_test/private_vagrant_cloud_manually/test/xenial64.json"
end
```
- Field **config.vm.box**  should be the same as name you have defined in **xenial64.json** file. Otherwise you will hit an error telling you that you are requesting different name
- Field **config.vm.box_url** should be full path to **xenial64.json**
- Now you are ready to do `vagrant up`, **without** adding the box manually
- During vagrant up process, you have to see line that shows **loading metadata for box...** and **Successfully added box 'xenial64' (v0.1)**
- Halt the VM, do not destroy it! `vagrant halt`

## Part IV - Adding the second box to the catalog (versioning)
- Currently you are into **test** directory 
- Add the **second_box** to **boxes** directory: `cp ../xenial64_0_2/xenial64_0.2.box boxes/`
- Determine sha1 of your **second_box**: `openssl sha1 boxes/xenial64_0.2.box`
- Edit **xenial64.json** with adding some new lines:
```
{
  "name": "xenial64",
  "description": "This box contains Ubuntu 16.04 LTS 64-bit.",
  "versions": [
    {
      "version": "0.1",
      "providers": [
        {
          "name": "virtualbox",
          "url": "file:///home/gberchev/spreadsheet/vagrant_test/private_vagrant_cloud_manually/test/boxes/xenial64_0.1.box",
          "checksum_type": "sha1",
          "checksum": "da4bff540d96784dca9d504bd93e2dc95d0fa03d"
        }
      ]
    },
    {
      "version": "0.2",
      "providers": [
        {
          "name": "virtualbox",
          "url": "file:///home/gberchev/spreadsheet/vagrant_test/private_vagrant_cloud_manually/test/boxes/xenial64_0.2.box",
          "checksum_type": "sha1",
          "checksum": "351c981d452d50832ba40907ebd1cff23b9ee2b3"
        }
      ]
    }
  ]
}
```
- Replace **url** under **"version": "0.2"** with full path to **xenial64_0.2.box** (beware with **///** after **file:**)
- Replace **checksum** under **"version": "0.2"** with the resulted sha1 from **second_box**

Now you can check whether your box is outdated (If everything is ok with your configuration, you are supposed to see the new version)
- **Vagrantfile** is located in our current working directory:
- Let's check whether our box is outdated: `vagrant box outdated`
- Note: Instead of manually asking for outdated boxes, vagrant will notify you automatically when you use the Vagrant commands like `vagrant up`, `vagrant reload`, `vagrant resume`, etc.!
- Execute: `vagrant box update`
- Note that box with version **0.1** still exists. Vagrant will never remove your boxes automatically because of potential data loss.
- Execute: `vagrant destroy`
- Remove the old box: `vagrant box remove 'xenial64' --box-version '0.1'`
- Now if you do `vagrant up` box with version **0.2** will load
- Clean your environment: `- vagrant box remove xenial64`

## Part V - Setup private vagrant cloud
Now we have all basic knowledge needed to configure our private vagrant cloud. I do not have the opportunity to use different computer for setup nginx server, that's why I am going to use my current one. 

Don't worry, configuration is the same!

We will use following structure:
```
- /var/www                              # document root
        `- vagrant
           `- xenial64                    # box name folder
              |- boxes                  # contains all available box files
              |  |- xenial64_0.1.box    # version 0.1
              |  `- xenial64_0.2.box    # version 0.2
              `- xenial64.json            # box catalog
```
Webserver configuration: 
- Install nginx: `sudo apt-get install -y nginx`
- Crete target folders: `sudo mkdir -p /var/www/vagrant/xenial64/boxes`
- Delete the **default** sym-linked config for virtual hosts (vhost): `sudo rm -rf /etc/nginx/sites-enabled/default`
- Create a new specific vhost config for **www.example.com**: `sudo vi /etc/nginx/sites-available/example.com`
- Add following configuration:
```
server {
    listen   80 default_server;
    listen   [::]:80 ipv6only=on default_server;
    
    server_name example.com www.example.com;

    root /var/www;

    # Match the box name in location and search for its catalog
    # e.g. http://www.example.com/vagrant/xenial64/ resolves /var/www/vagrant/xenial64/xenial64.json  
    location ~ ^/vagrant/([^\/]+)/$ {
        index $1.json;
        try_files $uri $uri/ $1.json =404;
        autoindex off;
    }

    # Enable auto indexing for the folder with box files
    location ~ ^/vagrant/([^\/]+)/boxes/$ {
        try_files $uri $uri/ =404;
        autoindex on;
        autoindex_exact_size on;
        autoindex_localtime on;
    }

    # Serve json files with content type header application/json
    location ~ \.json$ {
        add_header Content-Type application/json;
    }

    # Serve box files with content type application/octet-stream
    location ~ \.box$ {
        add_header Content-Type application/octet-stream;
    }

    # Deny access to document root and the vagrant folder
    location ~ ^/(vagrant/)?$ {
        return 403;
    }
}
```
- Sym-link the vhost config to enable it: `sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/000-example.com`
- Now our boxes will not be stored anymore on the local filesystem. We need to create following file in our webserver: `sudo vi /var/www/vagrant/xenial64/xenial64.json`
- add inside following configuration:
```
{
  "name": "vagrant/xenial64",
  "description": "This box contains Ubuntu 16.04 LTS 64-bit.",
  "versions": [
    {
      "version": "0.1",
      "providers": [
        {
          "name": "virtualbox",
          "url": "http://www.example.com/vagrant/xenial64/boxes/xenial64_0.1.box",
          "checksum_type": "sha1",
          "checksum": "da4bff540d96784dca9d504bd93e2dc95d0fa03d"
        }
      ]
    },
    {
      "version": "0.2",
      "providers": [
        {
          "name": "virtualbox",
          "url": "http://www.example.com/vagrant/xenial64/boxes/xenial64_0.2.box",
          "checksum_type": "sha1",
          "checksum": "351c981d452d50832ba40907ebd1cff23b9ee2b3"
        }
      ]
    }
  ]
}
```
- Note that we have changed **name** field in order to make it more similar to [real Vagrant Cloud](https://app.vagrantup.com/boxes/search)
- Place **first_box** and **second_box** to webserver directory: `/var/www/vagrant/xenial64/boxes/`
- In order to reach newly created webserver please add following line into **/etc/hosts** file
```
192.x.x.x   www.example.com
```
- Replace `192.x.x.x` with your IP address

## Part VI - Test your private vagrant cloud
- Open URL `http://www.example.com/vagrant/xenial64/boxes/` and you have to list both boxes:
- Go to your test directory: `cd ~/gberchev/spreadsheet/vagrant_test/private_vagrant_cloud_manually/test` (Note that the **path** on your computer probably will be different)
- Edit the Vagrantfile in this way:
```
Vagrant.configure("2") do |config|
  config.vm.box = "vagrant/xenial64"
  config.vm.box_url = "http://www.example.com/vagrant/xenial64/"
end
```
- Execute: `vagrant up` and everything should work!
- Clean after the last test: `vagrant destroy`, `vagrant box remove vagrant/xenial64` and `rm -rf .vagrant`

ATTENTION! Now we will do something more fancy!
- Change to different directory (No really matter where... I will go to ~): `cd`
- Export ENV variable **VAGRANT_SERVER_URL**: `export VAGRANT_SERVER_URL="http://www.example.com/"` 
- Let's throw some light on this! By default, if you do `vagrant init -m <box_name>`, vagrant will search the box into [Vagrant Cloud](https://app.vagrantup.com/boxes/search). After you set Environment variable **VAGRANT_SERVER_URL**, you will change the default searching path of vagrant and now will point to our newly configured webserver **http://www.example.com/**
- Type: `vagrant init -m vagrant/xenial64`
- Type: `vagrant up` and enjoy! Your private vagrant cloud works like the real [Vagrant Cloud](https://app.vagrantup.com/boxes/search)
- Do not forget to clean after yourself: `vagrant destroy`, `vagrant box remove vagrant/xenial64` and `rm -rf .vagrant`

## TODO
