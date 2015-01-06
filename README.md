# ansible-pm2-plugins

A simple configuration and deployment rig for Overview plugins, using [Ansible](https://github.com/ansible/ansible) and [PM2](https://github.com/Unitech/PM2). WIP.

## Deploy a plugin

1. Install Ansible with `pip install ansible`.

1. In your plugin, be sure that you've got a PM2 JSON process file, which defines the start-up script and any other configuration parameters. (See the full list of supported options [here](https://github.com/Unitech/PM2/blob/master/ADVANCED_README.md#user-content-json-app-declaration).)

1. Open up the `plugin.yml` file and fill in the `XXX` values. The important ones:

  - `app_dir` - The location on the server where the source code will be deployed.

  - `repo` - The git repository to deploy from.

  - `command`, in the "Build the front-end application" play - A command to build production-reader JavaScript and CSS payloads for the app, if necessary. Eg, things like `gulp dist` or `grunt compile`, depending on the build rig.

1. Open up the `hosts` file and replace the single host under `[webservers]` with the host for the new plugin.

1. Deploy with `ansible-playbook -i hosts plugin.yml`. If the `nmp install` task fails, it's probably because node dependencies are trying to compile things using apt packages that aren't installed. Add new dependencies to `roles/common/tasks/main.yml`.
