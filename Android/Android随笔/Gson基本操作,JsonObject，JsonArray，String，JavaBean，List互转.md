```java
String、JsonObject、JavaBean 互相转换
    User user = new Gson().fromJson(jsonObject, User.class);
    User user = new Gson().fromJson(string, User.class);
    String string = new Gson().toJson(user);
    JsonObject jsonObject = new Gson().toJsonTree(user).getAsJsonObject(); 
    JsonObject jsonObject = new JsonParser().parse(string).getAsJsonObject();
String、JsonArray、List互相转换
    List<User> userList = gson.fromJson(string, new TypeToken<List<User>>() {}.getType()); 
    List<User> userList = gson.fromJson(jsonArray, new TypeToken<List<User>>() {}.getType()); 
    String string = new Gson().toJson(userList); 
    JsonArray jsonArray = new Gson().toJsonTree(userList, new TypeToken<List<User>>() {}.getType()).getAsJsonArray();
    JsonArray jsonArray = new JsonParser().parse(string).getAsJsonArray();
```
