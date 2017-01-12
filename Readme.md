# Installation

```bash
curl -s http://robo.l85.eu/robo.phar -o /usr/local/bin/robo
chmod +x /usr/local/bin/robo
```

# Example for RoboFile.php

```php
<?php

class RoboFile extends \Robo\Tasks
{
    use \LarsMalach\Robo\Task\Deployment\loadTasks;

    /**
     * @param string $instanceKey
     * @param array $opt
     * @return void
     */
    public function deploy($instanceKey, array $opt = ['tag|t' => null, 'branch|b' => null])
    {
        $this->stopOnFail();
        $this->deplInit($instanceKey, 'RoboServers.yaml' /* you can use yaml or json files */, $opt);
        $this->collectionBuilder()
            ->addTask($this->deplExecOnServers(new \Robo\Task\Base\Exec('echo "test"')))
            ->addTask($this->deplGitClone())
            ->addTask($this->deplComposerInstall())
            ->addTask($this->deplNpmInstall())
            ->addTask($this->deplBowerInstall())
            ->addTask($this->deplDeployFiles())
            ->addTask($this->deplRemoveLocalTemporaryDirectory())
            ->addTask($this->deplCreateMaintenanceFlag())
            ->addTask($this->deplRelease())
            ->addTask($this->deplMagentoCacheClear())
            ->addTask($this->deplRemoveMaintenanceFlag())
            ->addTask($this->deplRemoveOldReleases())
            ->run();
    }
}
```

Example for RoboServers.yaml:
```yaml
abstract:
  abstract: true
  repository: git@github.com:lars85/robo.git
  webDirecotry: Web
  servers:
    abstract:
      abstract: true
      host: domain.de
      user: www-data

production:
  superTypes: ['abstract']
  branch: master
  context: production
  servers:
    domain.de:
      superTypes: ['abstract']
      path: /var/www/domain.de

stage:
  branch: master
  context: production
  servers:
    stage.domain.de:
      path: /var/www/stage.domain.de
      host: domain.de
      user: www-data

example:
  # abstract can be true or false. If abstract is true, you can not deploy it. Its just there to extend other ones.
  abstract: false
  # extend the current configuration with another ones
  superTypes: ['parent1', 'parent2']
  branch: master
  # tag has higher priority
  tag: 1.2.3
  # context can be 'production' or 'development'
  context: production
  # by default projectName will be generated by the git repository url
  projectName: example
  repository: git@github.com:lars85/robo.git
  # keepReleases is by default 5
  keepReleases: 5
  # webDirectory is by default empty ''. You can use it if the web directory is sub directory of the repository (of example 'Web' oder 'src')
  webDirectory: Web
  # change bower directory. by default in the release directory
  bower.dir: skin/frontend/enterprise/default
  # change npm directory. by default in the release directory
  npm.dir: skin/frontend/enterprise/default
  servers:
    serverName1:
      host: domain.de
      user: www-data
      path: /var/www/domain.de
      abstract: false
      superTypes: []
    serverName2:
      host: domain2.de
      user: www-data
      path: /var/www/domain.de
```

You can see all possible properties in
src/Model/Deployment.php or
src/Model/Server.php

# Build

```bash
vendor/bin/robo phar:build
vendor/bin/robo phar:install
```