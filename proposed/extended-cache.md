Common Interface for Caching libraries
====================================

## Proposed extended implementation

This documents adds missing interfaces from the original cache proposal
allowing for more functionality that originaly specified.

### CacheItemInterface

```php

<?php

namespace PSR\Cache\Item;

/**
 * Interface for caching object
 */
interface AdvancedCacheItemInterface extends CacheItemInterface
{
    /**
     * Get the namespace of the cache item
     *
     * @return string
     */
    public function getNamespace();

    /**
     * Set the namespace of the cache driver
     *
     * @param string $namespace
     *
     * @return CacheItemInterface
     */
    public function setNamespace($namespace);

    /**
     * Get the tags of an item
     *
     * @return string[]
     */
    public function getTags();

    /**
     * Set the tags of an item
     *
     * @param array $tags
     *
     * @return CacheItemInterface
     */
    public function setTags(array $tags);

}

```

### AdvancedDriverInterface

```php

<?php

namespace PSR\Cache\Driver;

/**
 * Interface for advanced cache drivers
 */
interface AdvancedDriverInterface extends DriverInterface
{
    /**
     * Lock a certain entry for the specified amount of time.
     *
     * @param string $item
     * @param int    $lifeTime Life time of the cache entry
     *
     * @return boolean Result of the operation
     */
    public function lock($item, $lifeTime = 0);

    /**
     * Unlock the specified entry
     *
     * @param string $item
     *
     * @return boolean Result of the operation
     */
    public function unlock($item);

    /**
     * Get items that match the specified tag
     *
     * @param string $tag Tag name
     *
     * @return array
     */
    public function getByTag($tag);

    /**
     * Get items that match all the tags list.
     * If the $mustHaveAll is set to false then any item matching
     * any tag from the list will be returned
     *
     * @param string[] $tags
     *
     * @return array
     */
    public function getByTags($tags, $mustHaveAll = true);

    /**
     * Remove all the items that match the specified tag
     *
     * @param string $tag
     */
    public function removeByTag($tag);

    /**
     * Remove items that match the specified tag
     *
     * @param string[] $tags
     */
    public function removeByTags($tags, $mustHaveAll = true);

    /**
     * Get items that match the specified namespace
     * If $includingChildren is set to true then the all the children from
     * the subnamespaces will be retrieved as well
     *
     * @param string $namespace Namespace
     *
     * @return array
     */
    public function getByNamespace($namespace, $includingChildren = false);

    /**
     * Remove items that match the specified namespace
     * If $includingChildren is set to true then the all the children from
     * the subnamespaces will be removed as well
     *
     * @param string[] $tags
     */
    public function removeByNamespace($namespace, $includingChildren = false);

}

```

### AdvancedCacheProxyInterface

```php

<?php

namespace PSR\Cache;

use PSR\Cache\Item\AdvancedCacheItemInterface;

/**
 * This is our advanced cache proxy
 */
class AdvancedCacheProxyInterface
{
    /**
     * Lock a certain entry for the specified amount of time.
     *
     * @param CacheItemInterface $item
     * @param int                $lifeTime Life time of the cache entry
     *
     * @return boolean Result of the operation
     */
    public function lock(CacheItemInterface $item, $lifeTime = 0);

    /**
     * Unlock the specified entry
     *
     * @param CacheItemInterface $item
     *
     * @return boolean Result of the operation
     */
    public function unlock(CacheItemInterface $item);

    /**
     * Get items that match the specified tag
     *
     * @param string $tag Tag name
     *
     * @return AdvancedCacheItemInterface[]
     */
    public function getByTag($tag);

    /**
     * Get items that match all the tags list.
     * If the $mustHaveAll is set to false then any item matching
     * any tag from the list will be returned
     *
     * @param string[] $tags
     *
     * @return AdvancedCacheItemInterface[]
     */
    public function getByTags($tags, $mustHaveAll = true);

    /**
     * Remove all the items that match the specified tag
     *
     * @param string $tag
     */
    public function removeByTag($tag);

    /**
     * Remove items that match the specified tag
     *
     * @param string[] $tags
     */
    public function removeByTags($tags, $mustHaveAll = true);

    /**
     * Get items that match the specified namespace
     * If $includingChildren is set to true then the all the children from
     * the subnamespaces will be retrieved as well
     *
     * @param string $namespace Namespace
     *
     * @return AdvancedCacheItemInterface[]
     */
    public function getByNamespace($namespace, $includingChildren = false);

    /**
     * Remove items that match the specified namespace
     * If $includingChildren is set to true then the all the children from
     * the subnamespaces will be removed as well
     *
     * @param string[] $tags
     */
    public function removeByNamespace($namespace, $includingChildren = false);

}

```