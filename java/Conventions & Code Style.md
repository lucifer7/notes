### 1. String
#### 1.1 Prefer char[] over String for security sensitive system
String is immutable, waiting for GC kicks in, use char[] (e.g. password) will not be present