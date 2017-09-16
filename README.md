How to make doctrine entity class extendable?
=============

One of the so called drawback of Symfony - Doctrine is that it doesn't allow entity class to be extendable (Read 'Entities & Entity Mapping' with [Symfony documentation](https://symfony.com/doc/current/bundles/override.html). There are other solutions of the same like [Mapped Superclasses](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/inheritance-mapping.html) but that tend to be somewhat difficult, limited and less flexible(?). So, I have tried to device a simple solution which follows.

The very idea of this article is to create such architecture with Symfony to have doctrine entity class extendable.

Basically, I have used symfony features provided for [doctrine](http://symfony.com/doc/current/reference/configuration/doctrine.html) in combination of PHP [traits](http://php.net/manual/en/language.oop5.traits.php) to achieve this.

Steps
-------------

Please follow below steps which will create a extendable doctrine entity called 'User':

Step 1
------------

Add below configuration settings with 'app/config/config.yml':

``` yml
doctrine:
    orm:
        #
        mappings:
            AppEntity:
                type: annotation
                dir: '%kernel.project_dir%/src/Entity'
                is_bundle: false
                prefix: App\Entity
                alias: App
```
This will make Symfony scan 'src/Entity' for doctrine entities. Please [click here](http://symfony.com/doc/current/reference/configuration/doctrine.html#mapping-entities-outside-of-a-bundle) for detailed information.

Step 2
------------

Now create a directory namely 'Entity' in 'src/' directory.

Within 'src/Entity' directory create another directory namely 'App'.

Now within 'src/Entity/App' directory create another directory namely 'Entity'.

Final directory heirarchy will be 'src/Entity/App/Entity' - this is the place where our reside.

Step 3
------------

In 'src/Entity/App/Entity' directory, create a new file 'User.php' - we will create an entity for user:

```php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * User
 *
 * @ORM\Entity()
 */
class User
{

}
```

Please note that this will be empty entity class with no fields and other table related attributes. Later, we will modify this entity class in later step.

Step 4
------------

Now, in 'src/AppBundle/Entity' directory, create a new file 'UserTrait.php' which will be pretty similar to normal entity class but will be a trait class.

```php
namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * UserTrait
 *
 * @ORM\Table(name="user")
 */
Trait UserTrait
{
    /**
     * @var int
     *
     * @ORM\Column(name="id", type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;

    /**
     * @var string
     *
     * @ORM\Column(name="firstname", type="string", length=50, nullable=true)
     */
    private $firstname;

    /**
     * @var string
     *
     * @ORM\Column(name="lastname", type="string", length=50, nullable=true)
     */
    private $lastname;

    /**
     * @var string
     *
     * @ORM\Column(name="email", type="string", length=255, nullable=true)
     */
    private $email;


    /**
     * Get id
     *
     * @return int
     */
    public function getId()
    {
        return $this->id;
    }

    /**
     * Set firstname
     *
     * @param string $firstname
     *
     * @return User
     */
    public function setFirstname($firstname)
    {
        $this->firstname = $firstname;

        return $this;
    }

    /**
     * Get firstname
     *
     * @return string
     */
    public function getFirstname()
    {
        return $this->firstname;
    }

    /**
     * Set lastname
     *
     * @param string $lastname
     *
     * @return User
     */
    public function setLastname($lastname)
    {
        $this->lastname = $lastname;

        return $this;
    }

    /**
     * Get lastname
     *
     * @return string
     */
    public function getLastname()
    {
        return $this->lastname;
    }

    /**
     * Set email
     *
     * @param string $email
     *
     * @return User
     */
    public function setEmail($email)
    {
        $this->email = $email;

        return $this;
    }

    /**
     * Get email
     *
     * @return string
     */
    public function getEmail()
    {
        return $this->email;
    }
}
```

You can notice that it has all field / table definitions.

Step 5
------------

Modify 'src/Entity/App/Entity/User.php' file created with step 3 above as below:

```php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use AppBundle\Entity\UserTrait;

/**
 * User
 *
 * @ORM\Entity()
 */
class User
{
    use UserTrait;
}
```
As you can notice that we have just used the 'UserTrait' class and included with our main entity 'User' class.

Step 6
------------

Run below command and see if 'User' entity works or not and creates the database table with relevant database source:

```bash
php bin/console doctrine:schema:update --dump-sql

php bin/console doctrine:schema:update --force
```

Step 7
------------

Now, you can overwrite any field definition, add new columns like below:

```php
<?php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use AppBundle\Entity\UserTrait;

/**
 * User
 *
 * @ORM\Entity()
 */
class User
{
    use UserTrait;

    /**
     *
     * @var string @ORM\Column(name="firstname", type="string", length=100, nullable=true)
     */
    private $firstname;

    /**
     *
     * @var string @ORM\Column(name="phone", type="string", length=20, nullable=true)
     */
    private $phone;

    /**
     * Set phone
     *
     * @param string $phone
     *
     * @return User
     */
    public function setPhone($phone)
    {
        $this->phone = $phone;

        return $this;
    }

    /**
     * Get phone
     *
     * @return string
     */
    public function getPhone()
    {
        return $this->phone;
    }
}
```

As you can notice that we have increased 'firstname' field length to 100 from 50 characters and also have added a new field 'phone'

Again, run commands with previous step and see if that works or not.

How to handle relations?
------------

Please find the example code below where we have created a new entity called 'Post' and have associated user with it:

src/AppBundle/Entity/PostTrait.php:

```php
namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * PostTrait
 *
 * @ORM\Table(name="post")
 */
Trait PostTrait
{

    /**
     *
     * @var int @ORM\Column(name="id", type="integer")
     *      @ORM\Id
     *      @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;

    /**
     *
     * @var string @ORM\Column(name="title", type="string", length=100, nullable=true)
     */
    private $title;

    /**
     * The user who made the purchase.
     *
     * @var User @ORM\ManyToOne(targetEntity="User")
     */
    protected $user;

    /**
     * Get id
     *
     * @return int
     */
    public function getId()
    {
        return $this->id;
    }

    /**
     * Set title
     *
     * @param string $title
     *
     * @return User
     */
    public function setTitle($title)
    {
        $this->title = $title;

        return $this;
    }

    /**
     * Get title
     *
     * @return string
     */
    public function getTitle()
    {
        return $this->title;
    }

    /**
     *
     * @param User $user
     */
    public function setUser($user)
    {
        $this->user = $user;
    }

    /**
     *
     * @return User
     */
    public function getUser()
    {
        return $this->user;
    }
}
```
src/Entity/App/Entity/Post.php:

```php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use AppBundle\Entity\PostTrait;

/**
 * Post
 *
 * @ORM\Entity()
 */
class Post
{
    use PostTrait;
}
```

What next?
------------

We can have a new command which scans the whole project directory including 'vendor' and then creates such entity classes from its relevant trait classes.

Conclusion
------------

This is the most simple case imagined. We can have more than one trait classes to be included with our main entity and final entity will be created as per the inheritance rules trait is having.

Altough, this works fine, it can be said as workaround and not the solution as far as I can imagine. Hope, Doctrine and Symfony would come up with proper - legitimate solution of the same. Thanks for reading - have a great day!

License
-------

This bundle is under the MIT license. See the complete license [in the bundle](LICENSE)

About
-----

HncExtendableEntityBundle is a [Hiren Chhatbar](https://github.com/hirenchhatbar) initiative.


Reporting an issue or a feature request
---------------------------------------

It may that I have missed something which throws an error so I would request to report them.

Issues and feature requests are tracked in the [Github issue tracker](https://github.com/hirenchhatbar/HncExtendableEntityBundle/issues).

When reporting a bug, it may be a good idea to reproduce it in a basic project
built using the [Symfony Standard Edition](https://github.com/symfony/symfony-standard)
to allow developers of the bundle to reproduce the issue by simply cloning it
and following some steps.