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

1. Architecture example
spring boot+spring cloud+kafka+redis+Oracle12c 和+fastdfs+hadoop+spark<-后台
	前端是nginx+haproxy+keepalived+cdn
	监控是zabbix+elk
