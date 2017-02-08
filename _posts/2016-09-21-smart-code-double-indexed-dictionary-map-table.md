---
layout: post
title: '[Smart Code] Double Indexed Dictionary, Map or Table'
---

I think that many of us need to index something using two or more fields. Well, the first and easiest way to solve it is using two Hastables (Hashmaps, Dictionaries,...) with the different fields as keys. Just like:

<!--more-->

> This is C# but in Java/Android/C++ have we have getters instead of indexed properties and we can use HashMap, Hashtables, etc...

```c#
public class DoubleIndexedDictionary<K1, K2, V>
{
    private Dictionary<K1, V> firstIndex;
    private Dictionary<K2, V> secondIndex;

    /* This can be a getValueByKey1 in other language */
    public V this[K1 key1]
    {
        get { return firstIndex[key1]; }
        set { firstIndex[key1] = value; }
    }

    /* This can be a getValueByKey2 in other language */
    public V this[K2 key2]
    {
        get { return secondIndex[key2]; }
        set { secondIndex[key2] = value; }
    }

    public DoubleIndexedCache()
    {
        firstIndex = new Dictionary<K1, V>();
        secondIndex = new Dictionary<K2, V>();
    }

    public void Add(K1 key1, K2 key2, V value)
    {
        if (firstIndex.ContainsKey(key1))
            throw new ArgumentException("Duplicated first key.");
        if (secondIndex.ContainsKey(key2))
            throw new ArgumentException("Duplicated second key.");

        firstIndex.Add(key1, value);
        secondIndex.Add(key2, value);
    }

    // etc...
}
```

With this approach we can't have many key 1 differenciated by the key 2, it has an overcost since when you add a method you should test that the dictionaries keeps sincronized.

And you end with O(n*2) cost when you insert, you have two make all the operations two times: Go through the keys, extend it when you need more space and save the reference.

And here comes the trick, you can use a custom Object with as many fields as indexed fields you want in the collection, override correctly the HashCode and Equals method and use it as key in our internal dictionary. If made correctly, you will get rid off the problems I mentioned before. Here is how I did it:

First the MultiKey Object for hold the keys:

```c#
public struct MultiKey<TKey1, TKey2>
{
    public readonly TKey1 Key1;
    public readonly TKey2 Key2;

    public MultiKey(TKey1 key1)
    {
        this.Key1 = key1;
        this.Key2 = default(TKey2);
    }

    public MultiKey(TKey2 key2)
    {
        this.Key1 = default(TKey1);
        this.Key2 = key2;
    }

    public MultiKey(TKey1 key1, TKey2 key2)
    {
        this.Key1 = key1;
        this.Key2 = key2;
    }

    public override Boolean Equals(Object obj)
    {
        MultiKey<TKey1, TKey2> other = (MultiKey<TKey1, TKey2>)obj;

        /* Only when one of the keys are default, to exclude it from the search */
        Boolean key1IsDefault = Object.Equals(this.Key1, default(TKey1)) ^ Object.Equals(other.Key1, default(TKey1));
        Boolean key2IsDefault = Object.Equals(this.Key2, default(TKey2)) ^ Object.Equals(other.Key2, default(TKey2));

        if (!key1IsDefault && !key2IsDefault)
            return Object.Equals(this.Key1, other.Key1) && Object.Equals(this.Key2, other.Key2);

        if (!key1IsDefault)
            return Object.Equals(this.Key1, other.Key1);

        if (!key2IsDefault)
            return Object.Equals(this.Key2, other.Key2);

        return false;
    }

    public override Int32 GetHashCode()
    {
        int xor = 0;

        if (!Object.Equals(this.Key1, default(TKey1)))
            xor = this.Key1.GetHashCode();

        if (!Object.Equals(this.Key2, default(TKey2)))
            xor ^= this.Key2.GetHashCode();

        return xor;
    }
}
```

The custom hashcode returns a the hashcodes of the keys xored. Doing this, I can know if the Key has the first, the second or the two keys when comparing it, I'll explain it in the comparator. The equals checks the items only when the keys in the two items are present, this behaviour can be changed to check the two keys always, but you will lost the flexibility to store multiple key2 with a single key1.

Now, the comparator:

```c#
public class MultiKeyComparator<TKey1, TKey2> : IEqualityComparer<MultiKey<TKey1, TKey2>>
{
    public Boolean Equals(MultiKey<TKey1, TKey2> left, MultiKey<TKey1, TKey2> right)
    {
        int leftHash = left.GetHashCode();
        int rightHash = right.GetHashCode();
        int xored = leftHash ^ rightHash;

        if (leftHash == rightHash || xored == left.Key1?.GetHashCode() || xored == left.Key2?.GetHashCode())
        {
            return left.Equals(right);
        }

        return false;
    }

    public Int32 GetHashCode(MultiKey<TKey1, TKey2> obj)
    {
        return 0;
    }
}
```

First, I override the HashCode method to return 0 so the dictionary will jump directly to the Equals. In the equals, I use the hashcodes in order to speed up the comparations, but with a custom check. Then, I check for the hashcode to being equal or the xored hashcodes to equal the first or seconde key. If the check are passed, we know that there's a probability for the objects to being equal.

And, finally, the DoubleIndexedDictionary class:

```c#
public class DoubleDictionary<K1, K2, V>
{
    private CartifDictionary<MultiKey<K1, K2>, V> innerDictionary;

    public IEnumerable<V> Values { get { return innerDictionary.Values; } }
    public IEnumerable<K1> FirstKeys { get { return innerDictionary.Keys.Select(k => k.Key1); } }
    public IEnumerable<K2> SecondKeys { get { return innerDictionary.Keys.Select(k => k.Key2); } }
    public IEnumerable<MultiKey<K1, K2>> Keys { get { return innerDictionary.Keys; } }

    public V this[K1 key1]
    {
        get { return innerDictionary[new MultiKey<K1, K2>(key1)]; }
        set { innerDictionary[new MultiKey<K1, K2>(key1)] = value; }
    }

    public V this[K2 key2]
    {
        get { return innerDictionary[new MultiKey<K1, K2>(key2)]; }
        set { innerDictionary[new MultiKey<K1, K2>(key2)] = value; }
    }

    public V this[K1 key1, K2 key2]
    {
        get { return innerDictionary[new MultiKey<K1, K2>(key1, key2)]; }
        set { innerDictionary[new MultiKey<K1, K2>(key1, key2)] = value; }
    }

    public DoubleDictionary()
    {
        innerDictionary = new CartifDictionary<MultiKey<K1, K2>, V>(new MultiKeyComparator<K1, K2>());
    }

    public DoubleDictionary(IEnumerable<V> values, Func<V, K1> selectorKey1, Func<V, K2> selectorKey2) : this()
    {
        AddRange(values, selectorKey1, selectorKey2);
    }

    public void Add(K1 key1, K2 key2, V value)
    {
        innerDictionary.Add(new MultiKey<K1, K2>(key1, key2), value);
    }

    public void AddRange(IEnumerable<V> values, Func<V, K1> selectorKey1, Func<V, K2> selectorKey2)
    {
        values.ThrowIfArgumentIsNull("values");
        selectorKey1.ThrowIfArgumentIsNull("selectorKey1");
        selectorKey2.ThrowIfArgumentIsNull("selectorKey2");

        foreach (V value in values)
            this.Add(selectorKey1(value), selectorKey2(value), value);
    }
}
```

As you see, you only have one dictionary built with the custom comparator. And indexed properties to get the values. But you will got a problem if the Keys are of the same class so one improve is to have the GetByKeyN methods to get the values when this case comes. I added also an AddRange method that uses to lambdas to add the values to the dictionaries. But what I want with this code is to provide a skeleton to build your own collection with the requierements you got. This class filled my needs so I didn't add more functionalities but you can do it and upload it (if you want) to the github repository (// TODO INSERT REPOSITORY HERE).

> I used C# but if you want the code in other language and translating it you get stuck, ask for help and I'll try to update this post and help you.
