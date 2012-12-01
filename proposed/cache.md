Common Interface for Caching libraries
====================================


This document describes a simple yet extensible interface for a cache item,
a cache driver and a cache proxy.

The final implementations should be allowed to decorate the objects with more
functionality that the one proposed but they MUST implement the indicated
interfaces first.

Also, since this involves caching, which is used to get better performance
from systems, the implementation detail should be as simple as possible.

For this reason, this document doesn't describe how tags, namespaces or
locking problems are not addressed and are left out to be implemented by
each vendor as it sees fit.

### CacheItem

Using the cache item approach to save the user that we can guarantee that
regardles of the driver implementation, the user will always be able to
retrieve the same data he expects to retrieve.

The item must store the user value as well as additional metadata for it.

The cache item should also contain a function that allows the user to retrieve
the remaining TTL of the item in order to better coordinate with its expiry
time.

### CacheDriver

A driver needs to be decoupled by the proxy so that it only implements the
basic operations described in either the DriverInterface or in
BatchDriverInterface or both.

It is the cache proxy responsabilty to serialize the information in the right
way if the driver doesn't support for serialization. When saving the cache item
the drivers should perform the save operation only.

### CacheProxy

There are two types of cache proxies that can be used.
The simple one, which provides the basic functionality like get/set/remove.

Cache proxies are reponsible for sending the right data to the drivers, be it
in the form of a serialized CacheItem or directly as a CacheItem object.

## Proposed implementation

### CacheItemInterface

```php

<?php

namespace PSR\Cache\Item;

/**
 * Interface for caching object
 */
interface CacheItemInterface
{
    /**
     * Set the value of the key to store our value under
     *
     * @param string $cacheKey
     *
     * @return CacheItemInterface
     */
    public function setKey($cacheKey);

    /**
     * Get the key of the object
     *
     * @return string
     */
    public function getKey();

    /**
     * Set the value to be stored in the cache
     *
     * @param mixed $cacheValue
     *
     * @return CacheItemInterface
     */
    public function setValue($cacheValue);

    /**
     * Get the value of the object
     *
     * @return mixed
     */
    public function getValue();

    /**
     * Set the TTL value
     *
     * @param int $ttl
     *
     * @return CacheItemInterface
     */
    public function setTtl($ttl);

    /**
     * Get the TTL of the object
     *
     * @return int
     */
    public function getTtl();

    /**
     * Get the remaining time in seconds until the item will expire
     * The implementation should save the expiry time in the item metadata on
     * save event and then retrieve it from the object metadata and substract
     * it from the current time
     *
     * *Note* certain delays can occur as the save event won't be able to
     * provide actual save time of when the user called the save method and
     * the real save time when the driver will save the item
     *
     * @return int
     */
    public function getRemainingTtl();

    /**
     * Set a metadata value
     *
     * @param string $key
     * @param mixed $value
     *
     * @return CacheItemInterface
     */
    public function setMetadata($key, $value);

    /**
     * Do we have any metadata with the object
     *
     * @param string|null $key
     *
     * @return boolean
     */
    public function hasMetadata($key = null);

    /**
     * Get parameter/key from the metadata
     *
     * @param string|null $key
     *
     * @return mixed
     */
    public function getMetadata($key = null);

}

```

### DriverInterface

```php

<?php

namespace PSR\Cache\Driver;

/**
 * Interface for cache drivers
 */
interface DriverInterface
{
    /**
     * Set data into cache.
     *
     * @param string $key      Entry id
     * @param mixed  $value    Cache entry
     * @param int    $lifeTime Life time of the cache entry
     *
     * @return boolean
     */
    public function set($key, $value, $lifeTime = 0);

    /**
     * Check if an entry exists in cache
     *
     * @param string $key Entry id
     *
     * @return boolean
     */
    public function exists($key);

    /**
     * Get an entry from the cache
     *
     * @param string $key Entry id
     * @param boolean|null $exists If the operation was succesfull or not
     *
     * @return mixed The cached data or FALSE
     */
    public function get($key, &$exists = null);

    /**
     * Removes a cache entry
     *
     * @param string $key Entry id
     *
     * @return boolean
     */
    public function remove($key);

    /**
     * If this driver has support for serialization or not
     *
     * @return boolean
     */
    public function hasSerializationSupport();

}

```

### BatchDriverInterface

```php

<?php

namespace PSR\Cache\Driver;

/**
 * Interface for cache drivers that can support multiple operations at once
 */
interface BatchDriverInterface extends DriverInterface
{

    /**
     * Stores multiple items in the cache at once.
     *
     * The items must be provided as an associative array.
     *
     * @param array $items
     * @param int   $ttl
     *
     * @return boolean[]
     */
    public function setMultiple(array $items, $ttl = 0);

    /**
     * Fetches multiple items from the cache.
     *
     * The returned structure must be an associative array. If items were
     * not found in the cache, they should not be included in the array.
     *
     * This means that if none of the items are found, this method must
     * return an empty array.
     *
     * @param array $keys
     *
     * @return array
     */
    public function getMultiple(array $keys);

    /**
     * Deletes multiple items from the cache at once.
     *
     * @param array $keys
     *
     * @return boolean[]
     */
    public function removeMultiple(array $keys);

    /**
     * Check for multiple items if they appear in the cache.
     *
     * All items must be returned as an array. And each must array value
     * must either be set to true, or false.
     *
     * @param array $keys
     *
     * @return array
     */
    public function existsMultiple(array $keys);

    /**
     * If this driver has support for serialization or not
     *
     * @return boolean
     */
    public function hasSerializationSupport();

}

```

### CacheProxyInterface

```php

<?php

namespace PSR\Cache;

use PSR\Cache\Item\CacheItemInterface;

/**
 * This is our cache proxy
 */
class CacheProxyInterface
{

    /**
     * Get the default TTL of the instance
     *
     * @return int
     */
    public function getDefaultTtl();

    /**
     * Set the default TTL of the instance
     *
     * @param $defaultTtl
     *
     * @return CacheDriver
     */
    public function setDefaultTtl($defaultTtl);

    /**
     * Get cache entry
     *
     * @param string|CacheItemInterface $key
     * @param boolean|null $exists
     *
     * @return CacheItemInterface
     */
    public function get($key, &$exists = null);

    /**
     * Check if a cache entry exists
     *
     * @param string|CacheItemInterface $key
     *
     * @return boolean
     */
    public function exists($key);

    /**
     * Set a single cache entry
     *
     * @param CacheItemInterface $cacheItem
     *
     * @return boolean Result of the operation
     */
    public function set(CacheItemInterface $cacheItem);

    /**
     * Remove a single cache entry
     *
     * @param string|CacheItemInterface $key
     *
     * @return boolean Result of the operation
     */
    public function remove($key);

    /**
     * Set multiple keys in the cache
     * If $ttl is not passed then the default TTL for this driver will be used
     *
     * @param string[]|CacheItemInterface[]|mixed[] $items
     * @param null|int $ttl
     */
    public function setMultiple(array $items, $ttl = null);

    /**
     * Get multiple keys the cache
     *
     * @param string[]|CacheItemInterface[]|mixed[] $keys
     *
     * @return CacheItemInterface[]
     */
    public function getMultiple($keys);

    /**
     * Remove multiple keys from the cache
     *
     * @param string[]|CacheItemInterface[]|mixed[] $keys
     */
    public function removeMultiple($keys);

    /**
     * Check if multiple keys exists in the cache
     *
     * @param string[]|CacheItemInterface[]|mixed[] $keys
     *
     * @return boolean[]
     */
    public function existsMultiple($keys);

}

```