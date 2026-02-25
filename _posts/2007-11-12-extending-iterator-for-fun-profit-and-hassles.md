---
title: "Extending Iterator for fun, profit, and hassles"
date: 2007-11-12
author: Jonathan Owens
---

PHP's built-in [Iterator class](http://us.php.net/manual/en/language.oop5.iterations.php) is lovely, as is the [SPL's overcomplicated interpretation](http://us.php.net/manual/en/ref.spl.php). But as soon as you want to do something interesting, like say, go to the previous item in an object, both these implementations fall flat. That's right, you can't call `$iterator->prev()` on ANY SPL Iterator classes, because PHP's built-in Iterator doesn't have it! This seems to me a major oversight, especially considering the array functions that power most Iterator implementation have it [right there](http://us.php.net/manual/en/function.prev.php).

The other thing you can't do is focus an iterator on a particular item in it. This is more understandable, as arrays have no concept of this. Iterable objects should though.

So here at Treemo, we use lots of list objects. Lists of content, users, comments, etc. etc. Lists, obviously, should be iterable, but we'd like to be able to navigate in both directions, and focus on objects we're interested in.. So we cooked up a base class that does all its Iterator management by hand, because the only way to get this functionality cleanly is to do everything yourself. Because nobody should EVER have to do this twice, and because PHP didn't do it for me the way it ought to have, I'm gifting it to you.

```php
/**
 * A modifiable list object
 *
 * @package Core
 */
class ObjectList implements Iterator
{
  /**
   * The internal array backing for the list
   *
   * @var Array
   */
  protected $items;

  /**
   * The current index of the list.
   *
   * @var int
   */
  protected $key;

  /**
   * Create the list from either nothing or an array of existing items.
   *
   * @param Array $items
   */
  public function __construct ($items = null)
  {
    if (! is_null($items))
    {
      $this->items = $items;
    } else
    {
      $this->items = array();
    }
  }

  /**
   * Focus the list on its first element.
   */
  public function rewind ()
  {
    $this->key = 0;
  }

  /**
   * Get the next item in the list.
   *
   * @return Object
   */
  public function next ()
  {
    if (array_key_exists($this->key + 1, $this->items))
    {
      return $this->items[++ $this->key];
    } else
    {
      $this->key = count($this->items);
      return false;
    }
  }

  /**
   * Get the previous item in the list.
   *
   * @return Object
   */
  public function prev ()
  {
    if ($this->key < 0)
    {
      return false;
    }

    return $this->items[$this->key--];
  }

  /**
   * Focus the list on a particular object.
   *
   * @param object $item the needle
   * @return object or FALSE if the object was not found.
   */
  public function focus ($item)
  {
    $k = array_search($item, $this->items);
    if ($k !== false)
    {
      $this->key = $k;
      return $this->items[$this->key];
    }
    return false;
  }

  /**
   * Focus the list on its last item, and return it.
   *
   * @return object
   */
  public function end ()
  {
    $this->key = count($this->items) - 1;
    return $this->items[$this->key];
  }

  /**
   * Return the item that the list is currently focused on, or FALSE if it is
   * focused beyond the end of the list.
   *
   * @return object
   */
  public function current ()
  {
    return (isset($this->items[$this->key]) ? $this->items[$this->key] : false);
  }

  /**
   * Return the current list index.
   *
   * @return int
   */
  public function key ()
  {
    return $this->key;
  }

  /**
   * Return the status of the lists' index. Is it between 0 and the end of the list?
   *
   * @return bool
   */
  public function valid ()
  {
    return ($this->key >= 0) && ($this->key < count($this->items));
  }

  /**
   * Returns the length of the list.
   *
   * @return int
   */
  public function count ()
  {
    return count($this->items);
  }

  /**
   * Append an object onto the end of the in-memory list.
   *
   * @param object $newObject
   * @return int 1 for success, 0 for failure.
   */
  public function append ($newObject)
  {
    return array_push($this->items, $newObject);
  }

  /**
   * Prepend an object onto the beginning of the in-memory list.
   *
   * @param object $newObject
   * @return int 1 for success, 0 for failure
   */
  public function prepend ($newObject)
  {
    return array_unshift($this->items, $newObject);
  }

  /**
   * Insert an object into the list after $prevObject
   *
   * @param object $newObject
   * @param object $prevObject
   */
  public function insertAfter ($newObject, $prevObject)
  {
    $oldFocus = $this->key();

    $this->focus($prevObject);
    $k = $this->key();

    $this->items[$k] = $prevObject;

    $this->key = $oldFocus;
  }

}
```

Hope this is useful. Maybe I just miss Java a lot, but it really bugs me that more of this functionality isn't baked into PHP. Anyway, this class gets really interesting when you start extending it in a database-backed way:

```php
class UserFriendList extends ObjectList
{
  /**
   * Objects in this list are friends of this user
   *
   * @var User
   */
  protected $owner;

  public function __construct( User $user )
  {
    $this->owner = $user;

    $friendIds = $db->query( "SELECT user_id FROM friends WHERE owner_id = $user->user_id");
    foreach( $friendIds as $friend )
    {
      $this->items[] = new User( $friend );
    }
  }

  public function prepend( $newObject )
  {
    if( $db->query( "INSERT INTO friends (user_id, owner_id) VALUES ( $newObject->user_id, {$this->owner->user_id})"))
    {
      parent::prepend( $newObject );
    }
  }
}
```

...and so on and so forth. (Obviously this code is a quick example, don't run database queries like this!)

Good luck, and good coding!
