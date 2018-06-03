


<div align="center">__Un petit retour d'experiance: comment charger une extension DI symfony__</div>

La création et chrgement d'une extension, pour une architecture de dossiers qui ne suit pas la convention  de suffixer avec `Bundle`, eg: `AppBundle`<br>
Il y a deux solutions à proposer pour aboutir à enregistrer au niveau de la configuration du conteneur <a href="http://api.symfony.com/3.1/Symfony/Component/DependencyInjection/ContainerBuilder.html">containerBuider</a> associé au `kernel` du symfony.

**Pourquoi Les Extensions Symfony ?**

Symfony à la base de ces composant implémentés, comporte des extensions comme celle de FormExtenion, celle du TwigExtension, DoctrineExtension etc ... ,qui héritent chaqune de la classe `Extension`.

Une nouvelle extension va nous permetre, soit de charger des définitions de services à partir de fichiers de configuration soit de définir des services pour une telle application de manière plus **dynamique**. 

Par exemple, je me suis trouvé devant une situation dans laquelle je dois typer (type hint) avec l'une des deux classes abstraites soit `GainMemberAbstrait` soit `BarestoMemberAbstrait`, et là, le choix de la classe souhaitée dépend d'une paramétre pré configuré au niveau de `parameters.yml` , ainsi pour implémenter le besoin, il faudrait créer une extension pour configurer le conteneur syfmony pour qui'l puisse choisir et charger par la suite la classe souhaitée.

**Pourquoi les extensions ne sont pas chargées d’une façon automatique ?**

Parce que, pour aboutir à charger une extension, il fallait suivre un ensemble <a href="https://symfony.com/doc/current/bundles/extension.html#creating-an-extension-class"> de conventions de nommage</a>,  déjà si on se focalise sur la classe abstraite `Symfony/Component/HttpKernel/Bundle/Bundle` qui contient le bout de code sous dessous, responsable aux chargements des extensions à travers la methode `getContainerExtension()` . <br>
Lorsque la compilation du conteneur commence, cette méthode ne reconnaitre que `l'extension` qui se trouve dans le répertoire `DependencyInjection` d'un bundle de tel sorte que son nom doit être construit en remplaçant le suffixe `Bundle` du `nom de la classe AppBundle ` comme exmple par `Extension`.

```php
    protected function getContainerExtensionClass()
    {
        $basename = preg_replace('/Bundle$/', '', $this->getName());
        return $this->getNamespace().'\\DependencyInjection\\'.$basename.'Extension';
    }
```
<br>
Du coup, pour `AppBundle`, la fonction recherche une classe DependencyInjection \ AppExtension. Sauf que cette histoire n'est pas appliquable pour un projet qui ne suit pas l'ensemble des conventions déja mentionnées.


**Les solutions proposées :**


**1/** Le plus simple c'est q'au niveau de la classe `AppKernel.php`, il faudrait enregistrer l’extension à travers la fonction `addExtension()`, ca permet de cette facon de charger l'extension juste après la compélation `container->compile()`... comme Voici l'exemple suivant nous montre:

```php
//AppKernel.php

class AppKernel extends Kernel
{  //
    protected function build(ContainerBuilder $container)
    {
        $container->registerExtension(new \Portal\Common\Infrastructure\Symfony\DependencyInjection\AppExtension());
    }
//
}

```


**2/** Créer une classe qui termine avec le suffixe `Budnle` et qui hérite la classe abstraite `Bundle` et là j'ai suivit la solution proposé par la <a href="https://symfony.com/doc/current/bundles/extension.html#manually-registering-an-extension-class"> documentaion du symfony | Manually Registering an Extension Class </a> et là, il suffit de surcharger la fonction `getContainerExtension`:

```php
//portal/src/Portal/Common/Infrastructure/Symfony 

namespace Portal\Common\Infrastructure\Symfony;

use Portal\Common\Infrastructure\Symfony\DependencyInjection\AppContextExtension;
use Symfony\Component\HttpKernel\Bundle\Bundle;

class AppContextBundle extends Bundle
{
    public function getContainerExtension()
    {
        return new AppContextExtension();
    }
}
```

L'extension:

```php
namespace Portal\Common\Infrastructure\Symfony\DependencyInjection;

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Extension\Extension;

class AppContextExtension extends Extension
{
    public function load(array $configs, ContainerBuilder $container)
    {
        dump('it work');die;
    }
}
```
Après il suffit d'enregistrer la classe.
config/bundles.php
```php
Portal\Common\Infrastructure\Symfony\AppContextBundle::class => ['all' => true],
```
