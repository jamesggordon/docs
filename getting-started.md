# Getting Started

First, let's [install Deployer](installation.md). Run the following commands in the terminal:

```sh
curl -LO https://deployer.org/deployer.phar
mv deployer.phar /usr/local/bin/dep
chmod +x /usr/local/bin/dep
```

Now you can use Deployer via the `dep` command. 
Open up a terminal in your project directory and run:

```sh
dep init
```

This command will create the `deploy.php` file in the current directory. It is called *recipe* and contains configuration and tasks for deployment.
By default all recipes extend the [common](https://github.com/deployphp/deployer/blob/master/recipe/common.php) recipe. 


Defining your task is really simple:
 
```php
task('test', function () {
    writeln('Hello world');
});
```

To run that task, run:

```sh
dep test
```

The output will be:

```text
➤ Executing task test
Hello world
✔ Ok
```

Now lets create some task which will run commands on a remote host. For that we must configure deployer. 
Your newly created `deploy.php` file should contain `host` declaration like this:
 
```php
host('domain.com')
    ->stage('production')    
    ->set('deploy_path', '/var/www/domain.com');
```

> Also it's possible to declare hosts in separate yaml file. Find out more about [inventory](hosts.md#inventory-file).

You can find more about host configuration [here](hosts.md). Now let's define a task which will output a 
`pwd` command from the remote host:
 
```php
task('pwd', function () {
    $result = run('pwd');
    writeln("Current dir: $result");
});
```

Run `dep pwd`, and you will get this:

```text
➤ Executing task pwd
Current dir: /var/www/domain.com
✔ Ok
```

Now lets prepare for our first deploy. You need to configure parameters such as `repository`, `shared_files,` and others:
   
```php
set('repository', 'git@domain.com:username/repository.git');
set('shared_files', [...]);
```

You can return the parameter values in each task using the `get` function. 
Also you can override each configuration for each host:

```php
host('domain.com')
    ...
    ->set('shared_files', [...]);
```

More about configuration can be found [here](configuration.md).


Now let's deploy our application:
 
```sh
dep deploy
```

To include extra details in the output, you can increase verbosity with the `--verbose` option: 

* `-v`  for normal output,
* `-vv`  for more verbose output,
* `-vvv`  for debug.
 
Deployer will create the following directories on the host:

* `releases`  contains releases dirs,
* `shared` contains shared files and dirs,
* `current` symlink to current release.

Configure your hosts to serve your public directory from `current`.

> Note that deployer uses [ACL](https://en.wikipedia.org/wiki/Access_control_list) by default for setting up permissions.
> You can change this behavior with `writable_mode` config.    

By default deployer keeps last 5 releases, but you can increase this number by modifying the associated parameter:
 
```php
set('keep_releases', 10);
```

If there is an error in the deployment process, or something is wrong with your new release, 
simply run the following command to rollback to the previous working release:

```sh
dep rollback
```

You may want to run some task before/after another tasks. Configuring that is really simple!

Let's reload php-fpm after `deploy` finishes:

```php
task('reload:php-fpm', function () {
    run('sudo /usr/sbin/service php7-fpm reload');
});

after('deploy', 'reload:php-fpm');
```

If you need connect to host, Deployer has shortcut for faster access:

~~~sh
dep ssh
~~~

This command will connect to selected host and cd to `current_path`.

Read more about [configuring](configuration.md) deploy. 
