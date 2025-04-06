Collisions in a `HashMap` occur when multiple keys hash to the same bucket index. This situation arises because different keys can produce the same hash value. To handle such cases, `HashMap` employs **separate chaining**, where it stores multiple key-value pairs in a bucket using a linked list (or a balanced tree for performance optimization, starting from Java 8).

Let’s dive into an example to understand how this works.

## The Code: Simulating Collisions

The following example demonstrates what happens when multiple keys in a `HashMap` collide due to having the same hash code:

```java
import java.util.HashMap;

public class HashMapCollisionExample {
    public static void main(String[] args) {

       var map = new HashMap<KeyWithSameHash, String>();
       map.put(new KeyWithSameHash("Key1"), "Value1");
       map.put(new KeyWithSameHash("Key2"), "Value2");
       map.put(new KeyWithSameHash("Key3"), "Value3");

       // prints Value3
       System.out.println(map.get(new KeyWithSameHash("Key3")));
    }
}

class KeyWithSameHash {
    private String key;

    public KeyWithSameHash(String key) {
        this.key = key;
    }

    @Override
    public int hashCode() {
        return 1001; // Causes all keys to hash to the same bucket
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        KeyWithSameHash other = (KeyWithSameHash) obj;
        return key.equals(other.key);
    }

    @Override
    public String toString() {
        return key;
    }
}
```

## Step-by-Step Breakdown

1. **Hash Code Calculation**  
   When you call `map.put(new KeyWithSameHash("Key3"), "Value3")`, the `hashCode()` method of the key returns `1001` for all instances, forcing all keys to map to the same bucket.

2. **Bucket Index Determination**  
   Although the hash code is `1001`, the bucket index is calculated using a bitwise operation (`hashCode % bucketArrayLength`), ensuring entries are distributed across available buckets. However, in this case, all keys land in the same bucket because their hash codes are identical.

3. **Collision Handling**  
   Since multiple keys are mapped to the same bucket, `HashMap` handles the collision using a linked list (or balanced tree). Each entry in the bucket is stored as a node in the list.

4. **Key Lookup**  
   When you retrieve a value using a key (e.g., `new KeyWithSameHash("Key3")`), the `HashMap`:
    - Finds the bucket for the key.
    - Traverses the linked list in that bucket.
    - Compares each key in the list using the `equals()` method until it finds a match.

5. **Value Retrieval**  
   Once the key is located (where `equals()` returns `true`), the corresponding value is returned.

## Retrieval Complexity


Despite all keys having the same hash code, the `HashMap` successfully retrieves the correct value due to its `equals()` comparison mechanism. However, retrieval complexity can degrade to (O(n)) if all keys hash to the same bucket and the bucket uses a linked list, or to (O(log n)) if the bucket uses a balanced tree (introduced in Java 8).

## Wrap Up

1. **Efficient Collision Handling:** `HashMap` can handle collisions gracefully using separate chaining, ensuring reliable behavior even when hash codes collide.
2. **Importance of Proper `hashCode` Implementation:** A poor hash code implementation, as shown in this example, can lead to performance degradation since all keys fall into the same bucket.
3. **Balanced Trees for Performance:** Starting with Java 8, `HashMap` replaces the linked list with a balanced tree when collisions exceed a threshold, improving lookup times in heavily collided buckets.

---

Happy coding! 💻
