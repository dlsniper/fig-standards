Common Extended Interfaces for Caching libraries
====================================

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119][].

The final implementations MAY be able to decorate the objects with more
functionality that the one proposed but they MUST implement the indicated
interfaces/functionality first.

[RFC 2119]: http://tools.ietf.org/html/rfc2119

1. Description
-----------------

This documents adds more interfaces that caching libraries can implement in
order to further enhance their functionality.

2. Specifications
-----------------

In order to cover multiple levels of functionality, each additional
functionality is described as separate interface.

### 2.1 TaggableItemInterface

This allows a use to tag a cache ```Item``` with additional metadata in order
to facilitate various operations and usages such as: grouping items by labels,
finding similar items based on labels, removing items based on said labels.

### 2.2 NamespacedItemInterface

This allows grouping a cache ```Item``` via a tree-like structure.

An example of a namespace could be: ```/Store/Product/``` with an item stored
as this: ```/Store/Product/ProductID```.

### 2.3 LockableCacheInterface

In order to facilitate the operations on cache items that need to have
consistency while in high concurancy environments it is recommended to have a
the items that are changed locked to modifications from other instances.

A lock can be defined as:
- read-only lock: this type of lock allows other intances to read the
  information from the item while it is locked for writing.

- read-write lock: this type of lock denies both reading or writing of the
  item while it is locked.

If the storage engine does not provide this functionality, then the
implementation MUST provide emulation for it.

The lock duration is specified in microseconds as it allows better user
controlled granularity.

### 2.4 NamespacedCacheInterface

This interface facilitate operations with namespaces if the driver has support
for them or wants to emulate such support.

The getter method will retrieve the requested namespace items without children
namespaces/items by default. The user MUST explicitly request those in order to
retrieve them as doing so could potentially be a performance problem.

The same applies to the removing method which will only delete the items in the
specified namespace unless otherwise requested by user.

The item returned by this interface must implement the ```NamespacedItemInterface```

### 2.5 TaggableCacheInterface

This interface facilitate operations with tags if the driver has support for
them or wants to emulate such support.

By default, the tag matching will be done in ```all``` mode, which means that
the retrieved items have at least all tags requested by the user. The retrieved
items MAY contain other tags as well but MUST have all the tags requested by
user. If the user wants to retrieve all items that match any of the requested
tags then he MUST explicitely do so.

The same applies to the removing method which will delete the items that match
at least all the specified tags unless otherwise requested by user to delete
all the items that match any tag.

3. Interfaces
----------

### 3.1 NamespacedItemInterface

```php

<?php

namespace Psr\Cache;

/**
 * CacheItem with tag support
 */
interface NamespacedItemInterface extends ItemInterface
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
     */
    public function setNamespace($namespace);

}

```

### 3.2 TaggableItemInterface

```php

<?php

namespace Psr\Cache;

/**
 * CacheItem with tag support
 */
interface TaggableItemInterface extends ItemInterface
{
    /**
     * Get the tags of an item
     *
     * @return string[]
     */
    public function getTags();

    /**
     * Set the tags of an item
     *
     * @param string[] $tags
     */
    public function setTags(array $tags);

}

```

### 3.3 LockableCacheInterface

```php

<?php

namespace Psr\Cache;

/**
 * Lock support for cache driver
 */
interface LockableCacheInterface extends DriverInterface
{
    const READ_ONLY_LOCK = 1;
    const READ_WRITE_LOCK = 2;

    /**
     * Lock a certain entry for the specified amount of time in microseconds
     *
     * @param string $item
     * @param int    $lifeTime
     * @param int    $lockType
     *
     * @return Boolean
     */
    public function lock($item, $lifeTime, $lockType = LockableCacheInterface::READ_ONLY_LOCK);

    /**
     * Unlock the specified entry
     *
     * @param string $item
     *
     * @return Boolean
     */
    public function unlock($item);

}

```

### 3.4 NamespacedCacheInterface

```php

<?php

namespace Psr\Cache;

/**
 * Namespace support for cache driver
 */
interface NamespacedCacheInterface extends DriverInterface
{
    /**
     * Set the namespace separator used by the cache driver
     *
     * @param $separator
     */
    public function setNamespaceSeparator($separator);

    /**
     * Get the namespace separator used by the cache driver
     *
     * @return $string
     */
    public function getNamespaceSeparator();

    /**
     * Get items that match the specified namespace
     * If $includingChildren is set to true then the all the children from
     * the subnamespaces will be retrieved as well
     *
     * @param string  $namespace
     * @param Boolean $includingChildren
     *
     * @return NamespacedItemInterface[]
     */
    public function getByNamespace($namespace, $includingChildren = false);

    /**
     * Remove items that match the specified namespace
     * If $includingChildren is set to true then the all the children from
     * the subnamespaces will be removed as well
     *
     * @param string  $namespace
     * @param Boolean $includingChildren
     */
    public function removeByNamespace($namespace, $includingChildren = false);

}

```

### 3.5 TaggableCacheInterface

```php

<?php

namespace Psr\Cache;

/**
 * Tag support for cache driver
 */
interface TaggableCacheInterface extends DriverInterface
{
    /**
     * Get items that match the specified tag
     *
     * @param string $tag Tag name
     *
     * @return TaggableItemInterface[]
     */
    public function getByTag($tag);

    /**
     * Get items that match all the tags list.
     * If the $mustHaveAll is set to false then any item matching
     * any tag from the list will be returned
     *
     * @param string[] $tags
     * @param Boolean  $mustHaveAll
     *
     * @return TaggableItemInterface[]
     */
    public function getByTags(array $tags = array(), $mustHaveAll = true);

    /**
     * Remove all the items that match the specified tag
     *
     * @param string $tag
     *
     * @param Boolean
     */
    public function removeByTag($tag);

    /**
     * Remove items that match the specified tag
     *
     * @param string[] $tags
     * @param Boolean  $mustHaveAll
     *
     * @return Boolean[]
     */
    public function removeByTags(array $tags = array(), $mustHaveAll = true);

    /**
     * Returns the number of items that match the specified tag
     *
     * @param string $tag
     *
     * @return int
     */
    public function countByTag($tag);

    /**
     * Returns the number of items that match all the specified tags.
     * If $mustHaveAll is set to false then this will return the number of items
     * that match at least one of the specified tags
     *
     * @param string[] $tags
     * @param Boolean  $mustHaveAll
     *
     * @return int
     */
    public function countByTags(array $tags = array(), $mustHaveAll = true);

}

```
