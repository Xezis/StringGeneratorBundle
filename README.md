# StringGeneratorBundle

This bundle allows you to automatically generate a unique random string on an entity property, useful for
creating keys. Using Doctrine's `prePersist` callback, StringGenerator adds the generated string to a property
before the entity is persisted. It also checks whether the string is unique to that property (just in case) and if not, quietly
generates a new string.

## Install

Add `"vivait/string-generator-bundle": "~1.0"` to your composer.json and run `composer update`

Update your `AppKernel`:
```php
public function registerBundles()
{
    $bundles = array(
        ...
        new Vivait\StringGeneratorBundle\VivaitStringGeneratorBundle(),
}
```  
## Configure

The default configuration is shown below:
  
```yaml
vivait_string_generator:
  generators:
    string: vivait_generator.generator.string
    secure_bytes: vivait_generator.generator.secure_bytes
```
### Bundled generators
* `StringGenerator` generates a random string based on a pool or characters
* `SecureBytesGenerator` generates a secure random string using the `Symfony\Component\Security\Core\Util\SecureRandom` class

### Custom generators
You can use your own generators by implementing `GeneratorInterface` and defining the generator in the configuration, 
using either its service or classname.

## Basic usage

Add the `@Generate(generator="generator_name")` annotation to an entity property 
(where `generator_name` is the name of a generator defined in the configuration).

```php
use Vivait\StringGeneratorBundle\Annotation\GeneratorAnnotation as Generate;

/**
 * Api
 *
 * @ORM\Table()
 * @ORM\Entity()
 */
class Api
{
    /**
     * @var integer
     *
     * @ORM\Column(name="id", type="guid")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="UUID")
     */
    private $id;

    /**
     * @var string
     *
     * @ORM\Column(name="api_id", type="string", nullable=false)
     * @Generate(generator="string")
     */
    private $api_key;
```
## Options

### Length

To change the length of the generated string, add `length` to the annotation.
```php
@Generate(length=4)
```  

### Callbacks
    
It's possible to define callbacks on the `Generator` class that you are using. 
For example, with the bundled StringGenerator, you may wish to set the character pool. 

This can be achieved by setting the `callbacks` option. For example:

```php
@Generate(generator="string", callbacks={"setChars"="ABCDEFG"})
```

Here, `setChars()` is called in the `StringGenerator` class, passing `ABCDEFG` as a parameter.

It's even possible to set a callback value dynamically:

```php
/**
 * @var string
 *
 * @ORM\Column(name="short_id", type="string", length=255, nullable=false)
 * @Generate(generator="string", length=5, callbacks={"setPrefix"="getPrefix"})
 */
private $short_id;

public function getPrefix()
{
    return $this->getType(); //"default"
}
```

In this case `StringGenerator::setPrefix("default")` will be called

    
### Unique

Setting `unique` is boolean and tell if the string must be unique or not, by default `true`

```php
@Generate(generator="secure_bytes", unique=false)
```
