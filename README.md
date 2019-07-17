# Symfony migration from 2.8 to 3.4 LTS



- [Symfony migration from 2.8 to 3.4 LTS](#symfony-migration-from-28-to-34-lts)
    + [Put-in](#put-in)
    + [Actual Versions](#actual-versions)
    + [Resolve duplication](#resolve-duplication)
      - [- PHP](#--php)
      - [- SYMFONY](#--symfony)
      - [- PHPunit](#--phpunit)
          + [examples](#examples)
      - [- Bundles and libraries](#--bundles-and-libraries)
        * [- FosRestBundle](#--fosrestbundle)
  * [Todo list (Symfony)](#todo-list--symfony-)
      - [- Change dependencies version](#--change-dependencies-version)
      - [- Move logs and cache directories](#--move-logs-and-cache-directories)
      - [- Implicit methods override in the AppKernel class](#--implicit-methods-override-in-the-appkernel-class)
      - [- Move console to bin folder](#--move-console-to-bin-folder)
      - [- Quote service injection in the yaml](#--quote-service-injection-in-the-yaml)
      - [- Remove the webconfigurator](#--remove-the-webconfigurator)
      - [- Remove autoload and bootstrap files](#--remove-autoload-and-bootstrap-files)
      - [- Resolve repository configuration issue](#--resolve-repository-configuration-issue)
      - [- Move phpunit config file and tests](#--move-phpunit-config-file-and-tests)
      - [- Move SymfonyRequirements](#--move-symfonyrequirements)
      - [- Enabling annotations in sension_extra_bundle](#--enabling-annotations-in-sension-extra-bundle)
      - [- Update session configuration](#--update-session-configuration)
      - [- Update .gitignore](#--update-gitignore)
      - [- Quote paramaters in config.yml](#--quote-paramaters-in-configyml)
      - [- Resolve Env variable](#--resolve-env-variable)
      - [- Fixing the trusted_proxies](#--fixing-the-trusted-proxies)
      - [- Changing pattern by path](#--changing-pattern-by-path)
      - [- Bundle inheritance](#--bundle-inheritance)
          + [example](#example)
      - [- Authentication](#--authentication)
      - [- Resolve logout_on_user_change warning](#--resolve-logout-on-user-change-warning)
      - [- Update Composer.json](#--update-composerjson)
  * [Todo list (bundles and Libraries)](#todo-list--bundles-and-libraries-)
      - [- VichUploaderBundle](#--vichuploaderbundle)
      - [- JmsSerializer](#--jmsserializer)
      - [- google/api-client (Cloud Storage)](#--google-api-client--cloud-storage-)
  * [Launsh scripts](#launsh-scripts)
      - [composer install](#composer-install)
      - [launsh tests](#launsh-tests)
  * [Untreated  parts](#untreated--parts)
  * [Contributing](#contributing)




### Put-in
This is a symfony guide describing the migration from the version 2.8 lts to the version 3.8 lts .

The whole migration is separated into the following important part :

- Resolve Deprecated for Symfony 
- Resolve deprecated for PHP 
- Upgrade PHP version 
- Resolve dependencies version
- Change project folder structure
- Adapte configuration for new version symfony and other bundles.
- Upgrade Symfony framework with dependencies 
- Upgrade rest of others third parties (libraries and bundles)


### Actual Versions

This is the list of actual versions after migration  

|           |version |
|-----------|--------|
|  PHP      | 7.2.18 |
|  SYMFONY  | 3.4.27 |
|  PHPUnit  | 6.5.14 |



### Resolve duplication

#### - PHP 


| function | substitute | description |
|-----------|-----------|-------------|
| create_function | anonymous function| the create_function is deprecated since PHP 7.2 | 
| count| count | count don't support null in php 7.2|



#### - SYMFONY


| function | substitute | description |
|-----------|-----------|-------------|
| getRequest | getRequest| getting request from container is deprecated in 2.8 and will be removed in 3.0 | 
| addViolationAt | buildViolation | addViolation and addViolationAt are deprecated from 2.5 and will be removed until 3.0 |


- getRequest

_before_


~~~
/*
 * ...
 * @deprecated Deprecated since version 2.4, to be removed in 3.0. Ask
 *             Symfony to inject the Request object into your controller
 *             method instead by type hinting it in the method's signature.
 */
public function getRequest()
{
    return $this->container->get('request_stack')->getCurrentRequest();
}

// controller code 

 public function putLanguageTranslationsLabelAction($lang){
 
     $value = $this->getRequest()->get('translation')['value'];

 }


~~~

_after_


It's recommended to inject the request object instead 


~~~

// controller code 
 use Symfony\Component\HttpFoundation\Request;
 ...
 
 public function putLanguageTranslationsLabelAction(Request $request , $lang){
 
     $value = $request->get('translation')['value'];

 }

~~~

*for more , see the [link](https://stackoverflow.com/questions/20984816/what-is-the-best-way-to-get-the-request-object-in-the-controller)*


- addViolationAt


_before_

~~~
$context->addViolationAt('file', 'You must choose a file for your document', [], null);
~~~

_after_

~~~
$context->buildViolation('You must choose a file for your document', ['file']);
~~~


*for more details , see the [link](https://stackoverflow.com/questions/25264922/symfony-2-5-addviolationat-deprecated-use-buildviolation)*




| class / interface | substitute | description |
|-----------|-----------|-------------|
| ContainerAware | ContainerAwareTrait| ContainerAware has been deprecated in 2.8 and removed in 3.0 in favor of ContainerAwareTrait | 
| ValidatorInterface| ValidatorInterface | the namespace is changed to  **Symfony\Component\Validator\Validator\ValidatorInterface** instead of **~~Symfony\Component\Validator\ValidatorInterface~~**|
| ExecutionContextInterface | ExecutionContextInterface| the namespace is changed to **Symfony\Component\Validator\Context\ExecutionContextInterface** instead of **~~Symfony\Component\Validator\ExecutionContextInterface~~** |
| SecurityContext | Security | the class changed to **use Symfony\Component\Security\Core\Security** instead of **~~Symfony\Component\Security\Core\SecurityContext~~** | 
| SecurityContext| TokenStorageInterface| in some context like getting the current user , use  TokenStorageInterface instead|
| DefinitionDecorator(for symfony 4)| ChildDefinition | the DefinitionDecorator is deprecated and will be removed on symfony 4|

- ContainerAware


_before_

~~~
use Symfony\Component\DependencyInjection\ContainerAware
...

class MyClass extends ContainerAware
 {
  ...
 }
~~~

_after_

~~~
use Symfony\Component\DependencyInjection\ContainerAwareTrait
...

class MyClass  
 {
  use ContainerAwareTrait;
  
  ...
 }
~~~


- SecurityContext


_before_

~~~
public function setSecurityContext(SecurityContext $securityContext) {
      if ($securityContext->getToken() !== null) {
            $this->user = $securityContext->getToken()->getUser();
         }
         ...         
}
~~~

_after_

~~~
use Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorageInterface;

public function setSecurityContext(TokenStorageInterface $tokenStorage) {
      if ($tokenStorage->getToken() !== null) {
            $this->user = $tokenStorage->getToken()->getUser();
         }
     }
~~~

#### - PHPunit

| function | substitute | description |
|-----------|-----------|-------------|
| setExpectedException | expectException/ expectExceptionMessage| the second parameter contains the exception message, this was removed in PHPUnit 6,you should use expectExceptionMessage method instead. | 
| getMock| getMockBuilder | count don't support null in php 7.2|


###### examples


- setExpectedException

_before_

~~~~

$this->setExpectedException('Symfony\Component\HttpKernel\Exception\HttpException', 'Email group with the same name already exists');
~~~~

_after_

~~~~
$this->expectException('Symfony\Component\HttpKernel\Exception\HttpException');
$this->expectExceptionMessage('Email group with the same name already exists');
~~~~

- getMock

_before_

~~~
$this->reporterMock = $this->getMock('MyApp\AppBundle\Report\Reporter');
~~~

_after_

~~~

$this->reporterMock = $this->getMockBuilder('MyApp\AppBundle\Report\Reporter')
->disableOriginalConstructor()
->getMock();
~~~


#### - Bundles and libraries


##### - FosRestBundle


| function | substitute | description |
|-----------|-----------|-------------|
| setSerializationContext | setContext| [see more](https://github.com/FriendsOfSymfony/FOSRestBundle/blob/master/UPGRADING-2.0.md) | 


## Todo list (Symfony)

#### - Change dependencies version 

It's crurial to check all dependencies compatibility to avoid conflict . 

For the current project , here is the list of dependencies version:

~~~
{
    "require": {
        "php": ">=5.5.9",
        "symfony/symfony": "3.4.*",
        "doctrine/orm": "2.5.*",
        "doctrine/doctrine-bundle": "1.6.*",
        "doctrine/dbal": "2.5.*",
        "twig/twig": "^1.0",
        "twig/extensions": "1.5.4",
        "symfony/assetic-bundle": "2.8.*",
        "symfony/swiftmailer-bundle": "~2.6.4",
        "symfony/monolog-bundle": "~3.1.0",
        "sensio/distribution-bundle": "~5.0.19",
        "sensio/framework-extra-bundle": "~5.1.0",
        "incenteev/composer-parameter-handler": "~2.0",
        "friendsofsymfony/rest-bundle": "2.2.0",
        "nelmio/api-doc-bundle": "2.*",
        "doctrine/migrations": "1.5.0",
        "doctrine/doctrine-migrations-bundle": "1.3.*",
        "jms/serializer-bundle": "2.4.4",
        "jms/serializer": "1.10.0",
        "jms/metadata": "1.7.0",
        "jms/parser-lib": "1.0.0",
        "doctrine/doctrine-fixtures-bundle": "2.4.1",
        "leaseweb/api-caller-bundle": "1.2.11",
        "vich/uploader-bundle": "1.6.*",
        "knplabs/knp-gaufrette-bundle" : "~0.5",
        "stof/doctrine-extensions-bundle" : "~1.3.0",
        "nelmio/cors-bundle": "^1.5",
        "google/apiclient": "2.2.*"
    },
    "require-dev": {
        "behat/symfony2-extension": "*",
        "behat/mink-extension": "*",
        "behat/mink-browserkit-driver":"*",
        "mikey179/vfsstream": "~1.4.0",
        "symfony/phpunit-bridge": "^4.2",
        "sensio/generator-bundle": "^3.0"
    },

}
~~~

#### - Move logs and cache directories

Cache and logs folders are available on the var folder at the root of a symfony project 3 , so , it need to remove them from the old place **app**.





#### - Implicit methods override in the AppKernel class

In the AppKernel class , it necessary to override some kernel methods as : getRootDir & getLogDir & getCacheDir since the cache and logs folders  already moved to a new place.


 ~~~
 public function getRootDir()
     {
         return __DIR__;
     }
     public function getCacheDir()
     {
         return dirname(__DIR__).'/var/cache/'.$this->getEnvironment();
     }
     public function getLogDir()
     {
         return dirname(__DIR__).'/var/logs';
     }
 ~~~

#### - Move console to bin folder
The console is now available in bin folder.

Since the autoload is managed by composer , don't forget to subtitute : 

~~~
 require_once __DIR__.'/bootstrap.php.cache'; 
~~~
with
~~~
require __DIR__.'/../vendor/autoload.php';
~~~


#### - Quote service injection in the yaml 

In symfony 3 , service must be quoted  and try to avoid camelCase is service id and use underscore or dot instead  . 

_before_
~~~
myImporter:
        class: App\MyBundle\Plugins\Importer
        arguments: [@decoder, @submitter, @reader]
        calls:
          - ['addListener', [ @importation_reporter ] ]
          - ['setLogger', [ @logger.service ] ]
~~~

_After_

~~~
    my_importer:
        class: App\MyBundle\Plugins\Importer
        arguments: ["@decoder", "@submitter", "@reader"]
        calls:
          - ['addListener', [ "@importation_reporter" ] ]
          - ['setLogger', [ "@logger.service" ] ]    
~~~

[watch more ](https://symfony.com/blog/new-in-symfony-2-8-yaml-deprecations)

#### - Remove the webconfigurator

Delete the webconfigurator route from `app/config/routing_dev.yml` as this doesn't exists anymore (since v4 of DistributionBundle?)

~~~
# to delete
_configurator:
    resource: "@SensioDistributionBundle/Resources/config/routing/webconfigurator.xml"
    prefix:   /_configurator
~~~


#### - Remove autoload and bootstrap files 


Remove  `bootstap.php.cache` and `app/autoload.php` from `app` folder  and run 
` composer dump-autoload -o -a` to generate the `autoload file` in vendor directory.


#### - Resolve repository configuration issue 

may be you will see this awesome error 
~~~
Type error: Too few arguments to function Doctrine\ORM\EntityRepository::__construct(), 1 passed in /Users/.../var/cache/dev/appDevDebugProjectContainer.php on line 3434 and exactly 2 expected
~~~

It's caused because your yaml config seems te be something like :


~~~
mail.repository:
    class: App\MyBundle\Repository\MailRepository
    factory_service: doctrine.orm.default_entity_manager
    factory_method: getRepository
    arguments: ['MyCustomBundle:User']
~~~

you need to change it to be as the example bellow : 


~~~

mail.repository:
    class: App\MyBundle\Repository\MailRepository
    factory: ['@doctrine.orm.entity_manager', getRepository]
    arguments: ['MyCustomBundle:User']
    
~~~


#### - Move phpunit config file and tests 

- Move `phpunit.xml.dist` from `app` to root folder 

- Move all tests from `src/*Bundle/Tests/*` to `tests/*Bundle/*` , and don't forget to update namespaces . 

After , update the phpunit.xml.dist to point to the new tests folder

_before_

~~~
<phpunit  bootstrap = "bootstrap.php.cache" >
    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>../src/*/*Bundle/Tests</directory>
            <directory>../src/*/Bundle/*Bundle/Tests</directory>
        </testsuite>
    </testsuites>

</phpunit>

~~~
_after_

~~~
<phpunit bootstrap = "vendor/autoload.php" >
    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>tests</directory>
        </testsuite>
    </testsuites>
    
    <php>
        <server name="KERNEL_CLASS" value="AppKernel" />
    </php>
    
    <filter>
        <whitelist processUncoveredFilesFromWhitelist="true">
            <directory>src</directory>
            <exclude>
                <directory>vendor/*</directory>
                <directory>src/**</directory>
            </exclude>
        </whitelist>
    </filter>
    
</phpunit>
~~~

**It is required to declare the path to the application (AppKernel)**


#### - Move SymfonyRequirements


Move `app/SymfonyRequirements.php` to `var/SymfonyRequirements.php`


#### - Enabling annotations in sension_extra_bundle

Go to ` config.yml` , and  modify  sensio_framework_extra config : 

_before_ 


~~~
sensio_framework_extra:
    view:
        annotations: false
~~~


_after_


~~~
sensio_framework_extra:
    view:
        annotations: true
~~~

#### - Update session configuration
- Create sessions folder inside var folder .
- Update `config.yml` bellow the framework configuration :

~~~
framework:
# [...]
    session:
        handler_id:  session.handler.native_file
        save_path:   "%kernel.root_dir%/../var/sessions/%kernel.environment%"
~~~
- Need to commit the sessions folder inside the `var` folder after `.gitignore` modification

#### - Update .gitignore


Update `.gitignore` file using Symfony3 [one](https://github.com/symfony/symfony-standard/blob/3.4/.gitignore)



#### - Quote paramaters in config.yml

Parameters in config.yml should be double quoted , and replace single one by double.

_before_


~~~
doctrine:
    dbal:
        driver:   %database_driver%
        host:     %database_host%
        port:     %database_port%
        dbname:   %database_name%
        user:     %database_user%
        password: %database_password%
~~~

_after_
~~~
doctrine:
    dbal:
        driver:   "%database_driver%"
        host:     "%database_host%"
        port:     "%database_port%"
        dbname:   "%database_name%"
        user:     "%database_user%"
        password: "%database_password%"
~~~


#### - Resolve Env variable 

Symfony env variable should be declared with the SYMFONY__ preffix .


_before_

~~~
# vhosts.conf
SetEnv SYMFONY__MYFM__DATABASE__HOST     ${SYMFONY__MYFM__DATABASE__HOST}
~~~

~~~
# Dockerfile
ENV SYMFONY__MYFM__DATABASE__HOST   db
~~~

_after_

~~~
# vhosts.conf
SetEnv MYFM__DATABASE__HOST     ${MYFM__DATABASE__HOST}
~~~

~~~
# Dockerfile
ENV MYFM__DATABASE__HOST   db
~~~

[watch more](https://symfony.com/doc/current/configuration/environment_variables.html)
#### - Fixing the trusted_proxies


_before_

~~~
# app/config.yml
framework:
    trusted_proxies: ~
~~~

_after_


~~~
# web/app.php
Request::setTrustedProxies([]);

~~~
[watch more](https://symfony.com/blog/fixing-the-trusted-proxies-configuration-for-symfony-3-3)



#### - Changing pattern by path 

`pattern`  is deprecated , and should be replaced by path

_before_

~~~
acme_hello_namespace_homepage:
    pattern:  /hellons/{name}
    defaults: { _controller: AcmeHelloNamespaceBundle:Default:index }
~~~

_after_


~~~
hello:
    path:     /hellons/{name}
    defaults: { _controller: AcmeHelloNamespaceBundle:Hello:index }
~~~

[watch more](https://github.com/symfony/symfony/pull/6738/files#diff-0)


#### - Bundle inheritance

Bundle Inheritance is deprecated in Symfony 3.4 and it will be removed in Symfony 4.
###### example
Inheriting  NelmioApiDocBundle in order to override  controller behavior.


- Declaring a route with the same path to the view with the custom controller as the processing controller, the route should be declared before the bundle route.

- The getParent method should be removed in the inheriting Bundle. More details [here](https://symfony.com/doc/current/bundles/override.html)


[watch more](https://symfony.com/blog/new-in-symfony-3-4-deprecated-bundle-inheritance)


#### - Authentication

In case you are using `abstractGuardAuthenticator` , in symfony 2.8 , the methods `getCredentials` was called on every request , till the new version 3.4 , the method will be replaced with supports method (it will be removed on symfony 4) that will do the job of `getCredentials` .

_before_ 

~~~
  public function getCredentials(Request $request)
    {
        if (!$token = $request->headers->get(Constants::TOKEN_PARAMETER_NAME_KEY)) {
            // No token?
            return null;
        }
        return [
            Constants::TOKEN_KEY => $token
        ];
    } 
~~~

_after_

~~~
public function getCredentials(Request $request)
{
    return [
        Constants::TOKEN_KEY => $request->headers->get(Constants::TOKEN_PARAMETER_NAME_KEY)
    ];
}

public function supports(Request $request)
{
    return $request->headers->has(Constants::TOKEN_PARAMETER_NAME_KEY);
}
~~~
[watch more ](https://symfony.com/blog/new-in-symfony-3-4-guard-authentication-improvements)

#### - Resolve logout_on_user_change warning 


~~~
Not setting "logout_on_user_change" to true on firewall "name of your firewall" is deprecated as of 3.4, it will always be true in 4.0.
~~~ 
Copy that key. Then, open `app/config/security.yml`. Under the `"name of your firewall"`, paste this and set it to true.
~~~
firewalls:
     main:
        logout_on_user_change: true
~~~

In case you have more than one firewall paste it all over.

So, what the heck is this? Check it out: suppose you change your password while on your work computer. Previously, doing that did not cause you to be logged out on any other computers, like on your home computer. This was a security flaw, and the behavior was changed in Symfony 4. By turning this on, you can test to make sure your app doesn't have any surprises with that behavior.

[watch more](https://github.com/symfony/symfony-docs/issues/8428)

#### - Update Composer.json


- You  need to remove `bin-dir` property from config section.


~~~
# composer.json
...
"config": {
        "bin-dir": "bin"
    }
~~~

Add instead :

~~~
# composer.json
...
"config": {
        "platform": {
            "php": "7.2.17" // depend on your php version
        }
 }
~~~
[more about platfom config](https://getcomposer.org/doc/06-config.md#platform)

- AppKernel is autoloaded in composer 

~~~
# composer.json
...
"autoload": {
        "psr-4": {
            "": "src/"
        },
        "classmap": [ "app/AppKernel.php", "app/AppCache.php" ]
    },
    "autoload-dev": {
        "psr-4": { "Tests\\": "tests/" },
        "files": [ "vendor/symfony/symfony/src/Symfony/Component/VarDumper/Resources/functions/dump.php" ]
    },
...    
~~~

- Add extra section 

~~~
# composer.json
...
"extra": {
    "symfony-app-dir": "app",
    "symfony-bin-dir": "bin",
    "symfony-var-dir": "var",
    "symfony-web-dir": "web",
    "symfony-tests-dir": "tests",
    "symfony-assets-install": "relative",
    "branch-alias": {
        "dev-master": "3.4-dev"
    }
}
~~~

[watch more](https://github.com/symfony/symfony-standard/blob/3.4/composer.json)


## Todo list (bundles and Libraries)

Besides the above modifications, third party bundle's configurations has to be updated in order to be compliant with the new versions of dependencies:



#### - VichUploaderBundle


_before_
~~~
vich_uploader:
 db_driver: orm
 gaufrette: true
 storage: vich_uploader.storage.gaufrette
~~~
_after_
~~~
vich_uploader:
 db_driver: orm
 storage: gaufrette
~~~


#### - JmsSerializer

Changed the default datetime format from `ISO8601` (Y-m-d\TH:i:sO) to `RFC3339` (Y-m-d\TH:i:sP) 

~~~
jms_serializer:
  handlers:
    datetime:
      default_format: "Y-m-d\\TH:i:sO" # ISO8601
~~~

#### - google/api-client (Cloud Storage)

_before_

~~~
// setting the service account credentials
$credential = $client->loadServiceAccountJson($this->configuration['credentials_path'], [Google_Service_Storage::DEVSTORAGE_READ_WRITE]);
$client->setAssertionCredentials($credential);
~~~

_after_

~~~
// setting the service account credentials
$client->setAuthConfig($this->configuration['credentials_path']);
$client->setScopes([Google_Service_Storage::DEVSTORAGE_READ_WRITE]);
~~~



## Launsh scripts

#### composer install

~~~
composer install -o -a 
~~~


#### launsh tests


To execute test , you have to launch the : 

`./vendor/bin/simle-phpunit
`
This will create a symbolic link in bin folder for the first time to be able to execute test by typing  `bin/simple-phpunit` for the previous tests.



## Untreated  parts

The example of the current migration is based on symfony api project , thus , it still some parts that are not mentionned as forms .

I suggest for you this [guidance](https://gist.github.com/mickaelandrieu/5d27a2ffafcbdd64912f549aaf2a6df9) that can help you in this part.

After finishing this guide , it's not sure that you application must run correctly , may be your own logic can't go with so **dig and dig** .


## Contributing

Feel free to help us to improve this source for cover more . 

## Author
---

This support is done with the collaboration :

[EL AMEL ABDELKARIM](https://github.com/aelamel)

[CHABLI MOHAMMED YASSINE](https://github.com/yassinechabli)

