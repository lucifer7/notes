# Effective Java, 2nd Edition
## Lazy Initialization  Item-71
No necessary for lazy initialization unless:
1. To fix initialization circularity
2. Performance problem

Ways:
1. To solve initialization circularity, use synchronized methods
2. For high-performance on static field, use Field Lazy Initialization Holder


```
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}
static FieldType getField() {
    return FieldHolder.field;
}
```


3. For high-performance on instance field, use Double Check Lock(combined with modifier volatile)



