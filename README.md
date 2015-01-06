# ansible-pm2-plugins

A simple configuration and deployment rig for Overview plugins, using [Ansible](https://github.com/ansible/ansible) and [PM2](https://github.com/Unitech/PM2).

## Spin up a server

1. Create a new EC2 instance in the `plugins` subnet.

1. Create a new elastic IP, associate it with the instance.

1. Create a new subdomain (`new-plugin.plugins.overviewproject.org`), and point it at the new elastic IP.

1. Generate a new SSL key with `openssl genrsa -out server.key 2048`.

1. Create a certificate signing request with `openssl req -new -sha256 -key ~/server.key -out ~/osp-map.plugins.overviewproject.org.csr`. Be sure to leave the password blank.

1. Purchase a SSL certificate from [Namecheap](https://www.namecheap.com/security/ssl-certificates/domain-validation.aspx), using the CSR from the last step.

1. Once the certificate is validated, you'll get a .zip file with four certificates:

  - new-plugin_plugins_overviewproject_org.crt
  - COMODORSADomainValidationSecureServerCA.crt
  - COMODORSAAddTrustCA.crt
  - AddTrustExternalCARoot.crt

  Concatenate all of the certificates together, in this order, from most to least specific:

  `cat new-plugin_plugins_overviewproject_org.crt COMODORSADomainValidationSecureServerCA.crt COMODORSAAddTrustCA.crt AddTrustExternalCARoot.crt > bundle.crt`

1. Move `server.key` -> `/etc/nginx/cert.key`, and `bundle.crt` -> `/etc/nginx/cert.csr`.

## Deploy a plugin

1. Install Ansible with `pip install ansible`.

1. In your plugin, be sure that you've got a PM2 JSON process file, which defines the start-up script and any other configuration parameters. (For example, [this file](https://github.com/overview/osp-map/blob/master/app.json) in the OSP map plugin; see the full list of supported options [here](https://github.com/Unitech/PM2/blob/master/ADVANCED_README.md#user-content-json-app-declaration).)

1. Open up the `plugin.yml` file and fill in the `XXX` values. The important ones:

  - `app_dir` - The location on the server where the source code will be deployed.

  - `repo` - The git repository to deploy from.

  - `command`, in the "Build the front-end application" play - A command to build production-reader JavaScript and CSS payloads for the app, if necessary. Eg, things like `gulp dist` or `grunt compile`, depending on the build rig.

1. Open up the `hosts` file and replace the single host under `[webservers]` with the host for the new plugin.

1. Install the SSL certificate at `/etc/nginx/cert.{key|crt}`

1. Deploy with `ansible-playbook -i hosts plugin.yml`. If the `nmp install` task fails, it's probably because node dependencies are trying to compile things using apt packages that aren't installed. Register new dependencies in `roles/common/tasks/main.yml`.

1. And, the plugin should be running at `xxx.plugins.overviewproject.org`!
