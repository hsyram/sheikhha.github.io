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
<br/>
<br/>
<br/>


<h3>Item 2: Consider a builder when faced with many constructor parameters</h3>
<p>Static factories and constructors share a limitation: they do not scale well to large numbers of optional parameters.</p>
<p>Traditionally, programmers have used the <b>telescoping constructor pattern</b>, in which you provide a constructor with only the required parameters, another with a single optional parameter, a third with two optional parameters, and so on, culminating in a constructor with all the optional parameters.</p>
<p>A second alternative when you are faced with many constructor parameters is the <b>JavaBeans pattern</b>, in which you call a parameterless constructor to create the object and then call setter methods to set each required parameter and each optional parameter of interest.</p>
<p>There is a third alternative that combines the safety of the telescoping constructor pattern with the readability of the JavaBeans pattern. It is a form of the Builder pattern.</P>

{% highlight java %}
// Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
    
    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;
        // Optional parameters - initialized to default values
        private int calories      = 0;
        private int fat           = 0;
        private int carbohydrate  = 0;
        private int sodium        = 0;
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
		}
        public Builder calories(int val)
            { calories = val;      return this; }
        public Builder fat(int val)
            { fat = val;           return this; }
        public Builder carbohydrate(int val)
            { carbohydrate = val;  return this; }
        public Builder sodium(int val)
            { sodium = val;        return this; }
        public NutritionFacts build() {
            return new NutritionFacts(this); } 
	}
	
    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
	}
}

{% endhighlight %}
<p>Note that NutritionFacts is immutable, and that all parameter default values are in a single location. The builder’s setter methods return the builder itself so that invocations can be chained.</p>
<p>Here’s how the client code looks:</p>
{% highlight java %}
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).
     calories(100).sodium(35).carbohydrate(27).build();
{% endhighlight %}

<p><b>Note:</b> A minor advantage of builders over constructors is that builders can have multiple varargs parameters.</p>
<p>The Builder pattern is flexible. A single builder can be used to build multiple objects. The parameters of the builder can be tweaked between object creations to vary the objects. The builder can fill in some fields automatically, such as a serial number that automatically increases each time an object is created.</p>
<p>A builder whose parameters have been set makes a fine Abstract Factory</p>

{% highlight java %}
// A builder for objects of type T
   public interface Builder<T> {
       public T build();
   }
{% endhighlight %}

<ul><b>The Builder patternDisadvantage:</b>
<li>In order to create an object, you must first create its builder. While the cost of creating the builder is unlikely to be noticeable in practice, it could be a problem in some performance- critical situations.</li>
<li>The Builder pattern is more verbose than the telescoping constructor pattern.</li>
</ul>
<p><i>The Builder pattern is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters, especially if most of those parameters are optional. (four or more. But keep in mind that you may want to add parameters in the future)</i></p>
<br/>
<br/>
<br/>

<h3>Item 3: Enforce the singleton property with a private constructor or an enum type</h3>
{% highlight java %}
// Singleton with public final field
   public class Elvis {
       public static final Elvis INSTANCE = new Elvis();
       private Elvis() { ... }
       public void leaveTheBuilding() { ... }
   }
{% endhighlight %}

{% highlight java %}
// Singleton with static factory
public class Elvis {
private static final Elvis INSTANCE = new Elvis(); private Elvis() { ... }
public static Elvis getInstance() { return INSTANCE; }
       public void leaveTheBuilding() { ... }
   }
{% endhighlight %}

{% highlight java %}
// Enum singleton - the preferred approach
   public enum Elvis {
       INSTANCE;
       public void leaveTheBuilding() { ... }
   }
{% endhighlight %}
<br/>
<br/>
<br/>

<h3>Item 4: Enforce noninstantiability with a private constructor</h3>
<p>A class can be made noninstantiable by including a private constructor. Attempting to enforce noninstantiability by making a class abstract does not work. The class can be subclassed and the subclass instantiated. Furthermore, it misleads the user into thinking the class was designed for inheritance</p>

{% highlight java %}
 // Noninstantiable utility class
   public class UtilityClass {
       // Suppress default constructor for noninstantiability
       private UtilityClass() {
           throw new AssertionError();
		}
       ...  // Remainder omitted
   }
{% endhighlight %}
