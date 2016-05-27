1. Boolean exception check
from Guava
Preconditions.java
<pre>
public static void checkArgument(boolean expression) {
    if (!expression) {
      throw new IllegalArgumentException();
    }
}

...

    checkArgument(concurrencyLevel > 0);

</pre>

1. Static method use non-static(instance) field
<pre>
public static Car newInstance(Car car) {
  return new Car(car);
}
</pre>