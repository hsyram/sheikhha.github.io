---
layout: post
title:  "Effective Java Summery"
date:   2016-12-26 23:00:06 +0200
categories: java
---
<h2> 1. Creating and Destroying Objects</h2>
<h3>Item 1: Consider static factory methods instead of constructors</h3>
<p>A class can provide a public static factory method, which is simply a static method that returns an instance of the class.</p>

{% highlight java %}
public static Boolean valueOf(boolean b) {
       return b ? Boolean.TRUE : Boolean.FALSE;
}
{% endhighlight %}

<p>Note that a static factory method is not the same as the Factory Method pattern from Design Patterns.</p>

<h4>Advantages and Disadvantages</h4>
<ul>Advantages:
<li>Unlike constructors, they have names.</li>
<li>Unlike constructors, they are not required to create a new object each time they’re invoked.</li>
<li>Unlike constructors, they can return an object of any subtype of their return type.</li>
<li>They reduce the verbosity of creating parameterized type instances.</li>

{% highlight java %}
Map<String, List<String>> m = new HashMap<String, List<String>>();

//With static factories, the compiler can figure out the type parameters for you.
//This is known as type inference.
public static <K, V> HashMap<K, V> newInstance() {
       return new HashMap<K, V>();
}
Map<String, List<String>> m = HashMap.newInstance();
{% endhighlight %}
</ul>
<br/>

<ul>Disadvantages:
<li>The main disadvantage of providing only static factory methods is that classes without public or protected constructors cannot be subclassed.</li>
<li>They are not readily distinguishable from other static methods.</li>
</ul>
<br/>

<ul>Here are some common names for static factory methods:
<li><b>valueOf—</b> Returns an instance that has, loosely speaking, the same value as its parameters. Such static factories are effectively type-conversion methods.</li>
<li><b>of—</b> A concise alternative to valueOf, popularized by EnumSet (Item 32).</li>
<li><b>getInstance—</b> Returns an instance that is described by the parameters but cannot be said to have the same value. In the case of a singleton, getInstance takes no parameters and returns the sole instance.</li>
<li><b>newInstance—</b> Like getInstance, except that newInstance guarantees that each instance returned is distinct from all others.</li>
<li><b>getType—</b> Like getInstance, but used when the factory method is in a differ- ent class. Type indicates the type of object returned by the factory method.</li>
<li><b>newType—</b> Like newInstance, but used when the factory method is in a differ- ent class. Type indicates the type of object returned by the factory method.</li>
</ul>
<br/>

<h4>service provider frameworks:</h4>
<p>The class of the object returned by a static factory method need not even exist at the time the class containing the method is written. Such flexible static factory methods form the basis of service provider frameworks</p>

{% highlight java %}
// Service provider framework sketch
   // Service interface
   public interface Service {
       ... // Service-specific methods go here
}
   // Service provider interface
   public interface Provider {
       Service newService();
}
   // Noninstantiable class for service registration and access
   public class Services {
       private Services() { }  // Prevents instantiation (Item 4)
       // Maps service names to services
       private static final Map<String, Provider> providers =
           new ConcurrentHashMap<String, Provider>();
       public static final String DEFAULT_PROVIDER_NAME = "<def>";
       // Provider registration API
       public static void registerDefaultProvider(Provider p) {
           registerProvider(DEFAULT_PROVIDER_NAME, p);
}
public static void registerProvider(String name, Provider p){
           providers.put(name, p);
       }
       // Service access API
       public static Service newInstance() {
           return newInstance(DEFAULT_PROVIDER_NAME);
       }
       public static Service newInstance(String name) {
           Provider p = providers.get(name);
           if (p == null)
               throw new IllegalArgumentException(
                   "No provider registered with name: " + name);
           return p.newService();
       }
}
{% endhighlight %}


